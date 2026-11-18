# 04 · The Linux That Kubernetes Runs On

> **Level:** ★★ intermediate
>
> **You will learn:** the Linux features every Kubernetes node depends on, with a short recap of each and exactly **which Kubernetes feature uses it**: processes and signals, namespaces, cgroups v2, overlay filesystems, the OCI runtime chain, veth pairs and bridges, routing, iptables/nftables and conntrack, DNS resolution, systemd and journald, kernel modules and sysctls, and swap. After this, every "how it works inside" section in these notes will make sense.
>
> **You need:** topics 02–03. For depth, your Linux notes ([`097`](../../linux-mastery-rhel/notes/097-namespaces-and-cgroups.md) namespaces and cgroups, [`058`](../../linux-mastery-rhel/notes/058-how-networks-work.md)–[`063`](../../linux-mastery-rhel/notes/063-name-resolution-and-dns-client.md) networking, [`037`](../../linux-mastery-rhel/notes/037-systemd-services-and-targets.md)/[`098`](../../linux-mastery-rhel/notes/098-systemd-unit-files.md) systemd, [`055`](../../linux-mastery-rhel/notes/055-kernel-modules-and-sysctl.md) modules and sysctl, [`050`](../../linux-mastery-rhel/notes/050-swap.md) swap, [`111`](../../linux-mastery-rhel/notes/111-linux-foundations-of-kubernetes.md) Linux foundations of Kubernetes) and Docker notes ([`02`](../../Docker/notes/02-what-a-container-really-is.md), [`07`](../../Docker/notes/07-image-layers-and-overlayfs.md), [`11`](../../Docker/notes/11-container-from-the-inside.md), [`26`](../../Docker/notes/26-linux-networking-recap.md), [`31`](../../Docker/notes/31-resource-limits.md)).

---

## ⚡ Core: the 10-minute version

**The idea.** A container isn't a mini-VM. It's an ordinary **Linux process** with:
- **namespaces**: its own view of processes, network, hostname and files;
- **cgroups**: limits on CPU and memory;
- an **overlay filesystem** made from image layers.

Kubernetes builds everything on these: a **Pod** is a group of containers sharing one set of namespaces (held by a tiny "pause" container); resource limits become **cgroup** settings; Pod networking is **veth pairs, bridges and routes**; Services are **NAT rules** (iptables/nftables). The kubelet and containerd are **systemd services** that log to **journald**.

**Picture it.** A container is a person in a room with one-way glass (namespaces) and a metered electricity supply (cgroups).

**The one thing to know.** A container process is visible on the host:

```bash
docker run -d --name s busybox sleep 1000
ps -ef | grep "sleep 1000"          # the same process, seen from the host (in WSL)
cat /proc/$(pgrep -f "sleep 1000")/cgroup
```

**Try it now:**
1. Run the commands above and find the container's process and cgroup.
2. `docker exec s ps`: inside, the same process is PID 1.
3. `docker rm -f s`.

**You've got it if you can:** say what namespaces and cgroups each do, and which Kubernetes features rely on them (Pods, limits, networking).

> 📖 **That's the core.** On a first pass, do the mini-lab and move on to the next topic. Everything below is the **full reference**: come back for the details, the edge cases, interview answers and exam-level depth.

---

## What it is

Kubernetes is **not magic on top of Linux**. Every node is a Linux machine, and every Kubernetes feature is built from ordinary kernel features:

| Kubernetes idea | Linux feature underneath |
|---|---|
| A container | A process + **namespaces** + **cgroups** + its own root filesystem (**overlayfs**) |
| A Pod's shared IP and `localhost` | One shared **network namespace** for all containers in the Pod |
| CPU/memory requests and limits | **cgroup v2** settings (`cpu.weight`, `cpu.max`, `memory.max`) |
| `OOMKilled` | The kernel's **OOM killer** acting inside a cgroup |
| Graceful Pod shutdown | **Signals**: `SIGTERM`, then `SIGKILL` |
| Pod networking | **veth pairs**, **bridges**, **routes**, sometimes **VXLAN** tunnels |
| Services (ClusterIP, NodePort) | **iptables/nftables** NAT rules (or IPVS) + **conntrack** |
| Cluster DNS names | `/etc/resolv.conf` inside the Pod pointing at CoreDNS |
| kubelet and container runtime | **systemd** services, logs in **journald** |
| Node prerequisites | Kernel modules (`overlay`, `br_netfilter`) and **sysctls** (`net.ipv4.ip_forward`) |
| `securityContext` | User IDs, **capabilities**, **seccomp**, SELinux/AppArmor |

