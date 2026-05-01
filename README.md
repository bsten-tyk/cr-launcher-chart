# cr-launcher

A generic Helm chart for deploying arbitrary Kubernetes Custom Resources.

## How It Works

The chart accepts a list of resource manifests via `values.yaml` and renders each one as a Kubernetes resource. Helm template syntax within manifests is supported.

## Quick Start

```bash
helm repo add cr-launcher https://bsten-tyk.github.io/cr-launcher-chart/
helm install my-redis cr-launcher/cr-launcher -f my-values.yaml
```

## Configuration

| Parameter | Description | Default |
|-----------|-------------|---------|
| `resources` | List of Kubernetes resource manifests | `[]` |
| `nameOverride` | Override chart name | `""` |
| `fullnameOverride` | Override full release name | `""` |

Each entry in `resources` must be a valid YAML object with `apiVersion`, `kind`, and `metadata`.

## Examples

### Minimal DistributedRedisCluster

```yaml
resources:
  - apiVersion: redis.kun/v1alpha1
    kind: DistributedRedisCluster
    metadata:
      name: my-redis
      annotations:
        redis.kun/scope: cluster-scoped
    spec:
      image: redis:5.0.4-alpine
      masterSize: 3
      clusterReplicas: 0
```

### Multiple Resources (Secret + CR)

```yaml
resources:
  - apiVersion: v1
    kind: Secret
    metadata:
      name: redis-password
    type: Opaque
    stringData:
      password: changeme
  - apiVersion: redis.kun/v1alpha1
    kind: DistributedRedisCluster
    metadata:
      name: my-redis
      annotations:
        redis.kun/scope: cluster-scoped
    spec:
      image: redis:5.0.4-alpine
      masterSize: 3
      clusterReplicas: 0
      passwordSecret:
        name: redis-password
```

### Using Helm Template Syntax

You can use Helm templates within resource manifests:

```yaml
resources:
  - apiVersion: v1
    kind: ConfigMap
    metadata:
      name: {{ include "cr-launcher.fullname" . }}-config
    data:
      key: {{ .Values.someValue | quote }}
```

## More Examples

See the `examples/` directory for complete value files:

- `distributed-redis-cluster.yaml` - Minimal DRC deployment
- `distributed-redis-cluster-with-secret.yaml` - DRC with a companion Secret
- `distributed-redis-cluster-ha.yaml` - HA setup with cluster replicas

## Releasing a New Version

1. Update `version` in `cr-launcher/Chart.yaml`
2. Commit and push to `main`

The GitHub Actions workflow detects the version change, packages the chart, creates a GitHub Release, and updates the Helm repository index on `gh-pages`.

## Prerequisites

- The target CRD must already be registered in the cluster
- The corresponding operator/controller must be running
