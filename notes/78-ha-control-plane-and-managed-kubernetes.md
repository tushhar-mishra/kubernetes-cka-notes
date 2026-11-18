# 78 · Highly Available Control Planes and Managed Kubernetes

> **Level:** ★★★ advanced
>
> **You will learn:** what "highly available" means for each control-plane component; stacked vs external etcd topologies; the load balancer in front of API servers; leader election for the controller manager and scheduler (Leases); building an HA control plane with kubeadm (`--control-plane-endpoint`, `--upload-certs`, joining extra control planes); node and zone resilience; and **managed Kubernetes** (EKS, GKE, AKS): what the provider runs, what you still own (the shared-responsibility model), node groups, auto-upgrades, cost, and how to choose.
>
> **You need:** topics 06–08 (architecture), 46 (topology spread), 75 (kubeadm), 76 (upgrades), 77 (etcd quorum).

---

## ⚡ Core: the 10-minute version

**The idea.** A **highly available** control plane has:
- 3 control-plane nodes in 3 zones;
- a **load balancer** in front of the API servers (all active at once);
- an etcd cluster that keeps **quorum**;
- **leader election** (Lease objects), so only one scheduler and one controller manager act at a time.

kubeadm builds this with `--control-plane-endpoint` and `--upload-certs`, then `kubeadm join --control-plane` on the others. **Managed Kubernetes** (EKS, GKE, AKS) runs all of that for you. You still own the nodes, add-ons, workloads, permissions and timely upgrades.

**Picture it.** A board of three managers: decisions need a majority, a receptionist routes visitors to any of them, and one "on duty" badge says who acts.

**The one thing to know.**

```bash
kubectl -n kube-system get leases          # who leads kube-scheduler / kube-controller-manager
sudo kubeadm init --control-plane-endpoint "k8s-api.example.internal:6443" --upload-certs ...
```

**Try it now:**
1. On a 3-control-plane VM cluster, power off one node: the API keeps working.
2. Power off a second: writes stop, because quorum is lost.
3. On any cluster, watch the Lease holders.

**You've got it if you can:** explain HA for each component, and what you still own on a managed service.

> 📖 **That's the core.** On a first pass, do the mini-lab and move on to the next topic. Everything below is the **full reference**: come back for the details, the edge cases, interview answers and exam-level depth.

---

## What it is

A **highly available (HA)** control plane keeps the Kubernetes API and cluster management working when a control-plane machine, or a whole availability zone, fails. Typically: **3 control-plane nodes** in **3 zones**, a **load balancer** in front of the API servers, and an etcd cluster that keeps **quorum**.

**Managed Kubernetes** (Amazon **EKS**, Google **GKE**, Azure **AKS**, and others) means the cloud provider runs that HA control plane for you, as a service; you run the worker nodes (or not, with serverless options).

---

## Why it exists

With a single control-plane node (topic 75), losing that VM (hardware fault, botched upgrade, full disk) means **no API**: no deployments, no scaling, no self-healing of failed Pods (existing containers keep running, but nothing reacts). For production that's unacceptable. Running HA yourself is real work (etcd care, certificates, upgrades, load balancers), which is why most organisations use managed control planes.

---

## Analogy

A single control plane is a **company with one manager**: if they're off sick, nothing gets approved. HA is a **management board of three**: decisions need a majority (etcd quorum), the receptionist (load balancer) sends visitors to whichever manager is available (any API server), and for each job type only one manager acts at a time by holding the **"on duty" badge** (leader election lease); if they disappear, another picks it up within seconds. Managed Kubernetes is **outsourcing the board** to a professional firm: you still run your departments (nodes and apps).

---

## How it works inside

### Each component in HA

| Component | HA model |
|---|---|
| **kube-apiserver** | **Active-active**: stateless; all instances serve requests behind a **load balancer** (TCP 6443), with health checks on `/readyz` |
| **etcd** | **Raft cluster** (3 or 5 members): survives the loss of a minority (topic 77) |
| **kube-controller-manager**, **kube-scheduler** | **Active-passive** via **leader election**: every instance runs, but only the **leader** acts. Leadership is a **Lease** object in `kube-system` that the leader renews every few seconds; if it stops renewing (default lease 15 s), another instance takes over |
| **cloud-controller-manager** | Leader election, same as above |
| **CoreDNS** | ≥2 replicas spread across nodes/zones (with a PDB) |
| **Nodes / workloads** | Many nodes across zones; topology spread, PDBs, replicas ≥2 (topics 46, 76) |

