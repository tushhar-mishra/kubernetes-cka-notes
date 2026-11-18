# 82 · Control-Plane Troubleshooting (kubeadm Clusters)

> **Level:** ★★★ advanced
>
> **You will learn:** how control-plane components run on kubeadm (static Pods and their read-only **mirror Pods**) and therefore how to debug them **without the API** (`crictl`, CRI log files, `journalctl`, the manifests); reading sockets with **`ss`**; and exam-style playbooks for: **API server unreachable**, **etcd unhealthy**, Pods **Pending** (scheduler down vs no feasible node), **controller manager down**, **expired certificates** (check, renew, restart properly), the "**name is reserved**" containerd error, plus a broken kubelet on the control plane and a malformed manifest.
>
> **You need:** topics 07, 12, 42 (static Pods), 53 (cluster certificates), 75 (kubeadm), 77 (etcd), 81 (method).

---

## ⚡ Core: the 10-minute version

**The idea.** On kubeadm, the control plane is **static Pods** run by the kubelet from `/etc/kubernetes/manifests/`. When it breaks, `kubectl` may not work at all, so debug **on the node**:
- `ss -lntp` (is port 6443 listening?);
- `crictl ps -a` and `crictl logs` (is it crashing, and why?);
- the manifest files (typos in flags or paths);
- `journalctl -u kubelet`.

Classic cases: a bad API server flag; etcd unhealthy; Pending Pods with **no events** (scheduler down); Deployments not scaling (controller manager down); **expired certificates** (`kubeadm certs renew all`, then restart the static Pods).

**Picture it.** When the receptionist won't answer the phone, you walk into the building and read the staff register yourself.

**The one thing to know.**

```bash
sudo ss -lntp | grep -E '6443|2379|10257|10259'
sudo crictl ps -a | grep -E 'apiserver|etcd|scheduler|controller'
sudo crictl logs --tail=30 <container-id>
sudo kubeadm certs check-expiration
```

**Try it now** (on a VM cluster, with a backup of the manifests kept **outside** the manifests folder):
1. Break `--etcd-servers` in the API server manifest.
2. Find the problem with `crictl`, and fix it.
3. Do the same for the scheduler's `--kubeconfig`.

**You've got it if you can:** debug a dead API server without kubectl, and renew expired certificates correctly.

> 📖 **That's the core.** On a first pass, do the mini-lab and move on to the next topic. Everything below is the **full reference**: come back for the details, the edge cases, interview answers and exam-level depth.

---

## What it is

On **kubeadm** clusters the control plane is **four static Pods** on each control-plane node:
- the **kubelet** starts them directly from manifests in **`/etc/kubernetes/manifests/`** (`kube-apiserver.yaml`, `kube-controller-manager.yaml`, `kube-scheduler.yaml`, `etcd.yaml`), watching that folder (`staticPodPath` in the kubelet config). They're **not scheduled** by the scheduler;
- **the file is the source of truth**: delete the Pod via the API and it **comes back**; edit/move the file and the kubelet recreates/stops it.

**Mirror Pods:** for each static Pod the kubelet publishes a **read-only mirror Pod** object to the API so `kubectl get pods -n kube-system` shows it.
- **Name**: `<podName>-<nodeName>`, e.g. `kube-apiserver-control-plane`.
- You can **see status and logs** via the mirror Pod, but edits/deletes don't affect the real Pod: change the **file**.
- Mirror Pods carry the annotation `kubernetes.io/config.mirror` and always show their **node**.

**Kubeconfig refresher:** kubectl uses `~/.kube/config` (clusters, users, contexts). On a kubeadm control plane, `/etc/kubernetes/admin.conf` is the admin kubeconfig (copy it to `~/.kube/config`):

```bash
kubectl config get-contexts
kubectl config use-context <name>
kubectl config view --minify
kubectl config set-context --current --namespace dev
```

(Topics 12 and 52 cover kubeconfig fully.)

---

## Why it exists

When the control plane breaks, **kubectl may not work at all**, because it talks to the API server, which may be the broken thing. You have to debug **one level down**, on the node: the container runtime, log files, systemd, and the manifest files. These are also among the most common CKA troubleshooting tasks.

---

## Analogy

The control plane is a **head office** whose staff are hired by the **building manager** (kubelet) from **job descriptions pinned on a noticeboard** (manifest files). If the receptionist (API server) isn't answering the phone, you can't call in to ask what's wrong: you **walk into the building** (SSH), look at the **staff register** (`crictl ps`), read their **diaries** (container logs), and fix the **job description** on the noticeboard; the building manager re-hires them immediately.

