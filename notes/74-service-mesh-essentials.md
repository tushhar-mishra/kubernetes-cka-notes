# 74 · Service Mesh Essentials (Istio, Linkerd, Cilium)

> **Level:** ★★★ advanced
>
> **You will learn:** what a service mesh is and the problems it solves for service-to-service ("east-west") traffic: **mTLS** with workload identity, L7 authorization, retries/timeouts, traffic splitting, and golden-signal telemetry without code changes; data plane vs control plane; **sidecar** vs **sidecarless** (Istio ambient, Cilium) designs; how a sidecar intercepts traffic (iptables/eBPF, native sidecars); Istio and Linkerd basics; Gateway API for meshes (GAMMA); and when **not** to use a mesh.
>
> **You need:** topics 23 (multi-container Pods, native sidecars), 29–32 (networking, kube-proxy), 51–53 (TLS/mTLS), 58 (admission webhooks for injection), 61 (NetworkPolicies), 72 (Gateway API).

---

## ⚡ Core: the 10-minute version

**The idea.** A **service mesh** (Istio, Linkerd, Cilium) puts a proxy in the path of every service-to-service call. A sidecar per Pod, or a per-node proxy in "ambient"/sidecarless modes, gives you, without code changes:
- automatic **mutual TLS** between workloads, using their ServiceAccount identities;
- **L7 policies** ("only `orders` may `POST /charge` on payments");
- retries and timeouts;
- traffic splitting;
- per-request metrics.

It costs extra complexity and resources, so adopt one when you need those features at scale.

**Picture it.** One professional courier service with a courier at every office door, checking badges and logging every delivery.

**The one thing to know.**

```bash
linkerd install --crds | kubectl apply -f - && linkerd install | kubectl apply -f -
kubectl annotate ns shop linkerd.io/inject=enabled            # new Pods get the proxy
kubectl -n shop get pods                                      # READY 2/2 = app + proxy
linkerd viz -n shop edges deploy                              # SECURED √ = mTLS
```

**Try it now:**
1. Install Linkerd on kind and mesh a namespace with a client and a server.
2. Confirm `SECURED √`.

**You've got it if you can:** say what a mesh adds beyond NetworkPolicies, and how a sidecar intercepts traffic.

> 📖 **That's the core.** On a first pass, do the mini-lab and move on to the next topic. Everything below is the **full reference**: come back for the details, the edge cases, interview answers and exam-level depth.

---

## What it is

A **service mesh** is an infrastructure layer that handles **communication between services** inside (and across) clusters. A **data plane** of proxies sits in the path of every request between workloads. A **control plane** configures those proxies, issues their certificates and collects their telemetry.

What it gives you, **without changing application code**:
- **Security**: automatic **mutual TLS** (both sides prove identity with certificates) between workloads; **identity-based authorization** ("service `orders` may call `payments` `POST /charge`").
- **Reliability**: retries, timeouts, circuit breaking, outlier detection, load balancing by latency.
- **Traffic management**: canary and weighted routing, header-based routing, mirroring, fault injection.
- **Observability**: per-service request rate, error rate and latency ("golden signals"), distributed-tracing headers, a live service graph.

---

## Why it exists

In a microservices system, every service needs encryption, retries, timeouts, metrics and access control on every call. Building that into each service (in different languages) is repetitive and inconsistent. Kubernetes networking alone is **L3/L4**: NetworkPolicies allow or deny IPs and ports (topic 61), but they can't say "only `orders` with a valid identity may `POST /charge`", can't encrypt, and give no per-request metrics. Zero-trust requirements (encrypt and authenticate everything inside the network) push teams to a mesh.

---

## Analogy

Without a mesh, every department in a company sends its own couriers, with their own rules about envelopes, retries and logbooks. A mesh hires **one professional courier service**: a courier stands at every office door (the proxy), checks the sender's and receiver's company badges (mTLS identities), follows central rules on who may send what (authorization policies), retries failed deliveries, and logs every delivery for the operations dashboard. Headquarters (the control plane) issues the badges and the rulebook.

---

## How it works inside

### Data plane designs

**1. Sidecar proxies** (classic Istio, Linkerd)
- A proxy container is **injected** into every Pod by a **mutating admission webhook** (topic 58), e.g. Istio adds `istio-proxy` (Envoy) when the namespace is labelled `istio-injection=enabled`; Linkerd adds `linkerd-proxy` (a Rust micro-proxy) with the annotation `linkerd.io/inject: enabled`.
- An init step (an init container or a CNI plugin) installs **iptables rules in the Pod's network namespace** (topic 04) that **redirect all inbound and outbound TCP traffic** through the proxy. The app still thinks it talks plain HTTP to `payments:80`; the proxies upgrade the hop to **mTLS**.
- Now implemented as **native sidecars** (init containers with `restartPolicy: Always`, GA in v1.33; topic 23), which fixes start-up and Job-completion ordering problems.
- Cost: an extra proxy (memory/CPU) per Pod, extra latency per hop, and Pods must restart to get injected/upgraded.

