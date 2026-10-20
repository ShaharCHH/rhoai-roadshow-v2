# LLM Observability

See what your AI applications are really doing: every request, every tool call, every token, and what it cost.

## What is it?

Observability means being able to answer "what happened inside this system?" from the data it emits. For an LLM application that data comes in three forms:

* **Metrics** - numbers over time, such as requests per second, tokens per second, GPU utilisation and time-to-first-token.
* **Traces** - the end-to-end path of a single request through every service it touches (app, Llama Stack, vector database, MCP tool, model server).
* **LLM-level events** - the prompts, completions, token counts, cost and user feedback for each model call.

## Why do we care?

An LLM application is non-deterministic, spans many services and bills by usage. Without telemetry you cannot tell why an answer was slow, wrong or expensive, and you cannot prove to an auditor what the model was asked and what it said.

## Hands-on Lab

In this module you will look at two complementary approaches:

* **OpenShift-native observability** - OpenTelemetry, Tempo and Grafana, with vLLM metrics collected by User Workload Monitoring.
* **Langfuse** - an open-source, LLM-specific observability platform running on OpenShift.

You will learn:

- ✅ The telemetry architecture for Llama Stack on OpenShift AI
- ✅ How to follow a single request as a distributed trace
- ✅ How to read model-serving metrics in Grafana
- ✅ How to deploy Langfuse and inspect prompts, token usage and cost

The Agenda:

* [🔭 Introduction](5-llm-observability/0-intro.md)
* [🌌 Getting started - deploy the observability stack](5-llm-observability/1-getting-started.md)
* [🔎 Distributed tracing](5-llm-observability/2-tracing.md)
* [📈 Metrics and dashboards](5-llm-observability/3-metrics.md)
* [🪢 Langfuse on OpenShift](5-llm-observability/4-langfuse.md)
* [🧹 Clean up the lab environment](5-llm-observability/5-cleanup.md)

Click **Introduction** to get started.
