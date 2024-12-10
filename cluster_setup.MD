# ClusterForge AWS EKS Cluster Setup Guide

## Overview
This document outlines the setup and configuration of a production-ready Amazon EKS cluster for ClusterForge deployment. The setup includes cluster configuration, networking, storage, and monitoring components.

## Prerequisites
- AWS CLI installed and configured
- kubectl installed
- eksctl installed
- Flux CLI installed
- Access to AWS CloudShell or local terminal

## Cluster Architecture

### Node Groups
The cluster is configured with two managed node groups:

1. System Nodes (t3.large)
   - Purpose: Running system components and monitoring
   - Size: 2-3 nodes
   - Volume: 50GB per node
   - Taints: PreferNoSchedule for system workloads

2. Workload Nodes (t3.xlarge)
   - Purpose: Running application workloads
   - Size: 2-5 nodes (autoscaling)
   - Volume: 100GB per node
   - No taints

### Networking
- VPC CIDR: 192.168.0.0/16
- Private networking enabled
- Single NAT Gateway
- Both private and public endpoint access

### Storage Classes
Two storage classes are provided:
1. gp3-encrypted: Standard encrypted GP3 volumes
2. gp3-encrypted-retained: GP3 volumes with retention policy

## Setup Instructions

### 1. Cluster Creation
```bash
# Install eksctl if not present
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin

# Create cluster
eksctl create cluster -f enhanced-eks-config.yaml
```

### 2. Network Policies
Apply basic network policies for security:
```bash
kubectl apply -f network-policies.yaml
```

These policies:
- Deny all ingress by default
- Allow monitoring namespace access

### 3. Storage Configuration
Apply storage classes:
```bash
kubectl apply -f storage-classes.yaml
```

### 4. Monitoring Setup

#### Install Flux
```bash
# Install Flux CLI
curl -s https://fluxcd.io/install.sh | sudo bash

# Initialize Flux
flux install
```

#### Deploy Prometheus and Grafana
```bash
kubectl apply -f monitoring-setup.yaml
```

The monitoring stack includes:
- Prometheus for metrics collection
- Grafana for visualization
- 15-day metric retention
- Persistent storage for both components

### Accessing Monitoring

#### Grafana Access
1. Get the LoadBalancer URL:
```bash
kubectl get svc -n monitoring kube-prometheus-stack-grafana -o jsonpath='{.status.loadBalancer.ingress[0].hostname}'
```

2. Default credentials:
   - Username: admin
   - Password: clusterforge-admin (change immediately)

## Maintenance

### Scaling
Node groups will automatically scale based on demand:
- System nodes: 2-3 nodes
- Workload nodes: 2-5 nodes

### Storage
- Monitor PVC usage through Grafana dashboards
- GP3 volumes can be expanded without pod restart

### Backup Considerations
Consider implementing:
- EBS snapshots for persistent volumes
- Cluster state backup using tools like Velero
- etcd backup for cluster data

## Security

### Network Security
- All pods are denied ingress by default
- Monitoring namespace has specific access
- Cluster endpoints are accessible via both private and public endpoints

### Storage Security
- All EBS volumes are encrypted by default
- Retained storage class prevents accidental deletion

## Monitoring and Alerts

### Default Monitoring
- Node metrics
- Pod metrics
- Cluster state
- Storage usage
- Network metrics

## Troubleshooting

### Common Commands
```bash
# Check node status
kubectl get nodes

# Check pod status
kubectl get pods -A

# Check storage classes
kubectl get sc

# View monitoring stack status
kubectl get pods -n monitoring
```