```bash
kubectl -n kube-system get leases
```

```
NAME                      HOLDER                                        AGE
kube-controller-manager   cp-2_6f1c5b1e-…                               40d
kube-scheduler            cp-1_9a8e2d33-…                               40d
```

The holder names show which control-plane node currently leads each component.

### Topologies

**Stacked etcd** (kubeadm default): each control-plane node runs API server + controller manager + scheduler + an **etcd member**.
- ✅ Fewer machines, simpler; kubeadm automates it.
- ❌ Losing a node loses both an API server **and** an etcd member (coupled failure).
- Needs ≥3 control-plane nodes for etcd quorum.

**External etcd**: etcd runs on its **own** 3 or 5 machines; control-plane nodes run only the stateless components.
- ✅ Failure domains separated; etcd can get dedicated fast disks; scale them independently.
- ❌ Twice the machines; you operate an extra cluster (certificates, backups, upgrades).

### The load balancer and the endpoint

- All clients (kubelets, kube-proxy, controllers, users) reach the API through **one stable address**: the `--control-plane-endpoint` (DNS name or VIP of the load balancer) set at `kubeadm init`. It's in every kubeconfig and in the API server certificate's SANs: choose it **before** the first `init`; changing it later is painful (topic 75).
- Options: a cloud load balancer (AWS NLB, GCP internal LB), **HAProxy + keepalived** (a floating virtual IP on-prem), **kube-vip** (a VIP run as a static Pod on the control planes).
- Health-check `/readyz` on 6443, so an API server that is restarting (e.g. during an upgrade; topic 76) is taken out of rotation.

### Ports between control-plane nodes

TCP **6443** (API), **2379–2380** (etcd client/peer), **10250** (kubelet), **10257** (controller manager), **10259** (scheduler): see topic 75's security groups.

### Zones

Put each control-plane node (and etcd member) in a **different availability zone**: one zone outage then costs one member, and quorum survives. Spread workers and workloads across zones too (`topology.kubernetes.io/zone`, topic 46). Note that a 2-zone setup can't keep a 3-member etcd quorum through the loss of the zone holding two members: use **3 zones**.

---

## Commands / YAML

### Building an HA control plane with kubeadm (stacked etcd)

Prerequisites: 3 control-plane VMs prepared as in topic 75 (Steps 1–3), a load balancer forwarding TCP 6443 to all three (initially only the first will be healthy), DNS `k8s-api.example.internal` → the LB.

**First control plane:**

```bash
sudo kubeadm init \
  --control-plane-endpoint "k8s-api.example.internal:6443" \
  --upload-certs \
  --pod-network-cidr=192.168.0.0/16
```

```
You can now join any number of the control-plane node running the following command on each as root:

  kubeadm join k8s-api.example.internal:6443 --token 9vr73a.a8uxyaju799qwdjv \
	--discovery-token-ca-cert-hash sha256:7c2e69131a36ae2a042a339b33381c6d0d43887e2de83720eff5359e26aec866 \
	--control-plane --certificate-key f8902e114ef118304e561c3ecd4d0b543adc226b7a07f675f56564185ffe0c07

Please note that the certificate-key gives access to cluster sensitive data, keep it secret!
As a safeguard, uploaded-certs will be deleted in two hours; If necessary, you can use
"kubeadm init phase upload-certs --upload-certs" to reload certs afterward.
```

- `--upload-certs` encrypts the shared control-plane certificates (CA, `sa.key`, front-proxy CA, etcd CA) into a Secret (`kubeadm-certs`), so new control planes can download them with the **`--certificate-key`** (valid 2 hours).
- Install the CNI (topic 75 Step 5).

**Second and third control planes** (after Steps 1–3 on them):

```bash
sudo kubeadm join k8s-api.example.internal:6443 --token … \
  --discovery-token-ca-cert-hash sha256:… \
  --control-plane --certificate-key …
```

