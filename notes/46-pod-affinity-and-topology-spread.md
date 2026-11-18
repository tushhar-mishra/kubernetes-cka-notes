# 46 · Pod Affinity, Pod Anti-Affinity and Topology Spread Constraints

> **Level:** ★★ intermediate
>
> **You will learn:** how to place Pods **relative to other Pods** instead of relative to node labels: **pod affinity** (co-locate: put the cache next to the app), **pod anti-affinity** (separate: never two replicas on one node or in one zone), the crucial `topologyKey`, required vs preferred forms, namespace scoping, `matchLabelKeys`; and **topology spread constraints** (`maxSkew`, `whenUnsatisfiable`, `minDomains`): the modern, efficient way to spread replicas evenly across zones and nodes. Plus cluster-wide defaults and the Descheduler.
>
> **You need:** topics 17, 42 and 44. You've already seen pod anti-affinity in action in topic 40 (MySQL spread across AZs).

---

## ⚡ Core: the 10-minute version

**The idea.** You can place Pods **relative to other Pods**:
- **Pod affinity**: "put me near Pods labelled X" (a cache next to its app).
- **Pod anti-affinity**: "keep me away from Pods labelled X" (never two replicas on one node).
- **Topology spread constraints**: "keep my replicas evenly spread across zones or nodes".

"Near" is defined by a `topologyKey`: the node's hostname for the same node, the zone for the same zone.

**Picture it.** Seating guests at a wedding: couples together (affinity), rivals apart (anti-affinity), and the same number of guests at every table (spread).

**The one thing to know.**

```yaml
topologySpreadConstraints:
- maxSkew: 1                                   # difference between busiest and emptiest node ≤ 1
  topologyKey: kubernetes.io/hostname          # spread across nodes (use topology.kubernetes.io/zone for zones)
  whenUnsatisfiable: DoNotSchedule
  labelSelector: { matchLabels: { app: web } }
```

**Try it now:**
1. Deploy 4 replicas with this constraint.
2. `kubectl get pods -o wide`: evenly split across your nodes.
3. Change to a required anti-affinity on hostname with 4 replicas on 2 workers: two stay Pending.

**You've got it if you can:** choose between anti-affinity and topology spread for "survive a node or zone failure".

> 📖 **That's the core.** On a first pass, do the mini-lab and move on to the next topic. Everything below is the **full reference**: come back for the details, the edge cases, interview answers and exam-level depth.

---

## What it is

| Feature | Question it answers | Typical use |
|---|---|---|
| **Pod affinity** | "Schedule me **near** Pods with label X" | Put a cache Pod in the same zone/node as the app using it |
| **Pod anti-affinity** | "Schedule me **away from** Pods with label X" | Never put two replicas of a database on the same node/zone |
| **Topology spread constraints** | "Keep my replicas **evenly spread** across zones/nodes (within N of each other)" | Spread a web Deployment over 3 AZs: 4/3/3, never 10/0/0 |

"Near" and "away" are measured per **topology domain**, chosen by **`topologyKey`**: a **node label** whose value defines the domain:
- `kubernetes.io/hostname` → each **node** is a domain;
- `topology.kubernetes.io/zone` → each **zone** is a domain;
- `topology.kubernetes.io/region`, or any custom label (`rack`), likewise.

---

## Why it exists

- **High availability:** 3 replicas on the **same node** die together when that node dies; 3 replicas in the **same zone** die together in a zone outage. Default scheduling only *tends* to spread (by resources); it doesn't guarantee it.
- **Performance:** chatty services in the **same zone** avoid cross-zone latency **and cross-zone data transfer charges**.
- **Licensing and noisy neighbours:** keep two heavy workloads apart.

---

## Analogy

Planning **seating at a wedding**:
- **Pod affinity** = "seat the bride's family at the **same table** as the bride".
- **Pod anti-affinity** = "**never** seat two ex-partners at the same table" (or, for zone anti-affinity, "not even in the same **room**").
- **Topology spread** = "**balance** guests across tables: no table may have more than 2 more people than any other".
- **`topologyKey`** = whether "together" means **same table** (node) or **same room** (zone).

---

## How it works inside

