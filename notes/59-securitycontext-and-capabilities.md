# 59 · securityContext and Linux Capabilities: Running Containers With Least Privilege

> **Level:** ★★ intermediate
>
> **You will learn:** why a plain Pod often runs as **root** and why that's risky; every important `securityContext` field (`runAsUser`, `runAsGroup`, `runAsNonRoot`, `fsGroup`, `supplementalGroups`, `allowPrivilegeEscalation`, `privileged`, `readOnlyRootFilesystem`, `capabilities`) and **where** each can be set (Pod vs container); how they map to Linux (UIDs, setuid, `no_new_privs`, mount flags, capability sets) and to `docker run` flags; the writable-volume-with-read-only-root pattern; dropping **ALL** capabilities and adding back only what's needed; and user namespaces (`hostUsers: false`).
>
> **You need:** topics 04 (Linux under Kubernetes), 18, 37 (emptyDir).

---

## ⚡ Core: the 10-minute version

**The idea.** Many containers run as **root** by default, which is dangerous if they're compromised. A Pod's **securityContext** locks it down:
- run as a non-root UID (`runAsNonRoot`, `runAsUser`);
- block privilege escalation (`allowPrivilegeEscalation: false`);
- make the root filesystem read-only;
- **drop all Linux capabilities** and add back only what's needed.

`fsGroup` lets a non-root app write to its volumes. User namespaces (`hostUsers: false`) map container root to an unprivileged host user.

**Picture it.** Giving a hotel guest a room key instead of the master key, and sealing the furniture except one locker.

**The one thing to know.**

```yaml
spec:
  securityContext: { runAsNonRoot: true, runAsUser: 10001, fsGroup: 10001 }
  containers:
  - name: app
    image: busybox:1.37
    command: ["sh","-c","id; touch /tmp/x || true; sleep 3600"]
    securityContext:
      allowPrivilegeEscalation: false
      readOnlyRootFilesystem: true
      capabilities: { drop: ["ALL"] }
```

**Try it now:**
1. Run a plain busybox Pod and run `id`: root.
2. Run this one and check `id` and the logs: UID 10001, and `/tmp` is read-only.
3. Add an `emptyDir` at `/tmp` and the write works.

**You've got it if you can:** write a hardened securityContext from memory, and explain what each field does.

> 📖 **That's the core.** On a first pass, do the mini-lab and move on to the next topic. Everything below is the **full reference**: come back for the details, the edge cases, interview answers and exam-level depth.

---

## What it is

A **`securityContext`** is the part of a Pod spec that sets **privilege and access-control settings** for the Pod or for one container: which **user/group** the process runs as, whether it may **gain privileges**, whether its root filesystem is **read-only**, which **Linux capabilities** it has, and more (seccomp, AppArmor, SELinux in topic 60).

**Linux capabilities** are the pieces root's power is split into (topic 04): instead of "root can do everything", the kernel checks specific capabilities like `CAP_CHOWN` (change file owners) or `CAP_NET_ADMIN` (configure networking). Kubernetes lets you **drop** and **add** them per container.

Together these two mechanisms are the base of **Pod security**: least privilege, smaller attack surface, compliance, runtime security, especially in shared/multi-tenant clusters.

---

## Why it exists

### A default Pod runs as root

```bash
kubectl run debugpod --image=busybox -it --restart=Never -- sh
/ # id
uid=0(root) gid=0(root) groups=0(root),10(wheel)
```

- `uid=0(root)`: user ID **0** = root.
- `gid=0(root)`: primary group 0.
- `groups=0(root),10(wheel)`: also in `wheel`, the group traditionally allowed to use `sudo`/`su` to root.

**UID 0 is what makes a process root**, not the name "root": names are just labels from `/etc/passwd` and `/etc/group` (Linux notes: users and groups, topics [022](../../linux-mastery-rhel/notes/022-users-and-accounts.md) and [024](../../linux-mastery-rhel/notes/024-groups.md)).

Kubernetes **doesn't secure containers by default**. Root in a container is risky:
- **Privilege escalation / breakout**: a compromised root process has much more to work with to escape the container (kernel bugs, misconfigured mounts, devices).
- **Host access**: without user namespaces, UID 0 in the container **is** UID 0 on the host kernel; with a hostPath mount or a container escape, it's real root on the node.
- **Accidental misuse**: a root app can overwrite anything in its filesystem and mounted volumes.
- **Non-compliance**: CIS benchmarks, PCI DSS and others require least privilege.

