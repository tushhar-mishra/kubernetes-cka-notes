# 57 · RBAC: Roles, ClusterRoles and Bindings

> **Level:** ★★ intermediate
>
> **You will learn:** namespaced vs cluster-scoped resources; RBAC **verbs** and how kubectl commands map to them; the four RBAC objects (**Role, RoleBinding, ClusterRole, ClusterRoleBinding**) and the "ClusterRole reused through a RoleBinding" pattern; subjects (User, Group, ServiceAccount) and their `apiGroup` rules; `resourceNames`, subresources (`pods/log`, `pods/exec`) and non-resource URLs; the built-in roles (`cluster-admin`, `admin`, `edit`, `view`) and **aggregation**; how RBAC **prevents privilege escalation**; imperative RBAC commands for the exam; and how to audit who can do what.
>
> **You need:** topics 26 (namespaces), 54–56.

---

## ⚡ Core: the 10-minute version

**The idea.** **RBAC** (Role-Based Access Control) grants permissions as four objects:
- a **Role** (permissions in one namespace) or **ClusterRole** (cluster-wide, or reusable);
- a **RoleBinding** (grants a role in one namespace) or **ClusterRoleBinding** (everywhere).

Rules list API groups, resources and verbs (`get`, `list`, `watch`, `create`, `update`, `patch`, `delete`). RBAC only **adds** permissions; there's no "deny". Built-in ClusterRoles (`view`, `edit`, `admin`, `cluster-admin`) cover most needs.

**Picture it.** Key-card profiles (roles) programmed onto people's cards (bindings), for one floor or the whole building.

**The one thing to know.**

```bash
kubectl create role pod-reader --verb=get,list,watch --resource=pods -n dev
kubectl create rolebinding seema-reads --role=pod-reader --user=seema -n dev
kubectl create rolebinding ci-edit --clusterrole=edit --serviceaccount=dev:ci -n dev
kubectl auth can-i list pods -n dev --as=seema           # yes
kubectl auth can-i delete pods -n dev --as=seema         # no
```