**2. Sidecarless**
- **Istio ambient mode** (GA): a per-node **ztunnel** (a Rust L4 proxy) handles mTLS and L4 policy for all Pods on the node; optional per-namespace/service **waypoint proxies** (Envoy) add L7 features only where needed. No Pod restarts, much lower overhead.
- **Cilium service mesh**: eBPF in the kernel for L3/L4 plus a per-node Envoy for L7; mutual authentication via SPIFFE/SPIRE or WireGuard/IPsec encryption.

### Identity and mTLS

- Each workload's identity = its **ServiceAccount** (topic 55), expressed as a **SPIFFE ID**: `spiffe://cluster.local/ns/shop/sa/orders`.
- The control plane runs a **CA** (Istio: `istiod`; Linkerd: the identity controller with a trust anchor you provide) and issues **short-lived certificates** (hours) to each proxy, rotated automatically. The proxy proves its SA token to get them.
- Every hop: client proxy ⇄ server proxy **mTLS**; authorization policies are checked against the **peer identity** (not IPs, which change).

### Control plane

- **Istio**: `istiod` (config distribution to Envoys via the xDS APIs, the CA, sidecar injection webhook). Configured with CRDs: `PeerAuthentication` (mTLS mode), `AuthorizationPolicy`, `VirtualService`/`DestinationRule` (classic traffic API), or **Gateway API** routes; `Telemetry`.
- **Linkerd**: destination, identity and proxy-injector controllers; famously simple, mTLS on by default; `Server`/`AuthorizationPolicy` CRDs and Gateway API `HTTPRoute` for traffic.

### Gateway API for meshes (GAMMA)

The same Route types configure **east-west** traffic: attach an `HTTPRoute` to a **Service** (`parentRefs: [{kind: Service, name: payments}]`) and its rules (weights, header matches, timeouts) apply to calls **to that Service** inside the mesh. Istio, Linkerd and Cilium implement it, so one API covers north-south (Gateways, topic 72) and east-west.

### When not to use a mesh

- A handful of services, low security requirements: the operational cost (upgrades, debugging another layer, CRDs, resource use) may outweigh the benefits.
- Only need encryption? CNI-level encryption (Cilium WireGuard, Calico WireGuard) may be enough.
- Only need ingress features? A Gateway is enough.

---

## Commands / YAML

### Linkerd quick start (on kind)

```bash
curl -sL https://run.linkerd.io/install-edge | sh       # CLI (edge release; read the script first)
export PATH=$HOME/.linkerd2/bin:$PATH
linkerd check --pre
linkerd install --crds | kubectl apply -f -
linkerd install | kubectl apply -f -
linkerd check
```

```bash
kubectl create ns shop
kubectl annotate ns shop linkerd.io/inject=enabled
kubectl -n shop create deployment web --image=hashicorp/http-echo:1.0 -- /http-echo -text=hello -listen=:5678
kubectl -n shop expose deploy web --port=80 --target-port=5678
kubectl -n shop run client --image=curlimages/curl:8.10.1 --command -- sleep 3600
kubectl -n shop get pods
```

```
NAME                   READY   STATUS    RESTARTS   AGE
client                 2/2     Running   0          20s
web-5d7c9b8f6c-7xkqp   2/2     Running   0          25s
```

`2/2`: app container + injected `linkerd-proxy` (as a native sidecar, it appears among init containers in the spec).

```bash
kubectl -n shop exec client -c client -- curl -s web
linkerd viz install | kubectl apply -f - && linkerd viz check
linkerd viz -n shop edges deploy
```

```
SRC      DST   SRC_NS   DST_NS   SECURED
client   web   shop     shop     √
```

`SECURED √` = the call was **mTLS**, with no app change.

### Istio: strict mTLS and an authorization policy

```bash
istioctl install --set profile=ambient -y        # or 'default' for sidecar mode
kubectl label ns shop istio.io/dataplane-mode=ambient
```

```yaml
apiVersion: security.istio.io/v1
kind: PeerAuthentication
metadata: { name: default, namespace: shop }
spec:
  mtls: { mode: STRICT }                     # reject plaintext to workloads in 'shop'
---
apiVersion: security.istio.io/v1
kind: AuthorizationPolicy
metadata: { name: payments-allow-orders, namespace: shop }
spec:
  selector: { matchLabels: { app: payments } }
  action: ALLOW
  rules:
  - from: [{ source: { principals: ["cluster.local/ns/shop/sa/orders"] } }]   # identity, not IP
    to:   [{ operation: { methods: ["POST"], paths: ["/charge"] } }]           # L7 rule (needs a waypoint in ambient)
```

