# Langfuse on OpenShift

[Langfuse](https://langfuse.com) is an open-source LLM engineering platform. Where OpenTelemetry sees services and latency, Langfuse sees *conversations*: prompts, completions, token counts, cost, user feedback and evaluation scores.

The steps below follow [eformat/openshift-langfuse](https://github.com/eformat/openshift-langfuse), which is a minimal install built on the upstream [langfuse-k8s](https://github.com/langfuse/langfuse-k8s/blob/main/examples/minimal-installation/README.md) chart.

> **Lab use only**: the minimal install uses simple passwords. Never reuse these for anything real.

## 1. Create the project and secrets

```bash
oc new-project langfuse

oc create secret generic langfuse -n langfuse \
  --from-literal=salt="$(openssl rand -base64 32)" \
  --from-literal=encryption-key="$(openssl rand -hex 32)" \
  --from-literal=nextauth-secret="$(openssl rand -hex 16)" \
  --from-literal=postgresql-password=password \
  --from-literal=clickhouse-password=password \
  --from-literal=redis-password=password \
  --from-literal=s3-user=user \
  --from-literal=s3-password=password
```

> **Why generate them**: `salt` and `encryption-key` protect stored API keys. Generating fresh values avoids sharing the sample values from the public repo.

## 2. Install with Helm

The upstream chart is configured with its `values.yaml` for a minimal install, which also needs the `SALT` environment variable set to the secret's salt value.

```bash
export SALT="$(oc get secret langfuse -n langfuse -o jsonpath='{.data.salt}' | base64 -d)"

curl -fsSLO https://raw.githubusercontent.com/langfuse/langfuse-k8s/main/examples/minimal-installation/values.yaml

helm repo add langfuse https://langfuse.github.io/langfuse-k8s
helm repo update
helm install langfuse langfuse/langfuse -n langfuse -f values.yaml
```

Wait for the pods:

```bash
oc get pods -n langfuse -w
```

## 3. Expose it

```bash
oc create route edge langfuse --service=langfuse-web --insecure-policy=Redirect -n langfuse
oc get route langfuse -n langfuse -o jsonpath='{.spec.host}{"\n"}'
```

Open the URL, create a user and an organisation, then create a project and copy its **public and secret API keys**.

## 4. Send traces from an application

Langfuse accepts OpenTelemetry traces and has SDKs for Python. Point your LLM application at the Langfuse host with the project keys, run a few prompts, then open **Tracing** in Langfuse.

Look at:

* The full prompt and completion for each call
* Input/output token counts and calculated cost
* Latency per generation

> **Concept - complementary tools**: Tempo answers "which service was slow?"; Langfuse answers "what did the model say and what did it cost?". Many teams send the same OTel stream to both.

Click **Clean up** next.