Each one gets the certificates, writes the static Pods, and **joins etcd as a new member**.

Later (expired key/token):

```bash
sudo kubeadm init phase upload-certs --upload-certs          # prints a new certificate key
kubeadm token create --print-join-command                    # then add --control-plane --certificate-key <key>
```

**Workers** join the same endpoint as usual.

**Verify:**

```bash
kubectl get nodes
kubectl -n kube-system get pods -l tier=control-plane -o wide
kubectl -n kube-system exec etcd-cp-1 -- etcdctl --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key member list -w table
kubectl -n kube-system get leases
```

**Failure test:** power off `cp-1`. `kubectl` keeps working through the LB; the Lease holders move to `cp-2`/`cp-3` within ~15–20 s; etcd keeps quorum (2/3). Power off a second one and the API stops: quorum is lost (topic 77).

### Managed Kubernetes

**What the provider runs:** the API servers, etcd (replicated across zones, backed up), controller manager, scheduler, cloud-controller-manager, control-plane upgrades and patching, control-plane scaling, and the SLA.

**What you still own (shared responsibility):**
- **worker nodes** (or node pools): OS images, patching, upgrades (often a button or auto-upgrade), capacity, autoscaling (Cluster Autoscaler, **Karpenter**; topic 49);
- **add-ons** you install (CNI configuration or choice, CSI drivers, ingress/gateway controllers, monitoring);
- **workloads**, their security and RBAC, network policies, secrets;
- **identity integration** (IAM ↔ Kubernetes RBAC), cluster configuration (private endpoints, logging);
- **upgrading in time**: control-plane versions go out of support (topic 76);
- **backups of your apps** (Velero/GitOps; you **can't** snapshot managed etcd yourself).

| | **EKS** (AWS) | **GKE** (Google Cloud) | **AKS** (Azure) |
|---|---|---|---|
| Control plane | HA across 3 AZs; per-hour fee | Free tier zonal / paid regional HA | Free tier; paid "Standard" tier with SLA |
| Nodes | Managed node groups, self-managed, **Fargate** (serverless Pods), **EKS Auto Mode** (AWS manages nodes, LB, storage, networking add-ons) | Standard node pools or **Autopilot** (Google manages nodes; pay per Pod) | Node pools; **Virtual Nodes** (ACI); **AKS Automatic** |
| Networking | **VPC CNI** (Pods get VPC IPs) | VPC-native (alias IPs), Dataplane V2 (Cilium) | Azure CNI (overlay/pod subnet, Cilium option), kubenet (retiring) |
| Identity | IAM ↔ RBAC via **access entries**; Pod Identity/IRSA for workloads | Google IAM + RBAC; Workload Identity Federation | Entra ID + RBAC; Workload Identity |
| Load balancing | AWS Load Balancer Controller (ALB/NLB) | GKE Ingress/Gateway (Google LBs) | Application Gateway for Containers, Azure LB |
| Upgrades | You trigger (control plane, then node groups); extended support (paid) | **Release channels** with auto-upgrade + maintenance windows | Auto-upgrade channels + planned maintenance |

**Choosing:**
- **Managed** for most production: less toil, built-in HA and integrations; pick by your cloud.
- **Self-managed (kubeadm, Rancher/RKE2, k3s, Talos, OpenShift)** for on-prem, edge, air-gapped, specific compliance or full control, or very large fleets where cost/customisation matter.
- **Serverless node modes** (Fargate, Autopilot) remove node management at the cost of some flexibility (DaemonSets, privileged Pods, special hardware).

### Managed-cluster quick look (EKS)

```bash
eksctl create cluster --name demo --region eu-west-1 --nodes 3 --node-type t3.medium --zones eu-west-1a,eu-west-1b,eu-west-1c
kubectl get nodes -L topology.kubernetes.io/zone
kubectl get pods -n kube-system                 # no apiserver/etcd/scheduler Pods: they're hidden in AWS
aws eks describe-cluster --name demo --query 'cluster.{version:version,endpoint:endpoint,status:status}'
eksctl delete cluster --name demo --region eu-west-1
```

---

## Hands-on lab

1. Build a 3-control-plane kubeadm cluster with stacked etcd behind HAProxy (or kube-vip), using VMs (Multipass/Vagrant work locally).
2. Watch `kubectl -n kube-system get leases -w` while stopping the leading controller manager's node; time the failover.
3. Stop two control-plane nodes; see writes fail (`etcdserver: request timed out`); start one back and recover.
4. Remove and re-add a control-plane node (drain, `kubeadm reset`, `etcdctl member remove`, `kubeadm join --control-plane`).
5. (Optional) Create a small managed cluster on your cloud; compare `kubectl get pods -n kube-system` with kubeadm's; delete it afterwards.

---

## Where you'll use it

- **Production self-managed clusters**: 3+ control planes across zones, LB, etcd quorum, tested failover.
- **Choosing a platform**: managed vs self-managed trade-offs, cost and responsibility.
- **Incident response**: control-plane outages, quorum loss, failover times.
- **CKA**: understanding HA topologies and joining control-plane nodes (`--control-plane`, `--certificate-key`).

---

## When it breaks

| Symptom | Cause | Fix |
|---|---|---|
| `kubectl` intermittently fails | LB sends traffic to an unhealthy API server (no `/readyz` check) | Health-check `/readyz`; check each API server |
| `x509: certificate is valid for … not k8s-api.example.internal` | Endpoint not in the API server certificate SANs | Use the right endpoint at init, or add `certSANs` and regenerate certs |
| Control-plane join fails: `failed to download certificates` / `secret "kubeadm-certs" not found` | Certificate key expired (2 h) | `kubeadm init phase upload-certs --upload-certs` |
| Writes fail cluster-wide after losing nodes | etcd quorum lost | Restore members or rebuild from snapshot (topic 77) |
| Two schedulers acting at once | Leader election disabled/misconfigured | Keep `--leader-elect=true` (default) |
| One-zone outage takes the cluster down | Control planes/etcd not spread over 3 zones | Re-architect: 3 zones |
| Managed cluster "suddenly" upgraded / version unsupported | Auto-upgrade channel or end of support | Maintenance windows; plan upgrades (topic 76) |

---

## Quick recap

- HA control plane: **3+ control-plane nodes in 3 zones**, **load balancer** in front of active-active **API servers** (`/readyz` checks), **etcd quorum** (3 or 5 members), **leader election (Leases)** for controller manager and scheduler.
- Topologies: **stacked** etcd (simpler, coupled failures) vs **external** etcd (separated, more machines).
- kubeadm HA: `--control-plane-endpoint` (stable LB DNS) + `--upload-certs` on the first node; `kubeadm join … --control-plane --certificate-key` on the others.
- **Managed Kubernetes**: the provider runs the HA control plane and etcd; you own nodes (or use Fargate/Autopilot/Auto Mode), add-ons, workloads, IAM/RBAC, timely upgrades and app backups.

### Interview questions

1. **Q: How do kube-controller-manager and kube-scheduler run in an HA control plane?**
   A: An instance runs on each control-plane node, but only one is active: they use leader election through a Lease object in `kube-system`; if the leader stops renewing it, another instance takes over within seconds.
2. **Q: Stacked vs external etcd?**
   A: Stacked runs an etcd member on each control-plane node (simpler, fewer machines, but losing a node loses both an API server and an etcd member); external runs etcd on dedicated machines (separate failure domains, independent scaling, more to operate).
3. **Q: In EKS, what are you responsible for?**
   A: Worker nodes (unless using Fargate/Auto Mode), their OS patching and upgrades, add-ons, workloads and their security, IAM-to-RBAC mapping, network policies, application backups, and upgrading the cluster version before it goes out of support. AWS runs and backs up the control plane and etcd.

---

## Appendix: notes on the original course material

This topic was rebuilt from the original CKA course notes. This appendix keeps the audit trail out of your way while you learn.

### Sources

none: a **new topic**. It expands HA notes elsewhere: the HA ports and HA upgrade callouts in Days 54–55 (topics 75–76), managed-service support windows in Day 55, and the EKS clusters of Days 44 and 50–51 (topics 40, 70–71).