**Not every Pod runs as root:** some images set a non-root user in their Dockerfile (`USER 1000`), and security-focused images (distroless `:nonroot`, Bitnami, Chainguard) avoid root by design. But **don't rely on the image**:
- set `runAsUser`/`runAsNonRoot` etc. **explicitly in the manifest**, whatever the image does;
- **enforce** it cluster-wide with Pod Security Standards, Gatekeeper or Kyverno (topics 58, 60).

That's defence in depth and least privilege.

---

## Analogy

A container is a **hotel room** in a building (the node). Running as root is like giving a guest the **master key**: if they're careless or a thief takes over the room, every door is open. `securityContext` gives them a **room key** (non-root UID), **seals the furniture** (read-only root filesystem) except one **locker** they can use (a writable volume with `fsGroup`), and **removes the "upgrade me" button** (no privilege escalation). Capabilities are **individual keys** cut from the master key: hand out only "the gym key" (`NET_BIND_SERVICE`), not the whole ring.

---

## How it works inside

### Pod-level fields (`spec.securityContext`)

| Field | What it does | Linux mechanism | Recommendation |
|---|---|---|---|
| `runAsUser` | UID of every container's main process (e.g. `1000`); no `/etc/passwd` entry needed | The runtime sets the process UID | A non-zero UID |
| `runAsGroup` | **Primary GID** of the processes | Process GID | A non-root GID matching your volumes/app |
| `fsGroup` | A **supplemental GID** added to all processes; supported volumes are **chowned/chmodded** to this group (with setgid on directories) so files created there belong to it | Recursive `chown :GID` + `chmod g+rwX,g+s` by the kubelet at mount time | Set it when non-root users need to write to volumes |
| `runAsNonRoot` | The kubelet **refuses to start** a container whose effective UID would be 0 | A check before start | Always `true` |
| `supplementalGroups` | Extra GIDs for all processes | Supplementary groups | When needed |
| `fsGroupChangePolicy` | `OnRootMismatch` = skip the slow recursive chown if the volume root already matches | — | `OnRootMismatch` for big volumes |
| `sysctls` | Namespaced kernel parameters for the Pod (e.g. `net.ipv4.ip_local_port_range`) | `/proc/sys` in the Pod's namespaces | Only "safe" ones are allowed by default |

