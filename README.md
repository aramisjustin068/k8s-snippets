# k8s-snippets

The Kubernetes manifests I copy into every new service, kept in one place so the
defaults stop drifting between teams. Three files, no templating engine, no
chart to install.

Each manifest uses `APP_NAME`, `IMAGE` and `NAMESPACE` as placeholders. Replace
them however you like — `sed`, `envsubst`, or a Kustomize patch.

## What is here

| File | Purpose |
| --- | --- |
| `deployment.yaml` | Deployment with probes, resource limits, and a non-root security context |
| `service.yaml` | ClusterIP Service matching the Deployment selector |
| `networkpolicy.yaml` | Default-deny ingress plus one explicit allow rule |

## Usage

```sh
export APP_NAME=checkout-api
export IMAGE=registry.example.com/checkout-api:2026.03.1
export NAMESPACE=payments

for f in deployment.yaml service.yaml networkpolicy.yaml; do
  envsubst < "$f" | kubectl apply -f -
done
```

Dry run first if the namespace is shared:

```sh
envsubst < deployment.yaml | kubectl apply --dry-run=server -f -
```

## Conventions baked in

- Requests are set on every container; limits are set on memory but not CPU, so
  throttling does not hide behind a latency graph.
- `readinessProbe` and `livenessProbe` hit different paths on purpose. Liveness
  should not fail because a dependency is slow.
- Containers run as a non-root user with a read-only root filesystem.
- `topologySpreadConstraints` spread replicas across nodes before zones.
- The NetworkPolicy denies all ingress first, then allows the ingress
  controller. Adding a dependency means adding a rule and saying why.

## License

Released under the MIT License.
