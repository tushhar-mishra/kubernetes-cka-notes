# 49 · VPA, In-Place Pod Resize and Node Autoscaling (Cluster Autoscaler, Karpenter, KEDA)

> **Level:** ★★★ advanced
>
> **You will learn:** the **Vertical Pod Autoscaler** (why, components, update modes including the new `InPlaceOrRecreate`, recommendation fields, the demo); **in-place Pod resize** (GA in v1.35: the `resize` subresource, `resizePolicy`, resize status); combining HPA and VPA safely; and scaling the **nodes**: **Cluster Autoscaler** (node groups, scale-up on Pending Pods, scale-down of underused nodes), **Karpenter** (just-in-time, right-sized nodes, consolidation), and **KEDA** (event-driven scaling, scale to zero).
>
> **You need:** topics 41, 47 and 48.

---

## ⚡ Core: the 10-minute version

**The idea.** Three more ways to scale:
- **Vertical Pod Autoscaler (VPA)**: recommends, or applies, **bigger or smaller requests** per Pod. It's great for right-sizing.
- **In-place Pod resize** (GA in v1.35): change a running Pod's CPU or memory without recreating it.
- **Node autoscaling** adds and removes **nodes**. **Cluster Autoscaler** adds nodes when Pods are Pending for lack of room; **Karpenter** picks right-sized instances; **KEDA** scales on events and down to zero.

**Picture it.** HPA hires more staff; VPA gives each worker a bigger desk; the cluster autoscaler rents another office floor.

**The one thing to know.**

```bash
# resize a running container in place (no new Pod)
kubectl patch pod web --subresource resize --patch \
  '{"spec":{"containers":[{"name":"nginx","resources":{"requests":{"cpu":"200m"},"limits":{"cpu":"400m"}}}]}}'
kubectl get pod web -o jsonpath='{.status.containerStatuses[0].resources}'; echo
```

**Try it now:**
1. Run a Pod with CPU requests/limits.
2. Resize it with the command above.
3. Confirm the RESTARTS count didn't change.
4. (Optional) Install VPA in `Off` mode and read its recommendations with `kubectl describe vpa`.

**You've got it if you can:** say when to use HPA vs VPA vs a node autoscaler, and why HPA and VPA shouldn't both act on CPU.

> 📖 **That's the core.** On a first pass, do the mini-lab and move on to the next topic. Everything below is the **full reference**: come back for the details, the edge cases, interview answers and exam-level depth.

---

## What it is

- The **Vertical Pod Autoscaler (VPA)** automatically adjusts a workload's **CPU and memory requests (and limits proportionally)** based on **actual usage**. It doesn't change replica counts. It ensures Pods have **enough** to run efficiently while **minimising waste** from over-provisioning. It's an add-on from the `kubernetes/autoscaler` project, not built into Kubernetes.
- **In-place Pod resize** is the Kubernetes feature that lets a running Pod's resources change **without recreating it**: the basis for non-disruptive vertical scaling.
- **Node autoscalers** add and remove **nodes**: **Cluster Autoscaler** (CA; works with cloud node groups) and **Karpenter** (provisions individual right-sized instances directly).
- **KEDA** (Kubernetes Event-Driven Autoscaling) scales workloads on **event sources** (queues, streams, cron, Prometheus queries), including to **zero**.

---

## Why it exists

**Why VPA:**
- **Removes guesswork**: no manual tuning of CPU/memory requests.
- Ideal for **stateful or single-replica** apps where adding replicas (HPA) doesn't help.
- Handles **dynamic workloads** whose needs change over time.
- Even if you never let it act, its **recommendations** fix the most common production problem: requests that are wildly wrong (wasting money or causing OOMs/throttling).

**Why node autoscaling:** the HPA can add Pods, but if no node has room they sit `Pending`. Something has to add machines, and remove them when they're idle to save money.

---

## Analogy

- **HPA** = hiring **more** cashiers when the shop is busy.
- **VPA** = giving each cashier a **bigger or smaller desk** based on how much they actually use. Old way: they had to move to a new desk (Pod recreated). **In-place resize**: the desk is extended while they keep working.
- **Cluster Autoscaler** = renting another **floor of the same standard size** when all desks are taken.
- **Karpenter** = renting **exactly the room size you need, right now**, and moving people around to give back half-empty rooms (consolidation).
- **KEDA** = opening a till the moment a **queue** appears, and closing the shop completely when nobody's there.

---

## How it works inside

### VPA components

