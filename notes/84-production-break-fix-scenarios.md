# 84 · Production Break-Fix Scenarios

> **Level:** ★★★ advanced
>
> **You will learn:** to work through realistic incidents end to end, using the method from topic 81 and the playbooks from topics 82–83: each scenario has a **symptom**, a **timeline of evidence**, the **root cause**, the **fix**, and **how to prevent it**. They cover rollouts, resources, networking, DNS, storage, security, upgrades, the control plane and capacity: the kinds of failures that actually page people.
>
> **You need:** most of the course; especially topics 21, 25, 41, 47, 58, 61, 77, 80–83.

---

## ⚡ Core: the 10-minute version

**The idea.** These are realistic incident stories, each with symptom → evidence → root cause → fix → prevention:
- 502s during deploys (no readiness probe);
- nightly OOM kills (limits sized for daytime);
- DNS dying after default-deny NetworkPolicies;
- Pending StatefulSets after a node-pool change (zonal volumes);
- CI suddenly Forbidden (GitOps pruned RoleBindings);
- etcd full;
- expired certificates;
- 5-second DNS delays;
- ephemeral-storage evictions;
- a failing webhook blocking every Pod.

Most incidents follow a **change**.

**Picture it.** A flight simulator: practise the emergencies here so the real ones feel familiar.

**The one thing to know.** First question in every incident: **"what changed?"**

```bash
kubectl rollout history deploy/<name> -n <ns>
kubectl get events -A --sort-by=.lastTimestamp | tail -30
git log --oneline -10            # in your GitOps / manifests repo
```

**Try it now:** reproduce scenario 3 (default-deny without DNS) and scenario 10 (a fail-closed webhook with its Pods scaled to 0) on kind. Write a 5-line postmortem for each.

**You've got it if you can:** tell any one scenario end to end, including prevention.

> 📖 **That's the core.** On a first pass, do the mini-lab and move on to the next topic. Everything below is the **full reference**: come back for the details, the edge cases, interview answers and exam-level depth.

---

## What it is

A set of **incident walkthroughs**. Read the symptom, try to solve it yourself (ideally reproduce it in a lab), then compare with the investigation. Each ends with **prevention**, because the goal in production is to make the same incident impossible or harmless next time.

---

## Why it exists

Knowing commands isn't the same as debugging under pressure. Real incidents combine several layers, are often caused by a **change**, and give misleading first signals. Practising full scenarios builds the pattern recognition interviewers (and pagers) test.

---

## Analogy

Flight simulators: pilots practise engine failures and storms in a simulator so the real thing feels familiar. These scenarios are your simulator.

---

## How it works inside

Every scenario follows: **Symptom → Evidence (commands + output) → Root cause → Fix (mitigate, then repair) → Prevention**.

---

## Commands / YAML

### Scenario 1: The rollout that ate the site

**Symptom:** after a routine deploy, 30% of checkout requests return 502 for 4 minutes, then recover.

**Evidence:**

```bash
kubectl rollout history deploy/checkout -n shop
kubectl describe deploy checkout -n shop | grep -A3 -E 'Strategy|Readiness'
kubectl get events -n shop --sort-by=.lastTimestamp | grep checkout | tail
```

```
StrategyType:           RollingUpdate
RollingUpdateStrategy:  50% max unavailable, 25% max surge
    Readiness:  <none>
Normal  Killing  …  Stopping container checkout
```

**Root cause:** no **readiness probe** (new Pods received traffic while still warming up, before connecting to their database), `maxUnavailable: 50%` halved capacity, and no `preStop` delay, so old Pods were killed while the load balancer still routed to them.

**Fix:** add a readiness probe; `maxUnavailable: 0`, `maxSurge: 25%`; a short `preStop: sleep 10` + adequate `terminationGracePeriodSeconds` (topics 19, 22, 25).

```yaml
strategy: { type: RollingUpdate, rollingUpdate: { maxUnavailable: 0, maxSurge: 25% } }
template:
  spec:
    terminationGracePeriodSeconds: 40
    containers:
    - name: checkout
      readinessProbe: { httpGet: { path: /ready, port: 8080 }, periodSeconds: 5, failureThreshold: 2 }
      lifecycle: { preStop: { sleep: { seconds: 10 } } }
```

**Prevention:** admission policy requiring readiness probes (topic 58); progressive delivery (canary with automated rollback on error rate; topics 72, 74); SLO burn alerts (topic 80).

