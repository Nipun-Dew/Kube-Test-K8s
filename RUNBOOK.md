# Kubernetes Cluster Setup Guide

This guide provides step-by-step instructions for setting up a Kubernetes cluster with Minikube on an EC2 instance.

## Prerequisites

- EC2 instance running Amazon Linux (at least 8gb RAM)
- SSH access to the EC2 instance
- Correct EC2 Host added into repository secrets for GitHub Actions
- Correct EC2 SSH key added into repository secrets for GitHub Actions
- Working Docker images should be available in the docker hub

## Installation

### 1. Update System and Install Dependencies

```bash
sudo yum update
sudo yum install git -y
sudo yum install docker -y
sudo service docker start
sudo systemctl enable docker
sudo usermod -aG docker ec2-user
```

**Note:** After running the above commands, you need to logout and login again via SSH for the Docker group changes to take effect.

### 2. Install Minikube

```bash
curl -LO https://github.com/kubernetes/minikube/releases/latest/download/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube && rm minikube-linux-amd64
```

### 3. Install kubectl

```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install kubectl /usr/local/bin/kubectl && rm kubectl
```

### 4. Install Helm

```bash
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
```

## Starting the Cluster

### 1. Start Minikube

```bash
minikube start --driver=docker
```

### 2. Enable Ingress

```bash
minikube addons enable ingress
```

### 3. Configure Host Entries

Add the following entries to your hosts file:

**On EC2 instance:**
```
<minikube_ip> app.lugx
```

**On local environment:**
```
127.0.0.1 app.lugx
```

### 4. Create SSH Tunnel (for local access)

To connect to the application from your local machine:

```bash
ssh -i <your-key-file.pem> -L 8080:<minikube_ip>:80 ec2-user@<ec2_public_ip>
```

## Deploy Application

### 1. Clone Repository

```bash
git clone https://github.com/Nipun-Dew/Kube-Test-K8s.git
```

### 2. Trigger GitHub Workflow

1. Navigate to the repository on GitHub
2. Go to **Actions** tab
3. Select **Deploy Kubernetes** workflow
4. Click **Run workflow** to start a new deployment

## Setup Monitoring Tools

### 1. Create Monitoring Namespace

```bash
kubectl create namespace monitoring
```

### 2. Install Prometheus

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm install prometheus prometheus-community/prometheus --namespace monitoring
```

### 3. Expose Prometheus

```bash
kubectl expose service prometheus-server --namespace monitoring --type=NodePort --target-port=9090 --name=prometheus-server-ext
```

### 4. Get Minikube IP and Prometheus Port

```bash
kubectl get svc -n monitoring
minikube ip
```

### 5. Create SSH Tunnel for Prometheus

```bash
ssh -i <your-key-file.pem> -L <preferred_local_port>:<minikube_ip>:<prometheus-port> ec2-user@<ec2_public_ip>
```

### 6. Install Grafana

```bash
helm repo add grafana https://grafana.github.io/helm-charts
helm install grafana grafana/grafana --namespace monitoring
```

### 7. Get Grafana Admin Password

```bash
kubectl get secret --namespace monitoring grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo
```

### 8. Expose Grafana

```bash
kubectl expose service grafana --namespace monitoring --type=NodePort --target-port=3000 --name=grafana-ext
```

### 9. Create SSH Tunnel for Grafana

```bash
ssh -i <your-key-file.pem> -L <preferred_local_port>:<minikube_ip>:<grafana-port> ec2-user@<ec2_public_ip>
```

## Troubleshooting

### Common Issues

- **Docker permission denied**: Make sure you've logged out and back in after adding user to docker group
- **Minikube won't start**: Check if Docker is running and user has proper permissions
- **Application not accessible**: Verify ingress is enabled and host entries are correct

### Useful Commands

```bash
# Check cluster status
minikube status

# View all pods
kubectl get pods --all-namespaces

# Check ingress status
kubectl get ingress

# View minikube IP
minikube ip

# Access Kubernetes dashboard
minikube dashboard
```
