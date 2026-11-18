# 81 · A Troubleshooting Method for Kubernetes

> **Level:** ★★ intermediate
>
> **You will learn:** a repeatable way to debug any Kubernetes problem: define the symptom, **locate the layer** (client → API/control plane → scheduling → node/runtime → Pod/container → network/DNS/Service → storage → application), follow the request path, form and test one hypothesis at a time, and fix with the smallest change. Plus the core toolkit (`describe`, Events, `logs --previous`, `get -o yaml`, `debug`, `auth can-i`, `crictl`, `journalctl`, `ss`, `nslookup`), the status → cause map, and incident-response hygiene (communicate, mitigate first, timeline, runbooks, postmortems).
>
> **You need:** topics 06–10 (architecture, the `kubectl apply` journey), 19–22 (Pod lifecycle, debugging, probes), 29–33 (networking), 79–80.

---

## ⚡ Core: the 10-minute version

**The idea.** Debug any problem with the same method:
1. **Define the symptom** precisely, including "what changed?".
2. **Find the layer** along the request's path: client → API → controllers → scheduler → kubelet/runtime → container → network/DNS → storage.
3. Gather evidence cheapest-first: `get`, `describe` + Events, `logs --previous`, `-o yaml`.
4. Test **one hypothesis at a time**.
5. Make the smallest fix and verify the **original** symptom is gone.

In a real incident: mitigate first, communicate, and keep a timeline.

**Picture it.** A doctor's routine: symptoms, vital signs, the likeliest organ first, one test at a time, and stabilising the patient before investigating.

**The one thing to know.** The 60-second first look:

```bash
kubectl get nodes -o wide
kubectl get pods -A --field-selector=status.phase!=Running,status.phase!=Succeeded
kubectl get events -A --sort-by=.lastTimestamp | tail -20
kubectl describe pod <p> | sed -n '/Events:/,$p'; kubectl logs <p> --previous
```

**Try it now:**
1. Have someone break 3 things in your kind cluster (a bad image, a Service selector typo, a stopped kubelet).
2. Find each one using the method.
3. Time yourself.

**You've got it if you can:** explain how you'd approach any "it's broken" report, and decode exit codes 127, 137 and 143.

> 📖 **That's the core.** On a first pass, do the mini-lab and move on to the next topic. Everything below is the **full reference**: come back for the details, the edge cases, interview answers and exam-level depth.

---

## What it is

**Troubleshooting** is finding **which component** in a chain broke and **why**, using evidence rather than guesses. Kubernetes is layered (API → controllers → scheduler → kubelet → runtime → kernel → network), and each layer leaves **evidence**: object **status and conditions**, **Events**, **logs**, and **metrics**. A method tells you where to look first.

---

## Why it exists

Under pressure, people jump to random fixes ("restart it", "delete the Pod"), which may hide the cause, make it worse, or destroy evidence. A systematic approach is faster on average, works on unfamiliar systems, and is exactly what the CKA (which is ~30% troubleshooting) and on-call rotations reward.

---

## Analogy

A **doctor's diagnostic routine**: ask about symptoms (what exactly is wrong, since when, what changed?), take vitals (status, Events, metrics), examine the most likely organ first (the layer), order a specific test (one command to confirm or reject a hypothesis), treat the cause, then check the patient recovered. And in an emergency, **stabilise first** (mitigate), diagnose second.

---

## How it works inside

### The method

**1. Define the symptom precisely.**
- What exactly fails? (`kubectl` times out? Pod `Pending`? HTTP 503 from the Ingress? Only one node?)
- Since when? **What changed** just before (deploy, config, upgrade, certificate expiry, node replacement)? `kubectl rollout history`, Git/GitOps history, recent Events.
- Scope: one Pod, one node, one namespace, the whole cluster?

**2. Locate the layer**: follow the path of the failing operation:

```
kubectl / client ─► kubeconfig, network, auth (401/403)         topics 12, 52, 54–57
   ▼
API server / etcd ─► reachable? healthy? admission webhooks?    topics 14, 58, 77, 82
   ▼
controllers ─► Deployment → ReplicaSet → Pods created?          topics 24–28, 82
   ▼
scheduler ─► Pod Pending? why? (resources, taints, affinity, PVC) topics 41–47, 82
   ▼
kubelet / runtime ─► image pull, container create, probes, OOM  topics 08, 20–22, 41, 83
   ▼
container / app ─► exit codes, logs, config, dependencies       topics 19, 21, 34–35
   ▼
network ─► Pod IP → Service → EndpointSlices → DNS → NetworkPolicy → Ingress/Gateway   topics 29–33, 61, 69–73, 83
   ▼
storage ─► PVC bound? attach/mount errors? permissions          topics 37–40, 83
```

**3. Gather evidence top-down, cheapest first.**
- `kubectl get <thing> -o wide` (status, node, restarts, age)
- `kubectl describe <thing>` (conditions + **Events** at the bottom)
- `kubectl get events -n <ns> --sort-by=.lastTimestamp`
- `kubectl logs <pod> [-c c] [--previous]`
- `kubectl get <thing> -o yaml` (the full spec and status: what the cluster **actually** has, versus what you think you applied)

