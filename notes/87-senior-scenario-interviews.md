# 87 · Senior Scenario and Design Interviews

> **Level:** ★★★ advanced
>
> **You will learn:** how senior/SRE/platform interviews differ (open-ended design and incident scenarios, trade-offs, communication), a structure for answering them, and **worked answers** to common prompts: designing a multi-tenant platform, zero-downtime deployments, multi-region and DR, securing a cluster, scaling for a traffic spike, migrating off ingress-nginx, a cluster upgrade strategy, cost reduction, and live incident questions.
>
> **You need:** the whole course, especially topics 61, 68, 72, 76–78, 80–85.

---

## ⚡ Core: the 10-minute version

**The idea.** Senior interviews give **open problems**: "design a platform for 30 teams", "survive a region outage", "secure this cluster", "the bill doubled". Answer with a structure:
1. clarify requirements;
2. give a high-level design;
3. deep-dive two or three areas;
4. cover failure modes and trade-offs;
5. explain rollout and operations.

For incidents: stabilise → scope → evidence by layer → mitigate → root cause → prevent → communicate.

**Picture it.** An architect's interview: they want to hear about the soil, the budget and the building codes before any drawing.

**The one thing to know.** Always say the **trade-off**: "shared clusters are cheaper and simpler; dedicated clusters isolate blast radius; here I'd choose…".

**Try it now:**
1. Pick one prompt and set a 30-minute timer.
2. Talk through your design out loud, recording yourself.
3. Check it against the worked answer.

**You've got it if you can:** structure any design or incident question without notes.

> 📖 **That's the core.** On a first pass, do the mini-lab and move on to the next topic. Everything below is the **full reference**: come back for the details, the edge cases, interview answers and exam-level depth.

---

## What it is

**Scenario interviews** give an open problem ("design…", "what would you do if…") and assess how you **structure** it, which **trade-offs** you see, how deep your **operational** knowledge is, and how you **communicate**. There's rarely one right answer; there are well-reasoned ones.

---

## Why it exists

Senior engineers are hired to make decisions under uncertainty and to prevent incidents, not just to know commands. Interviewers want to hear requirements-first thinking, failure modes, security, observability, cost, and how you'd roll something out safely.

---

## Analogy

Like an architect's interview: they don't ask you to name types of bricks, they ask you to design a house for a family on a hillside, and they listen for questions about budget, soil, sunlight and building codes before any drawing.

---

## How it works inside

### A structure for any design prompt (≈ 5 phases)

1. **Clarify requirements** (2–3 min): users, traffic and growth, availability target (SLO), latency, data and statefulness, compliance, team size and skills, budget, cloud/on-prem constraints.
2. **High-level design**: draw the boxes: clusters/regions, ingress/gateway, workloads, data stores, CI/CD/GitOps, observability, identity.
3. **Deep dive** on 2–3 critical areas the interviewer cares about (security, scaling, DR…).
4. **Failure modes and trade-offs**: what breaks, blast radius, alternatives you rejected and why.
5. **Rollout and operations**: migration plan, testing, observability, runbooks, cost.

### A structure for incident prompts

**Stabilise → scope → evidence by layer → hypothesis → mitigate → root cause → prevent → communicate** (topics 81, 84). Say out loud what you'd check and **why**.

---

## Commands / YAML

### Prompt 1: "Design a Kubernetes platform for 30 product teams."

- **Requirements to ask:** compliance (PCI?), tenancy isolation level, regions, team autonomy vs standards, budget.
- **Tenancy model:** namespace-per-team (soft multi-tenancy) on shared clusters per environment; separate clusters for hard isolation (PCI, untrusted code). Per namespace: RBAC from SSO groups (`admin` via RoleBinding), **ResourceQuota + LimitRange**, **PSA restricted**, default-deny **NetworkPolicy** + allow-dns, generated automatically (Kyverno generate or a namespace operator) (topics 26, 41, 57, 60, 61).
- **Guardrails:** admission policies (approved registries, signed images, probes, resources, labels), cost labels; exceptions via reviewed policy exceptions (topics 58, 63).
- **Delivery:** GitOps (Argo CD ApplicationSets; AppProjects per team), golden Helm charts/Kustomize bases (topics 64–68).
- **Traffic:** Gateway API: platform owns GatewayClass/Gateways and TLS (cert-manager, ExternalDNS); teams own HTTPRoutes in their namespaces (topic 72).
- **Observability:** shared Prometheus/Grafana/Loki/Tempo or SaaS, per-team dashboards, SLO tooling, alerts routed by `team` label (topic 80).
- **Operations:** managed control planes, node pools per workload class, Karpenter/CA, upgrade cadence with canary clusters, etcd/Velero backups (topics 76–78).
- **Trade-offs:** shared clusters = efficiency and simpler ops but noisy-neighbour and blast-radius risk; many small clusters = isolation but fleet-management overhead (mitigate with GitOps + cluster templates).

