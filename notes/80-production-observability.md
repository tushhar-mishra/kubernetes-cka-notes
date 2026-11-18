# 80 · Production Observability: Prometheus, Grafana, Loki, OpenTelemetry, Alerts, SLOs

> **Level:** ★★★ advanced
>
> **You will learn:** how a production observability stack fits together: **Prometheus** (pull-based scraping, service discovery, PromQL, recording/alerting rules) installed with **kube-prometheus-stack** (Prometheus Operator, `ServiceMonitor`/`PodMonitor`, Alertmanager, Grafana, node-exporter, kube-state-metrics); the essential **queries and alerts** for Kubernetes; methods (**USE**, **RED**, golden signals); **SLIs/SLOs/error budgets** and burn-rate alerting; logs with **Loki** (or EFK); traces with **OpenTelemetry** + Tempo/Jaeger; Kubernetes **audit logs** and **Events**; long-term storage and cardinality control.
>
> **You need:** topics 48, 65 (Helm), 66–67 (CRDs/operators), 79.

---

## ⚡ Core: the 10-minute version

**The idea.** Production observability means:
- **Prometheus** scrapes metrics. `kube-prometheus-stack` bundles it with Alertmanager, Grafana, node-exporter and kube-state-metrics.
- **ServiceMonitors** tell it what to scrape.
- **Alerts** fire on user-facing symptoms, ideally **SLO burn rates**, and each links to a runbook.
- **Loki** collects logs; **OpenTelemetry** carries traces to Tempo or Jaeger.
- Don't forget Kubernetes **Events** and API **audit logs**.

Measure resources with **USE** (utilisation, saturation, errors) and services with **RED** (rate, errors, duration).

**Picture it.** An airport control tower: radar (metrics), radio logs, flight plans (traces), and alarms only for real danger.

**The one thing to know.**

```promql
sum(rate(http_requests_total{job="api",code=~"5.."}[5m])) / sum(rate(http_requests_total{job="api"}[5m]))   # error ratio
histogram_quantile(0.99, sum by (le) (rate(http_request_duration_seconds_bucket{job="api"}[5m])))           # p99 latency
```

**Try it now:**
1. `helm install kps prometheus-community/kube-prometheus-stack -n monitoring --create-namespace`
2. Port-forward Grafana and open the Kubernetes dashboards.
3. Add a ServiceMonitor for a sample app.

**You've got it if you can:** explain USE vs RED, and why you alert on SLO burn rather than CPU.

> 📖 **That's the core.** On a first pass, do the mini-lab and move on to the next topic. Everything below is the **full reference**: come back for the details, the edge cases, interview answers and exam-level depth.

---

## What it is

A **production observability stack** collects, stores, queries, visualises and **alerts** on all three signals across the cluster and its applications:

```
             ┌─────────────── Grafana (dashboards, explore, correlate) ───────────────┐
             │                        │                          │                    │
        Prometheus  ◄── scrape ── exporters (node, KSM,     Loki ◄── Alloy/Fluent Bit   Tempo/Jaeger ◄── OTel Collector ◄── apps (OTel SDKs)
          │  rules                kubelet/cAdvisor, apps)        (DaemonSet: Pod logs    (traces)
          ▼                                                         + journald)
      Alertmanager ──► Slack / PagerDuty / email
          │
   (remote_write) ──► long-term store (Thanos / Mimir / AMP / Grafana Cloud)
```

---

## Why it exists

Topic 79's tools give **signals**; production needs **history** (what was normal last week?), **alerts** (wake someone only when users are affected), **correlation** (from a latency spike → the trace → the logs of the slow Pod), and **objectives** (are we reliable enough?). Without them, incidents are noticed by customers first and debugged by guesswork.

---

## Analogy

