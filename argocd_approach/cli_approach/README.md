# Apache Application Deployment (CLI Approach)

This directory contains the Kubernetes manifests for deploying an Apache web server cluster using the ArgoCD command-line interface (CLI).

## 🚀 Deployment Architecture

The application consists of a multi-replica deployment serving an Apache HTTP server, exposed to the public internet via a Google Cloud Platform (GCP) network Passthrough Load Balancer.

- **Deployment Name:** `apache`
- **Replicas:** 4 
- **Image:** `httpd:2.4`
- **Service Type:** `LoadBalancer` (Exposed on Port 80)

---

## 🛠️ Prerequisites & Setup

Before running the deployment command, ensure your Cloud Shell environment is authenticated and connected to the ArgoCD API server.

### 1. Establish Network Tunnel to ArgoCD
Since ArgoCD runs inside an isolated private network, open a secure background port-forward tunnel:

```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443 > /dev/null 2>&1 &
```

## 2. Fetch Admin Password & Authenticate CLI
Extract the auto-generated cluster secret and authenticate your local argocd CLI binary:

```
# Get and decode password
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo ""

# Log in to local session
argocd login 127.0.0.1:8080 --insecure --username admin
```

## 📦 How to Deploy via ArgoCD CLI
Execute the following tracking command to declare, initialize, and sync your GitOps repository application:

```
argocd app create apache-app \
  --server 127.0.0.1:8080 \
  --insecure \
  --repo https://github.com/arvindgupta-cloud/deploy-portfolio-using-argocd.git \
  --revision main \
  --path cli_approach \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace default \
  --sync-policy automated \
  --self-heal \
  --auto-prune
```


