# Cloud To-Do App - Full Stack Deployment Project

## Project Overview
A complete cloud-native to-do application demonstrating Infrastructure as Code, containerization, Kubernetes orchestration, and CI/CD automation on Microsoft Azure.

## Architecture
```
┌─────────────────┐    ┌──────────────────┐    ┌─────────────────┐
│   GitHub Repo   │───▶│  GitHub Actions  │───▶│   Azure ACR     │
│                 │    │    (CI/CD)       │    │ (Container Reg) │
└─────────────────┘    └──────────────────┘    └─────────────────┘
                                │                        │
                                ▼                        │
┌─────────────────┐    ┌──────────────────┐              │
│   Terraform     │───▶│   Azure VM       │◀─────────────┘
│ (Infrastructure)│    │   + Kubernetes   │
└─────────────────┘    └──────────────────┘
```

## Technology Stack
- **Frontend**: HTML/CSS/JavaScript
- **Backend**: Python Flask
- **Database**: SQLite
- **Containerization**: Docker
- **Orchestration**: Kubernetes (K3s)
- **Infrastructure**: Terraform
- **Cloud Platform**: Microsoft Azure
- **CI/CD**: GitHub Actions
- **Container Registry**: Azure Container Registry (ACR)

## Project Structure
```
todo-app/
├── app.py                    # Flask application
├── templates/
│   └── index.html           # HTML template
├── Dockerfile               # Container definition
├── requirements.txt         # Python dependencies
├── main.tf                  # Terraform infrastructure
├── cloud-init.yml          # VM initialization script
├── k8s-manifests.yml       # Kubernetes deployment configs
├── .github/workflows/
│   └── deploy.yml          # CI/CD pipeline
└── .gitignore              # Git ignore rules
```

## Infrastructure Details

### Azure Resources Created
- **Resource Group**: `rg-todo-app`
- **Virtual Machine**: Standard B2s (2 vCPUs, 4GB RAM)
- **Container Registry**: Basic tier ACR
- **Virtual Network**: Custom VNet with security group
- **Storage Account**: Terraform state management

### Cost Optimization
- Started with B1s VM (~$15/month) for cost efficiency
- Upgraded to B2s VM (~$30/month) to resolve resource constraints
- Total monthly cost: ~$35-40 including ACR and networking

## Deployment Process

### Phase 1: Local Development
1. Built Flask to-do application with SQLite backend
2. Created Docker container with health checks
3. Tested application locally

### Phase 2: Infrastructure Provisioning
```bash
terraform init
terraform plan
terraform apply
```
Creates all Azure resources using Infrastructure as Code principles.

### Phase 3: CI/CD Pipeline
GitHub Actions workflow triggered on push to main branch:
1. Build and test application
2. Create Docker image
3. Push image to Azure Container Registry
4. Deploy to Kubernetes cluster on Azure VM
5. Health checks and validation

## Challenges and Solutions

### Challenge 1: ImagePullBackOff Error
**Problem**: Kubernetes pods couldn't pull Docker images from ACR
**Root Cause**: VM not authenticated with Azure Container Registry
**Solution**: 
- Created Kubernetes docker-registry secret with ACR credentials
- Updated deployment to use `imagePullSecrets`
- Commands used:
```bash
az acr credential show --name <acr-name>
kubectl create secret docker-registry acr-secret \
  --docker-server=<acr-name>.azurecr.io \
  --docker-username=<username> \
  --docker-password=<password> \
  -n todo-app
```

### Challenge 2: VM Resource Constraints
**Problem**: Deployment timeouts and TLS handshake failures
**Root Cause**: B1s VM (1 vCPU, 1GB RAM) insufficient for K3s + image pulling
**Solution**: Upgraded to B2s VM (2 vCPUs, 4GB RAM)
```bash
az vm deallocate --resource-group rg-todo-app --name vm-todo-k8s
az vm resize --resource-group rg-todo-app --name vm-todo-k8s --size Standard_B2s
az vm start --resource-group rg-todo-app --name vm-todo-k8s
```

### Challenge 3: Template Not Found Error (HTTP 500)
**Problem**: Flask returning 500 error - `jinja2.exceptions.TemplateNotFound: index.html`
**Root Cause**: HTML template in wrong directory structure
**Solution**: 
- Moved `index.html` from root directory to `templates/` folder
- Flask requires templates in `/templates/` subdirectory by convention
- Rebuilt Docker image with correct file structure

### Challenge 4: SSH Authentication in CI/CD
**Problem**: GitHub Actions couldn't SSH into VM for deployment
**Root Cause**: Host key verification prompt blocking automated deployment
**Solution**: Added SSH options to skip host key checking:
```yaml
ssh -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null -i ssh_key
```

## Key Learning Points

### Technical Skills Demonstrated
- Infrastructure as Code with Terraform
- Container orchestration with Kubernetes
- CI/CD pipeline design and implementation
- Cloud resource management and cost optimization
- Troubleshooting containerized applications
- Security configuration (SSH keys, ACR authentication)

### DevOps Best Practices Applied
- **Progressive Deployment**: Manual testing before automation
- **Infrastructure Versioning**: All infrastructure defined in code
- **Secrets Management**: Proper handling of credentials and keys
- **Health Checks**: Application and infrastructure monitoring
- **Error Handling**: Comprehensive logging and debugging

## Deployment Commands Reference

### Initial Setup
```bash
# Generate SSH keys
ssh-keygen -t rsa -b 4096 -f ~/.ssh/id_rsa

# Deploy infrastructure
terraform init
terraform plan
terraform apply

# Test SSH connection
ssh -i ~/.ssh/id_rsa azureuser@<VM_IP>
```

### Troubleshooting Commands
```bash
# Check pod status
kubectl get pods -n todo-app

# View pod logs
kubectl logs -f deployment/todo-app -n todo-app

# Describe pod for detailed info
kubectl describe pod <pod-name> -n todo-app

# Check ACR authentication
az acr credential show --name <acr-name>

# Test application health
curl http://<VM_IP>:30080/health
```

### Kubernetes Management
```bash
# Restart deployment
kubectl rollout restart deployment/todo-app -n todo-app

# Delete and recreate pods
kubectl delete pods --all -n todo-app

# Apply updated manifests
kubectl apply -f k8s-manifests.yml
```

## GitHub Secrets Required
- `AZURE_CREDENTIALS`: Service principal JSON for Azure authentication
- `VM_SSH_PRIVATE_KEY`: Private SSH key for VM access

## Access Information
- **Application URL**: `http://<VM_PUBLIC_IP>:30080`
- **Health Check**: `http://<VM_PUBLIC_IP>:30080/health`
- **VM SSH**: `ssh -i ~/.ssh/id_rsa azureuser@<VM_PUBLIC_IP>`

## Future Enhancements
- Add Prometheus and Grafana for monitoring
- Implement HTTPS with SSL certificates
- Add database persistence with Azure Database
- Implement horizontal pod autoscaling
- Add integration tests in CI/CD pipeline
- Set up log aggregation and alerting

## Project Timeline
- **Day 1**: Application development and local testing
- **Day 2**: Infrastructure setup with Terraform
- **Day 3**: Kubernetes deployment and troubleshooting
- **Day 4**: CI/CD pipeline implementation and fixes
- **Total Time**: 4 days with evening work sessions

This project successfully demonstrates end-to-end cloud application deployment using modern DevOps practices and provides hands-on experience with real-world challenges and solutions.