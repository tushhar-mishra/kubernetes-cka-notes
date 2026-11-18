# 77 · etcd Backup, Restore and Disaster Recovery

> **Level:** ★★★ advanced
>
> **You will learn:** what etcd stores and how (keys under `/registry`, Raft consensus, quorum, revisions, compaction); talking to etcd safely with `etcdctl` and its TLS certificates; taking **snapshots** (and verifying them); **restoring** a kubeadm cluster from a snapshot step by step; quorum loss and member replacement in HA etcd; etcd health and maintenance (defrag, alarms, size); and a broader **disaster-recovery** strategy (Velero, GitOps, RPO/RTO, testing restores).
>
> **You need:** topics 07 (control plane), 42 (static Pods), 53 (certificates), 75 (kubeadm), 76 (upgrades).

---

## ⚡ Core: the 10-minute version

**The idea.** **etcd** holds the entire state of the cluster: lose it and you lose every object. Back it up with a **snapshot**, verify the snapshot, and store it **off the cluster** together with the PKI and encryption keys. To restore on kubeadm:
1. Stop the API server and etcd (move their manifests out of `/etc/kubernetes/manifests`).
2. Restore the snapshot into a **new** data directory.
3. Point etcd's manifest at it.
4. Start etcd, then the API server.

etcd stays writable only while a **majority** (quorum) of its members is up.

**Picture it.** Photocopying the company's master ledger and keeping the copy in a safe elsewhere.

**The one thing to know.**

```bash
sudo ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key snapshot save /opt/snap.db
sudo etcdutl snapshot status /opt/snap.db -w table
sudo etcdutl snapshot restore /opt/snap.db --data-dir /var/lib/etcd-restored
```

**Try it now:**
1. Take a snapshot.
2. Delete a Deployment.
3. Restore the snapshot and see the Deployment come back.

This is a near-certain CKA task.

**You've got it if you can:** back up and restore etcd from memory, and explain quorum.

> 📖 **That's the core.** On a first pass, do the mini-lab and move on to the next topic. Everything below is the **full reference**: come back for the details, the edge cases, interview answers and exam-level depth.

---

## What it is

**etcd** ("et-see-dee", from `/etc` + "distributed") is a **distributed, consistent key-value store**: the **only** place Kubernetes keeps cluster state. Every object (Pods, Deployments, Secrets, ConfigMaps, CRs, RBAC, Leases) is stored there; **only the API server talks to etcd**.

A **snapshot** is a point-in-time copy of the whole etcd database. **Restoring** creates a new etcd data directory from a snapshot, and the cluster's state goes back to that moment.

---

## Why it exists

Lose etcd and you lose the cluster's **memory**: every Deployment, Service, Secret, RBAC rule, CR. Running containers keep running for a while, but nothing can be managed or recovered. Causes: disk failure, accidental `kubectl delete` of critical objects or a whole namespace, a botched upgrade, corruption, ransomware, losing quorum. A tested **backup + restore** procedure is the difference between minutes of downtime and rebuilding everything.

---

## Analogy

etcd is the **company's master ledger**: every decision ever made is recorded there and every manager (controller) reads from it. With three or five **bookkeepers** keeping identical copies (Raft), a change only counts once a **majority** has written it down (quorum). A snapshot is a **photocopy of the whole ledger** stored in the safe. If the ledger burns, you restore from the photocopy, and anything written after the copy was made is lost (the RPO).

---

## How it works inside

### Data model

- Keys like `/registry/<resource>/<namespace>/<name>`, e.g. `/registry/deployments/default/web`, `/registry/secrets/kube-system/…`.
- Values are **Protobuf-encoded** objects (not human-readable JSON), and Secrets may be **encrypted** (topic 63).
- Every write increases a global **revision**; the API server's `resourceVersion` comes from it, and **watches** (topic 14) stream changes since a revision.
- Old revisions are kept until **compaction** (the API server compacts every 5 minutes); the freed space is reclaimed by **defragmentation**.
- Default size limit (**quota**): 2 GiB (`--quota-backend-bytes`, up to ~8 GiB recommended). Hitting it raises a **NOSPACE alarm**: etcd becomes **read-only** and the cluster stops accepting writes.

