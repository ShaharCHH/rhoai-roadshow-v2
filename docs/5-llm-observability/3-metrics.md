# Metrics and dashboards

Traces explain one request. Metrics describe the system as a whole.

## 1. How metrics get collected

vLLM exposes a Prometheus-format `/metrics` endpoint. The `uwm` chart enables **User Workload Monitoring** (OpenShift's Prometheus for non-platform namespaces) and creates **PodMonitors** that tell it which pods to scrape.

```bash
oc get podmonitors,servicemonitors -A | grep -i -E 'vllm|llama'
oc get configmap user-workload-monitoring-config -n openshift-user-workload-monitoring
```

> **Where are my monitors?** Do not expect them in `observability-hub`. A `PodMonitor` or `ServiceMonitor` can live in any namespace, so search all namespaces as above. In OpenShift **user workload monitoring** a monitor only scrapes pods and services in **its own namespace**: the `namespaceSelector` field is ignored (`oc get prometheus user-workload -n openshift-user-workload-monitoring -o jsonpath='{.spec.ignoreNamespaceSelectors}'` prints `true`). Namespaces labelled `openshift.io/cluster-monitoring=true` are skipped too. On the roadshow cluster the model runs in `llama-serving`, so its monitor must be created in `llama-serving`. A monitor anywhere else shows up in the list but scrapes nothing.

## 2. Open Grafana

```bash
oc get route -n observability-hub | grep -i grafana
```

Open the route and browse the pre-built dashboards. Generate load from the Playground while you watch.

## 3. Metrics worth knowing

| Metric | Meaning | Why it matters |
|--------|---------|----------------|
| Time to first token | Delay before the first word appears | The latency users feel |
| Tokens per second | Generation throughput | Capacity planning |
| Requests running / waiting | Active and queued requests | Waiting > 0 means the server is saturated |
| GPU cache usage | KV-cache fill level | A full cache forces queueing |

> **Tip**: these are the same numbers you measured in the RHAIIS module, now collected continuously instead of from a one-off benchmark.

Click **Langfuse on OpenShift** next.
