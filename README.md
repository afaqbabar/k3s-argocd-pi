# K3s Raspberry Pi Cluster with Argo CD

This repository contains the GitOps configuration for a K3s cluster running on Raspberry Pi 4, managed by Argo CD.

## Repository Structure

```
k3s-argocd-pi/
├── README.md
├── apps/                    # Application manifests
│   ├── nginx/              # Nginx test application
│   └── monitoring/         # Future monitoring stack
├── argocd-apps/           # Argo CD Application definitions
│   ├── nginx-app.yaml     # Nginx application
│   └── root-app.yaml      # Root application (App of Apps)
└── projects/              # Argo CD project definitions
```

## Applications

- **nginx**: Simple nginx deployment for testing GitOps workflow
- **monitoring**: (Future) Prometheus/Grafana stack for cluster monitoring

## Getting Started

1. Install the root application:
   ```bash
   kubectl apply -f argocd-apps/root-app.yaml
   ```

2. Access Argo CD UI:
   ```bash
   kubectl port-forward svc/argocd-server -n argocd 8080:443
   ```
   
3. Open https://localhost:8080 in your browser

## Adding New Applications

1. Create application manifests in `apps/<app-name>/`
2. Create Argo CD application definition in `argocd-apps/<app-name>-app.yaml`
3. Commit and push - Argo CD will automatically sync

## Cluster Information

- **Platform**: Raspberry Pi 4
- **OS**: Raspberry Pi OS
- **Kubernetes**: K3s
- **GitOps**: Argo CD
- **Container Runtime**: containerd
