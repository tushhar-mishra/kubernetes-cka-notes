# 08 · Node Components: kubelet, Container Runtime, CRI, kube-proxy

> **Level:** ★★ intermediate
>
> **You will learn:** what the kubelet does all day (Pod sync loop, node heartbeats, probes, eviction, garbage collection), the **CRI** (Container Runtime Interface) and the chain kubelet → containerd → shim → runc, the history of **dockershim** and why "Kubernetes removed Docker" didn't break your images, the runtimes you can use, the `crictl` tool, and a first look at kube-proxy.
>
> **You need:** topics 04, 06, 07.

---

## ⚡ Core: the 10-minute version

**The idea.** On every node:
- the **kubelet** is the site manager. It reads which Pods are assigned to its node, asks the **container runtime** (containerd) to start them through the **CRI** interface, runs health probes, and reports status back.
- containerd uses a small **shim** and **runc** to create the actual Linux processes, and calls the **CNI** plugin to give each Pod an IP.
- **kube-proxy** programs the node's NAT rules so Service IPs reach the right Pods.

Docker's own daemon isn't involved any more: **dockershim was removed in v1.24**, but your Docker-built images still run unchanged.

**Picture it.** The kubelet is a foreman reading work orders; containerd is the machine that builds what the foreman orders.

**The one thing to know.** On a node, `crictl` is your `docker ps`:

```bash
docker exec -it lab-worker bash
crictl ps        # running containers
crictl pods      # Pod sandboxes
systemctl status kubelet --no-pager
```

**Try it now:**
1. Run the commands above inside a kind node.
2. `journalctl -u kubelet -n 20`: kubelet logs.
3. Start a Pod with kubectl and find it with `crictl ps`.

**You've got it if you can:** draw kubelet → CRI → containerd → shim → runc, and say what kube-proxy does (and doesn't do).

> 📖 **That's the core.** On a first pass, do the mini-lab and move on to the next topic. Everything below is the **full reference**: come back for the details, the edge cases, interview answers and exam-level depth.

---

## What it is

Every node (workers **and** control-plane nodes) runs:

| Component | Day 07 / Day 25 summary | Runs as |
|---|---|---|
| **kubelet** | "The primary node agent. Communicates with the control plane and ensures containers are running as expected." | A **systemd service** on the host (not a Pod) |
| **Container runtime** | "The software responsible for running containers on the nodes." | A systemd service (`containerd` or `crio`) |
| **kube-proxy** | "Maintains network rules on each node so traffic to Services reaches Pods." | A **DaemonSet** Pod on each node (or replaced by the CNI) |

---

## Why it exists

The control plane only **writes decisions** into the API server ("Pod X goes on node 2"). Something on node 2 has to **turn that record into real processes**, keep them alive, report back, and clean up. That's the kubelet, driving a runtime through a standard interface so Kubernetes doesn't care which runtime you picked.

---

## Analogy

A **restaurant franchise**:
- Head office (control plane) sends the order "table 7: two pizzas".
- The **kubelet** is the **shift manager** in one branch. They read new orders for their branch, tell the kitchen, check the food is actually cooking (probes), report status back to head office, and throw away food nobody ordered.
- The **container runtime** is the **kitchen**.
- The **CRI** is the **standard order slip**: any kitchen that understands the slip can work in the franchise.
- **kube-proxy** is the **host desk sign**: "pizza orders → counter 3".

---

## How it works inside

### 1. The kubelet's jobs

1. **Registers the node**: creates its `Node` object with capacity (CPU, memory, max Pods), labels and addresses.
2. **Heartbeats**: renews a **Lease** in the `kube-node-lease` namespace every ~10 s and updates `Node.status` periodically. If heartbeats stop for ~40 s (`node-monitor-grace-period`), the node controller marks the node **`NotReady`/`Unknown`** and taints it, and Pods are evicted after a toleration period (default 300 s, topic 43).
3. **Pod sync loop**: watches the API server for Pods with `spec.nodeName = <this node>`, plus **static Pods** from `/etc/kubernetes/manifests` (topic 42). For each Pod it:
   - admits it (enough resources? allowed?),
   - makes the runtime create the **Pod sandbox** (the pause container + network namespace; the runtime calls the **CNI plugin** to give it an IP),
   - mounts **volumes** (ConfigMaps, Secrets, PVs through CSI),
   - pulls images and starts **init containers**, then app containers,
   - creates the **cgroups** for requests and limits (topic 41).
