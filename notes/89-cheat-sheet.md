# 89 · Kubernetes Cheat Sheet

> **Level:** ★ reference
>
> **You will learn:** nothing new: this is a **one-page-style reference** of the commands, YAML snippets, file paths, ports and troubleshooting moves used throughout the course, each with the topic where it's explained.
>
> **You need:** the topics referenced.

---

## ⚡ Core: the 10-minute version

**The idea.** This is a one-page-style reference of the commands, YAML snippets, file paths, ports, defaults and "status → first move" moves used throughout the course. It isn't for reading top to bottom: keep it open while you work.

**Picture it.** The laminated quick-reference card next to a machine.

**The one thing to know.** The five sections you'll use most:
- **Setup**;
- **Create (generators)**;
- **Inspect and debug**;
- **Nodes and control plane**;
- **Status → first move**.

**Try it now:** close this page, write your own cheat sheet from memory, then compare it with this one. What you forgot is what to practise next.

**You've got it if you can:** solve common tasks without searching for the syntax.

> 📖 **That's the core.** On a first pass, do the mini-lab and move on to the next topic. Everything below is the **full reference**: come back for the details, the edge cases, interview answers and exam-level depth.

---

## What it is

A quick-reference card: keep it open in a second window while working.

## Why it exists

Recall under pressure (on-call, the CKA) is easier from a compact list than from 88 topics.

## Analogy

The laminated quick-reference card next to a machine: not the manual, just what you reach for every day.

## How it works inside

Grouped by task. `k` = `kubectl` (`alias k=kubectl`). `$do` = `--dry-run=client -o yaml`.

---

## Commands / YAML

### Setup and context (topics 11–12, 52)

```bash
alias k=kubectl; export do="--dry-run=client -o yaml"
source <(kubectl completion bash); complete -o default -F __start_kubectl k
kind create cluster --name lab --config kind.yaml; kind get clusters; kind delete cluster --name lab
k config get-contexts; k config use-context <ctx>; k config current-context
k config set-context --current --namespace=<ns>
k config view --minify --raw
k cluster-info; k version; k get --raw='/readyz?verbose'
```

### Discover and explain (topics 13–14, 16)

```bash
k api-resources [--namespaced=true] [-o wide]; k api-versions
k explain deploy.spec.strategy.rollingUpdate [--recursive]
k get <res> -o wide | -o yaml | -o json | -o name
k get pods -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.status.podIP}{"\n"}{end}'
k get pods -o custom-columns=NAME:.metadata.name,NODE:.spec.nodeName,STATUS:.status.phase
k get pods --sort-by=.status.startTime; k get pods --field-selector status.phase=Pending -A
k get pods -l 'app=web,env in (dev,qa)' --show-labels
k get events -A --sort-by=.lastTimestamp
```

### Create (generators) (topics 15, 18, 24–28, 30–31, 34–35)

```bash
k run nginx --image=nginx --port=80 --labels=app=web $do
k create deploy web --image=nginx --replicas=3 $do
k expose deploy web --port=80 --target-port=8080 [--type=NodePort] $do
k create svc nodeport web --tcp=80:8080 --node-port=30080 $do
k create cm cfg --from-literal=K=V --from-file=app.conf --from-env-file=.env $do
k create secret generic s --from-literal=user=admin $do
k create secret tls t --cert=tls.crt --key=tls.key
k create secret docker-registry reg --docker-server=… --docker-username=… --docker-password=…
k create job j --image=busybox -- sh -c 'echo hi' $do
k create cronjob c --image=busybox --schedule='*/5 * * * *' -- date $do
k create ns dev; k create sa app -n dev
k create quota q --hard=pods=10,requests.cpu=2,requests.memory=4Gi -n dev
k create ingress web --class=traefik --rule="host.example.com/path*=svc:80"
```

### Change and roll out (topics 15, 25, 48)

