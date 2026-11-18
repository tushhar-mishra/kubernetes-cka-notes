# 35 · Secrets: Sensitive Data, Encoding vs Encryption, and Doing It Safely

> **Level:** ★★ intermediate
>
> **You will learn:** what Secrets are for; why **base64 is encoding, not encryption**; Secret types (`Opaque`, `tls`, `dockerconfigjson`, `basic-auth`, `ssh-auth`, `service-account-token`); `data` vs `stringData`; consuming Secrets as env vars or files (and why files are safer); where Secrets actually live (etcd, kubelet tmpfs); turning on **encryption at rest** with an `EncryptionConfiguration` (and KMS); RBAC for Secrets; and the production options beyond plain Secrets (External Secrets Operator, Sealed Secrets, Vault, the Secrets Store CSI driver).
>
> **You need:** topics 34 and 07 (etcd). Encryption is explained just enough here; topic 50 covers it properly.

---

## ⚡ Core: the 10-minute version

**The idea.** A **Secret** is a ConfigMap for sensitive values (passwords, tokens, keys). Its values are **base64-encoded, which isn't encryption**: anyone who can read the Secret can decode it. So protect Secrets with:
- tight RBAC permissions;
- encryption at rest in etcd;
- in production, an external manager (Vault, a cloud secret manager via External Secrets Operator).

Prefer mounting Secrets as **files** rather than env vars.

**Picture it.** base64 is writing a password in pig latin: it isn't hidden, just disguised.

**The one thing to know.**

```bash
kubectl create secret generic db --from-literal=password='s3cr3t'
kubectl get secret db -o jsonpath='{.data.password}' | base64 -d; echo     # anyone with read access can do this
```

```yaml
volumeMounts: [{ name: db, mountPath: /etc/db, readOnly: true }]
volumes: [{ name: db, secret: { secretName: db } }]
```

**Try it now:**
1. Create the Secret and decode it.
2. Mount it in a Pod and `cat /etc/db/password`.
3. Run `kubectl auth can-i get secrets` to see who's allowed to read Secrets.

**You've got it if you can:** explain why base64 isn't security, and name two ways to really protect Secrets.

> 📖 **That's the core.** On a first pass, do the mini-lab and move on to the next topic. Everything below is the **full reference**: come back for the details, the edge cases, interview answers and exam-level depth.

---

## What it is

**Why Secrets:** production apps need sensitive data:
- **database credentials**, so apps can authenticate to databases;
- **API tokens**, keys for APIs and third-party services;
- **SSH private keys**, for encrypted access to remote systems;
- **TLS certificates and keys**, for encryption and identity on HTTPS.

Putting these in images or in plain manifests is insecure.

A **Secret** is a Kubernetes object for **small amounts of sensitive data** (up to 1 MiB). Structurally it's like a ConfigMap, but:
- values are stored **base64-encoded** in `data`;
- Kubernetes treats them more carefully: they can be **encrypted at rest**, are often restricted more tightly by RBAC, are mounted into Pods on **tmpfs** (memory, never written to node disk), and are only sent to nodes that run a Pod using them.

Pods get Secrets as:
1. **environment variables**,
2. **mounted files** (volumes),
3. **command-line arguments** (through env vars; less common).

Kubernetes itself also uses Secrets: image-pull credentials (topic 62), TLS for Ingress (topic 71), bootstrap tokens.

---

## Why it exists

Separating secrets from code and images means:
- the same image runs everywhere, with different credentials per environment;
- credentials can be **rotated** without rebuilding;
- access can be **controlled** (RBAC) and **audited** (API audit logs);
- the platform can protect them better than a random ConfigMap.

But plain Secrets alone are **not** a complete secret-management solution. See "How it works inside".

---

## Analogy

A Secret is a **sealed envelope** in the office mail system, not a safe. Base64 is like writing the password in **mirror writing**: anyone who looks carefully can read it. Real protection comes from **who's allowed to open the mailroom** (RBAC), **locking the filing cabinet** (encryption at rest), and ideally keeping the real valuables in a **bank vault** (an external secret manager) with the envelope just holding a claim ticket.

---

## How it works inside

### Encoding vs encryption