**4. Form one hypothesis and test it** with a specific command ("if the selector is wrong, the EndpointSlice is empty": check it). Change **one thing at a time**; note what you did.

**5. Fix with the smallest change**, ideally declaratively (edit the manifest/Git, re-apply), then **verify** the original symptom is gone, not just that "the Pod is Running".

**6. Prevent recurrence**: alert, test, guardrail (admission policy), runbook, postmortem.

### Status → likely cause map

| What you see | First suspects | Look at |
|---|---|---|
| `kubectl`: `connection refused` / timeout | API server down, wrong server/port, firewall | kubeconfig `server:`, `crictl ps` on control plane (topic 82) |
| `Unauthorized` (401) / `Forbidden` (403) | Credentials/cert expired / RBAC | `kubectl auth whoami`, `auth can-i` (topics 54–57) |
| `Internal error … failed calling webhook` | Admission webhook down | Webhook configs (topic 58) |
| Deployment exists, **no Pods** | Controller manager down; quota/LimitRange; PSA rejected | `describe rs` events (topics 41, 60, 82) |
| `Pending` | Scheduler down; insufficient resources; taints; affinity; unbound PVC | `describe pod` → Events (`FailedScheduling`) |
| `ContainerCreating` (stuck) | Volume attach/mount, CNI failure, image pull slow | Events; `crictl`; CNI Pods |
| `ErrImagePull` / `ImagePullBackOff` | Wrong name/tag, private registry, rate limit | Events (topics 20, 62) |
| `CreateContainerConfigError` | Missing ConfigMap/Secret/key; `runAsNonRoot` conflict | Events (topics 34–35, 59) |
| `CrashLoopBackOff` | App exits: bad config, missing dependency, wrong command | `logs --previous`, exit code (topic 21) |
| `OOMKilled` (exit 137) | Memory limit too low or leak | `describe` last state, metrics (topic 41) |
| `Running` but `0/1 READY` | Readiness probe failing | `describe` probe events (topic 22) |
| `Running`, ready, but unreachable | Service selector/ports, NetworkPolicy, DNS, Ingress | EndpointSlices, `nslookup`, policies (topic 83) |
| Node `NotReady` | kubelet/runtime down, CNI, disk/memory pressure, certs | `describe node`, `journalctl -u kubelet` (topic 83) |
| `Terminating` forever | Finalizers; node unreachable | `get -o yaml` finalizers (topics 66, 83) |
| `Evicted` | Node pressure (disk/memory) | Node conditions, kubelet thresholds (topic 47) |

### Exit codes

| Code | Meaning |
|---|---|
| 0 | Exited normally (bad for long-running containers: they "complete" → restart) |
| 1 / 2 | Application error / misuse |
| 126 / 127 | Command not executable / **not found** (typo in `command`, missing binary) |
| 137 | **SIGKILL** (128+9): OOMKilled, or killed after the grace period |
| 139 | SIGSEGV (128+11): segmentation fault |
| 143 | **SIGTERM** (128+15): graceful stop |

### Tools by layer

| Layer | Tools |
|---|---|
| Client | `kubectl config view --minify`, `kubectl cluster-info`, `kubectl version`, `-v=6..9` (HTTP calls) |
| API/auth | `kubectl get --raw='/readyz?verbose'`, `auth whoami`, `auth can-i --list --as=…` |
| Objects | `get -o wide/-o yaml`, `describe`, `get events`, `explain` |
| Pods | `logs [--previous] [-c]`, `exec`, `debug -it --image=busybox --target=<c>` (ephemeral container), `debug --copy-to` (copy with a changed command), `port-forward` |
| Nodes | `kubectl debug node/<n> -it --image=ubuntu` (host namespaces), `ssh`; `systemctl status kubelet containerd`, `journalctl -u kubelet`, `crictl ps -a / logs / inspect / pods`, `df -h`, `free -m`, `dmesg -T` (OOM killer, disk errors) |
| Network | `nslookup/dig`, `curl -v`, `nc -zv host port`, `ss -tlpn`, `ip addr/route`, `iptables-save`/`nft list ruleset`, `tcpdump`, a netshoot debug Pod (`nicolaka/netshoot`) |
| Metrics | `kubectl top`, Grafana/Prometheus (topic 80) |

### Incident hygiene

- **Mitigate first** when users are affected: roll back (`kubectl rollout undo`, `git revert`), scale up, shift traffic, fail over. Root cause can come after service is restored.
- **Communicate**: one incident channel, an incident commander, regular status updates.
- **Keep a timeline** of observations and actions (timestamps): invaluable for the postmortem.
- **Preserve evidence** before deleting things (`kubectl get pod -o yaml > pod.yaml`, `logs --previous > …`, `describe`).
- **Blameless postmortem**: what happened, impact, detection, root cause and contributing factors, what went well, action items with owners.
- **Runbooks** linked from alerts: symptom → checks → fixes (topic 80).

---

## Commands / YAML

### The 60-second first look

```bash
kubectl config current-context && kubectl get --raw='/readyz' && echo
kubectl get nodes -o wide
kubectl get pods -A --field-selector=status.phase!=Running,status.phase!=Succeeded
kubectl get events -A --sort-by=.lastTimestamp | tail -20
kubectl top nodes
```

