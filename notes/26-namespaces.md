# 26 · Namespaces: Partitioning a Cluster

> **Level:** ★ foundation
>
> **You will learn:** what namespaces are and what they do **and don't** isolate; the built-in namespaces; namespaced vs cluster-scoped resources; creating, using, switching and deleting namespaces (and why deletion can hang); how Services in other namespaces are reached; and the production patterns (per team or per environment, with RBAC, ResourceQuota and NetworkPolicy).
>
> **You need:** topics 12, 14 and 25. One preview: a **Service** gives a set of Pods one stable name and IP (topic 30); its DNS name includes the namespace (topic 33).

---

## ⚡ Core: the 10-minute version

**The idea.** **Namespaces** divide one cluster into named areas (e.g. `dev`, `prod`, `team-a`).
- The same object name can exist in different namespaces.
- Quotas, permissions (RBAC), network policies and Pod security rules are applied **per namespace**.
- Namespaces **don't** isolate network traffic or nodes on their own.
- Built-in ones: `default`, `kube-system` (cluster components), `kube-public` and `kube-node-lease`.

**Picture it.** Folders on a shared drive: tidy, and each can have its own permissions, but they're still on the same disk.

**The one thing to know.**

```bash
kubectl create namespace dev
kubectl get pods -n dev
kubectl get pods -A                                  # all namespaces
kubectl config set-context --current --namespace=dev # make dev the default
# from another namespace, reach a Service as  <service>.<namespace>
```

**Try it now:**
1. Create `dev` and `qa`, each with a Deployment called `web`.
2. From a Pod in `qa`, `curl web.dev`.
3. Delete the `qa` namespace and watch everything in it disappear.

**You've got it if you can:** say what namespaces do and don't isolate, and how to reach a Service in another namespace.

> 📖 **That's the core.** On a first pass, do the mini-lab and move on to the next topic. Everything below is the **full reference**: come back for the details, the edge cases, interview answers and exam-level depth.

---

## What it is

A **namespace** is a **logical partition** inside a cluster: a named "folder" that groups and separates resources. Namespaces give you:
- **Organisation and grouping**: manage resources by team, department or project.
- **No naming conflicts**: the **same name** can exist in **different namespaces** (`web` in `dev` and `web` in `prod`).
- **Resource management**: **ResourceQuotas** and **LimitRanges** apply per namespace (topic 41).
- **Environment separation**: dev, test and prod, or separate projects.
- **A security boundary for policy**: RBAC permissions and NetworkPolicies are usually written per namespace.

Earlier (topic 12) we described a namespace as a **virtual cluster inside a physical cluster**. That's the right intuition: each namespace feels like its own small cluster, but they all share the same nodes and control plane.

