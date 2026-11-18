# Kubernetes: From Zero to Production

One learning path, in reading order. Each topic builds on the ones before it. When a later idea has to appear early, it is explained right there in a line or two, and a pointer like "(topic 82)" just tells you where the full story is. You never need to jump ahead to follow along.

**Every topic has three layers:**

1. **⚡ Core: the 10-minute version.** The idea in plain words, a mental model, the one command or YAML to know, a mini-lab, and a self-check.
2. **The full reference**, always in the same shape: What it is · Why it exists · Analogy · How it works inside (including what Linux and the container runtime are doing) · Commands / YAML (real output, explained) · Hands-on lab · Where you'll use it · When it breaks · Quick recap + interview questions.
3. **Appendix: notes on the original course material.** Which original Day sections the topic came from, and every correction to them in full (what the original said, what is true now and why). In the topic body, a correction shows as a short ✅ **Correct today** line or ⚠️ **Fix to the original** box that links here.

**Current Kubernetes (checked November 2026):** targets **v1.37** (released August 2026). Deprecated or removed things (dockershim, PodSecurityPolicy, basic auth, the Endpoints API, kube-proxy IPVS mode, cgroup v1, ingress-nginx, `patchesJson6902`/`patchesStrategicMerge`, ReplicationController…) are marked as such where they come up and are never taught as the main way. Facts that change often (feature stages, tool versions, the ingress-nginx retirement, Helm 4, Bitnami images) were checked against current sources in November 2026.

**Your lab:** kind (Kubernetes IN Docker) on the WSL2 Docker Engine from your Docker notes. Topic 11 sets it up. The cloud-only demos (EKS) say so clearly.

**Related notes:** [Linux notes](../linux-mastery-rhel/INDEX.md) · [Docker notes](../Docker/INDEX.md). Topics link straight to the relevant Linux or Docker topic files wherever Kubernetes builds on them.

**Sample outputs** show the *shape* of real output; IPs, names, hashes and versions are illustrative. The commands were not run while writing these notes, so your own terminal is the source of truth. Run every lab, and when the output differs, trust the output.

**Levels:** ★ foundation · ★★ intermediate · ★★★ advanced. **Status:** ✅ your original material, rewritten in this format · ➕ expanded with significant new material · 🔀 merged from several original days · 🆕 new topic.

**Proof nothing was lost:** [COVERAGE.md](COVERAGE.md) maps every original day file, section, image and YAML file to its new home. The original `Day NN/` folders are kept unchanged.

## How to study with these notes

| Your goal | Read | Time |
|---|---|---|
| **First pass: get the whole picture** | Only the **⚡ Core** of each topic, in order, doing every mini-lab | ~15 hours (89 × ~10 min) |
| **Become job-ready** | Second pass: the **full reference** of each topic, plus its Hands-on lab | A few weeks |
| **CKA preparation** | Full reference for the topics in [88](notes/88-cka-exam-strategy.md)'s domain table, then timed practice | 6–8 weeks (plan in topic 88) |
| **Interview tomorrow** | Cores of Parts 1–4 + [86](notes/86-interview-question-bank.md) + [87](notes/87-senior-scenario-interviews.md) | An evening |
| **Fixing something right now** | [81](notes/81-troubleshooting-method.md) → [82](notes/82-control-plane-troubleshooting.md)/[83](notes/83-data-plane-troubleshooting.md) → [89](notes/89-cheat-sheet.md) | Minutes |

You can skip a topic's appendix entirely unless you want to know how it differs from the original course notes.

---

## Part 0 · Foundations you bring

| # | Level | Status | Topic | What you'll learn |
|---|---|---|---|---|
| 01 | ★ | ➕ | [Why Kubernetes](notes/01-why-kubernetes.md) | Life before Docker, what Docker fixed, what it can't do at scale, orchestration, desired state, origin, Kubernetes vs Swarm vs Compose |
| 02 | ★ | 🔀 | [Docker recap I: containers and images](notes/02-docker-recap-containers-and-images.md) | VMs vs containers, image/container/registry, CLI → dockerd → containerd → runc, image names, pulling from ECR/Artifact Registry, Docker ↔ kubectl mapping |
| 03 | ★ | 🔀 | [Docker recap II: Dockerfiles](notes/03-docker-recap-dockerfiles.md) | Instructions, build context, shell vs exec form and PID 1, CMD vs ENTRYPOINT → `command`/`args`, multi-stage builds, layers, best practices |
| 04 | ★★ | 🆕 | [The Linux that Kubernetes runs on](notes/04-linux-under-kubernetes.md) | Signals, namespaces, the pause container, cgroups v2, overlayfs, the OCI runtime chain, veth/bridges/VXLAN, iptables/nftables, conntrack, resolv.conf, systemd |
| 05 | ★ | ✅ | [How apps are shaped](notes/05-app-architecture-monolith-3tier-microservices.md) | Monolith, three-tier, deployment eras, microservices, stateless vs stateful, external session stores, where databases run |

