# 21 · Debugging Pods: Pending, ImagePullBackOff, CrashLoopBackOff, OOMKilled and Friends

> **Level:** ★★ intermediate
>
> **You will learn:** a repeatable method for any broken Pod; what every common STATUS means and its usual causes (`Pending`, `ContainerCreating`, `ErrImagePull`/`ImagePullBackOff`, `CrashLoopBackOff`, `OOMKilled`, `CreateContainerConfigError`, `RunContainerError`, `Evicted`, `Terminating`, not `READY`); the tools: `describe`, events, `logs --previous`, `exec`, `port-forward`, `kubectl debug` (ephemeral containers and node debugging), throwaway test Pods and `crictl`; and how to prove each fix.
>
> **You need:** topics 18–20.

---

## ⚡ Core: the 10-minute version

**The idea.** Debug every broken Pod in the same order:
1. `kubectl get pod -o wide`: what's the STATUS?
2. `kubectl describe pod`: the **Events** at the bottom, plus the last state and exit code.
3. `kubectl logs --previous`: what the crashed container printed.
4. Check the config it depends on (image, ConfigMaps, Secrets, probes, resources).
5. Make the smallest fix and verify.

Most problems have a typical cause:

| STATUS | Usually means |
|---|---|
| `Pending` | No node fits |
| `ImagePullBackOff` | Wrong image name or no access |
| `CrashLoopBackOff` | The app keeps exiting |
| `OOMKilled` | Memory limit too low |
| `CreateContainerConfigError` | A ConfigMap or Secret is missing |

**Picture it.** A doctor's checklist: symptoms, vital signs, history, then treatment. Never surgery first.

**The one thing to know.**

```bash
kubectl describe pod <p> | sed -n '/Events:/,$p'
kubectl logs <p> --previous
kubectl debug -it <p> --image=busybox:1.37 --target=<container>   # for images without a shell
```

**Try it now:** create one broken Pod of each kind and diagnose it with the commands above:
1. a bad image tag;
2. `command: ["false"]`;
3. a 10Mi memory limit on a memory-hungry app;
4. a reference to a missing ConfigMap.

**You've got it if you can:** name the first command you'd run for each status above.

> 📖 **That's the core.** On a first pass, do the mini-lab and move on to the next topic. Everything below is the **full reference**: come back for the details, the edge cases, interview answers and exam-level depth.

---

## What it is

A **method** plus a **lookup table**. Almost every Pod problem shows up in three places:
1. the **STATUS** column of `kubectl get pods`,
2. the **Events** and container **State / Last State** in `kubectl describe pod`,
3. the app's **logs** (current and **previous** container).

---

## Why it exists

"My Pod isn't working" is the most common ticket in any Kubernetes team and a guaranteed CKA topic. Guessing wastes hours. A fixed order of checks turns most cases into a two-minute job.

---

## Analogy

A **doctor's triage**: first the vital signs (STATUS), then the patient's history (Events, Last State, exit code), then tests (logs, exec, debug containers). You don't order an MRI before taking the temperature.

---

## How it works inside

### The method (Day 22 workflow + Day 58 approach)

```
1. kubectl get pod <p> -o wide              → STATUS, READY, RESTARTS, NODE
2. kubectl describe pod <p>                 → Events (bottom!), State/Last State, Exit Code, Conditions
3. kubectl logs <p> [-c c] [--previous]     → what the app said before dying
4. Validate config                          → image, command/args (ENTRYPOINT vs CMD), env, mounts, resources, probes
5. Reproduce small                          → a minimal test Pod with the same image/env/command
6. Smallest fix → verify immediately        → kubectl get pods -w
```