**Kubernetes Secrets use base64 *encoding*, not encryption.** The data is **obfuscated, not secured**: anyone who can read the Secret object can decode it instantly.

**Why encode at all?**
- It hides values from **casual observation** (someone glancing at your screen doesn't instantly see a password).
- It makes **binary data** (certificates, keystores) safe to carry in text formats like YAML and JSON.

But **encoding isn't encryption**: base64 is reversible with one command and no key. To actually **protect** data you need **encryption** (plus Secrets' access controls).

| Feature | Encoding | Encryption |
|---|---|---|
| **Purpose** | Format data for safe transport/storage | Protect data and keep it confidential |
| **Reversible** | Yes, by anyone | Yes, **only with the right key** |
| **Security** | None | Yes |
| **Use case** | Text-safe transmission, compatibility | Passwords, tokens, keys |
| **Examples** | Base64, URL encoding | AES, RSA, TLS |
| **To decode** | Any base64 tool | The decryption key |

(**AES** = a symmetric encryption algorithm; **RSA** = an asymmetric one; **TLS** = the protocol that encrypts HTTPS. Topic 50 explains all three.)

### Where a Secret's value actually goes

```
kubectl apply  ──TLS──▶ API server ──(encrypted? only if configured!)──▶ etcd
                            │
          kubelet on a node that runs a Pod using it ◀──TLS── (Node authorizer: only Secrets its Pods need)
                            │
          tmpfs (RAM) under /var/lib/kubelet/pods/<uid>/volumes/kubernetes.io~secret/…  ──mount──▶ container
          or: env var in the container process's environment
```

**Weak points:**
1. **etcd**: by default Secrets are stored **unencrypted** (just base64) in etcd and in **etcd backups**. Anyone with etcd or backup access reads them all. → Enable **encryption at rest**.
2. **RBAC**: anyone who can `get`/`list` Secrets in a namespace, **or create a Pod there** (a Pod can mount any Secret in its namespace), can read them. → Limit both.
3. **Inside the container**: env vars leak into crash dumps, `/proc/<pid>/environ`, child processes, and sometimes logs ("print all config at start-up"). → Prefer **files**.
4. **Git**: base64 Secrets committed to Git are effectively public. → Never commit them; use Sealed Secrets / External Secrets / SOPS.

### Secret types

| `type` | Holds | Created with |
|---|---|---|
| `Opaque` (default) | Arbitrary key-values | `kubectl create secret generic` |
| `kubernetes.io/tls` | `tls.crt` + `tls.key` | `kubectl create secret tls NAME --cert=… --key=…` |
| `kubernetes.io/dockerconfigjson` | Registry credentials (`.dockerconfigjson`) | `kubectl create secret docker-registry` (topic 62) |
| `kubernetes.io/basic-auth` | `username`, `password` | YAML |
| `kubernetes.io/ssh-auth` | `ssh-privatekey` | YAML |
| `kubernetes.io/service-account-token` | Long-lived SA token (legacy) | Manually, with an annotation (topic 55) |
| `bootstrap.kubernetes.io/token` | Node bootstrap tokens (kubeadm join) | kubeadm |

Typed Secrets are **validated** (e.g. a `tls` Secret must contain both keys).

### `data` vs `stringData`

```yaml
data:
  DB_PASSWORD: ZnJvbnRlbmRwYXNz   # base64 — you encode it
stringData:
  DB_HOST: db.prod.internal       # plain text — the API server encodes it for you (write-only convenience)
```

`stringData` is merged into `data` on write and never shown back. If the same key is in both, `stringData` wins.

### Encryption at rest

The API server can **encrypt Secret values before writing them to etcd**. You give it an **`EncryptionConfiguration`** file:

```yaml
# /etc/kubernetes/enc/enc.yaml (on every control-plane node)
apiVersion: apiserver.config.k8s.io/v1
kind: EncryptionConfiguration
resources:
  - resources: ["secrets"]          # also possible: configmaps, custom resources…
    providers:
      - aescbc:                     # first provider = used for NEW writes
          keys:
            - name: key1
              secret: <base64 of 32 random bytes>    # head -c 32 /dev/urandom | base64
      - identity: {}                # fallback: still READ old unencrypted data
```