### Scenario 2: OOMKilled only at night

**Symptom:** the `reports` Pods restart every night around 02:00; `CrashLoopBackOff` briefly.

**Evidence:**

```bash
kubectl get pod -n analytics -l app=reports -o custom-columns=NAME:.metadata.name,RESTARTS:.status.containerStatuses[0].restartCount,LAST:.status.containerStatuses[0].lastState.terminated.reason
```

```
NAME                       RESTARTS   LAST
reports-6c9d7f5b8-x2k9p    14         OOMKilled
```

Grafana: memory working set climbs to exactly the **512Mi limit** during the nightly batch; CPU is throttled at the same time.

**Root cause:** limits sized for daytime traffic; the nightly batch loads a full dataset into memory.

**Fix:** raise memory **request = limit** to a measured value (e.g. 1.5Gi) for this workload; move the batch to a separate **CronJob** with its own resources (topic 28); or use VPA recommendations / in-place resize (topic 49).

**Prevention:** alert on `container_memory_working_set_bytes / limit > 0.9`; load-test the batch; right-size from VPA in recommendation mode.

### Scenario 3: DNS dies after a security hardening change

**Symptom:** right after the security team applied "default-deny" NetworkPolicies, every service in `payments` fails with `getaddrinfo ENOTFOUND`.

**Evidence:**

```bash
kubectl get netpol -n payments
kubectl run t -n payments --rm -it --image=busybox:1.37 --restart=Never -- nslookup kubernetes.default
```

```
NAME               POD-SELECTOR   AGE
default-deny-all   <none>         12m
;; connection timed out; no servers could be reached
```

**Root cause:** `policyTypes: [Ingress, Egress]` with no egress rules blocks **DNS (UDP/TCP 53 to CoreDNS)** (topic 61).

**Fix:** add an `allow-dns` egress policy (AND-form selector for `kube-dns` Pods in `kube-system`), then the app-specific allows.

**Prevention:** ship default-deny **with** allow-dns as one bundle (Kyverno generate policy); test policies in staging; NetworkPolicy changes through review.

### Scenario 4: Pending Pods after a node-pool upgrade

**Symptom:** after replacing the node pool, a StatefulSet's Pods are `Pending`: `0/6 nodes are available: 3 node(s) had volume node affinity conflict, 3 node(s) didn't match Pod's node affinity/selector`.

**Evidence:**

```bash
kubectl describe pod db-1 -n data | sed -n '/Events:/,$p'
kubectl get pv $(kubectl get pvc data-db-1 -n data -o jsonpath='{.spec.volumeName}') -o jsonpath='{.spec.nodeAffinity}'; echo
kubectl get nodes -L topology.kubernetes.io/zone,pool
```

```
{"required":{"nodeSelectorTerms":[{"matchExpressions":[{"key":"topology.kubernetes.io/zone","operator":"In","values":["eu-west-1c"]}]}]}}
NAME   ZONE         POOL
n1     eu-west-1a   general-v2
n2     eu-west-1b   general-v2
…      (no nodes in eu-west-1c with pool=general-v2)
```

**Root cause:** the new pool has no nodes in **eu-west-1c**, where `db-1`'s EBS volume lives (zonal block storage), and the StatefulSet's `nodeSelector` targets the new pool label.

**Fix:** add nodes in eu-west-1c to the pool (or a dedicated pool per zone); Pods schedule and attach.

**Prevention:** node pools spanning all zones used by volumes; `WaitForFirstConsumer` StorageClasses (topic 39); upgrade runbooks checking volume zones; PDBs so drains don't take multiple replicas down (topic 76).

### Scenario 5: "Forbidden" for the CI pipeline after an upgrade

**Symptom:** deploys from CI fail with `deployments.apps is forbidden: User "system:serviceaccount:ci:deployer" cannot patch resource "deployments" in API group "apps" in the namespace "shop"`, starting the day after a platform change.

**Evidence:**

```bash
kubectl auth can-i patch deployments -n shop --as=system:serviceaccount:ci:deployer
kubectl get rolebinding -n shop -o wide | grep deployer
git -C platform-gitops log --oneline -5 -- rbac/
```

```
no
(no output)
3f2a1c9 refactor: move team RBAC to ApplicationSet
```

**Root cause:** a GitOps refactor moved RoleBindings into a new folder that the ApplicationSet doesn't include; with **prune** enabled, Argo CD deleted the old bindings (topic 68).

