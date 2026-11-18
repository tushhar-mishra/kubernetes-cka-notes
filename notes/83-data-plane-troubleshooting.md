# 83 · Data-Plane Troubleshooting: Nodes, CNI, Services, DNS, Storage, Apps

> **Level:** ★★★ advanced
>
> **You will learn:** what each data-plane piece does and how it fails (kubelet, container runtime, CNI, kube-proxy, CoreDNS, CSI); the fixed debugging order (**node → Pod IPs → Services/endpoints → DNS → storage → app**); and exam-style playbooks for **Node NotReady**, **ContainerCreating / CNI not initialised**, Pod-to-Pod and **ClusterIP** failures, **DNS**, **image pulls**, **CrashLoopBackOff/OOMKilled/probes**, **PVC Pending / mount errors**, and **NodePort/LoadBalancer** reachability, each as problem → causes → checks → minimal fix → verify.
>
> **You need:** topics 08, 19–22, 29–33, 37–41, 61, 81, 82.

---

## ⚡ Core: the 10-minute version

**The idea.** The **data plane** (kubelet, runtime, CNI, kube-proxy, CoreDNS, CSI) is where workloads run. Check it **in dependency order**:
1. **Node** healthy (kubelet and containerd running; no swap, certificate or pressure problems)?
2. Pods getting **IPs** (CNI)?
3. Services have **endpoints**, and kube-proxy's rules exist?
4. **DNS** resolves from a Pod?
5. **PVCs** bind and mount?
6. Finally the **app** (probes, limits, config).

**Picture it.** A delivery network: depot, address labels, sorting centre, phone directory, storage lockers. Check them in that order.

**The one thing to know.** One test Pod walks the whole chain:

```bash
kubectl run t --rm -it --image=busybox:1.37 --restart=Never -- sh
/ # wget -qO- -T 3 http://<pod-ip>:<port>     # CNI
/ # wget -qO- -T 3 http://<cluster-ip>         # kube-proxy + endpoints
/ # nslookup web.shop                          # DNS
/ # wget -qO- -T 3 http://web.shop             # everything together
```

The first step that fails tells you the broken layer.

**Try it now:** stop the kubelet on a worker, scale CoreDNS to 0, and typo a Service selector, one at a time. Diagnose each one with the chain above.

**You've got it if you can:** debug Node NotReady, empty endpoints, DNS failures and Pending PVCs.

> 📖 **That's the core.** On a first pass, do the mini-lab and move on to the next topic. Everything below is the **full reference**: come back for the details, the edge cases, interview answers and exam-level depth.

---

## What it is

The **data plane** is where workloads actually **run and talk**: the **kubelet**, the **container runtime** (containerd/CRI-O), the **CNI** plugin, **kube-proxy**, **CoreDNS** and **CSI** drivers. Everything you've learnt so far shapes it: priority and preemption, admission and Pod Security, securityContext and capabilities, requests/limits and probes, storage. **Make sure those basics are clear before troubleshooting**; when stuck, revisit them.

### Quick refresher: the pieces

| Piece | What it does | If it's broken | Ask yourself |
|---|---|---|---|
| **kubelet** (node agent) | Registers the node, starts/stops Pods via CRI, runs probes, reports status; reads `/etc/kubernetes/kubelet.conf` | Node **NotReady**; probes, `exec`, `logs` stall | "Is the node alive and manageable?" |
| **Container runtime** | Pulls images, creates sandboxes and containers, serves the **CRI socket** | kubelet can't create Pods; pulls fail; existing containers keep running **unmanaged** | "Is the CRI up and is the kubelet using the right socket?" |
| **CNI** | Assigns Pod IPs, wires veth pairs/routes, enforces NetworkPolicy (if supported); config in **`/etc/cni/net.d`**, agent as a DaemonSet | Pods stuck **ContainerCreating** ("CNI plugin not initialized") | "Does a new Pod get an IP immediately?" |
| **kube-proxy** | Programs **nftables/iptables** (or IPVS, deprecated) rules mapping Service VIPs and NodePorts → ready endpoints (from EndpointSlices); handles `sessionAffinity`, `externalTrafficPolicy` | ClusterIP/NodePort traffic fails or is flaky | "Do Services have endpoints, and are rules applied?" |
| **CoreDNS** | Resolves Service names via the `kube-dns` Service; forwards external names upstream | `nslookup kubernetes.default` fails in Pods | "Can a Pod resolve names right now?" |
| **CSI** | Provisions/attaches/mounts volumes (controller + node plugins) for StorageClass/PVC/PV | **PVC Pending**, mount errors | "Do PVCs bind, and do Pods mount?" |

