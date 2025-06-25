# ArgoCD Configuration Project

This repository contains Kubernetes manifests and ArgoCD application configurations for deploying a Golang HTTP application using GitOps principles.

## 🎯 Project Overview

This project demonstrates a GitOps workflow using ArgoCD to manage Kubernetes deployments. It includes:
- Base Kubernetes manifests for a Golang HTTP application
- Environment-specific overlays using Kustomize (staging, production, and Docker Desktop)
- ArgoCD Application configuration for automated deployment
- Properly configured ingress with TLS support (and localhost for Docker Desktop)

## 📁 Project Structure

```
argocd-config/
├── application.yaml          # ArgoCD Application definition
├── base/                     # Base Kubernetes manifests
│   ├── deployment.yaml       # Golang HTTP deployment
│   ├── service.yaml          # Golang HTTP service
│   ├── ingress.yaml          # Ingress configuration with localhost
│   └── kustomization.yaml    # Base kustomization config
├── overlays/                 # Environment-specific configurations
│   ├── staging/              # Staging environment
│   │   ├── kustomization.yaml # Staging kustomization
│   │   └── patch.yaml        # Staging-specific patches (staging.localhost)
│   ├── production/           # Production environment
│   │   ├── kustomization.yaml # Production kustomization
│   │   └── patch.yaml        # Production-specific patches (prod.localhost)
│   └── docker-desktop/       # Docker Desktop environment
│       ├── kustomization.yaml # Docker Desktop kustomization
│       └── patch.yaml        # Docker Desktop patches (localhost)
└── README.md                 # This file
```

## 🚀 Getting Started

### Prerequisites

- Kubernetes cluster with ArgoCD installed
- NGINX Ingress Controller installed
- `kubectl` configured to access your cluster
- `argocd` CLI tool (optional, for command-line management)

### For Docker Desktop Users

If you're using Docker Desktop Kubernetes, the configuration is already set up to use `localhost` domains:

1. **Install NGINX Ingress Controller for Docker Desktop:**
   ```bash
   kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.8.0/deploy/static/provider/baremetal/deploy.yaml
   ```

2. **Update your hosts file** (optional, for subdomain support):
   ```bash
   # On macOS/Linux, edit /etc/hosts
   # On Windows, edit C:\Windows\System32\drivers\etc\hosts
   
   127.0.0.1 localhost
   127.0.0.1 staging.localhost
   127.0.0.1 prod.localhost
   ```

3. **Deploy using the Docker Desktop overlay:**
   ```bash
   # Update application.yaml to use docker-desktop overlay
   # Then apply:
   kubectl apply -f application.yaml
   ```

4. **Access your application:**
   - Base/Production: http://localhost
   - Staging: http://staging.localhost
   - Production (with TLS): https://prod.localhost

### For Production/Cloud Deployments

1. **Update the ingress configuration:**
   Edit the ingress files and replace the localhost domains with your actual domains:
   - `base/ingress.yaml` - Base configuration with your domain
   - `overlays/staging/patch.yaml` - Staging configuration (staging.your-domain.com)
   - `overlays/production/patch.yaml` - Production configuration (your-domain.com)

2. **Update the ArgoCD Application configuration:**
   Edit `application.yaml` and replace `<YOUR_GIT_REPO_URL>` with your actual Git repository URL:
   ```yaml
   source:
     repoURL: https://github.com/your-username/your-repo.git
   ```

3. **Apply the ArgoCD Application:**
   ```bash
   kubectl apply -f application.yaml
   ```

4. **Verify the deployment:**
   ```bash
   kubectl get applications -n argocd
   kubectl get pods -n staging
   kubectl get ingress -n staging
   ```

## 🔧 Configuration Details

### Base Configuration (`base/`)

The base directory contains the core Kubernetes manifests:
- **deployment.yaml**: Defines a Golang HTTP deployment with 1 replica
- **service.yaml**: Exposes the Golang HTTP deployment on port 80
- **ingress.yaml**: Ingress configuration optimized for localhost (Docker Desktop)
- **kustomization.yaml**: References the base resources

### Ingress Configuration

The ingress is configured for different environments:

#### Docker Desktop (localhost)
- **Base**: `localhost` - Simple HTTP access
- **Staging**: `staging.localhost` - Staging environment
- **Production**: `prod.localhost` - Production-like testing

#### Production/Cloud
- **Base**: `your-domain.com` - Your actual domain
- **Staging**: `staging.your-domain.com` - Staging subdomain
- **Production**: `your-domain.com` - Production domain with TLS

#### Key Ingress Features:
- Environment-specific host configurations
- TLS support for production environments
- Optimized proxy timeouts
- Configurable body size limits
- Support for cert-manager integration

### Environment Overlays (`overlays/`)

Environment-specific configurations that customize the base manifests:

#### Docker Desktop Environment (`overlays/docker-desktop/`)
- **kustomization.yaml**: References the base configuration and applies patches
- **patch.yaml**: 
  - Single replica for local development
  - Simplified ingress without TLS
  - Uses `localhost` for easy access

#### Staging Environment (`overlays/staging/`)
- **kustomization.yaml**: References the base configuration and applies patches
- **patch.yaml**: 
  - Increases replicas from 1 to 2
  - Uses `staging.localhost` for Docker Desktop or `staging.your-domain.com` for production
  - No TLS configuration for easier development

#### Production Environment (`overlays/production/`)
- **kustomization.yaml**: References the base configuration and applies patches
- **patch.yaml**:
  - Increases replicas from 1 to 3 for high availability
  - Uses `prod.localhost` for Docker Desktop or `your-domain.com` for production
  - Maintains TLS configuration for security
  - Includes cert-manager integration