1. **Recommender**: continuously analyses **historical and current usage** (from metrics-server, optionally Prometheus) and computes optimal requests. It uses decaying histograms of usage (recent data weighs more), plus a safety margin, and adapts over time.
2. **Updater**: finds Pods whose requests are far from the recommendation and **applies** it, by **evicting** them (they're recreated with new values) or, in `InPlaceOrRecreate` mode, by **resizing in place**. It respects **PodDisruptionBudgets** (PDBs: rules that keep a minimum number of Pods available during voluntary disruptions; topic 76).
3. **Admission controller** (a mutating webhook; topic 58): **intercepts Pod creation** and **rewrites** the requests to the current recommendation, so new Pods start right-sized.

### VPA update modes

| Mode | Behaviour |
|---|---|
| **`Off`** | **Recommend only**: nothing is changed |
| **`Initial`** | Set recommended values **only when Pods are created**; never touch running Pods |
| **`Recreate`** | Apply to running Pods by **evicting and recreating** them |
| **`InPlaceOrRecreate`** | Try an **in-place resize** first; fall back to eviction if the node can't fit it or the resize stalls |
| ~~`Auto`~~ | (Day 20's mode) Actively update running Pods. **Deprecated** in recent VPA releases; currently behaves like `Recreate`. Use `Recreate` or `InPlaceOrRecreate` explicitly. |

> ✅ **Correct today** ([C1](#c1)): in-place Pod resize was **beta** (on by default) in **v1.33–1.34** and became **GA in v1.35**. VPA uses it through the **`InPlaceOrRecreate`** update mode (introduced in VPA 1.4, maturing in later releases), which **falls back to eviction** when in-place isn't possible. So "without restarting" is the goal, not a guarantee.

> ✅ **Correct today** ([C2](#c2)): the reason is the VPA updater's **`--min-replicas` setting (default 2)**: it won't **evict** Pods of a workload with fewer than 2 live replicas. That's why the demo uses 2 replicas. (You can lower it, or set `minReplicas` per VPA in `updatePolicy`.) The HPA has nothing to do with it, and running HPA and VPA on the same CPU/memory metrics is **not** recommended (below).

### In-place Pod resize (GA v1.35)

- Container `resources` in a running Pod can be changed via the Pod's **`resize` subresource**:
  ```bash
  kubectl patch pod web --subresource resize --patch \
    '{"spec":{"containers":[{"name":"web","resources":{"requests":{"cpu":"300m"},"limits":{"cpu":"600m"}}}]}}'
  ```
- The kubelet checks the node has room, then updates the container's **cgroups** (`cpu.max`, `cpu.weight`, `memory.max`; topic 41) through the runtime: **no new Pod, same IP, same name**.
- **`resizePolicy`** per resource: `NotRequired` (default: apply live) or `RestartContainer` (restart just that container, for apps that only read limits at start-up, e.g. a JVM heap):
  ```yaml
  resizePolicy:
  - { resourceName: cpu, restartPolicy: NotRequired }
  - { resourceName: memory, restartPolicy: RestartContainer }
  ```
- Status: Pod conditions **`PodResizePending`** (node can't fit it now, `Deferred`; or never, `Infeasible`) and **`PodResizeInProgress`**; `status.containerStatuses[].resources` shows what's actually applied.
- Limits: can't change the QoS class; memory **decreases** are applied only if usage is below the new limit; not for Windows Pods or some special resources.
- Deployments etc. don't use it automatically. Changing a Deployment's template still rolls out new Pods; it's VPA (or you, per Pod) that uses the subresource. Pod-level resources resizing is beta (v1.36).

### HPA vs VPA

- **Use HPA** for apps that **scale horizontally** (stateless).
- **Use VPA** to **find the right requests** for a container, and for apps that **can't** scale horizontally (stateful, legacy, single-instance).
- **Together**: the HPA adjusts the replica count from load; the VPA tunes per-Pod CPU/memory. Example: **batch processing**: VPA sizes Pods for varying job sizes, while HPA scales replicas by **queue length**.
- **The rule:** don't let both act on the **same** resource metric. If HPA scales on CPU **% of request** and VPA keeps changing the CPU **request**, they chase each other. Safe combinations: HPA on custom/external metrics (queue, RPS) + VPA on CPU/memory; or VPA in `Off` mode just for recommendations.

**Distinctions:**
- **HPA requires requests** (utilisation is a % of the request; topic 48).
- **VPA doesn't require initial requests**: it observes usage and sets them. But **set initial requests anyway**, so the scheduler places the Pod on a node that can hold it from day one.

**VPA as a recommendation tool:** even if you won't let VPA restart Pods in production, run it in **`Off`** mode to collect real usage and get **accurate request suggestions**, which is especially useful when developers can't say what their apps need.

### Cluster Autoscaler (CA)

- Watches for Pods that are **Pending because of insufficient resources** (`FailedScheduling` with `Insufficient cpu/memory`, or unsatisfiable affinity a new node would satisfy).
- **Simulates** which **node group** (an AWS Auto Scaling Group, GCP MIG, Azure VMSS…) would fit them if it had one more node, picks one by its **expander** strategy (`least-waste`, `priority`, `random`…), and **increases that group's desired size**.
- **Scale-down**: every ~10 s it looks for nodes whose requested utilisation is below a threshold (default 50%) for a while (default 10 min), checks their Pods can move elsewhere (respecting PDBs, and not moving Pods with local storage or the annotation `cluster-autoscaler.kubernetes.io/safe-to-evict: "false"`), **drains** them and shrinks the group.
- Limitations: one instance type per node group; scale-up takes minutes (VM boot + kubelet join); many node groups to manage.

### Karpenter (Day 20's "advanced option")

- An open-source node autoscaler (started by AWS; now a Kubernetes SIG project with AWS and Azure providers).
- Watches Pending Pods directly and **launches the cheapest instance type(s) that fit them**, from a wide allowed set (across sizes, families, spot/on-demand, architectures), **without node groups**.
- Configured with **`NodePool`** (constraints: instance families, zones, capacity type, taints, limits, disruption budget) and **`EC2NodeClass`** (AMI, subnets, security groups, IAM role).
- **Consolidation**: continually replaces underused or expensive nodes with fewer/cheaper ones, and expires old nodes (drift and age) for patching.
- Typically faster and more cost-efficient than CA on AWS; EKS Auto Mode is built on it.

### KEDA

- Adds a **`ScaledObject`** (for Deployments/StatefulSets) or **`ScaledJob`**, with **triggers** for 60+ event sources: Kafka lag, SQS/RabbitMQ queue length, Prometheus queries, cron schedules, HTTP request rate…
- Runs a metrics adapter (serving the External Metrics API) and **creates/manages an HPA** for you; adds **scale to zero** and back (activation threshold).

```yaml
apiVersion: keda.sh/v1alpha1
kind: ScaledObject
metadata: { name: orders-worker }
spec:
  scaleTargetRef: { name: orders-worker }
  minReplicaCount: 0
  maxReplicaCount: 50
  triggers:
  - type: aws-sqs-queue
    metadata: { queueURL: https://sqs.us-east-1.amazonaws.com/123456789012/orders, queueLength: "20", awsRegion: us-east-1 }
    authenticationRef: { name: keda-aws-auth }
```

### Worker nodes are never vertically scaled

Resizing a node = changing the VM instance type, which means a **node restart** and **evicting every Pod**: disruptive and impractical. Production clusters scale nodes **horizontally** only (CA/Karpenter), using node groups per workload type (high-memory, high-CPU, GPU).

---

## Commands / YAML

### VPA demo

**Step 1: install VPA** ([installation guide](https://github.com/kubernetes/autoscaler/blob/master/vertical-pod-autoscaler/docs/installation.md#install-command)):

```bash
git clone https://github.com/kubernetes/autoscaler.git
cd autoscaler/vertical-pod-autoscaler
./hack/vpa-up.sh            # installs CRDs, recommender, updater, admission controller (needs openssl for its webhook certs)
kubectl get pods -n kube-system | grep vpa
# later: ./hack/vpa-down.sh to uninstall
```

(A Helm chart is the more common way in real clusters.) Metrics-server must be running (topic 48).

**Step 2: Deployment + Service** (`Day 20/nginx-vpa.yaml`):

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deploy
spec:
  replicas: 2                  # ≥ 2, so the updater is allowed to evict (min-replicas)
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.29
        resources:
          requests:
            cpu: "100m"
          limits:
            cpu: "100m"
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-svc
spec:
  selector:
    app: nginx
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
  type: ClusterIP
```

```bash
kubectl apply -f "Day 20/nginx-vpa.yaml"
```

**Step 3: the VPA object** (`Day 20/vpa.yaml`):

```yaml
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: nginx-vpa
spec:
  targetRef:
    apiVersion: "apps/v1"
    kind: Deployment
    name: nginx-deploy
  updatePolicy:
    # "Off"               - recommend only
    # "Initial"           - set requests at Pod creation only
    # "Recreate"          - evict and recreate Pods to apply (Day 20 used "Auto", now deprecated)
    # "InPlaceOrRecreate" - resize in place, fall back to eviction
    updateMode: "Recreate"
  resourcePolicy:              # optional guard-rails
    containerPolicies:
    - containerName: "*"
      minAllowed: { cpu: 25m, memory: 32Mi }
      maxAllowed: { cpu: "1", memory: 1Gi }
      controlledValues: RequestsAndLimits     # or RequestsOnly
```

```bash
kubectl apply -f "Day 20/vpa.yaml"
```

(Day 20's text applies `nginx-vpa.yaml` again at this step; the VPA object is in **`vpa.yaml`**.)

**Step 4: read the recommendation**

```bash
kubectl describe vpa nginx-vpa
```

```
Recommendation:
  Container Recommendations:
    Container Name:  nginx
    Lower Bound:
      Cpu:     25m
      Memory:  262144k
    Target:
      Cpu:     25m
      Memory:  262144k
    Uncapped Target:
      Cpu:     25m
      Memory:  262144k
    Upper Bound:
      Cpu:     25m
      Memory:  262144k
```

| Field | Meaning |
|---|---|
| **Lower Bound** | The minimum safe requests; don't go below |
| **Target** | The recommended requests from observed usage (what VPA applies) |
| **Uncapped Target** | The raw recommendation **before** applying `minAllowed`/`maxAllowed` constraints |
| **Upper Bound** | The maximum sensible requests; above this is waste |

Pods are moved **towards Target**.

**Uncapped Target vs Target:** Uncapped Target is the pure usage-based recommendation, ignoring any policy; **Target** is that value **clamped by the VPA's `resourcePolicy`** (`minAllowed`/`maxAllowed`). With no such policy they're identical, as in this demo.

> ℹ️ **Note:** the original says Target respects "**quotas** or administrative policies". The VPA recommender only knows its own `resourcePolicy` bounds; it doesn't read ResourceQuotas or LimitRanges. (The admission controller does keep within a LimitRange's limits when applying values.) A quota can still **reject** a recreated Pod whose new requests don't fit.

**Step 5: watch VPA act**

```bash
kubectl describe vpa nginx-vpa          # updated recommendations and conditions
kubectl get events --sort-by=.lastTimestamp | grep -i -E 'evict|vpa'
kubectl get pods -o custom-columns='NAME:.metadata.name,CPU_REQ:.spec.containers[0].resources.requests.cpu'
```

```
Normal  EvictedByVPA  pod/nginx-deploy-6d4b7c9f8-k2xjp  Pod was evicted by VPA Updater to apply resource recommendation.
NAME                            CPU_REQ
nginx-deploy-6d4b7c9f8-q9zt2    25m
nginx-deploy-6d4b7c9f8-x7m2c    25m
```

The new Pods came up with **25m** CPU (and a memory request) instead of the original 100m: the admission controller rewrote them.

### In-place resize, by hand

```bash
kubectl run resize-demo --image=nginx:1.29 --overrides='{"spec":{"containers":[{"name":"resize-demo","image":"nginx:1.29","resources":{"requests":{"cpu":"100m","memory":"64Mi"},"limits":{"cpu":"200m","memory":"128Mi"}}}]}}'
kubectl patch pod resize-demo --subresource resize --patch \
  '{"spec":{"containers":[{"name":"resize-demo","resources":{"requests":{"cpu":"300m"},"limits":{"cpu":"500m"}}}]}}'
kubectl get pod resize-demo -o jsonpath='{.status.containerStatuses[0].resources}{"\n"}{.status.containerStatuses[0].restartCount}{"\n"}'
```

```
{"limits":{"cpu":"500m","memory":"128Mi"},"requests":{"cpu":"300m","memory":"64Mi"}}
0
```

- New values are active and `restartCount` is still **0**: resized in place.

---

## Hands-on lab

1. Install VPA and run the demo in `Off` mode first: read the recommendation without anything changing. Then switch to `Recreate` and watch the evictions.
2. Repeat with `replicas: 1`: nothing is evicted (min-replicas). Set `updatePolicy.minReplicas: 1` and see it act.
3. Try `InPlaceOrRecreate` (needs a recent VPA) and compare Pod names/restart counts before and after.
4. Do the manual in-place resize; then request more CPU than the node can give and read the `PodResizePending` condition (`Infeasible`).
5. (Cloud, optional) On EKS: install Karpenter or Cluster Autoscaler, scale a Deployment with big requests beyond capacity, and watch nodes appear; scale back down and watch consolidation/scale-down.
6. (Optional) Install KEDA with Helm and scale a Deployment with a `cron` trigger (e.g. 3 replicas during 9–5).

---

## Where you'll use it

- **Right-sizing**: VPA `Off` recommendations (or tools like Goldilocks that wrap it) to fix requests across hundreds of services.
- **Stateful/singleton apps**: VPA (preferably in place) instead of HPA.
- **Cost**: Karpenter consolidation, spot capacity, scale-to-zero with KEDA for queue workers and dev environments.
- **Capacity**: Cluster Autoscaler/Karpenter so HPA scale-ups actually get nodes.

---

## When it breaks

| Symptom | Cause | Fix |
|---|---|---|
| VPA shows no recommendation | metrics-server missing, or not enough history yet | Check metrics; wait (minutes to hours) |
| VPA never updates Pods | Mode `Off`/`Initial`, single replica (min-replicas), or PDB blocks | Check mode and replicas; PDB |
| Pods restart repeatedly after VPA | Recommendations oscillate; HPA and VPA on the same metric | Bounds in `resourcePolicy`; separate metrics |
| Resized Pod shows `PodResizePending: Infeasible` | The node can't ever fit the new size | Smaller resize, or let VPA recreate it elsewhere |
| Pending Pods but no new nodes | CA/Karpenter not seeing them (taints, affinity no node group satisfies, max size reached, IAM) | Autoscaler logs; NodePool/node group config |
| Nodes never scale down | PDBs with 0 allowed disruptions, Pods with local storage or safe-to-evict false, DaemonSet-only nodes misjudged | Fix PDBs and annotations; tune thresholds |

---

## Quick recap

- **VPA** = recommender + updater + admission webhook; modes `Off`, `Initial`, `Recreate`, `InPlaceOrRecreate` (`Auto` deprecated); fields Lower/Target/Uncapped/Upper; needs ≥ 2 replicas to evict by default.
- **In-place resize** (GA v1.35): `kubectl patch --subresource resize`; `resizePolicy`; no new Pod.
- HPA + VPA: fine on **different** metrics; never both on CPU/memory.
- Nodes scale **horizontally**: **Cluster Autoscaler** (node groups) or **Karpenter** (right-sized instances, consolidation). **KEDA** for event-driven scaling and scale to zero.

### Interview questions

1. **Q: When would you use VPA instead of HPA?**
   A: For workloads that can't scale out (stateful, singleton, legacy), and to find correct requests for any workload (recommendation mode).
2. **Q: Can you change a Pod's CPU limit without restarting it?**
   A: Yes, since v1.35 in-place resize is GA: patch the Pod's `resize` subresource; the kubelet updates the cgroup. A `resizePolicy` of `RestartContainer` can force a container restart for apps that need it.
3. **Q: Cluster Autoscaler vs Karpenter?**
   A: CA scales pre-defined node groups up and down based on Pending Pods and underutilisation; Karpenter provisions individual instances of the best-fitting types directly from Pending Pods' requirements, and continuously consolidates for cost, without managing node groups.

---

## Appendix: notes on the original course material

This topic was rebuilt from the original CKA course notes. This appendix keeps the audit trail out of your way while you learn.

### Sources

Day 20 (VPA: what it is, key behaviour, the in-place note, why use VPA, operating modes, the single-replica note, components, VPA as a recommendation tool, HPA vs VPA recommendations and complementary use, distinctions; VPA demo steps 1–5; references; plus the Cluster Autoscaler/Karpenter mentions under "scaling worker nodes"), files `Day 20/nginx-vpa.yaml`, `Day 20/vpa.yaml`.

### Corrections to the original notes

Each entry shows what the original said and what's true now. In the topic above, each correction appears as a ✅ **Correct today** line or a ⚠️ **Fix to the original** box, linked to its entry here.

<a id="c1"></a>**C1 · Corrected (Day 20).** the note "as of Kubernetes v1.33, VPA supports in-place resizing without restarting pods". **Now:** in-place Pod resize was **beta** (on by default) in **v1.33–1.34** and became **GA in v1.35**. VPA uses it through the **`InPlaceOrRecreate`** update mode (introduced in VPA 1.4, maturing in later releases), which **falls back to eviction** when in-place isn't possible. So "without restarting" is the goal, not a guarantee.

<a id="c2"></a>**C2 · Corrected (Day 20).** "If you have only **1 replica**, VPA will avoid restarting the pod even in Auto mode to prevent downtime… safe to leave at 1 replica if you're using HPA alongside VPA, as HPA handles scaling". **Now:** the reason is the VPA updater's **`--min-replicas` setting (default 2)**: it won't **evict** Pods of a workload with fewer than 2 live replicas. That's why the demo uses 2 replicas. (You can lower it, or set `minReplicas` per VPA in `updatePolicy`.) The HPA has nothing to do with it, and running HPA and VPA on the same CPU/memory metrics is **not** recommended (below).
