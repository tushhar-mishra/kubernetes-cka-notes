# 86 · Interview Question Bank

> **Level:** ★★ intermediate → ★★★ advanced
>
> **You will learn:** to answer the most common Kubernetes interview questions clearly and correctly: grouped by part of the course, each with a **short model answer** and a pointer to the topic that explains it in depth. Use it for revision: cover the answer, say yours out loud, then compare.
>
> **You need:** the topics referenced in each answer.

---

## ⚡ Core: the 10-minute version

**The idea.** About 110 common interview questions with short model answers, grouped by part of the course, plus rapid-fire facts (ports, defaults, commands). Each answer links to the topic that explains it in depth.

**Picture it.** Flashcards: quick recall first, then back to the textbook for misses.

**The one thing to know.** A good answer = **the mechanism** ("how it works") **plus a trade-off or failure mode**, said in under a minute.

**Try it now:**
1. Pick 10 random questions and answer each aloud in 60 seconds.
2. Compare with the model answers.
3. Reread the linked topic for every miss.

**You've got it if you can:** answer every "Part" section without looking, including the "why" behind each answer.

> 📖 **That's the core.** On a first pass, do the mini-lab and move on to the next topic. Everything below is the **full reference**: come back for the details, the edge cases, interview answers and exam-level depth.

---

## What it is