4. **Health checks**: runs **liveness, readiness and startup probes** (topic 22) and restarts containers per the `restartPolicy` (topic 19).
5. **Reports status**: writes `Pod.status` (phase, conditions, container states, IP) back to the API server.
6. **Node-pressure eviction**: if the node runs low on memory or disk, it **evicts** Pods to protect the node (topic 47).
7. **Garbage collection**: deletes dead containers and unused images when disk usage crosses thresholds (`imageGCHighThresholdPercent`, default 85%).
8. **Serves an HTTPS API** on port **10250**: the API server calls it for `kubectl logs`, `exec`, `port-forward` and metrics.

The kubelet is configured by a **config file** (`/var/lib/kubelet/config.yaml`, a `KubeletConfiguration` object) plus a few flags in its systemd drop-in (`/etc/systemd/system/kubelet.service.d/10-kubeadm.conf` on kubeadm), and it talks to the API server using `/etc/kubernetes/kubelet.conf` (a kubeconfig, topic 52).

### 2. The CRI and the runtime chain

**CRI = Container Runtime Interface**: a **gRPC API** defined by Kubernetes. It has two services:
- **RuntimeService**: `RunPodSandbox`, `CreateContainer`, `StartContainer`, `StopContainer`, `ExecSync`, `ContainerStatus`…
- **ImageService**: `PullImage`, `ListImages`, `RemoveImage`.

Any runtime that implements these calls can be used by the kubelet. On a node:

```
kubelet
  │  CRI gRPC over unix:///run/containerd/containerd.sock
  ▼
containerd (CRI plugin built in)
  │  creates the Pod sandbox → calls the CNI plugin (/opt/cni/bin, config in /etc/cni/net.d) for networking
  │  pulls and unpacks images into its snapshot store
  ▼
containerd-shim-runc-v2   (one per Pod; parent of its containers; survives containerd restarts)
  ▼
runc   (OCI runtime: reads config.json, sets namespaces/cgroups/mounts, execs the process, exits)
```

Note: it's the **runtime** that calls the CNI plugin when it creates the sandbox, not the kubelet directly.

### 3. Runtimes you can use (Day 25 CRI table, updated)

*(Indicative, not exhaustive.)*

| Runtime | Description |
|---|---|
| **containerd** | The default for most clusters (kind, kubeadm, EKS, GKE, AKS). Lightweight, OCI-compliant, a CNCF graduated project. Kubernetes v1.36+ expects **containerd 2.x**; 1.7 is the last 1.x line still tolerated, and its support is being phased out. |
| **CRI-O** | A minimal runtime built **only** for Kubernetes; versions track Kubernetes versions. Default in Red Hat OpenShift. |
| **Kata Containers** | Runs each Pod inside a **lightweight VM** for strong isolation of untrusted workloads. Plugs in under containerd/CRI-O as a different OCI runtime. |
| **gVisor** (Google) | A **user-space kernel** that intercepts system calls (sandboxing); limits direct host-kernel access. Also plugs in under containerd (`runsc`). |

Kata and gVisor are chosen per Pod with a **RuntimeClass** (topic 60).

### 4. The dockershim story

- In the early days, the kubelet talked to **Docker Engine** directly through built-in code called **dockershim** (a "shim" that translated kubelet calls into Docker API calls).
- Docker Engine itself never implemented CRI, and it already used containerd underneath, so the path was kubelet → dockershim → dockerd → containerd → runc: an extra layer to maintain.
- **dockershim was deprecated in v1.20 and removed in v1.24 (May 2022).** Clusters switched to containerd or CRI-O directly.
- **Your images still work**: images built with `docker build` are standard **OCI images**; containerd and CRI-O run them unchanged.
- What broke: anything that mounted `/var/run/docker.sock` on nodes (old CI "Docker-in-Docker" builds, some monitoring agents), or ran `docker ps` on nodes. Use `crictl` instead.
- If you *must* keep Docker Engine on nodes, Mirantis maintains an external adapter, **cri-dockerd**.