### Pod affinity / anti-affinity rules

```yaml
affinity:
  podAntiAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:     # HARD rule
    - labelSelector:                                     # which OTHER Pods we're talking about
        matchLabels: { app: web }
      topologyKey: kubernetes.io/hostname               # "same domain" = same node
      # namespaces: [shop]  /  namespaceSelector: {}    # where to look for those Pods (default: this Pod's namespace)
      # matchLabelKeys: [pod-template-hash]             # only compare against Pods with the same value of these keys
```

Meaning: "**don't** schedule me in a **node** (domain) that already runs a Pod labelled `app=web`".

How the scheduler evaluates it, for each candidate node:
1. Read the node's value of `topologyKey` (e.g. its hostname, or its zone `us-east-1a`).
2. Find all **existing** Pods matching `labelSelector` (in the chosen namespaces) that run on nodes with the **same** value.
3. **Affinity**: the node is allowed only if there's **at least one** such Pod in its domain. **Anti-affinity**: only if there are **none**.
4. Rules are checked **symmetrically** for anti-affinity: an existing Pod's anti-affinity can also block a new Pod.

Forms:
- **`requiredDuringSchedulingIgnoredDuringExecution`**: hard; if impossible, the Pod stays **Pending**.
- **`preferredDuringSchedulingIgnoredDuringExecution`**: soft; each term has a `weight` (1–100) that adds to the node's score.

```yaml
affinity:
  podAntiAffinity:
    preferredDuringSchedulingIgnoredDuringExecution:
    - weight: 100
      podAffinityTerm:
        labelSelector: { matchLabels: { app: web } }
        topologyKey: topology.kubernetes.io/zone
```

**Gotchas:**
- **Self-matching**: a Deployment's own replicas carry the label they select, so required **anti**-affinity on `kubernetes.io/hostname` means **at most one replica per node**. With 3 nodes and 4 replicas, the 4th is Pending (exactly Day 44's zone example: 4 MySQL Pods, 3 AZs → Pending; the fix was the **preferred** form).
- **Rolling updates**: with required per-node anti-affinity and `maxSurge`, new Pods may not fit next to old ones. Use **`matchLabelKeys: [pod-template-hash]`** so the rule only counts Pods of the **same revision**, or use preferred rules.
- **Cost**: inter-pod affinity is computationally **expensive** for the scheduler in large clusters (it must look at Pods across nodes). The docs advise against it beyond several hundred nodes; topology spread is cheaper.
- `IgnoredDuringExecution` again: nothing moves if the situation changes later.

**Pod affinity example (co-location):**

```yaml
# a cache that must run in the same zone as the app it serves
affinity:
  podAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
    - labelSelector: { matchLabels: { app: checkout } }
      topologyKey: topology.kubernetes.io/zone
```

### Topology spread constraints

```yaml
spec:
  topologySpreadConstraints:
  - maxSkew: 1                                  # max allowed difference between the busiest and emptiest domain
    topologyKey: topology.kubernetes.io/zone    # domains = zones
    whenUnsatisfiable: DoNotSchedule            # hard (default)… or ScheduleAnyway (soft: just a score)
    labelSelector:
      matchLabels: { app: web }                 # which Pods to count
    matchLabelKeys: [pod-template-hash]         # count only Pods of the same revision (rolling updates)
    minDomains: 3                               # treat it as if at least 3 zones exist (with DoNotSchedule)
    nodeAffinityPolicy: Honor                   # only count domains of nodes this Pod's nodeAffinity allows
    nodeTaintsPolicy: Ignore                    # whether tainted nodes count as domains
  - maxSkew: 1
    topologyKey: kubernetes.io/hostname         # AND spread over nodes inside the zones
    whenUnsatisfiable: ScheduleAnyway
    labelSelector:
      matchLabels: { app: web }
```

**How `maxSkew` works:** skew = (matching Pods in a domain) − (minimum matching Pods in any eligible domain). The scheduler only allows a placement if, **after** placing the Pod, every domain's skew is ≤ `maxSkew`.