A revision bank of **~110 questions** with concise answers. Answers are deliberately short (what you'd say in 30–60 seconds); follow the topic link for depth.

## Why it exists

Interviews test whether you can **explain** concepts and **reason** about trade-offs, not just run commands. Practising aloud with model answers is the fastest way to find gaps.

## Analogy

Flashcards before an exam: quick recall first, then go back to the textbook (the topic) for whatever you got wrong.

## How it works inside

1. Read the question; answer aloud in under a minute.
2. Compare with the model answer: did you mention the key mechanism and a trade-off?
3. For misses, reread the linked topic and redo its lab.

---

## Commands / YAML

### Part 1: Foundations (topics 01–05)

1. **What problem does Kubernetes solve?** Running containers across many machines reliably: scheduling, self-healing, scaling, service discovery, rollouts and declarative config, so you describe the desired state and controllers maintain it. (01)
2. **Container vs VM?** Containers share the host kernel and isolate processes with namespaces and cgroups: lightweight, fast to start; VMs virtualise hardware with their own kernel: stronger isolation, heavier. (02, 04)
3. **What makes a container image?** Read-only layers (a filesystem built by Dockerfile instructions) plus config (entrypoint, env); identified by a digest; layers are shared and cached. (02–03)
4. **`CMD` vs `ENTRYPOINT`?** ENTRYPOINT is the executable; CMD gives default arguments (or the default command if no ENTRYPOINT). In Kubernetes, `command` overrides ENTRYPOINT and `args` overrides CMD. (03, 18)
5. **Which Linux features make containers work?** Namespaces (pid, net, mnt, uts, ipc, user, cgroup) for isolation; cgroups for resource limits; capabilities, seccomp, LSMs for privilege; overlay filesystems for images. (04)
6. **Monolith vs microservices?** One deployable unit vs many independently deployable services: microservices allow independent scaling/releases but add network, observability and consistency complexity. (05)
7. **Stateless vs stateful?** Stateless apps keep no local state, so any replica can serve any request (Deployments); stateful apps need stable identity and storage (StatefulSets, operators). (05, 40)

### Part 2: Architecture (topics 06–10)

8. **Name the control-plane components and their jobs.** API server (front door, validation, the only etcd client), etcd (state), scheduler (assigns Pods to nodes), controller manager (reconciliation loops), cloud-controller-manager (cloud integration). (07)
9. **Node components?** kubelet (runs Pods via CRI, reports status, probes), container runtime (containerd/CRI-O), kube-proxy (Service rules), plus the CNI plugin. (08)
10. **What happens after `kubectl apply -f deploy.yaml`?** Authn → authz → admission → validation → stored in etcd → Deployment controller creates a ReplicaSet → ReplicaSet controller creates Pods → scheduler binds them → kubelet pulls images, CNI wires networking, containers start → status flows back. (10)
11. **CRI, CNI, CSI?** Plugin interfaces for container runtimes, Pod networking and storage, so Kubernetes stays vendor-neutral. (09)
12. **Why was dockershim removed?** Kubernetes talks to runtimes via CRI; Docker Engine didn't implement CRI natively, so the shim was removed in 1.24; containerd/CRI-O run the same OCI images. (08)
13. **Is the API server stateless?** Yes; all state is in etcd, so you can run several API servers behind a load balancer. (07, 78)

### Part 3: kubectl, YAML, API (topics 11–17)

14. **What's in a kubeconfig?** Clusters (server + CA), users (credentials), contexts (cluster + user + namespace) and the current context. (12, 52)
15. **The four required fields of a manifest?** `apiVersion`, `kind`, `metadata`, and (for most kinds) `spec`. (13)
16. **Core vs named API groups?** Core (`v1`) lives at `/api/v1` (Pods, Services); named groups at `/apis/<group>/<version>` (apps/v1 Deployments). (14)
17. **Imperative vs declarative?** Commands that perform actions vs files describing desired state applied with `kubectl apply` (three-way merge; GitOps-friendly). (15)
18. **Labels vs annotations?** Labels are identifying key/values used by selectors; annotations hold non-identifying metadata for tools. (17)
19. **Equality vs set-based selectors?** `app=web` vs `env in (dev,qa)`; Deployments/ReplicaSets support `matchExpressions`. (17)

### Part 4: Workloads (topics 18–28)

20. **Why Pods and not containers?** A Pod is the scheduling unit: one or more containers sharing network namespace, IPC and volumes, always co-scheduled. (18)
21. **Pod phases?** Pending, Running, Succeeded, Failed, Unknown; container states are Waiting/Running/Terminated. (19)
22. **What happens when a Pod is deleted?** Marked Terminating, removed from endpoints, preStop runs, SIGTERM, grace period (30 s default), then SIGKILL. (19)
23. **`restartPolicy` options?** Always (Deployments), OnFailure, Never (Jobs use OnFailure/Never). (19)
24. **`imagePullPolicy` defaults?** `Always` for `:latest` or no tag, otherwise `IfNotPresent`. (20)
25. **Liveness vs readiness vs startup probes?** Liveness restarts a stuck container; readiness controls whether it receives traffic; startup delays the others for slow starters. (22)
26. **Sidecar vs init container?** Init containers run to completion before app containers; sidecars run alongside (native sidecars: init containers with `restartPolicy: Always`, GA in 1.33). (23)
27. **ReplicaSet vs Deployment?** A ReplicaSet keeps N Pods; a Deployment manages ReplicaSets to provide rolling updates and rollbacks. (24–25)
28. **`maxSurge` and `maxUnavailable`?** How many extra Pods may be created and how many may be unavailable during a rolling update. (25)
29. **How do you roll back?** `kubectl rollout undo deploy/x [--to-revision=N]`; the Deployment scales the old ReplicaSet back up. (25)
30. **DaemonSet use cases?** One Pod per (selected) node: log agents, CNI, node-exporter, CSI node plugins. (27)
31. **Job vs CronJob?** A Job runs Pods to completion (with retries/parallelism); a CronJob creates Jobs on a schedule. (28)
32. **What does a namespace isolate?** Names and policy scope (RBAC, quotas, NetworkPolicies, PSA); not nodes or network by default. (26)

### Part 5: Networking (topics 29–33)

33. **The Kubernetes network model?** Every Pod gets an IP; all Pods can reach each other without NAT; agents on a node can reach its Pods. CNI plugins implement it. (29)
34. **Why Services?** Pod IPs change; a Service gives a stable virtual IP and DNS name and load-balances across ready Pods selected by labels. (30)
35. **ClusterIP vs NodePort vs LoadBalancer vs ExternalName vs headless?** Internal VIP; a port on every node; a cloud LB in front; a DNS CNAME; DNS returning Pod IPs directly with no VIP. (31)
36. **How does kube-proxy implement a Service?** It watches Services and EndpointSlices and programs nftables/iptables (IPVS deprecated) NAT rules on every node; it's not in the data path itself. (32)
37. **What's an EndpointSlice?** The list of ready Pod IPs/ports behind a Service, sharded for scale; replaces the deprecated Endpoints API. (32)
38. **Service DNS name format?** `<svc>.<ns>.svc.cluster.local`; StatefulSet Pods: `<pod>.<headless-svc>.<ns>.svc.cluster.local`. (33, 40)
39. **What does `ndots:5` do?** Names with fewer than 5 dots are tried against the search domains first: convenient but causes extra queries for external names. (33)

### Part 6: Config and storage (topics 34–40)

40. **ConfigMap vs Secret?** Both inject config; Secrets are for sensitive data (base64-encoded, can be encrypted at rest, tighter RBAC), not encrypted by default. (34–35)
41. **Env vars vs volume mounts for config?** Env vars are fixed at container start; mounted ConfigMaps/Secrets update in place (not with subPath). (34)
42. **emptyDir vs hostPath vs PVC?** Pod-lifetime scratch space; a node path (dangerous, node-coupled); durable storage via a claim. (37–38)
43. **PV vs PVC vs StorageClass?** The storage resource; a request for storage; a template for dynamic provisioning. (38–39)
44. **Access modes?** RWO (one node), ROX, RWX (many nodes), RWOP (one Pod, GA 1.29). (38)
45. **Reclaim policies?** Retain (keep data), Delete (remove the backing volume); Recycle is deprecated. (38)
46. **`WaitForFirstConsumer` why?** Delays binding/provisioning until a Pod is scheduled, so the volume is created in the right zone. (39)
47. **StatefulSet guarantees?** Stable names and network identities, per-Pod PVCs (volumeClaimTemplates), ordered deployment/scaling. (40)

### Part 7: Scheduling and scaling (topics 41–49)

48. **Requests vs limits?** Requests are used for scheduling and guaranteed share; limits cap usage (CPU throttled, memory OOMKilled). (41)
49. **QoS classes?** Guaranteed (requests = limits for all), Burstable, BestEffort; eviction order under pressure is BestEffort first. (41)
50. **ResourceQuota vs LimitRange?** Quota caps a namespace's total; LimitRange sets per-container defaults and min/max. (41)
51. **Taints/tolerations vs node affinity?** Taints repel Pods from nodes unless tolerated; affinity attracts Pods to nodes. Combine them for dedicated nodes. (43–45)
52. **Pod anti-affinity vs topology spread?** Anti-affinity keeps Pods apart (hard/soft); topology spread balances counts across domains with a max skew. (46)
53. **What is preemption?** The scheduler evicts lower-priority Pods to place a higher-priority Pod when no node fits. (47)
54. **What's a static Pod?** A Pod defined by a manifest file the kubelet runs directly; the API shows a read-only mirror Pod. (42, 82)
55. **How does HPA decide replicas?** `desired = ceil(current × currentMetric / target)`, using metrics-server (or custom/external metrics), with stabilisation windows. (48)
56. **HPA vs VPA vs Cluster Autoscaler/Karpenter?** More Pods vs bigger Pods vs more nodes. (48–49)

### Part 8: Security (topics 50–63)

57. **Symmetric vs asymmetric encryption in TLS?** Asymmetric (and ECDHE key exchange) establishes keys and proves identity; symmetric encrypts the bulk data. (50)
58. **What does a CA do?** Signs certificates, vouching that a public key belongs to a name; clients trust the CA's certificate. (51)
59. **What's mTLS?** Both client and server present certificates; Kubernetes components use it heavily. (52–53)
60. **401 vs 403?** Authentication failed vs authenticated but not authorised. (54, 56)
61. **How are users added?** Externally: OIDC identities (preferred) or certificates signed by the cluster CA (CSR API); RBAC grants permissions. (54)
62. **ServiceAccount tokens today?** Short-lived, audience- and Pod-bound projected JWTs from the TokenRequest API, auto-rotated by the kubelet. (55)
63. **Role vs ClusterRole; RoleBinding vs ClusterRoleBinding?** Namespaced vs cluster-wide permissions; namespace-scoped vs cluster-wide grants; a RoleBinding can reference a ClusterRole for one namespace. (57)
64. **Can RBAC deny?** No, it's additive only. (56–57)
65. **Authorization vs admission?** Authz checks request attributes (who, verb, resource); admission sees and can mutate/reject the object. (56, 58)
66. **Examples of admission controllers?** NamespaceLifecycle, LimitRanger, ServiceAccount, ResourceQuota, PodSecurity, NodeRestriction, mutating/validating webhooks, ValidatingAdmissionPolicy. (58)
67. **What replaced PodSecurityPolicy?** Pod Security Admission with Baseline/Restricted standards (PSP removed in 1.25). (60)
68. **Key securityContext settings?** `runAsNonRoot`, `runAsUser`, `allowPrivilegeEscalation: false`, `readOnlyRootFilesystem`, `capabilities.drop: [ALL]`, `seccompProfile: RuntimeDefault`. (59–60)
69. **NetworkPolicy behaviour?** Pods selected by a policy are isolated for that direction; only allowed traffic passes; enforced by the CNI; AND vs OR selector semantics; remember DNS for egress. (61)
70. **How do you secure Secrets?** Encryption at rest (KMS), RBAC, external managers (ESO/Vault/CSI), file mounts, rotation, no Git plaintext. (35, 63)
71. **How do you pull from private registries?** `docker-registry` Secrets via `imagePullSecrets` (Pod or ServiceAccount) or node/workload identity credential providers. (62)

### Part 9: Packaging and extending (topics 64–68)

72. **Kustomize vs Helm?** Overlays/patches on plain YAML (built into kubectl) vs templated, versioned charts with release management. (64–65)
73. **What's a Helm release and how does rollback work?** A named installation; each change is a revision stored as a Secret; rollback re-applies an old revision as a new one. (65)
74. **CRD vs CR vs controller vs operator?** Type definition; instance; reconciliation loop; a controller + CRDs encoding operational knowledge. (66–67)
75. **What is GitOps?** Desired state in Git, applied and continuously reconciled by an in-cluster agent (Argo CD/Flux) with drift detection. (68)

### Part 10: Traffic (topics 69–74)

76. **Ingress vs Ingress controller?** Rules object vs the component implementing them; Ingress is frozen, ingress-nginx retired. (69)
77. **Path vs host routing?** Route by URL path prefix or by Host header (SNI for TLS). (69–71)
78. **Why Gateway API?** Role separation, cross-namespace delegation, more protocols, portable traffic features without annotations. (72–73)
79. **What does a service mesh add?** mTLS identity, L7 policy, retries/timeouts, traffic splitting, telemetry for east-west traffic. (74)

### Part 11: Cluster operations (topics 75–80)

80. **What does `kubeadm init` set up?** Certificates, kubeconfigs, static Pod control plane, bootstrap tokens, CoreDNS, kube-proxy. (75)
81. **Upgrade order and skew rules?** One minor at a time, control plane first, then nodes; kubelets ≤ 3 minors older, never newer. (76)
82. **What does `kubectl drain` do?** Cordons and evicts Pods via the Eviction API, honouring PDBs. (76)
83. **How do you back up etcd?** `etcdctl snapshot save` with TLS flags; restore with `etcdutl snapshot restore` to a new data dir and update the manifest. (77)
84. **HA control plane design?** 3 control planes in 3 zones, LB for API servers, etcd quorum, leader election for scheduler/controller manager. (78)
85. **Where are kubelet logs vs container logs?** `journalctl -u kubelet`; `kubectl logs` / `/var/log/pods`. (79)
86. **How does `kubectl top` work?** Aggregated metrics API → metrics-server → kubelet resource metrics. (79)
87. **What do you alert on?** User-facing symptoms via SLO burn rates; causes for diagnosis; every alert with a runbook. (80)

### Part 12: Troubleshooting (topics 81–84)

88. **Pod Pending: causes?** Insufficient resources, taints, affinity, unbound PVC, scheduler down. (81–83)
89. **CrashLoopBackOff: approach?** `logs --previous`, `describe` (exit code, reason), check command/config/dependencies/memory. (21)
90. **Exit code 137?** SIGKILL: often OOMKilled. (21, 81)
91. **Service not reachable?** Check EndpointSlices (selector/readiness), ports/targetPort, kube-proxy, NetworkPolicy, DNS. (32, 83)
92. **Node NotReady?** kubelet/runtime status and logs, swap, certificates, API reachability, pressure, CNI. (83)
93. **API server down on kubeadm?** `crictl ps -a`, container logs, static Pod manifest, etcd health, port 6443. (82)
94. **Certificates expired?** `kubeadm certs check-expiration`, `renew all`, restart control-plane Pods, update kubeconfig. (82)
95. **DNS failing in Pods?** CoreDNS Pods/logs, kube-dns endpoints, resolv.conf, NetworkPolicies on port 53. (33, 83)

### Rapid-fire (one-liners)

96. Default Service CIDR on kubeadm? `10.96.0.0/12`.
97. Default termination grace period? 30 s.
98. Max Pods per node (default kubelet)? 110.
99. NodePort range? 30000–32767.
100. kubelet port? 10250. API server? 6443. etcd? 2379/2380.
101. Where are static Pod manifests on kubeadm? `/etc/kubernetes/manifests/`.
102. Command to see why a Pod is Pending? `kubectl describe pod` (Events).
103. Shortcut for all resources' short names? `kubectl api-resources`.
104. Explain a field? `kubectl explain deploy.spec.strategy`.
105. Generate YAML quickly? `kubectl create deploy x --image=nginx --dry-run=client -o yaml`.
106. Check permissions as someone else? `kubectl auth can-i <verb> <resource> --as=<user>`.
107. List Pods on a node? `kubectl get pods -A --field-selector spec.nodeName=<node>`.
108. Restart a Deployment's Pods? `kubectl rollout restart deploy/<name>`.
109. Get a Secret's value? `kubectl get secret s -o jsonpath='{.data.key}' | base64 -d`.
110. Which component evicts Pods under memory pressure? The kubelet.

---

## Hands-on lab

1. Pick 20 random questions; answer aloud; time yourself; mark misses.
2. For each miss, redo the linked topic's lab.
3. Pair up: one person asks follow-ups ("why?", "what if…?") to each answer: the follow-ups are what real interviews dig into.

---

## Where you'll use it

Interview preparation (junior to senior), certification revision (CKA/CKAD/CKS), and teaching others.

---

## When it breaks

| Interview mistake | Better |
|---|---|
| Reciting definitions only | Add the mechanism ("how it works") and a trade-off |
| Saying "I'd just restart it" | Show the debugging method (topic 81) |
| Outdated facts (PSP, dockershim, `extensions/v1beta1`) | Use current versions; mention what replaced them |
| Claiming certainty when unsure | Say how you'd verify (`kubectl explain`, docs, a quick lab) |

---

## Quick recap

- ~110 questions across all parts, with 30–60 second answers and topic links.
- Practise aloud, focus on mechanisms and trade-offs, and revisit topics for misses.
- Senior-level scenario questions are in topic 87; exam tactics in topic 88; commands in topic 89.

### Interview questions

1. **Q: Explain Kubernetes to a non-technical manager in one minute.**
   A: It's an automated operations system for containers: you describe what should run (how many copies, which version, how to reach it) and it keeps reality matching that description, moving and restarting apps when machines fail and rolling out new versions safely.
2. **Q: What's the most important concept in Kubernetes?**
   A: Declarative desired state reconciled by controllers: everything (scaling, self-healing, rollouts) follows from controllers continuously comparing spec with status and acting.
3. **Q: What changed in Kubernetes recently that matters to operators?**
   A: For example: native sidecars GA, in-place Pod resize GA, nftables kube-proxy GA and IPVS deprecated, the Endpoints API deprecated in favour of EndpointSlices, Gateway API as the successor to the frozen Ingress API with ingress-nginx retired, and kubelet skew extended to three minors.

---

## Appendix: notes on the original course material

This topic was rebuilt from the original CKA course notes. This appendix keeps the audit trail out of your way while you learn.

### Sources

none: a **new topic** consolidating interview material. Every topic in this course also ends with 2–3 interview questions; this bank adds breadth and ties them together.
