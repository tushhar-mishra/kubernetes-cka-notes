# 88 · CKA Exam Strategy

> **Level:** ★★ intermediate
>
> **You will learn:** what the CKA (Certified Kubernetes Administrator) exam is like (format, environment, domains and weights), how to set up your terminal in the first minutes, time management and question triage, how to use the allowed documentation, the commands that save the most time, and a study plan mapped to this course.
>
> **You need:** the whole course; especially the labs in topics 11–18, 24–28, 30–33, 38–47, 54–57, 61, 69–77, 81–83.

---

## ⚡ Core: the 10-minute version

**The idea.** The **CKA** is a hands-on exam: about 2 hours, about 15–20 tasks on real clusters, about 66% to pass. Troubleshooting is the biggest domain (~30%). The marks go to **speed and accuracy**:
- set up aliases in the first minutes;
- **switch to the right context for every task**;
- generate YAML instead of typing it;
- verify every task;
- flag stuck tasks and move on.

**Picture it.** A kitchen rush: prep your station first, cook the quick dishes, don't burn time on one soufflé, and check every plate.

**The one thing to know.**

```bash
alias k=kubectl; export do="--dry-run=client -o yaml"
k config use-context <the-one-the-task-names>        # EVERY task
k create deploy web --image=nginx --replicas=3 $do > web.yaml
```

**Try it now:** do 15 tasks from this course's labs at 7 minutes each, then take the killer.sh simulator sessions before the exam.

**You've got it if you can:** finish a timed mock with time left to re-check.

> 📖 **That's the core.** On a first pass, do the mini-lab and move on to the next topic. Everything below is the **full reference**: come back for the details, the edge cases, interview answers and exam-level depth.

---

## What it is

The **CKA** is a **performance-based** exam: no multiple choice. You solve tasks on real clusters from a command line, in a remote desktop environment, within a time limit (about **2 hours**, with roughly **15–20 tasks** of different weights). You need about **66%** to pass. The exam Kubernetes version tracks recent releases.

**Domains and approximate weights** (current curriculum; verify before your exam):

| Domain | Weight | Course topics |
|---|---|---|
| **Troubleshooting** | ~30% | 19–22, 29–33, 79, 81–84 |
| **Cluster Architecture, Installation & Configuration** | ~25% | 06–12, 42, 54–58, 64–67, 75–78 |
| **Services & Networking** | ~20% | 29–33, 61, 69–73 |
| **Workloads & Scheduling** | ~15% | 18, 24–28, 34–35, 41–49 |
| **Storage** | ~10% | 36–40 |

(The recent curriculum explicitly includes **Helm**, **Kustomize**, **CRDs/operators**, **Gateway API**, and **extension interfaces** (CNI/CSI/CRI), all covered here.)

---

## Why it exists

The CKA validates hands-on admin skills employers trust. Most candidates who fail know the content but run out of **time**, get stuck on one task, or make context/namespace mistakes. Strategy is worth points.

---

## Analogy

An exam like a **kitchen service rush**: you know the recipes (Kubernetes); passing depends on **mise en place** (terminal setup, aliases), cooking the quick dishes first (easy tasks), not burning time on one stubborn soufflé (flag and move on), and checking each plate before it leaves (verify).

---

## How it works inside

### The environment

- A remote Linux desktop with a terminal and a browser for the **allowed docs** (kubernetes.io docs, blog, and a few others listed by the exam; check the current list).
- **Several clusters**: each task tells you which **context** to use (`kubectl config use-context …`) and sometimes to `ssh` to a node. **Running a task's commands on the wrong cluster = zero points.**
- You'll often need `sudo -i` on nodes for kubeadm/etcd/kubelet tasks.
- Copy/paste works differently in the remote desktop (often Ctrl+Shift+C/V in the terminal).

### First 2 minutes: set up

```bash
alias k=kubectl                         # usually pre-configured; check with: type k
export do="--dry-run=client -o yaml"    # k create deploy x --image=nginx $do > x.yaml
export now="--force --grace-period=0"   # k delete pod x $now
source <(kubectl completion bash); complete -o default -F __start_kubectl k
```

`~/.vimrc` (for clean YAML editing):

```vim
set expandtab tabstop=2 shiftwidth=2 autoindent
```

### Per-task routine

1. **Read the whole task**; note the **context**, **namespace**, names, and exact requirements (port numbers, labels, file paths for output).
2. **Switch context** with the given command (copy it).
3. **Generate, don't type, YAML**: `k create … $do > file.yaml`, edit, `k apply -f`.
4. **Verify** (`k get`, `describe`, `curl` from a test Pod, `auth can-i`).
5. If stuck > ~5–7 minutes: **flag it** and move on; come back at the end.