---

## Why it exists

When something breaks in production, `kubectl` often isn't enough. A Pod is `OOMKilled` (cgroups), a Service doesn't route (iptables/nftables), DNS takes 5 seconds (resolv.conf `ndots` + conntrack), a node is `NotReady` (kubelet systemd unit). SREs debug these **at the Linux level**. Interviewers know this, so "what is a container, really?" and "how does a ClusterIP work?" are standard questions.

---

## Analogy

Kubernetes is a **hotel chain's head office**. It decides which guest goes to which hotel and room. But the **rooms, walls, locks, power meters and phone lines** are built by each hotel's building (the Linux kernel):

- **Namespaces = walls**: what a guest can see.
- **cgroups = power and water meters with fuses**: how much a guest can use.
- **veth pairs and bridges = the internal phone wiring**.
- **iptables/nftables = the switchboard** that forwards a call for "Reception" to whichever receptionist is on duty (that's a Service).
- **systemd = the building manager** who keeps the staff (kubelet, containerd) on duty and restarts them if they collapse.

Head office only sends instructions. The building does the work.

---

## How it works inside

### 1. Processes, PID 1 and signals

- Every program running on Linux is a **process** with a **PID** (process ID).
- A **signal** is a short notification the kernel delivers to a process. The important ones:
  - `SIGTERM` (15) = "please stop": the app can clean up.
  - `SIGKILL` (9) = "die now": can't be caught; the kernel ends the process.
- **PID 1** is special. In a container, your app usually is PID 1 inside its PID namespace. The kernel does **not** apply default signal actions to PID 1 (a PID 1 that doesn't install a `SIGTERM` handler simply ignores it), and PID 1 must reap zombie children.
- **Kubernetes use:** stopping a Pod = `SIGTERM` → wait `terminationGracePeriodSeconds` (default 30) → `SIGKILL` (topic 19). Exit code **137** = 128 + 9 = killed by SIGKILL (often the OOM killer). **143** = 128 + 15 = exited after SIGTERM.

### 2. Namespaces: what a process can see

A **namespace** gives a process its own private copy of one global resource. Linux has 8 types:

| Namespace | Isolates | How Kubernetes uses it |
|---|---|---|
| `net` (network) | Interfaces, IPs, routes, firewall rules, ports | **One per Pod**, shared by all its containers |
| `mnt` (mount) | Mounted filesystems | One per container: its own `/` |
| `pid` | Process IDs | One per container by default; can be shared per Pod with `shareProcessNamespace: true` |
| `uts` | Hostname | One per Pod: the hostname is the Pod name |
| `ipc` | Shared memory, message queues | One per Pod: containers in a Pod can use shared memory |
| `user` | User/group ID mapping | Off by default; opt in with `hostUsers: false` (user namespaces for Pods) |
| `cgroup` | View of the cgroup tree | One per container |
| `time` | Some clocks | Not used |

**Day 07 on network namespaces, in plain words:**
- Each **Pod** gets its **own network namespace**: its own IP address, its own network interfaces (`eth0`, `lo`) and its own routing table.
- **All containers in one Pod share that network namespace.** So they talk to each other over **`localhost`**, and two containers in one Pod can't both listen on the same port.
- Pods are isolated from each other's network stacks, but every Pod can reach every other Pod **by its IP** (that's the Kubernetes network model, topic 29).

**How the Pod shares one namespace: the pause container.** When the kubelet creates a Pod, the runtime first starts a tiny **"pause" container** (image `registry.k8s.io/pause`) whose only job is to **hold** the Pod's network, IPC and UTS namespaces open (it just sleeps forever). Your app containers are then started **inside those existing namespaces** (using the `setns()` system call, like `docker run --network container:X`). If an app container restarts, the Pod keeps its IP, because the pause container (the "sandbox") never stopped. Topic 18 shows this live.

### 3. cgroups v2: what a process can use

A **cgroup** (control group) is a group of processes the kernel **measures and limits together**. You manage it by writing numbers into files under `/sys/fs/cgroup`.

| Kubernetes setting | cgroup v2 file | Effect |
|---|---|---|
| CPU **request** | `cpu.weight` | Share of CPU **when the CPU is busy** (topic 41) |
| CPU **limit** | `cpu.max` (e.g. `50000 100000` = 0.5 CPU) | Hard cap: the process is **throttled** when it hits it |
| Memory **limit** | `memory.max` | Hard cap: exceed it and the **OOM killer** kills a process → `OOMKilled` |
| Memory **request** | (used for scheduling; also `memory.min`/`memory.low` with the MemoryQoS feature) | — |

- The kubelet builds a tree: `kubepods.slice` → QoS class (`burstable`, `besteffort`) → one cgroup per **Pod** → one per **container**.
- **cgroup driver:** the kubelet and the runtime must agree on *who* manages cgroups. On systemd-based Linux, both use the **`systemd`** driver (in containerd: `SystemdCgroup = true`). A mismatch was a classic kubeadm setup bug (topic 75). Since v1.34 the kubelet can ask the runtime which driver to use.
- ⚠️ **cgroup v1** is deprecated. Since **v1.35** the kubelet **refuses to start on cgroup v1** unless you explicitly override it (`failCgroupV1: false`). Use a distro with cgroup v2 (Ubuntu 22.04+, RHEL 9+). Check: `stat -fc %T /sys/fs/cgroup` → `cgroup2fs`.

### 4. The root filesystem: overlayfs

The runtime unpacks image layers once into a **snapshot** store (`/var/lib/containerd/...`) and **stacks** them with **overlayfs**: read-only image layers at the bottom, a thin writable layer on top per container. Writes go to the top layer (**copy-on-write**). When the container is deleted, the writable layer is gone. That's why Kubernetes needs **volumes** for data that must survive (topics 37–39).

### 5. The OCI runtime chain

- **OCI** = **Open Container Initiative**, the standards body that defines the **image format** (layers + config) and the **runtime spec** (a `config.json` describing namespaces, cgroups, mounts and the process to run).
- On a Kubernetes node:

```
kubelet ──CRI (gRPC over /run/containerd/containerd.sock)──▶ containerd
        ──▶ containerd-shim-runc-v2 (one per Pod; stays as the parent of the containers)
        ──▶ runc (creates namespaces + cgroups from config.json, starts the process, then exits)
```

- The **shim** lets containerd restart or upgrade without killing containers, and it collects the exit code.
- **CRI-O** is an alternative to containerd built only for Kubernetes; same idea, also uses runc (or crun).

### 6. Networking pieces

| Piece | What it is | Kubernetes use |
|---|---|---|
| **veth pair** | A virtual cable with two ends; a packet in one end comes out the other. | One end is `eth0` **inside the Pod's netns**, the other (`vethXXXX`/`caliXXXX`) is on the **node**. |
| **Linux bridge** | A virtual switch connecting interfaces. | Some CNI plugins (kindnet, flannel's `cni0`) plug Pod veths into a bridge. |
| **Routing table** | "To reach network X, send via Y." | Each node has routes to other nodes' Pod subnets (topic 29). |
| **VXLAN / IP-in-IP** | Wrap a packet inside another packet to cross a network that doesn't know Pod IPs. | Overlay networking in Calico/Flannel/Cilium. |
| **iptables / nftables** | The kernel's packet-filter and NAT rules (netfilter). | **kube-proxy** writes rules that DNAT a Service IP to a Pod IP (topic 32); NetworkPolicy may also use them. |
| **conntrack** | The kernel's connection-tracking table: remembers each flow's NAT translation. | Makes Service NAT work for replies; a full table drops packets (`nf_conntrack: table full`). |
| **IPVS** | Kernel layer-4 load balancer. | An alternative kube-proxy mode, **deprecated in v1.37** (topic 32). |
| **eBPF** | Small verified programs loaded into the kernel. | Cilium replaces kube-proxy and iptables with eBPF. |

### 7. DNS resolution

Programs resolve names with the C library's resolver, which reads **`/etc/resolv.conf`** (which DNS servers, which **search domains**, the **`ndots`** option) and **`/etc/nsswitch.conf`** (whether to check `/etc/hosts` first). In a Pod, the kubelet writes a `resolv.conf` that points at **CoreDNS** with search domains such as `default.svc.cluster.local`. That's why `curl backend` works inside the cluster (topic 33).

### 8. systemd, journald, modules, sysctls, swap

- **systemd** starts and supervises services. On a kubeadm node, **kubelet** and **containerd** are systemd units: `systemctl status kubelet`, `journalctl -u kubelet -f`. The control-plane components are **not** systemd units on kubeadm. They're **static Pods** that the kubelet runs from `/etc/kubernetes/manifests` (topic 42).
- **Kernel modules** must be loaded on nodes: `overlay` (container filesystems) and `br_netfilter` (lets iptables see bridged traffic).
- **sysctls** (kernel settings): `net.ipv4.ip_forward=1` (the node must route packets for Pods) and `net.bridge.bridge-nf-call-iptables=1`.
- **Swap:** the kubelet traditionally refused to start with swap on, so kubeadm guides say `swapoff -a`. Swap support for Linux nodes is now available (`LimitedSwap` mode, configured in the kubelet), but disabling swap is still the simple default (topic 75).

---

## Commands / YAML

These run on a **node** (in kind, a node is a container: `docker exec -it <node-name> bash`, topic 11).

**Find a container's process and its namespaces:**

```bash
crictl ps                                     # containers the kubelet's runtime is running
crictl inspect <container-id> | grep -m1 '"pid"'
lsns -p <pid>
```

```
        NS TYPE   NPROCS   PID USER  COMMAND
4026532301 mnt         1  2817 root  nginx: master process nginx -g daemon off;
4026532302 pid         1  2817 root  nginx: master process nginx -g daemon off;
4026532244 net         3  2756 65535 /pause
4026532242 uts         3  2756 65535 /pause
4026532243 ipc         3  2756 65535 /pause
```

- `crictl` = the CRI command-line tool, a `docker`-like CLI that talks to containerd/CRI-O (topic 08).
- `NS` = namespace ID; same number = shared namespace.
- Note the **`net`, `uts` and `ipc` namespaces belong to `/pause`**: nginx joined the pause container's namespaces. `mnt` and `pid` are nginx's own.

**See the cgroup limits Kubernetes set:**

```bash
cat /proc/<pid>/cgroup
cat /sys/fs/cgroup/kubepods.slice/kubepods-burstable.slice/<pod-slice>/<container-scope>/memory.max
```

```
0::/kubepods.slice/kubepods-burstable.slice/kubepods-burstable-pod7c1f….slice/cri-containerd-9a3e….scope
268435456
```

- `0::` = cgroup v2 (one unified tree).
- `kubepods-burstable` = this Pod's QoS class is Burstable (topic 41).
- `268435456` bytes = 256 MiB = the container's `resources.limits.memory`.

**See Service NAT rules (iptables mode):**

```bash
iptables -t nat -L KUBE-SERVICES -n | head
```

```
Chain KUBE-SERVICES (2 references)
target                     prot opt source     destination
KUBE-SVC-NPX46M4PTMTKRN6Y  tcp  --  0.0.0.0/0  10.96.0.1    /* default/kubernetes:https cluster IP */ tcp dpt:443
KUBE-SVC-TCOU7JCQXEZGVUNU  udp  --  0.0.0.0/0  10.96.0.10   /* kube-system/kube-dns:dns cluster IP */ udp dpt:53
```

- Each line = "packets to this ClusterIP:port jump to that Service's chain", which then DNATs to one Pod IP. Topic 32 walks the full chain.

---

## Hands-on lab: build a two-"Pod" network by hand

Do this in your WSL2 Ubuntu (root needed). It's exactly what a CNI plugin does for every Pod.

```bash
# 1. Two network namespaces = two "Pods"
sudo ip netns add podA
sudo ip netns add podB

# 2. A bridge = the node's virtual switch
sudo ip link add br0 type bridge
sudo ip addr add 10.244.0.1/24 dev br0
sudo ip link set br0 up

# 3. A veth pair per "Pod": one end inside, one end on the bridge
for p in A B; do
  sudo ip link add veth$p type veth peer name eth0-$p
  sudo ip link set eth0-$p netns pod$p
  sudo ip link set veth$p master br0 up
done
sudo ip netns exec podA ip addr add 10.244.0.10/24 dev eth0-A
sudo ip netns exec podB ip addr add 10.244.0.11/24 dev eth0-B
for p in A B; do
  sudo ip netns exec pod$p ip link set eth0-$p up
  sudo ip netns exec pod$p ip link set lo up
  sudo ip netns exec pod$p ip route add default via 10.244.0.1
done

# 4. Test: "Pod A" pings "Pod B" by IP
sudo ip netns exec podA ping -c 2 10.244.0.11
```

```
64 bytes from 10.244.0.11: icmp_seq=1 ttl=64 time=0.061 ms
64 bytes from 10.244.0.11: icmp_seq=2 ttl=64 time=0.052 ms
```

What you built: each namespace has its own `eth0-*` and IP, like a Pod; the bridge is the node; the default route lets them leave the node. A CNI plugin does these same steps automatically (topic 29).

Two more checks:
- `sudo ip netns exec podA ss -tlnp` shows **no** listeners from the host: separate network stack.
- `stat -fc %T /sys/fs/cgroup` should print `cgroup2fs`.

Clean up: `sudo ip netns del podA; sudo ip netns del podB; sudo ip link del br0`.

---

## Where you'll use it

- **OOMKilled / throttling investigations:** read cgroup files and `memory.events`.
- **"Service doesn't work":** dump iptables/nftables rules, check conntrack.
- **Node `NotReady`:** `systemctl status kubelet`, `journalctl -u kubelet`, `crictl ps`.
- **Building nodes** with kubeadm: modules, sysctls, cgroup driver, swap (topic 75).
- **Security reviews:** which namespaces are shared with the host (`hostNetwork`, `hostPID`), which capabilities are granted (topic 59).

---

## When it breaks

| Symptom | Linux cause | Where it's covered |
|---|---|---|
| `OOMKilled`, exit code 137 | Container hit `memory.max` | 41 |
| App slow, CPU never above limit | CFS throttling from `cpu.max` | 41 |
| kubelet won't start: "cgroup v1 is not supported" | Node still on cgroup v1 | 75 |
| Pods stuck `ContainerCreating`, "network plugin not ready" | CNI didn't set up the netns/veth | 29, 83 |
| Service IP times out | Missing iptables/nftables rules, `ip_forward=0`, or conntrack full | 32, 83 |
| DNS lookups take 5 s | `ndots:5` search-domain expansion + conntrack races on UDP | 33 |
| Pod takes 30 s to stop | PID 1 ignores SIGTERM | 03, 19 |

---

## Quick recap

- A container is a process with namespaces (view), cgroups (limits) and an overlayfs root.
- A Pod is a **shared network/IPC/UTS namespace** held by the **pause** container; app containers join it.
- Requests/limits become **cgroup v2** files; the kubelet and runtime must use the same (systemd) cgroup driver.
- Pod networking = veth + bridge/routes (+ overlay); Services = **netfilter NAT + conntrack** (or eBPF).
- kubelet and containerd are **systemd** services; logs are in **journald**.

### Interview questions

1. **Q: How do containers in the same Pod talk to each other?**
   A: Over `localhost`: they share one network namespace, created and held by the pause (sandbox) container.
2. **Q: What enforces a container's memory limit, and what happens when it's exceeded?**
   A: The kernel, through the container's cgroup `memory.max`. The OOM killer kills a process in that cgroup; Kubernetes shows `OOMKilled` and exit code 137.
3. **Q: Why must `net.ipv4.ip_forward` be 1 on Kubernetes nodes?**
   A: Nodes route packets between Pod network interfaces and other nodes; with forwarding off, Pod traffic can't leave the node.

---

## Appendix: notes on the original course material

This topic was rebuilt from the original CKA course notes. This appendix keeps the audit trail out of your way while you learn.

### Sources

Day 07 ("What are network namespaces?"), plus new material.