---

## Why it exists

Most production incidents and many CKA tasks are data-plane problems: a node down, a Service with no endpoints, DNS broken by a policy, a PVC that won't bind. They're rarely mysterious if you check the pieces **in dependency order**, because each layer depends on the one before it.

---

## Analogy

A **delivery network**: the **depot manager** (kubelet) and **warehouse machinery** (runtime) must work before anything ships; each parcel needs an **address label** (CNI gives the Pod an IP); the **sorting centre** (kube-proxy) routes parcels for "Service X" to a real address; the **phone directory** (CoreDNS) turns names into numbers; **storage lockers** (CSI) must be assigned before goods can be stored. If parcels aren't arriving, check in that order.

---

## How it works inside

### The fast path

Work in this order (control plane assumed OK; topic 82):
1. **Node health**: kubelet/CRI, node conditions, pressure.
2. **CNI and Pod IPs**: `/etc/cni/net.d/`, the CNI DaemonSet, does a new Pod get an IP?
3. **Services**: selector → **EndpointSlices**; then **kube-proxy** health.
4. **DNS**: CoreDNS Pods + lookups from a Pod.
5. **Storage**: PVC/CSI.
6. **App**: probes, limits, args, config.

Rules:
- **Read Events first**: `describe` usually spells out the failure.
- Make the **smallest fix** that turns signals green, then **verify immediately**.
- Use a throwaway **busybox** (or netshoot) Pod for `nslookup`/`wget` tests, and clean up.

Lab assumptions: **kubeadm + containerd + Calico**, staying inside the cluster (no cloud firewall specifics).

---

## Commands / YAML

### 1) Node NotReady / kubelet not registered

**Problem:** `kubectl get nodes` shows `NotReady` (or the node never registers); new Pods don't schedule there. After ~5 minutes, Pods on it get evicted (the `unreachable`/`not-ready` taints + 300 s tolerations; topic 58).

**Likely causes → fixes:**
- **kubelet stopped/stuck** → `sudo systemctl start kubelet` (check `systemctl is-enabled kubelet`).
- **Swap on** → `sudo swapoff -a`, remove swap from `/etc/fstab`, start the kubelet.
- **Bad kubelet kubeconfig/certificates** (wrong `server:` port, bad CA path, expired client cert) → restore `/etc/kubernetes/kubelet.conf` (correct `server: https://<api>:6443`, valid CA), `sudo systemctl restart kubelet`.
- **Runtime down** (containerd/CRI-O) → `sudo systemctl start containerd`, then restart the kubelet.
- **Clock skew / can't resolve or reach the API** → restart time sync (`systemd-timesyncd`/chrony); fix `/etc/hosts`/DNS; make sure `:6443` is reachable.
- **Resource pressure** (disk, inodes, memory, PIDs) → free space/inodes (old images: `crictl rmi --prune`, logs), reduce load; the condition clears when below thresholds.
- *(optional)* **cgroup driver mismatch** → align kubelet and runtime (systemd), restart both (topic 75).

**Checks:**

