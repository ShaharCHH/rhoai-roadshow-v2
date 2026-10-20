# Getting started - deploy the observability stack

In this exercise you deploy the observability stack and a small Llama Stack application that emits telemetry.

> **Requirements**: cluster-admin on OpenShift 4.12+ with OpenShift AI 2.19+, Helm 3.8+, the `oc` CLI and a GPU node for the model. The full stack needs roughly 8 CPU cores and 16GB of memory, plus the model's own needs.

## 1. Get the code

```bash
git clone https://github.com/rh-ai-kickstart/llama-stack-observability.git
cd llama-stack-observability
```

## 2. Install everything

The repository ships an installer that runs the three phases in order: operators, observability infrastructure, then AI workloads.

```bash
./scripts/install-full-stack.sh
```

> **Why three phases?** The operators install the CRDs (custom resource types such as `TempoStack` and `OpenTelemetryCollector`). The later phases create objects of those types, so they must exist first.

If you prefer to run one phase at a time:

```bash
./scripts/install-operators.sh
./scripts/deploy-observability.sh
./scripts/deploy-ai-workloads.sh
```

## 3. Verify

```bash
oc get pods -n observability-hub
oc get pods -n llama-serve
oc get podmonitors -n observability-hub
```

All pods should reach `Running`. The model pod can take several minutes to pull its image and load weights.

Click **Distributed tracing** next.