From Day 58's approach:
- **Read Events first**: `describe` usually spells out the failure.
- Make the **smallest** change that turns the signal green; **verify immediately**.
- Use a throwaway **busybox** Pod for `nslookup`/`wget` checks, then clean up.
- If the problem isn't the Pod: check **node health** (kubelet, runtime, pressure) → **CNI and Pod IPs** → **Services/endpoints** → **DNS** → **storage** → then app-level issues (probes, limits). Topic 83 covers the cluster-level scenarios.

### Where in the lifecycle each status comes from

| STATUS | Stage (topic 10) | Meaning |
|---|---|---|
| `Pending` (no node) | Scheduler | No node fits |
| `Pending` + node, or `ContainerCreating` | kubelet / CNI / volumes | Sandbox, network or volume not ready yet |
| `ErrImagePull` / `ImagePullBackOff` | Runtime pulling | Can't get the image |
| `CreateContainerConfigError` | kubelet building the container | Referenced ConfigMap/Secret/key missing |
| `RunContainerError` / `StartError` | Runtime starting the process | Command not found, bad mount, permission |
| `CrashLoopBackOff` | App | Started, then exited, repeatedly |
| `OOMKilled` | Kernel cgroup | Exceeded memory limit |
| `Running`, `0/1 READY` | Readiness probe | Running but not accepting traffic |
| `Evicted` | kubelet node pressure | Node ran out of memory/disk (topic 47) |
| `Terminating` (stuck) | Deletion | Finalizers, or unreachable node |
| `Unknown` | Node | Node not reporting |

### Pod status deep dive

| Status | Description | Common causes | How to debug |
|---|---|---|---|
| **CrashLoopBackOff** | Container repeatedly crashes and is restarted with exponential back-off | App bugs; missing dependencies or config; wrong command/args; resource exhaustion (**OOMKilled**); **liveness probe** failures | `kubectl logs <p> --previous`; check env, ConfigMaps, Secrets; check exit code; verify liveness probe |
| **ImagePullBackOff** | Pulls failed; kubelet is retrying with back-off | Invalid image or tag; private registry auth; network/DNS to registry; rate limits | `kubectl describe pod`; verify name/tag; try `crictl pull` on the node |
| **ErrImagePull** | The most recent pull failed (becomes ImagePullBackOff while waiting to retry) | Same as above | Events in `describe` |
| **OOMKilled** | Container exceeded its **memory limit**; kernel killed it (exit 137) | Memory leak, high usage, limit too low | `describe` (Last State: OOMKilled); `kubectl top pod`; adjust requests/limits (topic 41) |
| **Unknown** | Pod state can't be determined | Node failure or network issue | `kubectl get nodes`; node logs (topic 83) |
| **CreateContainerConfigError** | Container can't be configured | ConfigMap/Secret or a key in it doesn't exist; invalid env reference | `describe` message names the missing object |
| **Pending** | Not scheduled or not started | Insufficient CPU/memory, taints, affinity, unbound PVC | `describe` → `FailedScheduling` |

