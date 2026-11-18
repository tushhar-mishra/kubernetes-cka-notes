# 47 · Priority, Preemption and Eviction

> **Level:** ★★ intermediate
>
> **You will learn:** **PriorityClasses** (values, the reserved system classes, `globalDefault`, `preemptionPolicy`); how priority orders the scheduling queue; **preemption**: how the scheduler picks and evicts lower-priority victims (and what it respects); **node-pressure eviction**: how the kubelet evicts Pods when a node runs low on memory, disk or PIDs (thresholds, ranking by usage, priority and QoS); **API-initiated eviction** (drains, PodDisruptionBudgets); and how all of these differ from OOM kills.
>
> **You need:** topics 41 (QoS) and 42 (scheduler).

---

## ⚡ Core: the 10-minute version

**The idea.**
- **Priority**: a **PriorityClass** gives important Pods a higher number. They're scheduled first and, if the cluster is full, the scheduler can **preempt** (evict) lower-priority Pods to make room.
- **Eviction**: separately, when a **node** runs short of memory or disk, the **kubelet** evicts Pods. The biggest users relative to their requests, and lower-priority Pods, go first. **QoS** decides the order in which containers are OOM-killed.

**Picture it.** Priority is the hospital triage queue; preemption is moving a less urgent patient out of a bed; eviction is the building manager clearing an overcrowded floor.

**The one thing to know.**

```yaml
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata: { name: critical }
value: 100000
preemptionPolicy: PreemptLowerPriority
description: "Customer-facing services"
---
# in a Pod spec:  priorityClassName: critical
```

**Try it now:**
1. Fill your cluster with low-priority Pods that request most of the CPU.
2. Create a Pod with `priorityClassName: critical`.
3. Watch the low-priority Pods get preempted (`kubectl get events | grep -i preempt`).

**You've got it if you can:** distinguish preemption (scheduler) from node-pressure eviction (kubelet).

> 📖 **That's the core.** On a first pass, do the mini-lab and move on to the next topic. Everything below is the **full reference**: come back for the details, the edge cases, interview answers and exam-level depth.

---

## What it is

Not all workloads are equal: payment processing is critical; a dev dashboard or a batch job isn't. Kubernetes has three mechanisms that decide **who wins when resources run out**:

| Mechanism | Who does it | When | Based on |
|---|---|---|---|
| **Priority** (scheduling order) | Scheduler | Always: higher-priority Pending Pods are scheduled first | `PriorityClass` value |
| **Preemption** | Scheduler | A **new** Pod can't fit anywhere | Priority |
| **Node-pressure eviction** | Kubelet | The **node** is low on memory/disk/PIDs | Usage above requests, then priority, then usage |
| (**OOM kill**) | Linux kernel | A cgroup or the node is out of memory **right now** | `oom_score_adj` (from QoS) |

---

## Why it exists

During a **resource crunch**, the cluster should **favour critical workloads**. **Pod priority** tells the scheduler **which Pods to prefer**. And if the cluster is already full when a critical Pod arrives, **preemption** evicts lower-priority Pods to make room, so critical apps get resources at the cost of non-essential ones.

Separately, a node must protect **itself**: if memory or disk runs out, the kubelet, container runtime and OS could fail, taking **every** Pod on the node down. Node-pressure eviction sacrifices some Pods early to keep the node alive.

---

## Analogy

An **airline flight**:
- **Priority** = boarding groups: business class boards first.
- **Preemption** = an **overbooked** flight: when a priority passenger (high-priority Pod) can't get a seat, the airline **bumps** economy passengers (lower-priority Pods), politely (graceful termination), to free seats.
- **Node-pressure eviction** = the plane is **too heavy to take off**: the crew removes the passengers carrying the most excess baggage over their allowance (usage over requests) first, then by ticket class (priority).
- **`preemptionPolicy: Never`** = a business-class passenger who gets **priority boarding** but **refuses to bump** anyone: they'll wait for the next flight instead.

