# Kubernetes Setup Guide for Amazon Linux 2023
This guide documents the steps to set up a Kubernetes development environment using Minikube on Amazon Linux 2023.

## Prerequisites
- An EC2 instance running Amazon Linux 2023
- Sufficient permissions to run sudo commands
- At least 2 CPU cores and 2GB RAM recommended

## Step 1: Install Docker
```bash
# Update system packages
sudo yum update -y

# Install Docker
sudo yum install docker -y

# Start and enable Docker service
sudo systemctl start docker
sudo systemctl enable docker

# Add current user to docker group
sudo usermod -aG docker $USER
sudo groupadd docker

# Apply group changes
newgrp docker
```

## Step 2: Install kubectl
```bash
# Download kubectl binary
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"

# Make kubectl executable and move to path
sudo chmod +x kubectl
sudo mv kubectl /usr/local/bin/
```

## Step 3: Install Minikube
```bash
# Download Minikube binary
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64

# Make it executable and move to path
sudo install minikube-linux-amd64 /usr/local/bin/minikube
```

## Step 4: Start Minikube Cluster
```bash
# Start Minikube with Docker driver
minikube start --driver=docker
```

## Step 5: Configure kubectl
```bash
# Create .kube directory
mkdir -p $HOME/.kube

# Get Minikube kubectl configuration
minikube kubectl -- config view --flatten > ~/.kube/config
```

## Step 6: Verify Installation
```bash
# Check nodes
kubectl get nodes

# Check all pods
kubectl get pods -A

# Check cluster info
kubectl cluster-info
```

## Common Issues and Solutions

### Docker Permission Issues
If you see "permission denied" errors with Docker:
```bash
sudo chmod 666 /var/run/docker.sock
sudo systemctl restart docker
newgrp docker
```

### Minikube Won't Start
If Minikube fails to start:
```bash
minikube delete
minikube start --driver=docker --alsologtostderr -v=5
```

## Cleanup
To remove the Minikube cluster:
```bash
minikube stop
minikube delete
```

## Additional Notes
- The setup uses Docker as the container runtime
- Minikube creates a single-node Kubernetes cluster
- This setup is suitable for development and testing, not for production use
- Remember to log out and log back in after adding user to docker group for changes to take effect