### Time management

- ~120 minutes / ~17 tasks ≈ 7 min each; weights vary (4%–13%).
- **First pass**: do everything quick and sure. **Second pass**: flagged tasks by weight.
- Don't gold-plate: meet the stated requirements exactly; extra features don't score.

### Using the docs efficiently

- Search kubernetes.io for the **task page** with a ready YAML: e.g. "PersistentVolume", "NetworkPolicy", "Ingress", "etcd backup" (Operating etcd clusters), "kubeadm upgrade", "static pod", "pod security admission", "Gateway API".
- `kubectl explain <kind>.<field> --recursive` is often faster than the docs for field names.
- Bookmark nothing beforehand (not allowed), but **know which page** has what.

---

## Commands / YAML

### Generators (the biggest time savers)

```bash
k run nginx --image=nginx $do                                     # Pod
k run tmp --rm -it --image=busybox:1.37 --restart=Never -- sh     # throwaway test Pod
k create deploy web --image=nginx --replicas=3 $do
k expose deploy web --port=80 --target-port=8080 --type=NodePort $do
k create svc clusterip db --tcp=5432:5432 $do
k create cm app --from-literal=MODE=prod --from-file=app.conf $do
k create secret generic db --from-literal=pass=s3cr3t $do
k create job once --image=busybox -- sh -c 'echo hi' $do
k create cronjob nightly --image=busybox --schedule="0 2 * * *" -- sh -c 'date' $do
k create sa app-sa
k create role pod-reader --verb=get,list,watch --resource=pods $do
k create rolebinding rb --role=pod-reader --serviceaccount=default:app-sa $do
k create clusterrole node-reader --verb=get,list --resource=nodes $do
k create ingress web --rule="shop.example.com/api*=api:80" --class=nginx $do
k create quota q --hard=pods=10,requests.cpu=4 $do
k create priorityclass high --value=100000 $do
```

### Editing and inspecting fast

```bash
k get pods -A -o wide --sort-by=.metadata.creationTimestamp
k get pod x -o yaml > x.yaml                          # then edit and replace:
k replace --force -f x.yaml                           # recreate a Pod with changed immutable fields
k set image deploy/web nginx=nginx:1.27
k scale deploy web --replicas=5
k rollout status|history|undo deploy/web
k label node n1 disktype=ssd; k taint node n1 key=val:NoSchedule; k taint node n1 key=val:NoSchedule-
k get pods -l app=web -o jsonpath='{.items[*].metadata.name}'
k get nodes -o custom-columns=NAME:.metadata.name,TAINTS:.spec.taints
k auth can-i create deploy -n dev --as=system:serviceaccount:dev:app-sa
k top pods -A --sort-by=cpu
k get events -A --sort-by=.lastTimestamp | tail
```

### Classic task templates

**etcd backup/restore** (topic 77):

```bash
ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key snapshot save /opt/backup.db
etcdutl snapshot restore /opt/backup.db --data-dir /var/lib/etcd-restore     # then edit etcd.yaml hostPath
```

**Upgrade a node** (topic 76): repo → `apt-get install kubeadm=X` → `kubeadm upgrade apply vX` (first CP) / `kubeadm upgrade node` → `kubectl drain` → kubelet/kubectl → `systemctl daemon-reload && systemctl restart kubelet` → `kubectl uncordon`.

**Join a node / fix a kubelet** (topics 75, 83): `kubeadm token create --print-join-command`; `systemctl status kubelet`, `journalctl -u kubelet`, fix config, `systemctl restart kubelet`.

**NetworkPolicy** (topic 61): copy the docs example; mind AND vs OR; include DNS for egress.

**Static Pod** (topic 42): write the manifest into the kubelet's `staticPodPath` (check `/var/lib/kubelet/config.yaml`) on the **right node**.

**PV/PVC/Pod** (topic 38): match `storageClassName`, access modes, size; verify `Bound`.

**Sidecar/log container** (topic 23): shared `emptyDir`, second container `tail -f`.

**HPA** (topic 48): `k autoscale deploy web --min=2 --max=5 --cpu-percent=70`.

**Helm** (topic 65): `helm repo add`, `helm install -n ns --create-namespace -f values.yaml`, `helm upgrade`, `helm list -A`, `helm rollback`.

**Gateway API** (topics 72–73): Gateway + HTTPRoute with `parentRefs`, `backendRefs`, path matches; check `status`.