**Fix:** mitigate by re-applying the RoleBinding (or reverting the commit); fix the ApplicationSet path.

**Prevention:** CI validation of rendered manifests (diff showing deletions), `Prune=false` or confirmation for RBAC objects, alerts on `403` spikes from service accounts.

### Scenario 6: The cluster that stopped accepting writes

**Symptom:** `kubectl apply` everywhere fails: `etcdserver: mvcc: database space exceeded`. Reads still work.

**Evidence:**

```bash
sudo etcdctl $ETCD endpoint status -w table          # topic 77 flags
sudo etcdctl $ETCD alarm list
kubectl get --raw /metrics | grep apiserver_storage_objects | sort -k2 -n | tail -3
```

```
| DB SIZE 2.1 GB |
memberID:8e9e05c52164694d alarm:NOSPACE
apiserver_storage_objects{resource="events"} 1.2e+06
apiserver_storage_objects{resource="backuprequests.ops.example.com"} 840000
```

**Root cause:** a buggy controller created hundreds of thousands of custom resources (and Events), filling etcd's 2 GiB quota.

**Fix:** stop the controller; delete the junk objects (`kubectl delete backuprequests --all -A`, in batches); compact + **defrag**; **disarm** the alarm; consider raising the quota to 8 GiB.

**Prevention:** monitor etcd DB size and object counts; ResourceQuotas on object counts (`count/backuprequests.ops.example.com`) per namespace; controllers that garbage-collect their objects (TTLs, ownerReferences).

### Scenario 7: Certificates expired on a forgotten cluster

**Symptom:** Monday morning, an on-prem kubeadm cluster: `kubectl` → `x509: certificate has expired or is not yet valid`; workloads still serve traffic.

**Evidence:** `sudo kubeadm certs check-expiration` → all `<invalid>`; the cluster hadn't been upgraded for 13 months.

**Root cause:** kubeadm-issued certificates last 1 year; upgrades renew them, and there were none.

**Fix:** topic 82's procedure: check the clock, `kubeadm certs renew all`, restart control-plane static Pods, update kubeconfigs; check kubelet client certs on nodes.

**Prevention:** monitor certificate expiry (`apiserver_client_certificate_expiration_seconds`, a cron running `kubeadm certs check-expiration`); upgrade at least yearly (topic 76).

### Scenario 8: Intermittent 5-second latency

**Symptom:** about 1% of requests between services take **exactly ~5 s** extra.

**Evidence:** traces show the extra time in DNS resolution before the HTTP call; CoreDNS metrics are normal; the client Pods' `/etc/resolv.conf` has `options ndots:5`.

**Root cause:** with `ndots:5`, a name like `api.payments.example.com` is first tried against every search domain (`…svc.cluster.local` etc.): several queries per lookup, A and AAAA in parallel over UDP. Occasional lost UDP packets (a conntrack race on the node) cause the resolver's **5-second retry timeout**.

**Fix:** use fully qualified names with a trailing dot for external hosts, or set `dnsConfig: {options: [{name: ndots, value: "2"}]}` for those Pods; run **NodeLocal DNSCache** (a per-node DNS cache that avoids conntrack for DNS); `single-request-reopen`-style resolver options where the libc supports them (topic 33).

**Prevention:** dashboards on DNS latency per client; NodeLocal DNSCache as a platform default in large clusters.

### Scenario 9: Node pressure eviction cascade

**Symptom:** several Pods `Evicted` on two nodes; Deployments recreate them elsewhere, then those nodes start evicting too.

**Evidence:**

```bash
kubectl describe node n4 | grep -E 'DiskPressure|MemoryPressure'
kubectl get pods -A --field-selector=status.phase=Failed | grep Evicted | head
kubectl describe pod <evicted> | grep -A2 Message
```

```
DiskPressure   True   KubeletHasDiskPressure   kubelet has disk pressure
Message:  The node was low on resource: ephemeral-storage. Threshold quantity: 10%, available: 6%. Container app was using 18Gi, request is 0, has larger consumption of ephemeral-storage.
```

**Root cause:** an app writes large temp files to its writable layer/emptyDir with **no ephemeral-storage requests/limits**; wherever it lands, the node fills up.

**Fix:** set `ephemeral-storage` requests/limits on that container (or an `emptyDir.sizeLimit`); clean up `Failed` Pods; fix the app's temp-file handling.

