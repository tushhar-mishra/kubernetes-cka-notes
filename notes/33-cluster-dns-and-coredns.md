# 33 · Cluster DNS and CoreDNS

> **Level:** ★★ intermediate
>
> **You will learn:** how names like `backend-svc` become IPs inside a cluster; CoreDNS's Deployment and Service; the DNS names Kubernetes creates for Services, headless Services, StatefulSet Pods and Pod IPs; a Pod's `/etc/resolv.conf` (nameserver, search list, **`ndots:5`**) and why it affects performance; `/etc/hosts` and nsswitch; the Corefile and its plugins; `dnsPolicy` and `dnsConfig`; NodeLocal DNSCache; and a full DNS troubleshooting checklist.
>
> **You need:** topics 04 (resolv.conf), 26 and 30–32. Your Linux notes ([`063`](../../linux-mastery-rhel/notes/063-name-resolution-and-dns-client.md) name resolution, [`081`](../../linux-mastery-rhel/notes/081-dns-server-bind.md) DNS server) and Docker notes ([`29`](../../Docker/notes/29-container-dns.md) container DNS) cover DNS basics.

---

## ⚡ Core: the 10-minute version

**The idea.** Inside a cluster, apps find each other by **name**. **CoreDNS** (Pods in `kube-system`, behind the `kube-dns` Service) answers:
- `<service>.<namespace>.svc.cluster.local` → the Service's IP;
- headless Services → the individual Pod IPs.

Every Pod's `/etc/resolv.conf` points at CoreDNS and has a **search list** starting with its own namespace. That's why `web` works from the same namespace, but you need `web.other-ns` from elsewhere.

**Picture it.** CoreDNS is the company phone directory; the search list means you can dial an internal extension without the full number.

**The one thing to know.**

```bash
kubectl run t --rm -it --image=busybox:1.37 --restart=Never -- sh
/ # cat /etc/resolv.conf
/ # nslookup kubernetes.default
/ # nslookup web.default.svc.cluster.local
```

**Try it now:**
1. Do the lookups above.
2. Look up a Service in another namespace with and without the namespace part.
3. Scale CoreDNS to 0 (`kubectl -n kube-system scale deploy coredns --replicas=0`), watch lookups fail, then scale it back to 2.

**You've got it if you can:** build a Service's full DNS name, and explain why short names only work in the same namespace.

> 📖 **That's the core.** On a first pass, do the mini-lab and move on to the next topic. Everything below is the **full reference**: come back for the details, the edge cases, interview answers and exam-level depth.

---

## What it is

Kubernetes clusters run an internal **DNS server** so workloads find each other **by name** instead of by changing IPs. **CoreDNS** is that server: the default since v1.13, replacing the older **kube-dns**.

**Key characteristics:**
- Runs as a **Deployment** in `kube-system`, usually with **2+ replicas** for high availability.
- Exposed through a **ClusterIP Service named `kube-dns`** (the old name was kept for compatibility), usually at **`10.96.0.10`**.
- **Plugin-based**: the **`kubernetes`** plugin answers for Services and Pods from the API server's data; **`forward`** sends everything else (e.g. `google.com`) to upstream resolvers; other plugins add caching and metrics.

> ℹ️ The legacy **kube-dns** add-on is formally **deprecated in v1.37** (⚠️ *verify in the v1.37 release notes before quoting*). If an old cluster still runs it, migrate to CoreDNS (kubeadm has used CoreDNS by default for years).

---

## Why it exists

Topic 30 gave every Service a stable virtual IP. But apps want **names**: `http://orders:8080`, not `http://10.96.41.120:8080`. Names also work across clusters and environments without code changes. DNS is the standard, language-neutral way to do service discovery, so Kubernetes provides it out of the box.

---

## Analogy

CoreDNS is the cluster's **phone directory desk**. Every Service is listed as `<name>.<department>.svc.cluster.local`. When you ask for just "orders", the desk assumes you mean **your own department** first (the search list). For numbers outside the company (`google.com`), it calls the **outside directory service** (upstream DNS) for you.

---

## How it works inside

### The CoreDNS objects

```bash
kubectl get deploy -n kube-system coredns
kubectl get svc -n kube-system kube-dns
```