Example: zones A/B/C currently have 2/2/1 `app=web` Pods, `maxSkew: 1`:
- placing in **C** → 2/2/2, skew 0 ✅
- placing in **A** → 3/2/1, skew 2 ❌ (with `DoNotSchedule`)

So with 10 replicas over 3 zones you get **4/3/3**, never 10/0/0.

**Why `minDomains`:** with cluster autoscaling, a zone may currently have **zero nodes**, so it isn't a domain at all, and the spread happily puts everything in the 2 remaining zones. `minDomains: 3` makes the scheduler act as if 3 domains are needed, leaving Pods Pending and triggering scale-up in the empty zone.

**Spread vs anti-affinity:**

| | Pod anti-affinity (required, hostname) | Topology spread (maxSkew 1, hostname) |
|---|---|---|
| Replicas > nodes | Extra replicas **Pending** | Allowed: spreads evenly, 2 per node and so on |
| Expressiveness | "None in the same domain" | "Balanced within N" |
| Scheduler cost | Higher | Lower |
| Best for | Strict one-per-domain (DB replicas, quorum members) | General HA spreading of stateless apps |

### Cluster-wide defaults

The scheduler applies **default spread constraints** to Pods that don't set any (spread by hostname and zone with `ScheduleAnyway`, using the Pod's Service/ReplicaSet as the selector). That's why replicas usually spread "on their own" somewhat. You can change the defaults in the `KubeSchedulerConfiguration` `PodTopologySpread` plugin args.

### Things only happen at scheduling time

None of these rules move running Pods. After a zone recovers or nodes are added, replicas stay unbalanced. The **Descheduler** (a kubernetes-sigs add-on) can periodically evict Pods that violate spread/affinity constraints so the scheduler places them again.

---

## Commands / YAML

### Anti-affinity: one replica per node

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web
spec:
  replicas: 3
  selector: { matchLabels: { app: web } }
  template:
    metadata: { labels: { app: web } }
    spec:
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector: { matchLabels: { app: web } }
            topologyKey: kubernetes.io/hostname
      tolerations:                                   # allow the control-plane node too (kind lab only)
      - { key: node-role.kubernetes.io/control-plane, operator: Exists, effect: NoSchedule }
      containers:
      - name: nginx
        image: nginx:1.29
```

```bash
kubectl apply -f web.yaml
kubectl get pods -l app=web -o wide --no-headers | awk '{print $7}'
kubectl scale deploy web --replicas=4
kubectl describe pod $(kubectl get pod -l app=web --field-selector=status.phase=Pending -o name | head -1) | tail -2
```

```
lab-control-plane
lab-worker
lab-worker2

Warning  FailedScheduling  … 0/3 nodes are available: 3 node(s) didn't match pod anti-affinity rules. preemption: 0/3 nodes are available: 3 No preemption victims found for incoming pod.
```

### Topology spread on kind (simulate zones with labels)

```bash
kubectl label nodes lab-control-plane topology.kubernetes.io/zone=zone-a
kubectl label nodes lab-worker        topology.kubernetes.io/zone=zone-b
kubectl label nodes lab-worker2       topology.kubernetes.io/zone=zone-c
```

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: spread
spec:
  replicas: 7
  selector: { matchLabels: { app: spread } }
  template:
    metadata: { labels: { app: spread } }
    spec:
      topologySpreadConstraints:
      - maxSkew: 1
        topologyKey: topology.kubernetes.io/zone
        whenUnsatisfiable: DoNotSchedule
        labelSelector: { matchLabels: { app: spread } }
        matchLabelKeys: [pod-template-hash]
      tolerations:
      - { key: node-role.kubernetes.io/control-plane, operator: Exists, effect: NoSchedule }
      containers:
      - name: nginx
        image: nginx:1.29
```

```bash
kubectl apply -f spread.yaml
kubectl get pods -l app=spread -o wide --no-headers | awk '{print $7}' | sort | uniq -c
```

```
      2 lab-control-plane
      3 lab-worker
      2 lab-worker2
```

Balanced 3/2/2 (skew 1).

### Pod affinity: co-locate

