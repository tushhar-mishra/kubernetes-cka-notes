# 63 · Supply-Chain Security and Secrets in Production

> **Level:** ★★★ advanced
>
> **You will learn:** what "software supply chain" means for containers; image **scanning** (Trivy), **SBOMs**, **signing and verification** (Sigstore/cosign) and enforcing them at admission; trusted base images, digests and minimal images; and how real teams handle **Secrets**: encryption at rest in etcd (KMS), External Secrets Operator, Sealed Secrets, SOPS, Vault and the Secrets Store CSI driver, with the trade-offs.
>
> **You need:** topics 03, 20, 35 (Secrets), 58 (admission), 62 (registries).

---

## ⚡ Core: the 10-minute version

**The idea.** Securing what you **run** means securing the whole **supply chain**:
- use minimal base images and pin them by **digest**;
- **scan** for known vulnerabilities (Trivy);
- produce an **SBOM** (the list of everything inside);
- **sign** images in CI (cosign);
- **verify signatures at admission** (Kyverno).

Securing **secrets** means:
- encrypting them in etcd (KMS);
- keeping plaintext out of Git (External Secrets Operator, Sealed Secrets, SOPS);
- tight RBAC, file mounts, and short-lived credentials.

**Picture it.** Food safety: check the ingredients, list them on the label, seal the package at the factory, and have the shop refuse unsealed goods.

**The one thing to know.**

```bash
trivy image --severity HIGH,CRITICAL --ignore-unfixed nginx:1.27      # what's vulnerable, and is there a fix?
cosign sign <registry>/app@sha256:<digest>                           # in CI
cosign verify <registry>/app@sha256:<digest> --certificate-identity-regexp '…' --certificate-oidc-issuer '…'
```

**Try it now:**
1. Scan `nginx:1.27` and a minimal image (e.g. a Chainguard image) and compare.
2. Create a Secret and check whether your cluster encrypts it at rest (topic 77 shows how to read etcd).

**You've got it if you can:** list the supply-chain steps from build to admission, and three ways to keep Secrets out of Git.

> 📖 **That's the core.** On a first pass, do the mini-lab and move on to the next topic. Everything below is the **full reference**: come back for the details, the edge cases, interview answers and exam-level depth.

---

## What it is

The **software supply chain** is everything between a developer's commit and a running container: source code, dependencies, base images, the build system, the registry, and deployment. A **supply-chain attack** compromises one of those links (a malicious dependency, a poisoned base image, a hijacked CI pipeline, a swapped image tag) so you deploy something you never intended.

**Secrets management** is how passwords, API keys, certificates and tokens reach Pods **without** being exposed in Git, images, logs, or a readable etcd.

---

## Why it exists

- Images contain hundreds of OS packages and libraries; new CVEs (Common Vulnerabilities and Exposures: publicly catalogued security flaws) appear daily.
- Tags are **mutable**: `app:1.0.0` today may not be `app:1.0.0` tomorrow.
- Kubernetes Secrets are just **base64** in the API, and stored **unencrypted in etcd by default** (topic 35). Anyone with etcd access or a backup, or `get secrets` RBAC, reads them.
- Secrets committed to Git live there **forever** (history).

---

## Analogy

Supply-chain security is **food safety**: you check ingredients (dependency/CVE scanning), keep an **ingredient list** on every package (SBOM), **seal** packages at the factory (signing), and the shop **refuses unsealed or tampered packages** (admission verification). Secrets management is a **bank vault with a courier service**: valuables live in the vault (secret manager), and couriers deliver just what each shop needs, when needed, in locked bags (encrypted at rest, scoped access, rotation).

---

## How it works inside

### Securing images: build → scan → sign → verify

**1. Start small and trusted**
- Minimal base images: **distroless**, **Chainguard/Wolfi**, `-slim`/`alpine`, or `scratch` for static binaries: fewer packages = fewer CVEs and no shell for attackers.
- Multi-stage builds (topic 03): compilers and build tools stay out of the final image.
- Run as non-root (topic 59).