### ArgoCD Application (`application.yaml`)

The ArgoCD Application configuration defines:
- **Source**: Git repository and path to the desired overlay
- **Destination**: Kubernetes cluster and namespace
- **Sync Policy**: Automated sync with pruning and self-healing enabled

## 🛠️ Customization

### Switching Between Environments

To switch between different environments, update the `application.yaml`:

```yaml
# For Docker Desktop development
source:
  path: overlays/docker-desktop

# For staging
source:
  path: overlays/staging

# For production
source:
  path: overlays/production
```

### Adding New Environments

1. Create a new overlay directory:
   ```bash
   mkdir -p overlays/development
   ```

2. Create environment-specific files:
   ```yaml
   # overlays/development/kustomization.yaml
   apiVersion: kustomize.config.k8s.io/v1beta1
   kind: Kustomization
   bases:
     - ../../base
   patchesStrategicMerge:
     - patch.yaml
   ```

3. Add environment-specific patches:
   ```yaml
   # overlays/development/patch.yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: golang-http-deployment
   spec:
     replicas: 1  # Single replica for development
   ---
   apiVersion: networking.k8s.io/v1
   kind: Ingress
   metadata:
     name: golang-http-ingress
     annotations:
       nginx.ingress.kubernetes.io/rewrite-target: /
   spec:
     ingressClassName: nginx
     rules:
     - host: dev.localhost  # For Docker Desktop
       http:
         paths:
         - path: /
           pathType: Prefix
           backend:
             service:
               name: golang-http-service
               port:
                 number: 80
   ```

### TLS Certificate Management

For production environments, you can use cert-manager for automatic certificate management:

1. **Install cert-manager** (if not already installed):
   ```bash
   kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.13.0/cert-manager.yaml
   ```

2. **Configure a ClusterIssuer** for Let's Encrypt:
   ```yaml
   apiVersion: cert-manager.io/v1
   kind: ClusterIssuer
   metadata:
     name: letsencrypt-prod
   spec:
     acme:
       server: https://acme-v02.api.letsencrypt.org/directory
       email: your-email@example.com
       privateKeySecretRef:
         name: letsencrypt-prod
       solvers:
       - http01:
           ingress:
             class: nginx
   ```

3. **The ingress will automatically request certificates** when the annotation is present:
   ```yaml
   annotations:
     cert-manager.io/cluster-issuer: "letsencrypt-prod"
   ```

### Modifying the Application

To modify the Golang HTTP application:
1. Update the base manifests in `base/`
2. Create environment-specific patches in `overlays/<environment>/`
3. Update domain names in ingress configurations
4. Commit and push changes
5. ArgoCD will automatically sync the changes

## 🔍 Monitoring and Troubleshooting

### Check Application Status
```bash
# View ArgoCD applications
kubectl get applications -n argocd

# Get detailed application info
kubectl describe application my-app-staging -n argocd

# View application logs
kubectl logs -n argocd deployment/argocd-application-controller
```

### Check Kubernetes Resources
```bash
# View resources in staging namespace
kubectl get all -n staging

# Check pod status
kubectl get pods -n staging

# View service details
kubectl get svc -n staging

# Check ingress status
kubectl get ingress -n staging
kubectl describe ingress golang-http-ingress -n staging
```

### Docker Desktop Specific Commands
```bash
# Check if NGINX Ingress Controller is running
kubectl get pods -n ingress-nginx

# Get the external IP (should be localhost for Docker Desktop)
kubectl get svc -n ingress-nginx

# Test localhost access
curl http://localhost
curl http://staging.localhost
curl http://prod.localhost
```

### Check TLS Certificate Status
```bash
# View certificates (if using cert-manager)
kubectl get certificates -n staging
kubectl get certificaterequests -n staging

# Check certificate details
kubectl describe certificate golang-http-tls -n staging
```

### Common Issues

1. **Application not syncing**: Check if the Git repository URL is correct and accessible
2. **Resources not created**: Verify the namespace exists or `CreateNamespace=true` is set
3. **Image pull errors**: Ensure the Golang HTTP image is accessible from your cluster
4. **Ingress not working**: 
   - Verify NGINX Ingress Controller is installed
   - For Docker Desktop: Check if localhost resolves correctly
   - For production: Check if the domain resolves to your cluster's external IP
   - Ensure TLS certificates are properly configured
5. **TLS certificate issues**:
   - Verify cert-manager is installed and running
   - Check ClusterIssuer configuration
   - Ensure domain ownership and DNS configuration
6. **Docker Desktop specific issues**:
   - Make sure Docker Desktop has enough resources allocated
   - Verify the ingress controller is using the correct service type
   - Check if localhost ports are not conflicting with other services

## 📚 Additional Resources

- [ArgoCD Documentation](https://argo-cd.readthedocs.io/)
- [Kustomize Documentation](https://kustomize.io/)
- [Kubernetes Ingress Documentation](https://kubernetes.io/docs/concepts/services-networking/ingress/)
- [NGINX Ingress Controller](https://kubernetes.github.io/ingress-nginx/)
- [cert-manager Documentation](https://cert-manager.io/docs/)
- [Docker Desktop Kubernetes](https://docs.docker.com/desktop/kubernetes/)

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Test the deployment
5. Submit a pull request

## 📄 License

This project is licensed under the MIT License - see the LICENSE file for details.

---

**Note**: Remember to replace `<YOUR_GIT_REPO_URL>` with your actual Git repository URL before deploying.