```bash
# cluster view
kubectl get nodes -o wide
kubectl describe node <node> | sed -n '/Conditions:/,/Addresses:/p'    # Ready, MemoryPressure, DiskPressure, PIDPressure
# on the node
sudo systemctl status kubelet --no-pager
sudo journalctl -u kubelet -b -o cat | tail -n 80
sudo systemctl status containerd --no-pager
sudo crictl info | grep -iE 'cgroup|runtimeType' | head
# swap / resources
swapon --show; free -h; df -h; df -i
# API reachability and time
getent hosts <api-server-hostname>
nc -vz <api-server-hostname> 6443         # or: curl -vk https://<api>:6443/healthz
timedatectl
```

```
Conditions:
  Type             Status    Reason                       Message
  MemoryPressure   Unknown   NodeStatusUnknown            Kubelet stopped posting node status.
  Ready            Unknown   NodeStatusUnknown            Kubelet stopped posting node status.
```

`Unknown` + "Kubelet stopped posting node status" = the kubelet isn't talking to the API (stopped, crashed, or can't connect). `False` with a reason (e.g. `KubeletNotReady … container runtime network not ready`) = the kubelet runs but something it depends on doesn't.

**Log clues:**
- swap: `running with swap on is not supported`;
- certificates/CA: `x509: certificate signed by unknown authority`, `no such file or directory` (CA/key), `certificate has expired or is not yet valid`;
- API connectivity: `failed to run Kubelet: … :6443`, `connection refused`, `i/o timeout`;
- runtime/CRI: `container runtime network not ready`, `failed to connect to CRI` / `rpc error … containerd.sock: connect: no such file or directory`;
- pressure: `eviction manager: …`, `imagefs has insufficient space`.

**Expired kubelet client certificate** (the kubelet normally rotates it; if it expired anyway, e.g. the node was off for a long time): on a **worker**, the simplest fix is to drain and **re-join** it (`kubeadm reset` + a fresh `kubeadm join`); on a kubeadm control plane, regenerate `kubelet.conf` as described in the kubeadm certificate docs, then restart the kubelet.

### 2) ContainerCreating / CNI not initialised → topic 29

Summary: `describe pod` shows `FailedCreatePodSandBox … failed to setup network for sandbox` or the node condition `NetworkPluginNotReady … cni plugin not initialized`. Check the CNI DaemonSet (`kubectl -n calico-system get pods -o wide` / `kube-system` for others), `ls /etc/cni/net.d/` and `/opt/cni/bin/` on the node, the CNI agent's logs. Fix: restore the config/binaries or re-apply the CNI add-on, restart its DaemonSet; verify a new Pod gets an IP (`kubectl get pod -o wide`).

### 3) Pod-to-Pod across nodes / ClusterIP unreachable → topic 32 (and 29, 61)

Summary: check the Service and its **EndpointSlices** (`kubectl get endpointslices -l kubernetes.io/service-name=<svc>`; empty = selector/label or readiness problem), `port`/`targetPort` vs the container port, kube-proxy health (`kubectl -n kube-system get ds kube-proxy`, its logs), and **NetworkPolicies** in the namespace. Test with direct Pod IPs first (CNI), then the ClusterIP (kube-proxy), then the name (DNS). Cross-node failures with working same-node traffic point at the overlay (VXLAN/IP-in-IP ports blocked, MTU).

### 4) DNS failures → topic 33

Summary: from a test Pod, `nslookup kubernetes.default` and `nslookup <svc>.<ns>`; check CoreDNS Pods and logs, the `kube-dns` Service's endpoints, `/etc/resolv.conf` in the Pod, the CoreDNS ConfigMap (Corefile), and NetworkPolicies blocking UDP/TCP 53 (topic 61).

### 5) ImagePullBackOff / ErrImagePull → topics 20, 21, 62

Summary: Events tell you `not found` (name/tag), `unauthorized`/`pull access denied` (credentials: `imagePullSecrets`), `toomanyrequests` (rate limits) or `i/o timeout` (node egress/DNS/proxy). Test the pull on the node with `crictl pull <image>`.

### 6) CrashLoopBackOff / OOMKilled / probe failures → topics 19, 21, 22, 41