```yaml
# kube-apiserver static Pod manifest additions
    - --encryption-provider-config=/etc/kubernetes/enc/enc.yaml
  volumeMounts:
    - { name: enc, mountPath: /etc/kubernetes/enc, readOnly: true }
  volumes:
    - { name: enc, hostPath: { path: /etc/kubernetes/enc, type: DirectoryOrCreate } }
```

- Providers are tried **in order for reading**; the **first** is used for **writing**. `identity` = no encryption.
- Existing Secrets stay unencrypted until **rewritten**: `kubectl get secrets -A -o json | kubectl replace -f -`.
- Verify by reading etcd directly: an encrypted value starts with `k8s:enc:aescbc:v1:key1:` instead of readable text.
- **Better:** the **`kms` v2** provider. The data-encryption keys are wrapped by a key in a cloud KMS or HSM (AWS KMS, Azure Key Vault, GCP KMS, Vault), so the master key never sits in a file on the control plane. Managed clusters (EKS, GKE, AKS) offer KMS envelope encryption as an option, and some enable it by default.
- `aesgcm` and `secretbox` also exist; `aescbc` is the classic CKA example (`secretbox` or `kms` are preferred for new setups).

### Production secret management

| Approach | How | Notes |
|---|---|---|
| **External Secrets Operator (ESO)** | A controller reads from AWS Secrets Manager, GCP Secret Manager, Azure Key Vault, Vault… and **creates/updates Kubernetes Secrets** | Most common today; Git holds only the *reference* (`ExternalSecret`) |
| **Secrets Store CSI Driver** | Mounts secrets **directly from the external store as files**, optionally syncing to a Secret | Secrets need never be stored in etcd |
| **Sealed Secrets** (Bitnami) | Encrypt a Secret with the cluster's public key into a `SealedSecret` that's **safe to commit**; the in-cluster controller decrypts it | Simple GitOps-friendly option |
| **SOPS** (+ age/KMS) | Encrypt values in YAML files in Git; decrypt in CI or Argo CD/Flux | Popular with Flux |
| **HashiCorp Vault** | Central secret store with dynamic, short-lived credentials (e.g. per-Pod DB users), via the Vault Agent injector or CSI | Most powerful, most to operate |
| **Workload identity** | Pods get **cloud IAM roles** (EKS Pod Identity, GKE Workload Identity) instead of static keys | Best: no long-lived secret at all |

Topic 63 covers these in depth.

---

## Commands / YAML

### Demo 1: inject a Secret into a Pod

**Step 1: create the Secret** (`Day 28/secret-1.yaml`):

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: frontend-secret
type: Opaque                         # user-defined key-values
data:
  DB_USER: ZnJvbnRlbmR1c2Vy          # base64 of 'frontenduser'
  DB_PASSWORD: ZnJvbnRlbmRwYXNz      # base64 of 'frontendpass'
  # values under data: MUST be base64-encoded
```

Encode and decode:

```bash
echo -n 'frontenduser' | base64            # ZnJvbnRlbmR1c2Vy
echo -n 'frontendpass' | base64            # ZnJvbnRlbmRwYXNz
echo 'ZnJvbnRlbmR1c2Vy' | base64 --decode  # frontenduser
echo 'ZnJvbnRlbmRwYXNz' | base64 --decode  # frontendpass
```

- **`-n`** stops `echo` adding a newline, which would otherwise become part of the encoded password: a classic "password is wrong" bug.

```bash
kubectl apply -f "Day 28/secret-1.yaml"
```

Easier and safer (no hand-encoding; Day 28 best practice):

```bash
kubectl create secret generic frontend-secret \
  --from-literal=DB_USER=frontenduser \
  --from-literal=DB_PASSWORD=frontendpass
kubectl create secret generic app-cert --from-file=tls.crt --from-file=tls.key
```

(`--from-literal` puts the password in your shell history; `--from-file` or reading from a prompt avoids that.)

**Step 2: use it from the Deployment** (`Day 28/frontend-secret-1.yaml`):

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend-deploy
spec:
  replicas: 1
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
            - name: DB_USER                  # env var in the container
              valueFrom:
                secretKeyRef:
                  name: frontend-secret      # the Secret
                  key: DB_USER               # the key in it
            - name: DB_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: frontend-secret
                  key: DB_PASSWORD
```