An airport control tower: **radar** (metrics) shows every plane's position and speed over time; the **radio log** (logs) records every message; **flight plans** (traces) show each plane's full route and where it was held; **alarms** (Alertmanager) sound only for real dangers, routed to the right controller; and the **on-time target** ("99.5% of flights within 15 minutes", an SLO) decides whether there's room for risky changes this month.

---

## How it works inside

### Prometheus

- **Pull model**: Prometheus **scrapes** HTTP `/metrics` endpoints every 15–60 s. Each sample: `metric_name{label="value",…} value timestamp`.
- **Metric types**: **counter** (only goes up: requests total; use `rate()`), **gauge** (up/down: memory in use), **histogram** (buckets: latency distribution → percentiles with `histogram_quantile`), **summary**.
- **Service discovery**: it asks the Kubernetes API for Pods/Services/Endpoints/Nodes and scrapes them automatically, adding labels like `namespace`, `pod`, `container`.
- **PromQL**: the query language. **Recording rules** precompute expensive queries; **alerting rules** fire when an expression stays true for a duration (`for: 10m`).
- **Storage**: local TSDB, usually days to weeks. For months/years and many clusters: **remote_write** to Thanos, Mimir, Amazon Managed Prometheus, Grafana Cloud.
- **Cardinality**: every unique label combination is a separate time series. Labels like user IDs, request IDs or full URLs explode memory. Keep labels bounded.

### kube-prometheus-stack

The standard Helm chart bundles:
- **Prometheus Operator** with CRDs: `Prometheus`, `Alertmanager`, **`ServiceMonitor`** (scrape the Endpoints behind a Service), **`PodMonitor`**, **`PrometheusRule`** (alert/recording rules), `AlertmanagerConfig`;
- **Prometheus**, **Alertmanager**, **Grafana** (with ready-made Kubernetes dashboards);
- **node-exporter** (DaemonSet, host metrics), **kube-state-metrics** (object state);
- default **alerts** for Kubernetes and node health (from the kubernetes-mixin project).

### Alertmanager

Receives firing alerts from Prometheus and **groups** (one notification for 50 Pods failing the same way), **deduplicates**, **silences** (during maintenance), **inhibits** (suppress "Pod down" when "node down" fires), and **routes** by labels (`severity`, `team`) to receivers (PagerDuty, Slack, Opsgenie, email, webhooks).

### What to measure: methods

- **USE** (for **resources**: nodes, disks, network): **U**tilisation, **S**aturation (queueing, throttling, pressure), **E**rrors.
- **RED** (for **services**): **R**ate (requests/s), **E**rrors (failed/s), **D**uration (latency distribution).
- **Four golden signals** (Google SRE): latency, traffic, errors, saturation.

### SLIs, SLOs and error budgets

- **SLI** (indicator): a measured ratio, e.g. "fraction of requests served successfully in < 300 ms".
- **SLO** (objective): the target over a window, e.g. **99.9% over 30 days**.
- **Error budget**: 100% − SLO = 0.1% ≈ **43 minutes** of failure per 30 days. Budget left → ship features; budget burnt → focus on reliability.
- **Burn-rate alerts**: page when the budget is being consumed fast (e.g. 14.4× the sustainable rate over 1 h **and** 5 min), ticket on slow burns. This alerts on **user impact**, not on every CPU blip. Tools like **Pyrra** or **Sloth** generate these rules.
- Alert on **symptoms** (users see errors/latency) rather than causes (CPU high); use cause metrics for **diagnosis**.

### Logs at scale

- **Loki**: indexes only **labels** (namespace, app, Pod), stores compressed log chunks in object storage: cheap. Query with **LogQL** (`{namespace="shop", app="api"} |= "error" | json | status >= 500`). Agent: **Grafana Alloy** (or Fluent Bit) as a DaemonSet.
- **EFK/OpenSearch**: full-text indexing, powerful search, more expensive to run.
- Use **structured (JSON) logs** with a **trace ID** field, so you can jump between logs and traces.

### Traces with OpenTelemetry

