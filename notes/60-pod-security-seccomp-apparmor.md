# 60 · Pod Security Standards, Pod Security Admission, seccomp and AppArmor

> **Level:** ★★★ advanced
>
> **You will learn:** the three **Pod Security Standards** (Privileged, Baseline, Restricted) and what each forbids; **Pod Security Admission** (PSA): namespace labels, the `enforce`/`audit`/`warn` modes, version pinning, exemptions and a cluster-wide default; what replaced **PodSecurityPolicy** (removed); how to roll it out safely; **seccomp** (system-call filtering, `RuntimeDefault`, custom profiles) and **AppArmor/SELinux** (file and resource access confinement); and how these stack with securityContext.
>
> **You need:** topics 04, 58 (admission), 59 (securityContext).

---

## ⚡ Core: the 10-minute version

**The idea.** securityContext lets a Pod be safe; **Pod Security Admission** makes namespaces **require** it. You label a namespace with one of three **Pod Security Standards**:
- **privileged**: anything goes;
- **baseline**: blocks known escalations;
- **restricted**: non-root, drop all capabilities, seccomp.

For each label, choose a mode: **enforce** (reject), **warn** (tell the user) or **audit** (log). It replaced PodSecurityPolicy, which was removed in v1.25. On top of that:
- **seccomp** limits which kernel system calls a container may make;
- **AppArmor** and **SELinux** limit which files and resources it may touch.

**Picture it.** Building codes with three grades, and an inspector at each floor's door who checks the grade posted on that floor.

**The one thing to know.**

```bash
kubectl create ns secure
kubectl label ns secure pod-security.kubernetes.io/enforce=restricted pod-security.kubernetes.io/warn=restricted
kubectl run bad -n secure --image=nginx          # rejected, with a list of exactly what to fix
```

**Try it now:**
1. Run the commands above.
2. Fix the Pod using topic 59's securityContext plus `seccompProfile: { type: RuntimeDefault }` until it's accepted.
3. Dry-run a label on `default`: `kubectl label --dry-run=server --overwrite ns default pod-security.kubernetes.io/enforce=baseline`.

**You've got it if you can:** name the three levels and three modes, and say what replaced PodSecurityPolicy.

> 📖 **That's the core.** On a first pass, do the mini-lab and move on to the next topic. Everything below is the **full reference**: come back for the details, the edge cases, interview answers and exam-level depth.

---

## What it is

**Pod Security Standards (PSS)** are three **official, cumulative policy levels** defining how locked-down a Pod must be:

| Level | Meaning | For |
|---|---|---|
| **Privileged** | **No restrictions** | System/infrastructure Pods: CNI, CSI, node agents (`kube-system`) |
| **Baseline** | Blocks **known privilege escalations**; most ordinary images work unchanged | Default for typical apps |
| **Restricted** | Baseline **plus** hardening best practices (non-root, drop ALL, seccomp…) | Security-sensitive apps; the target for everything you build |

**Pod Security Admission (PSA)** is the built-in **`PodSecurity` admission plugin** (on by default, GA since v1.25; topic 58) that enforces those levels **per namespace**, based on namespace **labels**.

**seccomp** ("secure computing mode") is a Linux kernel feature that **filters system calls**: a profile says which syscalls a process may make; others fail (`EPERM`) or kill the process.

**AppArmor** (Ubuntu/Debian/SUSE) and **SELinux** (RHEL/Fedora; Linux notes, topics [072](../../linux-mastery-rhel/notes/072-selinux-fundamentals.md)–[076](../../linux-mastery-rhel/notes/076-selinux-mls-mcs.md)) are **Linux Security Modules (LSMs)**: **mandatory access control** that confines what files, capabilities and network operations a process may use, even as root.

---

## Why it exists

`securityContext` (topic 59) lets each Pod **choose** to be safe. Nothing makes it. Any developer allowed to create Pods could create one with `privileged: true` and `hostPath: /`, which is root on the node. **PSA** makes the safe choices **mandatory** per namespace, with no extra software.

> ⚠️ **History, PodSecurityPolicy:** the original mechanism, **PodSecurityPolicy (PSP)**, was deprecated in v1.21 and **removed in v1.25**: it was confusing (policies were granted through RBAC `use` and silently mutated Pods), hard to roll out, and easy to bypass. Old notes and blog posts mentioning `kind: PodSecurityPolicy` or `policy/v1beta1` are obsolete. The replacements are **PSA** (built-in, simple, three fixed levels) and **policy engines** (Kyverno, Gatekeeper, ValidatingAdmissionPolicy) for anything custom.