**Prevention:** LimitRange defaults for ephemeral storage (topic 41); alerts on node filesystem usage; log rotation configured.

### Scenario 10: A webhook takes down deployments cluster-wide

**Symptom:** no one can create Pods in any namespace: `Internal error occurred: failed calling webhook "validate.kyverno.svc-fail": … context deadline exceeded`.

**Evidence:** `kubectl get validatingwebhookconfigurations`; Kyverno Pods `CrashLoopBackOff` (OOMKilled after a policy-report backlog); `failurePolicy: Fail`.

**Root cause:** Kyverno's Pods were **OOMKilled** (memory limits too low for a growing policy-report backlog), so its admission webhook stopped answering. Because the webhook was configured with **`failurePolicy: Fail`**, the API server rejected every Pod creation it couldn't validate. (Kyverno excludes its own namespace, so its replacement Pods could still start, but they crashed again for the same memory reason.)

**Fix:** raise Kyverno's memory limits / scale it; in an emergency, temporarily delete or narrow the webhook configuration, then restore it.

**Prevention:** HA replicas and PDBs for webhooks, sensible limits, exclude `kube-system` and the webhook's namespace, short `timeoutSeconds`, alerts on webhook latency/errors (`apiserver_admission_webhook_rejection_count`, `…_request_duration_seconds`) (topic 58).

---

## Hands-on lab

Reproduce at least five scenarios on kind:
- 1: a Deployment without readiness probe behind a Service, plus `hey`/`curl` loops during a rollout; then the fixed version.
- 2: a Pod allocating memory (`polinux/stress --vm-bytes 600M`) with a 512Mi limit.
- 3: default-deny without DNS.
- 6 (in miniature): create thousands of ConfigMaps in a loop and watch `apiserver_storage_objects`.
- 9: a container writing to `/tmp` with `ephemeral-storage` limits set, and watch the eviction.
- 10: install Kyverno with a `failurePolicy: Fail` policy, scale Kyverno to 0, try to create a Pod.

For each, write a short postmortem: impact, timeline, root cause, contributing factors, action items.

---

## Where you'll use it

- **On-call** and **incident reviews**.
- **Senior interviews** (topic 87): "tell me about an incident you handled".
- **Designing guardrails**: every prevention item here is a platform feature.

---

## When it breaks

Common traps in incident handling:

| Trap | Better |
|---|---|
| Restarting things before capturing evidence | Save `describe`, `logs --previous`, `-o yaml` first |
| Fixing the symptom only (e.g. raising limits forever) | Find why usage grew |
| Several simultaneous changes during an incident | One change at a time, logged in the timeline |
| No rollback plan for the fix itself | Every change reversible |
| Blame | Blameless postmortems focused on systems |

---

## Quick recap

- Most incidents follow a **change**: check rollout history, Git, upgrades first.
- Recurring causes: missing readiness probes/rollout safety, wrong resource limits, NetworkPolicies without DNS, zonal storage vs node placement, RBAC drift via GitOps prune, etcd filling up, expired certificates, DNS `ndots` and conntrack, ephemeral-storage pressure, fail-closed webhooks.
- Always end with **prevention**: probes, policies, alerts, quotas, runbooks, upgrades.

### Interview questions

1. **Q: Tell me how you'd investigate intermittent 502s during deploys.**
   A: Correlate with rollout times, check the readiness probe and rollout strategy (maxUnavailable/maxSurge), termination handling (preStop, grace period, load-balancer deregistration delay), and the ingress/LB logs; fix with proper readiness, zero-unavailable surges and graceful shutdown.
2. **Q: All writes to the cluster fail with "database space exceeded". What do you do?**
   A: It's etcd's NOSPACE alarm: find and stop the source of object growth, delete excess objects, compact and defragment etcd, disarm the alarm, then add monitoring and object-count quotas.
3. **Q: After enabling default-deny NetworkPolicies, nothing can resolve names. Why?**
   A: The egress default-deny blocks DNS traffic to CoreDNS; add an egress rule allowing UDP/TCP 53 to the `kube-dns` Pods in `kube-system`.

---

## Appendix: notes on the original course material

This topic was rebuilt from the original CKA course notes. This appendix keeps the audit trail out of your way while you learn.

### Sources

none: a **new topic**. The scenarios combine failure modes from across the original notes (Days 22, 23, 41, 46–48, 54–58) into production-style stories.