> ✅ **Correct today** ([C1](#c1)): Kubernetes nodes don't run Docker (topic 08). Test on the node with **`crictl pull <image>`**, which uses the same runtime and network path as the kubelet. (`docker pull` on your laptop only proves the image exists, not that the node can reach it.)

### Scenario: ImagePullBackOff / ErrImagePull (Day 58 #5)

**Problem:** the Pod won't start because the image can't be pulled.

| Likely cause | Fix |
|---|---|
| **Wrong image name/tag** | Point `image:` at a valid `repo:tag`; redeploy |
| **Private registry needs auth** | Create an `imagePullSecret` and reference it in the Pod or its ServiceAccount (topic 62) |
| **Stale local image / policy** | Use a correct, new tag; delete the Pod (or `kubectl rollout restart`) so it pulls again |
| Registry unreachable / DNS / proxy | Check from the node: `crictl pull`, DNS, firewall |
| Rate limited (`toomanyrequests`) | Authenticate; mirror/cache |

```bash
kubectl describe pod <p> | sed -n '/Events:/,$p'           # the exact pull error
crictl pull <image:tag>                                     # on the node: try the pull yourself
kubectl get sa <sa> -o yaml | grep -A2 imagePullSecrets     # is a pull secret attached?
```

Fix for a private registry (topic 62 explains each flag):

```bash
kubectl create secret docker-registry regcred \
  --docker-server=<REG> --docker-username=<U> --docker-password=<P> --docker-email=<E>
# reference it in the Pod spec (imagePullSecrets) or attach it to the ServiceAccount, then:
kubectl delete pod <p>       # the controller recreates it and pulls again
kubectl get pods -w
```

**Example:** Events show `manifest unknown` → fix the **tag** → the Pod pulls and becomes **Running**.

Reading pull errors:

| Event text contains | Meaning |
|---|---|
| `manifest unknown`, `not found` | Tag (or repo) doesn't exist |
| `pull access denied`, `insufficient_scope`, `401 Unauthorized` | Private repo, missing/wrong credentials |
| `no such host`, `i/o timeout` | Node can't resolve/reach the registry |
| `toomanyrequests` | Registry rate limit |
| `x509: certificate signed by unknown authority` | Private registry with a CA the node doesn't trust |

### Scenario: CrashLoopBackOff / OOMKilled / probe failures (Day 58 #6)

**Problem:** the container starts, then exits again and again; readiness never becomes Ready.

| Likely cause | Fix |
|---|---|
| **Bad command/args, missing config or volume** | Correct `command`/`args`, env, mounts; re-apply |
| **Probes too aggressive** (app needs warm-up). A **probe** is a periodic health check the kubelet runs; a failing **liveness** probe makes the kubelet **restart** the container (topic 22). | Relax `initialDelaySeconds`, `periodSeconds`, `timeoutSeconds`, or add a **startup probe** |
| **OOMKilled** (exit 137) / limits too low | Raise memory requests/limits, or reduce the app's usage |
| Dependency down (DB not reachable) and the app exits | Make the app retry; init container to wait (topic 23) |
| Short-lived command under `restartPolicy: Always` | Use a Job, or a long-running command |

```bash
kubectl logs <p> --previous                                       # why did the LAST run die?
kubectl describe pod <p> | sed -n '/State:/,/Events:/p'           # OOMKilled? Exit Code?
kubectl get deploy <d> -o yaml | sed -n '/livenessProbe:/,/readinessProbe:/p'   # probe settings
```

```bash
# Example fix: more memory, then roll the Pods
kubectl set resources deploy/<d> --requests=memory=256Mi --limits=memory=512Mi
kubectl rollout restart deploy/<d>      # not strictly needed: set resources already triggers a rollout
kubectl get pods -w
```

**Example:** `describe` shows **OOMKilled** → increase the memory limit → the Pod stabilises.

**Reading the exit code** (topic 19): `1` app error (read logs); `127` command not found (wrong `command`); `126` not executable / permission; `137` SIGKILL: OOM (`Reason: OOMKilled`) or liveness-probe kill after the grace period; `143` SIGTERM (stopped normally).

### Tools beyond logs and describe

**Throwaway test Pod:**

```bash
kubectl run test-pod --rm -it --restart=Never --labels=app=test-pod --image=busybox:1.37 -- sh
# inside: wget -qO- http://backend-svc:9090 ; nslookup backend-svc ; exit
```

**`kubectl debug`: ephemeral containers.** Distroless or minimal images have no shell. An **ephemeral container** is a temporary container **added to a running Pod** (it shares the Pod's network namespace, and with `--target` the process namespace of one container). It never restarts and has no ports or probes.

```bash
kubectl debug -it <pod> --image=nicolaka/netshoot --target=<container>
```

```
Defaulting debug container name to debugger-7x2kq.
If you don't see a command prompt, try pressing enter.
debugger-7x2kq:~# ps aux        # sees the target container's processes
debugger-7x2kq:~# ss -tlnp      # its listening sockets (shared network)
```

**Debug a copy of the Pod** (for crash-looping Pods: change the command so it stays up):

```bash
kubectl debug <pod> -it --copy-to=<pod>-dbg --container=<c> -- sh
kubectl debug <pod> -it --copy-to=<pod>-dbg --set-image=*=busybox:1.37    # swap images
```

**Debug a node** (a privileged Pod with the host filesystem at `/host`):

```bash
kubectl debug node/lab-worker -it --image=busybox:1.37
# chroot /host ; then crictl ps, journalctl -u kubelet, etc. (if available on the node)
```

**Port-forward** to test the app directly, bypassing Services and Ingress:

```bash
kubectl port-forward pod/<p> 8080:80     # then: curl localhost:8080
```

**Events across the namespace**, newest last:

```bash
kubectl get events --sort-by=.lastTimestamp
kubectl events --for pod/<p> --watch
```

**Keep a crashing container alive** to inspect it: temporarily override its command (topic 03):

```yaml
command: ["sleep", "3600"]
```

---

## Commands / YAML

### Example session: CrashLoopBackOff

```bash
kubectl get pods
```

```
NAME                   READY   STATUS             RESTARTS      AGE
api-5f7d8c9b4-q2w9x    0/1     CrashLoopBackOff   4 (32s ago)   2m
```

- `RESTARTS 4 (32s ago)` = 4 restarts, the last one 32 s ago; gaps grow up to 5 min.

```bash
kubectl describe pod api-5f7d8c9b4-q2w9x | sed -n '/State:/,/Restart Count/p'
```

```
    State:          Waiting
      Reason:       CrashLoopBackOff
    Last State:     Terminated
      Reason:       Error
      Exit Code:    1
      Started:      Tue, 17 Nov 2026 12:10:01 +0000
      Finished:     Tue, 17 Nov 2026 12:10:02 +0000
    Ready:          False
    Restart Count:  4
```

- Ran for **1 second** and exited **1**: the app itself is failing at start-up. Read the logs.

```bash
kubectl logs api-5f7d8c9b4-q2w9x --previous
```

```
Error: environment variable DATABASE_URL is not set
```

→ Fix the env (topic 34), apply, watch it become `1/1 Running`.

### Example: CreateContainerConfigError

```bash
kubectl describe pod web-7c9d… | grep -A3 Events: ; kubectl describe pod web-7c9d… | tail -2
```

```
  Warning  Failed     5s (x3 over 20s)  kubelet  Error: configmap "web-config" not found
```

→ Create the ConfigMap (or fix its name). The kubelet retries automatically.

---

## Hands-on lab: break it, then fix it

Create each, diagnose it **only** with `get`, `describe`, `logs`, then fix:

```bash
kubectl run bad-tag   --image=nginx:9.99                                   # ErrImagePull → ImagePullBackOff
kubectl run crash     --image=busybox:1.37 -- sh -c 'echo boom; exit 3'    # CrashLoopBackOff, exit 3
kubectl run notfound  --image=busybox:1.37 --command -- nosuchcmd          # RunContainerError / StartError (exit 127-ish)
kubectl run oom --image=polinux/stress --overrides='{"spec":{"containers":[{"name":"oom","image":"polinux/stress","args":["--vm","1","--vm-bytes","200M","--vm-hang","1"],"resources":{"limits":{"memory":"100Mi"}}}]}}'   # OOMKilled
kubectl run cfgerr --image=nginx --overrides='{"spec":{"containers":[{"name":"cfgerr","image":"nginx","envFrom":[{"configMapRef":{"name":"missing-cm"}}]}]}}'   # CreateContainerConfigError
kubectl run huge --image=nginx --overrides='{"spec":{"containers":[{"name":"huge","image":"nginx","resources":{"requests":{"cpu":"64"}}}]}}'   # Pending, Insufficient cpu
```

For each, write down: STATUS → event or Last State line → root cause → fix. Then:
- `kubectl debug -it crash --image=busybox:1.37 --copy-to=crash-dbg -- sh` and look around.
- `kubectl debug node/lab-worker -it --image=busybox:1.37` and `ls /host/var/log/pods`.
- Clean up: `kubectl delete pod bad-tag crash notfound oom cfgerr huge crash-dbg; kubectl get pods | grep node-debugger | awk '{print $1}' | xargs -r kubectl delete pod`.

---

## Where you'll use it

- **On-call**: the first five minutes of most incidents.
- **CKA**: "fix the broken Pod/Deployment" tasks; time is short, so follow the method.
- **Reviews**: spotting future crashes (no limits, aggressive liveness probes, missing ConfigMaps) before they ship.

---

## When it breaks (debugging the debugging)

| Problem | Cause | Fix |
|---|---|---|
| `kubectl logs` shows nothing useful | App logs to files, not stdout | Log to stdout/stderr; or `exec` and read the file |
| `logs --previous` says "previous terminated container not found" | No restart yet, or the Pod was recreated (new name) | Check the new Pod; look at the ReplicaSet's events |
| `exec` fails: "executable file not found" | No shell in the image | `kubectl debug` with a tool image |
| Events list is empty | Events expire (~1 h) | Reproduce, or check your event exporter / logging |
| `error: ephemeral containers are disabled` | Very old cluster or policy | Use `--copy-to`, or debug on the node with `crictl` |

---

## Quick recap

- **Order:** `get -o wide` → `describe` (Events, State/Last State, exit code) → `logs --previous` → validate config → reproduce small → smallest fix → verify.
- Pull errors: read the event text (`manifest unknown`, `denied`, `no such host`, `toomanyrequests`); test with `crictl pull` on the node.
- CrashLoopBackOff: exit code + previous logs; OOMKilled = memory limit; check liveness probes.
- `kubectl debug` (ephemeral container, `--copy-to`, `node/`) for images without shells and for node-level checks.

### Interview questions

1. **Q: A Pod is in CrashLoopBackOff. What do you do?**
   A: `describe` to get the exit code and last state (OOMKilled? exit 1? 127?), `logs --previous` for the app's error, then check command/args, env, ConfigMaps/Secrets, resources and liveness probes; fix and watch it stabilise.
2. **Q: ErrImagePull vs ImagePullBackOff?**
   A: ErrImagePull is the latest pull failure; ImagePullBackOff means the kubelet is waiting (with exponential back-off) before trying again. Same causes: name/tag, auth, network, rate limits.
3. **Q: How do you debug a distroless container with no shell?**
   A: `kubectl debug -it <pod> --image=<tools-image> --target=<container>` adds an ephemeral container sharing its network and process namespaces; or debug a copy with `--copy-to`.

---

## Appendix: notes on the original course material

This topic was rebuilt from the original CKA course notes. This appendix keeps the audit trail out of your way while you learn.

### Sources

Day 22 (Pod status deep dive table; debugging common Pod errors, step-by-step workflow), Day 58 (scenario 5: ImagePullBackOff/ErrImagePull; scenario 6: CrashLoopBackOff/OOMKilled/probe failures; "how to use the scenarios" approach), Day 13 (temporary troubleshooting Pod).

### Corrections to the original notes

Each entry shows what the original said and what's true now. In the topic above, each correction appears as a ✅ **Correct today** line or a ⚠️ **Fix to the original** box, linked to its entry here.

<a id="c1"></a>**C1 · Corrected (Day 22).** the table suggested testing an image pull "manually (`docker pull`)". **Now:** Kubernetes nodes don't run Docker (topic 08). Test on the node with **`crictl pull <image>`**, which uses the same runtime and network path as the kubelet. (`docker pull` on your laptop only proves the image exists, not that the node can reach it.)