```yaml
apiVersion: v1
kind: Pod
metadata: { name: cache, labels: { app: cache } }
spec:
  affinity:
    podAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector: { matchLabels: { app: web } }
        topologyKey: kubernetes.io/hostname
  containers: [{ name: redis, image: "redis:8" }]
```

```bash
kubectl get pod cache -o wide     # lands on a node that already runs an app=web Pod
```

---

## Hands-on lab

1. Run the anti-affinity Deployment; scale to 4 and see the Pending replica; switch to the **preferred** form and see it schedule.
2. Do a rolling update (`kubectl set image deploy/web nginx=nginx:1.28`) with the required rule: watch it get stuck (the surge Pod has nowhere to go). Add `matchLabelKeys: [pod-template-hash]` to the anti-affinity term and try again.
3. Run the spread example with 7 and then 10 replicas; check the distribution each time.
4. Cordon `lab-worker2` (`kubectl cordon`) and scale the spread Deployment up: with `DoNotSchedule` some replicas go Pending (zone-c can't take them, and the others would exceed the skew). Change to `ScheduleAnyway` and compare. Uncordon.
5. Run the `cache` Pod with pod affinity.
6. Clean up and remove the zone labels (`kubectl label nodes --all topology.kubernetes.io/zone-`).

---

## Where you'll use it

- **Every HA production Deployment**: zone spread (topology spread) and node spread.
- **Stateful systems**: required anti-affinity per zone or node for quorum members (etcd, Kafka, ZooKeeper, database replicas; topic 40).
- **Cost and latency**: affinity to keep chatty services in one zone; `trafficDistribution: PreferClose` on Services (topic 31) complements it.
- **PodDisruptionBudgets** (topic 76) protect the spread during voluntary disruptions like drains.

---

## When it breaks

| Symptom | Cause | Fix |
|---|---|---|
| `didn't match pod anti-affinity rules` | Required anti-affinity + more replicas than domains | Add domains, use preferred, or topology spread |
| `didn't match pod topology spread constraints` | `DoNotSchedule` + skew would be exceeded (cordoned/full domain) | Add capacity in that domain, raise `maxSkew`, or `ScheduleAnyway` |
| Rolling update stuck | Old Pods count against new ones | `matchLabelKeys: [pod-template-hash]`, or preferred rules |
| Everything in two zones even though there are three | Third zone has no nodes right now (not a domain) | `minDomains`, autoscaler config per zone |
| Spread looks wrong after an outage | Rules apply only at scheduling time | Descheduler, or roll the Deployment |
| Scheduling slow in a huge cluster | Many inter-pod affinity rules | Prefer topology spread |
| Affinity "ignored" | `labelSelector` doesn't match the target Pods, or wrong `namespaces` | Check labels and namespace scope |

---

## Quick recap

- **Pod affinity** = co-locate with matching Pods; **anti-affinity** = avoid them; the **domain** comes from `topologyKey` (hostname = node, zone = zone).
- Required = hard (Pending if impossible); preferred = weighted score.
- **Topology spread**: `maxSkew` + `topologyKey` + `whenUnsatisfiable`, plus `minDomains` and `matchLabelKeys`: balanced, cheaper, works when replicas > domains.
- All rules are scheduling-time only; the Descheduler can rebalance.

### Interview questions

1. **Q: How do you guarantee no two replicas run on the same node?**
   A: Required pod anti-affinity with `topologyKey: kubernetes.io/hostname` selecting the app's own label (extra replicas beyond the node count stay Pending), or a topology spread constraint by hostname with `maxSkew: 1` if you just need balance.
2. **Q: What does `maxSkew` mean?**
   A: The maximum allowed difference in the number of matching Pods between any two eligible topology domains after placement.
3. **Q: Pod affinity vs node affinity?**
   A: Node affinity selects nodes by their labels; pod affinity/anti-affinity selects nodes based on which Pods (by label) are already running in the same topology domain.

---

## Appendix: notes on the original course material

This topic was rebuilt from the original CKA course notes. This appendix keeps the audit trail out of your way while you learn.

### Sources

new topic. It expands the pod anti-affinity rules used in Day 44 (required/preferred zone anti-affinity, single-AZ `kubernetes.io/hostname` variant) and the brief mentions of affinity in Days 16 and 18.
