# CDR Lab Deployment Guide

## Repository Overview
This repository serves as a comprehensive lab environment for practicing Cloud Detection and Response (CDR) using XSIAM and Prisma Cloud. The labs simulate real-world cloud-native threats in a controlled environment.


## Configuration Files

### 1. cdr.yml
Contains configurations for:
- Malicious pod (`alpine-cdr-1`) with two containers:
  - `malicous-container`: Runs malicious activity simulation script
  - `atk-ttp-test-runner`: Runs attack technique testing
- DaemonSet running a cryptominer (`xmrig`) on all nodes
- CronJob running a cryptominer every 30 minutes

### 2. ctxcld.yml
Deploys Cortex Cloud application:
- Deployment with 3 replicas
- LoadBalancer service exposing ports 8080 and 5005
- Custom application for monitoring/security

### 3. dvwa.yaml
Deploys Damn Vulnerable Web Application (DVWA):
- Single replica deployment
- LoadBalancer service on port 80
- Deliberately vulnerable web application for security testing

## AWS Deployment Instructions

### Prerequisites
- AWS account with appropriate permissions
- AWS CLI installed and configured
- `kubectl` installed
- `eksctl` installed (recommended)

### Step 1: Create EKS Cluster
```bash
eksctl create cluster \
  --name cdr-lab-cluster \
  --region us-west-2 \
  --node-type t3.medium \
  --nodes 2 \
  --nodes-min 2 \
  --nodes-max 3
```

### Step 2: Configure kubectl
```bash
aws eks update-kubeconfig --name cdr-lab-cluster --region us-west-2
```

### Step 3: Deploy Applications
```bash
# Deploy malicious container and cryptominer
kubectl apply -f cdr.yml

# Deploy Cortex Cloud application
kubectl apply -f ctxcld.yml

# Deploy DVWA
kubectl apply -f dvwa.yaml
```

### Step 4: Access Applications
```bash
# Get LoadBalancer URLs
kubectl get svc cortexcloud-service
kubectl get svc dvwa-service
```

## Important Considerations

### Security
- Configure appropriate security groups for LoadBalancer services
- Consider running in a separate AWS account or VPC
- Be aware of the cryptominer deployment's resource consumption

### Cost Management
- EKS cluster costs
- EC2 instance costs for worker nodes
- Load Balancer costs
- Data transfer costs
- Monitor cryptominer resource usage

### Cleanup
```bash
# Delete deployments
kubectl delete -f cdr.yml
kubectl delete -f ctxcld.yml
kubectl delete -f dvwa.yaml

# Delete cluster
eksctl delete cluster --name cdr-lab-cluster --region us-west-2
```

## Notes
- The repository does not include Cortex XDR agent deployment
- The `ctxcld.yml` references a private ECR repository image
- DVWA is a known vulnerable application - use in isolated environment
- Consider modifying configurations for less aggressive testing

## Additional Configuration Options
1. Security group configurations
2. Monitoring setup
3. Cost-effective configurations (e.g., spot instances)
4. AWS-specific YAML modifications

## Deploying Only cdr.yml to AWS

### Prerequisites
- AWS account with appropriate permissions
- AWS CLI installed and configured
- `kubectl` installed
- `eksctl` installed (recommended)

### Step 1: Create EKS Cluster
```bash
eksctl create cluster \
  --name cdr-lab-cluster \
  --region us-west-2 \
  --node-type t3.medium \
  --nodes 2 \
  --nodes-min 2 \
  --nodes-max 3
```

### Step 2: Configure kubectl
```bash
aws eks update-kubeconfig --name cdr-lab-cluster --region us-west-2
```

### Step 3: Deploy cdr.yml
```bash
# Deploy malicious container and cryptominer
kubectl apply -f cdr.yml

# Verify the deployment
kubectl get pods
kubectl get daemonsets -n kube-system
kubectl get cronjobs
```

### Step 4: Monitor the Deployment
```bash
# Check the malicious pod status
kubectl describe pod alpine-cdr-1

# Check the cryptominer daemonset
kubectl describe daemonset kube-controller -n kube-system

# Check the cronjob
kubectl describe cronjob cryptominer-persistance
```

### Important Security Considerations
1. **Resource Monitoring**:
   - Monitor CPU usage across nodes
   - Watch for unusual network traffic
   - Check for unexpected process creation

2. **Access Control**:
   - Ensure proper IAM roles and permissions
   - Monitor for unauthorized access attempts
   - Check for privilege escalation attempts

3. **Network Security**:
   - Review security group configurations
   - Monitor outbound connections
   - Check for unusual port scanning activity

### Cleanup
```bash
# Delete the malicious deployment
kubectl delete -f cdr.yml

# Delete the cluster (optional)
eksctl delete cluster --name cdr-lab-cluster --region us-west-2
```

### Notes
- The deployment will start mining cryptocurrency to a specific wallet
- CPU usage will be significant due to cryptominer operations
- Monitor your AWS bill for unexpected costs
- Consider using AWS Budgets to set cost alerts
- Run in a separate AWS account or VPC if possible 