```bash
k apply -f file.yaml | -k overlay/ ; k diff -f file.yaml
k edit deploy web; k patch deploy web -p '{"spec":{"replicas":4}}'
k set image deploy/web nginx=nginx:1.27; k set env deploy/web MODE=prod; k set resources deploy/web --requests=cpu=100m,memory=128Mi
k scale deploy web --replicas=5; k autoscale deploy web --min=2 --max=10 --cpu-percent=70
k rollout status|history|undo|restart|pause|resume deploy/web
k replace --force -f pod.yaml
k delete pod x --force --grace-period=0
k label node n1 disktype=ssd; k label pod x app-
k annotate deploy web kubernetes.io/change-cause="v2"
k taint node n1 dedicated=gpu:NoSchedule; k taint node n1 dedicated=gpu:NoSchedule-
k cordon|uncordon n1; k drain n1 --ignore-daemonsets --delete-emptydir-data
```

### Inspect and debug (topics 19–22, 79, 81)

```bash
k describe pod x                                   # Events at the bottom
k logs x [-c c] [-f] [--previous] [--since=10m] [--tail=50]
k logs -l app=web --all-containers --prefix
k exec -it x [-c c] -- sh
k debug -it x --image=busybox:1.37 --target=<container>
k debug node/n1 -it --image=ubuntu:24.04
k port-forward svc/web 8080:80
k run tmp --rm -it --image=busybox:1.37 --restart=Never -- sh     # nslookup / wget tests
k top nodes; k top pods -A --sort-by=memory
k get pod x -o jsonpath='{.status.containerStatuses[0].lastState}'
```

### RBAC and auth (topics 54–57)

```bash
k auth whoami
k auth can-i create deploy -n dev [--as=jane | --as=system:serviceaccount:dev:app]
k auth can-i --list -n dev --as=jane
k create role r --verb=get,list,watch --resource=pods,pods/log -n dev
k create clusterrole cr --verb=get,list --resource=nodes
k create rolebinding rb --role=r --user=jane --group=devs --serviceaccount=dev:app -n dev
k create clusterrolebinding crb --clusterrole=cr --user=jane
k create token app -n dev --duration=1h
# CSR flow: openssl genrsa -out u.key 2048; openssl req -new -key u.key -subj "/CN=jane/O=devs" -out u.csr
#  → CertificateSigningRequest (signerName kubernetes.io/kube-apiserver-client, usages [client auth])
k certificate approve jane; k get csr jane -o jsonpath='{.status.certificate}' | base64 -d > u.crt
```

### Networking (topics 29–33, 61, 69–73)

```bash
k get svc,endpointslices -n ns
k get endpointslices -l kubernetes.io/service-name=web
k get netpol -A; k describe netpol x
k get ingress,ingressclass -A
k get gatewayclass,gateway,httproute -A; k describe httproute r    # status: Accepted/ResolvedRefs
nslookup web.shop.svc.cluster.local; wget -qO- -T 3 http://web.shop
k -n kube-system get pods -l k8s-app=kube-dns; k -n kube-system logs -l k8s-app=kube-dns
```

NetworkPolicy essentials:

```yaml
spec:
  podSelector: { matchLabels: { app: db } }
  policyTypes: [Ingress, Egress]
  ingress:
  - from:
    - namespaceSelector: { matchLabels: { kubernetes.io/metadata.name: shop } }
      podSelector: { matchLabels: { app: api } }        # same item = AND
    ports: [{ protocol: TCP, port: 5432 }]
  egress:
  - to:
    - namespaceSelector: { matchLabels: { kubernetes.io/metadata.name: kube-system } }
      podSelector: { matchLabels: { k8s-app: kube-dns } }
    ports: [{ protocol: UDP, port: 53 }, { protocol: TCP, port: 53 }]
```

### Storage (topics 37–40)

```bash
k get sc,pv,pvc -A; k describe pvc data
k annotate sc standard storageclass.kubernetes.io/is-default-class=true --overwrite
k patch pvc data -p '{"spec":{"resources":{"requests":{"storage":"20Gi"}}}}'     # expansion
k get volumeattachments; k get csidrivers
```