Kubernetes **base64-decodes** the values and injects them as **plain text**: inside the container they're ordinary env vars, even though the Secret object stores them encoded.

```bash
kubectl apply -f "Day 28/frontend-secret-1.yaml"
```

**Step 3: verify**

```bash
kubectl get pods -l app=frontend
kubectl exec -it <pod-name> -- /bin/sh
echo $DB_USER        # frontenduser
echo $DB_PASSWORD    # frontendpass
```

### Alternative: mount the Secret as files (Day 28, `Day 28/frontend-secret-2.yaml`)

```yaml
      containers:
        - name: frontend-container
          image: nginx:1.29
          volumeMounts:
            - name: secret-volume          # refers to the volume below
              mountPath: /etc/secrets      # directory in the container
              readOnly: true               # the container can't modify it
      volumes:
        - name: secret-volume
          secret:
            secretName: frontend-secret    # which Secret
            defaultMode: 0400              # file permissions (owner read-only)
            # items:                       # optional: only some keys, custom file names
            #   - key: DB_USER
            #     path: db-user.txt
            #   - key: DB_PASSWORD
            #     path: db-password.txt
```

- Each key becomes a **file** in `/etc/secrets`: `/etc/secrets/DB_USER`, `/etc/secrets/DB_PASSWORD`.
- The files contain the **decoded plain text**.
- Without `items`, **all** keys are mounted.
- The volume is **tmpfs** (RAM-backed) on the node.

`Day 28/backend-2.yaml` mounts a `backend-secret` the same way into the http-echo backend (create `backend-secret` first).

Why files are usually better than env vars:
- Files **update** when the Secret changes (unless `subPath`), so rotation without restarts is possible.
- Env vars are visible to anything that can dump the process environment, and are inherited by child processes.
- Many apps support `*_FILE` conventions (`POSTGRES_PASSWORD_FILE=/etc/secrets/DB_PASSWORD`).

### Dynamic updates (Day 28, same rules as ConfigMaps)

- `secretKeyRef` / `envFrom` env vars are evaluated **once** at container start; updating the Secret doesn't change them.
- Volume-mounted Secrets (without `subPath`) **are updated** through the kubelet's symlink swap, but the app must **re-read** the files.
- `subPath` mounts are **not** updated.
- For live rotation: volume mounts without `subPath` + an app that reloads (or a reloader controller such as Stakater Reloader, which rolls Deployments when a Secret changes).

### Reading Secrets (carefully)

```bash
kubectl get secret frontend-secret -o jsonpath='{.data.DB_PASSWORD}' | base64 -d; echo
kubectl get secret frontend-secret -o go-template='{{range $k,$v := .data}}{{$k}}={{$v|base64decode}}{{"\n"}}{{end}}'
```

### RBAC: who can read Secrets

```bash
kubectl auth can-i get secrets -n prod --as=dev-user
kubectl auth can-i create pods -n prod --as=dev-user     # a Pod can mount any Secret in its namespace!
```

Topic 57 shows how to grant Secret access to a **single named Secret** (`resourceNames`).

### Immutable Secrets

```yaml
immutable: true
```

Same as ConfigMaps: can't be edited; reduces kubelet watch load; rotate by creating a new name.

---

## Hands-on lab