```
NAME      READY   UP-TO-DATE   AVAILABLE   AGE
coredns   2/2     2            2           82s

NAME       TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)                  AGE
kube-dns   ClusterIP   10.96.0.10   <none>        53/UDP,53/TCP,9153/TCP   3m57s
```

| Port | Protocol | Purpose |
|---|---|---|
| `53` | UDP/TCP | DNS queries (TCP for large answers and retries) |
| `9153` | TCP | Prometheus metrics: request counts, response codes, cache hits, latency |

### How Pods know to use it

The **kubelet** writes each Pod's `/etc/resolv.conf` from its config (`clusterDNS: [10.96.0.10]`, `clusterDomain: cluster.local`; topic 08). So every DNS query from a Pod goes to the `kube-dns` ClusterIP, which kube-proxy DNATs to a CoreDNS Pod (topic 32). CoreDNS **watches** Services, EndpointSlices and (optionally) Pods through the API server and answers from memory.

### The names Kubernetes creates

Kubernetes automatically registers **A records** (name → IPv4; AAAA for IPv6) for Services. The hierarchy:

| Level | Component | Service example | Pod-IP example | Meaning |
|---|---|---|---|---|
| Cluster domain | `cluster.local` | `cluster.local` | `cluster.local` | The cluster's private top domain (set by `clusterDomain`) |
| Type | `svc` / `pod` | `svc` | `pod` | Record for a Service or a Pod |
| Namespace | `<namespace>` | `default` | `default` | The object's namespace |
| Name | `<resource>` | `nginx-svc` | `10-244-44-55` | Service name, or the Pod IP with dashes |

| Resource | FQDN pattern | Example | Resolves to |
|---|---|---|---|
| **Service** (ClusterIP) | `<svc>.<ns>.svc.cluster.local` | `nginx-svc.default.svc.cluster.local` | The ClusterIP |
| **Headless Service** | `<svc>.<ns>.svc.cluster.local` | `headless-svc.app1-ns.svc.cluster.local` | **All ready Pod IPs** (several A records) |
| **StatefulSet Pod** (via headless Service) | `<pod>.<svc>.<ns>.svc.cluster.local` | `pod-0.headless-svc.app1-ns.svc.cluster.local` | That Pod's IP (stable name; topic 40) |
| **Pod by IP** (`pods` option) | `<a-b-c-d>.<ns>.pod.cluster.local` | `10-244-2-10.app1-ns.pod.cluster.local` | `10.244.2.10` |
| **Named port** (SRV) | `_<port>._<proto>.<svc>.<ns>.svc.cluster.local` | `_http._tcp.nginx-svc.default.svc.cluster.local` | Port number + target name |
| **ExternalName** | `<svc>.<ns>.svc.cluster.local` | `cka-db-svc.default.svc…` | CNAME to the external name (topic 31) |

**FQDN** = **F**ully **Q**ualified **D**omain **N**ame: the complete name. A **trailing dot** (`nginx-svc.default.svc.cluster.local.`) marks it as absolute (no search domains added); it's usually omitted on the command line but present in DNS messages.