### Scheduling snippets (topics 41–47)

```yaml
resources: { requests: { cpu: 100m, memory: 128Mi }, limits: { memory: 256Mi } }
nodeSelector: { disktype: ssd }
tolerations: [{ key: dedicated, operator: Equal, value: gpu, effect: NoSchedule }]
affinity:
  nodeAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      nodeSelectorTerms: [{ matchExpressions: [{ key: zone, operator: In, values: [a, b] }] }]
topologySpreadConstraints:
- { maxSkew: 1, topologyKey: topology.kubernetes.io/zone, whenUnsatisfiable: DoNotSchedule, labelSelector: { matchLabels: { app: web } } }
priorityClassName: high
```

### Probes and security snippets (topics 22, 59–60)

```yaml
startupProbe:   { httpGet: { path: /healthz, port: 8080 }, failureThreshold: 30, periodSeconds: 5 }
readinessProbe: { httpGet: { path: /ready,   port: 8080 }, periodSeconds: 5 }
livenessProbe:  { httpGet: { path: /healthz, port: 8080 }, periodSeconds: 10 }
securityContext:                                  # Pod level
  runAsNonRoot: true
  runAsUser: 10001
  fsGroup: 10001
  seccompProfile: { type: RuntimeDefault }
containers:
- securityContext:                                # container level
    allowPrivilegeEscalation: false
    readOnlyRootFilesystem: true
    capabilities: { drop: [ALL] }
```

```bash
k label ns prod pod-security.kubernetes.io/enforce=restricted pod-security.kubernetes.io/warn=restricted
```

### Packaging (topics 64–68)

```bash
k kustomize overlays/prod; k apply -k overlays/prod; kustomize edit fix
helm repo add|update; helm search repo|hub x; helm show values repo/chart
helm install rel repo/chart -n ns --create-namespace -f values.yaml [--set k=v] [--version x]
helm upgrade --install rel ./chart -n ns -f values-prod.yaml --atomic
helm list -A; helm history rel -n ns; helm rollback rel 1 -n ns; helm uninstall rel -n ns
helm template rel ./chart -f v.yaml; helm lint ./chart; helm get values|manifest rel -n ns
k get crd; k api-resources --api-group=<group>
```

### Nodes and control plane (topics 42, 75–78, 82–83)

```bash
# files
/etc/kubernetes/manifests/          # static Pods (apiserver, controller-manager, scheduler, etcd)
/etc/kubernetes/pki/                # cluster CA and certs; pki/etcd/ for etcd
/etc/kubernetes/{admin,super-admin,controller-manager,scheduler,kubelet}.conf
/var/lib/kubelet/config.yaml        # kubelet config (staticPodPath, cgroupDriver…)
/var/lib/etcd                       # etcd data (kubeadm)
/etc/cni/net.d/  /opt/cni/bin/      # CNI config and binaries
/var/log/pods/  /var/log/containers/ # container logs
/etc/containerd/config.toml          # containerd (SystemdCgroup, sandbox image)

# services and runtime
systemctl status|restart kubelet containerd; journalctl -u kubelet -f
crictl ps -a; crictl logs <id>; crictl pods; crictl images; crictl inspect <id>; crictl rmi --prune
ss -tlpn | grep -E '6443|2379|10250|10257|10259'

# kubeadm
kubeadm init --control-plane-endpoint=<lb>:6443 --pod-network-cidr=… [--upload-certs]
kubeadm token create --print-join-command
kubeadm join <ep>:6443 --token … --discovery-token-ca-cert-hash sha256:… [--control-plane --certificate-key …]
kubeadm upgrade plan; kubeadm upgrade apply vX.Y.Z; kubeadm upgrade node
kubeadm certs check-expiration; kubeadm certs renew all     # then restart control-plane static Pods
kubeadm reset -f

# packages (Debian/Ubuntu, pkgs.k8s.io per-minor repo)
apt-mark unhold kubeadm kubelet kubectl; apt-get install -y kubeadm=1.37.x-1.1; apt-mark hold kubeadm kubelet kubectl
apt-cache madison kubeadm

# etcd
ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt --key=/etc/kubernetes/pki/etcd/server.key \
  snapshot save /opt/snap.db | endpoint health | endpoint status -w table | member list -w table
etcdutl snapshot status /opt/snap.db -w table
etcdutl snapshot restore /opt/snap.db --data-dir /var/lib/etcd-restored    # point etcd.yaml hostPath here
```