> ✅ **Correct today** ([C1](#c1)): Podman is **not** a CRI runtime (it's a CLI tool like Docker), and dockershim isn't just deprecated: it was **removed in v1.24**. Docker Engine works only via the external cri-dockerd.

### 5. CRI responsibilities

What the runtime does for Kubernetes through the CRI (indicative):
- Pulls images from registries.
- Starts, stops and deletes containers (and Pod sandboxes).
- Sets up Pod networking by **calling the CNI plugin**.
- Captures container **logs** (to files under `/var/log/pods/…`, which `kubectl logs` reads through the kubelet).
- Applies CPU/memory limits through cgroups (the values come from the kubelet).
- Exposes all of this through the standardised **gRPC** API, so Kubernetes stays runtime-agnostic.

### 6. kube-proxy: a first look

Every **Service** gets a stable virtual IP (ClusterIP, topic 30). Packets sent to that IP must be rewritten to a real Pod IP. **kube-proxy** watches Services and **EndpointSlices** (lists of ready Pod IPs) and **writes kernel rules** on its node to do that rewriting:
- **iptables mode** (default on most clusters),
- **nftables mode** (GA since v1.33; the modern successor; becoming the default in later releases),
- **IPVS mode** (**deprecated in v1.37**).

kube-proxy **does not sit in the packet path** and **does not proxy traffic itself** (despite its name, which comes from an early "userspace" mode that was removed in v1.26). The kernel does the forwarding using kube-proxy's rules. Some CNIs (Cilium) replace kube-proxy entirely with eBPF. Topic 32 shows the full rule chain and corrects the original notes' description.

---

## Commands / YAML

### kubelet on the node

```bash
docker exec -it lab-worker bash                  # kind: enter a node (it's a container)
systemctl status kubelet --no-pager | head -8
```

```
● kubelet.service - kubelet: The Kubernetes Node Agent
     Loaded: loaded (/etc/systemd/system/kubelet.service; enabled; preset: enabled)
    Drop-In: /etc/systemd/system/kubelet.service.d
             └─10-kubeadm.conf, 11-kind.conf
     Active: active (running) since Tue 2026-11-17 09:12:04 UTC; 2h ago
   Main PID: 201 (kubelet)
```

- `enabled` = starts at boot. `Drop-In` = extra config files that add flags.
- Logs: `journalctl -u kubelet -f`. **The first place to look when a node is `NotReady`.**

```bash
grep -E 'cgroupDriver|staticPodPath|clusterDNS' -A1 /var/lib/kubelet/config.yaml
```

```
cgroupDriver: systemd
clusterDNS:
- 10.96.0.10
staticPodPath: /etc/kubernetes/manifests
```

- `cgroupDriver: systemd` must match containerd's `SystemdCgroup = true` (topic 04).
- `clusterDNS` = the CoreDNS Service IP written into every Pod's `/etc/resolv.conf` (topic 33).
- `staticPodPath` = where static Pod manifests live (topic 42).

### crictl: docker-like CLI for the CRI

```bash
crictl ps
```

```
CONTAINER      IMAGE          CREATED        STATE    NAME         ATTEMPT  POD ID         POD
9a3e1f0c2b11   a8758716bb6aa  20 minutes ago Running  nginx        0        5d2c7e9a1f03   web-6d4b7c9f8-k2xjp
c41f7d22e9a0   0e1c5d6a7b8c9  2 hours ago    Running  kube-proxy   0        1b9a3f4e5d6c   kube-proxy-7nqfh
```

- `ATTEMPT` = how many times this container was restarted in its Pod (matches `RESTARTS` in kubectl).
- `POD ID` = the **sandbox** ID.

| Task | Command |
|---|---|
| List Pod sandboxes | `crictl pods` |
| List containers (incl. exited) | `crictl ps -a` |
| Container logs | `crictl logs <container-id>` |
| Inspect (PID, mounts, cgroup) | `crictl inspect <container-id>` |
| Exec into a container | `crictl exec -it <container-id> sh` |
| List / pull / remove images | `crictl images` / `crictl pull nginx:1.29` / `crictl rmi <image>` |
| Resource stats | `crictl stats` |

`crictl` reads its endpoint from `/etc/crictl.yaml` (`runtime-endpoint: unix:///run/containerd/containerd.sock`). It's for **debugging**: containers you start with crictl aren't known to the API server, and the kubelet may delete them.

Other low-level tools: `ctr` (containerd's own debug CLI; Kubernetes containers live in its `k8s.io` namespace: `ctr -n k8s.io containers ls`) and `nerdctl` (a Docker-compatible CLI for containerd).

### Node object from the API side

```bash
kubectl describe node lab-worker | sed -n '/Conditions:/,/Addresses:/p'
```

```
Conditions:
  Type             Status  LastHeartbeatTime                 Reason                       Message
  ----             ------  -----------------                 ------                       -------
  MemoryPressure   False   Tue, 17 Nov 2026 11:20:41 +0000   KubeletHasSufficientMemory   kubelet has sufficient memory available
  DiskPressure     False   Tue, 17 Nov 2026 11:20:41 +0000   KubeletHasNoDiskPressure     kubelet has no disk pressure
  PIDPressure      False   Tue, 17 Nov 2026 11:20:41 +0000   KubeletHasSufficientPID      kubelet has sufficient PID available
  Ready            True    Tue, 17 Nov 2026 11:20:41 +0000   KubeletReady                 kubelet is posting ready status
Addresses:
```

- The `*Pressure` conditions drive eviction (topic 47).
- `Ready True` = the kubelet is healthy and the runtime and CNI are working. If the CNI isn't installed, you'll see `Ready False … container runtime network not ready: NetworkReady=false … cni plugin not initialized` (topic 83).

---

## Hands-on lab

On kind (topic 11):

1. `docker exec -it lab-worker bash`, then run `crictl pods` and `crictl ps`. Match them to `kubectl get pods -A -o wide | grep lab-worker`.
2. Find the shim and runc processes: `ps -ef | grep -E 'containerd-shim|pause' | grep -v grep`. One shim per Pod; one `/pause` per Pod.
3. Kill a container behind Kubernetes' back: `crictl stop <nginx-container-id>`. Watch `kubectl get pod -w` from your laptop: the kubelet notices and restarts it (`RESTARTS` goes to 1).
4. Break the kubelet: `systemctl stop kubelet`. After ~40 s, `kubectl get nodes` shows `lab-worker NotReady`. Running containers **keep running** (`crictl ps`), but nothing reports their status. `systemctl start kubelet` to recover.
5. Read the kubelet log during step 4: `journalctl -u kubelet --since "5 min ago" | tail`.

---

## Where you'll use it

- **Node `NotReady` incidents**: kubelet logs, runtime status, CNI files, certificates.
- **Image problems on one node**: `crictl images`, `crictl pull` to test registry access from that node.
- **Disk-full nodes**: image GC thresholds, `/var/lib/containerd` size.
- **Migrations**: upgrading containerd 1.7 → 2.x, cgroup v1 → v2, verifying no workloads depend on `docker.sock`.

---

## When it breaks

| Symptom | Cause | Fix |
|---|---|---|
| Node `NotReady`, kubelet `inactive` | kubelet crashed: bad config, swap on, cgroup v1, expired client cert | `journalctl -u kubelet`; fix, restart (topic 83) |
| `NotReady`, "container runtime is down" | containerd stopped or socket path wrong | `systemctl status containerd`; check `crictl info` |
| `NotReady`, "cni plugin not initialized" | No CNI installed or `/etc/cni/net.d` empty | Install/repair the CNI (topic 29) |
| Pods `ContainerCreating` forever, "failed to create pod sandbox" | CNI error or pause image can't be pulled | `kubectl describe pod`, kubelet and containerd logs |
| `ImageGCFailed`, DiskPressure | Node disk full of images/logs | Clean up, resize disk, tune GC |
| Tooling broke after upgrade, "docker.sock not found" | dockershim removed | Use containerd socket / crictl / BuildKit |

---

## Quick recap

- The **kubelet** turns Pod records into running containers, runs probes, reports status, evicts under pressure and garbage-collects; it's a systemd service.
- **CRI** is the gRPC contract: kubelet → containerd/CRI-O → shim → runc. The runtime calls **CNI** for networking.
- **dockershim was removed in v1.24**; Docker-built images still run; use `crictl` on nodes.
- **kube-proxy** writes iptables/nftables rules for Services; it doesn't forward packets itself.

### Interview questions

1. **Q: Did removing dockershim break Docker images?**
   A: No. Docker builds OCI images, which containerd and CRI-O run natively. Only tools that relied on the Docker daemon on nodes (`docker.sock`, `docker ps`) broke.
2. **Q: A node shows NotReady. Where do you look first?**
   A: On the node: `systemctl status kubelet` and `journalctl -u kubelet`, then the runtime (`systemctl status containerd`, `crictl info`), then the CNI config, then the kubelet's certificates and its connectivity to the API server.
3. **Q: What does the pause container do?**
   A: It's the Pod sandbox: it holds the Pod's network (and IPC/UTS) namespaces so app containers can join them and restart without the Pod losing its IP.

---

## Appendix: notes on the original course material

This topic was rebuilt from the original CKA course notes. This appendix keeps the audit trail out of your way while you learn.

### Sources

Day 07 (kubelet, kube-proxy, container runtime), Day 25 (node components, CRI plugin table, CRI responsibilities, Docker/dockershim note). The same sections reappear in `Storage-In-Kubernetes-Masterclass/README.md` (Days 24–28).

### Corrections to the original notes

Each entry shows what the original said and what's true now. In the topic above, each correction appears as a ✅ **Correct today** line or a ⚠️ **Fix to the original** box, linked to its entry here.

<a id="c1"></a>**C1 · Corrected (Day 25).** the CRI table listed "**Podman** (daemonless alternative)" and "**Docker** (previously supported via dockershim; **now deprecated**)" as runtimes. **Now:** Podman is **not** a CRI runtime (it's a CLI tool like Docker), and dockershim isn't just deprecated: it was **removed in v1.24**. Docker Engine works only via the external cri-dockerd.
