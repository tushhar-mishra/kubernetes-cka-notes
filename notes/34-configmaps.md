# 34 · ConfigMaps: Configuration Outside the Image

> **Level:** ★ foundation
>
> **You will learn:** why configuration must live outside container images; what a ConfigMap is; the ways a Pod consumes one (single env vars, all keys with `envFrom`, command-line arguments, mounted files); how mounting really works (a directory of files; `items`; `subPath`; why a mount hides existing files); what happens when a ConfigMap changes (env vars never update, volume files do, `subPath` files don't); immutable ConfigMaps; and the nginx demo that serves an `index.html` from a ConfigMap.
>
> **You need:** topics 13, 15, 25 and 31. One preview: a **volume** is a directory that Kubernetes makes available inside a container at a `mountPath` (full story in topics 36–38).

---

## ⚡ Core: the 10-minute version

**The idea.** Keep configuration **out of your image**, so the same image runs in dev and prod. A **ConfigMap** stores non-secret settings as key/value pairs. A Pod can read them:
- as **environment variables**, which are fixed at container start;
- as **files** in a mounted directory, which update automatically about a minute after the ConfigMap changes (your app must re-read them).

**Picture it.** The image is the appliance; the ConfigMap is its settings card, swapped without rebuilding the appliance.

**The one thing to know.**

```bash
kubectl create configmap app-config --from-literal=MODE=prod --from-literal=COLOR=blue
```

```yaml
containers:
- name: app
  image: busybox:1.37
  command: ["sh","-c","env | grep -E 'MODE|COLOR'; cat /config/MODE; sleep 3600"]
  envFrom: [{ configMapRef: { name: app-config } }]      # all keys as env vars
  volumeMounts: [{ name: cfg, mountPath: /config }]      # all keys as files
volumes: [{ name: cfg, configMap: { name: app-config } }]
```

**Try it now:**
1. Create the ConfigMap and the Pod, then read the logs.
2. Edit the ConfigMap (`kubectl edit cm app-config`).
3. After about a minute, `kubectl exec` and `cat /config/MODE`: the file changed, the env var didn't.

**You've got it if you can:** choose between env vars and files, and explain why env vars don't update.

> 📖 **That's the core.** On a first pass, do the mini-lab and move on to the next topic. Everything below is the **full reference**: come back for the details, the edge cases, interview answers and exam-level depth.

---

## What it is

A **ConfigMap** is a Kubernetes object that stores **non-confidential configuration** as **key-value pairs**. Values can be short strings (`ENVIRONMENT: production`) or whole files (an `nginx.conf`, an `index.html`).

Pods consume ConfigMaps in three main ways:
1. as **environment variables**,
2. as **command-line arguments** (less common; built from env vars),
3. as **configuration files** in a **mounted volume**.

> **Important:** ConfigMaps store plain text, with no encoding or protection. For anything sensitive, use a **Secret** (topic 35).

**What are environment variables?** Key-value pairs that the operating system and programs use to configure behaviour without code changes. On Linux:

```bash
export APP_NAME=frontend
export ENVIRONMENT=production
echo $APP_NAME      # frontend
echo $ENVIRONMENT   # production
```

Shell scripts and applications read them at start-up to configure themselves. (Your Linux notes, topic [012](../../linux-mastery-rhel/notes/012-variables-and-environment.md), cover them in depth.)

---

## Why it exists

**Why ConfigMaps:**
- Many hard-coded environment variables in Pod specs become **unmanageable**.
- You want to **change configuration without rebuilding the image**.
- Config needs to reach the app as **env vars, CLI arguments or files**.

ConfigMaps **decouple configuration from images**, so the **same image** runs in dev, staging and prod with different ConfigMaps. That's the "config in the environment" rule of twelve-factor apps.

**Without a ConfigMap (Day 28, `Day 28/ev.yaml`):** values hard-coded in the Deployment:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend-deploy
spec:
  replicas: 3
  selector:
    matchLabels:
      app: frontend
  template:
    metadata:
      labels:
        app: frontend
    spec:
      containers:
        - name: frontend-container
          image: nginx
          env:
            - name: APP
              value: frontend
            - name: ENVIRONMENT
              value: production
```

It works, but it doesn't scale and can't be reused across environments.

---

## Analogy

The image is a **coffee machine**; the ConfigMap is the **settings card** taped to it ("strength 3, 200 ml"). The same machine model works in every office; each office just gets its own card. Some settings are read **once at power-on** (env vars); others are a **recipe book on the shelf** the machine can re-read (mounted files).

---

## How it works inside

- A ConfigMap is an ordinary namespaced object stored in **etcd**. Its keys live in `data` (UTF-8 text) or `binaryData` (base64 for binary files). The total size limit is **1 MiB** (an etcd limit).
- A Pod can only use ConfigMaps **in its own namespace**.
- **Env vars** are resolved by the kubelet **when the container starts** and passed to the process. They're never updated afterwards.
- **Volume** consumption: the kubelet writes each key as a **file** into a directory on the node (`/var/lib/kubelet/pods/<uid>/volumes/kubernetes.io~configmap/<name>/`) and bind-mounts that directory into the container. The files are actually **symlinks** into a hidden timestamped folder (`..data → ..2026_11_17_10_41_03.123456789/`). On update, the kubelet writes a new folder and **atomically swaps** the `..data` symlink, so apps never see a half-written file.
- The kubelet re-syncs mounted ConfigMaps periodically (its sync loop plus a cache TTL), so updates reach files within **about a minute**, not instantly.
- If a referenced ConfigMap (or key) **doesn't exist**, the container can't start: **`CreateContainerConfigError`** (topic 21), unless you mark the reference `optional: true`.

---

## Commands / YAML

### Demo 1: ConfigMap as environment variables

This builds on the `frontend-deploy` from topic 30/31.

**Step 1: create the ConfigMap**

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: frontend-cm            # how Pods refer to it
data:
  APP: "frontend"              # application name
  ENVIRONMENT: "production"    # deployment environment
```

```bash
kubectl apply -f frontend-cm.yaml
```

> **Order:** create the **ConfigMap before the Deployment**, so the Pods can read it when they start. (If you don't, the Pods sit in `CreateContainerConfigError` until it appears.)

(`Day 28/cm-1.yaml` is the same ConfigMap with `APP: "backend"`, which is handy for seeing a changed value.)

**Step 2: reference it from the Deployment** (`Day 28/frontend-cm-1.yaml`, which also contains the NodePort Service):

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend-deploy
spec:
  replicas: 3
  selector:
    matchLabels:
      app: frontend
  template:
    metadata:
      labels:
        app: frontend
    spec:
      containers:
      - name: frontend-container
        image: nginx:1.29
        env:
        - name: APP                  # env var name INSIDE the container
          valueFrom:
            configMapKeyRef:
              name: frontend-cm      # which ConfigMap
              key: APP               # which key in it
        - name: ENVIRONMENT
          valueFrom:
            configMapKeyRef:
              name: frontend-cm
              key: ENVIRONMENT
```

- The env var **name** doesn't have to equal the **key**: `name: SERVICE` with `key: APP` gives a variable `SERVICE` holding APP's value.

```bash
kubectl apply -f frontend-deploy.yaml
```

**Verify:**

```bash
kubectl get pods -l app=frontend
kubectl exec -it <pod-name> -- printenv | grep -E 'APP|ENVIRONMENT'
```

```
APP=frontend
ENVIRONMENT=production
```

**Load every key at once** with `envFrom` (one env var per key):

```yaml
        envFrom:
        - configMapRef:
            name: frontend-cm
          prefix: CFG_               # optional: CFG_APP, CFG_ENVIRONMENT
```

**As command-line arguments:** first make an env var, then reference it with Kubernetes' `$(VAR)` syntax (expanded by the kubelet, no shell needed):

```yaml
        args: ["--env=$(ENVIRONMENT)"]
```

**Imperative creation:**

```bash
kubectl create configmap <name> --from-literal=key1=value1 --from-literal=key2=value2
kubectl create configmap frontend-cm --from-literal=APP=frontend --from-literal=ENVIRONMENT=production
kubectl create configmap nginx-conf --from-file=nginx.conf            # key = file name
kubectl create configmap web --from-file=index.html=./site/home.html   # custom key
kubectl create configmap app-env --from-env-file=app.env               # KEY=VALUE lines
```

**Imperative vs declarative:**
- **Imperative** (`kubectl create configmap`): quick tests, demos, interactive experiments, when you don't need Git tracking.
- **Declarative** (YAML + `kubectl apply`): production; version control; collaboration, auditing, repeatable deployments.
- **Rule of thumb:** imperative for temporary tasks; declarative for anything repeatable or audited. (Best of both: `kubectl create configmap … --dry-run=client -o yaml > cm.yaml`.)

### Demo 2: ConfigMap as a file

**Step 1: add an HTML file to the ConfigMap** (`Day 28/cm-2.yaml` is a longer variant):

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: frontend-cm
data:
  APP: "frontend"
  ENVIRONMENT: "production"
  index.html: |                      # when mounted, this key becomes a file named index.html
    <!DOCTYPE html>
    <html>
    <head><title>Welcome</title></head>
    <body>
      <h1>This page is served by nginx. Welcome to Cloud With VarJosh!!</h1>
    </body>
    </html>
```

- `|` = a YAML **block scalar** in **literal style**: "keep the following lines exactly as written, line breaks and indentation included" (topic 13). Perfect for HTML, scripts and config files in YAML.

**Step 2: mount it in the Deployment**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend-deploy
spec:
  replicas: 3
  selector:
    matchLabels:
      app: frontend
  template:
    metadata:
      labels:
        app: frontend
    spec:
      containers:
      - name: frontend-container
        image: nginx:1.29
        env:
        - name: APP
          valueFrom: { configMapKeyRef: { name: frontend-cm, key: APP } }
        - name: ENVIRONMENT
          valueFrom: { configMapKeyRef: { name: frontend-cm, key: ENVIRONMENT } }
        volumeMounts:
        - name: html-volume
          mountPath: /usr/share/nginx/html     # nginx's web root (a DIRECTORY)
          readOnly: true
      volumes:
      - name: html-volume
        configMap:
          name: frontend-cm
          items:                    # only these keys become files
          - key: index.html         # the ConfigMap key…
            path: index.html        # …written as this file name
```

**Directories, files and `items`:**
- A ConfigMap volume is mounted as a **directory**. Mounting it at a path that's an existing **file** (e.g. `/usr/share/nginx/html/index.html`) fails, because Kubernetes **can't mount a directory over a file**. That's why the `mountPath` is the directory `/usr/share/nginx/html`.
- **Without `items`**, **every key** becomes a file: `/usr/share/nginx/html/APP`, `/usr/share/nginx/html/ENVIRONMENT`, `/usr/share/nginx/html/index.html`.
- **With `items`**, only the listed keys appear (here just `index.html`).
- **A mount hides what was there.** Mounting over `/usr/share/nginx/html` hides **everything** the image had in that directory (nginx's `50x.html` disappears). That's often fine for a web root, but fatal for `/etc` or a directory full of other config.

### Mounting one key as a single file: `subPath`

```yaml
        volumeMounts:
        - name: html-volume
          mountPath: /usr/share/nginx/html/index.html   # the exact FILE path in the container
          subPath: index.html                           # take only this key from the volume
```

"Mount only `index.html` from the ConfigMap at `/usr/share/nginx/html/index.html`." This works even though the image already has that file, because it replaces a **file with a file**. `Day 28/frontend-cm-2.yaml` uses exactly this.

| `subPath` advantages | `subPath` disadvantages |
|---|---|
| Fine-grained control of what goes where | The file is **not updated** when the ConfigMap changes (only on Pod restart) |
| Inject one file **without hiding** the rest of the directory | Slightly more complex syntax |
| Mix image content with ConfigMap content | Path mistakes still fail the mount |

**Summary:** `mountPath: <dir>` mounts a directory; `subPath` mounts a single file at an exact path; `items` chooses which keys become files.

### Step 3: expose it with a NodePort Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: frontend-svc
spec:
  type: NodePort          # a fixed port on every node (topic 31)
  selector:
    app: frontend
  ports:
    - protocol: TCP
      port: 80            # ClusterIP port
      targetPort: 80      # container port
      nodePort: 31000     # 30000–32767
```

```bash
kubectl apply -f frontend-svc.yaml
```

**Verify:**

```bash
kubectl get pods -l app=frontend
kubectl exec -it <pod-name> -- cat /usr/share/nginx/html/index.html
curl http://<Node-IP>:31000          # any node's IP
curl http://localhost:31000          # on kind, via extraPortMappings (topic 11)
```

```html
<!DOCTYPE html>
<html>
<head><title>Welcome</title></head>
<body>
  <h1>This page is served by nginx. Welcome to Cloud With VarJosh!!</h1>
</body>
</html>
```

nginx serves the `index.html` that came from the ConfigMap.

### What happens when a ConfigMap changes (Day 28, "dynamic updates")

| Consumed as | Updates inside the running container? |
|---|---|
| `env` / `envFrom` | **No.** Evaluated **once** when the container starts |
| Volume (no `subPath`) | **Yes**, within about a minute (atomic symlink swap). The **app must re-read** the file |
| Volume with `subPath` | **No.** The file is set up once and not refreshed |

- The kubelet implements volume updates by writing a new version and swapping **symlinks**; your app still has to notice (watch the file, reload on a signal, or periodic re-read). nginx, for example, needs `nginx -s reload`.
- For **live updates without restarts**, use volume mounts **without** `subPath`, and an app that hot-reloads, or a **config-reloader** sidecar.
- Otherwise, roll the Pods after changing config: `kubectl rollout restart deploy/frontend-deploy`. A common pattern is to put a **hash of the config** in a Pod-template annotation (Helm's `checksum/config`, Kustomize's `configMapGenerator` name suffixes), so any config change automatically triggers a rollout.

### Immutable ConfigMaps

```yaml
apiVersion: v1
kind: ConfigMap
metadata: { name: frontend-cm-v2 }
immutable: true
data: { ENVIRONMENT: production }
```

- Can't be changed (only deleted and recreated, ideally under a **new name**).
- Protects against accidental edits, and the kubelet **stops watching** it, which reduces API server load in big clusters.

---

## Hands-on lab

1. Apply the ConfigMap and Deployment from Demo 1; check `printenv`.
2. Change `ENVIRONMENT` to `staging` (`kubectl edit cm frontend-cm`). `printenv` still shows `production`. Run `kubectl rollout restart deploy/frontend-deploy` and check again.
3. Apply Demo 2 (directory mount with `items`). Edit the HTML in the ConfigMap, wait about a minute, `cat` the file again: updated. Look at the symlinks: `kubectl exec <pod> -- ls -la /usr/share/nginx/html`.
4. Notice `50x.html` is gone from that directory. Switch to the `subPath` version (`Day 28/frontend-cm-2.yaml`): `50x.html` is back. Edit the ConfigMap again: the `subPath` file does **not** change.
5. Remove `items:` and see `APP` and `ENVIRONMENT` appear as files in the web root (`curl localhost:31000/APP`). That's why `items` matters.
6. Reference a missing ConfigMap and read the `CreateContainerConfigError` event; then add `optional: true` and see the Pod start.
7. Try mounting the whole ConfigMap volume (no `subPath`) at `/usr/share/nginx/html/index.html` and observe the failure.
8. Clean up: `kubectl delete deploy frontend-deploy; kubectl delete svc frontend-svc; kubectl delete cm frontend-cm`.

---

## Where you'll use it

- Application settings per environment (feature flags, URLs, log levels).
- Whole config files: `nginx.conf`, `prometheus.yml`, `application.properties`, Fluent Bit pipelines.
- CoreDNS's Corefile and kube-proxy's config are themselves ConfigMaps (topics 32–33).
- Helm and Kustomize generate ConfigMaps per environment (topics 64–65).

**Best practices:**
- Never put sensitive data in ConfigMaps.
- Mount only the keys you need (`items`).
- Manage ConfigMaps with version-controlled tools (Kustomize, Helm).
- Use `subPath` to drop in single files, knowing it won't live-update.

---

## When it breaks

| Symptom | Cause | Fix |
|---|---|---|
| `CreateContainerConfigError: configmap "x" not found` | ConfigMap missing or in another namespace | Create it in the Pod's namespace; or `optional: true` |
| `couldn't find key FOO in ConfigMap` | Wrong key name | Fix the key |
| Changed config isn't applied | Env vars, or a `subPath` mount, or app doesn't reload | Restart/rollout; use volume mounts + reload |
| App broke after mounting config into `/etc/app` | The mount hid other files in that directory | Mount elsewhere, or use `subPath` for single files |
| `not a directory` / mount errors | Mounted a directory volume over an existing file | Use `subPath`, or mount at the parent directory |
| `ConfigMap "x" is invalid: … Too long` | Over 1 MiB | Split it, or use a volume/object storage |
| Env var names invalid with `envFrom` | Keys like `index.html` aren't valid env var names | They're skipped (warning event); use `items` or separate ConfigMaps |

---

## Quick recap

- ConfigMap = non-secret key-value config, max 1 MiB, namespaced; consumed as `env`/`envFrom`, args (`$(VAR)`), or mounted files.
- Volume mount = a directory of files (hides what was there); `items` picks keys; `subPath` mounts one file at an exact path.
- Updates: env never; volume files yes (~1 min, app must reload); `subPath` never.
- Create imperatively with `--from-literal/--from-file/--from-env-file`; manage declaratively in Git; consider `immutable: true`.

### Interview questions

1. **Q: You updated a ConfigMap but the app still uses the old value. Why?**
   A: If it's consumed as env vars or via `subPath`, the running container never sees changes; restart the Pods. If mounted as a volume, files update within about a minute, but the app must re-read them.
2. **Q: How do you put one file from a ConfigMap into a directory that already has other files?**
   A: Mount it with `subPath` at the exact file path, so it doesn't hide the rest of the directory.
3. **Q: How do you force Pods to restart when config changes?**
   A: Roll the Deployment, ideally automatically by putting a hash of the ConfigMap in a Pod-template annotation (Helm checksum) or using generated names (Kustomize configMapGenerator).

---

## Appendix: notes on the original course material

This topic was rebuilt from the original CKA course notes. This appendix keeps the audit trail out of your way while you learn.

### Sources

Day 28 (introduction; why ConfigMaps; what a ConfigMap is; environment variables explained; hard-coded env vars; Demo 1 env vars with imperative creation and imperative vs declarative; Demo 2 mounted file: block scalar, volume mount, directory vs file, `items`, `subPath` with pros/cons, summary; Step 3 NodePort Service; verification incl. kind; ConfigMap best practices; dynamic updates section), files `Day 28/cm-1.yaml`, `cm-2.yaml`, `ev.yaml`, `frontend-cm-1.yaml`, `frontend-cm-2.yaml`. Secrets are in topic 35.
