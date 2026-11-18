# 44 · nodeSelector and Node Affinity: Pulling Pods onto the Right Nodes

> **Level:** ★★ intermediate
>
> **You will learn:** label-based placement; **`nodeSelector`** (simple, strict, AND-only) and its limits; **node affinity**: the required (hard) and preferred (soft) forms, `nodeSelectorTerms` (OR) vs `matchExpressions` (AND), the operators `In`, `NotIn`, `Exists`, `DoesNotExist`, `Gt`, `Lt`, weights and how they score; "anti-affinity" with `NotIn`/`DoesNotExist`; what "IgnoredDuringExecution" means (nothing is re-evaluated after scheduling, unlike `NoExecute` taints); and the well-known node labels you can use.
>
> **You need:** topics 17 (labels/selectors), 42 and 43.

---

## ⚡ Core: the 10-minute version

**The idea.** To **pull** Pods onto particular nodes, use node labels:
- **`nodeSelector`** is the simple form: "only nodes with these exact labels".
- **Node affinity** is the flexible form. `required…` is a hard rule (the Pod stays Pending if no node matches); `preferred…` is a soft, weighted wish. It also has operators: `In`, `NotIn`, `Exists`, `DoesNotExist`, `Gt`, `Lt`.

Both are checked only at scheduling time.

**Picture it.** nodeSelector is "I only eat at restaurants with a vegan label"; preferred affinity is "I'd rather sit by the window, but any table will do".

**The one thing to know.**

```yaml
affinity:
  nodeAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      nodeSelectorTerms:
      - matchExpressions:
        - { key: disktype, operator: In, values: [ssd] }
    preferredDuringSchedulingIgnoredDuringExecution:
    - weight: 50
      preference: { matchExpressions: [ { key: zone, operator: In, values: [a] } ] }
```

**Try it now:**
1. `kubectl label node lab-worker disktype=ssd`, then apply a Pod with the required rule: it lands on `lab-worker`.
2. Remove the label and create another one: it stays Pending.
3. Running Pods don't move when labels change.

**You've got it if you can:** choose between nodeSelector and node affinity, and explain hard vs soft rules.

> 📖 **That's the core.** On a first pass, do the mini-lab and move on to the next topic. Everything below is the **full reference**: come back for the details, the edge cases, interview answers and exam-level depth.

---

## What it is

The scheduler places Pods automatically, but admins often need to **steer** specific workloads onto **specific nodes**. Kubernetes gives two **label-based** tools:

1. **`nodeSelector`** (basic): "only nodes with **these exact labels**".
2. **Node affinity** (advanced): hard **and** soft rules, **OR/AND** logic, set-based operators.

Both select nodes by their **labels** (topic 17). Taints (topic 43) **repel**; selectors and affinity **attract**.

**Why place Pods on specific nodes:**
- workloads that need **special hardware** (GPUs, SSDs);
- **environment isolation** (prod vs staging);
- respecting **resource or licensing constraints** (e.g. only large nodes).

---

## Why it exists

Taints keep Pods **off** nodes, but tolerations never **send** a Pod anywhere (topic 43). To say "this Pod **must** (or **should**) run on SSD nodes, in zone A, on arm64", you need label-matching rules the scheduler enforces.

---

## Analogy

A **job application with requirements**:
- **`nodeSelector`** = "Must have: driving licence AND forklift certificate." No flexibility: no match, no job (Pending).
- **Required node affinity** = the same, but you can write **"SSD OR NVMe"**, **"anything except dev"**, **"has a GPU label of any kind"**.
- **Preferred node affinity** = "**Nice to have**: SSD (10 points), HDD (5 points)." The best-scoring candidate wins, but someone still gets the job.

---

## How it works inside

### nodeSelector

```yaml
spec:
  nodeSelector:
    storage: ssd
    env: prod
```

- The simplest way to tie a Pod to nodes by **labels**.
- Nodes must carry **all** the listed labels (**AND**).
- **Exact key=value match only**: no operators.
- No matching node → the Pod stays **`Pending`**.

