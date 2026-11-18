# 85 · Production Readiness Checklist

> **Level:** ★★★ advanced
>
> **You will learn:** a practical checklist, with the reason for each item and the topic that explains it, for deciding whether a **workload** and a **cluster** are ready for production: reliability, resources, security, networking, storage, observability, delivery, operations and cost. Use it for launch reviews, platform audits and interview answers.
>
> **You need:** the whole course; each item links to its topic.

---

## ⚡ Core: the 10-minute version

**The idea.** Before going live, check the **workload**:
- 2+ replicas across zones, with a PDB;
- readiness, liveness and startup probes, and graceful shutdown;
- measured requests and limits;
- a restricted securityContext and a dedicated ServiceAccount;
- signed images pinned by digest, and Secrets from a manager;
- NetworkPolicies;
- metrics, logs and traces, with SLO alerts and runbooks;
- tested backups.

Then check the **platform**:
- HA control plane, supported version and an upgrade plan;
- etcd backups;
- Pod Security, admission policies and encryption at rest;
- audit logs, observability, quotas and DR drills.

Automate these checks with policies and linters, so nobody has to remember them.

**Picture it.** A pilot's pre-flight checklist: experts still run it every single time.

**The one thing to know.**

```bash
kubectl get deploy -A -o json | jq -r '.items[] | select(any(.spec.template.spec.containers[]; .readinessProbe == null)) | "\(.metadata.namespace)/\(.metadata.name)"'
kubectl get ns -L pod-security.kubernetes.io/enforce
```

**Try it now:**
1. Run the checks in this topic against your cluster.
2. Fix the three most important gaps.

**You've got it if you can:** list the workload and platform essentials from memory.

> 📖 **That's the core.** On a first pass, do the mini-lab and move on to the next topic. Everything below is the **full reference**: come back for the details, the edge cases, interview answers and exam-level depth.

---

## What it is

A **production readiness review (PRR)** is a structured check that a service (or the platform it runs on) can be **operated safely**: it survives failures, scales, is secure, observable and recoverable, and someone knows how to run it at 3 a.m.

---

## Why it exists

Most outages come from gaps that were known best practice: no readiness probe, one replica, no limits, no backups, no alerts, privileged containers. A checklist turns lessons learnt into a habit, so nothing depends on someone remembering.

---

## Analogy

A **pre-flight checklist**: experienced pilots still run it every time, because skipping one item (fuel, flaps) is how good pilots crash.

---

## How it works inside

Score each item **✅ done / ⚠️ accepted risk (with owner and reason) / ❌ missing**. Not every item applies to every workload; the point is to decide consciously.

---

## Commands / YAML

### A. Workload checklist

**Reliability and availability**
- [ ] **≥ 2 replicas** (stateless) spread across nodes **and zones**: `topologySpreadConstraints` or pod anti-affinity (topics 25, 46).
- [ ] **PodDisruptionBudget** that still allows drains (e.g. `maxUnavailable: 1`) (topic 76).
- [ ] **Readiness probe** (traffic only when ready), **liveness probe** only for real deadlocks, **startupProbe** for slow starters (topic 22).
- [ ] **Graceful shutdown**: handles SIGTERM, `preStop` delay if behind load balancers, suitable `terminationGracePeriodSeconds` (topic 19).
- [ ] **Rollout strategy**: `maxUnavailable: 0` / sensible `maxSurge`, `minReadySeconds`, `revisionHistoryLimit`; rollback tested (topic 25).
- [ ] Dependencies fail gracefully (timeouts, retries with backoff, circuit breakers) (topic 74).
- [ ] Stateful parts: StatefulSet/operator with tested backup **and restore** (topics 40, 67, 77).

**Resources and scaling**
- [ ] **CPU and memory requests** set from measurements; **memory limit** set; CPU limit considered (throttling vs noisy neighbours) (topic 41).
- [ ] QoS class understood (Guaranteed for critical, Burstable typical) (topic 41).
- [ ] **ephemeral-storage** requests/limits if writing temp data (topic 41).
- [ ] **HPA** (or KEDA) with sane min/max; or VPA recommendations reviewed (topics 48–49).
- [ ] **PriorityClass** for critical services (topic 47).

**Security**
- [ ] **Non-root**, `allowPrivilegeEscalation: false`, `capabilities: drop: [ALL]`, `readOnlyRootFilesystem: true`, `seccompProfile: RuntimeDefault`: passes **Pod Security "restricted"** (topics 59–60).
- [ ] **Dedicated ServiceAccount**, `automountServiceAccountToken: false` unless it calls the API; least-privilege RBAC (topics 55, 57).
- [ ] Images from the **trusted registry**, pinned by **digest** or immutable tag, **scanned**, **signed** (topics 20, 62, 63).
- [ ] **Secrets** from a secret manager (ESO/CSI/Vault), mounted as files, never in Git or images (topics 35, 63).
- [ ] **NetworkPolicy**: default-deny + explicit ingress/egress (incl. DNS) (topic 61).
- [ ] Cloud access via **workload identity**, not static keys (topics 55, 70).