- Apps are instrumented with **OpenTelemetry SDKs** (or auto-instrumentation: the **OpenTelemetry Operator** can inject agents into Java/Python/Node.js/.NET Pods via annotations).
- Each request carries a **trace context** (`traceparent` header, W3C standard) across services; each service records **spans** (timed operations).
- The **OpenTelemetry Collector** (Deployment or DaemonSet) receives OTLP data, batches/filters/samples it, adds Kubernetes metadata, and exports to **Tempo**, **Jaeger** or a vendor.
- **Exemplars** link a latency histogram bucket in Prometheus to a specific trace in Grafana.

### Kubernetes-native signals you shouldn't forget

- **Events** (`kubectl get events`): scheduling failures, image pulls, probe failures, evictions. Stored only ~1 hour by default, so export them (e.g. with an event exporter or the OTel `k8sobjects` receiver) to your log system.
- **Audit logs**: the API server can record **who did what** (user, verb, resource, response) according to an **audit policy** (`--audit-policy-file`, `--audit-log-path`); essential for security and incident forensics. Managed services expose them (EKS control-plane logging to CloudWatch, GKE Cloud Audit Logs, AKS diagnostic settings).
- **Control-plane metrics**: API server latency/errors (`apiserver_request_duration_seconds`), etcd (`etcd_disk_wal_fsync_duration_seconds`, leader changes; topic 77), scheduler pending Pods, CoreDNS errors/latency.

---

## Commands / YAML

### Install kube-prometheus-stack

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts && helm repo update
helm install kps prometheus-community/kube-prometheus-stack -n monitoring --create-namespace \
  --set grafana.adminPassword='change-me' \
  --set prometheus.prometheusSpec.retention=7d
kubectl -n monitoring get pods
kubectl -n monitoring port-forward svc/kps-grafana 3000:80        # http://localhost:3000 (admin / change-me)
kubectl -n monitoring port-forward svc/kps-kube-prometheus-stack-prometheus 9090:9090
```

On kubeadm/kind, some control-plane targets (etcd, scheduler, controller manager) show **down** because they listen on `127.0.0.1` only: bind their metrics to `0.0.0.0` (`--bind-address`, `--listen-metrics-urls`) in lab clusters, or disable those scrapes.

### Scrape your app: ServiceMonitor

```yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: shop-api
  namespace: shop
  labels: { release: kps }              # kube-prometheus-stack selects ServiceMonitors with this label by default
spec:
  selector: { matchLabels: { app: shop-api } }   # the Service's labels
  endpoints:
  - port: metrics                        # NAMED port on the Service
    interval: 30s
    path: /metrics
```

### Essential PromQL

```promql
# CPU used per Pod (cores)
sum by (namespace, pod) (rate(container_cpu_usage_seconds_total{container!=""}[5m]))

# Memory working set vs limit (the kill threshold, topic 41)
sum by (namespace, pod, container) (container_memory_working_set_bytes{container!=""})
  / on (namespace, pod, container) group_left
  kube_pod_container_resource_limits{resource="memory"}

# CPU throttling ratio (limits too low)
sum by (namespace, pod) (rate(container_cpu_cfs_throttled_periods_total[5m]))
  / sum by (namespace, pod) (rate(container_cpu_cfs_periods_total[5m]))

# Pods restarting
increase(kube_pod_container_status_restarts_total[1h]) > 3

# Deployments not fully available
kube_deployment_status_replicas_available < kube_deployment_spec_replicas

# RED for an HTTP service
sum(rate(http_requests_total{job="shop-api"}[5m]))                                   # rate
sum(rate(http_requests_total{job="shop-api",code=~"5.."}[5m]))
  / sum(rate(http_requests_total{job="shop-api"}[5m]))                               # error ratio
histogram_quantile(0.99, sum by (le) (rate(http_request_duration_seconds_bucket{job="shop-api"}[5m])))   # p99 latency

