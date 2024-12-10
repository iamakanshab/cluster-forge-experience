# ClusterForge Developer Quickstart Guide

## 🚀 Quick Setup (5 minutes)

### Prerequisites Installation
```bash
# Install required tools
brew install kubectl helm go@1.23    # For macOS
# OR
sudo apt-get install kubectl helm    # For Ubuntu
go install golang.org/dl/go1.23@latest

# Verify installations
kubectl version --client
helm version
go version
```

### Deploy ClusterForge Stack

1. Clone and Setup
```bash
# Clone repository
git clone https://github.com/silogen/cluster-forge.git
cd cluster-forge

# If using Docker (recommended for consistency)
docker compose up -d
```

2. One-Command Deploy
```bash
# Using Docker
docker compose run forge --smelt --cast --forge

# OR using Go directly
go run . --smelt --cast --forge
```

### 🔍 Quick Verification
```bash
# Check if everything is running
kubectl get pods -A

# Get dashboard access
kubectl get svc -n kubernetes-dashboard
```

## 👩‍💻 Developer Common Tasks

### Adding New Components
1. Edit `input/config.yaml`:
```yaml
components:
  - name: your-component
    enabled: true
    namespace: your-namespace
```

2. Regenerate and deploy:
```bash
go run . --smelt --cast --forge
```

### Accessing Services

#### Grafana
```bash
kubectl port-forward svc/grafana 3000:3000 -n monitoring
# Visit http://localhost:3000
# Default credentials: admin/admin
```

#### Prometheus
```bash
kubectl port-forward svc/prometheus 9090:9090 -n monitoring
# Visit http://localhost:9090
```

### Quick Debugging
```bash
# Check pod logs
kubectl logs -f <pod-name> -n <namespace>

# Check pod details
kubectl describe pod <pod-name> -n <namespace>

# Get all resources in a namespace
kubectl get all -n <namespace>
```

## 📊 Monitoring Your Apps

### Add Service Monitor
```yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: your-app
  namespace: monitoring
spec:
  selector:
    matchLabels:
      app: your-app
  endpoints:
  - port: metrics
```

### Basic Grafana Dashboard
1. Login to Grafana
2. Import dashboard <-----link-\---> for basic Kubernetes metrics
3. Add your app's metrics using PromQL:
```promql
rate(http_requests_total{app="your-app"}[5m])
```

## 🔄 Common Workflows

### Update Configuration
```bash
# Edit configs in working/
vim working/your-component.yaml

# Reapply
go run . --forge
```

### Scale Components
```bash
kubectl scale deployment your-app --replicas=3 -n your-namespace
```

### View Logs
```bash
# Using Grafana Loki
1. Open Grafana
2. Go to Explore
3. Select Loki
4. Query: {app="your-app"}
```

## 🆘 Quick Troubleshooting

### Common Issues & Fixes

1. Pods not starting:
```bash
kubectl get events -n <namespace>
```

2. Resource issues:
```bash
kubectl top pods -n <namespace>
```

3. Configuration issues:
```bash
kubectl get configmap -n <namespace>
kubectl describe configmap <name> -n <namespace>
```

### Quick Reset
```bash
# Delete problematic resources
kubectl delete pod <pod-name> -n <namespace>

# Rerun forge
go run . --forge
```

## 🔗 Quick References

- Metrics: `http://localhost:9090/metrics`
- Dashboards: `http://localhost:3000`
- Documentation: `/docs` in repository
- Logs: Grafana Loki interface

## 📝 Development Best Practices

1. Always version your changes
2. Use resource limits
3. Follow naming conventions
4. Document custom changes
5. Use separate namespaces