### Prompt 2: "How do you achieve zero-downtime deployments?"

- **App side:** readiness probes, graceful SIGTERM handling, idempotent startup, backward-compatible DB migrations (expand → migrate → contract) (topics 19, 22).
- **Kubernetes:** `maxUnavailable: 0`, `maxSurge`, `minReadySeconds`, PDBs, `preStop` delay to cover load-balancer deregistration, enough replicas across zones (topics 25, 76).
- **Traffic:** canary or blue/green via Gateway API weights / mesh / Argo Rollouts, automated analysis on error rate and latency with auto-rollback (topics 72, 74).
- **Verify:** synthetic checks during rollout, SLO burn alerts (topic 80).

### Prompt 3: "Design for a regional outage (multi-region / DR)."

- Clarify **RPO/RTO** per service; not everything needs active-active.
- **Tiers:** active-active (stateless + globally replicated data, global load balancing/DNS failover), active-passive warm standby (second region running scaled-down, data replicated), backup-and-restore (cheap, slower).
- **Kubernetes pieces:** identical clusters per region from IaC + GitOps (re-creatable), Velero/etcd backups replicated cross-region, images in replicated registries, secrets in a multi-region manager (topics 68, 77).
- **Data is the hard part:** managed databases with cross-region replicas, or operators supporting replication; understand consistency trade-offs.
- **Test:** regular failover drills (game days); measure actual RTO/RPO.

### Prompt 4: "How would you secure a Kubernetes cluster?"

Layered answer (4C: Cloud, Cluster, Container, Code):
- **Cloud/infra:** private API endpoint, node hardening (CIS, kube-bench), minimal IAM, IMDS restrictions (topic 70).
- **Cluster access:** OIDC/SSO, RBAC least privilege, no shared admin certs, audit logging, short-lived CI credentials (topics 54–57, 80).
- **Admission:** PSA restricted, Kyverno/Gatekeeper policies, image signature verification (topics 58, 60, 63).
- **Workloads:** securityContext hardening, dedicated ServiceAccounts with no automount, NetworkPolicies, mesh mTLS where needed (topics 55, 59, 61, 74).
- **Data:** Secrets encryption at rest (KMS), external secret managers, etcd access restricted and backups encrypted (topics 63, 77).
- **Supply chain:** scanning, SBOMs, signing, pinned digests, registry allow-lists (topics 62–63).
- **Detection/response:** runtime security (Falco), audit log alerts, incident runbooks.

### Prompt 5: "Traffic will be 10× next week (a sales event). Prepare."

- Load-test now; find the bottleneck (app, DB, cache, external APIs).
- **Pod scaling:** HPA on the right metric (RPS/queue length via custom metrics or KEDA rather than CPU alone), raised `maxReplicas`, pre-scale before the event (topic 48).
- **Node capacity:** Cluster Autoscaler/Karpenter limits, cloud quotas (instances, IPs, load balancers), warm capacity or overprovisioning Pods with low priority (topics 47, 49).
- **Dependencies:** DB connection pools, read replicas, caches, rate limits; CoreDNS scaling/NodeLocal DNSCache (topic 33).
- **Protect:** PDBs, priority classes for critical paths, rate limiting at the gateway, feature flags to shed load.
- **Freeze** risky changes; staffed war room; dashboards ready (topic 80).

### Prompt 6: "We run ingress-nginx. What now?"

- It's **retired** (no more security fixes): treat as a risk with a deadline (topic 69).
- **Inventory:** all Ingresses and annotations in use (rewrites, auth, rate limits, canaries, snippets).
- **Choose a target:** Gateway API implementation (Envoy Gateway, NGINX Gateway Fabric, Istio, Cilium, cloud controllers) or a maintained Ingress controller as a stepping stone.
- **Translate:** `ingress2gateway`, map annotations to filters/policies; snippets need redesign.
- **Run in parallel:** new Gateway with its own address; shift DNS gradually per host; monitor; then decommission (topics 72–73).

### Prompt 7: "Describe your cluster upgrade strategy."

