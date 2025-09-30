# Helm Chart Quick Reference

This guide provides quick commands for common Helm operations with the P110-Exporter chart.

## Prerequisites

```bash
# Verify Helm is installed
helm version

# Verify kubectl is configured
kubectl cluster-info

# Create secret for TAPO credentials (REQUIRED before installation)
kubectl create secret generic p110-exporter-secret \
  --from-literal=tapo-email='your-email@example.com' \
  --from-literal=tapo-password='your-password'
```

## Installation

### Basic Installation

```bash
# Install with minimal configuration
helm install p110-exporter ./helm/p110-exporter \
  --set existingSecret="p110-exporter-secret" \
  --set config.devicesEnv="study=192.168.1.102,living_room=192.168.1.183"
```

### Install with values file

```bash
# Create your custom values file
cat > my-values.yaml <<EOF
existingSecret: "p110-exporter-secret"

config:
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
EOF

# Install with custom values
helm install p110-exporter ./helm/p110-exporter -f my-values.yaml
```

### Install in specific namespace

```bash
# Create namespace and secret
kubectl create namespace monitoring
kubectl create secret generic p110-exporter-secret \
  -n monitoring \
  --from-literal=tapo-email='your-email@example.com' \
  --from-literal=tapo-password='your-password'

# Install chart
helm install p110-exporter ./helm/p110-exporter \
  -n monitoring \
  --set existingSecret="p110-exporter-secret" \
  --set config.devicesEnv="study=192.168.1.102"
```

## Testing Before Installation

### Lint the chart

```bash
helm lint ./helm/p110-exporter
```

### Dry-run installation

```bash
# Create a test secret first
kubectl create secret generic test-secret \
  --from-literal=tapo-email='test@example.com' \
  --from-literal=tapo-password='testpass' \
  --dry-run=client -o yaml | kubectl apply -f -

# Dry-run the installation
helm install p110-exporter ./helm/p110-exporter \
  --dry-run \
  --debug \
  --set existingSecret="test-secret"
```

### Generate templates without installation

```bash
helm template my-release ./helm/p110-exporter \
  --set existingSecret="test-secret" \
  > generated-manifests.yaml
```

## Upgrading

### Upgrade with new values

```bash
helm upgrade p110-exporter ./helm/p110-exporter -f my-values.yaml
```

### Upgrade specific values

```bash
helm upgrade p110-exporter ./helm/p110-exporter \
  --reuse-values \
  --set config.maxRetryCount=5
```

## Managing Releases

### List releases

```bash
helm list
# Or in specific namespace
helm list -n monitoring
```

### Get release status

```bash
helm status p110-exporter
```

### Get release values

```bash
helm get values p110-exporter
```

### Get all release information

```bash
helm get all p110-exporter
```

### Rollback to previous version

```bash
helm rollback p110-exporter
# Or to specific revision
helm rollback p110-exporter 1
```

## Uninstalling

```bash
helm uninstall p110-exporter
# Or in specific namespace
helm uninstall p110-exporter -n monitoring
```

## Troubleshooting

### Check pod status

```bash
kubectl get pods -l app.kubernetes.io/name=p110-exporter
```

### View pod logs

```bash
kubectl logs -l app.kubernetes.io/name=p110-exporter --tail=100 -f
```

### Describe pod

```bash
kubectl describe pod -l app.kubernetes.io/name=p110-exporter
```

### Check service

```bash
kubectl get svc -l app.kubernetes.io/name=p110-exporter
```

### Port-forward to test metrics

```bash
kubectl port-forward svc/p110-exporter-p110-exporter 9333:9333
# Then access http://localhost:9333/metrics
```

### Check ConfigMap

```bash
kubectl get configmap -l app.kubernetes.io/name=p110-exporter
kubectl describe configmap <configmap-name>
```

### Check Secret

```bash
kubectl get secret -l app.kubernetes.io/name=p110-exporter
# Don't use 'describe' as it shows secret values
```

### View generated manifests

```bash
helm get manifest p110-exporter
```

## Common Configuration Examples

### Enable Prometheus ServiceMonitor

```bash
# Create secret first
kubectl create secret generic p110-exporter-secret \
  --from-literal=tapo-email='your-email@example.com' \
  --from-literal=tapo-password='your-password'

# Install with ServiceMonitor
helm install p110-exporter ./helm/p110-exporter \
  --set existingSecret="p110-exporter-secret" \
  --set config.devicesEnv="study=192.168.1.102" \
  --set serviceMonitor.enabled=true \
  --set serviceMonitor.additionalLabels.release=prometheus
```

### Enable Ingress

```bash
# Create secret first
kubectl create secret generic p110-exporter-secret \
  --from-literal=tapo-email='your-email@example.com' \
  --from-literal=tapo-password='your-password'

# Install with Ingress
helm install p110-exporter ./helm/p110-exporter \
  --set existingSecret="p110-exporter-secret" \
  --set config.devicesEnv="study=192.168.1.102" \
  --set ingress.enabled=true \
  --set ingress.hosts[0].host=p110.example.com \
  --set ingress.hosts[0].paths[0].path=/ \
  --set ingress.hosts[0].paths[0].pathType=Prefix
```

### Set resource limits

```bash
# Create secret first
kubectl create secret generic p110-exporter-secret \
  --from-literal=tapo-email='your-email@example.com' \
  --from-literal=tapo-password='your-password'

# Install with resource limits
helm install p110-exporter ./helm/p110-exporter \
  --set existingSecret="p110-exporter-secret" \
  --set config.devicesEnv="study=192.168.1.102" \
  --set resources.limits.cpu=200m \
  --set resources.limits.memory=256Mi \
  --set resources.requests.cpu=100m \
  --set resources.requests.memory=128Mi
```

### Use existing secret with custom keys

```bash
# Create secret with custom key names
kubectl create secret generic my-tapo-creds \
  --from-literal=email='your-email@example.com' \
  --from-literal=password='your-password'

# Install referencing the custom secret
helm install p110-exporter ./helm/p110-exporter \
  --set config.devicesEnv="study=192.168.1.102" \
  --set existingSecret=my-tapo-creds \
  --set existingSecretEmailKey=email \
  --set existingSecretPasswordKey=password
```

### Set node affinity

```bash
# Create secret first
kubectl create secret generic p110-exporter-secret \
  --from-literal=tapo-email='your-email@example.com' \
  --from-literal=tapo-password='your-password'

# Install with node selector
helm install p110-exporter ./helm/p110-exporter \
  --set existingSecret="p110-exporter-secret" \
  --set config.devicesEnv="study=192.168.1.102" \
  --set nodeSelector."kubernetes\.io/hostname"=node-1
```

## Package and Distribute

### Package the chart

```bash
helm package ./helm/p110-exporter
# This creates p110-exporter-0.1.0.tgz
```

### Create chart index

```bash
helm repo index . --url https://your-repo-url/charts
```

## Validation

### Verify all resources are created

```bash
kubectl get all -l app.kubernetes.io/name=p110-exporter
```

### Check if metrics endpoint is accessible

```bash
kubectl run -it --rm debug --image=curlimages/curl --restart=Never -- \
  curl http://p110-exporter-p110-exporter:9333/metrics
```

## Additional Resources

- [Helm Documentation](https://helm.sh/docs/)
- [P110-Exporter Repository](https://github.com/insta-code1/P110-Exporter)
- [Helm Chart README](./README.md)
- [Grafana Dashboard 17104](https://grafana.com/grafana/dashboards/17104-energy-monitoring/)