> ✅ **Correct today** ([C1](#c1)): it's the other way round: if **`runAsNonRoot: true`** is set and the container would run as UID 0 (the image's `USER` is root and no `runAsUser` is given), the kubelet refuses to start it, and the Pod shows `CreateContainerConfigError: container has runAsNonRoot and image will run as root`. If the field is **omitted**, nothing is checked and the container happily runs as root. (If the image's `USER` is a **name** rather than a number, the kubelet can't verify it and also refuses: `image has non-numeric user … cannot verify user is non-root`; use a numeric `USER` or set `runAsUser`.)

> ℹ️ **Note:** `fsGroup` ownership changes only apply to volume types that support them (emptyDir, most block/CSI volumes, Secrets/ConfigMaps/projected); **not** to hostPath or many NFS setups. On huge volumes the recursive chown can make Pod start-up slow, hence `fsGroupChangePolicy: OnRootMismatch`.

### Container-level fields (`spec.containers[].securityContext`)

| Field | What it does | Linux mechanism | Recommendation |
|---|---|---|---|
| `allowPrivilegeEscalation` | Whether a process can gain **more** privileges than its parent, e.g. via a **setuid** binary | `false` sets the **`no_new_privs`** flag: setuid/setgid bits and file capabilities are ignored on `exec` | `false` |
| `privileged` | Full access to the host: all capabilities, all devices (`/dev`), no seccomp/AppArmor confinement, writable `/sys` | Like `docker run --privileged` | Never, except special node agents (some CNI/CSI/storage daemons) |
| `readOnlyRootFilesystem` | Mounts the container's root filesystem **read-only**; no writes to `/`, `/etc`, `/var`, `/tmp`… | The overlay root mounted `ro` | `true` for apps that don't need to write; give them a writable volume for what they do need |
| `capabilities` | `drop` and `add` Linux capabilities | Bounding/effective capability sets | `drop: ["ALL"]`, add only what's needed |
| `runAsUser`/`runAsGroup`/`runAsNonRoot` | Same as Pod level; **override** it for this container | — | — |
| `seccompProfile`, `appArmorProfile`, `seLinuxOptions` | Kernel confinement (topic 60) | Also settable at Pod level | `RuntimeDefault` |

**Privilege escalation:** in Linux, a process can gain more power than it started with, even as a non-root user, via **setuid binaries**: a file owned by root with the setuid bit (`-rwsr-xr-x`, like `/usr/bin/passwd` or `sudo`) runs **as root** whoever launches it. Without protection, a container that starts with a safe UID could still escalate. `allowPrivilegeEscalation: false` closes that path.

When a field is set at both levels, **the container-level value wins**.

### Scope: Pod vs container

| Field | Pod level | Container level | Notes |
|---|---|---|---|
| `runAsUser` | ✅ | ✅ | Container overrides Pod |
| `runAsGroup` | ✅ | ✅ | Container overrides Pod |
| `runAsNonRoot` | ✅ | ✅ | Container overrides Pod |
| `seccompProfile`, `appArmorProfile`, `seLinuxOptions` | ✅ | ✅ | Container overrides Pod (topic 60) |
| `fsGroup`, `fsGroupChangePolicy`, `supplementalGroups`, `sysctls` | ✅ | ❌ | Pod-level only (volume ownership, groups, kernel params apply to the whole Pod) |
| `allowPrivilegeEscalation` | ❌ | ✅ | Container only |
| `privileged` | ❌ | ✅ | Container only |
| `readOnlyRootFilesystem` | ❌ | ✅ | Container only |
| `capabilities` | ❌ | ✅ | Container only |

✅ = supported, ❌ = not valid there.

- Pod-level `securityContext` lives at `spec.securityContext` and applies to all containers (including init containers) unless overridden.
- Container-level lives at `spec.containers[].securityContext` and takes precedence.

### Linux capabilities

In Linux, root's power is really many separate **capabilities**. Kubernetes manages them **per container** through `securityContext.capabilities` (not at Pod level):
- **drop** unnecessary ones: `drop: ["ALL"]`;
- **add** only what's needed: `add: ["NET_BIND_SERVICE"]`.

This gives finer control than UID/GID alone and follows least privilege.

**The default set.** Container runtimes (containerd, CRI-O, Docker) start containers with a **reduced** set, not root's full ~40. containerd's default: `CHOWN`, `DAC_OVERRIDE`, `FOWNER`, `FSETID`, `KILL`, `SETGID`, `SETUID`, `SETPCAP`, `SETFCAP`, `NET_BIND_SERVICE`, `NET_RAW`, `SYS_CHROOT`, `MKNOD`, `AUDIT_WRITE`.

> ✅ **Correct today** ([C2](#c2)): `NET_RAW` is in the default set (it allows raw sockets: `ping`, but also packet spoofing), but **`SYS_ADMIN` is not**: you only get it by adding it or running `privileged`.

> ℹ️ **Note:** the original notes said "containers — **even when not running as root** — are granted a minimal set… `CHOWN`, `SETUID`…". The default set is the container's **bounding** set. A **root** process gets those capabilities **effective**; a **non-root** process normally has an **empty effective set** (it can't `chown` other users' files even though `CHOWN` is in the bounding set), unless the binary has file capabilities (which `allowPrivilegeEscalation: false` blocks) or capabilities are made ambient. That's why "non-root + drop ALL" is so strong. Conversely, Day 41's demo shows that **root with `drop: ["ALL"]` can't do much**: UID 0 isn't power without capabilities.

**Best practice:** start with **none** (`drop: ["ALL"]`) and add only what's absolutely required, tailored to the app.

**Common capabilities:**

| Capability | Allows |
|---|---|
| `NET_ADMIN` | Change interfaces, routes, firewall (iptables/nftables) rules (CNI plugins, VPNs, kube-proxy) |
| `SYS_ADMIN` | Very broad: mounting filesystems, many admin operations; almost "root" (avoid) |
| `SYS_TIME` | Set the system clock (time-sync daemons) |
| `CHOWN` | Change file ownership |
| `SETUID` | Change a process's user ID |
| `DAC_OVERRIDE` | Bypass file read/write/execute permission checks ("Discretionary Access Control" = normal Unix permissions) |
| `NET_BIND_SERVICE` | Bind to ports < 1024 as non-root |
| `NET_RAW` | Raw/packet sockets (ping, packet capture, spoofing) |

In Kubernetes YAML, write names **without** the `CAP_` prefix (`NET_ADMIN`, not `CAP_NET_ADMIN`).

### Mapping to Docker (Docker notes, topics [41](../../Docker/notes/41-users-and-capabilities.md) and [42](../../Docker/notes/42-userns-and-rootless.md))

| `docker run` | Kubernetes `securityContext` |
|---|---|
| `--user 1000:3000` | `runAsUser: 1000`, `runAsGroup: 3000` |
| `--group-add 2000` | `supplementalGroups: [2000]` (`fsGroup` also chowns volumes) |
| `--read-only` | `readOnlyRootFilesystem: true` |
| `--cap-drop ALL --cap-add NET_BIND_SERVICE` | `capabilities: {drop: [ALL], add: [NET_BIND_SERVICE]}` |
| `--security-opt no-new-privileges` | `allowPrivilegeEscalation: false` |
| `--privileged` | `privileged: true` |
| `--userns` | `hostUsers: false` (below) |

### User namespaces: `hostUsers: false`

With `spec.hostUsers: false`, the Pod gets its **own user namespace** (topic 04): UID 0 **inside** the container maps to an unprivileged, high UID (e.g. 1,000,000+) **on the host**. Even if the process is "root" and escapes, it's nobody on the node. **GA since v1.36** (it needs a recent kernel, a runtime that supports ID-mapped mounts, and a supporting filesystem). A great extra layer for workloads that insist on running as root.

---

## Commands / YAML

### The insecure baseline (`Day 41/00-default-pod.yaml`)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secure-pod          # despite the name: no securityContext at all
spec:
  containers:
    - name: app
      image: busybox
      command: ["sh", "-c", "sleep 3600"]
```

`id` inside → `uid=0(root)`.

### Demo 1: `secure-pod` (`Day 41/01-security-context copy.yaml`)

**Step 1: deploy with Pod- and container-level security contexts**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secure-pod
spec:
  securityContext:                  # Pod level: all containers
    runAsUser: 1000                 # process UID
    runAsGroup: 3000                # primary GID
    fsGroup: 2000                   # supplemental GID; owns supported volumes
    runAsNonRoot: true              # refuse to start as UID 0
  containers:
    - name: app
      image: busybox
      command: ["sh", "-c", "sleep 3600"]
      securityContext:              # container level
        allowPrivilegeEscalation: false   # no setuid gains (no_new_privs)
        readOnlyRootFilesystem: true      # / is read-only
```

```bash
kubectl apply -f "Day 41/01-security-context copy.yaml"
```

**Step 2: verify the identity**

```bash
kubectl exec -it secure-pod -- sh
~ $ id
uid=1000 gid=3000 groups=2000,3000
```

- `uid=1000` ← `runAsUser`.
- `gid=3000` ← `runAsGroup` (primary group).
- `groups=2000,3000`: **2000** from `fsGroup` (volume access), **3000** the primary group.
- UID 0 is root; `runAsNonRoot: true` guarantees the Pod would fail rather than run as 0.

**Step 3: read-only root filesystem**

```bash
~ $ mkdir test_dir
mkdir: can't create directory 'test_dir': Read-only file system
```

Expected: `readOnlyRootFilesystem: true` makes `/` (including `/etc`, `/var`, `/tmp`) immutable, so attackers can't drop tools or change binaries/configs at runtime.

**Step 4: privilege escalation**

```bash
~ $ sudo su
sh: sudo: not found
~ $ su -
su: must be suid to work properly
```

- busybox has no `sudo`; its `su` isn't setuid in this image.
- Even if a setuid `su` existed, `allowPrivilegeEscalation: false` (`no_new_privs`) would stop it gaining root.

Check the flag from inside: `grep NoNewPrivs /proc/1/status` → `NoNewPrivs: 1`.

**Step 5: `whoami`**

```bash
~ $ whoami
whoami: unknown uid 1000
```

Not an error: there's no line for UID 1000 in busybox's `/etc/passwd`, so the **name** can't be shown. Linux security works on **UIDs/GIDs**; usernames are only for humans.

### Demo 2: read-only root + one writable volume with `fsGroup` (`Day 41/02-security-context.yaml`)

**Goal:** no writes to the root filesystem, but the app may write to `/writable`, using `fsGroup` for group access to the mounted volume.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secure-writable
spec:
  securityContext:
    runAsUser: 1000
    runAsGroup: 3000
    fsGroup: 2000                     # the emptyDir will be group-owned by 2000
    runAsNonRoot: true
  containers:
    - name: app
      image: busybox
      command: ["sh", "-c", "sleep 3600"]
      securityContext:
        allowPrivilegeEscalation: false
        readOnlyRootFilesystem: true
      volumeMounts:
        - name: writable-vol
          mountPath: /writable
  volumes:
    - name: writable-vol
      emptyDir: {}
```

```bash
kubectl apply -f "Day 41/02-security-context.yaml"
kubectl exec -it secure-writable -- sh
~ $ id
uid=1000 gid=3000 groups=2000,3000
~ $ mkdir /tmp/test
mkdir: can't create directory '/tmp/test': Read-only file system
~ $ cd /writable && mkdir test_dir && touch myfile && ls -lthr
total 4K
drwxr-sr-x    2 1000     2000        4.0K Jun 28 09:07 test_dir
-rw-r--r--    1 1000     2000           0 Jun 28 09:07 myfile
```

**What this shows:**
- The container is non-root with an immutable root filesystem.
- It can still write to `/writable` because the volume is **group-owned by 2000** (`fsGroup`) with group write permission, and the process is in group 2000.
- The **`s`** in `drwxr-sr-x` is the **setgid** bit on the directory: new files inside inherit the directory's group (2000), keeping access consistent. (Linux notes: special permissions, topic [027](../../linux-mastery-rhel/notes/027-special-permissions-acls-and-attributes.md).)

**Takeaway:** a secure pattern for logs, temp files and caches: lock down the root filesystem, give one isolated writable volume, grant access with `fsGroup`. (Many apps also need a writable `/tmp`: mount an `emptyDir` there.)

### Demo 3: capabilities (Day 41; `Day 41/03-capability-pod.yaml` variant)

**Step 1: drop all, add back two**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: cap-pod
spec:
  containers:
    - name: app
      image: busybox               # runs as root (UID 0), so it can USE the capabilities it keeps
      command: ["sh", "-c", "sleep 3600"]
      securityContext:
        allowPrivilegeEscalation: false
        capabilities:
          drop: ["ALL"]
          add: ["NET_ADMIN", "SYS_TIME"]
```

The repo file `Day 41/03-capability-pod.yaml` is the same Pod named **`cap-pod-2`** with the `add:` lines **commented out**, i.e. **drop ALL, add nothing**: use it to see every privileged operation fail.

**Step 2: exec in**

```bash
kubectl apply -f cap-pod.yaml
kubectl exec -it cap-pod -- sh
/ # grep Cap /proc/1/status
CapInh: 0000000000000000
CapPrm: 0000000002001000
CapEff: 0000000002001000
CapBnd: 0000000002001000
CapAmb: 0000000000000000
```

`0x2001000` = bit 12 (`NET_ADMIN`) + bit 25 (`SYS_TIME`). Decode with `capsh --decode=0000000002001000` on any Linux machine.

**Step 3: added capabilities work**

```bash
/ # date -s "2025-06-01 12:00:00"       # SYS_TIME
/ # ip link set eth0 down               # NET_ADMIN
```

> ✅ **Correct today** ([C3](#c3)): the system clock is **not namespaced**: there's one clock per kernel, so `date -s` changes the **node's** clock, for every Pod on it (on kind, the shared Docker/WSL kernel; it can break TLS certificate checks and etcd leases). Don't run it on a cluster you care about; use `SYS_TIME` only for real time-sync daemons. (`ip link set eth0 down` affects only the Pod's own network namespace, but it does cut the Pod off the network.)

**Step 4: dropped capabilities fail**

```bash
/ # touch myfile && chown 1000:1000 myfile
chown: myfile: Operation not permitted
```

Even as **root**, `chown` fails: `CHOWN` was dropped.

**Key takeaway:** only the explicitly added capabilities exist; everything else is denied:

```yaml
capabilities:
  drop: ["ALL"]
  add: ["NET_ADMIN", "SYS_TIME"]
```

**Bonus (Day 41 offered it):** add `CHOWN` to the list and repeat Step 4: now it works. Try `SYS_ADMIN` and `mount -t tmpfs none /mnt`: it works, which shows why `SYS_ADMIN` is near-root.

### A production-grade baseline

```yaml
spec:
  automountServiceAccountToken: false
  securityContext:
    runAsNonRoot: true
    runAsUser: 10001
    runAsGroup: 10001
    fsGroup: 10001
    seccompProfile: { type: RuntimeDefault }      # topic 60
  containers:
  - name: app
    image: registry.example.com/app:1.4.2
    securityContext:
      allowPrivilegeEscalation: false
      readOnlyRootFilesystem: true
      capabilities: { drop: ["ALL"] }              # add NET_BIND_SERVICE only if binding <1024
    volumeMounts:
    - { name: tmp, mountPath: /tmp }
  volumes:
  - { name: tmp, emptyDir: {} }
```

This satisfies the **Restricted** Pod Security Standard (topic 60).

---

## Hands-on lab

1. Run `00-default-pod.yaml` and check `id` (root) and `grep Cap /proc/1/status`.
2. Apply Demo 1 and Demo 2; repeat every step.
3. Remove `runAsUser` from Demo 1 (keep `runAsNonRoot: true`) and watch `CreateContainerConfigError` in `kubectl describe`.
4. Run `03-capability-pod.yaml` (drop ALL, add nothing) as root: try `chown`, `ping 8.8.8.8` (needs `NET_RAW`), `ip link set eth0 down`: all fail.
5. Make nginx run as non-root: `nginx:1.27` fails (wants port 80 and root-owned dirs). Use `nginxinc/nginx-unprivileged` (port 8080), or add `NET_BIND_SERVICE` and writable `emptyDir`s for `/var/cache/nginx` and `/var/run`.
6. Try `hostUsers: false` on a busybox Pod; inside, `id` shows root, but on the node (`docker exec lab-worker ps -o user,pid,cmd -e | grep sleep`) the process has a high UID.
7. Clean up: `kubectl delete pod debugpod secure-pod secure-writable cap-pod cap-pod-2 --ignore-not-found`.

---

## Where you'll use it

- **Every production workload**: non-root, read-only root, drop ALL, no privilege escalation.
- **Fixing apps that break under Pod Security "restricted"** (writable dirs, ports, numeric users).
- **Node agents** (CNI, CSI, monitoring) that legitimately need `NET_ADMIN`, `SYS_ADMIN` or `privileged`: review them carefully.
- **CKA/CKS**: write a securityContext, add/drop capabilities, explain `fsGroup`.

---

## When it breaks

| Symptom | Cause | Fix |
|---|---|---|
| `CreateContainerConfigError: container has runAsNonRoot and image will run as root` | `runAsNonRoot: true`, image runs as UID 0, no `runAsUser` | Set `runAsUser`, or use a non-root image |
| `image has non-numeric user (nginx), cannot verify user is non-root` | Image `USER` is a name | Set a numeric `runAsUser` |
| App crashes: `Read-only file system` | `readOnlyRootFilesystem: true` and the app writes to `/tmp`, cache or PID dirs | Mount `emptyDir`s at those paths |
| `Permission denied` writing to a volume | Non-root user, volume owned by root | `fsGroup`; check the volume type supports it; for hostPath/NFS fix ownership on the server |
| Pod start very slow with a big PV | Recursive chown from `fsGroup` | `fsGroupChangePolicy: OnRootMismatch` |
| `bind: permission denied` on port 80 | Non-root, no `NET_BIND_SERVICE` | Use a port ≥ 1024, or add `NET_BIND_SERVICE` |
| `ping: permission denied (are you root?)` | `NET_RAW` dropped | Expected; use another health check, or add `NET_RAW` if really needed |
| Clock on the node suddenly wrong | Someone ran `date -s` with `SYS_TIME` | Don't grant `SYS_TIME` to apps |

---

## Quick recap

- Plain containers often run as **root (UID 0)**; set `runAsNonRoot: true` + a numeric `runAsUser`, and don't trust images to do it.
- Pod level: `runAsUser`, `runAsGroup`, `runAsNonRoot`, `fsGroup` (volume group ownership), `supplementalGroups`, `sysctls`. Container level: `allowPrivilegeEscalation` (no_new_privs), `privileged`, `readOnlyRootFilesystem`, `capabilities`. Container overrides Pod.
- Capabilities: `drop: ["ALL"]`, add back the minimum; default runtime set includes `NET_RAW` but not `SYS_ADMIN`; root without capabilities is weak; non-root normally has no effective capabilities.
- Pattern: read-only root + `emptyDir` for writable paths + `fsGroup`. Extra layer: `hostUsers: false`.

### Interview questions

1. **Q: What does `allowPrivilegeEscalation: false` actually do?**
   A: It sets the Linux `no_new_privs` flag on the container process, so setuid/setgid binaries and file capabilities can't give a process more privileges than its parent.
2. **Q: Your Pod is "running as root". What do you change?**
   A: Set `runAsNonRoot: true` and a numeric `runAsUser`/`runAsGroup`, `fsGroup` for volumes, drop all capabilities, `allowPrivilegeEscalation: false`, `readOnlyRootFilesystem: true` with emptyDirs for writable paths, and enforce it with Pod Security Admission.
3. **Q: Why does `fsGroup` make a volume writable for a non-root user?**
   A: The kubelet changes the volume's group to the fsGroup GID with group write and the setgid bit, and adds that GID to the container processes' supplementary groups, so they write as group members and new files inherit the group.

---

## Appendix: notes on the original course material

This topic was rebuilt from the original CKA course notes. This appendix keeps the audit trail out of your way while you learn.

### Sources

Day 41 (introduction; why Pod security, the `id` demo and the risks of root; important note on images that run as non-root; Pod security overview; security context: purpose and field-by-field overview, Pod-level and container-level fields; scope table and clarifications; demo 1 `secure-pod` steps 1–5; demo 2 writable volume with `fsGroup`; Linux capabilities: what they are, best practice, common capabilities table; demo: capabilities steps 1–4 and key takeaway; conclusion), files `Day 41/00-default-pod.yaml`, `Day 41/01-security-context copy.yaml`, `Day 41/02-security-context.yaml`, `Day 41/03-capability-pod.yaml`. seccomp, AppArmor and Pod Security Admission are in topic 60.

### Corrections to the original notes

Each entry shows what the original said and what's true now. In the topic above, each correction appears as a ✅ **Correct today** line or a ⚠️ **Fix to the original** box, linked to its entry here.

<a id="c1"></a>**C1 · Corrected (Day 41).** "`runAsNonRoot`: Enforces non-root execution. **If omitted** and the image defaults to UID 0 (root), the pod fails to start". **Now:** it's the other way round: if **`runAsNonRoot: true`** is set and the container would run as UID 0 (the image's `USER` is root and no `runAsUser` is given), the kubelet refuses to start it, and the Pod shows `CreateContainerConfigError: container has runAsNonRoot and image will run as root`. If the field is **omitted**, nothing is checked and the container happily runs as root. (If the image's `USER` is a **name** rather than a number, the kubelet can't verify it and also refuses: `image has non-numeric user … cannot verify user is non-root`; use a numeric `USER` or set `runAsUser`.)

<a id="c2"></a>**C2 · Corrected (Day 41).** "they still retain some sensitive capabilities (like `NET_RAW` or **`SYS_ADMIN`**) unless explicitly dropped". **Now:** `NET_RAW` is in the default set (it allows raw sockets: `ping`, but also packet spoofing), but **`SYS_ADMIN` is not**: you only get it by adding it or running `privileged`.

<a id="c3"></a>**C3 · Corrected (Day 41).** "If `CAP_SYS_TIME` is honored… the system time **inside the container** should update". **Now:** the system clock is **not namespaced**: there's one clock per kernel, so `date -s` changes the **node's** clock, for every Pod on it (on kind, the shared Docker/WSL kernel; it can break TLS certificate checks and etcd leases). Don't run it on a cluster you care about; use `SYS_TIME` only for real time-sync daemons. (`ip link set eth0 down` affects only the Pod's own network namespace, but it does cut the Pod off the network.)