```
kind-lab
ok
NAME                STATUS   ROLES           AGE   VERSION
lab-control-plane   Ready    control-plane   9d    v1.37.x
lab-worker          Ready    <none>          9d    v1.37.x
lab-worker2         NotReady <none>          9d    v1.37.x       ← a lead
NAMESPACE   NAME                     READY   STATUS             RESTARTS   AGE
shop        api-7f9c6d4b5-2xk8q      0/1     CrashLoopBackOff   6          9m    ← another lead
…
```

### Follow one Pod down the layers

```bash
kubectl describe pod api-7f9c6d4b5-2xk8q -n shop | sed -n '/State:/,/Ready:/p;/Events:/,$p'
kubectl logs api-7f9c6d4b5-2xk8q -n shop --previous --tail=30
kubectl get pod api-7f9c6d4b5-2xk8q -n shop -o jsonpath='{.status.containerStatuses[0].lastState.terminated}'; echo
```

```
    State:          Waiting
      Reason:       CrashLoopBackOff
    Last State:     Terminated
      Reason:       Error
      Exit Code:    1
…
Error: connect ECONNREFUSED 10.96.12.7:5432
{"exitCode":1,"reason":"Error","startedAt":"…","finishedAt":"…"}
```

Hypothesis: the database Service has no endpoints → `kubectl get endpointslices -n shop -l kubernetes.io/service-name=postgres` → empty → `kubectl get pods -n shop -l app=postgres` → Pending → `describe` → `FailedScheduling … pod has unbound immediate PersistentVolumeClaims` → storage layer (topic 38). **One chain of evidence**, layer by layer.

### Debug containers

```bash
kubectl debug -it api-7f9c6d4b5-2xk8q -n shop --image=busybox:1.37 --target=api -- sh   # shares the app's process namespace
kubectl debug api-7f9c6d4b5-2xk8q -n shop -it --copy-to=api-debug --container=api -- sh  # copy of the Pod, shell instead of the app
kubectl debug node/lab-worker2 -it --image=ubuntu:24.04                                   # host via /host
```

---

## Hands-on lab

Practise the method on deliberately broken setups (have a partner break them, or script it):
1. A Deployment whose image tag doesn't exist.
2. A Service with a selector typo.
3. A Pod requesting 100 CPUs.
4. A Pod mounting a missing ConfigMap key.
5. A NetworkPolicy blocking DNS.
6. A stopped kubelet on a worker (`docker exec lab-worker2 systemctl stop kubelet`).
For each: write down the symptom, the layer, the evidence command that proved it, the fix, and how you verified it. Time yourself; aim for < 5 minutes each (CKA pace).

---

## Where you'll use it

- **On-call** and **incident response**.
- **CKA/CKS** troubleshooting tasks (the largest exam domain).
- **Interviews**: "a Pod is Pending / a Service doesn't respond, walk me through it".

---

## When it breaks

The method itself fails when:
- **you skip "what changed?"**: most incidents follow a change;
- **you change several things at once**: you won't know what fixed it (or what broke it next);
- **you trust the manifest in Git rather than the live object**: check `-o yaml`;
- **you stop at "Running"**: verify the user-facing symptom;
- **you delete the evidence** (Pods, nodes) before capturing logs/describe output.

---

## Quick recap

- Define the symptom (what, since when, **what changed**, scope) → **locate the layer** along the request path → gather evidence cheapest-first (`get`, `describe` + Events, `logs --previous`, `-o yaml`) → one hypothesis at a time → smallest fix → verify the original symptom → prevent recurrence.
- Know the status → cause map and exit codes (127 command not found, 137 OOM/SIGKILL, 143 SIGTERM).
- Tools per layer: kubectl, `debug`, `auth can-i`, `crictl`, `journalctl`, `ss`, DNS/curl/nc, metrics.
- In incidents: mitigate first, communicate, keep a timeline, preserve evidence, write blameless postmortems and runbooks.

### Interview questions

1. **Q: A user says "the app is down". What do you do first?**
   A: Clarify the symptom and scope (which URL, error, since when, what changed), check the obvious health signals (Ingress/Gateway, Service endpoints, Pod status/Events, recent deploys), mitigate if users are impacted (rollback/scale/fail over), then dig through the layers for root cause.
2. **Q: A container exits with code 137. What does that tell you?**
   A: It was killed by SIGKILL (128+9): most often OOMKilled for exceeding its memory limit (check `lastState.terminated.reason`), or killed after the termination grace period.
3. **Q: Why look at `kubectl get -o yaml` rather than your manifest?**
   A: The live object shows what the cluster actually has, including defaults, mutations by admission controllers, status and conditions, which may differ from what you think you applied.

---

## Appendix: notes on the original course material

This topic was rebuilt from the original CKA course notes. This appendix keeps the audit trail out of your way while you learn.

### Sources

none: a **new topic**, giving the method that topics 82 (Day 57, control plane) and 83 (Day 58, data plane) apply. Day 58's "approach" section is already in topic 21.