seccomp and AppArmor exist because even a non-root container shares the **host kernel**: the kernel has hundreds of syscalls, and a bug in a rarely used one can be a container escape. Blocking syscalls and confining file access shrinks that attack surface.

---

## Analogy

Pod Security Standards are **building codes** with three grades: "anything goes" (warehouse), "standard residential", and "hospital-grade". **PSA** is the **inspector at the door of each floor (namespace)** who reads the sign on that floor ("this floor: hospital-grade") and turns away anything that doesn't meet it. He can also just **warn** you or **note it in the log** without refusing. **seccomp** is a **list of tools** the worker may pick up (syscalls); **AppArmor** is a list of **rooms** they may enter (files, network).

---

## How it works inside

### What each level checks (summary)

**Baseline** forbids:
- `privileged: true`;
- host namespaces: `hostNetwork`, `hostPID`, `hostIPC`;
- `hostPath` volumes;
- `hostPort`s (except a known list, or none);
- adding capabilities beyond the default set (e.g. `SYS_ADMIN`, `NET_ADMIN`);
- unconfined AppArmor, custom SELinux types/users;
- `procMount: Unmasked`;
- unsafe `sysctls`;
- seccomp `Unconfined`.

**Restricted** = Baseline **plus**:
- volume types limited to: configMap, csi, downwardAPI, emptyDir, ephemeral, persistentVolumeClaim, projected, secret (no hostPath, no NFS directly…);
- `allowPrivilegeEscalation: false` (every container);
- `runAsNonRoot: true` (and `runAsUser` not 0);
- `capabilities.drop` must include **`ALL`**; only **`NET_BIND_SERVICE`** may be added;
- `seccompProfile.type` must be **`RuntimeDefault` or `Localhost`** (set at Pod or every container level).

