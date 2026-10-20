# Introduction

## The architecture

The lab uses the [Llama Stack observability kickstart](https://github.com/rh-ai-kickstart/llama-stack-observability). It is a set of Helm charts that wire telemetry from AI workloads into OpenShift-native tooling:

| Component | Role |
|-----------|------|
| `otel-collector` | Receives telemetry from Llama Stack and other workloads, processes it and forwards it |
| `tempo` | Distributed tracing backend, storing traces in S3-compatible storage |
| `grafana` | Dashboards, including pre-built ones for vLLM and Llama Stack |
| `uwm` | User Workload Monitoring, with PodMonitors that scrape vLLM and other AI workloads |
| `distributed-tracing-ui-plugin` | Shows traces directly in the OpenShift console |

Traces flow `Llama Stack -> OpenTelemetry collector -> Tempo -> Grafana / OpenShift console`. Metrics flow `vLLM /metrics -> PodMonitor -> User Workload Monitoring -> Grafana`.

> **Concept - OpenTelemetry (OTel)**: a vendor-neutral standard for emitting traces, metrics and logs. Your application talks OTel once, and you can change the backend (Tempo, Jaeger, Datadog...) without touching the code.

## Two tools, two questions

| Question | Best tool |
|----------|-----------|
| Why was this request slow? Which service failed? | Tracing with Tempo |
| Is the model server saturated? Is the GPU busy? | Metrics with Grafana |
| What prompt did the user send, what did the model answer, what did it cost? | Langfuse |

They overlap, and in practice teams use both.

## Agenda

* [🌌 Getting started - deploy the observability stack](1-getting-started.md)
* [🔎 Distributed tracing](2-tracing.md)
* [📈 Metrics and dashboards](3-metrics.md)
* [🪢 Langfuse on OpenShift](4-langfuse.md)
* [🧹 Clean up the lab environment](5-cleanup.md)

Click **Getting started** next.
