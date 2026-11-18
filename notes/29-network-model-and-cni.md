# 29 · The Kubernetes Network Model and CNI: How Pods Get IPs and Talk Across Nodes

> **Level:** ★★★ advanced
>
> **You will learn:** the three rules of the Kubernetes network model; the address ranges a cluster uses (node IPs, Pod CIDR, Service CIDR); what happens, step by step, when a Pod gets its IP (kubelet → runtime → CNI ADD → veth + IPAM + routes); how a packet travels Pod-to-Pod **on the same node** and **across nodes** (routed vs overlay/VXLAN vs cloud-native); how Calico, Cilium, Flannel and the AWS VPC CNI differ; MTU; and how to debug "Pods have no IP" and cross-node failures.
>
> **You need:** topics 04 (veth, bridges, routes, VXLAN), 08, 09 and 18.

---

## ⚡ Core: the 10-minute version

**The idea.** Kubernetes networking has three rules:
1. Every Pod gets its own IP.
2. Every Pod can reach every other Pod **without NAT**.
3. Node agents can reach the Pods on their node.

Kubernetes doesn't implement this itself: a **CNI plugin** does (Calico, Cilium, kindnet). When a Pod starts, the runtime calls the plugin, which creates a virtual cable (a veth pair) into the Pod, assigns an IP from the node's Pod range, and sets routes. Between nodes, traffic is either **routed** or wrapped in an **overlay** (VXLAN).

**Picture it.** Every apartment (Pod) gets a direct phone number that works from any apartment in the city, without going through a receptionist.

**The one thing to know.** The three address ranges must not overlap:

```bash
kubectl get nodes -o wide                                   # node IPs
kubectl get pods -A -o wide | head                          # Pod IPs (Pod CIDR)
kubectl get svc kubernetes                                  # Service IP (Service CIDR)
docker exec lab-worker ls /etc/cni/net.d                    # the CNI config the runtime uses
```

**Try it now:**
1. Run two Pods on different nodes (check `-o wide`).
2. `kubectl exec` into one and `ping` the other's IP.
3. Find the veth interfaces on a node with `docker exec lab-worker ip link | grep veth`.

**You've got it if you can:** state the three rules, and say what the CNI plugin does when a Pod is created.

> 📖 **That's the core.** On a first pass, do the mini-lab and move on to the next topic. Everything below is the **full reference**: come back for the details, the edge cases, interview answers and exam-level depth.

---

## What it is

The **Kubernetes network model** is a short set of rules every cluster must follow. Kubernetes itself **doesn't implement them**; a **CNI plugin** does (topic 09).

**The rules:**
1. **Every Pod gets its own IP address** (shared by its containers).
2. **Every Pod can reach every other Pod by that IP, on any node, without NAT** (Network Address Translation: rewriting addresses). The IP a Pod sees for itself is the IP others see.
3. **Agents on a node** (kubelet, system daemons) **can reach all Pods on that node.**

Result: a **flat network**. Pods behave like VMs on one big LAN. Apps don't need port mapping (`docker run -p`) or to know which node a peer runs on.

**CNI's role:**
- **Network setup:** gives every Pod a **unique IP** in the cluster and connects it.
- **Traffic routing:** makes sure packets reach Pods **on the same node or on other nodes**, implementing the routes and rules across the cluster's network.
- **Pod-to-Pod communication:** including the traffic that kube-proxy has already redirected from a Service IP to a Pod IP (topic 32).

---

## Why it exists

Docker's default networking (bridge + NAT + published ports, your Docker notes topics [26](../../Docker/notes/26-linux-networking-recap.md)–[28](../../Docker/notes/28-publishing-ports.md)) works on **one host**. With hundreds of nodes, port mapping and NAT become unmanageable: port clashes, apps advertising addresses others can't reach, and no way to know where a peer lives. Kubernetes avoids all that by requiring a flat, routable Pod network, and leaves **how** to build it to pluggable CNIs, so it works on laptops, bare metal and every cloud.

---

## Analogy

