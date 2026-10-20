# Langfuse on OpenShift

[Langfuse](https://langfuse.com) is an open-source LLM engineering platform. Where OpenTelemetry sees services and latency, Langfuse sees *conversations*: prompts, completions, token counts, cost, user feedback and evaluation scores.

The steps below follow [eformat/openshift-langfuse](https://github.com/eformat/openshift-langfuse), which is a minimal install built on the upstream [langfuse-k8s](https://github.com/langfuse/langfuse-k8s/blob/main/examples/minimal-installation/README.md) chart.

> **Lab use only**: the minimal install uses simple passwords. Never reuse these for anything real.

## 1. Install the prerequisites

The current chart creates `ClickHouseCluster` and `KeeperCluster` objects, so the ClickHouse operator must exist first. It also relies on cert-manager, which this cluster already has (check with `oc get crd certificates.cert-manager.io`).

```bash
helm install clickhouse-operator oci://ghcr.io/clickhouse/clickhouse-operator-helm \
  --version 0.0.8 \
  --namespace clickhouse-operator --create-namespace

oc wait --for=condition=Established \
  crd/clickhouseclusters.clickhouse.com \
  crd/keeperclusters.clickhouse.com \
  --timeout=120s
```

> **Why**: a CRD (custom resource definition) teaches Kubernetes a new object type. Without these two, `helm install` fails with "no matches for kind ClickHouseCluster".

## 2. Create the project and secret

One secret feeds Langfuse and the bundled Postgres, ClickHouse, Valkey (Redis) and SeaweedFS (S3). The extra keys (`POSTGRES_*`, `USERDB_*`, `default`, `seaweedfs_s3_config`) are required by those sub-charts.

```bash
oc new-project langfuse

oc apply -n langfuse -f - <<EOF
apiVersion: v1
kind: Secret
type: Opaque
metadata:
  name: langfuse
stringData:
  salt: "$(openssl rand -hex 16)"
  encryption-key: "$(openssl rand -hex 32)"
  nextauth-secret: "$(openssl rand -hex 16)"
  postgresql-password: "password"
  POSTGRES_USER: "postgres"
  POSTGRES_PASSWORD: "password"
  POSTGRES_DB: "langfuse"
  USERDB_USER: "langfuse"
  USERDB_PASSWORD: "password"
  clickhouse-password: "password"
  redis-password: "password"
  default: "password"
  s3-user: "user"
  s3-password: "password"
  seaweedfs_s3_config: |
    {"identities":[{"name":"user","credentials":[{"accessKey":"user","secretKey":"password"}],"actions":["Admin","Read","Write","List","Tagging"]}]}
EOF
```

> **Why the repeated passwords**: the Postgres sub-chart reads `POSTGRES_PASSWORD`/`USERDB_PASSWORD` while Langfuse reads `postgresql-password`, so they must match. The same applies to `redis-password` and `default`, and to the S3 user and password inside `seaweedfs_s3_config`.

## 3. Install with Helm

```bash
curl -fsSLO https://raw.githubusercontent.com/langfuse/langfuse-k8s/main/examples/minimal-installation/values.yaml

helm repo add langfuse https://langfuse.github.io/langfuse-k8s
helm repo update
helm install langfuse langfuse/langfuse -n langfuse -f values.yaml --version 2.1.6
```

Wait for the pods (this takes a few minutes):

```bash
oc get pods -n langfuse -w
```

## 4. Expose it

```bash
oc create route edge langfuse --service=langfuse-web --insecure-policy=Redirect -n langfuse
oc get route langfuse -n langfuse -o jsonpath='{.spec.host}{"\n"}'
```

Tell Langfuse its public URL, otherwise login redirects point at localhost:

```bash
helm upgrade langfuse langfuse/langfuse -n langfuse -f values.yaml --version 2.1.6 \
  --set langfuse.nextauth.url=https://$(oc get route langfuse -n langfuse -o jsonpath='{.spec.host}')
```

Open the URL, create a user and an organisation, then create a project and copy its **public and secret API keys**.

## 5. Send traces from an application

Langfuse accepts OpenTelemetry traces and has SDKs for Python. Point your LLM application at the Langfuse host with the project keys, run a few prompts, then open **Tracing** in Langfuse.

Look at:

* The full prompt and completion for each call
* Input/output token counts and calculated cost
* Latency per generation

> **Concept - complementary tools**: Tempo answers "which service was slow?"; Langfuse answers "what did the model say and what did it cost?". Many teams send the same OTel stream to both.

Click **Clean up** next.