**How it works:**
1. Label nodes: `kubectl label nodes <node-name> <label-key>=<label-value>`.
2. Add `nodeSelector` to the Pod spec.
3. The scheduler only considers nodes carrying those labels (the `NodeAffinity` filter plugin handles `nodeSelector` too).

**Limitations:**

| Limitation | Explanation |
|---|---|
| **Strict placement** | No matching node → Pending |
| **No preferences** | No "soft" rules: match or don't |
| **No OR** | Can't say `storage=ssd` **or** `storage=hdd` |

### Node affinity

An advanced version of `nodeSelector`:
- **hard constraints** (must match) and **soft preferences** (prefer, not mandatory);
- complex **AND/OR** conditions;
- **set-based** operators (`In`, `NotIn`, `Exists`…).

| Type | Behaviour |
|---|---|
| `requiredDuringSchedulingIgnoredDuringExecution` | **Hard rule**: only matching nodes; otherwise **Pending** |
| `preferredDuringSchedulingIgnoredDuringExecution` | **Soft rule**: matching nodes score higher, but other nodes are allowed |

**"IgnoredDuringExecution"** = rules are checked **only at scheduling time**. If a node's labels change later, Pods already running there **stay**.

### The structure: terms are OR, expressions are AND

```yaml
affinity:
  nodeAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      nodeSelectorTerms:            # list of TERMS: a node must match ANY ONE term (OR)
      - matchExpressions:           # within a term: ALL expressions must match (AND)
        - { key: storage, operator: In, values: [ssd] }
        - { key: env,     operator: In, values: [prod] }
      - matchExpressions:           # OR this whole other term
        - { key: storage, operator: In, values: [nvme] }
```

Reads as: **(storage=ssd AND env=prod) OR (storage=nvme)**.