---

## How it works inside

### Ports to know (control-plane node)

| Port | Component | Binding (kubeadm default) |
|---|---|---|
| 6443 | kube-apiserver | all interfaces |
| 2379 / 2380 | etcd client / peer | 127.0.0.1 + node IP / node IP |
| 2381 | etcd metrics/health | 127.0.0.1 |
| 10250 | kubelet API | all interfaces |
| 10248 | kubelet healthz | 127.0.0.1 |
| 10257 | kube-controller-manager (secure) | 127.0.0.1 |
| 10259 | kube-scheduler (secure) | 127.0.0.1 |
| 10249 / 10256 | kube-proxy metrics / healthz | 127.0.0.1 / all |

### The `ss` primer

**`ss` = socket statistics**: listening sockets and live connections (the modern `netstat`; Linux notes: networking tools, topics [095](../../linux-mastery-rhel/notes/095-tcp-ip-deep-dive.md) and [096](../../linux-mastery-rhel/notes/096-packet-capture.md)).

| Flag | Meaning |
|---|---|
| `-t` | TCP only |
| `-l` | **listening** sockets only (servers waiting for connections) |
| `-p` | show the owning **process** |
| `-n` | numeric (no DNS/service-name lookups) |
| `-s` | summary totals |

**Reading `ss -tlpn` on a control-plane node (Day 57's output):**

```
State   Recv-Q  Send-Q   Local Address:Port    Peer Address:Port  Process
LISTEN  0       4096         127.0.0.1:9099         0.0.0.0:*      users:(("calico-node",pid=2752,fd=6))
LISTEN  0       4096         127.0.0.1:9098         0.0.0.0:*      users:(("calico-typha",pid=1578,fd=7))
LISTEN  0       4096         127.0.0.1:2381         0.0.0.0:*      users:(("etcd",pid=1183,fd=14))
LISTEN  0       4096         127.0.0.1:2379         0.0.0.0:*      users:(("etcd",pid=1183,fd=7))
LISTEN  0       4096         127.0.0.1:10249        0.0.0.0:*      users:(("kube-proxy",pid=1538,fd=18))
LISTEN  0       4096         127.0.0.1:10248        0.0.0.0:*      users:(("kubelet",pid=927,fd=14))
LISTEN  0       4096         127.0.0.1:10257        0.0.0.0:*      users:(("kube-controller",pid=4600,fd=3))
LISTEN  0       4096         127.0.0.1:10259        0.0.0.0:*      users:(("kube-scheduler",pid=4561,fd=3))
LISTEN  0       4096         127.0.0.1:34961        0.0.0.0:*      users:(("containerd",pid=576,fd=10))
LISTEN  0       4096        127.0.0.54:53           0.0.0.0:*      users:(("systemd-resolve",pid=350,fd=17))
LISTEN  0       4096           0.0.0.0:22           0.0.0.0:*      users:(("sshd",pid=3924,fd=3),("systemd",pid=1,fd=87))
LISTEN  0       4096     172.31.35.156:2380         0.0.0.0:*      users:(("etcd",pid=1183,fd=6))
LISTEN  0       4096     172.31.35.156:2379         0.0.0.0:*      users:(("etcd",pid=1183,fd=8))
LISTEN  0       4096     127.0.0.53%lo:53           0.0.0.0:*      users:(("systemd-resolve",pid=350,fd=15))
LISTEN  0       4096                 *:5473               *:*      users:(("calico-typha",pid=1578,fd=6))
LISTEN  0       4096              [::]:22              [::]:*      users:(("sshd",pid=3924,fd=4),("systemd",pid=1,fd=88))
LISTEN  0       4096                 *:10256              *:*      users:(("kube-proxy",pid=1538,fd=17))
LISTEN  0       4096                 *:10250              *:*      users:(("kubelet",pid=927,fd=16))
LISTEN  0       4096                 *:6443               *:*      users:(("kube-apiserver",pid=7103,fd=3))
```

- **Local Address:Port**: where **this machine** listens. `127.0.0.1` = only reachable from the node itself; `0.0.0.0`/`*`/`[::]` = all interfaces; a specific IP = that interface.
- **Peer Address:Port** on a LISTEN row is always a wildcard (`0.0.0.0:*`): no remote yet. Firewalls/SGs still decide who may actually connect.
- **Recv-Q / Send-Q** on LISTEN rows: current accept-queue length vs backlog limit (4096).

**Established connections: drop `-l`** to see who's actually talking:

```bash
ss -tnp state established | grep 172.31.44.224
```

Control-plane view:

```
[::ffff:172.31.35.156]:6443   [::ffff:172.31.44.224]:32192   # worker → API server
[::ffff:172.31.35.156]:10250  [::ffff:172.31.44.224]:40342   # worker/Pod (e.g. metrics-server) → CP kubelet
[::ffff:172.31.35.156]:5473   [::ffff:172.31.44.224]:47678   # Calico Felix ↔ Typha
```

Worker-1 view:

```
172.31.44.224:34778   172.31.35.156:6443    # worker (kubelet/kube-proxy) → API server
172.31.44.224:10250   192.168.226.86:45078  # this kubelet serving a scraper (a Pod IP)
172.31.44.224:47662   172.31.35.156:5473    # Felix → Typha
172.31.44.224:22      223.228.207.145:35358 # your SSH session
```

**Rule:** if the **Local** port is a well-known server port (6443, 10250), this row is the **server side**; if the **Peer** port is well-known, this machine is the **client**. (`::ffff:` = an IPv4 address shown in IPv6 form.)

### Where to look when kubectl doesn't work

```bash
sudo crictl ps -a | grep -E 'apiserver|etcd|scheduler|controller'   # running? restarting? exited?
sudo crictl logs --tail=50 <container-id>                            # logs of a (possibly exited) container
sudo ls /var/log/containers/ | grep apiserver                        # or read the CRI log files directly
sudo tail -n 50 /var/log/containers/kube-apiserver-*.log
sudo journalctl -u kubelet --since "10 min ago" | tail -50           # kubelet errors (bad manifests, image pulls, certs)
sudo systemctl status kubelet containerd --no-pager
```

(`crictl` talks to containerd via `/run/containerd/containerd.sock`; if it complains about endpoints, set it once in `/etc/crictl.yaml`: `runtime-endpoint: unix:///run/containerd/containerd.sock`.)

---

## Commands / YAML

### Scenario 1: API server unreachable / kubectl times out

**Problem:** `kubectl get nodes` hangs, or says `The connection to the server 172.31.35.156:6443 was refused` / `i/o timeout`.

**Likely causes:**
- bad flags/paths in `/etc/kubernetes/manifests/kube-apiserver.yaml` (certificate/key paths, `--advertise-address`, `--client-ca-file`, `--etcd-servers`, admission plugins);
- port 6443 not listening, or taken by another process;
- host firewall / cloud security group blocking 6443;
- **etcd** unreachable (scenario 2).

**Checks:**

```bash
sudo ss -lntp | grep 6443                                   # listening?
sudo crictl ps -a | grep kube-apiserver                     # running or crash-looping?
sudo tail -n 50 /var/log/containers/kube-apiserver-*.log    # what is it complaining about?
sudo sed -n '1,120p' /etc/kubernetes/manifests/kube-apiserver.yaml     # typos/paths/addresses
```

Typical log lines and meanings:

```
E… run.go: "command failed" err="open /etc/kubernetes/pki/apiserver.crtx: no such file or directory"   ← bad path
W… logging.go: [core] grpc: addrConn.createTransport failed to connect to {Addr: "127.0.0.1:2379"…}: connection refused   ← etcd down
Error: unknown flag: --enable-admission-plugin                                                            ← typo in a flag name
```

**Fix:** correct cert/key paths (under `/etc/kubernetes/pki/`), set `--advertise-address` to the node's reachable IP, clean up `--enable-admission-plugins`; save: the **kubelet restarts** the static Pod automatically (watch `crictl ps`, up to ~1 minute); open 6443 in firewalls/SGs.

**Example:** `--advertise-address=127.0.0.1` in a multi-node cluster: other nodes and remote kubectl can't reach the API. Set it to the node IP (e.g. `172.31.x.x`).

> **Malformed manifest (YAML syntax error):** the kubelet can't parse the file, so the Pod **disappears** entirely (no container, nothing in `crictl ps -a`). Look in `journalctl -u kubelet` for `Could not process manifest file … error converting YAML to JSON`. Before editing static manifests, **copy the original** outside the manifests folder (`sudo cp kube-apiserver.yaml /root/kube-apiserver.yaml.bak`). Don't leave backups **inside** `/etc/kubernetes/manifests/`: the kubelet would try to run them too.

### Scenario 2: etcd unhealthy → API server crash-loops

**Problem:** `kube-apiserver` logs show timeouts to `127.0.0.1:2379`, or the `etcd` Pod is in `CrashLoopBackOff`.

**Likely causes → fixes:**
- **Wrong `--data-dir` / missing hostPath mount** → point it back at the host-mounted path (kubeadm default **`/var/lib/etcd`**) and make sure the volumeMount covers it; permissions **root:root 0700**.
- **localhost removed from client URLs** (`--listen-client-urls`/`--advertise-client-urls`) → for stacked etcd keep **`https://127.0.0.1:2379`**, or change the API server's `--etcd-servers` to the node IP (and the certificate SANs must match).
- **TLS cert/key path or SAN mismatch** → fix paths in `etcd.yaml`; wrong SANs → renew etcd certificates with kubeadm (or reissue with correct SANs).
- **Disk full / slow I/O** on the data directory → free space; investigate I/O stalls; once healthy, `etcdctl defrag` if the database is large/fragmented (topic 77).
- **Firewall/SG** (multi-node/HA etcd) → allow **2379/tcp** (client) and **2380/tcp** (peer) between members.
- **Clock skew** (TLS and Raft misbehave) → sync time (chrony/systemd-timesyncd).

**Quick checks:**

```bash
sudo crictl ps -a | grep etcd                         # running/crashing?
sudo tail -n 100 /var/log/containers/etcd-*.log       # path/TLS/disk errors
sudo ss -lntp | grep -E ':2379|:2380'                 # listening where?
df -h /var/lib/etcd                                    # disk full?
sudo grep -E -- '--data-dir|listen-client-urls|advertise-client-urls' /etc/kubernetes/manifests/etcd.yaml

export ETCDCTL_API=3
export ETCDCTL_CACERT=/etc/kubernetes/pki/etcd/ca.crt
export ETCDCTL_CERT=/etc/kubernetes/pki/etcd/server.crt
export ETCDCTL_KEY=/etc/kubernetes/pki/etcd/server.key
sudo -E etcdctl endpoint health                        # liveness (default endpoint 127.0.0.1:2379)
sudo -E etcdctl endpoint status --write-out=table      # DB size, leader, errors/alarms
```

(`sudo -E` keeps the exported variables; or run as root.)

**Log clues:**
- **Bad data dir / not mounted**: `"data-dir":"…"` and `"opened backend db","path":"…"` showing an **unexpected path**; a tiny DB plus "started as single-node" = a **fresh, empty** store (the cluster looks wiped!). Fix the path; **don't** let it run on the empty store.
- **TLS**: `x509` errors, `no such file or directory` for cert/key paths.
- **Connectivity mismatch**: the API server times out to `127.0.0.1:2379` while etcd listens only on the node IP (or vice versa).
- **Disk/I/O**: `no space left on device`, slow WAL sync warnings (`slow fdatasync`), filesystem errors.

### Scenario 3: Pods stay Pending

**Problem:** new Pods stay `Pending` with no `Scheduled` event.

**Fast triage (20 seconds):**

```bash
kubectl get pod <p> -o wide                         # NODE <none> = unscheduled
kubectl describe pod <p> | sed -n '/Events:/,$p'     # "0/N nodes are available: …" reasons, or NO events at all
kubectl -n kube-system get pods | grep scheduler     # scheduler (static Pod) healthy?
```

**No events at all** → the scheduler isn't working (A). **`FailedScheduling` events** → the scheduler works but can't place the Pod (B).

**A) Scheduler unavailable.** Likely causes: bad `--kubeconfig` in `kube-scheduler.yaml`; health/secure port mis-set (e.g. `--secure-port=0`) so its **liveness probe** fails and it's restarted repeatedly; clock skew breaking leader election; image typo.

