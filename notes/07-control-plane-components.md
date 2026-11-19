# 07 · Control-Plane Components in Depth

> **Level:** ★★ intermediate
>
> **You will learn:** how each control-plane component really works: the API server's request pipeline, watches and resource versions; etcd's key layout, the Raft algorithm and quorum; how the scheduler filters and scores nodes; how controllers use informers and work queues to reconcile; leader election; and what the cloud controller manager does. Plus the ports, files and commands to inspect each one.
>
> **You need:** topic 06.

---

## ⚡ Core: the 10-minute version

**The idea.** Each control-plane component has one job:
- **API server**: checks every request (who are you, are you allowed, is the object valid), stores it in **etcd**, and tells watchers about the change.
- **etcd**: keeps the data consistent across members using the **Raft** protocol. It needs a **majority** (quorum) to accept writes.
- **scheduler**: filters out nodes that can't run a Pod, scores the rest, and writes the winner into `spec.nodeName`.
- **controllers**: run endless **reconcile loops** (watch → compare desired vs actual → act). Only one copy of each is active at a time (leader election).

**Picture it.** The API server is the receptionist and records clerk; etcd is the filing cabinet; the scheduler is the seating planner; controllers are supervisors who keep checking the plan against reality.

**The one thing to know.**

```bash
kubectl get --raw='/readyz?verbose' | tail -5         # API server health checks
kubectl -n kube-system get leases                     # who leads scheduler/controller-manager
```

**Try it now:**
1. Run both commands above.
2. `docker exec lab-control-plane ls /etc/kubernetes/manifests`: the control plane is defined by these files.
3. Watch a controller work: `kubectl get events -w` in one terminal, `kubectl scale deploy web --replicas=4` in another.

**You've got it if you can:** describe the API server request pipeline and what quorum means for etcd.

> 📖 **That's the core.** On a first pass, do the mini-lab and move on to the next topic. Everything below is the **full reference**: come back for the details, the edge cases, interview answers and exam-level depth.

---

## What it is

The **control plane** is five programs. Each is a normal Linux process; on kubeadm and kind clusters they run as **static Pods** on control-plane nodes (topic 42), and on managed clusters (EKS/GKE/AKS) the cloud runs them for you.

| Component | Day 07 / Day 25 summary | Default port |
|---|---|---|
| **kube-apiserver** | "The front end of the control plane. Exposes the Kubernetes API so users and tools can interact with the cluster." "Validates and processes REST API requests, enforcing security policies." | 6443 (HTTPS) |
| **etcd** | "A distributed, consistent key-value store that stores all the cluster's configuration data and state." | 2379 (clients), 2380 (peers) |
| **kube-scheduler** | "Assigns Pods to worker nodes based on resource availability and other constraints / scheduling policies." | 10259 (health/metrics) |
| **kube-controller-manager** | "Implements control loops that keep the desired state: replication, scaling, endpoint management, garbage collection." | 10257 (health/metrics) |
| **cloud-controller-manager** | "Integrates Kubernetes with the cloud provider: external load balancers, storage provisioning, node lifecycle." | 10258 |

---

## Why it exists

One giant program would be hard to scale, secure and replace. Splitting the brain into **one store (etcd)**, **one gateway (API server)** and **many independent loops (scheduler, controllers)** gives you:
- **One security choke point**: every change is authenticated, authorised and checked once, in the API server.
- **Replaceable parts**: you can run a second scheduler, add your own controllers, or swap the cloud integration.
- **Crash safety**: every component is **stateless** except etcd. Restart any of them and it simply re-reads the state from the API server.

---

## Analogy

A **newspaper**:
- **etcd** = the **archive**: the official record of everything published.
- **API server** = the **editor's desk**: every story must pass the desk, which checks who sent it (authentication), whether they're allowed (authorisation), and whether it follows the style guide (validation and admission) before filing it in the archive. Anyone can subscribe to updates from the desk.
- **Scheduler** = the **assignment editor** who sees unassigned stories and picks a reporter for each.
- **Controllers** = **section editors**: each watches one kind of story and keeps nudging until reality matches the plan.
- **Cloud controller manager** = the **liaison** with the printing company outside the building.