---

## How it works inside

### PriorityClass

Instead of putting raw numbers in every Pod, you define reusable classes:

```yaml
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: high-priority
value: 1000                  # higher = more important
globalDefault: false
description: "Critical production workloads"
```

```yaml
spec:
  priorityClassName: high-priority
```

- **Higher value = higher priority.** A Pod with 1000 is scheduled before one with 100.
- **User range:** **−2,147,483,648 to 1,000,000,000** (one billion).
- **Reserved system classes** (above 1 billion), used by control-plane and node-critical add-ons:

| Class | Value | Used by |
|---|---|---|
| `system-node-critical` | 2,000,001,000 | Things a node can't live without: etcd, kube-apiserver static Pods, kube-proxy, CNI agents |
| `system-cluster-critical` | 2,000,000,000 | Cluster-critical add-ons: CoreDNS, metrics-server |

  These always outrank user classes and can't be redefined.
- **Pods without a class** get priority **0**, unless a class has **`globalDefault: true`**. Only **one** class may be the global default (a second one is rejected).
- PriorityClasses are **cluster-scoped** (usable from any namespace); `kubectl get pc` lists them.
- The admission plugin **Priority** resolves `priorityClassName` into `spec.priority` when the Pod is created; changing a class later doesn't change existing Pods.
- You can restrict who may use a class (e.g. only `kube-system` may use high classes) with a **ResourceQuota scoped to a PriorityClass** (topic 41).

### Priority in the scheduling queue

The scheduler's queue is sorted by **priority** (then age). When the cluster has room for some but not all Pending Pods, **higher-priority ones are placed first**. This is true **even with `preemptionPolicy: Never`** (Day 46: "priority affects scheduling; preemption controls eviction").

### Preemption

When a high-priority Pod fails the **filter** phase on every node (topic 42), the scheduler's **PostFilter** step tries preemption:
1. For each node, it pretends to remove **lower-priority** Pods (lowest first) until the new Pod would fit, checking that the **other** constraints (affinity, etc.) would then pass.
2. It prefers the node where the victims cause the **least damage**: violating the fewest **PodDisruptionBudgets** (topic 76), lowest victim priorities, fewest victims, latest start times.
3. It sets the preemptor's **`status.nominatedNodeName`** (visible as `NOMINATED NODE` in `kubectl get pods -o wide`) and **deletes the victims gracefully** (they get their normal `terminationGracePeriodSeconds`).
4. When the victims are gone, the preemptor is scheduled there (unless something higher-priority grabbed the space meanwhile).

Notes:
- **Graceful:** victims are terminated normally; their controllers (Deployments etc.) recreate them, and they'll be scheduled again when room appears.
- **Only when there's no other way:** preemption happens only if the Pod truly can't be scheduled otherwise.
- **PDBs are respected on a best-effort basis**: if the only option violates a PDB, preemption still happens.
- Preemption only frees **requests**; it won't help a Pod blocked by taints, affinity or a missing PVC.

**`preemptionPolicy`:**

```yaml
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: high-non-disruptive
value: 900
globalDefault: false
preemptionPolicy: Never           # default is PreemptLowerPriority
description: "High priority, but should not preempt running pods"
```

- Default: **`PreemptLowerPriority`**: this class's Pods may evict lower-priority Pods.
- **`Never`**: Pods of this class **don't preempt**; if they can't fit, they **wait** (still ahead of lower-priority Pods in the queue).
- Useful when you want to **avoid killing user-facing or batch jobs**, prefer **manual intervention**, or run **important but non-disruptive** workloads.

