# 68 · GitOps with Argo CD and Flux

> **Level:** ★★★ advanced
>
> **You will learn:** what GitOps is (Git as the single source of truth + an in-cluster agent that continuously **pulls** and **reconciles**); push-based CI/CD vs pull-based GitOps; drift detection and self-healing; **Argo CD** (Applications, sync policies, health, sync waves, ApplicationSets) and **Flux** (GitRepository, Kustomization, HelmRelease); repo layouts for environments; promotion; secrets in GitOps; and common pitfalls.
>
> **You need:** topics 55 (ServiceAccounts), 57 (RBAC), 63 (secrets), 64 (Kustomize), 65 (Helm), 66–67 (CRDs/operators).

---

## ⚡ Core: the 10-minute version

**The idea.** **GitOps** means Git holds the desired state of everything, and an agent **inside** the cluster (Argo CD or Flux) continuously pulls from Git, applies it, and **fixes drift**:
- manual `kubectl edit` changes are reverted;
- objects deleted from Git are removed.

You change production by merging a pull request, and roll back with `git revert`. CI never needs cluster credentials.

**Picture it.** A site foreman who walks the building every few minutes with the approved blueprint, undoing unapproved changes.

**The one thing to know.**

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata: { name: guestbook, namespace: argocd }
spec:
  project: default
  source: { repoURL: https://github.com/argoproj/argocd-example-apps.git, path: kustomize-guestbook, targetRevision: HEAD }
  destination: { server: https://kubernetes.default.svc, namespace: guestbook }
  syncPolicy: { automated: { prune: true, selfHeal: true }, syncOptions: [CreateNamespace=true] }
```

**Try it now:**
1. Install Argo CD and apply this Application.
2. Scale the guestbook Deployment by hand and watch Argo CD put it back.

**You've got it if you can:** explain pull-based GitOps vs push-based CI/CD, and what prune and self-heal do.

> 📖 **That's the core.** On a first pass, do the mini-lab and move on to the next topic. Everything below is the **full reference**: come back for the details, the edge cases, interview answers and exam-level depth.

---

## What it is

**GitOps** is a way of operating Kubernetes where:
1. the **desired state** of everything (apps, add-ons, cluster config) is **declared** in **Git**;
2. Git is the **single source of truth**: changes happen by **pull request**;
3. an **agent inside the cluster** (Argo CD or Flux, both CNCF graduated) **pulls** that state and applies it;
4. the agent **continuously reconciles**: if the live cluster drifts from Git, it's detected and (optionally) **corrected**.

It's the Kubernetes controller pattern (topic 66) applied to your whole deployment process: Git holds `.spec`, the cluster is the actual state, the GitOps controller closes the gap.

---

## Why it exists

**Push-based CI/CD** (Jenkins/GitHub Actions running `kubectl apply`/`helm upgrade`):
- the CI system needs **cluster-admin-ish credentials** stored outside the cluster (a juicy target);
- if someone runs `kubectl edit` in prod, nothing notices: **drift**;
- "what's running in prod?" means reading cluster state, not a reviewed source;
- rolling back = re-running old pipelines.

**Pull-based GitOps:**
- **no inbound cluster credentials** in CI (CI only builds images and updates Git);
- every change is a **reviewed, audited commit** (who, what, when, why);
- **rollback = `git revert`**;
- drift is **visible** and **self-healed**;
- **disaster recovery**: point a fresh cluster at the repo and it rebuilds itself;
- scales to **many clusters** from one repo.

---

## Analogy

Git is the **architect's approved blueprint** in the site office. The GitOps agent is a **site foreman who walks the building every few minutes** comparing it with the blueprint. If a worker moves a wall without approval (`kubectl edit`), the foreman notices and puts it back. To change the building you don't shout at workers; you get a **revised blueprint approved** (pull request), and the foreman builds what it says.

---

## How it works inside

### The loop

```
 Developer ──PR──► Git repo (manifests: Kustomize/Helm/YAML)
                       ▲                       │
 CI: build, test,      │ commit new image tag  │  poll / webhook
 scan, push image ─────┘                       ▼
                               ┌──────── GitOps controller (in cluster) ───────┐
                               │ 1. fetch Git at a revision                    │
                               │ 2. render (kustomize build / helm template)   │
                               │ 3. diff rendered vs live objects              │
                               │ 4. apply (server-side apply) if OutOfSync     │
                               │ 5. assess health, report status               │
                               └───────────────────────────────────────────────┘
```

- **CI** builds and pushes the image, then **updates the image tag in Git** (or an image-updater tool does it automatically).
- The controller notices the new commit (polling, default ~3 min, or a Git webhook), renders and applies.

### Argo CD

- Runs in `argocd` namespace: **api-server** (UI/CLI/API), **repo-server** (clones Git and renders manifests), **application-controller** (diffs and syncs), Redis, Dex (SSO).
- Core CR: **`Application`**: source (repo, path, revision, Helm/Kustomize options) → destination (cluster, namespace) + sync policy.
- **Sync status**: `Synced` / `OutOfSync`; **health**: `Healthy` / `Progressing` / `Degraded` / `Missing` (understands Deployments, StatefulSets, Ingress, many CRDs).
- **Sync policy**: manual, or `automated` with **`prune`** (delete objects removed from Git) and **`selfHeal`** (revert manual changes).
- **Sync waves/hooks**: `argocd.argoproj.io/sync-wave: "-1"` orders things (CRDs and namespaces first, then operators, then apps); `PreSync`/`PostSync` hook Jobs (DB migrations).
- **AppProjects**: guardrails: which repos, clusters, namespaces and kinds a team's Applications may use (multi-tenancy).
- **ApplicationSet**: generates many Applications from a template + generators (list of clusters, Git directories, pull requests).
- **App of apps**: one Application that points at a folder of Application manifests (bootstrapping a whole cluster).

### Flux

- A set of controllers (the **GitOps Toolkit**) in `flux-system`, configured purely through CRs, with no UI by default:
  - **source-controller**: `GitRepository`, `OCIRepository`, `HelmRepository`, `Bucket`: fetches artifacts;
  - **kustomize-controller**: `Kustomization` (Flux's CR, not to be confused with Kustomize's file): applies a path, with `prune`, health checks, `dependsOn`;
  - **helm-controller**: `HelmRelease`: Helm installs/upgrades driven declaratively, with drift detection and remediation;
  - **notification-controller** (alerts to Slack/Teams, Git commit statuses), **image-reflector/automation controllers** (write new image tags back to Git).
- `flux bootstrap github …` installs Flux **and commits its own manifests to the repo**, so Flux manages itself.

| | Argo CD | Flux |
|---|---|---|
| UI | Rich web UI | CLI/CRs (third-party UIs exist) |
| Model | `Application` objects, central control plane, can manage many clusters | Controllers per cluster, everything as CRs |
| Multi-tenancy | AppProjects + SSO/RBAC | Kubernetes RBAC, per-tenant ServiceAccounts |
| Helm | Renders templates, applies (no Helm release record) | Real Helm releases via helm-controller |
| Image automation | Argo CD Image Updater (separate) | Built-in controllers |

Both are excellent; pick by team preference (UI and central view → Argo CD; pure-Kubernetes, composable → Flux).

### Repository layouts

```
gitops-repo/
├── apps/
│   ├── shop/
│   │   ├── base/                 # Kustomize base (topic 64)
│   │   └── overlays/{dev,stage,prod}/
│   └── payments/ …
├── infrastructure/               # ingress controller, cert-manager, monitoring (Helm via HelmRelease/Application)
│   ├── controllers/
│   └── configs/
└── clusters/
    ├── dev/                      # what THIS cluster runs: points at apps/*/overlays/dev, infrastructure
    └── prod/
```

- **Promotion** between environments = a PR changing the image tag (or version) in the next overlay; can be automated (Kargo, image updaters, CI jobs) with approvals for prod.
- Keep **app source code** and **deployment config** in separate repos (or folders) so config changes don't trigger app builds and vice versa.

### Secrets in GitOps

Never commit plaintext Secrets. Use (topic 63): **External Secrets Operator** (Git holds `ExternalSecret` references), **Sealed Secrets** (Git holds encrypted `SealedSecret`s), or **SOPS** (Flux decrypts natively; Argo CD via plugins).

### Security model

- The controller has **powerful in-cluster RBAC** (it creates everything). Protect it: SSO, AppProjects/least privilege, restrict who can merge to the GitOps repo (branch protection, CODEOWNERS), signed commits optionally verified by the controller.
- **Git becomes your control plane**: whoever can push to `main` can change prod.

---

## Commands / YAML

### Install Argo CD and deploy an app

```bash
kubectl create namespace argocd
kubectl apply -n argocd --server-side -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
kubectl -n argocd get pods
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath='{.data.password}' | base64 -d; echo
kubectl -n argocd port-forward svc/argocd-server 8080:443      # UI at https://localhost:8080 (user admin)
```

(`--server-side` avoids "annotation too long" errors with Argo CD's large CRDs. Pin a version instead of `stable` in real use.)

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: guestbook-dev
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/argoproj/argocd-example-apps.git
    targetRevision: HEAD
    path: kustomize-guestbook          # a Kustomize directory: Argo CD runs kustomize build
  destination:
    server: https://kubernetes.default.svc    # this cluster
    namespace: guestbook-dev
  syncPolicy:
    automated:
      prune: true                       # delete what's removed from Git
      selfHeal: true                    # undo manual drift
    syncOptions:
      - CreateNamespace=true
      - ServerSideApply=true
```

```bash
kubectl apply -f guestbook-app.yaml
kubectl -n argocd get applications
```

```
NAME            SYNC STATUS   HEALTH STATUS
guestbook-dev   Synced        Healthy
```

Test self-heal:

```bash
kubectl -n guestbook-dev scale deploy kustomize-guestbook-ui --replicas=5
kubectl -n argocd get application guestbook-dev -w      # OutOfSync → Synced; replicas back to Git's value
```

### Flux equivalent

```bash
flux check --pre
flux bootstrap github --owner=<you> --repository=fleet --path=clusters/dev --personal
```

```yaml
apiVersion: source.toolkit.fluxcd.io/v1
kind: GitRepository
metadata: { name: podinfo, namespace: flux-system }
spec:
  interval: 1m
  url: https://github.com/stefanprodan/podinfo
  ref: { branch: master }
---
apiVersion: kustomize.toolkit.fluxcd.io/v1
kind: Kustomization
metadata: { name: podinfo, namespace: flux-system }
spec:
  interval: 10m
  sourceRef: { kind: GitRepository, name: podinfo }
  path: ./kustomize
  prune: true
  targetNamespace: podinfo
  wait: true                      # health-check what it applied
```

```bash
flux get kustomizations
flux reconcile kustomization podinfo --with-source      # don't wait for the interval
```

### A HelmRelease (Flux)

```yaml
apiVersion: helm.toolkit.fluxcd.io/v2
kind: HelmRelease
metadata: { name: ingress-nginx, namespace: ingress-nginx }
spec:
  interval: 30m
  chart:
    spec:
      chart: ingress-nginx
      version: "4.x"
      sourceRef: { kind: HelmRepository, name: ingress-nginx, namespace: flux-system }
  values:
    controller: { replicaCount: 2 }
```

(ingress-nginx is retired: this is just a familiar chart for the example; see topics 69–72 for its successors.)

---

## Hands-on lab

1. Install Argo CD on kind; deploy the guestbook Application; open the UI and explore the resource tree.
2. Fork the example repo, point the Application at your fork, change `replicas` in Git, push, and watch it sync (or click **Refresh**).
3. Make a manual change with `kubectl` and watch self-heal revert it; disable `selfHeal` and see `OutOfSync` instead.
4. Delete a manifest from Git with `prune: true` and watch the object disappear.
5. Build a mini repo with `apps/shop/overlays/{dev,prod}` and an ApplicationSet (Git directory generator) that creates one Application per overlay.
6. (Optional) Do the same with Flux `bootstrap` in a second kind cluster.
7. Clean up: delete Applications (with the `resources-finalizer.argocd.argoproj.io` finalizer they also delete their resources), then Argo CD.

---

## Where you'll use it

- **Most modern platform teams** deploy everything (apps and add-ons) via Argo CD or Flux.
- **Multi-cluster fleets** (ApplicationSets, Flux per cluster).
- **Audits and DR**: Git history as the change log; rebuild clusters from Git.
- **Incident response**: "what changed?" = `git log`; rollback = revert.

---

## When it breaks

| Symptom | Cause | Fix |
|---|---|---|
| App permanently `OutOfSync` | Fields mutated by controllers/admission (HPA replicas, injected sidecars, defaulted fields) | Ignore those fields (`ignoreDifferences`), don't set `replicas` with an HPA, use server-side apply |
| `ComparisonError: … no matches for kind` | CRDs not installed before the CRs | Sync waves / Flux `dependsOn`; install CRDs first |
| Manual hotfix keeps getting reverted | `selfHeal` working as designed | Change Git (or pause auto-sync briefly during an incident and record it) |
| Objects deleted unexpectedly | `prune` + a manifest removed/renamed/moved | Review PRs; `Prune=false` annotation on critical objects |
| Repo-server errors: `kustomize build failed` / `helm template failed` | Broken manifests in Git | CI validation (`kustomize build`, `helm lint`, kubeconform) on PRs |
| App `Degraded` | A workload unhealthy (crash loops, failed rollout) | Debug as usual (topic 21) |
| Huge Argo CD memory/CPU | Very many apps/objects | Sharding, resource exclusions, tuning |
| Secrets showing in Git | Plaintext committed | Rotate the secret immediately; adopt ESO/Sealed Secrets/SOPS; scrub history |

---

## Quick recap

- GitOps = desired state in **Git**, applied by an in-cluster agent that **pulls** and **continuously reconciles** (drift detection, self-heal, prune).
- Benefits: no CI cluster credentials, PR-based audit trail, `git revert` rollbacks, DR and multi-cluster at scale.
- **Argo CD**: `Application`, sync/health status, automated sync with prune/selfHeal, waves/hooks, AppProjects, ApplicationSets, UI. **Flux**: `GitRepository` + `Kustomization` + `HelmRelease` controllers, bootstrap, image automation.
- Keep secrets out of Git (ESO/Sealed Secrets/SOPS); protect the repo like production.

### Interview questions

1. **Q: What's the difference between push-based CI/CD and GitOps?**
   A: Push-based pipelines run `kubectl`/`helm` from CI with cluster credentials and don't notice later drift; GitOps keeps desired state in Git and an in-cluster controller pulls and continuously reconciles it, so CI needs no cluster access, every change is a reviewed commit, and drift is detected and corrected.
2. **Q: Someone hotfixed a Deployment with `kubectl edit` and Argo CD reverted it. Why, and what should they do?**
   A: Automated sync with self-heal restores Git's state; the fix should be made via a commit/PR to the GitOps repo (or auto-sync paused deliberately during an incident, then the change committed).
3. **Q: How do you handle secrets in a GitOps workflow?**
   A: Never store plaintext; commit references or encrypted forms: External Secrets Operator pulling from a secret manager, Sealed Secrets, or SOPS-encrypted files decrypted by the controller.

---

## Appendix: notes on the original course material

This topic was rebuilt from the original CKA course notes. This appendix keeps the audit trail out of your way while you learn.

### Sources

none: a **new topic**. It builds on mentions in Day 37 (Argo CD and Flux as in-cluster GitOps controllers that poll Git and apply desired state; topic 55), Day 39 (GitOps pipelines automating DR; topic 66), Day 40 (Argo CD/Flux as operators; topic 67) and Day 42's GitOps-friendliness of Kustomize (topic 64).
