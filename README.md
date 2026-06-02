# deploy-portfolio-using-argocd

A modern portfolio website deployment using **ArgoCD**, **Kubernetes**, and **Docker**, showcasing cloud infrastructure best practices.

## 🚀 Overview

This repository contains a fully containerized portfolio application deployed on Kubernetes using ArgoCD for GitOps-style continuous deployment. It demonstrates:

- **Container-first approach** with Docker & Nginx
- **Infrastructure as Code** with Kubernetes manifests
- **GitOps workflow** using ArgoCD for automated deployments
- **Scalable architecture** with Kubernetes deployments
- **Modern portfolio website** built with HTML, Tailwind CSS, and JavaScript

## 📁 Project Structure

```
deploy-portfolio-using-argocd/
├── app/
│   ├── index.html           # Portfolio website (main page)
│   ├── Dockerfile           # Container image definition
│   ├── image/              # Assets (favicon, images)
│   └── css/                # Stylesheets
├── k8s/
│   ├── namespace.yaml      # Kubernetes namespace
│   ├── deployment.yaml     # Deployment configuration
│   └── service.yaml        # LoadBalancer service
├── README.md               # This file
└── .github/workflows/      # CI/CD workflows (if present)
```

## 🛠️ Tech Stack

| Component | Technology |
|-----------|-----------|
| **Frontend** | HTML5, Tailwind CSS, JavaScript |
| **Containerization** | Docker, Nginx (Alpine) |
| **Orchestration** | Kubernetes (K8s) |
| **Deployment** | ArgoCD (GitOps) |
| **Web Server** | Nginx Alpine |
| **Image Registry** | Docker Hub |

## 📋 Prerequisites

- **Docker** (for building images)
- **Kubernetes cluster** (K8s 1.20+)
- **ArgoCD** installed on your cluster
- **kubectl** configured
- **Docker Hub account** (or alternative registry)

## 🚢 Getting Started

### 1. Clone the Repository

```bash
git clone https://github.com/arvindgupta-cloud/deploy-portfolio-using-argocd.git
cd deploy-portfolio-using-argocd
```

### 2. Build Docker Image

```bash
cd app
docker build -t <your-username>/portfolio:latest .
docker push <your-username>/portfolio:latest
```

Replace `<your-username>` with your Docker Hub username.

### 3. Update Image Tag in Deployment

Edit `k8s/deployment.yaml` and update the image reference:

```yaml
image: <your-username>/portfolio:IMAGE_TAG
```

### 4. Create Namespace

```bash
kubectl apply -f k8s/namespace.yaml
```

### 5. Deploy Manually (Without ArgoCD)

```bash
kubectl apply -f k8s/
```

Or deploy using ArgoCD (see ArgoCD Setup below).

### 6. Access the Application

```bash
kubectl port-forward -n portfolio svc/portfolio-service 8080:80
```

Visit `http://localhost:8080` in your browser.

## 🔄 ArgoCD Setup

### Install ArgoCD

```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

### Access ArgoCD UI

```bash
kubectl port-forward -n argocd svc/argocd-server 8443:443
```

Get the initial password:

```bash
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```

Visit `https://localhost:8443` and login with:
- **Username:** `admin`
- **Password:** (from above)

### Create ArgoCD Application

Create an `argocd-app.yaml`:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: portfolio-app
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/arvindgupta-cloud/deploy-portfolio-using-argocd
    targetRevision: main
    path: k8s
  destination:
    server: https://kubernetes.default.svc
    namespace: portfolio
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
      - CreateNamespace=true
```

Apply it:

```bash
kubectl apply -f argocd-app.yaml
```

## 📦 Kubernetes Manifests

### Namespace (`k8s/namespace.yaml`)
Creates a dedicated namespace for the portfolio application.

### Deployment (`k8s/deployment.yaml`)
- **Replicas:** 1 (can be scaled)
- **Image:** `ag126667/portfolio:IMAGE_TAG`
- **Port:** 80 (HTTP)
- **RevisionHistoryLimit:** 1 (minimal rollback history)

### Service (`k8s/service.yaml`)
- **Type:** LoadBalancer
- **Port:** 80
- **TargetPort:** 80
- **Selector:** `app: portfolio`

## 🐳 Docker Configuration

### Dockerfile

```dockerfile
FROM nginx:alpine
COPY . /usr/share/nginx/html/
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