> ✅ **Correct today** ([C1](#c1)): `preemptionPolicy` controls whether Pods of that class **cause** preemption. It gives them **no protection** from being preempted themselves. To **protect** stateful apps or agents from preemption, give them a **higher priority** (and PDBs); to stop a class from **disrupting** others, use `Never`.

### No proactive eviction by the scheduler

If high- and low-priority Pods are already running and the cluster hits a crunch (node failure, CPU spikes), the **scheduler doesn't proactively evict** low-priority Pods. Preemption only happens when a **new** Pod can't be scheduled. Runtime problems are handled on the node:
- **CPU spikes** → CFS throttling (topic 41), not evictions;
- **memory/disk exhaustion** → **node-pressure eviction** by the kubelet (next section) or **OOM kills** by the kernel.

> ✅ **Correct today** ([C2](#c2)): **node-pressure eviction** (the kubelet evicting Pods when the node is low on memory/disk/PIDs) **does** use priority as one of its ranking criteria (below). What isn't tied to priority are **CPU throttling** and **kernel OOM kills** (those use QoS-derived `oom_score_adj`).

### QoS vs priority

**QoS class** (Guaranteed / Burstable / BestEffort, from requests and limits; topic 41) is **separate** from priority. It matters during **node-level pressure**: higher QoS classes are less likely to be evicted or OOM-killed. Priority drives scheduling order and preemption, and is one input to node-pressure eviction.

### Node-pressure eviction (kubelet)

The kubelet watches **eviction signals** and compares them with **thresholds** (kubelet config `evictionHard`/`evictionSoft`):

| Signal | Default hard threshold (Linux) | Node condition |
|---|---|---|
| `memory.available` | `< 100Mi` | `MemoryPressure` |
| `nodefs.available` (kubelet's disk: `/var/lib/kubelet`, logs, emptyDirs) | `< 10%` | `DiskPressure` |
| `nodefs.inodesFree` | `< 5%` | `DiskPressure` |
| `imagefs.available` (runtime's disk: images, writable layers) | `< 15%` | `DiskPressure` |
| `pid.available` | (set via config) | `PIDPressure` |

When a threshold is crossed:
1. The node condition becomes `True` and a **`NoSchedule` taint** is added (topic 43), so no new Pods arrive.
2. For **disk**, the kubelet first tries to **reclaim**: delete dead containers and **unused images**.
3. Then it **evicts Pods one by one**, ranked by:
   1. whether the Pod's usage of the starved resource **exceeds its request** (those go first; BestEffort Pods have request 0, so they always "exceed");
   2. **Pod priority** (lower first);
   3. usage relative to requests (biggest offenders first).
4. Evicted Pods get phase **`Failed`** with reason **`Evicted`** and a message saying which resource. Their controllers create replacements, which may land on healthier nodes.

**Soft thresholds** (`evictionSoft` + `evictionSoftGracePeriod`) wait a grace period before evicting and honour the Pod's termination grace period (up to `evictionMaxPodGracePeriod`); **hard** thresholds evict immediately with no grace period.

If memory runs out **faster** than the kubelet can react, the kernel OOM killer acts first, picking the process with the highest `oom_score_adj` (BestEffort 1000 → Burstable → Guaranteed −997; topic 41).

### API-initiated eviction

`kubectl drain` (topic 76) and other tools call the **Eviction API** (`POST …/pods/<name>/eviction`), which **respects PodDisruptionBudgets**: if evicting the Pod would violate a PDB, the API returns `429 Too Many Requests` and the drain waits. That's different from node-pressure eviction, which can't wait for PDBs.

---

## Commands / YAML

### Demo: priority and preemption on a single-node kind cluster

```bash
kind create cluster --name cka-2025
```

**Step 1: capacity and existing classes**

```bash
kubectl get nodes
kubectl describe node cka-2025-control-plane | sed -n '/Allocatable:/,/System Info/p'
kubectl get pc
```

```
NAME                     STATUS   ROLES           AGE   VERSION
cka-2025-control-plane   Ready    control-plane   23m   v1.37.0

Allocatable:
  cpu:                11
  memory:             16110752Ki

NAME                      VALUE        GLOBAL-DEFAULT   AGE
system-cluster-critical   2000000000   false            24m
system-node-critical      2000001000   false            24m
```

(The CPU count is whatever your machine gives Docker; Day 46's had **11**.)

Who uses the system classes:

```bash
kubectl describe pod -n kube-system etcd-cka-2025-control-plane | grep -i priority
kubectl describe pod -n kube-system kube-apiserver-cka-2025-control-plane | grep -i priority
```

```
Priority:             2000001000
Priority Class Name:  system-node-critical
```

**Observation:** the **2 billion+** range is reserved for system components; user classes max out at **1 billion**. Trying `value: 100000000000` fails:

```
error when creating "pc.yaml": PriorityClass in version "v1" cannot be handled as a PriorityClass: json: cannot unmarshal number 100000000000 into Go struct field PriorityClass.value of type int32
```

(That number doesn't even fit in a 32-bit integer. A value between 1,000,000,001 and 2,147,483,647 fits the type but is rejected by validation: `must be no greater than 1000000000`.)

**Step 2: create classes** (`Day 46/01-priorityclass.yaml`)

```yaml
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: high-priority
value: 1000
globalDefault: false
description: "Critical production workloads that must preempt lower-priority pods if needed"
---
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: medium-priority
value: 100
globalDefault: false
description: "Important batch or background workloads that can tolerate delay"
---
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: low-priority
value: 10
globalDefault: true               # Pods without priorityClassName get this one
description: "Default for development and non-critical test workloads"
```

```bash
kubectl apply -f "Day 46/01-priorityclass.yaml"
```

**Step 3: low-priority Pods** (`Day 46/02-low-priority-nginx.yaml`): 5 replicas × **1 vCPU** request, no class given → `low-priority` (global default).

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: lp-nginx
spec:
  replicas: 5
  selector:
    matchLabels:
      app: lp-nginx            # (Day 46 used app: nginx for both Deployments; see the correction below)
  template:
    metadata:
      labels:
        app: lp-nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.29
        resources:
          requests:
            memory: "64Mi"
            cpu: "1000m"
```

```bash
kubectl apply -f 02-low-priority-nginx.yaml
kubectl describe pod <lp-nginx-pod> | grep -i priority
```

```
Priority:             10
Priority Class Name:  low-priority
```

Node CPU requests now ≈ control plane (~950m) + 5 × 1000m = **~5.95 vCPU**.

**Step 4: high-priority Pods** (`Day 46/03-high-priority-nginx.yaml`): 5 replicas × **3 vCPU** = 15 vCPU requested, class `high-priority`.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hp-nginx
spec:
  replicas: 5
  selector:
    matchLabels:
      app: hp-nginx
  template:
    metadata:
      labels:
        app: hp-nginx
    spec:
      priorityClassName: high-priority
      containers:
      - name: nginx
        image: nginx:1.29
        resources:
          requests:
            memory: "64Mi"
            cpu: "3000m"
```

```bash
kubectl get pods -w        # in another terminal first
kubectl apply -f 03-high-priority-nginx.yaml
```

```
hp-nginx-5746bb7b5c-2xk9p   0/1   Pending             0     0s
lp-nginx-6c8f7d669b-7qj2m   1/1   Terminating         0     3m
lp-nginx-6c8f7d669b-x8w4d   1/1   Terminating         0     3m
hp-nginx-5746bb7b5c-2xk9p   0/1   ContainerCreating   0     2s
…
lp-nginx-6c8f7d669b-9mtn6   0/1   Pending             0     1s     ← recreated by its ReplicaSet, now waiting
```

- With 11 vCPU and ~5.95 used, the scheduler **evicts low-priority Pods** to fit the high-priority ones. As more `hp-nginx` Pods are placed, more `lp-nginx` Pods are evicted (their replacements wait `Pending`).

```bash
kubectl describe node cka-2025-control-plane | grep -E 'hp-nginx|lp-nginx'
kubectl get events --field-selector reason=Preempted
```

```
default   hp-nginx-5746bb7b5c-2xk9p   3 (27%)   0 (0%)   64Mi (0%)   0 (0%)
default   lp-nginx-6c8f7d669b-q4r7z   1 (9%)    0 (0%)   64Mi (0%)   0 (0%)

Normal   Preempted   pod/lp-nginx-6c8f7d669b-7qj2m   Preempted by pod 4e1c… on node cka-2025-control-plane
```

> ⚠️ **Fix to the original** ([C3](#c3)): both original Deployments used the **same** label/selector `app: nginx`. Two controllers with **overlapping selectors** is a known anti-pattern (the API warns about it, and tools like Services, PDBs and `kubectl` label queries can't tell the two apps apart). Each Deployment here has its own label (`lp-nginx`, `hp-nginx`). The `:latest` image tag is also pinned.

**Key takeaways:**
- Pod **priority** is used for scheduling order and for choosing preemption victims (and as one factor in node-pressure eviction).
- When resources are insufficient, **lower-priority Pods are preempted** for higher-priority ones.
- PriorityClasses give clear, consistent rules across teams, apps and environments.

**Best practices:**
- Meaningful names: `batch-low`, `api-medium`, `critical-high`.
- Tiers like `batch-job: 50`, `low: 100`, `medium: 500`, `high: 1000`.
- Only **one** `globalDefault` (ideally a low value, so unlabelled workloads are the first victims).
- Use `preemptionPolicy: Never` for important-but-non-disruptive classes.
- **Test preemption** before relying on it in production.

### Node-pressure eviction, seen live

```bash
kubectl get pods -A --field-selector=status.phase=Failed
kubectl describe pod <evicted-pod> | grep -A2 -E 'Status|Reason|Message'
```

```
Status:   Failed
Reason:   Evicted
Message:  The node was low on resource: memory. Threshold quantity: 100Mi, available: 91240Ki. Container app was using 1.2Gi, request is 256Mi, has larger consumption of memory.
```

Kubelet thresholds on a node:

```bash
docker exec lab-worker grep -A6 -i eviction /var/lib/kubelet/config.yaml
```

---

## Hands-on lab

1. Run the Day 46 demo on a single-node kind cluster; record which `lp-nginx` Pods were preempted and when the `hp-nginx` Pods got scheduled.
2. Set `preemptionPolicy: Never` on `high-priority` (uncomment it in `01-priorityclass.yaml`), redeploy: `hp-nginx` Pods now stay **Pending**, and nothing is evicted.
3. Scale `lp-nginx` down: the Pending `hp-nginx` Pods are scheduled **before** the Pending `lp-nginx` replacements (queue priority).
4. Try to create two classes with `globalDefault: true`: the second is rejected.
5. (Careful) Memory pressure on kind: run a BestEffort Pod that allocates a lot of memory (`polinux/stress --vm-bytes` large) while a Guaranteed Pod runs; watch which gets evicted or OOM-killed. Clean up quickly.
6. Delete the cluster: `kind delete cluster --name cka-2025`.

---

## Where you'll use it

- **Protecting critical workloads** in shared clusters: system add-ons, ingress controllers and monitoring get high or system priorities; batch work gets low.
- **Cost-efficient clusters**: low-priority **placeholder** ("balloon") Pods reserve spare capacity and are preempted instantly when real Pods arrive, giving fast scale-up without waiting for new nodes.
- **Incident analysis**: "why was my Pod killed?" → `Preempted` event (scheduler), `Evicted` reason (kubelet), or `OOMKilled` (kernel).

---

## When it breaks

| Symptom | Cause | Fix |
|---|---|---|
| Important Pods keep getting `Preempted` | Their priority is low (or the global default) | Give them a higher class; add PDBs |
| High-priority Pod Pending, nothing preempted | `preemptionPolicy: Never`, or the blocker isn't resources (taints, affinity, PVC) | Read `FailedScheduling`; preemption only frees requests |
| `no PriorityClass with name x was found` | Typo or missing class | Create it, or fix the name |
| Many `Evicted` Pods | Node memory/disk pressure | Right-size requests, add memory/disk, clean images/logs, limit ephemeral storage |
| Evicted Pods clutter `kubectl get pods` | Failed Pods stay until garbage-collected | `kubectl delete pods --field-selector=status.phase=Failed -A` |
| Node flaps in and out of `MemoryPressure` | Usage hovers near the threshold | Raise reservations (`kube-reserved`/`system-reserved`), tune thresholds |

---

## Quick recap

- **PriorityClass**: value up to 1e9 for users; system classes ~2e9; one `globalDefault`; Pods without a class get 0 or the default.
- **Priority** orders the scheduling queue; **preemption** gracefully evicts lower-priority Pods when a new Pod can't fit (PDBs best-effort); `preemptionPolicy: Never` = priority without preempting.
- **Node-pressure eviction** (kubelet): thresholds on memory/disk/PIDs; victims ranked by usage-over-request, then priority, then usage; `Evicted` status.
- **QoS** sets OOM-kill order; **drain** uses the Eviction API, which respects PDBs.

### Interview questions

1. **Q: What's the difference between preemption and eviction?**
   A: Preemption is the scheduler deleting lower-priority Pods to place a new higher-priority Pod; node-pressure eviction is the kubelet removing Pods because the node is short of memory/disk/PIDs; API eviction (drain) is a PDB-respecting removal request.
2. **Q: What does `preemptionPolicy: Never` do?**
   A: Pods of that class are still ordered by their priority in the scheduling queue but will never evict other Pods to make room; they wait instead.
3. **Q: Under memory pressure, which Pods does the kubelet evict first?**
   A: Pods whose memory usage exceeds their requests (BestEffort always do), then lower priority, then those exceeding their requests by the most.

---

## Appendix: notes on the original course material

This topic was rebuilt from the original CKA course notes. This appendix keeps the audit trail out of your way while you learn.

### Sources

Day 46 (introduction; why priority and preemption exist; what a PriorityClass is; preemption and scheduler behaviour; the notes on no proactive eviction and on QoS; special notes and best practices; the single-node kind demo: inspecting capacity and system classes, creating PriorityClasses, `preemptionPolicy: Never`, low-priority Pods, high-priority Pods and preemption; key takeaways; conclusion), files `Day 46/01-priorityclass.yaml`, `02-low-priority-nginx.yaml`, `03-high-priority-nginx.yaml`.

### Corrections to the original notes

Each entry shows what the original said and what's true now. In the topic above, each correction appears as a ✅ **Correct today** line or a ⚠️ **Fix to the original** box, linked to its entry here.

<a id="c1"></a>**C1 · Corrected (Day 46 best practices).** "apply `preemptionPolicy: Never` to logging agents or stateful apps". **Now:** `preemptionPolicy` controls whether Pods of that class **cause** preemption. It gives them **no protection** from being preempted themselves. To **protect** stateful apps or agents from preemption, give them a **higher priority** (and PDBs); to stop a class from **disrupting** others, use `Never`.

<a id="c2"></a>**C2 · Corrected (Day 46 note).** it says runtime issues "are handled by the kubelet and **not tied to priority**". **Now:** **node-pressure eviction** (the kubelet evicting Pods when the node is low on memory/disk/PIDs) **does** use priority as one of its ranking criteria (below). What isn't tied to priority are **CPU throttling** and **kernel OOM kills** (those use QoS-derived `oom_score_adj`).

<a id="c3"></a>**C3 · Corrected (Day 46 demo).** both original Deployments used the **same** label/selector `app: nginx`. Two controllers with **overlapping selectors** is a known anti-pattern (the API warns about it, and tools like Services, PDBs and `kubectl` label queries can't tell the two apps apart). Each Deployment here has its own label (`lp-nginx`, `hp-nginx`). The `:latest` image tag is also pinned.