`readOnlyRootFilesystem` is **not** required by Restricted (it's still recommended).

### PSA modes and labels

Each namespace can have up to three modes, each with a level and an optional **version**:

| Label | Effect on a violating Pod |
|---|---|
| `pod-security.kubernetes.io/enforce: <level>` | **Rejected** |
| `pod-security.kubernetes.io/audit: <level>` | Allowed; an **annotation in the audit log** |
| `pod-security.kubernetes.io/warn: <level>` | Allowed; the user sees a **warning** in kubectl output |
| `pod-security.kubernetes.io/<mode>-version: v1.37` | Pin the rules to a Kubernetes version's definition (default `latest`) |

Details:
- **enforce applies to Pods only**; `audit` and `warn` also check **workload templates** (Deployments, Jobs…), so you get warned when you `apply` a Deployment, not only when its Pods silently fail to appear.
- A Deployment in an enforcing namespace is **accepted**, but its ReplicaSet **can't create Pods**: look at `kubectl describe rs` events (`FailedCreate … violates PodSecurity`).
- **Exemptions** (by username, RuntimeClass or namespace) and **cluster-wide defaults** (for unlabelled namespaces) are set in an `AdmissionConfiguration` file passed to the API server with `--admission-control-config-file`.
- kind/kubeadm leave everything **Privileged** by default (no labels = no enforcement). Managed services vary.

### seccomp

- **`RuntimeDefault`**: the container runtime's default profile (containerd/CRI-O's, based on Docker's), which blocks ~50+ dangerous or obsolete syscalls (`keyctl`, `add_key`, `unshare` (mostly), `mount` without capability, `ptrace` restrictions, kernel module loading, `reboot`, `clock_settime` without capability…) while allowing everything ordinary apps need.
- **`Unconfined`**: no filter. Historically the **Kubernetes default**: unless you set a profile, containers ran **unconfined**, unlike `docker run`, which applies Docker's default profile.
- **`Localhost`**: a custom JSON profile stored on each **node** under the kubelet's seccomp directory (`/var/lib/kubelet/seccomp/`), referenced with `localhostProfile: profiles/my.json`.
- The kubelet option `seccompDefault: true` (GA in v1.27) makes **RuntimeDefault the default** for all Pods on that node; enable it everywhere.

```yaml
securityContext:
  seccompProfile:
    type: RuntimeDefault          # or Localhost + localhostProfile: profiles/audit.json
```

Inside the container, `grep Seccomp /proc/1/status` → `Seccomp: 2` (filter active), `0` (none).

**Building a custom profile:** run the app with an **audit/log** profile (`"defaultAction": "SCMP_ACT_LOG"`), collect the syscalls it uses from the kernel log, and turn them into an allow-list. The **Security Profiles Operator** automates recording, distribution to nodes and binding.

### AppArmor and SELinux

- **AppArmor**: profiles loaded into the **node's kernel** (`apparmor_parser`), referenced by name. Since v1.30 it's a proper field:

  ```yaml
  securityContext:
    appArmorProfile:
      type: RuntimeDefault           # or Localhost + localhostProfile: k8s-deny-write, or Unconfined
  ```

  (Old notes use the annotation `container.apparmor.security.beta.kubernetes.io/<container>: localhost/<profile>`; it's **deprecated** in favour of the field.) Check what's loaded on a node with `cat /sys/kernel/security/apparmor/profiles` or `aa-status`. The runtime's default AppArmor profile (`cri-containerd.apparmor.d`) applies automatically on AppArmor hosts.
- **SELinux**: on SELinux hosts (RHEL/Fedora/OpenShift), containers get labels like `system_u:system_r:container_t:s0:c123,c456`; MCS categories (`c123,c456`) keep Pods from reading each other's files. Set with `seLinuxOptions` (`level`, `type`). OpenShift relies heavily on this.

### How the layers stack

```
Request ──► PSA (admission): is this Pod spec allowed in this namespace?
              │
Pod runs ──► securityContext: UID/GID, no_new_privs, ro rootfs, capabilities
              │
              ├─► seccomp: which SYSCALLS the kernel accepts
              ├─► AppArmor/SELinux: which FILES/resources may be touched
              └─► namespaces + cgroups (topic 04): what it can see, how much it can use
```

PSA **checks the spec**; the kernel features **enforce at runtime**. You want both.

---

## Commands / YAML

### Label a namespace

```bash
kubectl create ns secure
kubectl label ns secure \
  pod-security.kubernetes.io/enforce=restricted \
  pod-security.kubernetes.io/enforce-version=latest \
  pod-security.kubernetes.io/warn=restricted \
  pod-security.kubernetes.io/audit=restricted
```

### A violating Pod

```bash
kubectl run bad -n secure --image=nginx:1.27
```

```
Error from server (Forbidden): pods "bad" is forbidden: violates PodSecurity "restricted:latest": allowPrivilegeEscalation != false (container "bad" must set securityContext.allowPrivilegeEscalation=false), unprivileged capabilities (container "bad" must set securityContext.capabilities.drop=["ALL"]), runAsNonRoot != true (pod or container "bad" must set securityContext.runAsNonRoot=true), seccompProfile (pod or container "bad" must set securityContext.seccompProfile.type to "RuntimeDefault" or "Localhost")
```

The message lists **every** failing check and the exact field to fix: a great checklist.

### A compliant Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: good
  namespace: secure
spec:
  securityContext:
    runAsNonRoot: true
    runAsUser: 101
    seccompProfile: { type: RuntimeDefault }
  containers:
  - name: web
    image: nginxinc/nginx-unprivileged:1.27      # listens on 8080, runs as UID 101
    ports: [{ containerPort: 8080 }]
    securityContext:
      allowPrivilegeEscalation: false
      capabilities: { drop: ["ALL"] }
```

```bash
kubectl apply -f good.yaml        # pod/good created
```

### Dry-run a level before enforcing

```bash
kubectl label --dry-run=server --overwrite ns default pod-security.kubernetes.io/enforce=baseline
```

```
Warning: existing pods in namespace "default" violate the new PodSecurity enforce level "baseline:latest"
Warning: debug-node: host namespaces, hostPath volumes, privileged
namespace/default labeled (server dry run)
```

Server-side dry-run tells you **which existing Pods** would violate the level, without changing anything.

### Cluster-wide default (kubeadm/kind; API server flag)

```yaml
# /etc/kubernetes/psa.yaml  (passed with --admission-control-config-file; mount it into the API server Pod)
apiVersion: apiserver.config.k8s.io/v1
kind: AdmissionConfiguration
plugins:
- name: PodSecurity
  configuration:
    apiVersion: pod-security.admission.config.k8s.io/v1
    kind: PodSecurityConfiguration
    defaults:                                  # for namespaces WITHOUT labels
      enforce: baseline
      enforce-version: latest
      warn: restricted
      audit: restricted
    exemptions:
      namespaces: [kube-system]
      usernames: []
      runtimeClasses: []
```

### seccomp check

```bash
kubectl exec good -n secure -- grep Seccomp /proc/1/status
```

```
Seccomp:        2
Seccomp_filters:        1
```

---

## Hands-on lab

1. Create `secure` with `enforce=restricted`; try `nginx:1.27` (rejected) and the compliant Pod (accepted).
2. Create a Deployment of plain `nginx:1.27` in `secure`: the `apply` shows a **warning**, the Deployment exists, but `kubectl get pods` is empty; find the reason in `kubectl describe rs`.
3. Label `default` with `warn=restricted` only: create Pods and read the warnings; nothing is blocked.
4. Server-side dry-run `enforce=baseline` on `kube-system` and read which system Pods would break (that's why system namespaces stay privileged).
5. seccomp: run busybox with `Unconfined` and with `RuntimeDefault`; compare `grep Seccomp /proc/1/status`; try `unshare -r sh` in both (blocked under RuntimeDefault without capabilities).
6. (Optional, AppArmor host) load a deny-write profile on a node (`apparmor_parser`), reference it with `appArmorProfile: Localhost`, and watch writes fail. (kind nodes share the host kernel: on WSL2, AppArmor is usually not available; use a Ubuntu VM or minikube with a VM driver.)
7. Clean up: `kubectl delete ns secure`; remove labels with `kubectl label ns default pod-security.kubernetes.io/warn-`.

---

## Where you'll use it

- **Every multi-tenant cluster**: label app namespaces `baseline` (enforce) + `restricted` (warn/audit), then move to restricted enforce.
- **Migrating off PSP** in old clusters before upgrading past 1.24.
- **Kubelet configuration**: `seccompDefault: true`.
- **CKS** (Certified Kubernetes Security Specialist): PSA, seccomp and AppArmor are core topics; CKA expects you to recognise them.

---

## When it breaks

| Symptom | Cause | Fix |
|---|---|---|
| `violates PodSecurity "restricted:latest": …` | Pod spec doesn't meet the namespace level | Fix the fields listed in the message (topic 59) |
| Deployment shows `0/3` ready, no Pods at all | Enforce blocked the ReplicaSet's Pods | `kubectl describe rs`; fix the template |
| System add-on breaks after labelling | Add-on needs host access (CNI, CSI, log agents) | Keep its namespace `privileged`; exempt it |
| Upgrade changed what's allowed | `enforce-version: latest` picked up new checks | Pin `enforce-version` to a tested version |
| App fails with `Operation not permitted` on a syscall | seccomp blocks it | Find it (`strace`, audit log), use a custom Localhost profile |
| Pod `Blocked` / `CreateContainerError: … apparmor profile not found` | The profile isn't loaded on that node | Load it on every node (DaemonSet / Security Profiles Operator) |
| Old manifests with `kind: PodSecurityPolicy` fail | PSP removed in 1.25 | Delete them; use PSA/policy engines |

---

## Quick recap

- **PSS**: Privileged (anything), Baseline (no known escalations), Restricted (non-root, drop ALL, no escalation, seccomp RuntimeDefault, safe volumes).
- **PSA** (`PodSecurity` admission, on by default) enforces them per **namespace label**: `enforce` (reject), `audit` (log), `warn` (tell the user); pin versions; cluster defaults and exemptions via `AdmissionConfiguration`.
- **PodSecurityPolicy was removed in v1.25.**
- **seccomp** filters syscalls (`RuntimeDefault`; make it the node default with `seccompDefault`); **AppArmor/SELinux** confine file and resource access. They enforce at runtime what PSA checks at admission.

### Interview questions

1. **Q: What replaced PodSecurityPolicy, and how do you roll it out safely?**
   A: Pod Security Admission with the Pod Security Standards (plus policy engines for custom rules). Roll out with `warn` and `audit` labels first, a server-side dry-run of the `enforce` label to find violators, fix the workloads, then enforce, keeping system namespaces privileged or exempt.
2. **Q: What does the Restricted level require?**
   A: Everything in Baseline (no privileged, host namespaces, hostPath, extra capabilities…) plus `runAsNonRoot`, `allowPrivilegeEscalation: false`, dropping ALL capabilities (only NET_BIND_SERVICE may be added), a RuntimeDefault or Localhost seccomp profile, and a limited set of volume types.
3. **Q: What's the difference between seccomp and AppArmor?**
   A: seccomp filters which system calls a process can make; AppArmor (an LSM) restricts which files, capabilities and network operations a process can use, by path-based profiles. Both reduce what a compromised container can do with the shared kernel.

---

## Appendix: notes on the original course material

This topic was rebuilt from the original CKA course notes. This appendix keeps the audit trail out of your way while you learn.

### Sources

none: a **new topic**. Day 38 lists the `PodSecurity` admission plugin (topic 58) and Day 41 mentions "Pod Security Standards" as an enforcement tool (topic 59); this topic covers them fully. It also covers the **PodSecurityPolicy** removal referenced in the correction list.