**Configuration**
- [ ] Config in ConfigMaps/Secrets, versioned; changes trigger rollouts (hash suffix or checksum annotation) (topics 34, 64, 65).
- [ ] Manifests in **Git**, deployed by GitOps/CI; no `kubectl edit` in prod (topic 68).
- [ ] Recommended labels (`app.kubernetes.io/name`, `instance`, `version`, `part-of`, `team`) (topic 17).

**Observability**
- [ ] Logs to **stdout/stderr**, structured, with trace IDs (topics 79–80).
- [ ] **Metrics** exposed (RED) and scraped (ServiceMonitor) (topic 80).
- [ ] **Traces** (OpenTelemetry) for multi-service paths (topic 80).
- [ ] **SLOs** defined; **alerts** on burn rate; every alert has a **runbook**; dashboard exists (topic 80).

**Networking and exposure**
- [ ] Exposed via **Gateway API/Ingress** with **TLS** (cert-manager/ACM), HTTP→HTTPS redirect (topics 69–73).
- [ ] Timeouts and body-size limits set at the edge; WAF/rate limits where public.
- [ ] Service ports named, `appProtocol` set if a mesh is used (topic 74).

**Operations**
- [ ] On-call owner, escalation path, runbooks; documented dependencies.
- [ ] Load-tested at expected peak + margin; failure tested (kill Pods, drain nodes).
- [ ] Data backup/restore **tested**; RPO/RTO agreed (topic 77).

### B. Cluster / platform checklist

**Control plane and versions**
- [ ] HA control plane (or managed), 3 zones, etcd quorum (topic 78).
- [ ] Supported Kubernetes version; **upgrade plan** at least yearly (certificates renew); add-on compatibility matrix (topic 76).
- [ ] **etcd backups** scheduled, encrypted, off-cluster, **restore drill** done; PKI and encryption keys backed up (topic 77).
- [ ] Certificate expiry monitored (topic 82).

**Nodes**
- [ ] Node pools across zones; autoscaling (Cluster Autoscaler/Karpenter) with sane limits (topics 49, 78).
- [ ] OS patching/image rotation process; containerd/kubelet versions aligned; `SystemdCgroup` (topic 75).
- [ ] System reservations (`kube-reserved`/`system-reserved`), eviction thresholds tuned (topic 47).
- [ ] Node problem detection and alerting on node conditions.

**Security**
- [ ] **RBAC** least privilege; no humans with `cluster-admin` day-to-day; SSO/OIDC for people; no long-lived client certs (topics 54, 57).
- [ ] **Pod Security Admission** labels on every namespace (restricted where possible) (topic 60).
- [ ] **Admission policies** (Kyverno/Gatekeeper/VAP): registries, labels, probes, resources, no `latest` (topic 58).
- [ ] **Secrets encryption at rest** (KMS) (topic 63).
- [ ] **Audit logging** enabled and shipped (topic 80).
- [ ] API server not publicly exposed (or restricted by IP/private endpoint); kubelet anonymous auth off (topics 53–54).
- [ ] Image scanning in CI and in-cluster; signature verification (topic 63).
- [ ] Network: default-deny per namespace + allow-dns generated automatically; egress controls (topic 61).

**Networking**
- [ ] CNI supports NetworkPolicy; Pod/Service CIDRs planned (no overlaps), IP capacity per node/subnet (topics 29, 75).
- [ ] Ingress/Gateway controller HA, maintained (not retired ingress-nginx), TLS automation (cert-manager), ExternalDNS (topics 69–73).
- [ ] CoreDNS scaled, NodeLocal DNSCache for large clusters (topic 33).

**Storage**
- [ ] Default StorageClass with `WaitForFirstConsumer`, appropriate `reclaimPolicy`, expansion enabled (topic 39).
- [ ] CSI drivers HA; volume snapshots/backups (Velero) (topics 39, 77).

**Observability**
- [ ] Metrics stack (Prometheus/managed) with long-term storage; control-plane, etcd, node, CoreDNS dashboards and alerts (topic 80).
- [ ] Central logs (including Events and audit logs) with retention (topics 79–80).
- [ ] Tracing pipeline available to teams (topic 80).

**Multi-tenancy and governance**
- [ ] Namespace per team/app with **ResourceQuota** + **LimitRange** (topics 26, 41).
- [ ] Tenancy model documented (who gets what RBAC, which PSA level, quotas).
- [ ] GitOps for platform add-ons and team namespaces; drift detection (topic 68).

**Cost**
- [ ] Requests close to real usage (VPA recommendations, dashboards); idle environments scaled down (kube-green, topic 67); spot/preemptible node pools for tolerant workloads; cost allocation by labels (e.g. OpenCost/Kubecost).

**Disaster recovery**
- [ ] Documented, **tested** procedure to rebuild a cluster from IaC + GitOps + backups; RPO/RTO met in the last drill (topic 77).

### Quick automated checks