```bash
kubectl -n kube-system logs -l component=kube-scheduler --tail=80       # works even if the Pod name changes
sudo grep -E 'kubeconfig|secure-port|image:' /etc/kubernetes/manifests/kube-scheduler.yaml
timedatectl status                                                       # NTP synchronized?
sudo crictl ps -a | grep scheduler; sudo crictl logs --tail=30 <id>      # if the API is also flaky
```

Fix: `--kubeconfig=/etc/kubernetes/scheduler.conf` (and the file exists); restore `--secure-port=10259`; sync time; wait for leader election. **Example:** `--kubeconfig` pointing to a wrong path → the scheduler can't authenticate → nothing gets scheduled. Revert it.

**B) Scheduler running, no feasible node.** Common blockers:
- **insufficient resources** (CPU/memory/ephemeral-storage **requests**);
- **taints** without tolerations (e.g. the control-plane taint);
- **label/affinity/topology** mismatch (nodeSelector, affinity, anti-affinity, spread);
- **PVC Pending** / storage topology (StorageClass, access mode, zone);
- **extended resources** (GPUs, hugepages) not present.

```bash
kubectl describe pod <p> | sed -n '/Events:/,$p'
kubectl get nodes -o custom-columns=NAME:.metadata.name,TAINTS:.spec.taints
kubectl get pvc
kubectl describe node <n> | sed -n '/Allocated resources/,/Events/p'
```