> ✅ **Correct today** ([C1](#c1)): a namespace **on its own isolates names, not traffic or compute**. Pods in different namespaces **can talk to each other freely** by default, run on the **same nodes**, and share the **same kernel**. Real isolation needs **NetworkPolicies** (traffic, topic 61), **RBAC** (who can do what, topic 57), **ResourceQuotas** (fair sharing, topic 41) and **Pod Security** (topic 60), all of which are configured **per namespace**. That's why namespaces are the foundation of multi-tenancy.

### Namespaced vs cluster-scoped

Not everything lives in a namespace:

| Namespaced (most things) | Cluster-scoped |
|---|---|
| Pods, Deployments, ReplicaSets, Services, ConfigMaps, Secrets, ServiceAccounts, Roles, RoleBindings, PVCs, Jobs, Ingresses, NetworkPolicies, ResourceQuotas | Nodes, Namespaces themselves, PersistentVolumes, StorageClasses, ClusterRoles, ClusterRoleBindings, CRDs, IngressClasses, PriorityClasses, CSIDrivers |

List them: `kubectl api-resources --namespaced=true` / `--namespaced=false`.

---

## Why it exists

One cluster usually hosts many apps and teams. Without namespaces you'd get name clashes, no way to say "team A may only touch team A's things", no per-team quotas, and one huge, unreadable `kubectl get pods`.

**Why use namespaces:**
- 🛡️ **Security:** limit access with RBAC and apply **network policies** per namespace.
- 🚦 **Resource management:** CPU and memory **quotas** and **limits**.
- 🎯 **Environment management:** isolate dev, test and prod.
- 📦 **Multi-tenancy:** many applications in one cluster without conflicts.
- 🧹 **Simplification:** manage related resources together (and delete them together).

---

## Analogy

Day 14's **large house with four families**:
- Without namespaces, all families share **common spaces**: no privacy, security or organisation.
- Giving each family its **own room** improves isolation, security and organisation.

**In Kubernetes:** the **house** = the **cluster**; the **families** = **applications/workloads**; the **rooms** = **namespaces**.

The analogy's limit: these rooms have **no locks by default**. Anyone can walk into another room (network traffic) until you add locks (NetworkPolicies) and keys (RBAC).

---

## How it works inside

- A namespace is an ordinary API object (`kind: Namespace`, core `v1`, cluster-scoped).
- A namespaced object's identity is **namespace + kind + name**, and its URL includes the namespace: `/api/v1/namespaces/app1-ns/pods/web` (topic 14).
- If a manifest has no `metadata.namespace`, kubectl uses the `-n` flag or the context's default namespace, else `default`. If the manifest **does** set a namespace and you pass a **different** `-n`, kubectl refuses (`the namespace from the provided object "x" does not match the namespace "y"`).
- Every namespace automatically gets a ServiceAccount named **`default`** (topic 55), a ConfigMap **`kube-root-ca.crt`** (the cluster CA, so Pods can verify the API server), and a label **`kubernetes.io/metadata.name: <name>`** (handy for NetworkPolicy namespace selectors).
- **DNS:** each Service gets the name `<service>.<namespace>.svc.cluster.local`. A Pod's DNS search list starts with **its own** namespace, so the short name `backend-svc` only finds Services in the **same** namespace (topic 33).

### Default namespaces

| Namespace | Purpose |
|---|---|
| `default` | Lets you start using a new cluster without creating a namespace first. Avoid it for real workloads. |
| `kube-system` | Objects created by Kubernetes itself: control-plane Pods, **CoreDNS** (whose Service is still called `kube-dns`), **kube-proxy**, the CNI. |
| `kube-public` | **Readable by all clients**, even unauthenticated ones. Holds public cluster information (e.g. the `cluster-info` ConfigMap used during kubeadm joins). |
| `kube-node-lease` | One **Lease** per node: the nodes' **heartbeats**, used by the control plane to judge **node health** (topic 08). |

**kind note:** kind clusters also have **`local-path-storage`**, where the **Local Path Provisioner** runs to provide persistent storage (topic 39).

### Deleting a namespace

`kubectl delete namespace app1-ns` removes **everything inside it**: Pods, Services, ConfigMaps, Secrets, PVCs (and therefore possibly the data behind them). The namespace goes into **`Terminating`** while the namespace controller deletes its contents. It can **hang in Terminating** if some object has a **finalizer** that never completes (often a CRD whose controller was already uninstalled, or an unavailable aggregated API).

---

## Commands / YAML

### Create

```bash
kubectl create namespace app1-ns          # imperative
```

```yaml
# Day 14/app1-ns.yaml (declarative)
apiVersion: v1
kind: Namespace
metadata:
  name: app1-ns
```

```bash
kubectl apply -f "Day 14/app1-ns.yaml"
```

### View and delete

```bash
kubectl get namespaces
```

```
NAME                 STATUS   AGE
app1-ns              Active   10s
default              Active   3d
kube-node-lease      Active   3d
kube-public          Active   3d
kube-system          Active   3d
local-path-storage   Active   3d
```

```bash
kubectl delete namespace app1-ns          # ⚠️ deletes everything inside
```

### Namespace flags

| Flag | Meaning |
|---|---|
| `-n <namespace>` / `--namespace` | Run the command in that namespace |
| `-A` / `--all-namespaces` | Run across **all** namespaces |

```bash
kubectl get pods -n app1-ns
kubectl get services -A
```

### Deploy the frontend and backend into a namespace

The files `Day 14/backend-deploy.yaml` and `frontend-deploy.yaml` contain a Deployment + Service each (http-echo backend on port 5678, exposed by `backend-svc` on 9090; nginx frontend exposed by NodePort `frontend-svc` on 31000).

```bash
kubectl apply -f "Day 14/frontend-deploy.yaml" -n app1-ns
kubectl apply -f "Day 14/backend-deploy.yaml" -n app1-ns
kubectl get all -n app1-ns
```

```
NAME                                   READY   STATUS    RESTARTS   AGE
pod/backend-deploy-7d9c8f5b6-4xkqz     1/1     Running   0          20s
...
NAME                   TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
service/backend-svc    ClusterIP   10.96.41.120   <none>        9090/TCP       20s
service/frontend-svc   NodePort    10.96.200.15   <none>        80:31000/TCP   21s
NAME                             READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/backend-deploy   3/3     3            3           20s
deployment.apps/frontend-deploy  3/3     3            3           21s
```

- `frontend-deploy.yaml` already sets `namespace: app1-ns` in its metadata; `backend-deploy.yaml` doesn't, so `-n app1-ns` decides where it goes.
- `kubectl get all` shows common types only (not ConfigMaps, Secrets, PVCs, Ingresses…).

### Test cross-namespace access

From a test Pod in **`default`**:

```bash
kubectl run test-pod --image=busybox:1.37 -it --rm --restart=Never -- /bin/sh
# inside:
wget -qO- backend-svc:9090                 # ❌ fails: bad address 'backend-svc'
wget -qO- http://backend-svc.app1-ns:9090  # ✅ Hello from Backend
```

**Format:** `http://<service-name>.<namespace-name>:<service-port>`
- `<service-name>` = the Service's name, e.g. `backend-svc`;
- `<namespace-name>` = the namespace it lives in, e.g. `app1-ns`;
- `<service-port>` = the Service (ClusterIP) **port**, e.g. `9090`.

From a test Pod **in `app1-ns`**:

```bash
kubectl run test-pod -n app1-ns --image=busybox:1.37 -it --rm --restart=Never -- /bin/sh
wget -qO- backend-svc:9090                 # ✅ works: same namespace
```

> ⚠️ **Fix to the original** ([C2](#c2)): the original used `curl` inside **busybox**, but busybox has **no `curl`**. Use `wget -qO- <url>` in busybox, or the `curlimages/curl` image.
>
> And the short name failing from `default` is **not network isolation**. It fails because the Pod's **DNS search list** starts with its own namespace (`default.svc.cluster.local`), so `backend-svc` is looked up as `backend-svc.default.svc.cluster.local`, which doesn't exist. The **network** path is wide open, which is why the fully qualified name works instantly. Topic 33 explains the search list; topic 61 shows how to actually block cross-namespace traffic.

### Set a default namespace for your context

Typing `-n app1-ns` on every command is tedious:

```bash
kubectl config set-context --current --namespace=app1-ns
kubectl config get-contexts           # NAMESPACE column shows app1-ns
```

Now `kubectl get pods` means `-n app1-ns` (topic 12).

### Useful extras

```bash
kubectl get ns app1-ns -o yaml                   # labels, status, finalizers
kubectl api-resources --verbs=list --namespaced -o name \
  | xargs -n1 kubectl get --show-kind --ignore-not-found -n app1-ns   # EVERYTHING in a namespace
kubectl create ns team-a --dry-run=client -o yaml > team-a.yaml
kubectl label ns team-a pod-security.kubernetes.io/enforce=restricted   # namespace-wide policy (topic 60)
```

---

## Hands-on lab

1. Create `app1-ns`, deploy the Day 14 frontend and backend into it, and run the cross-namespace test (fail with the short name, succeed with `.app1-ns`).
2. Inside a test Pod, `cat /etc/resolv.conf` in `default` and in `app1-ns`, and compare the `search` lines.
3. Create a Deployment named `web` in both `dev` and `prod`: the same name works in different namespaces.
4. Set your context's default namespace to `app1-ns`, run `kubectl get pods`, then set it back to `default`.
5. Delete `app1-ns` and watch everything inside disappear: `kubectl get ns app1-ns -w`.
6. (Advanced) Make a namespace hang: create a ConfigMap with a fake finalizer, `kubectl patch cm x -n stuck -p '{"metadata":{"finalizers":["example.com/never"]}}'`, delete the namespace, and see it stay `Terminating`. Fix it by removing the finalizer from the **ConfigMap** (`kubectl patch cm x -n stuck --type=merge -p '{"metadata":{"finalizers":null}}'`).

---

## Where you'll use it

- **Per team or per app** namespaces with RBAC RoleBindings, ResourceQuotas, LimitRanges, NetworkPolicies and Pod Security labels: the standard multi-tenant platform pattern.
- **Per environment** (dev/stage/prod) in smaller setups; larger companies usually use **separate clusters** for prod.
- **Helm and GitOps** install each app into its own namespace (topics 65, 68).
- **CKA**: tasks always say which namespace to use; forgetting `-n` is a classic mistake.

---

## When it breaks

| Symptom | Cause | Fix |
|---|---|---|
| `Error from server (NotFound): pods "x" not found` though it exists | Wrong namespace | `-n <ns>`, or find it with `kubectl get pods -A \| grep x` |
| Short Service name doesn't resolve | Service is in another namespace | Use `<svc>.<ns>` (or the full `…svc.cluster.local`) |
| Namespace stuck `Terminating` | A resource with an unfinished **finalizer**, or an unavailable API service | `kubectl get ns <ns> -o yaml` → `status.conditions` names the blocker; fix the controller or remove the stuck finalizer |
| `namespaces "x" not found` on apply | Namespace not created yet | Create it first, or put the Namespace object at the top of the file |
| `the namespace from the provided object does not match` | Manifest has `namespace:` and you passed a different `-n` | Make them agree |
| Data lost after deleting a namespace | PVCs deleted with it; `reclaimPolicy: Delete` removed the volumes (topic 38) | Back up; use `Retain` for important data |

---

## Quick recap

- Namespaces partition **names and policy**: same names in different namespaces; quotas, RBAC, NetworkPolicy and Pod Security per namespace.
- They do **not** isolate network traffic or nodes by themselves.
- Built-in: `default`, `kube-system`, `kube-public`, `kube-node-lease` (+ `local-path-storage` on kind).
- Cross-namespace Service access: `<svc>.<ns>[:port]`; the short name works only inside the same namespace (DNS search list).
- Deleting a namespace deletes everything in it; stuck `Terminating` = finalizers.

**Best practices:** segregate dev/test/prod; use namespaces for multi-tenancy (one per team or project); set **resource quotas** for CPU, memory and storage; use clear naming conventions (e.g. `team-app-env` → `frontend-prod-ns`); delete namespaces **carefully**; secure them with **network policies**.

### Interview questions

1. **Q: Do namespaces isolate network traffic?**
   A: No. By default Pods in any namespace can reach each other; you need NetworkPolicies (and a CNI that enforces them). Namespaces isolate names and are the scope for RBAC, quotas and policies.
2. **Q: Name some cluster-scoped resources.**
   A: Nodes, Namespaces, PersistentVolumes, StorageClasses, ClusterRoles/ClusterRoleBindings, CRDs, PriorityClasses, IngressClasses.
3. **Q: A namespace is stuck in Terminating. What do you do?**
   A: Check `kubectl get ns <ns> -o yaml` conditions to find which resources or finalizers block deletion (often an orphaned CRD or an unavailable API service), fix or remove that controller's finalizers, and only as a last resort remove finalizers manually.

---

## Appendix: notes on the original course material

This topic was rebuilt from the original CKA course notes. This appendix keeps the audit trail out of your way while you learn.

### Sources

Day 14 (what namespaces are, the house analogy, why use them, default namespaces, creating/viewing/deleting, namespace flags, deploying frontend and backend into a namespace, testing isolation, cross-namespace format, setting a default namespace, best practices, summary), files `Day 14/app1-ns.yaml`, `backend-deploy.yaml`, `frontend-deploy.yaml`.

### Corrections to the original notes

Each entry shows what the original said and what's true now. In the topic above, each correction appears as a ✅ **Correct today** line or a ⚠️ **Fix to the original** box, linked to its entry here.

<a id="c1"></a>**C1 · Corrected (Day 14).** "Isolation & Security: separate workloads to prevent unwanted interactions" can be read as "namespaces isolate workloads". **Now:** a namespace **on its own isolates names, not traffic or compute**. Pods in different namespaces **can talk to each other freely** by default, run on the **same nodes**, and share the **same kernel**. Real isolation needs **NetworkPolicies** (traffic, topic 61), **RBAC** (who can do what, topic 57), **ResourceQuotas** (fair sharing, topic 41) and **Pod Security** (topic 60), all of which are configured **per namespace**. That's why namespaces are the foundation of multi-tenancy.

<a id="c2"></a>**C2 · Corrected (Day 14).** the original used `curl` inside **busybox**, but busybox has **no `curl`**. Use `wget -qO- <url>` in busybox, or the `curlimages/curl` image.

And the short name failing from `default` is **not network isolation**. It fails because the Pod's **DNS search list** starts with its own namespace (`default.svc.cluster.local`), so `backend-svc` is looked up as `backend-svc.default.svc.cluster.local`, which doesn't exist. The **network** path is wide open, which is why the fully qualified name works instantly. Topic 33 explains the search list; topic 61 shows how to actually block cross-namespace traffic.