### Raft and quorum

- etcd members use the **Raft** consensus algorithm: one **leader** handles writes; a write is committed when a **majority** (quorum) of members has it.
- **Quorum = ⌊n/2⌋ + 1**: 3 members tolerate **1** failure, 5 tolerate **2**. Even numbers add no tolerance (4 still tolerates only 1). Use 3 or 5.
- Lose quorum (e.g. 2 of 3 down) → etcd **refuses writes and linearizable reads** → the API server can't work.
- etcd is sensitive to **disk latency** (fsync on every write: use SSDs) and **network latency** between members.

### Where etcd runs on kubeadm

- **Stacked**: an `etcd` static Pod on each control-plane node (`/etc/kubernetes/manifests/etcd.yaml`), data in **`/var/lib/etcd`** (a hostPath), certificates in `/etc/kubernetes/pki/etcd/`.
- **External**: a separate etcd cluster on its own machines (more isolation, more to run; topic 78).
- Client port **2379**, peer port **2380**; metrics on `127.0.0.1:2381`.

### Security

etcd uses **mutual TLS**: clients (the API server, you with `etcdctl`) must present a certificate signed by the **etcd CA**. Anyone with etcd access can read **every Secret** (unless encryption at rest is on), so snapshots are **highly sensitive**: encrypt them and restrict access.

---

## Commands / YAML

### Find the connection details

```bash
sudo grep -E -- '--(listen-client-urls|advertise-client-urls|cert-file|key-file|trusted-ca-file|data-dir)' /etc/kubernetes/manifests/etcd.yaml
```

```
    - --advertise-client-urls=https://10.0.1.10:2379
    - --cert-file=/etc/kubernetes/pki/etcd/server.crt
    - --data-dir=/var/lib/etcd
    - --key-file=/etc/kubernetes/pki/etcd/server.key
    - --listen-client-urls=https://127.0.0.1:2379,https://10.0.1.10:2379
    - --trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
```