Summary: `kubectl logs --previous`, `describe` → `Last State: Terminated, Reason: Error/OOMKilled, Exit Code: …`; check command/args/config, memory limits vs usage, liveness probes that kill slow-starting apps (use a **startupProbe**), readiness probes keeping Pods out of Services.

### 7) PVC Pending / volume mount errors (CSI)

**Problem:** the Pod is `Pending` because its PVC isn't `Bound`, or stuck `ContainerCreating` / `CreateContainerError` with mount/permission errors.

**Likely causes → fixes:**
- **No default StorageClass** (PVC without `storageClassName`), or wrong **access mode/size** with static PVs → mark an SC default; align the PVC's size and access modes with what's available (topics 38–39).
- **CSI driver unhealthy** (controller or node Pods not Running, missing CSIDriver, IAM permissions on clouds) → fix the driver; retry.
- **Bad mount spec** (volume/volumeMount **name** mismatch, wrong path/`subPath`, fsType) or **permissions** (non-root app can't write) → correct the names/paths; set `fsType`, use `fsGroup` (topic 59).
- **Zone/topology mismatch** (volume in zone A, Pod forced to zone B), or **RWO volume still attached** to another node (Multi-Attach error) → `WaitForFirstConsumer` binding; wait for/force detach of the old attachment.

**Checks:**

```bash
kubectl get pvc,pv,sc
kubectl describe pvc <pvc>                           # Events: why not Bound?
kubectl get pods -A | grep -i csi                    # driver Pods healthy? (namespace varies by driver)
kubectl get csidrivers,volumeattachments
kubectl describe pod <p> | sed -n '/Mounts:/,/Conditions:/p;/Events:/,$p'
```

```
Events:
  Type    Reason         Age               From                         Message
  Normal  FailedBinding  5s (x3 over 30s)  persistentvolume-controller  no persistent volumes available for this claim and no storage class is set
```

```
Warning  FailedAttachVolume  …  Multi-Attach error for volume "pvc-3c…" Volume is already exclusively attached to one node and can't be attached to another
Warning  FailedMount         …  MountVolume.SetUp failed for volume "data" : … permission denied
```

A PVC `Pending` with `WaitForFirstConsumer` and the message `waiting for first consumer to be created before binding` is **normal** until a Pod using it is scheduled.

**Fix (minimal, then verify):**

```bash
kubectl annotate sc <sc-name> storageclass.kubernetes.io/is-default-class=true --overwrite   # make an SC default
kubectl delete pvc <pvc> && kubectl apply -f <pvc>.yaml       # storageClassName is immutable on an existing PVC: recreate it
kubectl delete pod <p> && kubectl apply -f <pod>.yaml          # retry the workload
kubectl get pvc; kubectl get pod <p> -o wide
```

**Example:** no default SC → PVC `Pending` → mark an SC as default → PVC `Bound` → Pod `Running`.

> ℹ️ **Note:** marking a StorageClass default **after** a PVC was created without `storageClassName` may not help that PVC on older clusters: the default is applied at **admission** time (the `DefaultStorageClass` plugin; topic 58). Recent versions retroactively assign the default to unbound PVCs that have no class, but deleting and recreating the PVC (as above) works everywhere.

### 8) NodePort / LoadBalancer not reachable

**Problem:** `curl http://<nodeIP>:<nodePort>` (or the LB address) fails.

**Likely causes → fixes:**
- **Selector/targetPort mismatch** → match the Service **selector** to the Pod labels; set the right **targetPort**.
- **`externalTrafficPolicy: Local` with no local endpoint** → run a backend on that node or use **`Cluster`** (topic 31). (Local keeps the client IP but only nodes with endpoints answer; cloud LBs health-check `healthCheckNodePort` to skip the others.)
- **kube-proxy rules missing** (DaemonSet unhealthy) → check/restart kube-proxy.
- **Firewall / security group** blocks 30000–32767, or the LB's health checks fail → open the range / fix health checks.
- **LoadBalancer `<pending>`** → no cloud controller (kind/bare metal: MetalLB, cloud-provider-kind).

**Checks:**

```bash
kubectl describe svc <svc>                                    # type, nodePort, targetPort, externalTrafficPolicy
kubectl get endpointslices -l kubernetes.io/service-name=<svc> -o wide   # endpoints? on that node (for Local)?
kubectl get pods -l <selector> -o wide                        # which nodes host backends
kubectl -n kube-system get ds kube-proxy -o wide
kubectl -n kube-system logs ds/kube-proxy --tail=20
# on the node: is there a rule for the nodePort?
sudo nft list ruleset 2>/dev/null | grep -w <nodePort> || sudo iptables-save | grep -w <nodePort>
```

> ✅ **Correct today** ([C1](#c1)): in iptables/nftables mode, NodePorts are implemented purely as **NAT rules in the kernel**; current kube-proxy doesn't hold listening sockets on NodePorts, so `ss` normally shows **nothing** even when everything works. Check the **rules** instead (above), and `kubectl get endpointslices` (Day 58 used `kubectl get endpoints`, which still works but the Endpoints API is deprecated in favour of EndpointSlices; topic 32).

**Fix (minimal, then verify):**

```bash
kubectl patch svc <svc> -p '{"spec":{"selector":{"app":"web"}}}'              # selector fix (or edit/reapply)
kubectl patch svc <svc> --type=json -p '[{"op":"replace","path":"/spec/ports/0/targetPort","value":8080}]'
kubectl patch svc <svc> -p '{"spec":{"externalTrafficPolicy":"Cluster"}}'     # or schedule a Pod on that node
kubectl -n kube-system rollout restart ds/kube-proxy                            # if kube-proxy is unhealthy
curl -sS -m 3 http://<nodeIP>:<nodePort>/
```

**Example:** a Service with `externalTrafficPolicy: Local`, but the node you curl has **no endpoint** → place a Pod there (or switch to `Cluster`) → the NodePort works.

### A combined drill: one broken chain

```bash
kubectl run t --rm -it --image=busybox:1.37 --restart=Never -- sh
/ # wget -qO- -T 3 http://10.244.2.7:8080        # Pod IP direct: tests CNI
/ # wget -qO- -T 3 http://10.96.120.15            # ClusterIP: tests kube-proxy + endpoints
/ # nslookup web.shop.svc.cluster.local           # DNS
/ # wget -qO- -T 3 http://web.shop                # all together
```

The **first step that fails** tells you the layer.

**Conclusion:** data-plane issues aren't mysterious with a disciplined path: **kubelet + CRI** → Pods get **IPs** (CNI) → Services have **endpoints** and **kube-proxy** programs rules → **DNS** from inside a Pod → **PVC/CSI** → the **app** (probes, resources, args). Read Events before editing, apply the smallest fix that turns signals green, re-test with a busybox Pod. **Fast, minimal, verified.**

---

## Hands-on lab

On kind (`docker exec` into nodes) or a kubeadm cluster, break and fix one at a time:
1. `systemctl stop kubelet` on a worker → NotReady; then `stop containerd` → different condition message; restore both.
2. Move `/etc/cni/net.d/*` away on a worker → new Pods on it stick in ContainerCreating; restore.
3. Scale CoreDNS to 0 → DNS fails in Pods; restore.
4. Service selector typo → empty EndpointSlice; fix.
5. PVC with a non-existent StorageClass → Pending; fix by recreating with a valid class.
6. NodePort Service with `externalTrafficPolicy: Local`, curl a node without a backend (fails) and one with (works).
7. Fill a node's disk in a lab VM (`fallocate -l 20G /var/fill`) → DiskPressure, evictions, `NoSchedule` taint; clean up.

---

## Where you'll use it

- **On-call**: most cluster incidents live here.
- **CKA**: Node NotReady, kubelet fixes, Services without endpoints, DNS, PVC binding, NodePort issues.
- **Post-incident hardening**: alerts on node conditions, CoreDNS errors, PVC Pending (topic 80).

---

## When it breaks

| Symptom | Layer | First command |
|---|---|---|
| Node `NotReady` (Unknown) | kubelet/API connectivity | `systemctl status kubelet`; `journalctl -u kubelet` |
| Node `NotReady` (`network not ready`) | CNI | CNI DaemonSet, `/etc/cni/net.d` |
| `ContainerCreating` forever | CNI or volumes or image | `describe pod` Events |
| ClusterIP times out, Pod IP works | kube-proxy / endpoints | EndpointSlices; kube-proxy logs; rules |
| Names don't resolve | DNS | CoreDNS Pods/logs; policies on port 53 |
| PVC `Pending` | Storage | `describe pvc`; SC; CSI Pods |
| `Multi-Attach error` | Storage | Old attachment; RWO + node move |
| NodePort works on some nodes only | `externalTrafficPolicy: Local`, firewall | Endpoints per node; SGs |

---

## Quick recap

- Data plane = kubelet, runtime, CNI, kube-proxy, CoreDNS, CSI. Debug **in order**: node → Pod IPs → Services/endpoints → DNS → storage → app.
- **NotReady**: kubelet/runtime status and logs, swap, certs/kubeconfig, API reachability, time, pressure, cgroup driver.
- **PVC Pending**: default SC, access modes/size, CSI health, WaitForFirstConsumer, topology, Multi-Attach; recreate PVCs to change class.
- **NodePort**: selector/targetPort, `externalTrafficPolicy: Local` needs a local endpoint, kube-proxy rules (not listening sockets), firewalls.
- Read Events first, smallest fix, verify with a busybox Pod.

### Interview questions

1. **Q: A node shows NotReady. Walk through your checks.**
   A: `describe node` for conditions and messages; on the node, check kubelet and containerd status and kubelet logs (swap, certificates, API connectivity, CRI errors), resources (disk/inodes/memory/PIDs), time sync and reachability of the API on 6443; fix the cause and restart the kubelet.
2. **Q: A NodePort works on one node but not another. Why?**
   A: Likely `externalTrafficPolicy: Local` and the second node has no ready endpoint for the Service, or a firewall/security group differs, or kube-proxy isn't healthy on that node. Check EndpointSlices per node, kube-proxy, and firewall rules.
3. **Q: A PVC is Pending. What could be wrong?**
   A: No matching StorageClass (none set and no default), a CSI driver not running or lacking permissions, size/access modes that no PV matches (static provisioning), topology constraints, or simply `WaitForFirstConsumer` waiting for a Pod. `describe pvc` Events say which.

---

## Appendix: notes on the original course material

This topic was rebuilt from the original CKA course notes. This appendix keeps the audit trail out of your way while you learn.

### Sources

Day 58 (introduction; prerequisites and the note on data plane & prerequisites; quick refresher of the data-plane pieces; how to use the scenarios and the fast-path approach; scenario 1 Node NotReady; scenario 7 PVC/CSI; scenario 8 NodePort/LoadBalancer; conclusion; references). Scenarios 2–6 have their full write-ups in earlier topics and are summarised here: 2 (CNI) → topic 29, 3 (Services/kube-proxy) → topic 32, 4 (DNS) → topic 33, 5 and 6 (images, CrashLoop/OOM/probes) and the approach section → topic 21.

### Corrections to the original notes

Each entry shows what the original said and what's true now. In the topic above, each correction appears as a ✅ **Correct today** line or a ⚠️ **Fix to the original** box, linked to its entry here.

<a id="c1"></a>**C1 · Corrected (Day 58).** the original checks "is kube-proxy **listening** on the nodePort?" with `sudo ss -lnt 'sport = :<nodePort>'`. **Now:** in iptables/nftables mode, NodePorts are implemented purely as **NAT rules in the kernel**; current kube-proxy doesn't hold listening sockets on NodePorts, so `ss` normally shows **nothing** even when everything works. Check the **rules** instead (above), and `kubectl get endpointslices` (Day 58 used `kubectl get endpoints`, which still works but the Endpoints API is deprecated in favour of EndpointSlices; topic 32).