Other combinations:
- `values: [ssd, hdd]` under `In` = **OR within one key**.
- If a Pod has **both** `nodeSelector` and `nodeAffinity`, **both** must be satisfied.
- `matchFields` (instead of `matchExpressions`) matches **node fields**, practically only `metadata.name` (that's how DaemonSets pin Pods to nodes; topic 27).

### Operators

| Operator | Matches when |
|---|---|
| `In` | The node's label value is **in** the list |
| `NotIn` | The label value is **not** in the list (also true if the label is **missing**) |
| `Exists` | The node **has** the key (any value; no `values` allowed) |
| `DoesNotExist` | The node **doesn't have** the key |
| `Gt` | The label value, parsed as an **integer**, is **greater** than the single given value |
| `Lt` | The label value, as an integer, is **less** than the given value |

> **Clarification (Day 17's all-operators example):** `cpu Gt 4` means "the node has a **label** called `cpu` whose value is a number above 4". It does **not** read the node's real CPU count. Those are only labels an admin (or a tool like Node Feature Discovery) put there. Real capacity is handled by **requests** (topic 41).

### Weights for preferred rules

```yaml
preferredDuringSchedulingIgnoredDuringExecution:
- weight: 10                     # 1–100
  preference:
    matchExpressions: [{ key: storage, operator: In, values: [ssd] }]
- weight: 5
  preference:
    matchExpressions: [{ key: storage, operator: In, values: [hdd] }]
```

**How weights work:**
1. The scheduler first keeps only nodes passing the **required** rules (and every other filter).
2. For each node, it **adds up the weights** of the preferred terms that node matches.
3. That sum becomes the `NodeAffinity` **score**, which is **normalised and combined with the other scoring plugins** (resource balance, image locality, spreading…). The highest **total** wins.

So weights **influence** placement but don't guarantee it.

### "Anti-affinity" for nodes

There's no separate `nodeAntiAffinity` field. To keep Pods **away** from nodes:
- use `NotIn` or `DoesNotExist` in node affinity;
- or **taints** (which also repel Pods that don't even know about the rule).

(**Pod** anti-affinity, which keeps Pods away from **other Pods**, is a different feature; topic 46.)

### Nothing is re-evaluated after scheduling

- **`nodeSelector` is not retroactive:** it's applied **at scheduling time**. Change the node's labels afterwards and the Pod **stays** until it's deleted and recreated.
- **Node affinity** behaves the same (that's what `IgnoredDuringExecution` says).
- **Taints with `NoExecute`** are the exception: they **are** re-evaluated, and running Pods without a matching toleration are **evicted** (topic 43).

To actually move Pods after relabelling nodes, restart the workload (`kubectl rollout restart`), or use the **Descheduler** add-on, which evicts Pods violating current affinity/taint rules so they get rescheduled.

### Useful built-in node labels

| Label | Example value | Set by |
|---|---|---|
| `kubernetes.io/hostname` | `lab-worker` | kubelet |
| `kubernetes.io/os` | `linux` / `windows` | kubelet |
| `kubernetes.io/arch` | `amd64` / `arm64` | kubelet |
| `topology.kubernetes.io/region` | `us-east-1` | cloud controller |
| `topology.kubernetes.io/zone` | `us-east-1a` | cloud controller |
| `node.kubernetes.io/instance-type` | `m6i.large` | cloud controller |
| `node-role.kubernetes.io/control-plane` | `""` | kubeadm |
| `eks.amazonaws.com/nodegroup`, `cloud.google.com/gke-nodepool` | pool names | managed services |

(Note: labels with the `node-role.kubernetes.io/` or `kubernetes.io/` prefixes **can't be set by the kubelet itself** for security reasons (NodeRestriction admission, topic 58); admins set them with kubectl.)

---

## Commands / YAML

**Cluster setup:** `lab-control-plane` (control plane), `lab-worker` and `lab-worker2` (workers). (Day 17 used `my-second-cluster-…` names.)

### nodeSelector demo

**Step 1: see existing labels**

```bash
kubectl get nodes --show-labels
kubectl describe node lab-worker | sed -n '/Labels:/,/Annotations:/p'
```

```
Labels:             beta.kubernetes.io/arch=amd64
                    beta.kubernetes.io/os=linux
                    kubernetes.io/arch=amd64
                    kubernetes.io/hostname=lab-worker
                    kubernetes.io/os=linux
```

**Step 2: label a node**

```bash
kubectl label nodes lab-worker storage=ssd
```

**Step 3: a Deployment with `nodeSelector`**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ns-deploy
spec:
  replicas: 3
  selector:
    matchLabels:
      app: app1
  template:
    metadata:
      labels:
        app: app1
    spec:
      nodeSelector:
        storage: ssd               # only nodes labelled storage=ssd
      containers:
        - name: nginx
          image: nginx:1.29
```

All Pods land on `lab-worker`. With no `storage=ssd` node, they'd stay Pending.

**Step 4: several labels (AND)**

```bash
kubectl label nodes lab-worker env=prod
```

```yaml
      nodeSelector:
        storage: ssd
        env: prod
```

✔️ Only nodes with **both** labels. ❌ None → Pending. (`Day 17/ns.yaml` is this version.)

**Step 5: remove a label**

```bash
kubectl label nodes lab-worker storage-
```

New Pods with `nodeSelector: {storage: ssd}` can no longer be scheduled; **existing ones keep running** (not retroactive).

**Key takeaways:** simple and strict; all labels must match; too inflexible for complex needs; node affinity is the advanced option.

### Required node affinity demo

```bash
kubectl label nodes lab-worker storage=ssd
```

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: na-deploy
spec:
  replicas: 3
  selector:
    matchLabels:
      app: app1
  template:
    metadata:
      labels:
        app: app1
    spec:
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: storage
                operator: In
                values:
                  - ssd
      containers:
        - name: nginx
          image: nginx:1.29
```

Expected: only `storage=ssd` nodes; otherwise Pending. Same as `nodeSelector` here, but now complex conditions are possible. (Set-based selectors were introduced in topic 17/24.)

**OR condition:**

```bash
kubectl label nodes lab-worker2 storage=hdd
```

```yaml
            nodeSelectorTerms:
            - matchExpressions:
              - key: storage
                operator: In
                values:
                  - ssd
                  - hdd
```

Pods may go to ✅ `lab-worker` (ssd) or ✅ `lab-worker2` (hdd).

**AND condition (Day 17, `Day 17/na.yaml` with 20 replicas):**

```bash
kubectl label nodes lab-worker env=prod
```

```yaml
            nodeSelectorTerms:
            - matchExpressions:
              - key: storage
                operator: In
                values: [ssd]
              - key: env
                operator: In
                values: [prod]
```

Only nodes with **both** `storage=ssd` and `env=prod`; otherwise Pending.

**All operators:**

```yaml
affinity:
  nodeAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      nodeSelectorTerms:
      - matchExpressions:
        - { key: storage,     operator: In,           values: [ssd, hdd] }
        - { key: env,         operator: NotIn,        values: [dev] }
        - { key: high-memory, operator: Exists }
        - { key: dedicated,   operator: DoesNotExist }
        - { key: cpu,         operator: Gt,           values: ["4"] }
        - { key: disk,        operator: Lt,           values: ["500"] }
```

| Condition | Effect |
|---|---|
| `storage In (ssd, hdd)` | Node's `storage` is ssd **or** hdd |
| `env NotIn (dev)` | Not on nodes labelled `env=dev` |
| `high-memory Exists` | Only nodes that have a `high-memory` label |
| `dedicated DoesNotExist` | Avoid nodes with a `dedicated` label |
| `cpu Gt 4` | Only nodes whose **`cpu` label** value is above 4 |
| `disk Lt 500` | Only nodes whose **`disk` label** value is below 500 |

All six are in one term, so **all** must hold.

**Summary:** more flexible than nodeSelector; AND/OR through `matchExpressions`/terms; strict and soft rules; `NotIn`/`DoesNotExist` for anti-affinity behaviour.

### Preferred node affinity demo (Day 17, `Day 17/na-preferred.yaml` with 20 replicas)

**Step 1: label nodes**

```bash
kubectl label nodes lab-worker storage=ssd --overwrite
kubectl label nodes lab-worker2 storage=hdd --overwrite
```

**Step 2: required + preferred**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: preferred-na-deploy
spec:
  replicas: 5
  selector:
    matchLabels:
      app: app1
  template:
    metadata:
      labels:
        app: app1
    spec:
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:   # MUST be ssd or hdd
            nodeSelectorTerms:
            - matchExpressions:
              - key: storage
                operator: In
                values: [ssd, hdd]
          preferredDuringSchedulingIgnoredDuringExecution:  # PREFER ssd (10) over hdd (5)
            - weight: 10
              preference:
                matchExpressions:
                  - { key: storage, operator: In, values: [ssd] }
            - weight: 5
              preference:
                matchExpressions:
                  - { key: storage, operator: In, values: [hdd] }
      containers:
        - name: nginx
          image: nginx:1.29
```

**Explanation:** must run on `storage=ssd` **or** `storage=hdd` (hard); prefers ssd (weight 10) over hdd (weight 5).

**Step 3: deploy and observe**

```bash
kubectl apply -f preferred-na-deploy.yaml
kubectl get pods -o wide --sort-by=.spec.nodeName
```

**Expected:** Pods favour `lab-worker` (ssd); if it lacks resources, some go to `lab-worker2`. More replicas make the preference more visible. In practice you'll often see a mix even with room on `lab-worker`, because the affinity score competes with resource-balancing scores.

**Key considerations:**
- Weight **influences** placement; it doesn't guarantee it.
- Pods may land on lower-weight nodes if preferred ones lack resources.
- Other factors still apply: free CPU/memory, other affinity/anti-affinity rules, taints and tolerations.
- Use **required** for strict rules and **preferred** for weighted preferences.

---

## Hands-on lab

1. Do the nodeSelector demo, then remove the label and confirm the existing Pods keep running while a new replica (scale up) stays Pending.
2. Apply `Day 17/na.yaml` with only one node labelled `storage=ssd,env=prod`: all 20 replicas on that node.
3. Write a two-term rule: `(storage=ssd AND env=prod) OR (storage=hdd)` and check placement.
4. Apply `Day 17/na-preferred.yaml` and count Pods per node: `kubectl get pods -o wide --no-headers | awk '{print $7}' | sort | uniq -c`. Then change the weights to 100/1 and compare.
5. Label nodes `cpu=8` and `cpu=2`, and schedule with `cpu Gt 4`.
6. Clean up: `kubectl delete deploy --all; kubectl label nodes lab-worker lab-worker2 storage- env- cpu-`.

---

## Where you'll use it

- **Hardware-specific workloads**: GPU nodes, high-memory nodes, arm64 vs amd64 (`kubernetes.io/arch`).
- **Zones**: pin or prefer workloads per `topology.kubernetes.io/zone`.
- **Node pools**: put workloads on the right managed node group (`eks.amazonaws.com/nodegroup`).
- **Licensing and compliance**: run regulated workloads only on dedicated nodes (with taints, topic 45).

---

## When it breaks

| Symptom | Cause | Fix |
|---|---|---|
| `Pending`: `node(s) didn't match Pod's node affinity/selector` | No node has the required labels (typo, wrong value, label removed) | `kubectl get nodes --show-labels`; fix labels or the rule |
| Pods ignore your preference | Preferred weight outweighed by other scores, or preferred nodes are full | Raise the weight, use required, check capacity |
| Pods still on old nodes after relabelling | IgnoredDuringExecution | Roll the workload or use the Descheduler |
| `Gt`/`Lt` rule never matches | Label value isn't an integer, or more than one value given | One integer value; numeric labels |
| `Exists` rejected: `values must be empty` | `values` given with Exists/DoesNotExist | Remove `values` |
| Pods land on the right labelled node **and** other Pods also land there | Affinity attracts but doesn't repel | Add a taint (topic 45) |

---

## Quick recap

- **nodeSelector**: exact labels, AND only, hard.
- **Node affinity**: **required** (hard) / **preferred** (soft, weights 1–100); **terms = OR, expressions = AND**; operators In, NotIn, Exists, DoesNotExist, Gt, Lt (on **label** values).
- Anti-affinity to nodes = `NotIn`/`DoesNotExist` (or taints).
- Not retroactive: label changes don't move running Pods (unlike `NoExecute` taints).

### Interview questions

1. **Q: nodeSelector vs node affinity?**
   A: nodeSelector only supports exact key=value matches combined with AND, as a hard rule. Node affinity adds set operators, OR across terms, and soft preferences with weights.
2. **Q: What does "IgnoredDuringExecution" mean?**
   A: The rule is evaluated only when scheduling; if node labels later change, already-running Pods are not evicted or moved.
3. **Q: How do you express "SSD nodes, or any node in zone us-east-1a"?**
   A: Two `nodeSelectorTerms`: one with `storage In [ssd]`, one with `topology.kubernetes.io/zone In [us-east-1a]`. Terms are ORed.

---

## Appendix: notes on the original course material

This topic was rebuilt from the original CKA course notes. This appendix keeps the audit trail out of your way while you learn.

### Sources

Day 17 (introduction, node placement strategies, understanding nodeSelector and how it works, its limitations, cluster setup, nodeSelector demo steps 1–5, key takeaways, node affinity, types, required demo, OR condition, AND condition, operators and the all-operators example with explanation, node anti-affinity, summary, preferred affinity and weights, preferred demo, key considerations, "not retroactive" section for nodeSelector/affinity/NoExecute), files `Day 17/ns.yaml`, `na.yaml`, `na-preferred.yaml`.