## Part 1 · Architecture

| # | Level | Status | Topic | What you'll learn |
|---|---|---|---|---|
| 06 | ★ | ➕ | [The cluster at a glance](notes/06-the-cluster-at-a-glance.md) | Control plane vs nodes, every component in one line, objects, spec vs status, Deployment → ReplicaSet → Pod, core vs extensions |
| 07 | ★★ | ➕ | [Control-plane components in depth](notes/07-control-plane-components.md) | API server pipeline, watches, resourceVersion, etcd/Raft/quorum, scheduler filter/score/bind, controllers and informers, leader election, CCM |
| 08 | ★★ | ➕ | [Node components](notes/08-node-components.md) | kubelet's jobs, CRI and the containerd chain, dockershim's removal, runtimes, `crictl`, kube-proxy at a glance |
| 09 | ★★ | ✅ | [Plugins, interfaces and add-ons](notes/09-plugins-interfaces-and-add-ons.md) | CRI, CNI, CSI: how each is called, responsibilities, implementations, other extension points |
| 10 | ★★ | 🔀 | [What happens after `kubectl apply`](notes/10-what-happens-after-kubectl-apply.md) | The full trace: kubectl → authn/authz/admission → etcd → controllers → scheduler → kubelet → CRI → CNI → status |

## Part 2 · Your lab and kubectl

| # | Level | Status | Topic | What you'll learn |
|---|---|---|---|---|
| 11 | ★ | ➕ | [Building your lab: kind (and minikube)](notes/11-lab-setup-kind-and-minikube.md) | Install, config-file clusters, node images, `extraPortMappings`, nodes as containers, minikube comparison |
| 12 | ★ | 🔀 | [kubectl, kubeconfig and contexts](notes/12-kubectl-kubeconfig-and-contexts.md) | Clusters/users/contexts, switching safely, default namespaces, command shape |
| 13 | ★ | ✅ | [YAML and manifest anatomy](notes/13-yaml-and-manifest-anatomy.md) | Scalars, maps, lists, traps (tabs, booleans, leading zeros), apiVersion/kind/metadata/spec/status |
| 14 | ★★ | ➕ | [The Kubernetes API](notes/14-the-kubernetes-api.md) | Endpoints, core vs named groups, versions and deprecation, URLs, curl with certs, `kubectl proxy`, `api-resources`, `explain` |
| 15 | ★ | 🔀 | [Imperative vs declarative](notes/15-imperative-vs-declarative.md) | run/create/apply/diff/patch/replace, dry runs, generating YAML, client- vs server-side apply, CKA speed setup |
| 16 | ★★ | ✅ | [kubectl output: JSONPath and friends](notes/16-kubectl-output-jsonpath-and-friends.md) | How `kubectl get` works, YAML ↔ JSON, JSONPath operators, range loops, custom-columns, sort-by, selectors |
| 17 | ★ | ➕ | [Labels, selectors and annotations](notes/17-labels-selectors-and-annotations.md) | Equality vs set-based selectors, matchLabels/matchExpressions, adoption traps, recommended labels, annotations |

## Part 3 · Pods

| # | Level | Status | Topic | What you'll learn |
|---|---|---|---|---|
| 18 | ★ | ➕ | [Pods](notes/18-pods.md) | What containers in a Pod share, the sandbox, Pod IPs, bare vs managed Pods, everyday Pod commands |
| 19 | ★★ | ✅ | [Pod lifecycle, restarts and termination](notes/19-pod-lifecycle-restarts-and-termination.md) | Phases, container states, conditions, restart policies and back-off, SIGTERM → grace → SIGKILL, preStop, force delete |
| 20 | ★ | 🔀 | [Images and pull policy](notes/20-images-and-pull-policy.md) | Registries/repositories/tags/digests, name defaults, Always/IfNotPresent/Never and how the default is chosen |
| 21 | ★★ | 🔀 | [Debugging Pods](notes/21-debugging-pods.md) | A method; Pending, ImagePullBackOff, CrashLoopBackOff, OOMKilled, CreateContainerConfigError; `kubectl debug` |
| 22 | ★★ | ✅ | [Health probes](notes/22-health-probes.md) | Readiness, liveness, startup; HTTP/TCP/exec/gRPC; timing; multi-container behaviour; production rules |
| 23 | ★★ | ➕ | [Multi-container Pods](notes/23-multi-container-pods.md) | Init containers, **native sidecars**, ambassador, adapter; three demos |

