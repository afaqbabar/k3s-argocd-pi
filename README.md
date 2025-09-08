# 🚀 K3s Raspberry Pi Cluster with Argo CD + MetalLB + Monitoring

This repository contains the GitOps configuration for a **K3s cluster on Raspberry Pi 4**, managed by **Argo CD**.  
It bootstraps **MetalLB** for LoadBalancer services, deploys **Argo CD**, a test **Nginx app**, and the **Prometheus + Grafana monitoring stack**.  

The cluster is accessible from your **laptop over LAN** with fixed IPs.

---

## 📂 Repository Structure

```
k3s-argocd-pi/
├── apps/
│   ├── argocd/                     # Argo CD service override (LB on 192.168.178.212)
│   ├── metallb/                    # MetalLB IP pool + L2Advertisement
│   ├── monitoring/                 # Helm values for kube-prometheus-stack
│   └── nginx/                      # Nginx test app
├── argocd-apps/                    # Argo CD Applications
│   ├── argocd-app.yaml             # Manages Argo CD service
│   ├── metallb-app.yaml            # Deploys MetalLB
│   ├── monitoring-app.yaml         # Deploys Prometheus + Grafana
│   ├── nginx-app.yaml              # Deploys Nginx test app
│   └── root-app.yaml               # Root App (App of Apps)
└── projects/                       # Reserved for Argo CD projects
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
  - Nginx → `192.168.178.211`  
  - Grafana → `192.168.178.213`  

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
- **Monitoring stack (Prometheus + Grafana)**  
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

### Grafana (Monitoring)
```
http://192.168.178.213
```
Login with:
- Username: `admin`
- Password:
  ```bash
  kubectl -n monitoring get secret monitoring-grafana     -o jsonpath="{.data.admin-password}" | base64 -d
  ```

### Nginx Test App
```
http://192.168.178.211
```

---

## 🛠 Adding New Applications

1. Add manifests in `apps/<app-name>/`  
2. Create an ArgoCD app in `argocd-apps/<app-name>-app.yaml`  
3. Commit + push → ArgoCD syncs automatically  

---

## 🔧 Troubleshooting

- **Service stuck at `<pending>`** → another service may be holding the IP, check:
  ```bash
  kubectl get svc -A -o wide | grep <IP>
  ```
- **Grafana unreachable** → flush stale ARP cache on laptop:
  ```bash
  sudo arp -d 192.168.178.213
  ```
- **ArgoCD app shows “Unknown”** → ensure `helm.values` is inline, not pointing to missing files.
- **No external IP assigned** → confirm MetalLB is running and pool `192.168.178.210–220` doesn’t overlap with router DHCP.

---

## ✅ Current State

- ArgoCD UI → **192.168.178.212**  
- Grafana → **192.168.178.213**  
- Nginx test app → **192.168.178.211**  
- All apps are managed via **GitOps** with drift prevention enabled.