1. Create `frontend-secret` imperatively, apply `frontend-secret-1.yaml`, and check the env vars.
2. Switch to `frontend-secret-2.yaml`: `ls -l /etc/secrets`, `cat` the files, and `df -h /etc/secrets` (type `tmpfs`).
3. Update the password (`kubectl create secret generic frontend-secret --from-literal=DB_USER=frontenduser --from-literal=DB_PASSWORD=newpass --dry-run=client -o yaml | kubectl apply -f -`). After about a minute the **file** changes; the **env var** doesn't.
4. Prove Secrets are plain in etcd (kind):
   ```bash
   docker exec lab-control-plane sh -c 'ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 \
     --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kubernetes/pki/etcd/server.crt --key=/etc/kubernetes/pki/etcd/server.key \
     get /registry/secrets/default/frontend-secret' | strings | grep -i pass
   ```
   (If `etcdctl` isn't on the node, run it inside the etcd Pod.) You'll see the password.
5. Enable `aescbc` encryption at rest on the kind control plane using the YAML above (edit `/etc/kubernetes/manifests/kube-apiserver.yaml`; the API server restarts). Recreate the Secret and repeat step 4: the value now starts with `k8s:enc:aescbc:v1:key1:`.
6. Create a `kubernetes.io/tls` Secret from a self-signed certificate (`openssl req -x509 -newkey rsa:2048 -nodes -keyout tls.key -out tls.crt -subj /CN=demo -days 1`).
7. Clean up the Deployment and Secrets. (Leave encryption on if you like; it doesn't hurt the lab.)

---

## Where you'll use it

- Database credentials, API keys, TLS certificates for Ingress/Gateway, registry credentials.
- Security reviews and audits: "Is encryption at rest on? Who can read Secrets? Are Secrets in Git?"
- Platform design: choosing ESO/CSI/Sealed Secrets/Vault and workload identity.

**Best practices:**
- **Never store Secrets in Git** in plain or base64 form.
- Use `kubectl create secret` or Helm/Kustomize generators instead of hand-encoding.
- **Enable encryption at rest** for etcd (preferably KMS).
- **Restrict access with RBAC**, including who can create Pods in sensitive namespaces.
- Use **external secret managers** for stronger security and auditing.
- Mount specific keys as files with `items`/`subPath` when you only need some of them.
- Prefer files over env vars; rotate regularly; prefer workload identity over static cloud keys.

---

## When it breaks

| Symptom | Cause | Fix |
|---|---|---|
| `CreateContainerConfigError: secret "x" not found` | Missing Secret or wrong namespace | Create it in the Pod's namespace |
| App rejects the password | Encoded with a trailing newline (`echo` without `-n`) | Re-encode with `echo -n`, or use `--from-literal` |
| `illegal base64 data at input byte` on apply | Plain text in `data:` | Encode it, or use `stringData:` |
| Rotated Secret but app uses the old value | Env var or `subPath` | Use volume mounts + reload, or roll the Pods |
| API server won't start after enabling encryption | Wrong path/volume mount for the config, or a bad key | Check the static Pod manifest and `crictl logs` of the API server (topic 82) |
| Lost the encryption key | Encrypted Secrets are unreadable forever | Back up the key securely, with etcd backups (topic 77) |

---

## Quick recap

- Secrets hold sensitive data; values are **base64-encoded, not encrypted**.
- Consume via `secretKeyRef`/`envFrom` (static) or volumes (tmpfs files that update; prefer these).
- Protect them: **encryption at rest** (`EncryptionConfiguration`, ideally KMS v2), tight **RBAC** (including Pod creation), no Secrets in Git.
- Production: External Secrets Operator, Secrets Store CSI, Sealed Secrets/SOPS, Vault, and workload identity.

### Interview questions

1. **Q: Are Kubernetes Secrets encrypted?**
   A: Not by default: they're base64-encoded and stored in etcd as-is. You must enable encryption at rest (EncryptionConfiguration with aescbc/secretbox or a KMS provider) and restrict RBAC.
2. **Q: Env var or mounted file for a Secret: which is better?**
   A: Files: they're on tmpfs, can be updated without restarting, and are less likely to leak via process environment dumps or logs. Env vars are simpler but static.
3. **Q: How do you manage Secrets in a GitOps workflow?**
   A: Don't commit plain Secrets; use Sealed Secrets or SOPS-encrypted files, or commit only ExternalSecret references that the External Secrets Operator resolves from a cloud secret manager or Vault.

---

## Appendix: notes on the original course material

This topic was rebuilt from the original CKA course notes. This appendix keeps the audit trail out of your way while you learn.

### Sources

Day 28 (why Secrets; what a Secret is; encoding vs encryption distinction and table; Secrets Demo 1: create, encode/decode, Deployment with `secretKeyRef`, verification; alternative: mount as files with `items` notes; Secret best practices; dynamic updates; conclusion), files `Day 28/secret-1.yaml`, `frontend-secret-1.yaml`, `frontend-secret-2.yaml`, `backend-2.yaml`.