## Part 4 · Workloads

| # | Level | Status | Topic | What you'll learn |
|---|---|---|---|---|
| 24 | ★ | ✅ | [ReplicationController → ReplicaSet](notes/24-replicationcontroller-and-replicaset.md) | Keeping N Pods alive, set-based selectors, adoption, why not to use them directly |
| 25 | ★★ | ➕ | [Deployments](notes/25-deployments.md) | Rolling updates (maxSurge/maxUnavailable), Recreate, revisions and rollbacks, pause/resume/restart, blue-green and canary |
| 26 | ★ | ✅ | [Namespaces](notes/26-namespaces.md) | What they do and don't isolate, built-in namespaces, cross-namespace access, stuck Terminating |
| 27 | ★★ | ✅ | [DaemonSets](notes/27-daemonsets.md) | One Pod per node, node agents, tolerations, `operator: Exists`, updates, logging-agent demo |
| 28 | ★★ | ➕ | [Jobs and CronJobs](notes/28-jobs-and-cronjobs.md) | Completions, parallelism, backoffLimit, TTL, indexed Jobs, cron syntax, time zones, concurrency policy |

## Part 5 · Service networking

| # | Level | Status | Topic | What you'll learn |
|---|---|---|---|---|
| 29 | ★★★ | 🆕 | [The network model and CNI](notes/29-network-model-and-cni.md) | The three rules, CIDRs, CNI ADD step by step, same-node and cross-node paths, routed vs overlay vs VPC-native, MTU |
| 30 | ★ | ✅ | [Services I: ClusterIP](notes/30-services-clusterip.md) | Why Services, port/targetPort, EndpointSlices, frontend → backend demo, imperative Services |
| 31 | ★★ | ✅ | [Services II: NodePort, LoadBalancer, ExternalName, headless](notes/31-services-nodeport-loadbalancer-externalname-headless.md) | Each type's flow, `<pending>` LBs, ExternalName traps, selector-less Services, traffic policies |
| 32 | ★★★ | 🆕 | [Inside a Service: kube-proxy](notes/32-inside-a-service-kube-proxy.md) | iptables/nftables/IPVS(deprecated)/eBPF, the real NAT chains, conntrack, NodePort SNAT, "Service doesn't work" method |
| 33 | ★★ | ✅ | [Cluster DNS and CoreDNS](notes/33-cluster-dns-and-coredns.md) | Record types, resolv.conf and ndots, Corefile plugins, dnsPolicy/dnsConfig, NodeLocal DNSCache, troubleshooting |

## Part 6 · Configuration and storage

| # | Level | Status | Topic | What you'll learn |
|---|---|---|---|---|
| 34 | ★ | ✅ | [ConfigMaps](notes/34-configmaps.md) | Env vars vs mounted files, subPath, update behaviour, immutable ConfigMaps |
| 35 | ★★ | ➕ | [Secrets](notes/35-secrets.md) | Encoding vs encryption, Secret types, env vs files, encryption at rest, external secret managers |
| 36 | ★ | ✅ | [Docker storage recap](notes/36-docker-storage-recap.md) | Image layers, the writable layer, storage drivers, volumes, bind mounts |
| 37 | ★★ | ➕ | [Ephemeral volumes](notes/37-ephemeral-volumes.md) | emptyDir (disk/memory), downward API, projected volumes, ephemeral storage limits |
| 38 | ★★ | ✅ | [Persistent storage: PV and PVC](notes/38-persistent-storage-pv-pvc.md) | In-tree → CSI, hostPath and local, PV/PVC binding, access modes, reclaim policies |
| 39 | ★★ | ➕ | [StorageClasses and dynamic provisioning](notes/39-storageclasses-dynamic-provisioning.md) | Provisioners, WaitForFirstConsumer, expansion, snapshots |
| 40 | ★★★ | 🔀 | [StatefulSets](notes/40-statefulsets.md) | Stable names, DNS and volumes; ordering; MySQL on multi-AZ EKS; chaos test |

## Part 7 · Scheduling, resources and scaling

