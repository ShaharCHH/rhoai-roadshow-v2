# Cleanup the Lab Environment

Remove what you deployed so you can continue with other modules.

```bash
helm uninstall langfuse -n langfuse
oc delete project langfuse
```

If the observability stack was deployed for you by GitOps (the `observability-local-cluster` Argo CD application), leave it in place: deleting its namespaces would be recreated or reverted by the policy.

If you installed it yourself, run the uninstall steps documented in the [kickstart repository](https://github.com/rh-ai-kickstart/llama-stack-observability), or delete the `llama-serve` and `observability-hub` namespaces if they were created only for this lab:

```bash
oc delete project llama-serve observability-hub
```

> Check `oc get pods -A | grep -E 'llama-serve|observability-hub'` first, so you are sure nothing else lives in those namespaces.

🎉 Congratulations! You have completed the LLM Observability module.