A **company with offices in many cities** (nodes). Every employee (Pod) gets a **personal extension number** (Pod IP) that works **from anywhere**: dial it from any office and you reach them directly. How calls travel between cities (leased lines, VPN tunnels, the public phone network) is the **telecom provider's** business (the CNI). Employees just dial.

---

## How it works inside

### The three address ranges

| Range | Example (kind defaults) | Who uses it | Who assigns it |
|---|---|---|---|
| **Node network** | `172.18.0.0/16` (the Docker `kind` network) | Nodes' real interfaces | Your infrastructure (VPC, DHCP, Docker) |
| **Pod CIDR** (cluster-wide) | `10.244.0.0/16`, split per node: `10.244.0.0/24`, `10.244.1.0/24`… | Pod IPs | Controller manager (`--cluster-cidr`, `node.spec.podCIDR`) and/or the CNI's own IPAM |
| **Service CIDR** | `10.96.0.0/16` (12 on kubeadm) | ClusterIPs (virtual; topic 30) | API server (`--service-cluster-ip-range`) |

**CIDR** (Classless Inter-Domain Routing) notation: `10.244.1.0/24` = the 256 addresses `10.244.1.0`–`10.244.1.255` (the first 24 bits are fixed). These three ranges **must not overlap** with each other or with networks your Pods need to reach (VPN, on-prem). Overlaps cause mysterious routing failures.

```bash
kubectl get nodes -o custom-columns=NAME:.metadata.name,PODCIDR:.spec.podCIDR
```

```
NAME                PODCIDR
lab-control-plane   10.244.0.0/24
lab-worker          10.244.1.0/24
lab-worker2         10.244.2.0/24
```

### How a Pod gets its IP (CNI ADD)

1. The kubelet asks the runtime (CRI `RunPodSandbox`) to create the Pod sandbox.
2. containerd creates the **pause container** and a new **network namespace** (topic 04).
3. containerd reads the CNI config from `/etc/cni/net.d/` and **executes** the plugin binary in `/opt/cni/bin/` with `CNI_COMMAND=ADD`, the netns path and `CNI_IFNAME=eth0`.
4. The plugin:
   - creates a **veth pair**: one end is moved into the Pod netns and renamed `eth0`, the other stays on the node (`vethXXXX`, `caliXXXX`, `lxcXXXX`);
   - asks **IPAM** for a free IP from **this node's** Pod range;
   - configures `eth0` with that IP and adds a **default route** in the Pod;
   - on the node, adds a **route** to that Pod IP via the veth (or attaches the veth to a bridge);
   - for some plugins, also programs eBPF or NetworkPolicy state.
5. The plugin returns the IP; containerd tells the kubelet; the kubelet writes `status.podIP`.
6. On Pod deletion: `CNI_COMMAND=DEL` removes the veth and frees the IP.

### Same-node traffic

```
Pod A (10.244.1.5) eth0 ──veth──▶ node ──(route 10.244.1.7 dev vethB, or bridge)──▶ veth ──▶ Pod B (10.244.1.7) eth0
```

The packet never leaves the node. It's ordinary Linux routing or bridging between veth pairs: exactly what you built by hand in topic 04's lab.

### Cross-node traffic: three approaches