### Canary with Gateway API (GAMMA)

```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata: { name: payments-canary, namespace: shop }
spec:
  parentRefs: [{ group: "", kind: Service, name: payments, port: 80 }]   # east-west: the Service itself
  rules:
  - backendRefs:
    - { name: payments-v1, port: 80, weight: 95 }
    - { name: payments-v2, port: 80, weight: 5 }
    timeouts: { request: 2s }
```

---

## Hands-on lab

1. Install Linkerd on kind; mesh a namespace; confirm `2/2` Pods and `SECURED √` edges.
2. Use `linkerd viz stat deploy -n shop` and `linkerd viz top` to see live success rates and latencies.
3. Make one Pod fail half its requests (http-echo with a failing variant) and add a retry/timeout config; compare success rates.
4. Install Istio ambient on a fresh cluster; apply STRICT `PeerAuthentication`; try plaintext from an un-meshed namespace (fails) and from a meshed one (works).
5. Add the AuthorizationPolicy and test as `orders` (allowed) vs another ServiceAccount (`RBAC: access denied`, 403).
6. Do a 95/5 canary with an HTTPRoute attached to a Service; count responses.
7. Clean up (`linkerd uninstall | kubectl delete -f -`; `istioctl uninstall --purge`).

---

## Where you'll use it

- **Zero-trust and compliance** (encryption in transit everywhere, PCI DSS, HIPAA).
- **Large microservice platforms**: uniform retries/timeouts, golden signals, dependency maps.
- **Progressive delivery** (Flagger/Argo Rollouts with mesh traffic splitting).
- **Multi-cluster** service discovery and failover.

---

## When it breaks

| Symptom | Cause | Fix |
|---|---|---|
| Pods `1/2` / proxy not ready | Proxy can't reach the control plane, or certificate issues | `istioctl proxy-status` / `linkerd check --proxy` |
| Connections reset after enabling STRICT mTLS | Clients outside the mesh send plaintext | Mesh the clients, or use PERMISSIVE during migration |
| `403 RBAC: access denied` | AuthorizationPolicy denies (or no ALLOW matches) | Check principals/paths; `istioctl x authz check` |
| Jobs never complete | Old-style sidecar keeps the Pod alive | Use native sidecars (current versions do) |
| App can't reach external services | Egress policy / `REGISTRY_ONLY` mode | Add `ServiceEntry` or allow egress |
| Higher latency/memory | Sidecar per Pod | Ambient/sidecarless, tune proxy resources |
| Non-HTTP protocol broken | Protocol detection guessed wrong | Name Service ports (`tcp-…`, `http-…`) or set `appProtocol` |

---

## Quick recap

- A mesh = **data plane** proxies on every hop + a **control plane** (config, CA, telemetry): mTLS with ServiceAccount-based (SPIFFE) identities, L7 authorization, retries/timeouts, traffic splitting, golden signals, no code changes.
- **Sidecars** (injected by a mutating webhook, iptables redirection, native sidecars) vs **sidecarless** (Istio ambient ztunnel + waypoints, Cilium eBPF).
- Linkerd: simple, mTLS by default. Istio: feature-rich (PeerAuthentication, AuthorizationPolicy, Gateway API). **GAMMA**: HTTPRoutes attached to Services for east-west traffic.
- Adopt for security/observability at scale; skip it when a CNI or a Gateway covers your needs.

### Interview questions

1. **Q: What does a service mesh give you that NetworkPolicies don't?**
   A: Identity-based mutual TLS encryption between workloads, L7 authorization (methods, paths) tied to workload identity rather than IPs, plus retries/timeouts, traffic splitting and per-request telemetry; NetworkPolicies are L3/L4 allow-lists only.
2. **Q: How does a sidecar intercept traffic without code changes?**
   A: A mutating webhook injects the proxy container; an init step (or CNI plugin) adds iptables rules in the Pod's network namespace that redirect inbound and outbound traffic to the proxy, which then originates/terminates mTLS.
3. **Q: What's Istio ambient mode and why use it?**
   A: A sidecarless data plane: a per-node ztunnel provides mTLS and L4 policy, and optional waypoint proxies add L7 features. It avoids per-Pod sidecar overhead and Pod restarts for injection or upgrades.

---

## Appendix: notes on the original course material

This topic was rebuilt from the original CKA course notes. This appendix keeps the audit trail out of your way while you learn.

### Sources

none: a **new topic**. It builds on mentions of Istio sidecar injection via mutating webhooks (Day 38, topic 58), the Istio ingress gateway (Day 49, topic 69), Istio/Cilium as networking operators (Day 40, topic 67) and NetworkPolicies being L3/L4 only (Day 47, topic 61).
