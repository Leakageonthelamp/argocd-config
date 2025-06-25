# ArgoCD Configuration Project

This repository contains Kubernetes manifests and ArgoCD application configurations for deploying a sample nginx application using GitOps principles.

## 🎯 Project Overview

This project demonstrates a GitOps workflow using ArgoCD to manage Kubernetes deployments. It includes:
- Base Kubernetes manifests for a nginx application
- Environment-specific overlays using Kustomize
- ArgoCD Application configuration for automated deployment

## 📁 Project Structure

```
argocd-config/
├── application.yaml          # ArgoCD Application definition
├── base/                     # Base Kubernetes manifests
│   ├── deployment.yaml       # Nginx deployment
│   ├── service.yaml          # Nginx service
│   └── kustomization.yaml    # Base kustomization config
├── overlays/                 # Environment-specific configurations
│   └── staging/              # Staging environment
│       ├── kustomization.yaml # Staging kustomization
│       └── patch.yaml        # Staging-specific patches
└── README.md                 # This file
```

## 🚀 Getting Started

### Prerequisites

- Kubernetes cluster with ArgoCD installed
- `kubectl` configured to access your cluster
- `argocd` CLI tool (optional, for command-line management)

### Installation

1. **Clone this repository:**
   ```bash
   git clone <your-repo-url>
   cd argocd-config
   ```

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
   ```

## 🔧 Configuration Details

### Base Configuration (`base/`)

The base directory contains the core Kubernetes manifests:
- **deployment.yaml**: Defines a nginx deployment with 1 replica
- **service.yaml**: Exposes the nginx deployment on port 80
- **kustomization.yaml**: References the base resources

### Environment Overlays (`overlays/`)

Environment-specific configurations that customize the base manifests:

#### Staging Environment (`overlays/staging/`)
- **kustomization.yaml**: References the base configuration and applies patches
- **patch.yaml**: Increases replicas from 1 to 3 for staging

### ArgoCD Application (`application.yaml`)

The ArgoCD Application configuration defines:
- **Source**: Git repository and path to staging overlay
- **Destination**: Kubernetes cluster and namespace
- **Sync Policy**: Automated sync with pruning and self-healing enabled

## 🛠️ Customization

### Adding New Environments

1. Create a new overlay directory:
   ```bash
   mkdir -p overlays/production
   ```

2. Create environment-specific files:
   ```yaml
   # overlays/production/kustomization.yaml
   apiVersion: kustomize.config.k8s.io/v1beta1
   kind: Kustomization
   bases:
     - ../../base
   patchesStrategicMerge:
     - patch.yaml
   ```

3. Add environment-specific patches:
   ```yaml
   # overlays/production/patch.yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: nginx-deployment
   spec:
     replicas: 5  # More replicas for production
   ```

4. Create a new ArgoCD Application for production:
   ```yaml
   # production-application.yaml
   apiVersion: argoproj.io/v1alpha1
   kind: Application
   metadata:
     name: my-app-production
     namespace: argocd
   spec:
     project: default
     source:
       repoURL: <YOUR_GIT_REPO_URL>
       path: overlays/production
       targetRevision: HEAD
     destination:
       server: https://kubernetes.default.svc
       namespace: production
     syncPolicy:
       automated:
         prune: true
         selfHeal: true
       syncOptions:
       - CreateNamespace=true
   ```

### Modifying the Application

To modify the nginx application:
1. Update the base manifests in `base/`
2. Create environment-specific patches in `overlays/<environment>/`
3. Commit and push changes
4. ArgoCD will automatically sync the changes

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
```

### Common Issues

1. **Application not syncing**: Check if the Git repository URL is correct and accessible
2. **Resources not created**: Verify the namespace exists or `CreateNamespace=true` is set
3. **Image pull errors**: Ensure the nginx image is accessible from your cluster

## 📚 Additional Resources

- [ArgoCD Documentation](https://argo-cd.readthedocs.io/)
- [Kustomize Documentation](https://kustomize.io/)
- [Kubernetes Documentation](https://kubernetes.io/docs/)

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