> ✅ **Correct today** ([C1](#c1)): a headless Service name **does** have A/AAAA records: **one per ready Pod** (that's how clients discover the members). It has no **ClusterIP**, and it additionally gets **SRV** records for named ports. The per-Pod names (`pod-0.headless-svc…`) exist for Pods that set `hostname`/`subdomain` (which StatefulSets do automatically).

**Kubernetes DNS ≠ public DNS:** it follows normal DNS ideas (hierarchy, FQDNs), but the internal zone's effective root is **`cluster.local`**, with Kubernetes-specific subdomains `svc` and `pod`. It's a separate, private naming system that only exists inside the cluster.

**How CoreDNS resolves `nginx-svc`:** the Pod's resolver expands the short name with search domains (below) into `nginx-svc.default.svc.cluster.local.`; CoreDNS's `kubernetes` plugin recognises the `cluster.local` zone, the `svc` type, the `default` namespace and the `nginx-svc` name, and returns its ClusterIP.

### Inside a Pod: `/etc/resolv.conf` (Day 48 + ndots)

```
search default.svc.cluster.local svc.cluster.local cluster.local
nameserver 10.96.0.10
options ndots:5
```

- **`nameserver 10.96.0.10`** = the `kube-dns` Service's ClusterIP. Every lookup goes to CoreDNS.
- **`search …`** = suffixes tried for **short** names, **in order**, starting with the Pod's **own namespace** (`default.svc.cluster.local` here; `app1-ns.svc.cluster.local` for a Pod in `app1-ns`). The node's own search domains are appended after these.
- **`options ndots:5`** = if a name has **fewer than 5 dots**, try the **search list first**, and only then the name as-is.

For `nginx-svc` the resolver tries:
1. `nginx-svc.default.svc.cluster.local` ✅ (first match wins)
2. `nginx-svc.svc.cluster.local`
3. `nginx-svc.cluster.local`

**Why it matters:**
- Short names work **within the Pod's own namespace**; other namespaces need at least `<svc>.<ns>`.
- If the same Service name exists in several namespaces, a short name resolves to the one in **your** namespace.
- For predictable behaviour in multi-namespace systems, use `backend-svc.app1-ns` or the full FQDN.

**The `ndots:5` cost.** `api.github.com` has only 2 dots (< 5), so the resolver first tries `api.github.com.default.svc.cluster.local`, `…svc.cluster.local`, `…cluster.local` (and node search domains), **each for both A and AAAA**: up to ~8 failed queries before the real one. In busy clusters this multiplies DNS load and adds latency. Fixes:
- use a **trailing dot** in configs for external names (`api.github.com.`),
- lower `ndots` for that Pod via `dnsConfig` (below),
- run **NodeLocal DNSCache**.

### `/etc/hosts` and nsswitch

```bash
cat /etc/hosts
```

- Static name → IP mappings, checked **before** DNS (order set by `/etc/nsswitch.conf`: `hosts: files dns`).
- Example: the line `10.244.2.3 www.kubernetes.io` makes the Pod resolve `www.kubernetes.io` to `10.244.2.3` without asking DNS.
- The kubelet manages a Pod's `/etc/hosts` (it includes the Pod's own name and IP). Add entries properly with **`spec.hostAliases`**, not by editing the file (edits are lost when the container restarts).
- Alpine/musl images don't read `nsswitch.conf`, but still check `/etc/hosts` first.

**`nslookup` and `dig` bypass `/etc/hosts`:** they send **raw DNS queries** straight to the `nameserver` in `resolv.conf`, ignoring `/etc/hosts` and nsswitch. So `ping myhost` may succeed (via `/etc/hosts`) while `nslookup myhost` fails (no DNS record). Use `getent hosts myhost` to test what applications actually see.

### The Corefile

CoreDNS's config file is the **`Corefile`**, stored in the **`coredns` ConfigMap** in `kube-system` and mounted into the CoreDNS Pods, so you can change DNS behaviour without rebuilding the image.

```bash
kubectl get cm -n kube-system coredns -o yaml
```

```
.:53 {
    errors
    health {
       lameduck 5s
    }
    ready
    kubernetes cluster.local in-addr.arpa ip6.arpa {
       pods insecure
       fallthrough in-addr.arpa ip6.arpa
       ttl 30
    }
    prometheus :9153
    forward . /etc/resolv.conf {
       max_concurrent 1000
    }
    cache 30 {
       disable success cluster.local
       disable denial cluster.local
    }
    loop
    reload
    loadbalance
}
```

| Directive / plugin | Purpose |
|---|---|
| `.:53` | Main server block: all zones (`.`) on port 53 |
| `errors` | Log errors to stderr |
| `health { lameduck 5s }` | `/health` endpoint (liveness, port 8080); on shutdown keep serving for 5 s so queries drain |
| `ready` | `/ready` endpoint (readiness, port 8181) once all plugins are ready |
| `kubernetes cluster.local in-addr.arpa ip6.arpa` | Answer for the cluster zone, plus **reverse lookups** (IP → name) for IPv4 and IPv6 |
| `pods insecure` | Answer Pod-IP names (`10-244-44-55.default.pod.cluster.local`) **without checking** that such a Pod exists (kube-dns compatibility) |
| `fallthrough in-addr.arpa ip6.arpa` | If a reverse lookup isn't found, pass it to the next plugin instead of failing |
| `ttl 30` | TTL (time to live: how long clients may cache) for cluster records |
| `prometheus :9153` | Metrics endpoint |
| `forward . /etc/resolv.conf { max_concurrent 1000 }` | Send non-cluster queries to the upstream servers listed in the **CoreDNS Pod's** `/etc/resolv.conf` (which comes from the **node**); up to 1000 concurrent upstream queries |
| `cache 30 { disable success/denial cluster.local }` | Cache answers up to 30 s, but **not** for `cluster.local`, so cluster records always reflect current state |
| `loop` | Detect forwarding loops and stop (CoreDNS crashes with a "Loop detected" message) |
| `reload` | Reload automatically when the Corefile changes |
| `loadbalance` | **Shuffle the order of A/AAAA records** in each answer (round-robin DNS) |

> ✅ **Correct today** ([C2](#c2)): it **randomises the order of the records in DNS answers** (so clients don't always pick the first IP). Choosing between upstream servers is the `forward` plugin's own `policy` option (random by default).

**Common Corefile changes:**

```
# send a company domain to internal DNS servers
corp.example.com:53 {
    forward . 10.0.0.53 10.0.0.54
}
# fixed answers
hosts {
    10.20.30.40 legacy.internal
    fallthrough
}
# rewrite a name
rewrite name db.prod.internal mysql.prod.svc.cluster.local
```

### Pod DNS modes in CoreDNS

The `pods` option of the `kubernetes` plugin:

| Mode | Behaviour | Use |
|---|---|---|
| `pods disabled` (plugin default) | No Pod-IP records | When nothing needs them |
| **`pods insecure`** | Answers any `a-b-c-d.<ns>.pod…` name with that IP **without verifying** a Pod exists (kube-dns compatible; can be spoofed or stale) | **kubeadm/kind default Corefile**; not recommended where it matters |
| **`pods verified`** | Answers only if a Pod with that IP exists **in that namespace** (CoreDNS watches Pods, so more memory) | Recommended when you need Pod-IP records |

> ✅ **Correct today** ([C3](#c3)): kubeadm and kind Corefiles **do** include `pods insecure`, so `10-244-2-10.default.pod.cluster.local` **resolves** on them. What doesn't exist is a record for the **Pod's name** (`debug-pod.default.pod.cluster.local`): Pod-IP records use the **dashed IP**, never the Pod name. (2) The third mode was given as "`pods hostname`", which isn't a CoreDNS option. Per-Pod **names** come from setting a Pod's **`hostname` and `subdomain`** to match a headless Service (StatefulSets do this automatically), giving `<hostname>.<subdomain>.<ns>.svc.cluster.local`.

### `dnsPolicy` and `dnsConfig`

| `dnsPolicy` | Effect |
|---|---|
| `ClusterFirst` (the default for Pods) | Use CoreDNS; non-cluster names are forwarded upstream by CoreDNS |
| `ClusterFirstWithHostNet` | Required for `hostNetwork: true` Pods that still want cluster DNS |
| `Default` | Inherit the **node's** resolv.conf (no cluster names!). Despite its name, **not** the default |
| `None` | Ignore everything; use only `dnsConfig` |

```yaml
spec:
  dnsPolicy: ClusterFirst
  dnsConfig:
    options:
    - name: ndots
      value: "2"              # fewer pointless search-list lookups for external names
    - name: single-request-reopen   # glibc workaround for the A/AAAA conntrack race
    searches: [corp.example.com]
  hostAliases:
  - ip: "10.20.30.40"
    hostnames: ["legacy.internal"]
```

### NodeLocal DNSCache

An add-on DaemonSet that runs a **DNS cache on every node** at a link-local IP (`169.254.20.10`). Pods query their own node's cache: lower latency, far fewer queries reaching CoreDNS, uses TCP towards CoreDNS (avoiding the UDP conntrack race that caused "5-second DNS delays", topic 32), and keeps working through brief CoreDNS hiccups. Common in large production clusters.

---

## Commands / YAML

### DNS from inside a Pod

```bash
kubectl run debug-pod -it --image=nicolaka/netshoot --rm -- bash
```

(`nicolaka/netshoot` is a troubleshooting image with `dig`, `nslookup`, `curl`, `ping`, `telnet`, `tcpdump` and more.)

```bash
nslookup debug-pod
```

```
Server:		10.96.0.10
Address:	10.96.0.10#53

** server can't find debug-pod: NXDOMAIN
```

- **NXDOMAIN** = "this name doesn't exist". Pods aren't registered **by name**.

```bash
nslookup kubernetes.io        # external names work through the forward plugin
cat /etc/resolv.conf
```

### Same-namespace and cross-namespace lookups

```bash
kubectl create deployment nginx-deploy --image=nginx
kubectl expose deployment nginx-deploy --name=nginx-svc --port=80
kubectl get pods,svc
```

From a debug Pod in `default`:

```bash
nslookup nginx-svc
```

```
Server:  10.96.0.10
Name:    nginx-svc.default.svc.cluster.local
Address: 10.96.48.49
```

Works because the Pod and Service are both in `default`.

Another namespace:

```bash
kubectl create ns app1-ns
kubectl create deployment app1-deploy --image=nginx -n app1-ns
kubectl expose deployment app1-deploy --name=app1-svc --port=80 -n app1-ns
```

From the `default` debug Pod:

```bash
nslookup app1-svc.app1-ns      # ✅ app1-svc.app1-ns.svc.cluster.local → 10.96.x.x
nslookup app1-svc              # ❌ NXDOMAIN
```

**Why `nslookup app1-svc` fails:** the search list from a `default` Pod tries `app1-svc.default.svc.cluster.local`, `app1-svc.svc.cluster.local`, `app1-svc.cluster.local`, and none of these exist. **Key takeaway:** across namespaces, give at least `<service>.<namespace>`; the search domain `svc.cluster.local` completes it.

From a Pod **inside** `app1-ns`, the short name works, because its search list starts with `app1-ns.svc.cluster.local`:

```bash
kubectl run debug-pod-app1 -n app1-ns -it --image=nicolaka/netshoot --rm -- bash
nslookup app1-svc
```

### dig for detail

```bash
dig +short nginx-svc.default.svc.cluster.local
dig +search +short nginx-svc            # use the search list (dig doesn't by default)
dig SRV _http._tcp.nginx-svc.default.svc.cluster.local +short
dig -x 10.96.48.49 +short               # reverse lookup
```

```
10.96.48.49
10.96.48.49
0 100 80 nginx-svc.default.svc.cluster.local.
nginx-svc.default.svc.cluster.local.
```

---

## Hands-on lab

1. Run every Day 48 command above.
2. Compare `/etc/resolv.conf` in Pods in two namespaces.
3. Pod-IP names: `nslookup 10-244-1-5.default.pod.cluster.local` (use a real Pod IP). It resolves because of `pods insecure`. Now try a made-up IP: it also "resolves" (that's the insecurity).
4. Headless: create `kubectl expose deploy nginx-deploy --name=nginx-hl --cluster-ip=None --port=80` and `dig +short nginx-hl.default.svc.cluster.local`: one IP per Pod.
5. ndots cost: in the debug Pod, run `tcpdump -ni eth0 port 53 &` and then `curl -s -o /dev/null https://api.github.com`. Count the queries. Repeat with `api.github.com.` (trailing dot).
6. `/etc/hosts` vs DNS: add `hostAliases` for `legacy.internal` to a Pod; `getent hosts legacy.internal` works, `nslookup legacy.internal` fails.
7. Edit the Corefile to add a `hosts` block for `printer.lab → 10.1.2.3` with `fallthrough`; wait for `reload`, then resolve it.
8. Clean up the Deployments, Services and namespace.

---

## Where you'll use it

- **Service discovery** for every microservice; cross-namespace calls via `<svc>.<ns>`.
- **Hybrid networks**: forwarding corporate domains to internal DNS servers in the Corefile.
- **Performance tuning**: `ndots`, NodeLocal DNSCache, CoreDNS replicas/autoscaling for large clusters.
- **StatefulSets**: stable per-Pod names for databases and brokers (topic 40).

---

## When it breaks

### DNS failures (Day 58 #4)

**Problem:** Pods report **"no such host"**; Service names don't resolve.

**Likely causes:**
- CoreDNS Pods down or crashing.
- The kubelet's cluster DNS IP is wrong, so Pods' `/etc/resolv.conf` points at a bad IP.
- A NetworkPolicy blocks DNS, or CoreDNS can't reach its upstream servers.

**Checks:**

```bash
kubectl -n kube-system get pods -l k8s-app=kube-dns -o wide     # CoreDNS healthy?
kubectl -n kube-system logs -l k8s-app=kube-dns --tail=80       # loop / forward errors?
# from a test Pod:
nslookup kubernetes.default.svc.cluster.local 10.96.0.10        # ask the DNS IP directly
cat /etc/resolv.conf                                            # search/domain/options in the Pod
kubectl get svc -n kube-system kube-dns -o wide                 # the DNS ClusterIP
```

**Fix:** restart/fix CoreDNS; correct the cluster DNS IP in the kubelet config; remove blocking NetworkPolicies; make sure upstream resolvers are reachable.

**Example:** a Pod's `resolv.conf` shows the wrong nameserver → set the kubelet's cluster DNS to `10.96.0.10` (`clusterDNS` in `/var/lib/kubelet/config.yaml`, or the old `--cluster-dns` flag), restart the kubelet, recreate the Pod → lookups work.

### Full troubleshooting checklist

1. **Is CoreDNS running?** `kubectl get pods -n kube-system -l k8s-app=kube-dns`: Running, READY `1/1` per Pod.
2. **CoreDNS logs:** `kubectl logs -n kube-system -l k8s-app=kube-dns --tail=100`: loop errors, forward failures, `i/o timeout` to upstreams.
3. **Test from a Pod:** `kubectl run dns-test --rm -it --image=nicolaka/netshoot -- bash`, then `nslookup kubernetes.default` and `dig nginx-svc.default.svc.cluster.local`. Failures = in-cluster DNS broken.
4. **Pod's `resolv.conf`:** nameserver = the cluster DNS IP (usually `10.96.0.10` on kubeadm), correct search list.
5. **`nsswitch.conf`:** should contain `hosts: files dns` (files = `/etc/hosts` first, then DNS).
6. **Corefile:** the `kubernetes` plugin has the right domain (`cluster.local`) and options; `kubectl get configmap coredns -n kube-system -o yaml`.
7. **NetworkPolicies / firewalls:** UDP **and TCP** 53 to the CoreDNS Pods must be allowed (a default-deny egress policy without a DNS rule is the classic cause; topic 61).
8. **kubelet DNS settings:** `clusterDNS` and `clusterDomain` in the kubelet config (custom clusters).
9. **External names failing too?** Check `forward . /etc/resolv.conf` and that the **CoreDNS Pods'** `/etc/resolv.conf` (from the node) has working upstream resolvers.

> ✅ **Correct today** ([C4](#c4)): each CoreDNS Pod has **one** container, so each shows **`1/1`**; it's the **Deployment** that shows `2/2` (two replicas).

| Extra symptom | Cause | Fix |
|---|---|---|
| CoreDNS `CrashLoopBackOff`, logs say `Loop … detected` | Node's resolv.conf points at a local stub (`127.0.0.53`, systemd-resolved), so CoreDNS forwards to itself | Point the kubelet at the real file (`resolvConf: /run/systemd/resolve/resolv.conf`) or forward to explicit upstream IPs |
| External lookups slow (seconds) | `ndots:5` search expansion; upstream slow; conntrack race | Trailing dots, lower ndots, NodeLocal DNSCache, more CoreDNS replicas |
| Works by IP, fails by name only in one namespace | Wrong namespace / short name | `<svc>.<ns>` |
| `hostNetwork` Pod can't resolve Services | `dnsPolicy` defaults to the node's DNS | `dnsPolicy: ClusterFirstWithHostNet` |

---

## Quick recap

- CoreDNS (Deployment, `kube-system`) behind the `kube-dns` Service (`10.96.0.10`); the `kubernetes` plugin answers cluster names, `forward` handles the rest.
- Names: `<svc>.<ns>.svc.cluster.local` (ClusterIP); headless → all Pod IPs; StatefulSet Pods `<pod>.<svc>.<ns>.svc…`; Pod IPs `a-b-c-d.<ns>.pod…` (with `pods insecure|verified`).
- Pod `resolv.conf`: nameserver CoreDNS, search list starting with the Pod's namespace, `ndots:5`. Short names work within the same namespace only.
- `/etc/hosts` is checked first by apps but bypassed by nslookup/dig; use `hostAliases`.
- Debug: CoreDNS Pods/logs → test from a Pod → resolv.conf → Corefile → NetworkPolicy port 53 → kubelet `clusterDNS` → upstreams.

### Interview questions

1. **Q: How does a Pod resolve `backend` to an IP?**
   A: Its resolv.conf points at the kube-dns ClusterIP; the resolver appends search domains starting with `<ns>.svc.cluster.local`; CoreDNS's kubernetes plugin returns the Service's ClusterIP (or Pod IPs for a headless Service).
2. **Q: What does `ndots:5` do and why can it hurt?**
   A: Names with fewer than 5 dots are tried with every search domain first, so external names generate several failed queries (×A/AAAA) before the real one. That adds latency and load; mitigate with trailing dots, lower ndots, or NodeLocal DNSCache.
3. **Q: DNS works for external names but not for Services. Where do you look?**
   A: CoreDNS's kubernetes plugin config (domain), the Pod's search list and namespace, and whether the Service actually exists with that name in that namespace.

---

## Appendix: notes on the original course material

This topic was rebuilt from the original CKA course notes. This appendix keeps the audit trail out of your way while you learn.

### Sources

Day 48 (introduction, CoreDNS key characteristics, inspecting the Deployment/Service, ports, DNS mapping, FQDN patterns, DNS breakdown table, how CoreDNS resolves names, "Kubernetes DNS ≠ public DNS", explanation by type, records table and notes, DNS configuration inside a Pod, resolv.conf and search order, /etc/hosts vs resolv.conf, nslookup/dig bypass /etc/hosts, verifying records in one and across namespaces, Corefile, plugin reference, Pod DNS resolution modes, troubleshooting checklist 1–9, conclusion), Day 58 (scenario 4: DNS failures).

### Corrections to the original notes

Each entry shows what the original said and what's true now. In the topic above, each correction appears as a ✅ **Correct today** line or a ⚠️ **Fix to the original** box, linked to its entry here.

<a id="c1"></a>**C1 · Corrected (Day 48).** the records table says a headless Service has DNS type "**NONE** … *No cluster IP* (uses **SRV**)", and the notes say "headless Services do **not** have an A record for the service name". **Now:** a headless Service name **does** have A/AAAA records: **one per ready Pod** (that's how clients discover the members). It has no **ClusterIP**, and it additionally gets **SRV** records for named ports. The per-Pod names (`pod-0.headless-svc…`) exist for Pods that set `hostname`/`subdomain` (which StatefulSets do automatically).

<a id="c2"></a>**C2 · Corrected (Day 48).** `loadbalance` was described as "randomizes the order of **upstream nameservers**". **Now:** it **randomises the order of the records in DNS answers** (so clients don't always pick the first IP). Choosing between upstream servers is the `forward` plugin's own `policy` option (random by default).

<a id="c3"></a>**C3 · Corrected (Day 48).** (1) the notes say "most default clusters **do not enable** the `pods` plugin" but then show the default Corefile with `pods insecure`. **Now:** kubeadm and kind Corefiles **do** include `pods insecure`, so `10-244-2-10.default.pod.cluster.local` **resolves** on them. What doesn't exist is a record for the **Pod's name** (`debug-pod.default.pod.cluster.local`): Pod-IP records use the **dashed IP**, never the Pod name. (2) The third mode was given as "`pods hostname`", which isn't a CoreDNS option. Per-Pod **names** come from setting a Pod's **`hostname` and `subdomain`** to match a headless Service (StatefulSets do this automatically), giving `<hostname>.<subdomain>.<ns>.svc.cluster.local`.

<a id="c4"></a>**C4 · Corrected (Day 48).** the checklist expects CoreDNS Pods to show `READY 2/2`. **Now:** each CoreDNS Pod has **one** container, so each shows **`1/1`**; it's the **Deployment** that shows `2/2` (two replicas).