```bash
# workloads without readiness probes
kubectl get deploy -A -o json | jq -r '.items[] | select(any(.spec.template.spec.containers[]; .readinessProbe == null)) | "\(.metadata.namespace)/\(.metadata.name)"'
# containers without memory limits
kubectl get pods -A -o json | jq -r '.items[] | .metadata.namespace as $ns | .metadata.name as $p | .spec.containers[] | select(.resources.limits.memory == null) | "\($ns)/\($p)/\(.name)"'
# single-replica Deployments
kubectl get deploy -A -o jsonpath='{range .items[?(@.spec.replicas==1)]}{.metadata.namespace}/{.metadata.name}{"\n"}{end}'
# namespaces without Pod Security labels
kubectl get ns -L pod-security.kubernetes.io/enforce
# privileged containers
kubectl get pods -A -o json | jq -r '.items[] | select(any(.spec.containers[]; .securityContext.privileged == true)) | "\(.metadata.namespace)/\(.metadata.name)"'
# who is cluster-admin
kubectl get clusterrolebindings -o json | jq -r '.items[] | select(.roleRef.name=="cluster-admin") | .metadata.name + ": " + ([.subjects[]? | .kind + "/" + .name] | join(", "))'
```

Tools that automate much of this: **Polaris**, **kube-score**, **Kubescape** (NSA/CIS frameworks), **kube-bench** (CIS benchmark for nodes/control plane), **Popeye** (cluster sanitiser), and Kyverno policy reports.

```bash
kube-score score deploy.yaml
kubescape scan framework nsa
docker run --pid=host -v /etc:/etc:ro -v /var:/var:ro aquasec/kube-bench:latest run --targets node   # on a node
```

---

## Hands-on lab

1. Run the jq checks above against your kind cluster with the course's sample apps; list the gaps.
2. Run `kube-score` on three manifests from the course (e.g. `Day 47/demo/01-frontend.yaml`) and fix what it flags.
3. Run Kubescape on the cluster and pick the three highest-impact findings to fix.
4. Write a one-page PRR for one sample app (e.g. the 3-tier app from topic 61) using section A; mark accepted risks with owners.

---

## Where you'll use it

- **Launch reviews** for new services; **platform audits**; **compliance** evidence.
- **Interviews**: "what makes a workload production-ready?" (topic 87).
- **Your own clusters**: turn the list into admission policies and CI checks so it's enforced automatically.

---

## When it breaks

| Anti-pattern | Why it hurts | Fix |
|---|---|---|
| Checklist as paperwork, never enforced | Drift returns | Encode in admission policies, CI linters, templates |
| Liveness probe = readiness probe hitting dependencies | Restarts everything when a DB blips | Liveness checks only the process itself |
| CPU limits everywhere by default | Throttling and latency | Measure; consider requests only for CPU |
| PDB `minAvailable` = replicas | Drains never finish | `maxUnavailable: 1` |
| Backups without restore tests | False confidence | Scheduled restore drills |
| Alerts without runbooks/owners | Pages nobody can act on | Every alert: owner + runbook |

---

## Quick recap

- **Workload**: replicas across zones + PDB, readiness/liveness/startup probes, graceful shutdown, safe rollouts, requests/limits, autoscaling, restricted security context, dedicated SA + least RBAC, trusted signed images, secrets from a manager, NetworkPolicies, GitOps, logs/metrics/traces, SLOs + runbooks, tested backups.
- **Platform**: HA/managed control plane, supported versions + upgrade plan, etcd/PKI backups with drills, node pools across zones, PSA + admission policies, encryption at rest, audit logs, maintained ingress/gateway with TLS automation, default StorageClass, observability stack, quotas, cost controls, DR rehearsals.
- Automate it: policies, linters (kube-score, Polaris, Kubescape, kube-bench).

### Interview questions

1. **Q: What makes a Kubernetes workload production-ready?**
   A: Multiple replicas spread across zones with a PDB, proper readiness/liveness/startup probes and graceful shutdown, measured requests/limits and autoscaling, a restricted security context with a least-privilege ServiceAccount, trusted pinned images and managed secrets, NetworkPolicies, GitOps-managed config, metrics/logs/traces with SLO-based alerts and runbooks, and tested backups for any state.
2. **Q: How do you make sure teams actually follow these practices?**
   A: Enforce them in the platform: admission policies (Kyverno/Gatekeeper/PSA), CI linting (kube-score/Polaris), golden templates/Helm charts, default namespace bundles (quotas, LimitRanges, NetworkPolicies) generated automatically, and regular automated scans with reports.
3. **Q: What's on your checklist for the cluster itself?**
   A: HA or managed control plane, supported versions with a regular upgrade cadence, etcd and PKI backups with restore drills, multi-zone autoscaled node pools, RBAC with SSO, Pod Security and admission policies, encryption at rest, audit logging, maintained ingress/gateway with automated TLS, a sane default StorageClass, full observability, quotas, cost controls and a tested DR plan.

---

## Appendix: notes on the original course material

This topic was rebuilt from the original CKA course notes. This appendix keeps the audit trail out of your way while you learn.

### Sources

none: a **new topic** that consolidates best practices scattered through the original notes (e.g. Day 19 resources, Day 23 probes, Day 36 RBAC best practices, Day 41 least privilege, Day 45 registry practices, Day 47 least-privilege networking, Day 54–55 cluster operations, Day 56 observability).