**1. Routed (no encapsulation).** Each node knows "Pod range `10.244.2.0/24` lives behind node `172.18.0.4`". Routes are installed by the CNI (Calico distributes them with **BGP**; kindnet and Flannel's `host-gw` write static routes), or in a cloud by the **VPC route table**.

```
Pod A 10.244.1.5 → node1: route 10.244.2.0/24 via 172.18.0.4 → network → node2 → vethB → Pod B 10.244.2.9
```

- Fast, no overhead, easy to debug with `tcpdump` (real Pod IPs on the wire).
- Needs the underlying network to deliver packets between nodes **and accept Pod IPs as source/destination** (same L2 segment, BGP-capable routers, or the cloud's anti-spoofing check disabled).

**2. Overlay (encapsulation: VXLAN, IP-in-IP, Geneve).** The node **wraps** the Pod packet inside a normal node-to-node packet:

```
outer: 172.18.0.3 → 172.18.0.4  UDP/4789 (VXLAN)
inner:   10.244.1.5 → 10.244.2.9  TCP/80
```

- Works on any network that can carry node-to-node traffic (the network never sees Pod IPs).
- Costs: per-packet overhead (VXLAN adds 50 bytes), so the Pod **MTU** must be smaller (e.g. 1450 instead of 1500); harder to inspect; firewalls must allow the tunnel (UDP 4789 for VXLAN, IP protocol 4 for IP-in-IP).
- Used by Flannel (default), Calico (VXLAN or IP-in-IP mode), Cilium (tunnel mode).

**3. Cloud-native (VPC IPs).** Pods get **real VPC IPs**: the AWS VPC CNI attaches extra network interfaces/IPs to each EC2 instance (Azure CNI and GKE "VPC-native" work similarly). The cloud network routes Pod IPs natively.

- No overlay; Pods are directly reachable from the VPC (load balancers can target Pod IPs directly; topic 70).
- Consumes VPC IP addresses (subnet exhaustion is a real risk) and has per-instance IP limits (which cap Pods per node on EKS).

### Popular CNIs at a glance

| CNI | Data path | NetworkPolicy | Notes |
|---|---|---|---|
| **kindnet** | Routed (static routes) | Basic support in recent versions | kind's default; simple |
| **Flannel** | VXLAN overlay (or host-gw) | **No** (pair it with Calico = "Canal") | Simple, minimal |
| **Calico** | Routed with BGP, or VXLAN/IP-in-IP; optional eBPF data plane | **Yes**, plus its own extended policies | Very common on-prem and in exams; topics 61 and 75 install it |
| **Cilium** | eBPF; native routing or tunnel | **Yes**, including L7 (HTTP/DNS-aware) | Can **replace kube-proxy**; Hubble flow observability; WireGuard encryption |
| **AWS VPC CNI** | VPC-native IPs | Yes (EKS network policy agent, since 2023) | EKS default |
| **Azure CNI / GKE Dataplane V2** | VPC-native; GKE V2 is Cilium-based | Yes | Managed defaults |

(Weave Net is no longer maintained; topic 09.)

**Day 07 note:** some advanced CNIs (**Calico, Cilium**) can also take over **Service routing** themselves, reducing or removing kube-proxy's role (Cilium's kube-proxy replacement, Calico's eBPF mode). Topic 32 covers this.

### MTU (Maximum Transmission Unit)

The **MTU** is the largest packet an interface sends without splitting it. If a Pod's MTU is larger than what fits inside the overlay tunnel, big packets are dropped or fragmented. The classic symptom: **small requests work, large responses hang** (TLS handshakes stall, `curl` of big files freezes). The CNI must set the Pod MTU to the node MTU minus encapsulation overhead (most do this automatically).

### Pod-to-outside traffic

When a Pod talks to the internet, its source IP (`10.244.x.x`) isn't routable outside, so the node **SNATs** (rewrites the source to the node's IP, "masquerading") on the way out. That's why external services see the **node's** IP, not the Pod's. (In VPC-native CNIs, Pod IPs may be routable inside the VPC, but internet traffic still goes through NAT gateways.)

---

## Commands / YAML

### See a Pod's network from inside and outside

```bash
kubectl run net-a --image=nicolaka/netshoot -- sleep 3600
kubectl get pod net-a -o wide
kubectl exec net-a -- ip -brief addr
kubectl exec net-a -- ip route
```

```
NAME    READY   STATUS    IP           NODE
net-a   1/1     Running   10.244.1.5   lab-worker

lo               UNKNOWN        127.0.0.1/8
eth0@if9         UP             10.244.1.5/24

default via 10.244.1.1 dev eth0
10.244.1.0/24 via 10.244.1.1 dev eth0 src 10.244.1.5
```

- `eth0@if9` = this interface is one end of a veth pair; its peer is interface **index 9 on the node**.

On the node:

```bash
docker exec lab-worker ip -brief link | grep '^veth'
docker exec lab-worker ip route
```

```
veth3a1f2c9b@if2   UP   …
default via 172.18.0.1 dev eth0
10.244.0.0/24 via 172.18.0.3 dev eth0       # control-plane node's Pods
10.244.1.5 dev veth3a1f2c9b scope host      # a local Pod, through its veth
10.244.2.0/24 via 172.18.0.4 dev eth0       # lab-worker2's Pods
```

- kindnet is **routed**: one route per remote node's Pod range via that node's IP, and one host route per local Pod via its veth.

### Watch a packet cross nodes

```bash
kubectl run net-b --image=nicolaka/netshoot --overrides='{"spec":{"nodeName":"lab-worker2"}}' -- sleep 3600
B=$(kubectl get pod net-b -o jsonpath='{.status.podIP}')
docker exec lab-worker2 sh -c "timeout 10 tcpdump -ni eth0 icmp" &     # needs tcpdump on the node; or tcpdump inside net-b
kubectl exec net-a -- ping -c 2 $B
```

```
IP 10.244.1.5 > 10.244.2.7: ICMP echo request, id 3, seq 1, length 64
IP 10.244.2.7 > 10.244.1.5: ICMP echo reply, id 3, seq 1, length 64
```

- **Real Pod IPs on the node's wire**: no overlay, no NAT (rule 2 of the model).
- With a VXLAN CNI you'd see `UDP 172.18.0.3.xxxx > 172.18.0.4.4789: VXLAN` instead.

### CNI files and agent

```bash
docker exec lab-worker ls /etc/cni/net.d /opt/cni/bin
kubectl -n kube-system get ds -o wide | grep -Ei 'kindnet|calico|cilium|flannel'
```

---

## Hands-on lab

1. Run the commands above: match each Pod IP to its node's `podCIDR`, find its veth on the node, and read the node routing table.
2. Ping across nodes and capture it on the node (or with `tcpdump -ni eth0` inside `net-b`).
3. Check MTUs: `kubectl exec net-a -- ip link show eth0` vs `docker exec lab-worker ip link show eth0`.
4. Egress SNAT: `kubectl exec net-a -- curl -s ifconfig.me` shows your laptop's public IP, not `10.244.1.5`.
5. Break and fix the CNI (from Day 58 #2): on `lab-worker`, move `/etc/cni/net.d/10-kindnet.conflist` to `/root/`, delete a Pod scheduled there, and watch the new one stick in `ContainerCreating` with a sandbox/CNI error. Move the file back; the next sandbox attempt succeeds.
6. Optional: create a kind cluster with `disableDefaultCNI: true`, see nodes `NotReady`, install Calico (topic 61) and see them go `Ready`.
7. Clean up: `kubectl delete pod net-a net-b`.

---

## Where you'll use it

- **Choosing a CNI** for new clusters: policy support, eBPF, encryption, IP planning.
- **IP planning**: Pod and Service CIDRs that don't clash with corporate networks; enough VPC IPs for VPC-native CNIs.
- **Debugging** cross-node failures, MTU black holes, and "Pods have no IP".
- **Interviews**: "How do Pods on different nodes communicate?"

---

## When it breaks

### Pods stuck `ContainerCreating` / "CNI not initialized" (Day 58 #2)

**Problem:** Pods never get IPs; events show `network plugin is not ready: cni config uninitialized` or `failed to set up sandbox container … network`.

| Likely cause | Fix |
|---|---|
| **CNI DaemonSet not running** (crashed, scaled down, can't schedule) | `kubectl -n kube-system get ds`; restart it or re-apply the CNI manifest |
| **`/etc/cni/net.d` missing/empty or a bad config** | Restore a valid config (re-install the add-on); restart the kubelet if needed |
| **Wrong Pod CIDR / mismatched networking config** | Align the CNI's IP pool with the cluster's Pod CIDR (`kubeadm init --pod-network-cidr`); re-apply; restart the kubelet |
| IPAM pool exhausted (`no IP addresses available in range set`) | Bigger per-node range, fewer Pods per node, clean leaked IP allocations |

Checks:

```bash
kubectl -n kube-system get pods -o wide | grep -Ei 'calico|cilium|flannel|kindnet'   # CNI Pods healthy?
kubectl -n kube-system get ds -o wide | grep -Ei 'calico|cilium|flannel|kindnet'     # DESIRED = READY?
kubectl describe pod <p> | sed -n '/Events:/,$p'                                      # exact CNI error
ls -l /etc/cni/net.d/                                                                 # (on the node) config present?
sudo journalctl -u kubelet -b -o cat | grep -Ei 'cni|sandbox|plugin|network' | tail   # kubelet hints
```

Fix and verify:

```bash
kubectl -n kube-system rollout restart ds/<cni-daemonset>    # if the DS was scaled down or crashed
# if the config was missing: restore it or re-apply the CNI add-on, then:
sudo systemctl restart kubelet
kubectl delete pod <p>; kubectl get pod <p> -o wide          # the new Pod gets an IP and goes Running
```

**Example:** `Events: CNI plugin not initialized` + an empty `/etc/cni/net.d` → re-apply the CNI add-on → restart the kubelet → the new Pod gets an IP and becomes **Running**.

(The original Day 58 commands also grep for `weave`; Weave Net is no longer maintained.)

### Other network-model failures

| Symptom | Cause | Fix |
|---|---|---|
| Same-node Pods talk, cross-node don't | Overlay port blocked by firewall/security group (UDP 4789, IP-in-IP, BGP TCP 179), or missing routes | Open the ports; check CNI agent logs and routes |
| Cross-node works for small requests, large ones hang | **MTU** mismatch | Set the correct MTU in the CNI config |
| Pod IPs clash with a corporate/VPN network | Overlapping CIDRs | Re-plan the CIDRs (usually means rebuilding the cluster) |
| Cloud drops Pod traffic between nodes | Source/destination check or anti-spoofing | Disable it on instances, or use an overlay or the cloud CNI |
| New Pods can't get IPs on EKS | Subnet out of IPs or per-instance IP limit | Larger subnets, prefix delegation, custom networking |

---

## Quick recap

- **Model:** every Pod has its own IP; all Pods reach all Pods without NAT; node agents reach local Pods.
- **CNI ADD:** runtime runs the plugin → veth pair + IPAM IP + routes for the Pod's netns.
- **Cross-node:** routed (BGP/static/VPC routes), overlay (VXLAN/IP-in-IP; watch MTU and firewall ports), or VPC-native IPs.
- Three non-overlapping ranges: node network, Pod CIDR (per-node slices), Service CIDR.
- "No IP / ContainerCreating" → CNI DaemonSet, `/etc/cni/net.d`, CIDR alignment, kubelet logs.

### Interview questions

1. **Q: How does a Pod on node A reach a Pod on node B?**
   A: Through its veth to node A, where the CNI's routes send the packet either directly to node B (routed/VPC-native) or encapsulated in a VXLAN/IP-in-IP tunnel to node B, which delivers it to the target Pod's veth. Pod IPs are preserved: no NAT.
2. **Q: What does a CNI plugin actually do when a Pod is created?**
   A: The runtime executes it with ADD; it creates the Pod's interface (usually a veth pair), allocates an IP via IPAM, configures addresses and routes in the Pod and on the node, and returns the result.
3. **Q: Large HTTP responses between Pods hang, small ones work. Likely cause?**
   A: An MTU mismatch with an overlay network: packets too big for the tunnel are dropped. Lower the Pod MTU to account for encapsulation overhead.

---

## Appendix: notes on the original course material

This topic was rebuilt from the original CKA course notes. This appendix keeps the audit trail out of your way while you learn.

### Sources

Day 07 (CNI's role: network setup, unique IPs, traffic routing, Pod-to-Pod communication; the note that advanced CNIs can offload Service routing), Day 25 (CNI plugins), Day 58 (scenario 2: Pods stuck ContainerCreating / CNI not initialized). New material on the network model, data paths and MTU.