**2. Pin by digest**
- An image **digest** (`sha256:…`) is the hash of the image manifest: **immutable**. `nginx:1.27@sha256:abc…` always means exactly that content (topic 20).
- Tools like Renovate/Dependabot update digests via pull requests.

**3. Scan for vulnerabilities**
- **Trivy** (Aqua), **Grype** (Anchore), registry-built-in scanners (ECR, Harbor, GHCR/Docker Scout).
- In CI: fail the build on fixable **critical/high** CVEs; in the cluster: the **Trivy Operator** continuously scans running workloads and reports `VulnerabilityReport` objects.

```bash
trivy image --severity HIGH,CRITICAL --ignore-unfixed nginx:1.27
```

```
nginx:1.27 (debian 12.x)
Total: 3 (HIGH: 3, CRITICAL: 0)
┌──────────┬────────────────┬──────────┬────────┬───────────────────┬───────────────┐
│ Library  │ Vulnerability  │ Severity │ Status │ Installed Version │ Fixed Version │
├──────────┼────────────────┼──────────┼────────┼───────────────────┼───────────────┤
│ libexpat1│ CVE-2024-xxxxx │ HIGH     │ fixed  │ 2.5.0-1           │ 2.5.0-1+deb12u1│
…
```

Read: which package, which CVE, whether a fixed version exists (if yes: rebuild with an updated base).

**4. SBOM: Software Bill of Materials**
- A machine-readable list of every component and version in an image (formats **SPDX** and **CycloneDX**). Generate with `syft` or `docker buildx --sbom=true`.
- When the next Log4Shell-style CVE appears, you can answer "which images contain it?" in minutes.

**5. Sign and attest**
- **Sigstore cosign** signs an image digest and stores the signature in the registry next to it. **Keyless** signing uses your CI's OIDC identity (e.g. GitHub Actions), recorded in the public **Rekor** transparency log; no long-lived keys to leak.
- **Attestations** (signed statements) record *how* it was built: **SLSA provenance** (Supply-chain Levels for Software Artifacts), SBOMs, scan results.

```bash
cosign sign registry.example.com/app@sha256:abc…                    # keyless in CI
cosign verify registry.example.com/app@sha256:abc… \
  --certificate-identity-regexp 'https://github.com/myorg/.+' \
  --certificate-oidc-issuer https://token.actions.githubusercontent.com
```

**6. Verify at admission**
- **Kyverno `verifyImages`** or **Sigstore policy-controller** (or Gatekeeper + Ratify) reject Pods whose images aren't signed by your CI, and can **rewrite tags to digests**.
- Plus simple rules: images only from `registry.example.com/` (topic 58), no `:latest`.

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata: { name: verify-signed-images }
spec:
  webhookTimeoutSeconds: 15
  rules:
  - name: check-signature
    match: { any: [ { resources: { kinds: ["Pod"] } } ] }
    verifyImages:
    - imageReferences: ["registry.example.com/*"]
      failureAction: Enforce                # per-rule; spec.validationFailureAction is deprecated
      mutateDigest: true                    # pin the tag to the verified digest
      attestors:
      - entries:
        - keyless:
            subject: "https://github.com/myorg/*"
            issuer: "https://token.actions.githubusercontent.com"
```

**7. Protect the pipeline itself**: least-privilege CI credentials (OIDC federation instead of stored secrets; topic 55), protected branches and required reviews, pinned CI actions, isolated build runners.

### Secrets in production

**Layer 1: encrypt Secrets in etcd**
- The API server can encrypt resources before writing them to etcd, configured by an **`EncryptionConfiguration`** file (`--encryption-provider-config`).
- Providers: `aescbc`/`aesgcm`/`secretbox` (key in a file on the control plane: protects backups, not a compromised control plane) or **`kms` v2** (the data key is wrapped by an external **KMS**: AWS KMS, Azure Key Vault, GCP KMS, Vault; recommended).
- Managed clusters: EKS/GKE/AKS offer KMS envelope encryption (often on by default now).

```yaml
apiVersion: apiserver.config.k8s.io/v1
kind: EncryptionConfiguration
resources:
- resources: ["secrets"]
  providers:
  - aescbc:
      keys:
      - name: key1
        secret: <base64 of 32 random bytes>     # head -c 32 /dev/urandom | base64
  - identity: {}                                 # still able to READ old unencrypted data
