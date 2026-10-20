# Getting started - deploy the observability stack

In this exercise you make sure the observability stack and a small Llama Stack application that emits telemetry are running.

> **Requirements**: cluster-admin on OpenShift 4.12+ with OpenShift AI 2.19+, Helm 3.8+, the `oc` CLI and a GPU node for the model. The full stack needs roughly 8 CPU cores and 16GB of memory, plus the model's own needs.

## 1. Check whether the stack is already there

On the roadshow clusters the observability stack is usually deployed for you by GitOps (an Argo CD application that applies an Advanced Cluster Management *Policy*). Check before you install anything:

```bash
oc get applications.argoproj.io -n openshift-gitops | grep -i observability
oc get pods -n observability-hub
```

> **Why use the full resource name?** `oc get application` can resolve to a different API group (`app.k8s.io`) and report "not found". `applications.argoproj.io` is unambiguous.

If you see the `observability-local-cluster` application and running pods in `observability-hub` (Tempo, Grafana, MinIO, the OTel collector), **skip to step 3**. Do not run the install scripts: Helm will refuse to take over objects it did not create, with errors like:

```
Error: INSTALLATION FAILED: unable to continue with install: Secret "minio-tempo" in namespace
"observability-hub" exists and cannot be imported into the current release: invalid ownership
metadata; annotation validation error: missing key "meta.helm.sh/release-name"
```

> **Concept - Helm ownership**: `helm install` only manages objects carrying `meta.helm.sh/release-name` and `release-namespace` annotations. Objects rendered with `helm template | oc apply` (which is what GitOps tools do) have Helm-style labels but no release, so Helm treats them as foreign. The clue is `helm list -A` returning nothing while the labels say `managed-by: Helm`. Don't change these objects with `oc patch` either: the policy is set to `enforce` and reverts manual edits. Change the Git repo instead.

## 2. Install it yourself (clean cluster only)

Use this path only when step 1 found nothing.

```bash
git clone https://github.com/rh-ai-kickstart/llama-stack-observability.git
cd llama-stack-observability
./scripts/install-full-stack.sh
```

> **Why three phases?** The operators install the CRDs (custom resource types such as `TempoStack` and `OpenTelemetryCollector`). The later phases create objects of those types, so they must exist first.

If some operators are already installed (for example by OpenShift AI), run only the phases you need:

```bash
./scripts/deploy-observability.sh     # Tempo, OTel collector, Grafana, UWM
./scripts/deploy-ai-workloads.sh      # Llama Stack, model, MCP servers
```

## 3. Verify

```bash
oc get pods -n observability-hub
oc get pods -n llama-stack      # the namespace may be llama-serve in the kickstart
oc get podmonitors -n observability-hub
```

All pods should reach `Running`. The model pod can take several minutes to pull its image and load weights.

### Troubleshooting: the OTel collector is in `CrashLoopBackOff`

If `otel-collector-collector` keeps restarting, read its log:

```bash
oc logs deploy/otel-collector-collector -n observability-hub --tail=20
```

If it says `'service.telemetry.metrics' ... has invalid keys: address`, the collector config uses a setting that newer collector versions removed. In the `OpenTelemetryCollector` config, replace

```yaml
service:
  telemetry:
    metrics:
      address: 0.0.0.0:8888
```

with

```yaml
service:
  telemetry:
    metrics:
      level: detailed
```

Make the change in the source Git repository so GitOps does not revert it. Without a running collector, no traces reach Tempo and the next exercises will show nothing.

> **Gotcha - a GitOps fix can leave the old setting behind.** The policy applies these objects with `complianceType: musthave`, which means "these fields must exist", not "the object must match exactly". After you merge a change that *removes* a key (here `address`), the new value (`level: detailed`) is added but the old key stays in the live object, and the collector keeps crashing. Check with:
>
> ```bash
> oc get opentelemetrycollector otel-collector -n observability-hub \
>   -o jsonpath='{.spec.config.service.telemetry.metrics}{"\n"}'
> ```
>
> If `address` is still listed, remove it once by hand. The policy will not put it back, because the Git definition no longer contains it:
>
> ```bash
> oc patch opentelemetrycollector otel-collector -n observability-hub --type=json \
>   -p '[{"op":"remove","path":"/spec/config/service/telemetry/metrics/address"}]'
> ```
>
> Repeat for any sidecar collectors that still show it. The same applies to list entries, such as an old Prometheus scrape target that is left next to the new one.

### Troubleshooting: `no such host` when scraping the model

If the collector log repeats `lookup llama3-2-3b-predictor.<namespace>.svc... no such host`, the Prometheus scrape target names a namespace the model is not in. Find where it really runs and compare:

```bash
oc get inferenceservice -A
oc get opentelemetrycollector otel-collector -n observability-hub \
  -o jsonpath='{.spec.config.receivers.prometheus.config.scrape_configs[*].static_configs[*].targets}{"\n"}'
```

On the roadshow cluster the model runs in `llama-serving`.

Click **Distributed tracing** next.