---

## How it works inside

### 1. kube-apiserver: the request pipeline

Every request (`kubectl apply`, a kubelet status update, a controller creating a Pod) goes through the same stages:

```
HTTPS request
  → 1. Authentication   "who are you?"          (client certificate, bearer token, OIDC…)   topic 54
  → 2. Authorization    "may you do this?"       (Node, RBAC, Webhook…)                       topics 56–57
  → 3. Mutating admission   "change/default it"  (e.g. add default ServiceAccount)           topic 58
  → 4. Schema validation    "is it a valid object?"
  → 5. Validating admission "does it follow policy?" (quotas, Pod Security…)                 topics 58, 60
  → 6. Persist to etcd
  → 7. Notify all watchers of the change
```

Key ideas:
- **REST API**: objects are URLs. A Pod named `web` in namespace `default` lives at `/api/v1/namespaces/default/pods/web`. `GET` reads it, `POST` creates, `PUT`/`PATCH` update, `DELETE` deletes (topic 14).
- **Watch**: a client can say "send me every change to Pods from now on". The API server keeps the HTTP connection open and **streams events** (`ADDED`, `MODIFIED`, `DELETED`). This is how the scheduler, controllers and kubelets learn about work **instantly** without polling.
- **resourceVersion**: every object carries a version number (taken from etcd). It is used for:
  - **Optimistic concurrency**: if two clients update the same object, the second one's write is rejected with **`409 Conflict`** ("the object has been modified") because its `resourceVersion` is stale. It must re-read and retry.
  - **Resuming watches** from a known point.
- **Stateless and scalable**: you can run several API servers behind a load balancer (HA, topic 78). They all share one etcd.
- **Aggregation layer**: the API server can **proxy** some API groups to other servers (e.g. `metrics.k8s.io` → metrics-server, topic 48).

### 2. etcd: the cluster database

- A **key-value store**: keys are paths, values are the objects encoded in **protobuf** (a compact binary format):
  ```
  /registry/pods/default/web
  /registry/deployments/default/web
  /registry/secrets/kube-system/bootstrap-token-abc123
  ```
- **Raft consensus:** you run an **odd number** of etcd members (1, 3 or 5). One is the **leader**; writes go through it and are only **committed** when a **majority (quorum)** of members have saved them to disk.

| Members | Quorum | Failures tolerated |
|---|---|---|
| 1 | 1 | 0 |
| 3 | 2 | **1** |
| 5 | 3 | 2 |

  Lose quorum (e.g. 2 of 3 members down) and etcd **refuses writes**: the API server can't change anything.