```
Warning  FailedScheduling  0/3 nodes are available: 1 node(s) had untolerated taint {node-role.kubernetes.io/control-plane: }, 2 Insufficient cpu. preemption: 0/3 nodes are available: 1 Preemption is not helpful for scheduling, 2 No preemption victims found for incoming pod.
```

Fix: lower requests / free capacity / add nodes; add tolerations or adjust taints; correct labels/affinity/spread; fix the StorageClass/zone/access mode so the PVC binds (topics 41–47, 38–39).

### Scenario 4: controller manager down → nothing reconciles

**Problem:** you create/scale a Deployment and **no Pods appear**. Other controllers stall too: **EndpointSlices** don't update, **CSR approvals** hang (new nodes can't join), node-lifecycle taints don't change, **garbage collection** of owned objects stops, namespace deletion hangs.

**Likely causes:** controller manager not running / `CrashLoopBackOff` / not leader; bad `--kubeconfig`; `--secure-port=0` (liveness probe fails); missing/mispointed **cluster-signing** cert/key (`--cluster-signing-cert-file`, `--cluster-signing-key-file`); clock skew.

**Checks:**

```bash
kubectl -n kube-system get pods -l component=kube-controller-manager      # healthy?
kubectl -n kube-system logs -l component=kube-controller-manager --tail=80   # auth/flag/probe/TLS errors
sudo grep -E 'kubeconfig|secure-port|cluster-signing|feature-gates' /etc/kubernetes/manifests/kube-controller-manager.yaml
kubectl get leases -n kube-system | grep controller-manager               # an active leader?
sudo ss -lntp | grep 10257                                               # health port listening?
```