**Try it now:**
1. Run the commands.
2. Act as Seema (topic 54's context) and confirm.
3. Note that `kubectl logs` needs `get` on the **subresource** `pods/log`.

**You've got it if you can:** choose between Role/ClusterRole and RoleBinding/ClusterRoleBinding, and grant least privilege.

> 📖 **That's the core.** On a first pass, do the mini-lab and move on to the next topic. Everything below is the **full reference**: come back for the details, the edge cases, interview answers and exam-level depth.

---

## What it is

**RBAC (Role-Based Access Control)** regulates access based on **roles**: named sets of permissions that you hand to people or workloads. In Kubernetes it's the standard authorizer (`--authorization-mode=Node,RBAC`). It answers:
- **Who?** the user (Seema), group, or ServiceAccount;
- **What?** the resource (Pods, Deployments, Secrets…);
- **Which operation?** the verb (get, list, create, delete, update…);
- **Where?** one namespace, or cluster-wide.

The four building blocks (all in API group `rbac.authorization.k8s.io/v1`):

| Object | Scope | Purpose | Used by / binds to |
|---|---|---|---|
| **Role** | Namespace | Permissions **inside one namespace** | A RoleBinding |
| **RoleBinding** | Namespace | Gives a **Role or ClusterRole** to subjects, **in its namespace** | Users / Groups / ServiceAccounts |
| **ClusterRole** | Cluster | Permissions **cluster-wide**, for cluster-scoped resources, or a **reusable template** for many namespaces | A RoleBinding or ClusterRoleBinding |
| **ClusterRoleBinding** | Cluster | Gives a ClusterRole to subjects **in every namespace** (and on cluster-scoped resources) | Users / Groups / ServiceAccounts |

**Note:** a **RoleBinding can reference a ClusterRole**: the ClusterRole's rules then apply **only in that RoleBinding's namespace**. That's how one permission set is reused across many namespaces without copying it.

---

## Why it exists

Different people and tools need different powers: a developer in `dev` shouldn't delete things in `prod`; a monitoring agent only reads; CI deploys to some namespaces only. RBAC gives **least-privilege** access as **API objects**, so permissions are versioned in Git, reviewed, audited and changed without restarting anything (unlike ABAC, topic 56).

---

## Analogy

A hotel. A **Role** is a **key-card profile** ("housekeeping: rooms on floor 3, cleaning cupboard"). A **ClusterRole** is a profile that's valid on **every floor**, or a **template** profile the hotel reuses. A **RoleBinding** is **programming a guest's card** with a profile **for one floor**; a **ClusterRoleBinding** programs it for **the whole building**. Cards can only **open** doors; there's no "deny" setting. If no profile opens a door, it stays shut.

---

## How it works inside

### Namespaced vs cluster-scoped resources

1. Resources are **namespaced** (Pods, Deployments, Services, Secrets, Roles…) or **cluster-scoped** (Nodes, PersistentVolumes, Namespaces, StorageClasses, ClusterRoles…). Check with:

   ```bash
   kubectl api-resources --namespaced=true
   kubectl api-resources --namespaced=false
   ```

2. Every resource supports certain **verbs**; see them with `kubectl api-resources -o wide`.

### RBAC verbs

| Verb | HTTP | Meaning | kubectl example |
|---|---|---|---|
| `get` | GET (one object) | Read one resource | `kubectl get pod nginx -o yaml` |
| `list` | GET (collection) | List resources of a type | `kubectl get pods` |
| `watch` | GET `?watch=true` | Stream changes | `kubectl get pods --watch` (also needs `list`) |
| `create` | POST | Create | `kubectl create deployment nginx …`, `kubectl apply` of a new object |
| `update` | PUT | Replace a whole object | `kubectl replace -f deploy.yaml` |
| `patch` | PATCH | Modify part of an object | `kubectl patch …`, `kubectl apply` on an existing object, `kubectl edit`, `kubectl scale`, `kubectl label` |
| `delete` | DELETE (one) | Remove one resource | `kubectl delete pod nginx` |
| `deletecollection` | DELETE (collection) | Remove many in one call | API `DELETE /api/v1/namespaces/dev/pods`; the namespace controller uses it |

> ✅ **Correct today** ([C1](#c1)): `kubectl apply` sends **PATCH** requests for existing objects (client-side and server-side apply both), so it needs `patch` (+ `get`, and `create` for new objects), not `update`; `update` is a full **PUT**, e.g. `kubectl replace`. And `kubectl delete pods --all` **lists** the Pods and deletes them **one by one** (needs `list` + `delete`); `deletecollection` is a single collection-DELETE API call. Always verify with `kubectl auth can-i` or by running the command with `-v=6` to see the HTTP method.

**Special verbs:** `use` (PodSecurityPolicy, historic), `bind` and `escalate` (on roles; see escalation prevention below), `impersonate` (on users/groups/serviceaccounts), `approve`/`sign` (on CSR signers). `*` means all verbs.

### Rules: what a Role contains

```yaml
rules:
- apiGroups: [""]                  # "" = core group (pods, services, secrets, configmaps, nodes…)
  resources: ["pods", "pods/log"]  # pods/log, pods/exec, pods/portforward, deployments/scale = SUBresources
  resourceNames: ["web-0"]         # optional: only these named objects
  verbs: ["get"]
- apiGroups: ["apps"]              # find a resource's group: kubectl api-resources | grep deployments
  resources: ["deployments"]
  verbs: ["get", "list", "watch"]
- nonResourceURLs: ["/metrics", "/healthz"]   # only in ClusterRoles
  verbs: ["get"]
```

- Rules are **additive**: the union of all rules from all bindings that apply to you.
- **Subresources are separate**: `pods` doesn't include `pods/exec` or `pods/log`. To allow `kubectl exec` you need `create` on `pods/exec`; `kubectl logs` needs `get` on `pods/log`.
- `resourceNames` can't restrict `list`, `watch` or `create` (the name isn't known, or there are many) in the normal way; use them for `get`, `update`, `patch`, `delete`.

### Subjects

```yaml
subjects:
- kind: User
  name: seema                              # case-sensitive: the authenticated username
  apiGroup: rbac.authorization.k8s.io
- kind: Group
  name: junior-admins                      # e.g. the O= field of certs, or an OIDC groups claim
  apiGroup: rbac.authorization.k8s.io
- kind: ServiceAccount
  name: my-service-account
  namespace: dev                           # REQUIRED for ServiceAccounts
  # apiGroup: ""                            # empty (or omitted) for ServiceAccounts
```

Useful built-in groups: `system:authenticated` (everyone logged in), `system:serviceaccounts` (all SAs), `system:serviceaccounts:<ns>` (all SAs in a namespace).

> ✅ **Correct today** ([C2](#c2)): the API server **rejects** the binding at creation with a validation error (`subjects[0].apiGroup: Unsupported value: "rbac.authorization.k8s.io": supported values: ""`), so it isn't silent. The really silent mistakes are a **wrong name or namespace** in a subject, a **typo'd username**, or a **User** subject written for a ServiceAccount (`kind: User, name: system:serviceaccount:dev:my-sa` works, but is easy to get wrong). Bindings to subjects that don't exist are accepted without complaint, since Kubernetes has no user list to check against.

### Evaluation (what happens when Seema runs a command; Day 36)

```bash
kubectl create deployment nginx --image=nginx --namespace=dev
```

1. **Authentication**: the API server confirms it's Seema (her certificate's CN; topic 54).
2. **Authorization (RBAC)**: it gathers the RoleBindings **in `dev`** and **all** ClusterRoleBindings whose subjects include `seema` or one of her groups; finds `bind-workload-editor` → Role `workload-editor`; that Role has `create` on `deployments` in group `apps` → **allow**.
3. **Admission** and then **persistence**: the Deployment is created.

`roleRef` is **immutable**: to change which role a binding points to, delete and recreate the binding (`kubectl auth reconcile` does this for you).

### Privilege-escalation prevention

You **can't grant permissions you don't have**:
- creating or updating a Role/ClusterRole with rules you don't hold is refused, unless you have the **`escalate`** verb on roles;
- creating a binding to a role whose permissions you don't hold is refused, unless you have the **`bind`** verb on that role.

So a namespace `admin` can hand out `edit` or `view` in their namespace, but can't create a binding to `cluster-admin`.

### Built-in roles and aggregation

`kubectl get clusterroles` shows many `system:` roles (for components) and four **user-facing** ones:

| ClusterRole | Use with | Grants |
|---|---|---|
| `cluster-admin` | ClusterRoleBinding | **Everything**, everywhere (`*` on `*`) |
| `admin` | RoleBinding (per namespace) | Almost everything in a namespace, **including** Roles/RoleBindings; not the namespace itself or quotas |
| `edit` | RoleBinding | Read/write most objects, **no** RBAC; can read Secrets (hence run Pods as any SA in the namespace) |
| `view` | RoleBinding | Read-only, **excluding Secrets** |

`admin`, `edit` and `view` are **aggregated**: they have an `aggregationRule` that pulls in rules from any ClusterRole labelled e.g. `rbac.authorization.k8s.io/aggregate-to-edit: "true"`. So when you install a CRD, you can make `edit` users able to manage it:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: backups-edit
  labels:
    rbac.authorization.k8s.io/aggregate-to-edit: "true"     # merged into the built-in 'edit'
rules:
- apiGroups: ["example.com"]
  resources: ["backups"]
  verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
```

---

## Commands / YAML

### Setup: `dev` as the default namespace

```bash
kubectl create ns dev
kubectl create ns prod                                   # used in the last demo
kubectl config set-context --current --namespace=dev     # no more -n dev
```

(Day 36 assumes user Seema exists with a kubeconfig context, from the Day 34 flow in topic 54.)

### Demo 1: Role + RoleBinding (namespace-scoped) (`Day 36/1-role-rb.yaml`)

**Roles are strictly namespace-scoped:**
- a Role always lives in **one** namespace;
- it can only grant access to **namespaced** resources in that namespace, never to Nodes, PVs or Namespaces.

> ✅ **Correct today** ([C3](#c3)): it defaults to your **current context's namespace** (which is `default` unless you changed it). After `set-context --namespace=dev` above, a Role without `metadata.namespace` lands in **`dev`**.

Seema, a developer in `dev`, needs to **create and list Pods and Deployments**:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: dev
  name: workload-editor
rules:
# Rule 1: Pods (core API group)
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["create", "list"]
# Rule 2: Deployments ('apps' group: Deployments, StatefulSets, DaemonSets, ReplicaSets)
- apiGroups: ["apps"]
  resources: ["deployments"]
  verbs: ["create", "list"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: bind-workload-editor
  namespace: dev
subjects:
- kind: User                              # Subject 1: user Seema
  name: seema
  apiGroup: rbac.authorization.k8s.io
- kind: Group                             # Subject 2: a group, e.g. junior admins
  name: junior-admins
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: workload-editor
  apiGroup: rbac.authorization.k8s.io
```

**Key points:**
- A Role can have **multiple rules** (here: Pods and Deployments; different API groups need separate rules or a list).
- A RoleBinding's `subjects` is a **list**: users, groups and ServiceAccounts together.
- `roleRef` names the Role being granted.
- Comment your RBAC YAML: it helps reviews.

```bash
kubectl apply -f "Day 36/1-role-rb.yaml"
kubectl auth can-i create deployments -n dev --as=seema           # yes
kubectl auth can-i delete deployments -n dev --as=seema           # no
kubectl auth can-i list pods -n dev --as=anyone --as-group=junior-admins   # yes (group)
```

### Demo 2: ClusterRole + ClusterRoleBinding (`Day 36/2-cr-crb.yaml`)

**Clarification:** ClusterRoles aren't only for cluster-scoped resources. A ClusterRole on a **namespaced** resource (Deployments), bound with a **ClusterRoleBinding**, grants that access in **every namespace**: useful for platform admins or automation needing consistent access everywhere, without copying Roles into each namespace.

Seema needs to **read Nodes**: cluster-scoped, so a Role can't do it.

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: node-reader
rules:
- apiGroups: [""]
  resources: ["nodes"]                     # cluster-scoped
  verbs: ["get", "list", "watch"]          # read-only
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: bind-node-reader
subjects:
- kind: User
  name: seema
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: node-reader
  apiGroup: rbac.authorization.k8s.io
```

**Verify:**

```bash
kubectl apply -f "Day 36/2-cr-crb.yaml"
kubectl auth can-i create pods --namespace=dev --as=seema    # yes (Demo 1)
kubectl auth can-i get nodes --as=seema                      # yes
```

### Demo 3: one ClusterRole reused in a namespace (`Day 36/3-cr-rb.yaml`)

**Extra insight:** a ClusterRole is versatile:
1. **Reusable across namespaces**: reference it from a **RoleBinding** in each namespace that needs it, with no copies.
2. **Cluster-wide access to namespaced resources**: reference it from a **ClusterRoleBinding**.

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: elevated-workload-editor
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["create", "delete", "update"]
- apiGroups: ["apps"]
  resources: ["deployments"]
  verbs: ["create", "delete", "update", "patch"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: use-clusterrole-in-prod
  namespace: prod                          # the binding (and so the permissions) apply only in prod
roleRef:
  kind: ClusterRole                        # a ClusterRole, used namespace-locally
  name: elevated-workload-editor
  apiGroup: rbac.authorization.k8s.io
subjects:
- kind: User
  name: seema
  apiGroup: rbac.authorization.k8s.io
```

Seema gets these powers **only in `prod`**. Add RoleBindings in `staging`, `test`… to reuse it.

```bash
kubectl apply -f "Day 36/3-cr-rb.yaml"
kubectl auth can-i delete deployments -n prod --as=seema   # yes
kubectl auth can-i delete deployments -n dev  --as=seema   # no
```

### Imperative RBAC (fast for the exam)

```bash
kubectl create role pod-reader --verb=get,list,watch --resource=pods -n dev
kubectl create role log-reader --verb=get --resource=pods/log -n dev
kubectl create clusterrole node-reader --verb=get,list,watch --resource=nodes
kubectl create rolebinding seema-reads --role=pod-reader --user=seema -n dev
kubectl create rolebinding ci-edit --clusterrole=edit --serviceaccount=ci:deployer -n prod
kubectl create clusterrolebinding seema-nodes --clusterrole=node-reader --user=seema
kubectl create role x --verb=get --resource=deployments.apps --dry-run=client -o yaml   # generate YAML
```

`--serviceaccount=<namespace>:<name>`. Resources with a group can be written `deployments.apps`.

### Auditing who can do what

```bash
kubectl auth can-i --list --as=seema -n dev
kubectl get rolebindings,clusterrolebindings -A -o wide | grep -w seema
kubectl describe clusterrole admin | head -20
kubectl auth reconcile -f rbac.yaml        # create/update roles and bindings safely (handles immutable roleRef)
```

```
Resources                                       Non-Resource URLs   Resource Names   Verbs
pods                                            []                  []               [create list]
deployments.apps                                []                  []               [create list]
nodes                                           []                  []               [get list watch]
selfsubjectreviews.authentication.k8s.io        []                  []               [create]
…
```

Tools like `kubectl-who-can` and `rbac-lookup` (krew plugins) answer the reverse question: "who can delete secrets in prod?".

---

## Hands-on lab

1. Set up `dev`/`prod` and the Seema context (topic 54).
2. Apply the three Day 36 demos; check each with `auth can-i` as admin, then actually run commands **as Seema** (`kubectl --context seema@kind-lab …`).
3. Prove subresources are separate: give Seema `get pods` only; `kubectl logs` fails; add `get pods/log`; it works. Same for `kubectl exec` (`create pods/exec`).
4. Try escalation: give Seema `create roles` in `dev`, then as Seema create a Role with `delete secrets` → refused ("attempting to grant RBAC permissions not currently held").
5. Bind the built-in `view` ClusterRole to Seema in `dev`; confirm she can't read Secrets.
6. Make an aggregated ClusterRole for a CRD (topic 66) and see it appear in `kubectl describe clusterrole edit`.
7. Try a ServiceAccount subject with `apiGroup: rbac.authorization.k8s.io` and read the validation error.
8. Clean up: `kubectl delete -f "Day 36/"`; `kubectl config set-context --current --namespace=default`.

---

## Where you'll use it

- **Multi-team clusters**: namespace per team, `admin`/`edit`/`view` via RoleBindings to **SSO groups**.
- **Every operator/Helm chart** installs ClusterRoles; review them before installing (look for `*`, Secrets, `escalate`, `bind`, `impersonate`, `pods/exec`).
- **CI/CD**: SA per pipeline, RoleBinding to `edit` (or narrower) in target namespaces only.
- **CKA**: very common tasks (create Role/Binding, fix a binding, `auth can-i`).

---

## When it breaks

| Symptom | Cause | Fix |
|---|---|---|
| `Forbidden: User "seema" cannot list resource "deployments" in API group "apps"` | No rule, or rule in the wrong `apiGroups` (e.g. `""` for Deployments) | Fix the group: `kubectl api-resources` |
| Works in one namespace, not another | RoleBinding is namespaced | Add a RoleBinding there, or a ClusterRoleBinding |
| `cannot get resource "pods/log"` | Subresource not granted | Add `pods/log` / `pods/exec` |
| Binding has no effect | Subject typo; wrong SA namespace; username differs (case, OIDC prefix like `oidc:`) | `kubectl auth whoami` as the user; fix the subject |
| `roleRef: Invalid value … cannot change roleRef` | `roleRef` is immutable | Delete and recreate, or `kubectl auth reconcile` |
| `attempting to grant RBAC permissions not currently held` | Escalation prevention | Get the permissions, or `bind`/`escalate` from an admin |
| A user can do far too much | Bound to `cluster-admin`, in `system:masters`, or `edit` + Secrets reading | Audit bindings; least privilege |
| `kubectl get pods -w` fails, `get pods` works | Missing `watch` | Add `watch` |

---

## Quick recap

- **Role/RoleBinding** = one namespace; **ClusterRole/ClusterRoleBinding** = cluster-wide; a **RoleBinding → ClusterRole** reuses a permission set in one namespace.
- Rules = `apiGroups` + `resources` (+ subresources, `resourceNames`) + `verbs`; RBAC is **additive**, no deny.
- Subjects: User, Group (`apiGroup: rbac.authorization.k8s.io`), ServiceAccount (`namespace` required, `apiGroup: ""`).
- Built-ins: `cluster-admin`, `admin`, `edit`, `view` (aggregated). Escalation prevention: you can't grant what you don't hold.
- Best practices: least privilege; Roles for namespace access; ClusterRoleBindings sparingly; bind to **groups** and **SAs**; reuse ClusterRoles via RoleBindings; audit regularly; test with `kubectl auth can-i` before handing over access. RBAC is about **intent and impact**, not memorising YAML.

### Interview questions

1. **Q: When would you bind a ClusterRole with a RoleBinding?**
   A: To reuse one permission set (e.g. the built-in `edit`) in several namespaces: each RoleBinding grants the ClusterRole's rules only in its own namespace.
2. **Q: A developer can `get pods` but `kubectl logs` and `kubectl exec` fail. Why?**
   A: Logs and exec are subresources (`pods/log` with `get`, `pods/exec` with `create`) that must be granted separately.
3. **Q: How does Kubernetes stop someone with permission to create RoleBindings from making themselves cluster-admin?**
   A: Escalation prevention: you can only create roles with permissions you already hold and bind roles you hold (unless you have the `escalate` or `bind` verbs).

---

## Appendix: notes on the original course material

This topic was rebuilt from the original CKA course notes. This appendix keeps the audit trail out of your way while you learn.

### Sources

Day 36 (introduction; namespaced vs cluster-scoped resources and RBAC verbs with the verb table; what is RBAC; setting `dev` as the default namespace; Role and RoleBinding: Roles are strictly namespace-scoped, Seema's `workload-editor` Role, the RoleBinding with a user and a group, the ServiceAccount subject note; what happens when Seema runs a command; key pointers; ClusterRole and ClusterRoleBinding: the clarification, `node-reader`, binding, bonus SA tip; verifying access; extra insight: ClusterRoles in namespaces with `elevated-workload-editor` and the `prod` RoleBinding; quick recap table; best practices; conclusion), files `Day 36/1-role-rb.yaml`, `Day 36/2-cr-crb.yaml`, `Day 36/3-cr-rb.yaml`. Also Day 34's `seema-role`/`seema-binding` (step 8; shown in topic 54) and Day 35's `pod-reader` example (topic 56). The same sections reappear in `TLS-In-Kubernetes-Masterclass/README.md` (Days 30–34).

### Corrections to the original notes

Each entry shows what the original said and what's true now. In the topic above, each correction appears as a ✅ **Correct today** line or a ⚠️ **Fix to the original** box, linked to its entry here.

<a id="c1"></a>**C1 · Corrected (Day 36).** the table maps `update` to "`kubectl apply -f deployment.yaml`" and `deletecollection` to "`kubectl delete pods --all`". **Now:** `kubectl apply` sends **PATCH** requests for existing objects (client-side and server-side apply both), so it needs `patch` (+ `get`, and `create` for new objects), not `update`; `update` is a full **PUT**, e.g. `kubectl replace`. And `kubectl delete pods --all` **lists** the Pods and deletes them **one by one** (needs `list` + `delete`); `deletecollection` is a single collection-DELETE API call. Always verify with `kubectl auth can-i` or by running the command with `-v=6` to see the HTTP method.

<a id="c2"></a>**C2 · Corrected (Day 36).** "using a non-empty `apiGroup` with a ServiceAccount will **silently fail** and result in no permissions". **Now:** the API server **rejects** the binding at creation with a validation error (`subjects[0].apiGroup: Unsupported value: "rbac.authorization.k8s.io": supported values: ""`), so it isn't silent. The really silent mistakes are a **wrong name or namespace** in a subject, a **typo'd username**, or a **User** subject written for a ServiceAccount (`kind: User, name: system:serviceaccount:dev:my-sa` works, but is easy to get wrong). Bindings to subjects that don't exist are accepted without complaint, since Kubernetes has no user list to check against.

<a id="c3"></a>**C3 · Corrected (Day 36).** "If you don't explicitly set a namespace, it will default to the `default` namespace". **Now:** it defaults to your **current context's namespace** (which is `default` unless you changed it). After `set-context --namespace=dev` above, a Role without `metadata.namespace` lands in **`dev`**.