| # | Level | Status | Topic | What you'll learn |
|---|---|---|---|---|
| 41 | ★★ | ➕ | [Requests, limits, QoS and quotas](notes/41-requests-limits-qos-quotas.md) | cgroups, throttling, OOM kills, QoS classes, LimitRange, ResourceQuota |
| 42 | ★★ | ✅ | [The scheduler, manual scheduling and static Pods](notes/42-scheduler-manual-scheduling-static-pods.md) | nodeName, static Pods and mirror Pods |
| 43 | ★★ | ✅ | [Taints and tolerations](notes/43-taints-and-tolerations.md) | Effects, operators, NoExecute eviction, dedicated nodes |
| 44 | ★★ | ✅ | [nodeSelector and node affinity](notes/44-nodeselector-and-node-affinity.md) | Required vs preferred, operators, weights |
| 45 | ★★ | ✅ | [Taints vs affinity: using both](notes/45-taints-vs-affinity.md) | The dedicated-node pattern |
| 46 | ★★ | 🆕 | [Pod affinity, anti-affinity and topology spread](notes/46-pod-affinity-and-topology-spread.md) | Spreading replicas across nodes and zones |
| 47 | ★★ | ➕ | [Priority, preemption and eviction](notes/47-priority-preemption-eviction.md) | PriorityClass, preemption, node-pressure eviction |
| 48 | ★★ | 🔀 | [Metrics Server and HPA](notes/48-metrics-server-and-hpa.md) | Metrics pipeline, the HPA algorithm and behaviour |
| 49 | ★★★ | ➕ | [VPA, in-place resize and node autoscaling](notes/49-vpa-inplace-resize-node-autoscaling.md) | VPA modes, in-place resize, Cluster Autoscaler, Karpenter, KEDA |

## Part 8 · Security

| # | Level | Status | Topic | What you'll learn |
|---|---|---|---|---|
| 50 | ★ | ✅ | [Encryption basics](notes/50-encryption-basics.md) | Symmetric vs asymmetric, in transit vs at rest, SSH, the TLS handshake |
| 51 | ★★ | ✅ | [Certificates and CAs](notes/51-certificates-and-cas.md) | Public-key auth, mutual auth, public/private/self-signed CAs |
| 52 | ★★ | 🔀 | [kubeconfig in depth and mTLS](notes/52-kubeconfig-deep-dive-and-mtls.md) | Every kubeconfig field, KUBECONFIG merging, mutual TLS |
| 53 | ★★★ | ✅ | [TLS inside the cluster](notes/53-tls-inside-the-cluster.md) | Who is client and server, scheduler ↔ API server ↔ etcd ↔ kubelet |
| 54 | ★★ | 🔀 | [Authentication](notes/54-authentication.md) | Users vs ServiceAccounts, certificates, tokens, OIDC, the CSR flow for a new user |
| 55 | ★★ | ✅ | [ServiceAccounts and tokens](notes/55-serviceaccounts-and-tokens.md) | Bound/projected tokens, TokenRequest, the Jenkins demo |
| 56 | ★★ | ✅ | [Authorization modes](notes/56-authorization-modes.md) | Node, RBAC, ABAC, Webhook, AlwaysAllow/Deny, layering |
| 57 | ★★ | ✅ | [RBAC](notes/57-rbac.md) | Roles, ClusterRoles, bindings, `auth can-i`, least privilege |
| 58 | ★★★ | ✅ | [Admission control](notes/58-admission-control.md) | Mutating/validating, webhooks, ValidatingAdmissionPolicy (CEL), Gatekeeper vs Kyverno |
| 59 | ★★ | ✅ | [securityContext and capabilities](notes/59-securitycontext-and-capabilities.md) | runAsUser, fsGroup, read-only root, privilege escalation, Linux capabilities |
| 60 | ★★ | 🆕 | [Pod Security, seccomp and AppArmor](notes/60-pod-security-seccomp-apparmor.md) | PSP → Pod Security Admission, standards, seccomp, AppArmor, RuntimeClass |
| 61 | ★★★ | ✅ | [Network Policies](notes/61-network-policies.md) | Default allow → least privilege, selectors, AND vs OR, DNS egress, Calico demo |
| 62 | ★★ | ✅ | [Private registries](notes/62-private-registries.md) | imagePullSecrets, ServiceAccount-attached secrets, credential providers |
| 63 | ★★★ | 🆕 | [Supply chain and secrets in production](notes/63-supply-chain-and-secrets-in-production.md) | Scanning, signing, admission verification, External Secrets, Vault, audit logs |

## Part 9 · Packaging and extending

