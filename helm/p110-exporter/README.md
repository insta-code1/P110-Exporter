# P110-Exporter Helm Chart

This Helm chart deploys the [P110-Exporter](https://github.com/insta-code1/P110-Exporter) application to Kubernetes. The P110-Exporter exports energy consumption data from Tapo P110 smart devices to Prometheus.

## Prerequisites

- Kubernetes 1.16+
- Helm 3.0+
- Tapo P110 smart plug devices with known IP addresses
- Tapo account credentials (email and password)

## Installation

### Add the chart repository (if published)

```bash
helm repo add p110-exporter https://your-chart-repo-url
helm repo update
```

### Install from local chart

```bash
# From the repository root
helm install my-p110-exporter ./helm/p110-exporter
```

### Install with custom values

1. Create a `custom-values.yaml` file:

```yaml
config:
  tapoEmail: "your-email@example.com"
  tapoPassword: "your-password"
  devicesConfig: |
    devices:
      study: "192.168.1.102"
      living_room: "192.168.1.183"

resources:
  limits:
    cpu: 200m
    memory: 256Mi
  requests:
    cpu: 100m
    memory: 128Mi
```

2. Install the chart:

```bash
helm install my-p110-exporter ./helm/p110-exporter -f custom-values.yaml
```

### Using environment variable for devices

Instead of using a ConfigMap, you can specify devices via environment variable:

```yaml
config:
  tapoEmail: "your-email@example.com"
  tapoPassword: "your-password"
  devicesEnv: "study=192.168.1.102,living_room=192.168.1.183"
```

### Using an existing secret for credentials

If you have an existing Kubernetes secret with your Tapo credentials:

```bash
kubectl create secret generic my-tapo-secret \
  --from-literal=tapo-email='your-email@example.com' \
  --from-literal=tapo-password='your-password'
```

Then reference it in your values:

```yaml
existingSecret: "my-tapo-secret"
existingSecretEmailKey: "tapo-email"
existingSecretPasswordKey: "tapo-password"
```

## Configuration

The following table lists the configurable parameters of the P110-Exporter chart and their default values.

| Parameter | Description | Default |
|-----------|-------------|---------|
| `replicaCount` | Number of replicas | `1` |
| `image.repository` | Image repository | `povilasid/p110-exporter` |
| `image.pullPolicy` | Image pull policy | `IfNotPresent` |
| `image.tag` | Image tag | `""` (uses appVersion from Chart.yaml) |
| `imagePullSecrets` | Image pull secrets | `[]` |
| `nameOverride` | Override chart name | `""` |
| `fullnameOverride` | Override full chart name | `""` |
| `serviceAccount.create` | Create service account | `true` |
| `serviceAccount.annotations` | Service account annotations | `{}` |
| `serviceAccount.name` | Service account name | `""` |
| `podAnnotations` | Pod annotations | `{}` |
| `podSecurityContext` | Pod security context | See values.yaml |
| `securityContext` | Container security context | See values.yaml |
| `service.type` | Service type | `ClusterIP` |
| `service.port` | Service port | `9333` |
| `service.annotations` | Service annotations | `{}` |
| `ingress.enabled` | Enable ingress | `false` |
| `ingress.className` | Ingress class name | `""` |
| `ingress.annotations` | Ingress annotations | `{}` |
| `ingress.hosts` | Ingress hosts | See values.yaml |
| `ingress.tls` | Ingress TLS configuration | `[]` |
| `resources` | Resource requests and limits | `{}` |
| `livenessProbe.enabled` | Enable liveness probe | `true` |
| `readinessProbe.enabled` | Enable readiness probe | `true` |
| `autoscaling.enabled` | Enable horizontal pod autoscaler | `false` |
| `nodeSelector` | Node selector | `{}` |
| `tolerations` | Tolerations | `[]` |
| `affinity` | Affinity rules | `{}` |
| `config.tapoEmail` | Tapo account email | `""` |
| `config.tapoPassword` | Tapo account password | `""` |
| `config.port` | Prometheus port | `9333` |
| `config.maxRetryCount` | Max retry count for device connection | `3` |
| `config.devicesEnv` | Devices as environment variable | `""` |
| `config.devicesConfig` | Devices configuration (YAML format) | See values.yaml |
| `existingSecret` | Existing secret name for credentials | `""` |
| `existingSecretEmailKey` | Key in existing secret for email | `tapo-email` |
| `existingSecretPasswordKey` | Key in existing secret for password | `tapo-password` |
| `serviceMonitor.enabled` | Enable ServiceMonitor for Prometheus Operator | `false` |
| `serviceMonitor.additionalLabels` | Additional labels for ServiceMonitor | `{}` |
| `serviceMonitor.interval` | Scrape interval | `30s` |
| `serviceMonitor.scrapeTimeout` | Scrape timeout | `10s` |
| `podMonitor.enabled` | Enable PodMonitor for Prometheus Operator | `false` |
| `podMonitor.additionalLabels` | Additional labels for PodMonitor | `{}` |
| `podMonitor.interval` | Scrape interval | `30s` |
| `podMonitor.scrapeTimeout` | Scrape timeout | `10s` |

## Prometheus Integration

### Manual Prometheus Configuration

Add the following to your Prometheus configuration:

```yaml
scrape_configs:
  - job_name: 'p110-exporter'
    static_configs:
    - targets: ['my-p110-exporter.default.svc.cluster.local:9333']
```

### Using Prometheus Operator

If you have Prometheus Operator installed, enable the ServiceMonitor:

```yaml
serviceMonitor:
  enabled: true
  additionalLabels:
    release: prometheus  # Match your Prometheus Operator release label
```

## Grafana Dashboard

Import the Grafana dashboard by pasting ID **17104** from https://grafana.com/grafana/dashboards/17104-energy-monitoring/

## Exposed Metrics

The exporter exposes the following metrics:

- `tapo_p110_observation_rate_ms` - RED metrics for queries to the TP-Link TAPO P110 devices
- `tapo_p110_device_count` - Number of available TP-Link TAPO P110 Smart Sockets
- `tapo_p110_today_runtime_mins` - Current running time for the device today (minutes)
- `tapo_p110_month_runtime_mins` - Current running time for the device this month (minutes)
- `tapo_p110_today_energy_wh` - Energy consumed by the device today (Watt-hours)
- `tapo_p110_month_energy_wh` - Energy consumed by the device this month (Watt-hours)
- `tapo_p110_power_consumption_w` - Current power consumption for device (Watts)

## Upgrading

To upgrade an existing release:

```bash
helm upgrade my-p110-exporter ./helm/p110-exporter -f custom-values.yaml
```

## Uninstalling

To uninstall/delete the release:

```bash
helm uninstall my-p110-exporter
```

## Examples

### Minimal installation with inline configuration

```bash
helm install p110-exporter ./helm/p110-exporter \
  --set config.tapoEmail="your-email@example.com" \
  --set config.tapoPassword="your-password" \
  --set config.devicesEnv="study=192.168.1.102,living_room=192.168.1.183"
```

### Production installation with resources and monitoring

```yaml
# production-values.yaml
replicaCount: 1

config:
  tapoEmail: "your-email@example.com"
  tapoPassword: "your-password"
  port: 9333
  maxRetryCount: 3
  devicesConfig: |
    devices:
      study: "192.168.1.102"
      living_room: "192.168.1.183"
      bedroom: "192.168.1.184"

resources:
  limits:
    cpu: 200m
    memory: 256Mi
  requests:
    cpu: 100m
    memory: 128Mi

serviceMonitor:
  enabled: true
  additionalLabels:
    release: prometheus

podSecurityContext:
  runAsNonRoot: true
  runAsUser: 1000
  runAsGroup: 1000
  fsGroup: 1000

securityContext:
  capabilities:
    drop:
    - ALL
  readOnlyRootFilesystem: false
  allowPrivilegeEscalation: false

nodeSelector:
  kubernetes.io/os: linux

tolerations: []

affinity:
  podAntiAffinity:
    preferredDuringSchedulingIgnoredDuringExecution:
    - weight: 100
      podAffinityTerm:
        labelSelector:
          matchExpressions:
          - key: app.kubernetes.io/name
            operator: In
            values:
            - p110-exporter
        topologyKey: kubernetes.io/hostname
```

Install with:

```bash
helm install p110-exporter ./helm/p110-exporter -f production-values.yaml
```

## Troubleshooting

### Check pod logs

```bash
kubectl logs -l app.kubernetes.io/name=p110-exporter
```

### Check if the service is accessible

```bash
kubectl port-forward svc/my-p110-exporter 9333:9333
curl http://localhost:9333/metrics
```

### Verify configuration

```bash
kubectl get configmap my-p110-exporter -o yaml
kubectl get secret my-p110-exporter -o yaml
```

## Contributing

Contributions are welcome! Please open an issue or pull request on the [GitHub repository](https://github.com/insta-code1/P110-Exporter).

## License

This chart is licensed under the same license as the P110-Exporter application.