Cadence (every minor or every other, within support), pre-checks (deprecated APIs, add-on compatibility, backups), staging/canary cluster first, control plane then node pools with surge/blue-green node pools, PDBs and readiness, automated post-checks, rollback plan (for managed: node pools can roll back; control planes can't, so test first) (topic 76).

### Prompt 8: "Our Kubernetes bill doubled. Investigate."

- **Visibility:** cost allocation by namespace/label (OpenCost/Kubecost), cloud bill breakdown (nodes, load balancers, storage, data transfer).
- **Right-size:** requests vs actual usage (VPA recommendations); bin packing (Karpenter consolidation); remove CPU limits causing over-provisioning to compensate throttling (topics 41, 49).
- **Purchase:** spot/preemptible pools for tolerant workloads; reserved/savings plans for the baseline.
- **Waste:** idle dev/test environments (kube-green, scale-to-zero), orphaned PVs/load balancers (one LB per Service → shared Gateway), log/metric volume and retention (topics 67, 70, 80).
- **Guardrails:** quotas, budgets and alerts, policy requiring cost labels.

### Live incident prompts (answer with the method)

- **"Half of the Pods in prod are CrashLoopBackOff after a config change."** Roll back the config first (mitigate); then compare the ConfigMap diff, check `logs --previous`; prevent with config validation in CI and canary rollouts of config (hash-suffixed ConfigMaps trigger controlled rollouts).
- **"Latency doubled but nothing changed."** Something always changed: check deploys, traffic mix, dependency latency (traces), node changes (autoscaler added smaller/different nodes?), CPU throttling, noisy neighbours, DNS latency (topic 84 scenario 8).
- **"A node keeps going NotReady every few hours."** Correlate with kubelet/containerd logs, memory pressure (kubelet or containerd OOM-killed?), disk/inode exhaustion, kernel messages (`dmesg`), network flaps; cordon and replace while investigating.
- **"Someone ran `kubectl delete ns payments` in prod."** Stop further damage (GitOps will re-create declared objects; check prune settings); restore data/objects from Velero (namespace) or, if needed, an etcd snapshot (whole cluster); review RBAC (who could delete namespaces?) and add protections (admission policy blocking deletes of labelled namespaces, `Prune=false`, break-glass process) (topics 57, 58, 68, 77).

### Behavioural questions with technical depth

- **"Tell me about an outage you handled."** Use STAR (Situation, Task, Action, Result) + what you learnt and changed systemically (postmortem actions).
- **"A time you disagreed on a technical decision."** Show data-driven reasoning, trade-offs, and how you committed once decided.
- **"How do you introduce a breaking platform change to many teams?"** RFC, migration guides, tooling, deprecation timeline, metrics on adoption, office hours, guardrails that warn before they block.

---

## Hands-on lab

1. Pick three prompts; set a 30-minute timer for each; sketch the design (boxes and arrows) and talk through it aloud, recording yourself.
2. Have a friend play interviewer with follow-ups: "what if the region fails?", "what does it cost?", "how do you roll it out?".
3. For one design, actually build a minimal version on kind (e.g. Prompt 1's namespace bundle with Kyverno generate policies, PSA labels, quotas, default-deny + DNS).

---

## Where you'll use it

Senior DevOps/SRE/platform interviews, architecture reviews and RFCs, and leading incident response.

---

## When it breaks

| Interview anti-pattern | Better |
|---|---|
| Jumping into tools before requirements | Clarify SLOs, scale, constraints first |
| One "perfect" answer with no trade-offs | Name alternatives and why you chose |
| Ignoring operations (upgrades, on-call, cost) | Include day-2 operations explicitly |
| Ignoring data/state in DR answers | Data replication and consistency first |
| Over-engineering (mesh + multi-region for a small app) | Match complexity to requirements |

---

## Quick recap

- Design prompts: **requirements → high-level design → deep dives → failure modes/trade-offs → rollout and operations**.
- Incident prompts: **stabilise → scope → layered evidence → mitigate → root cause → prevent → communicate**.
- Recurring themes: tenancy and guardrails, zero-downtime delivery, DR with RPO/RTO, layered security, capacity and autoscaling, migrations (ingress-nginx → Gateway API), upgrades, cost.

### Interview questions

1. **Q: How do you decide between one big shared cluster and many small clusters?**
   A: Weigh isolation and blast radius (compliance, untrusted workloads, noisy neighbours, upgrade risk) against efficiency and operational overhead; common answers are shared clusters per environment with strong namespace guardrails, plus dedicated clusters for regulated or special workloads, all managed through GitOps.
2. **Q: What's your approach to making a platform change safe for 30 teams?**
   A: Announce with an RFC and timeline, ship tooling and docs, start in audit/warn mode (e.g. policies in Audit, PSA warn), measure violations, help teams fix them, then enforce, with a documented exception process.
3. **Q: How would you measure whether your platform is doing a good job?**
   A: Platform SLOs (API availability, deployment success rate and lead time, time to provision a namespace), incident counts and MTTR, cost per workload, adoption and developer satisfaction surveys.

---

## Appendix: notes on the original course material

This topic was rebuilt from the original CKA course notes. This appendix keeps the audit trail out of your way while you learn.

### Sources

none: a **new topic** for senior-level preparation, built on the course material.