### Ports

| Port | What |
|---|---|
| 6443 | kube-apiserver |
| 2379 / 2380 | etcd client / peer |
| 10250 | kubelet API |
| 10257 / 10259 | controller manager / scheduler (localhost) |
| 10249 / 10256 | kube-proxy metrics / health |
| 30000–32767 | NodePort range |
| 53 UDP/TCP | DNS (CoreDNS) |
| 4789 UDP / 179 TCP / 5473 TCP | VXLAN / BGP / Calico Typha |

### Defaults worth remembering

| Item | Default |
|---|---|
| Service CIDR (kubeadm) | `10.96.0.0/12` (kubernetes Service = `10.96.0.1`, DNS = `10.96.0.10`) |
| Grace period | 30 s |
| Max Pods per node | 110 |
| Container log rotation | 10 MiB × 5 files |
| kubeadm certs / CA | 1 year / 10 years |
| Bootstrap token TTL | 24 h |
| Not-ready/unreachable eviction | 300 s tolerations |
| Deployment strategy | RollingUpdate, 25% surge / 25% unavailable |
| Revision history | 10 (Deployments), 10 (Helm `--history-max`) |
| Projected SA token | ~1 h, auto-rotated |

### Status → first move (topics 81–83)

| Status | First move |
|---|---|
| Pending | `describe` → FailedScheduling reason (resources/taints/affinity/PVC) |
| ContainerCreating | `describe` → volumes/CNI/image |
| ErrImagePull / ImagePullBackOff | Image name/tag, registry auth |
| CreateContainerConfigError | Missing ConfigMap/Secret/key |
| CrashLoopBackOff | `logs --previous`, exit code |
| OOMKilled (137) | Memory limit vs usage |
| Running, not Ready | Readiness probe |
| Service unreachable | EndpointSlices, ports, NetworkPolicy, DNS |
| Node NotReady | kubelet/containerd status and logs |
| `kubectl` refused | API server container on the control plane |
| `x509 … expired` | `kubeadm certs check-expiration` (and the clock) |

---

## Hands-on lab

Recreate the cheat sheet from memory once a week for a month, then compare with this page; whatever you forgot is what to practise.

## Where you'll use it

Daily operations, on-call, and as the last-day review before the CKA.

## When it breaks

Commands and defaults change between versions: when something here doesn't work on your cluster, check `kubectl version`, `kubectl explain` and the docs for your release.

## Quick recap

Contexts → discover/explain → generate → apply/roll out → inspect/debug → RBAC → networking → storage → scheduling/security snippets → packaging → nodes/control plane/etcd → ports/defaults → status → first move.

### Interview questions

1. **Q: Quickest way to get a Pod manifest skeleton?**
   A: `kubectl run x --image=nginx --dry-run=client -o yaml > pod.yaml`.
2. **Q: How do you see the previous container's logs after a crash?**
   A: `kubectl logs <pod> -c <container> --previous`.
3. **Q: How do you find which node a Pod runs on and its IP?**
   A: `kubectl get pod <pod> -o wide`.

---

## Appendix: notes on the original course material

This topic was rebuilt from the original CKA course notes. This appendix keeps the audit trail out of your way while you learn.

### Sources

none: a **new topic** collecting commands from all Day folders (e.g. Day 08 kind, Day 09–10 Pod/Deployment commands, Day 13 imperative commands, Day 22 debugging, Day 32 kubeconfig, Day 34 CSR flow, Day 36 RBAC checks, Day 54–55 kubeadm, Day 56 logging, Day 57–58 troubleshooting, Day 59 output formatting).