**Troubleshooting** (topics 81–83): `describe`, Events, `logs --previous`, `crictl`, static Pod manifests, kubelet service, Service selectors and EndpointSlices.

### Verification snippets

```bash
k run t --rm -it --image=busybox:1.37 --restart=Never -- wget -qO- -T 3 http://web.shop:80
k run t --rm -it --image=busybox:1.37 --restart=Never -- nslookup web.shop
k get endpointslices -l kubernetes.io/service-name=web -n shop
k get pvc -A; k get pv
k get --raw='/readyz?verbose' | tail -3
```

### Study plan (6–8 weeks, using this course)

| Week | Topics | Practice |
|---|---|---|
| 1 | 01–17 (foundations, architecture, kubectl, YAML, API) | kind cluster; generators; `explain`; jsonpath |
| 2 | 18–28 (workloads) | Write every workload kind from memory; rollouts; Jobs |
| 3 | 29–35 (networking, Services, DNS, config) | Service/DNS debugging drills |
| 4 | 36–49 (storage, scheduling, scaling) | PV/PVC, taints/affinity, quotas, HPA |
| 5 | 50–63 (security) | CSR user flow, RBAC tasks, NetworkPolicies, PSA |
| 6 | 64–74 (Kustomize, Helm, CRDs, Ingress, Gateway API) | Helm install/upgrade, Gateway + routes |
| 7 | 75–80 (kubeadm, upgrades, etcd, HA, monitoring) | Build a VM cluster, upgrade it, etcd backup/restore |
| 8 | 81–89 + mock exams | Timed mocks (e.g. the **killer.sh** simulator included with registration), redo every miss |

---

## Hands-on lab

1. Set up the alias/vim/env vars routine until it takes < 1 minute.
2. Do a **timed** set of 15 tasks from the labs in this course (one per domain area), 7 minutes each; record which ran over.
3. Practise the context switch: create 3 kind clusters and always run `kubectl config use-context` before each task.
4. Do the etcd backup/restore and kubeadm upgrade on a VM cluster at least twice each.
5. Take both killer.sh sessions in the last two weeks; review every explanation.

---

## Where you'll use it

The CKA itself, and the same speed habits make you faster in real operations and in CKAD/CKS.

---

## When it breaks

| Exam mistake | Prevention |
|---|---|
| Working in the wrong context/namespace | Paste the task's `use-context` command every time; use `-n` explicitly |
| Typing YAML by hand | Generators with `$do`; docs examples |
| Spending 20 minutes on one task | Flag after ~7 minutes, return later |
| Not verifying | Always `get`/`describe`/test after applying |
| Output written to the wrong path/format | Re-read the task's exact file path and format |
| Forgetting `sudo` or which node to ssh to | Read the task header; `exit` back to the base terminal afterwards |
| YAML indentation errors | vim settings; `k apply --dry-run=server -f` to validate |

---

## Quick recap

- Performance-based, ~2 h, ~15–20 weighted tasks, ~66% to pass; **Troubleshooting ~30%** is the biggest domain.
- Set up `alias k`, `$do`, completion and vim in the first minutes; **switch context every task**.
- Generate YAML, copy docs examples, verify every task, flag and move on when stuck.
- Master: etcd backup/restore, kubeadm upgrade/join, static Pods, RBAC/CSR, NetworkPolicy, PV/PVC, Services/DNS debugging, Helm, Gateway API, node/kubelet troubleshooting.

### Interview questions

1. **Q: What's the biggest CKA domain and how did you prepare for it?**
   A: Troubleshooting (~30%); practise breaking and fixing clusters layer by layer: control-plane static Pods, kubelet and nodes, Services/DNS, storage and application failures, under time pressure.
2. **Q: How do you avoid context mistakes in the exam?**
   A: Run the provided `kubectl config use-context` command at the start of every task, use explicit `-n` namespaces, and check with `kubectl config current-context` when switching back from ssh sessions.
3. **Q: How do you produce manifests quickly?**
   A: Imperative generators with `--dry-run=client -o yaml`, edit the generated file, apply it; use `kubectl explain` and the docs examples for fields generators don't cover.

---

## Appendix: notes on the original course material

This topic was rebuilt from the original CKA course notes. This appendix keeps the audit trail out of your way while you learn.

### Sources

none: a **new topic**. The original course is a CKA course; its days are mapped to these domains below. Always confirm current details (curriculum version, duration, allowed resources) on the Linux Foundation / CNCF exam pages before booking: they change periodically.