| # | Level | Status | Topic | What you'll learn |
|---|---|---|---|---|
| 64 | ★★ | ✅ | [Kustomize](notes/64-kustomize.md) | Base/overlays, transformers, patches (deprecated forms marked) |
| 65 | ★★ | ✅ | [Helm](notes/65-helm.md) | Charts, templating, releases, upgrade/rollback, multi-environment values |
| 66 | ★★★ | ✅ | [CRDs and custom controllers](notes/66-crds-and-custom-controllers.md) | Custom resources, control loops, the BackupPolicy demo |
| 67 | ★★★ | ✅ | [Operators](notes/67-operators.md) | Why operators, components, OLM, kube-green demo |
| 68 | ★★ | 🆕 | [GitOps with Argo CD and Flux](notes/68-gitops.md) | Pull-based delivery, drift, sync, promotion |

## Part 10 · Traffic into the cluster

| # | Level | Status | Topic | What you'll learn |
|---|---|---|---|---|
| 69 | ★★ | ✅ | [Ingress concepts](notes/69-ingress-concepts.md) | Ingress vs controller, real-world flow, ingress-nginx retirement |
| 70 | ★★★ | ✅ | [Ingress on EKS: path routing](notes/70-ingress-on-eks-path-routing.md) | AWS Load Balancer Controller, IngressClass, IP vs instance targets |
| 71 | ★★★ | ✅ | [Ingress TLS and host routing](notes/71-ingress-tls-and-host-routing.md) | ACM, Route 53, subdomains |
| 72 | ★★ | ✅ | [Gateway API concepts](notes/72-gateway-api-concepts.md) | GatewayClass/Gateway/Routes, roles, what Ingress lacked |
| 73 | ★★ | ✅ | [Gateway API hands-on](notes/73-gateway-api-hands-on.md) | NGINX Gateway Fabric on kind |
| 74 | ★★ | 🆕 | [Service mesh essentials](notes/74-service-mesh-essentials.md) | Sidecar vs ambient, mTLS, traffic control, when you need one |

## Part 11 · Cluster operations

| # | Level | Status | Topic | What you'll learn |
|---|---|---|---|---|
| 75 | ★★★ | ➕ | [Build a cluster with kubeadm](notes/75-kubeadm-cluster-build.md) | Node prep, containerd, kubeadm init/join, Calico, updated to v1.37 |
| 76 | ★★★ | ➕ | [Versions, skew and upgrades](notes/76-versions-skew-and-upgrades.md) | Release cadence, skew rules, kubeadm upgrades, cordon/drain, PodDisruptionBudgets |
| 77 | ★★★ | 🆕 | [etcd backup, restore and DR](notes/77-etcd-backup-restore-and-dr.md) | `etcdctl snapshot`, restore, Velero |
| 78 | ★★ | 🆕 | [HA control plane and managed Kubernetes](notes/78-ha-control-plane-and-managed-kubernetes.md) | Stacked vs external etcd, EKS/GKE/AKS |
| 79 | ★★ | ✅ | [Monitoring and logging basics](notes/79-monitoring-and-logging-basics.md) | Observability pillars, metrics-server internals, log paths |
| 80 | ★★★ | 🆕 | [Production observability](notes/80-production-observability.md) | Prometheus, kube-state-metrics, Grafana, Loki, alerts |

## Part 12 · Troubleshooting

| # | Level | Status | Topic | What you'll learn |
|---|---|---|---|---|
| 81 | ★★ | 🆕 | [A troubleshooting method](notes/81-troubleshooting-method.md) | Symptom → layer → tool |
| 82 | ★★★ | ✅ | [Control-plane troubleshooting](notes/82-control-plane-troubleshooting.md) | API server, etcd, scheduler, controller manager, expired certificates, `ss` |
| 83 | ★★★ | ✅ | [Data-plane troubleshooting](notes/83-data-plane-troubleshooting.md) | NotReady, CNI, Services, DNS, images, crashes, PVCs, NodePort |
| 84 | ★★★ | 🆕 | [Production break-fix scenarios](notes/84-production-break-fix-scenarios.md) | Reproducible incidents to break and fix |

## Part 13 · Interviews and reference

| # | Level | Status | Topic | What you'll learn |
|---|---|---|---|---|
| 85 | ★★ | 🆕 | [Production-readiness checklist](notes/85-production-readiness-checklist.md) | One page before go-live |
| 86 | ★★ | 🆕 | [Interview question bank](notes/86-interview-question-bank.md) | Questions with model answers across all parts |
| 87 | ★★★ | 🆕 | [Senior scenario interviews](notes/87-senior-scenario-interviews.md) | Incidents, designs and judgement calls |
| 88 | ★★ | 🆕 | [CKA exam strategy](notes/88-cka-exam-strategy.md) | Time management, imperative shortcuts, using the docs |
| 89 | ★ | 🆕 | [Command cheat sheet](notes/89-cheat-sheet.md) | Every command, grouped by task |
