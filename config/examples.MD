# ClusterForge Component Configuration Examples

## Prometheus Configuration

### Basic prometheus-values.yaml
```yaml
prometheus:
  prometheusSpec:
    retention: 15d
    resources:
      requests:
        cpu: 1
        memory: 2Gi
      limits:
        cpu: 2
        memory: 4Gi
    storageSpec:
      volumeClaimTemplate:
        spec:
          storageClassName: gp3-encrypted
          resources:
            requests:
              storage: 50Gi
    serviceMonitorSelector:
      matchLabels:
        monitoring: prometheus
    podMonitorSelector:
      matchLabels:
        monitoring: prometheus
    ruleSelector:
      matchLabels:
        monitoring: prometheus

alertmanager:
  alertmanagerSpec:
    storage:
      volumeClaimTemplate:
        spec:
          storageClassName: gp3-encrypted
          resources:
            requests:
              storage: 10Gi

nodeExporter:
  resources:
    limits:
      cpu: 200m
      memory: 256Mi
    requests:
      cpu: 100m
      memory: 128Mi
```

## Grafana Configuration

### Basic grafana-values.yaml
```yaml
grafana:
  adminPassword: "yourSecurePassword"  # Change this!
  persistence:
    enabled: true
    size: 10Gi
    storageClassName: gp3-encrypted
  
  dashboardProviders:
    dashboardproviders.yaml:
      apiVersion: 1
      providers:
      - name: 'default'
        orgId: 1
        folder: ''
        type: file
        disableDeletion: false
        editable: true
        options:
          path: /var/lib/grafana/dashboards
  
  dashboards:
    default:
      kubernetes-cluster:
        gnetId: 7249
        revision: 1
        datasource: Prometheus
      node-exporter:
        gnetId: 1860
        revision: 27
        datasource: Prometheus

  resources:
    limits:
      cpu: 500m
      memory: 1Gi
    requests:
      cpu: 250m
      memory: 512Mi
      
  ingress:
    enabled: true
    annotations:
      kubernetes.io/ingress.class: alb
    hosts:
      - grafana.yourdomain.com
```

## Loki Configuration (Log Aggregation)

### Basic loki-values.yaml
```yaml
loki:
  auth_enabled: false
  storage:
    type: filesystem
  persistence:
    enabled: true
    size: 50Gi
    storageClassName: gp3-encrypted
  
  resources:
    limits:
      cpu: 1
      memory: 2Gi
    requests:
      cpu: 500m
      memory: 1Gi

promtail:
  resources:
    limits:
      cpu: 200m
      memory: 256Mi
    requests:
      cpu: 100m
      memory: 128Mi
```

## Kubernetes Dashboard Configuration

### Basic kubernetes-dashboard-values.yaml
```yaml
dashboard:
  resources:
    limits:
      cpu: 500m
      memory: 512Mi
    requests:
      cpu: 250m
      memory: 256Mi
  
  ingress:
    enabled: true
    annotations:
      kubernetes.io/ingress.class: alb
    
  extraArgs:
    - --token-ttl=43200
    - --enable-skip-login
    - --enable-insecure-login
    
  serviceAccount:
    create: true
    name: dashboard-admin
    clusterAdmin: true
```

## Custom Metrics Configuration

### metrics-server-values.yaml
```yaml
metrics-server:
  args:
    - --kubelet-preferred-address-types=InternalIP
    - --kubelet-insecure-tls
  resources:
    limits:
      cpu: 200m
      memory: 256Mi
    requests:
      cpu: 100m
      memory: 128Mi
```

## Alert Rules Configuration

### prometheus-rules.yaml
```yaml
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: cluster-alerts
  namespace: monitoring
  labels:
    monitoring: prometheus
spec:
  groups:
  - name: node
    rules:
    - alert: HighCPUUsage
      expr: 100 - (avg by(instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100) > 80
      for: 5m
      labels:
        severity: warning
      annotations:
        summary: High CPU usage on {{ $labels.instance }}
        
    - alert: HighMemoryUsage
      expr: (node_memory_MemTotal_bytes - node_memory_MemAvailable_bytes) / node_memory_MemTotal_bytes * 100 > 80
      for: 5m
      labels:
        severity: warning
      annotations:
        summary: High memory usage on {{ $labels.instance }}
```
