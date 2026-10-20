# Distributed tracing

A **trace** is the story of one request. It is made of **spans**, each span being one unit of work (a model call, a vector search, an MCP tool call) with a start time, a duration and metadata.

## 1. Generate some traffic

Open the **Llama Stack Playground** route in the `llama-serve` namespace:

```bash
oc get route -n llama-serve
```

Ask the model a question. Then try one that needs a tool, for example "What is the weather in Paris?", which goes through the weather MCP server.

## 2. Find your trace

In the OpenShift console, open **Observe -> Traces** (provided by the distributed tracing UI plugin) and select the Tempo instance in `observability-hub`.

Open a trace and look for:

* **The slowest span** - this is where the time went.
* **Parent/child structure** - how Llama Stack called the model server and the MCP tool.
* **Errors** - failed spans are flagged.

> **Concept - why traces beat logs here**: a single agent request can fan out to several services. Logs show each service in isolation; a trace joins them with one trace ID so you see the whole chain.

## 3. Think like an auditor

The kickstart includes a financial-services demo in which each step of a loan decision is captured as a trace. Consider what a trace gives a compliance team that application logs do not: a complete, ordered record of what the AI system did for a given request.

Click **Metrics and dashboards** next.