**Prove reconciliation has stalled:**

```bash
kubectl create deploy cm-test --image=nginx --replicas=1
kubectl scale deploy cm-test --replicas=3
kubectl get deploy cm-test; kubectl get rs,pods -l app=cm-test
kubectl describe deploy cm-test | sed -n '/Events:/,$p'
```

```
NAME      READY   UP-TO-DATE   AVAILABLE   AGE
cm-test   0/3     0            0           2m          ← desired 3, nothing created
No resources found in default namespace.               ← not even a ReplicaSet
Events:  <none>                                         ← no "Scaled up replica set …"
```

(If the controller manager died **after** the ReplicaSet existed, you'd see a ReplicaSet stuck at its old size.)

**Fix:** `--kubeconfig=/etc/kubernetes/controller-manager.conf` (and the certificates it references exist); `--secure-port=10257`; cluster-signing flags pointing at valid files in `/etc/kubernetes/pki/` (`ca.crt`/`ca.key`); sync time; save → the kubelet restarts the Pod; watch the Lease get a holder.

**Example:** `--cluster-signing-key-file` missing → CSRs stay **Pending**, new nodes can't join. Restore the key/path; the controller manager restarts, CSRs are approved and signed, Deployments scale again.

### Scenario 5: certificates expired (very common exam task, Day 57)

**Problem:** `x509: certificate has expired or is not yet valid` in API server/kubelet logs; nodes go `NotReady`; kubectl fails TLS.

**Causes:** kubeadm-managed certificates passed **NotAfter** (they last **1 year**; the CA 10 years); **clock skew** makes valid certificates look expired or not yet valid.

**Checks:**

```bash
sudo kubeadm certs check-expiration
sudo openssl x509 -in /etc/kubernetes/pki/apiserver.crt -noout -enddate
sudo journalctl -u kubelet -p err --since "2h" | grep -i x509
```

```
CERTIFICATE                EXPIRES                  RESIDUAL TIME   CERTIFICATE AUTHORITY   EXTERNALLY MANAGED
admin.conf                 Nov 18, 2026 10:00 UTC   <invalid>       ca                      no
apiserver                  Nov 18, 2026 10:00 UTC   <invalid>       ca                      no
…
CERTIFICATE AUTHORITY      EXPIRES                  RESIDUAL TIME   EXTERNALLY MANAGED
ca                         Nov 15, 2035 10:00 UTC   8y              no
```

**Simulating expiry in a lab:** stop time sync and jump the clock forward:

```bash
sudo systemctl stop systemd-timesyncd 2>/dev/null || true      # or:
sudo systemctl stop chronyd 2>/dev/null || true
sudo date -s '2027-12-01 12:00:00'                              # past the certs' NotAfter
sudo kubeadm certs check-expiration                             # now "expired"
sudo journalctl -u kubelet -p err --since "15m" | grep -i x509
```

(Only on a disposable VM: never on a kind node, which shares your host's clock; topic 59's `SYS_TIME` warning.)

**Fix (kubeadm):**

```bash
sudo kubeadm certs renew all                     # renews ALL kubeadm-managed certs, incl. those embedded in admin.conf, super-admin.conf, controller-manager.conf, scheduler.conf
# restart the static control-plane Pods so they load the new certificates:
sudo mkdir -p /etc/kubernetes/manifests-off
sudo mv /etc/kubernetes/manifests/*.yaml /etc/kubernetes/manifests-off/
sleep 20                                          # kubelet stops them
sudo mv /etc/kubernetes/manifests-off/*.yaml /etc/kubernetes/manifests/
mkdir -p ~/.kube && sudo cp -f /etc/kubernetes/admin.conf ~/.kube/config && sudo chown $(id -u):$(id -g) ~/.kube/config
kubectl get nodes
sudo kubeadm certs check-expiration
```

> ✅ **Correct today** ([C1](#c1)): > - `kubeadm certs renew all` **already renews** the client certificates embedded in `admin.conf`, `super-admin.conf`, `controller-manager.conf` and `scheduler.conf`. `init phase kubeconfig all` adds nothing (it reuses existing valid kubeconfig files).
> - **Restarting the kubelet does not restart static Pods**: their containers keep running with the **old** certificates loaded in memory. The kubeadm docs say to **restart the control-plane Pods**, e.g. move the manifests out of `/etc/kubernetes/manifests/` for ~20 seconds and back (or `crictl stop` each container). Restart etcd too if its certificates were renewed.
> - The kubelet's **own** client certificate (`/var/lib/kubelet/pki/kubelet-client-current.pem`) isn't renewed by `kubeadm certs renew`: the kubelet **rotates it automatically** (`rotateCertificates: true`). If it already expired, see topic 83's node scenario.

**Restore real time, then renew again:** if you renewed while the clock was in the future, the new certificates have **NotBefore in 2027**, so they're "not yet valid" once time is correct. Re-renew after fixing the clock:

```bash
sudo timedatectl set-ntp true                                   # re-enable NTP (systemd-timesyncd)…
sudo systemctl start chronyd 2>/dev/null || true                # …or chrony, if that's your agent
timedatectl status                                               # "System clock synchronized: yes", correct date
sudo kubeadm certs renew all
# restart the control-plane static Pods again (move manifests out and back, as above)
sudo cp -f /etc/kubernetes/admin.conf ~/.kube/config
kubectl get nodes; sudo kubeadm certs check-expiration
```

> ⚠️ **Fix to the original** ([C2](#c2)): the original restores time with `sudo date -s 'NOW'`. **`date -s now` sets the clock to what the system *currently* thinks "now" is**, which is still the fake 2027 date: it changes nothing. Re-enable NTP (`timedatectl set-ntp true`, or start chrony) or set the correct date explicitly.

**Prevention:** `kubeadm upgrade apply` renews certificates automatically (topic 76), so clusters upgraded at least yearly rarely expire. Otherwise monitor expiry (`kubeadm certs check-expiration` in a cron job, or Prometheus alerts on `apiserver_client_certificate_expiration_seconds`).

### After renewal: controller manager and scheduler NotReady

**Why it happens:**
1. **Time jump + reissued certificates**: until you re-renew with the correct time and the components reload their kubeconfigs, they fail to authenticate (NotBefore/NotAfter mismatch).
2. **Stale containerd state after many rapid restarts**: occasionally containerd keeps a dead container record that still "owns" the name, and the kubelet can't create the new container:

```
Error: failed to reserve container name "kube-controller-manager_kube-controller-manager-control-plane_kube-system_…": name "…" is reserved for "<old-container-id>"
```

That's a runtime bookkeeping issue, not an image or manifest bug.

**Fix (controller manager; the scheduler is identical with `kube-scheduler`):**

```bash
sudo crictl ps -a | grep -i kube-controller-manager           # all CM containers, running or exited
sudo crictl rm -f <CONTAINER_ID_1> <CONTAINER_ID_2>           # remove the stuck/old ones
sudo crictl pods | grep -i kube-controller-manager            # a stuck Pod sandbox too?
sudo crictl stopp <POD_SANDBOX_ID>; sudo crictl rmp <POD_SANDBOX_ID>
sudo systemctl restart kubelet                                # resync static Pods
kubectl -n kube-system get pods -l component=kube-controller-manager
kubectl -n kube-system logs -l component=kube-controller-manager --tail=50
kubectl -n kube-system get lease | grep controller-manager
```

**Also double-check:** certificate dates are sane and kubeconfigs reference fresh certificates (`kubeadm certs check-expiration`), and `kubectl get nodes` works.

**TL;DR:** "name is reserved for <ID>" = stale containerd state after rapid restarts → `crictl rm -f` the old containers (and sandbox if needed) → restart the kubelet. Make sure certificates were renewed with the **correct** system time.

### Bonus scenario: the kubelet on the control plane is broken

Symptoms: the control-plane Node is `NotReady`; static Pods may keep running (containers survive), but nothing restarts them and their mirror Pods show stale status.

```bash
sudo systemctl status kubelet --no-pager
sudo journalctl -u kubelet -n 50 --no-pager
```

Typical causes: a typo in `/var/lib/kubelet/config.yaml` or the systemd drop-in (`/usr/lib/systemd/system/kubelet.service.d/10-kubeadm.conf`, which loads `/var/lib/kubelet/kubeadm-flags.env`), a wrong `--kubeconfig`, the binary path, swap turned back on, containerd down. Fix, then `sudo systemctl daemon-reload && sudo systemctl restart kubelet`.

### Conclusion: the rhythm

1. **Probe the socket** first: `ss -lntp` for 6443/2379/10257/10259; drop `-l` to see live peers.
2. **Read the logs where the process runs**: `/var/log/containers/*.log`, `crictl logs`, `journalctl -u kubelet`.
3. **Trust the manifest**: fix flags and paths in `/etc/kubernetes/manifests/` (backup first, outside the folder); the kubelet reconciles.
4. **Validate etcd** before blaming the API server.
5. **Check leader election** (Leases) and **time sync** for scheduler/controller manager.
6. **Rotate kubeadm certificates** when x509 errors appear, restart the control-plane Pods, verify.

Keep changes small, verify after each step.

---

## Hands-on lab

On a kubeadm VM cluster (topic 75), with a backup of `/etc/kubernetes/manifests` outside that folder:
1. Change the API server's `--etcd-servers` port to 2380; watch kubectl fail; find it with `crictl` + logs; fix it.
2. Break the scheduler's `--kubeconfig` path; create a Pod; observe Pending with **no events**; fix.
3. Set `--secure-port=0` on the controller manager; watch liveness restarts; prove reconciliation stalls with `cm-test`; fix.
4. Put a YAML syntax error in `kube-scheduler.yaml`; notice the Pod vanishes; find the kubelet's parse error.
5. Change etcd's `--data-dir` to `/var/lib/etcd-new`; observe the "fresh store" symptoms (cluster looks empty!); revert.
6. Do the certificate expiry demo (time jump), the correct renewal + Pod restart, then restore time with NTP and renew again.
7. On kind, do the read-only parts: `docker exec -it lab-control-plane bash`, then `crictl ps`, `ss -tlpn`, `ls /etc/kubernetes/manifests`, `kubeadm certs check-expiration`.

---

## Where you'll use it

- **CKA** troubleshooting tasks (control plane, certificates, static Pods).
- **Self-managed production clusters**: certificate expiry is a classic outage; manifest edits during upgrades.
- **Understanding managed clusters**: the same components exist, hidden by the provider.

---

## When it breaks

| Symptom | Cause | Fix |
|---|---|---|
| `connection refused` to 6443 | API server not running | `crictl ps -a`, logs, manifest |
| `i/o timeout` to 6443 | Firewall/SG or wrong IP in kubeconfig | Open 6443; check `server:` |
| API server crash-loops: etcd connection refused | etcd down or URL mismatch | Scenario 2 |
| Cluster suddenly "empty" | etcd started on a new/empty data dir | Fix `--data-dir`/hostPath (don't write anything meanwhile) |
| Pods Pending, no events | Scheduler down | Scenario 3A |
| Deployments don't scale, CSRs Pending | Controller manager down/no leader/signing key missing | Scenario 4 |
| `x509: certificate has expired or is not yet valid` | Expired certs or clock skew | Check time first; `kubeadm certs renew all`; restart CP Pods |
| Static Pod disappeared after an edit | YAML error | `journalctl -u kubelet`; restore backup |
| Duplicate API server / weird behaviour | A backup manifest left inside `/etc/kubernetes/manifests/` | Move backups elsewhere |
| `name … is reserved for …` | Stale containerd state | `crictl rm -f`, `crictl rmp`, restart kubelet |

---

## Quick recap

- kubeadm control plane = static Pods from `/etc/kubernetes/manifests/` (file = source of truth); kubectl sees **mirror Pods** (`<name>-<node>`, read-only).
- When the API is down, debug on the node: `ss -lntp`, `crictl ps -a` / `crictl logs`, `/var/log/containers`, `journalctl -u kubelet`, manifests.
- Scenarios: API server (flags/paths/advertise address/etcd), etcd (data dir, URLs, TLS, disk, ports, time), Pending (no events = scheduler down; `FailedScheduling` = constraints), controller manager (kubeconfig, secure port, signing keys, lease), certificates (`check-expiration`, `renew all`, **restart the static Pods**, fix time with NTP and re-renew), stale containerd names.

### Interview questions

1. **Q: kubectl can't connect to the API server. How do you troubleshoot on a kubeadm cluster?**
   A: SSH to the control plane; check whether 6443 is listening (`ss -lntp`), whether the kube-apiserver container is running or crash-looping (`crictl ps -a`, `crictl logs`), read its errors, inspect `/etc/kubernetes/manifests/kube-apiserver.yaml` for bad paths/flags, check etcd health, and check firewalls; the kubelet restarts the static Pod after you fix the manifest.
2. **Q: How do you renew expired kubeadm certificates?**
   A: Verify the clock is correct, check with `kubeadm certs check-expiration`, run `kubeadm certs renew all` (which also renews the certs in the component kubeconfigs), restart the control-plane static Pods (move the manifests out and back), copy the new `admin.conf` to your kubeconfig, and verify.
3. **Q: You deleted the kube-scheduler Pod with kubectl and it came back. Why?**
   A: It's a static Pod managed by the kubelet from a manifest file; the API object is only a mirror Pod. To stop or change it you edit or remove the file in `/etc/kubernetes/manifests/`.

---

## Appendix: notes on the original course material

This topic was rebuilt from the original CKA course notes. This appendix keeps the audit trail out of your way while you learn.

### Sources

Day 57 (introduction; prerequisites; control-plane basics: static Pods, mirror Pods; kubeconfig refresher; scenario 1 API server unreachable with example; the `ss` primer: flags, reading `ss -tlpns`, local vs peer, established connections with CP and worker views; scenario 2 etcd: causes → fixes, quick checks, log clues; scenario 3 Pending: fast triage, A) scheduler unavailable, B) no feasible node; scenario 4 controller manager down: causes, checks, proving reconciliation stalled, fix, example; scenario 5 certificates expired: checks, the time-jump demo, the fix, restoring time and re-renewing; after renewal: controller-manager and scheduler NotReady, "name is reserved", fixes, double-checks, TL;DR; conclusion; references).

### Corrections to the original notes

Each entry shows what the original said and what's true now. In the topic above, each correction appears as a ✅ **Correct today** line or a ⚠️ **Fix to the original** box, linked to its entry here.

<a id="c1"></a>**C1 · Corrected (Day 57).** the original fix runs `kubeadm certs renew all`, then **`kubeadm init phase kubeconfig all`** "to re-generate kubeconfigs", then **`systemctl restart kubelet`** "so static pods reload new certs". **Now:**
- `kubeadm certs renew all` **already renews** the client certificates embedded in `admin.conf`, `super-admin.conf`, `controller-manager.conf` and `scheduler.conf`. `init phase kubeconfig all` adds nothing (it reuses existing valid kubeconfig files).
- **Restarting the kubelet does not restart static Pods**: their containers keep running with the **old** certificates loaded in memory. The kubeadm docs say to **restart the control-plane Pods**, e.g. move the manifests out of `/etc/kubernetes/manifests/` for ~20 seconds and back (or `crictl stop` each container). Restart etcd too if its certificates were renewed.
- The kubelet's **own** client certificate (`/var/lib/kubelet/pki/kubelet-client-current.pem`) isn't renewed by `kubeadm certs renew`: the kubelet **rotates it automatically** (`rotateCertificates: true`). If it already expired, see topic 83's node scenario.

<a id="c2"></a>**C2 · Corrected (Day 57).** the original restores time with `sudo date -s 'NOW'`. **`date -s now` sets the clock to what the system *currently* thinks "now" is**, which is still the fake 2027 date: it changes nothing. Re-enable NTP (`timedatectl set-ntp true`, or start chrony) or set the correct date explicitly.
