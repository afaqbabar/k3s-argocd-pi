# 🚀 K3s Raspberry Pi Cluster with Argo CD + MetalLB

This repository contains the GitOps configuration for a **K3s cluster on Raspberry Pi 4**, managed by **Argo CD**.  
It bootstraps **MetalLB** for LoadBalancer services and deploys a sample **Nginx app**.  
The **Argo CD UI** is always available at **`https://192.168.178.212`**.

---

## 📂 Repository Structure

```
k3s-argocd-pi/
├── apps/
│   ├── argocd/                     # ArgoCD service override (LB on 192.168.178.212)
│   │   └── argocd-server-service.yaml
│   ├── metallb/                    # MetalLB IP pool + L2Advertisement
│   │   └── metallb-config.yaml
│   └── nginx/                      # Nginx test app
│       ├── deployment.yaml
│       └── service.yaml
├── argocd-apps/                    # ArgoCD Applications
│   ├── argocd-app.yaml             # ArgoCD manages its own LB service
│   ├── metallb-app.yaml            # Deploys MetalLB
│   ├── nginx-app.yaml              # Deploys Nginx
│   └── root-app.yaml               # Root App (App of Apps)
└── projects/                       # Reserved for ArgoCD projects
```

---

## ⚙️ Cluster Information

- **Hardware**: Raspberry Pi 4  
- **OS**: Raspberry Pi OS  
- **Kubernetes**: K3s  
- **GitOps**: Argo CD  
- **Load Balancer**: MetalLB (L2 mode)  
- **Container Runtime**: containerd  
- **IP Pool**: `192.168.178.210–220`  
- **Fixed IPs**:  
  - ArgoCD → `192.168.178.212`  
  - Apps → dynamic IPs from pool  

---

## 🚀 Getting Started

### 1. Install K3s
```bash
curl -sfL https://get.k3s.io | sh -
kubectl get nodes
```

### 2. Install Argo CD
```bash
kubectl create namespace argocd
kubectl apply -n argocd   -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

### 3. Bootstrap ArgoCD with Root App
```bash
kubectl apply -f argocd-apps/root-app.yaml -n argocd
```

ArgoCD will then sync:
- **MetalLB** (IP pool `192.168.178.210–220`)  
- **ArgoCD service override** (UI on `192.168.178.212`)  
- **Nginx test app**  

---

## 🌐 Accessing Applications

### Argo CD UI
```
https://192.168.178.212
```

Default admin password:
```bash
kubectl -n argocd get secret argocd-initial-admin-secret   -o jsonpath="{.data.password}" | base64 -d
```

### Nginx App
Check assigned IP:
```bash
kubectl get svc nginx-service
```

Example:
```
nginx-service   LoadBalancer   10.43.219.63   192.168.178.213   80:31345/TCP   2m
```

Then open in your browser:
```
http://192.168.178.213
```

---

## 🛠 Adding New Applications

1. Add manifests in `apps/<app-name>/`  
2. Create an ArgoCD app in `argocd-apps/<app-name>-app.yaml`  
3. Commit + push → ArgoCD syncs automatically  

---

## 🔧 Troubleshooting

- **ArgoCD app shows “Unknown”** → check `repoURL` and `targetRevision` in your Application manifests.  
- **No external IP assigned** → make sure MetalLB is installed and the IP pool (`192.168.178.210–220`) doesn’t overlap with router DHCP.  
- **Can’t reach ArgoCD UI** → confirm that the `argocd-server` Service is `LoadBalancer` with `loadBalancerIP: 192.168.178.212`.  
- **Cluster clock skew** → if TLS errors occur, verify time on the Pi:
  ```bash
  date
  ```

---

✅ With this setup:  
- ArgoCD always lives at a **fixed IP (`192.168.178.212`)**  
- Apps get IPs dynamically (unless you set `loadBalancerIP`)  
- Everything is managed through **GitOps**  