```

After enabling, rewrite existing Secrets so they're re-stored encrypted: `kubectl get secrets -A -o json | kubectl replace -f -`. Check in etcd: `etcdctl get /registry/secrets/default/x | hexdump -C` shows `k8s:enc:aescbc:v1:key1` instead of plaintext (topic 77).

**Layer 2: keep plaintext out of Git**

| Tool | How | Pros | Cons |
|---|---|---|---|
| **External Secrets Operator (ESO)** | `ExternalSecret` CRs say "fetch key X from AWS Secrets Manager/Vault/GCP/Azure", the operator creates and **refreshes** a normal Secret | Source of truth in a real secret manager; rotation; Git holds only references | A Kubernetes Secret still exists (protect etcd + RBAC) |
| **Sealed Secrets** (Bitnami) | Encrypt with the cluster's public key (`kubeseal`); commit the `SealedSecret`; the in-cluster controller decrypts | Simple, GitOps-friendly | Tied to one cluster's key; rotation is manual; key backup essential |
| **SOPS** (+ age/KMS) | Encrypt values inside YAML files; Flux/Argo CD plugins or `helm-secrets` decrypt at deploy | Readable diffs (keys visible, values encrypted) | Decryption keys must reach the deployer |
| **Secrets Store CSI Driver** | Mounts secrets from Vault/AWS/Azure/GCP **directly as files** in the Pod (optionally syncing to a Secret) | Can avoid Kubernetes Secrets entirely | App reads files; per-provider setup |
| **Vault Agent injector / Vault Secrets Operator** | Vault authenticates the Pod by its SA token (topic 55) and injects secrets | Dynamic secrets (e.g. per-Pod DB credentials with short TTLs) | Run and operate Vault |

**Layer 3: access and hygiene**
- RBAC: few identities can `get/list/watch` Secrets (note: `list` returns contents too; `create pods` in a namespace effectively grants reading its Secrets by mounting them).
- Prefer **files** (volume mounts) over **env vars**: env vars leak into crash dumps, `/proc/<pid>/environ`, and child processes; mounted Secrets also **update** when rotated (env vars don't).
- `immutable: true` on Secrets that never change (safer, faster).
- **Rotate** regularly; prefer **short-lived, dynamic** credentials (Vault database secrets, cloud IAM via workload identity) over long-lived passwords.
- Scan repositories and images for leaked secrets (`gitleaks`, `trufflehog`, `trivy fs --scanners secret`).
- Audit logs on Secret access (topic 80).

---

## Commands / YAML

### External Secrets Operator example

```yaml
apiVersion: external-secrets.io/v1
kind: SecretStore
metadata: { name: aws-sm, namespace: shop }
spec:
  provider:
    aws:
      service: SecretsManager
      region: eu-west-1
      auth:
        jwt: { serviceAccountRef: { name: eso-reader } }   # IRSA/Pod Identity, no static keys
---
apiVersion: external-secrets.io/v1
kind: ExternalSecret
metadata: { name: db-creds, namespace: shop }
spec:
  refreshInterval: 1h
  secretStoreRef: { name: aws-sm, kind: SecretStore }
  target: { name: db-creds }                               # the Kubernetes Secret to create
  data:
  - secretKey: password
    remoteRef: { key: prod/shop/db, property: password }
```

```bash
kubectl get externalsecret -n shop
```

```
NAME       STORE    REFRESH INTERVAL   STATUS         READY
db-creds   aws-sm   1h                 SecretSynced   True
```

(The ESO API version has moved over time: older installs use `external-secrets.io/v1beta1`.)

### Sealed Secrets example

```bash
kubectl create secret generic api-key --from-literal=key=s3cr3t --dry-run=client -o yaml \
  | kubeseal --format yaml > api-key-sealed.yaml        # safe to commit