Those three files are what `etcdctl` needs. (Using the **server** cert as a client works because kubeadm's etcd server certificates include client-auth usage; `healthcheck-client.crt` or `apiserver-etcd-client.crt` are the "proper" client certs.)

Install `etcdctl` on the node if missing (`apt install etcd-client`, or the release tarball matching your etcd version), or run it inside the etcd Pod: `kubectl -n kube-system exec etcd-control-plane -- etcdctl …` (paths are the same inside).

Set the common flags once:

```bash
export ETCDCTL_API=3          # default in etcdctl 3.4+; required on older clients
ETCD="--endpoints=https://127.0.0.1:2379 \
 --cacert=/etc/kubernetes/pki/etcd/ca.crt \
 --cert=/etc/kubernetes/pki/etcd/server.crt \
 --key=/etc/kubernetes/pki/etcd/server.key"
```

### Health and membership

```bash
sudo etcdctl $ETCD endpoint health
sudo etcdctl $ETCD endpoint status --write-out=table
sudo etcdctl $ETCD member list --write-out=table
```

```
https://127.0.0.1:2379 is healthy: successfully committed proposal: took = 9.2ms
+------------------------+------------------+---------+---------+-----------+-----------+------------+
|        ENDPOINT        |        ID        | VERSION | DB SIZE | IS LEADER | RAFT TERM | RAFT INDEX |
+------------------------+------------------+---------+---------+-----------+-----------+------------+
| https://127.0.0.1:2379 | 8e9e05c52164694d |  3.5.21 |   12 MB |      true |         2 |      48211 |
+------------------------+------------------+---------+---------+-----------+-----------+------------+
```

- `DB SIZE` vs the quota; `IS LEADER`; `RAFT TERM` increases on each leader election (frequent increases = instability).

Peek at keys (read-only):

```bash
sudo etcdctl $ETCD get /registry/deployments/default --prefix --keys-only | head
```

### Take a snapshot

```bash
sudo mkdir -p /var/backups/etcd
sudo etcdctl $ETCD snapshot save /var/backups/etcd/snap-$(date +%F-%H%M).db
```

```
{"level":"info","msg":"created temporary db file","path":"/var/backups/etcd/snap-2026-11-18-1015.db.part"}
{"level":"info","msg":"fetched snapshot","endpoint":"https://127.0.0.1:2379","size":"12 MB","took":"0.2s"}
Snapshot saved at /var/backups/etcd/snap-2026-11-18-1015.db
```

**Verify it:**

```bash
sudo etcdutl snapshot status /var/backups/etcd/snap-2026-11-18-1015.db --write-out=table
```

```
+----------+----------+------------+------------+
|   HASH   | REVISION | TOTAL KEYS | TOTAL SIZE |
+----------+----------+------------+------------+
| 3c4f2a1b |    48230 |       1210 |      12 MB |
+----------+----------+------------+------------+
```

(`etcdctl snapshot status` still works in 3.5 but is deprecated; use `etcdutl`, the offline tool shipped with etcd.)

**Production:** run snapshots on a schedule (a CronJob on control-plane nodes or a systemd timer), copy them **off the cluster** (object storage, encrypted, versioned, separate account), keep several generations, and also back up **`/etc/kubernetes/pki`** (the CA keys; without them, restored data won't match new certificates) and the **encryption-at-rest config/KMS access** (topic 63).

### Restore a single-control-plane kubeadm cluster

Scenario: someone deleted important objects (or etcd data is corrupt); you restore to the snapshot.

**1. Stop the API server and etcd** (move their static Pod manifests away; the kubelet stops them):

```bash
sudo mkdir -p /etc/kubernetes/manifests-off
sudo mv /etc/kubernetes/manifests/{kube-apiserver,etcd}.yaml /etc/kubernetes/manifests-off/
sudo crictl ps | grep -E 'etcd|kube-apiserver'      # wait until both are gone
```

**2. Restore the snapshot into a NEW data directory:**

```bash
sudo etcdutl snapshot restore /var/backups/etcd/snap-2026-11-18-1015.db \
  --data-dir /var/lib/etcd-restored
```

```
{"level":"info","msg":"restoring snapshot","path":"/var/backups/etcd/snap-2026-11-18-1015.db","wal-dir":"/var/lib/etcd-restored/member/wal","data-dir":"/var/lib/etcd-restored",…}
{"level":"info","msg":"restored snapshot",…}
```

(Older guides use `etcdctl snapshot restore … --data-dir`: same effect, deprecated.) Restoring creates a **fresh, single-member** cluster identity from the snapshot.

**3. Point etcd at the restored data**: edit the moved manifest's **hostPath** for the data volume:

```bash
sudo sed -i 's#path: /var/lib/etcd$#path: /var/lib/etcd-restored#' /etc/kubernetes/manifests-off/etcd.yaml
sudo grep -A2 'name: etcd-data' /etc/kubernetes/manifests-off/etcd.yaml
```

```yaml
  - hostPath:
      path: /var/lib/etcd-restored
      type: DirectoryOrCreate
    name: etcd-data
```

(Alternative: keep the path and swap directories: `mv /var/lib/etcd /var/lib/etcd.broken && mv /var/lib/etcd-restored /var/lib/etcd`. Either way, the `--data-dir` flag inside the container stays `/var/lib/etcd`, because it's the **mount point**.)

**4. Start etcd, then the API server:**

```bash
sudo mv /etc/kubernetes/manifests-off/etcd.yaml /etc/kubernetes/manifests/
sudo crictl ps | grep etcd                                 # running?
sudo etcdctl $ETCD endpoint health
sudo mv /etc/kubernetes/manifests-off/kube-apiserver.yaml /etc/kubernetes/manifests/
```

**5. Let controllers reconcile and verify:**

```bash
kubectl get nodes
kubectl get deploy -A
sudo systemctl restart kubelet          # optional: makes kubelets resync quickly
```

Your deleted objects are back. Objects created **after** the snapshot are gone (and their Pods may be cleaned up by controllers). It's good practice to restart the controller manager and scheduler too (move their manifests out and back), so they rebuild their caches from the restored state rather than from stale watches.

**HA (stacked, 3 members):** restoring means building a **new cluster** from the snapshot: stop the API servers and etcd on **all** control planes, restore the **same snapshot** on each with its own member name and the full initial cluster list:

```bash
sudo etcdutl snapshot restore snap.db --data-dir /var/lib/etcd-restored \
  --name cp1 --initial-cluster cp1=https://10.0.1.10:2380,cp2=https://10.0.1.11:2380,cp3=https://10.0.1.12:2380 \
  --initial-cluster-token etcd-restore-1 --initial-advertise-peer-urls https://10.0.1.10:2380
```

Update each node's manifest, then start etcd everywhere, then the API servers.

### Losing one member in a 3-member cluster (no restore needed)

Quorum still holds (2/3). Replace the failed member:

```bash
sudo etcdctl $ETCD member list
sudo etcdctl $ETCD member remove <failed-member-id>
# on kubeadm, remove the dead control-plane node and join a fresh one:
kubectl delete node cp3
sudo kubeadm join … --control-plane --certificate-key …     # adds a new etcd member automatically
```

### Maintenance

```bash
sudo etcdctl $ETCD defrag --cluster                 # reclaim space (one member at a time in big clusters; brief pause)
sudo etcdctl $ETCD alarm list                       # e.g. "memberID:… alarm:NOSPACE"
sudo etcdctl $ETCD alarm disarm                     # after compaction + defrag freed space
```

Monitor: `etcd_mvcc_db_total_size_in_bytes`, `etcd_server_has_leader`, `etcd_server_leader_changes_seen_total`, `etcd_disk_wal_fsync_duration_seconds` (p99 should be < 10 ms) (topic 80).

### Disaster recovery beyond etcd

| Layer | Tool | What it protects |
|---|---|---|
| Cluster state (everything) | **etcd snapshots** | Full rollback of the cluster: all-or-nothing |
| Selected namespaces/objects **and volume data** | **Velero** (CRs `Backup`, `Schedule`, `Restore`; object storage + CSI snapshots) | Restoring one namespace, migrating apps to another cluster, PV data |
| Desired state | **GitOps** repo (topic 68) | Re-create apps/add-ons on a brand-new cluster |
| Infrastructure | Terraform/eksctl/kubeadm configs | Re-create the cluster itself |
| Data stores | Database-native backups (operators; topic 67) | Application-consistent data |

**Plan with RPO and RTO:**
- **RPO** (Recovery Point Objective): how much data loss is acceptable → backup frequency.
- **RTO** (Recovery Time Objective): how fast you must be back → automation, warm standby.

And **test restores regularly** (a game day): an untested backup isn't a backup. Managed clusters (EKS/GKE/AKS) **don't give you etcd access**: there you rely on Velero/GitOps (and the provider protects etcd itself).

```bash
velero schedule create daily --schedule="0 2 * * *" --include-namespaces shop --ttl 720h
velero restore create --from-backup daily-20261118020000
```

---

## Hands-on lab

On a kubeadm VM cluster (topic 75), or a single-node kind cluster via `docker exec` into the control-plane container (etcd is a static Pod there too):
1. Run `endpoint health/status`, `member list`; browse keys with `--keys-only`.
2. Create Deployments `before-1` and `before-2`; take and verify a snapshot.
3. Create `after-1`; delete `before-1`.
4. Do the full restore; verify `before-1` is back and `after-1` is gone.
5. Turn on encryption at rest (topic 63), create a Secret, and compare its raw value in etcd before/after (`etcdctl get /registry/secrets/default/x | hexdump -C | head`).
6. (Optional) Install Velero with MinIO as storage; back up and restore a single namespace.
7. Write the backup as a CronJob (hostPath to `/etc/kubernetes/pki/etcd` and a backup dir, `nodeSelector` + toleration for the control plane).

---

## Where you'll use it

- **Self-managed clusters**: scheduled, off-site, encrypted etcd snapshots + PKI backups.
- **Before upgrades and risky changes** (topic 76).
- **Incident recovery**: accidental deletions, corruption, quorum loss.
- **CKA**: "back up etcd to /opt/backup.db" and "restore from snapshot": a near-certain exam task.

---

## When it breaks

| Symptom | Cause | Fix |
|---|---|---|
| `etcdctl`: `context deadline exceeded` | Missing/wrong TLS flags or endpoint | Use the cacert/cert/key from `etcd.yaml`; `https://` |
| `etcdctl`: `transport: authentication handshake failed: x509` | Wrong CA or client cert | Use `/etc/kubernetes/pki/etcd/*` |
| API server `CrashLoopBackOff` after restore | etcd not running or not healthy yet | Start etcd first; check `crictl logs` |
| Restored but etcd won't start: `member … has already been bootstrapped` | Reused old data dir/WAL | Restore into an **empty** new directory |
| Cluster writes fail: `etcdserver: mvcc: database space exceeded` | NOSPACE alarm | Compact (API server does), defrag, `alarm disarm`, raise quota, find object bloat (Events, CRs) |
| Frequent leader elections, slow API | Slow disks / network | SSDs, dedicated disks, check fsync latency |
| Restored Secrets unreadable | Encryption key not backed up | Always back up the encryption config/KMS access with etcd |
| Pods "ghosting" after restore | Nodes still running Pods that the restored state doesn't know | Restart kubelets; controllers reconcile; clean up |

---

## Quick recap

- etcd = the cluster's **only** state store (keys under `/registry`, Protobuf values, revisions); Raft with **quorum ⌊n/2⌋+1** (use 3 or 5 members).
- `etcdctl` needs the endpoint + **etcd CA, cert, key** (from `etcd.yaml`).
- **Backup**: `etcdctl snapshot save` → verify with `etcdutl snapshot status` → store off-cluster, encrypted, with PKI and encryption keys.
- **Restore** (kubeadm): stop API server and etcd (move manifests) → `etcdutl snapshot restore --data-dir <new dir>` → point the etcd hostPath at it → start etcd, then the API server → verify.
- Maintain: defrag, alarms, size and fsync latency. DR = etcd + Velero + GitOps + IaC, with RPO/RTO targets and **tested** restores.

### Interview questions

1. **Q: How do you back up and restore etcd on a kubeadm cluster?**
   A: `etcdctl snapshot save` with the etcd CA, cert and key from the etcd manifest, verifying the snapshot; to restore, stop the API server and etcd static Pods, run `etcdutl snapshot restore` into a new data directory, update the etcd manifest's data hostPath, start etcd and then the API server, and verify.
2. **Q: Why do etcd clusters have an odd number of members?**
   A: Writes need a majority (quorum). 3 members tolerate 1 failure, 5 tolerate 2; adding a 4th member doesn't increase fault tolerance but adds overhead and another possible failure.
3. **Q: An etcd snapshot restores the whole cluster. How would you restore just one namespace?**
   A: Use a resource-level backup tool like Velero (or re-apply from GitOps), which backs up selected namespaces and objects (and volume data via CSI snapshots) and can restore them individually.

---

## Appendix: notes on the original course material

This topic was rebuilt from the original CKA course notes. This appendix keeps the audit trail out of your way while you learn.

### Sources

none: a **new topic**. Days 07 and 25 introduce etcd as the cluster's key-value store (topic 07) and Day 55 lists "take an etcd snapshot" as an upgrade pre-check (topic 76); this topic covers how. It's a core **CKA** task.
