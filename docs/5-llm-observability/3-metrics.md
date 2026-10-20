# Metrics and dashboards

Traces explain one request. Metrics describe the system as a whole.

## 1. How metrics get collected

vLLM exposes a Prometheus-format `/metrics` endpoint. The `uwm` chart enables **User Workload Monitoring** (OpenShift's Prometheus for non-platform namespaces) and creates **PodMonitors** that tell it which pods to scrape.

```bash
oc get podmonitors,servicemonitors -A | grep -i -E 'vllm|llama'
oc get configmap user-workload-monitoring-config -n openshift-user-workload-monitoring
```

> **Where are my monitors?** Do not expect them in `observability-hub`. A `PodMonitor` or `ServiceMonitor` can live in any namespace, and by default it only selects pods or services in **its own** namespace unless it sets a `namespaceSelector`. Search all namespaces as above and check that the monitor sits next to (or explicitly selects) the model's namespace. On the roadshow cluster the model runs in `llama-serving`, and a monitor in a different namespace without a `namespaceSelector` scrapes nothing, even though it shows up in the list.

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