# API server p99 latency by verb (excluding long-running WATCH)
histogram_quantile(0.99, sum by (le, verb) (rate(apiserver_request_duration_seconds_bucket{verb!~"WATCH|CONNECT"}[5m])))
```

### An alert rule

```yaml
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: shop-api-alerts
  namespace: shop
  labels: { release: kps }
spec:
  groups:
  - name: shop-api
    rules:
    - alert: ShopApiHighErrorRate
      expr: |
        sum(rate(http_requests_total{job="shop-api",code=~"5.."}[5m]))
          / sum(rate(http_requests_total{job="shop-api"}[5m])) > 0.02
      for: 10m
      labels: { severity: page, team: shop }
      annotations:
        summary: "shop-api 5xx ratio above 2% for 10m"
        runbook_url: https://runbooks.example.com/shop-api/high-errors
```

Every paging alert should have a **runbook** (topic 81).

### Loki + Alloy for logs

```bash
helm repo add grafana https://grafana.github.io/helm-charts
helm install loki grafana/loki -n monitoring -f loki-single-binary-values.yaml     # small single-binary mode for labs
helm install alloy grafana/alloy -n monitoring -f alloy-k8s-logs-values.yaml        # DaemonSet tailing Pod logs
```

```logql
{namespace="shop", app="shop-api"} |= "timeout"
sum by (pod) (count_over_time({namespace="shop"} |= "ERROR" [5m]))
```

### OpenTelemetry Collector (traces → Tempo)

```yaml
apiVersion: opentelemetry.io/v1beta1
kind: OpenTelemetryCollector
metadata: { name: otel, namespace: monitoring }
spec:
  mode: deployment
  config:
    receivers:
      otlp: { protocols: { grpc: {}, http: {} } }
    processors:
      batch: {}
      k8sattributes: {}                    # add namespace/pod/deployment to spans
    exporters:
      otlp/tempo: { endpoint: tempo.monitoring:4317, tls: { insecure: true } }
    service:
      pipelines:
        traces: { receivers: [otlp], processors: [k8sattributes, batch], exporters: [otlp/tempo] }
```

Apps send to `otel-collector.monitoring:4317` (env `OTEL_EXPORTER_OTLP_ENDPOINT`).

### A minimal audit policy (kubeadm)

```yaml
apiVersion: audit.k8s.io/v1
kind: Policy
rules:
- level: None                                  # skip noisy reads by system components
  users: ["system:kube-proxy"]
  verbs: ["watch"]
- level: Metadata                              # never log Secret bodies
  resources: [{ group: "", resources: ["secrets", "configmaps"] }]
- level: RequestResponse
  verbs: ["create", "update", "patch", "delete"]
