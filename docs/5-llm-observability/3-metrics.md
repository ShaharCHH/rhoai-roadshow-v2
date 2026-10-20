# Metrics and dashboards

Traces explain one request. Metrics describe the system as a whole.

## 1. How metrics get collected

vLLM exposes a Prometheus-format `/metrics` endpoint. The `uwm` chart enables **User Workload Monitoring** (OpenShift's Prometheus for non-platform namespaces) and creates **PodMonitors** that tell it which pods to scrape.

```bash
oc get podmonitors,servicemonitors -A | grep -i -E 'vllm|llama'
oc get configmap user-workload-monitoring-config -n openshift-user-workload-monitoring
```

> **Where are my monitors?** Do not expect them in `observability-hub`. A `PodMonitor` or `ServiceMonitor` can live in any namespace, so search all namespaces as above. A monitor only scrapes pods and services in **its own namespace** (user workload monitoring ignores `namespaceSelector`), so create it in the model's namespace; on the roadshow cluster that is `llama-serving`. Which Prometheus scrapes it depends on the namespace label: a namespace labelled `openshift.io/cluster-monitoring=true` (as `llama-serving` is) is scraped by the **platform** Prometheus in `openshift-monitoring`, and user workload monitoring skips it. So if the *user-workload* Prometheus shows no `vllm:*` series, query the platform one, or just use Grafana, whose datasource is `thanos-querier` and covers both.

## 2. Open Grafana

```bash
oc get route -n observability-hub | grep -i grafana
```

Open the route and log in with the Grafana admin credentials:

* **Username:** `rhel`
* **Password:** `rhel`

> These are the lab's default Grafana admin credentials (set in the `Grafana` custom resource in `observability-hub`). Change them for anything beyond a workshop.

Then browse the pre-built dashboards. Generate load from the Playground while you watch.

## 3. Metrics worth knowing

| Metric | Meaning | Why it matters |
|--------|---------|----------------|
| Time to first token | Delay before the first word appears | The latency users feel |
| Tokens per second | Generation throughput | Capacity planning |
| Requests running / waiting | Active and queued requests | Waiting > 0 means the server is saturated |
| GPU cache usage | KV-cache fill level | A full cache forces queueing |

> **Tip**: these are the same numbers you measured in the RHAIIS module, now collected continuously instead of from a one-off benchmark.

Click **Langfuse on OpenShift** next.