- **Disk speed matters most.** Every write is `fsync`ed to disk. Slow disks lead to slow leader heartbeats, then leader elections and API timeouts. Production etcd wants SSDs with low fsync latency.
- **MVCC** (multi-version concurrency control): etcd keeps old versions of keys (that's where `resourceVersion` comes from) until **compaction** removes them. **Defragmentation** reclaims the disk space afterwards. The default size quota is 2 GiB; if exceeded, etcd raises a `NOSPACE` alarm and goes read-only.
- **Secrets are stored here.** By default they're only base64-encoded, so you should enable **encryption at rest** (topic 35) and protect etcd backups (topic 77).
- **Backups:** `etcdctl snapshot save`. Losing etcd without a backup means losing the cluster (topic 77).

### 3. kube-scheduler: choosing a node

The scheduler watches for Pods whose **`spec.nodeName` is empty**. For each one it runs a **scheduling cycle**:

1. **Filter** (also called "predicates"): remove nodes that **can't** run the Pod:
   - not enough unrequested CPU/memory for the Pod's **requests** (topic 41);
   - a **taint** the Pod doesn't tolerate (topic 43);
   - not matching **nodeSelector/node affinity** (topic 44);
   - the requested `hostPort` is already used;
   - the volume is in another zone (topic 39).
2. **Score** ("priorities"): rank the remaining nodes from 0 to 100 with plugins such as "least allocated" (spread load), "image locality" (image already cached), affinity preferences (topic 44) and topology spread (topic 46).
3. **Bind**: create a **Binding** for the highest-scoring node. The API server then sets `pod.spec.nodeName`.

If no node passes the filter, the Pod stays **`Pending`** with an event such as `0/3 nodes are available: 3 Insufficient cpu`. If the Pod has a high priority, the scheduler may **preempt** (evict) lower-priority Pods to make room (topic 47).

The scheduler doesn't start containers or check that the Pod actually fits at runtime; it only records a decision. The kubelet does the rest.

### 4. kube-controller-manager: reconciliation loops

A single binary that runs **dozens of controllers**, each a loop:

```
for ever:
    desired := read from API server (via cache)
    actual  := read from API server (via cache)
    if desired != actual: take one step to fix it (through the API server)
```

Important controllers:

| Controller | Watches | Does |
|---|---|---|
| Deployment | Deployments, ReplicaSets | Creates/scales ReplicaSets for rollouts (topic 25) |
| ReplicaSet | ReplicaSets, Pods | Creates/deletes Pods to match `replicas` (topic 24) |
| Node (node lifecycle) | Nodes, leases | Marks nodes `NotReady` when heartbeats stop; taints them so Pods get evicted (topic 43) |
| EndpointSlice | Services, Pods | Keeps the list of ready Pod IPs behind each Service (topic 32) |
| Job / CronJob | Jobs, CronJobs | Runs Pods to completion, on schedules (topic 28) |
| Namespace | Namespaces | Deletes everything inside a namespace being deleted |
| ServiceAccount / token | Namespaces, ServiceAccounts | Creates the `default` ServiceAccount in each namespace (topic 55) |
| Garbage collector | Owner references | Deletes children whose owner is gone (delete a Deployment → its ReplicaSets and Pods go too) |
| PersistentVolume binder | PVs, PVCs | Binds claims to volumes (topic 38) |

**How controllers stay fast: informers.** A controller doesn't hammer the API server with `GET` calls. It uses an **informer**: one `LIST` at start-up to fill a local **cache**, then a **watch** to keep the cache up to date. Changes go into a **work queue**; workers pop keys (`namespace/name`) and reconcile them. If an action fails, the key is re-queued with a back-off. This pattern (informer + cache + work queue + reconcile) is exactly what you'll use if you write an operator (topics 66–67).

**Level-triggered, not edge-triggered.** A controller doesn't care *what changed*. It just compares the full desired state with the full actual state every time. So if it misses an event or restarts, the next loop still fixes everything. That's what makes Kubernetes self-healing.

### 5. Leader election

You run 3 control-plane nodes for HA, so there are 3 schedulers and 3 controller managers. If all 3 acted at once, they'd fight (three Pods created for one missing replica). So each one competes for a **Lease** object in `kube-system`. Only the holder (the **leader**) works; the others wait and take over if the leader stops renewing its lease (default lease about 15 s).

```bash
kubectl get lease -n kube-system
```

```
NAME                      HOLDER                                               AGE
kube-controller-manager   lab-control-plane_8a1c0d2e-...                       2d
kube-scheduler            lab-control-plane_f3b2a9c1-...                       2d
```

The API server doesn't need leader election: every instance can serve requests at the same time.

### 6. cloud-controller-manager

Cloud-specific logic lives outside the core so Kubernetes doesn't depend on any one cloud. The CCM runs:
- **Node controller**: when a node registers, it labels it with instance type, region and zone (`topology.kubernetes.io/zone`); when the VM is deleted in the cloud, it deletes the Node object.
- **Route controller**: on some clouds, programs VPC routes for Pod subnets.
- **Service controller**: when you create a `Service` of `type: LoadBalancer`, it creates a **real cloud load balancer** and writes its address into the Service's status (topic 31).

On AWS, the "AWS Load Balancer Controller" is a separate add-on with richer features (topic 70). On kind and bare metal there's no CCM; tools like MetalLB or cloud-provider-kind fill the gap.

---

## Commands / YAML

### Look at the control-plane Pods and their flags

```bash
kubectl -n kube-system get pod kube-apiserver-lab-control-plane -o yaml | grep -A40 'command:'
```

```
  - command:
    - kube-apiserver
    - --advertise-address=172.18.0.3
    - --authorization-mode=Node,RBAC
    - --client-ca-file=/etc/kubernetes/pki/ca.crt
    - --enable-admission-plugins=NodeRestriction
    - --etcd-servers=https://127.0.0.1:2379
    - --secure-port=6443
    - --service-cluster-ip-range=10.96.0.0/16
    - --tls-cert-file=/etc/kubernetes/pki/apiserver.crt
    ...
```

- `--authorization-mode=Node,RBAC` = which authorisers run, in order (topic 56).
- `--enable-admission-plugins` = extra admission plugins (topic 58).
- `--etcd-servers` = where etcd is; the API server is etcd's only client.
- `--service-cluster-ip-range` = the pool ClusterIPs come from (topic 30).
- `--client-ca-file`, `--tls-cert-file` = TLS settings (topic 53).

The same flags live in `/etc/kubernetes/manifests/kube-apiserver.yaml` on the control-plane node.

### Health endpoints

```bash
kubectl get --raw='/readyz?verbose' | tail -5
```

```
[+]poststarthook/apiservice-registration-controller ok
[+]poststarthook/apiservice-status-available-controller ok
[+]etcd ok
[+]etcd-readiness ok
readyz check passed
```

- `/readyz` = "ready to serve traffic?"; `/livez` = "alive, or should I be restarted?". `[+]` = check passed.

> ℹ️ `kubectl get componentstatuses` (`kubectl get cs`) is **deprecated** (since v1.19) and unreliable. Use `/readyz`, `/livez`, and the static Pods' status instead.

### Talk to etcd directly (control-plane node)

```bash
docker exec -it lab-control-plane bash          # kind: enter the node
ETCDCTL_API=3 etcdctl \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key \
  get /registry/deployments --prefix --keys-only | head
```

```
/registry/deployments/kube-system/coredns
/registry/deployments/default/web
```

- `--cacert/--cert/--key` = etcd only accepts clients with a certificate signed by its CA (mutual TLS, topic 53).
- `--prefix --keys-only` = list keys under a path without printing the binary values.
- (If `etcdctl` isn't installed in the node, run it from inside the etcd Pod: `kubectl -n kube-system exec etcd-lab-control-plane -- etcdctl …`.)

```bash
etcdctl … endpoint status -w table
```

```
+------------------------+------------------+---------+---------+-----------+-----------+------------+
|        ENDPOINT        |        ID        | VERSION | DB SIZE | IS LEADER | RAFT TERM | RAFT INDEX |
+------------------------+------------------+---------+---------+-----------+-----------+------------+
| https://127.0.0.1:2379 | 8e9e05c52164694d |   3.6.4 |  5.2 MB |      true |         2 |      18342 |
+------------------------+------------------+---------+---------+-----------+-----------+------------+
```

- `DB SIZE` = watch this against the 2 GiB default quota.
- `IS LEADER` = this member leads (always true with one member).
- `RAFT TERM` = increases on every leader election; a fast-rising term means an unstable cluster (usually slow disks or network).

### Watch the scheduler's decisions

```bash
kubectl get events --field-selector reason=Scheduled -A | tail -3
```

```
default   12s   Normal   Scheduled   pod/web-6d4b7c9f8-k2xjp   Successfully assigned default/web-6d4b7c9f8-k2xjp to lab-worker2
```

---

## Hands-on lab

On your kind cluster (topic 11):

1. Watch the API server stream events: in one terminal run `kubectl get pods --watch`; in another, `kubectl run w --image=nginx`, then `kubectl delete pod w`. You'll see the Pod go through each state in real time. That's the same watch mechanism controllers use.
2. Trigger an optimistic-concurrency conflict:
   ```bash
   kubectl create deployment cc --image=nginx
   kubectl get deploy cc -o yaml > a.yaml
   kubectl scale deploy cc --replicas=2          # changes resourceVersion
   kubectl replace -f a.yaml
   ```
   ```
   Error from server (Conflict): error when replacing "a.yaml": Operation cannot be fulfilled on deployments.apps "cc": the object has been modified; please apply your changes to the latest version and try again
   ```
3. List etcd keys for your Deployment with the `etcdctl` command above.
4. Make a Pod unschedulable: `kubectl run big --image=nginx --overrides='{"spec":{"containers":[{"name":"big","image":"nginx","resources":{"requests":{"cpu":"500"}}}]}}'`, then `kubectl describe pod big`. Read the `FailedScheduling` event: that's the filter phase reporting why every node failed.
5. Check the leader leases with `kubectl get lease -n kube-system`.
6. Clean up: `kubectl delete deploy cc; kubectl delete pod big`.

---

## Where you'll use it

- **Debugging Pending Pods**: the scheduler's `FailedScheduling` message tells you which filter failed.
- **Operating etcd**: backups, defrag, quota alarms, disk latency (topic 77).
- **CI/CD and operators**: handling `409 Conflict` retries and understanding watches.
- **HA design**: odd etcd member counts and leader election (topic 78).

---

## When it breaks

| Symptom | Likely component and cause | Check |
|---|---|---|
| `The connection to the server … was refused` | API server down (bad flag in its manifest, cert expired) | `crictl ps -a \| grep apiserver`, its logs — topic 82 |
| API slow, `etcdserver: request timed out` | etcd slow disk or lost quorum | `etcdctl endpoint status/health`, disk latency |
| `etcdserver: mvcc: database space exceeded` | etcd over quota | compact + defrag + `alarm disarm` (topic 77) |
| New Pods stay `Pending`, no events at all | Scheduler down | `kubectl -n kube-system get pods` |
| Deployments don't create Pods, deleted Pods not replaced | Controller manager down | its Pod and logs |
| `409 Conflict` in a script | Normal optimistic concurrency | Re-read and retry, or use `kubectl apply`/patch |

---

## Quick recap

- **API server**: the only door; pipeline = authn → authz → mutating admission → validation → validating admission → etcd → notify watchers.
- **etcd**: Raft, odd members, quorum; fast disks; back it up; encrypt Secrets.
- **Scheduler**: filter → score → bind; sets `nodeName`.
- **Controllers**: level-triggered reconcile loops using informers, caches and work queues; leader election for HA.
- **CCM**: cloud load balancers, node labels/cleanup, routes.

### Interview questions

1. **Q: Why should etcd have an odd number of members?**
   A: Quorum is a majority. 3 members tolerate 1 failure, 4 members still tolerate only 1 (quorum 3), so the even member adds cost without adding fault tolerance.
2. **Q: How does the scheduler pick a node?**
   A: It filters out nodes that can't fit the Pod (resources, taints, affinity, ports, volumes), scores the rest with weighted plugins, and binds the Pod to the best node by setting `nodeName`.
3. **Q: What does "level-triggered" mean for controllers, and why does it matter?**
   A: They reconcile the whole current state each time instead of reacting only to individual events, so missed events or restarts don't leave the cluster wrong; the next loop corrects it.

---

## Appendix: notes on the original course material

This topic was rebuilt from the original CKA course notes. This appendix keeps the audit trail out of your way while you learn.

### Sources

Day 07 (control-plane component list), Day 25 (control-plane components). Much of this is new depth. The same sections reappear in `Storage-In-Kubernetes-Masterclass/README.md` (Days 24–28).