- **Base Image:** `nginx:alpine` (minimal, ~40MB)
- **Static Content:** Copied to Nginx default directory
- **Port:** 80 (HTTP)
- **Entrypoint:** Runs Nginx in foreground mode

## 🌐 Portfolio Features

The portfolio website includes:

- **Responsive Design** - Mobile-first, works on all devices
- **Dark/Light Theme** - Theme toggle functionality
- **Interactive Terminal** - Animated terminal window in hero section
- **Sections:**
  - Hero introduction with CTA buttons
  - About Me section
  - Experience timeline
  - Skills & Tools showcase
  - Featured Projects
  - Certifications
  - Contact information
- **SEO Optimized** - Meta tags, JSON-LD structured data
- **Accessibility** - ARIA labels, semantic HTML

## 📊 Deployment Workflow

```
GitHub Push
    ↓
Docker Image Build
    ↓
Push to Docker Hub
    ↓
Update k8s/deployment.yaml
    ↓
ArgoCD Detects Changes
    ↓
Auto-sync to Kubernetes
    ↓
Rolling Update Complete
```

## 🔧 Managing Updates

### Update Portfolio Content

1. Edit `app/index.html`
2. Commit and push to GitHub
3. Rebuild Docker image: `docker build -t <username>/portfolio:v2 .`
4. Push: `docker push <username>/portfolio:v2`
5. Update `k8s/deployment.yaml` with new image tag
6. ArgoCD automatically syncs the changes

### Scale Replicas

```bash
kubectl scale deployment portfolio-app -n portfolio --replicas=3
```

Or update `k8s/deployment.yaml`:

```yaml
spec:
  replicas: 3
```

### Check Deployment Status

```bash
kubectl get deployments -n portfolio
kubectl get pods -n portfolio
kubectl get svc -n portfolio
```

### View Logs

```bash
kubectl logs -n portfolio deployment/portfolio-app
```

## 📈 Performance Tips

- **Image optimization:** Using Alpine base (only ~40MB)
- **Caching:** Configure appropriate HTTP cache headers
- **CDN:** Deploy behind a CDN for global distribution
- **Horizontal Scaling:** Increase replicas based on load

## 🔐 Security Considerations

- [ ] Use read-only root filesystem: `readOnlyRootFilesystem: true`
- [ ] Set resource limits and requests
- [ ] Use non-root user in Dockerfile
- [ ] Implement network policies
- [ ] Enable RBAC in Kubernetes
- [ ] Use secrets for sensitive data
- [ ] Regular image scanning for vulnerabilities

Example security update to Dockerfile:

```dockerfile
FROM nginx:alpine
RUN addgroup -g 101 nginx && \
    adduser -S -D -H -u 101 -h /var/cache/nginx -s /sbin/nologin -G nginx -g nginx nginx
COPY --chown=nginx:nginx . /usr/share/nginx/html/
USER nginx
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

## 🚨 Troubleshooting

### Pod not starting?
```bash
kubectl describe pod -n portfolio <pod-name>
kubectl logs -n portfolio <pod-name>
```

### Service not accessible?
```bash
kubectl get svc -n portfolio
kubectl get endpoints -n portfolio
```

### ArgoCD not syncing?
- Check repository credentials
- Verify the Git branch matches `targetRevision`
- Check ArgoCD application status: `argocd app get portfolio-app`

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch: `git checkout -b feature/amazing-feature`
3. Commit changes: `git commit -m 'Add amazing feature'`
4. Push to branch: `git push origin feature/amazing-feature`
5. Open a Pull Request

## 📄 License

This project is open source and available under the MIT License.

## 👤 Author

**Arvind Gupta**
- 🌐 Website: [arvindgupta.cloud](https://arvindgupta.cloud)
- 💼 LinkedIn: [arvindguptag](https://linkedin.com/in/arvindguptag)
- 📧 Email: [arvindgupta.cloud@gmail.com](mailto:arvindgupta.cloud@gmail.com)
- 📍 Location: Pune, India

## 🎓 Certifications

- 4x Google Cloud Certified
- Cloud Architect & DevOps Engineer
- 100+ Cloud Labs Developed

## 📚 Resources

- [Kubernetes Documentation](https://kubernetes.io/docs/)
- [ArgoCD Documentation](https://argo-cd.readthedocs.io/)
- [Docker Documentation](https://docs.docker.com/)
- [Nginx Documentation](https://nginx.org/en/docs/)

---

**Last Updated:** June 2, 2026