- level: Metadata                              # everything else: who/what/when
```

API server flags: `--audit-policy-file=/etc/kubernetes/audit-policy.yaml --audit-log-path=/var/log/kubernetes/audit.log --audit-log-maxage=7 --audit-log-maxbackup=5 --audit-log-maxsize=100` (plus hostPath mounts in the static Pod manifest).

```bash
sudo tail -1 /var/log/kubernetes/audit.log | jq '{user: .user.username, verb, uri: .requestURI, code: .responseStatus.code}'
```

---

## Hands-on lab

1. Install kube-prometheus-stack on kind; open Grafana's "Kubernetes / Compute Resources / Namespace (Pods)" dashboard.
2. Deploy a sample app exposing metrics (e.g. `podinfo`), add a ServiceMonitor, and find its metrics in Prometheus (Status → Targets).
3. Write the RED queries for it; create a PrometheusRule; trigger it (send errors with `curl …/status/500` in a loop) and see it in Alertmanager.
4. Install Loki + Alloy; query the app's logs; build a panel combining error rate and error logs.
5. Instrument podinfo-style tracing (or run the OpenTelemetry demo app), send traces through the Collector to Tempo, and jump from a slow request to its trace.
6. Enable audit logging on a kubeadm cluster and find who deleted a test Deployment.
7. Define an SLO (99% of requests < 300 ms over 7 days) and compute the error budget with PromQL.

---

## Where you'll use it

- **Every production platform**: dashboards per team, alerts with runbooks, on-call.
- **Capacity planning** (USE metrics, requests vs usage), **cost optimisation**.
- **Incident response and postmortems** (topic 81, 84).
- **Security**: audit logs and Events in the SIEM (Security Information and Event Management system).

---

## When it breaks

| Symptom | Cause | Fix |
|---|---|---|
| ServiceMonitor ignored | Missing the label the Prometheus CR selects (`release: kps`), wrong namespace selector, unnamed port | Check `prometheus.spec.serviceMonitorSelector`; name the port |
| Target `down`: `connection refused` | Metrics bound to localhost (control plane), NetworkPolicy blocking | Bind address; allow monitoring namespace |
| Prometheus OOMKilled | High cardinality, long retention, too many targets | Drop labels/metrics (relabelling), shard, remote_write |
| Alert storm | Alerting on causes, no grouping/inhibition | Symptom-based SLO alerts; Alertmanager grouping/inhibit rules |
| Alerts never arrive | Alertmanager route/receiver misconfigured | `amtool check-config`; test with a dummy alert |
| Logs missing for some Pods | Agent not on every node (taints), parse errors | Tolerations on the DaemonSet; check agent logs |
| Traces broken into pieces | Context not propagated (a proxy strips headers, uninstrumented hop) | Propagate `traceparent`; mesh/ingress tracing config |
| Audit log fills the disk | Policy too verbose (RequestResponse on everything) | Tune the policy; rotation flags; ship off-node |

---

## Quick recap

- **Prometheus** scrapes and stores metrics (counters/gauges/histograms, PromQL, rules); **kube-prometheus-stack** packages it with the Operator (`ServiceMonitor`, `PrometheusRule`), Alertmanager, Grafana, node-exporter, KSM.
- Measure with **USE** (resources) and **RED**/golden signals (services); alert on **SLO burn rates** (symptoms), with runbooks; diagnose with cause metrics.
- **Logs**: Loki (labels, cheap) or EFK (full-text) via a DaemonSet agent; structured logs with trace IDs.
- **Traces**: OpenTelemetry SDKs/auto-instrumentation → Collector → Tempo/Jaeger.
- Don't forget **Events** (export them) and **audit logs** (who did what). Control cardinality; use remote storage for history.

### Interview questions

1. **Q: How would you monitor a Kubernetes cluster in production?**
   A: kube-prometheus-stack (Prometheus with node-exporter, kube-state-metrics, kubelet/cAdvisor and control-plane metrics, Alertmanager, Grafana), ServiceMonitors for apps, SLO-based alerting with runbooks, centralised logs (Loki/EFK), traces via OpenTelemetry, exported Events and API audit logs, with remote long-term storage.
2. **Q: What's the difference between the USE and RED methods?**
   A: USE (utilisation, saturation, errors) is for resources like nodes, disks and network; RED (rate, errors, duration) is for request-driven services. Together they cover infrastructure and user-facing health.
3. **Q: What's an error budget and how does it guide alerting?**
   A: The allowed unreliability implied by an SLO (e.g. 0.1% of requests over 30 days for 99.9%). You page on fast budget burn (user-impacting), ticket on slow burn, and use the remaining budget to balance feature velocity and reliability work.

---

## Appendix: notes on the original course material

This topic was rebuilt from the original CKA course notes. This appendix keeps the audit trail out of your way while you learn.

### Sources

none: a **new topic**. It continues Day 56 (topic 79), which introduced the tools (Prometheus, Grafana, Loki, EFK, OpenTelemetry, Jaeger, Tempo, kube-state-metrics, component `/metrics`) without building a production stack.