kubectl apply -f api-key-sealed.yaml                     # controller creates Secret 'api-key'
```

---

## Hands-on lab

1. `trivy image nginx:1.27`, then `trivy image cgr.dev/chainguard/nginx`; compare CVE counts and image sizes.
2. Generate an SBOM: `syft nginx:1.27 -o spdx-json > sbom.json`; search it for `openssl`.
3. Push an image to a local registry, sign it with `cosign generate-key-pair` + `cosign sign --key`, and verify; install Kyverno with a `verifyImages` policy using that public key; deploy signed vs unsigned images.
4. On kind, enable `aescbc` encryption: write the config into the control-plane node, add the flag and a volume mount to `kube-apiserver.yaml`, create a Secret, and read it raw from etcd before/after (topic 77 shows `etcdctl` access).
5. Install Sealed Secrets via Helm and seal/unseal a Secret.
6. Clean up the controllers and policies.

---

## Where you'll use it

- **CI pipelines**: scan, SBOM, sign on every build; fail on fixable critical CVEs.
- **Cluster admission**: only signed images from approved registries.
- **Every app with credentials**: ESO/Vault/CSI instead of hand-made Secrets; KMS encryption at rest.
- **Audits** (SOC 2, ISO 27001, PCI DSS) and **incident response** ("are we affected by CVE-X?").

---

## When it breaks

| Symptom | Cause | Fix |
|---|---|---|
| Pods rejected: `image verification failed` | Unsigned image, wrong identity/issuer in the policy, or registry unreachable from the policy engine | Sign in CI; fix attestor settings; network access |
| Admission timeouts on large rollouts | Signature checks are slow | Raise `webhookTimeoutSeconds`; enable caching |
| `ExternalSecret` status `SecretSyncedError` | Wrong IAM/permissions or key path | Describe the ExternalSecret; fix the store's auth |
| App keeps old password after rotation | Using env vars, or app doesn't re-read files | Volume mount + reload/restart (e.g. Reloader controller) |
| API server won't start after enabling encryption | Bad config path/mount or key size | Fix the file/mount (topic 82) |
| Secrets unreadable after restoring a backup | Encryption key not backed up | **Back up the encryption config/KMS access with etcd** |
| SealedSecret won't decrypt in a new cluster | Different controller key | Restore the sealing key or re-seal |

---

## Quick recap

- Supply chain: minimal trusted bases, **digests**, **scanning** (Trivy), **SBOMs**, **signing** (cosign keyless) and **verification at admission** (Kyverno/policy-controller), plus a locked-down CI.
- Secrets: **encrypt etcd** (KMS v2), keep plaintext **out of Git** (ESO, Sealed Secrets, SOPS, CSI driver, Vault), tight RBAC, files over env vars, rotation, short-lived credentials.

### Interview questions

1. **Q: How do you make sure only images built by your CI run in the cluster?**
   A: Sign images in CI (cosign, keyless with the CI's OIDC identity), restrict registries, and verify signatures at admission with Kyverno `verifyImages` or the Sigstore policy-controller, pinning tags to digests.
2. **Q: Kubernetes Secrets are "just base64". How do you secure them?**
   A: Encrypt at rest with a KMS provider, restrict RBAC (`get/list/watch` on secrets, and who can create Pods), keep them out of Git with External Secrets/Sealed Secrets/SOPS or mount directly via the Secrets Store CSI driver, prefer volume mounts, rotate and use short-lived credentials.
3. **Q: What is an SBOM and why does it matter?**
   A: A Software Bill of Materials listing every component and version in an artifact; it lets you instantly find which images are affected by a newly disclosed vulnerability and supports compliance.

---

## Appendix: notes on the original course material

This topic was rebuilt from the original CKA course notes. This appendix keeps the audit trail out of your way while you learn.

### Sources

none: a **new topic**. It expands points made elsewhere: Day 28's warning that Secrets are only base64 (topic 35), Day 45's best practices (external secret managers, rotation; topic 62), Day 38's image-provenance policies (topic 58), and Day 37/55's Vault and Sealed Secrets mentions.
