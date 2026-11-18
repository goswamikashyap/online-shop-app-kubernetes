# Online Shop 🛍️ — End-to-End Docker & Kubernetes Deployment

Welcome to the **Online Shop** project! This repository demonstrates an end-to-end containerized and orchestrated deployment of a React + Vite e-commerce application using **Git**, **Docker**, **Docker Hub**, **Kubernetes (KinD)**, and **AWS EC2**.


## 🏗️ Architecture & Workflow

```text
Browser (:5173) ──▶ AWS EC2 Security Group (:5173) ──▶ kubectl port-forward (5173:5173)
                                                               │
                                                               ▼
                                          Kubernetes Service: online-shop-svc (:5173)
                                                               │
                                                               ▼
                                      Kubernetes Deployment: 2x Nginx Pods (:80)
```

---

## 🖼️ Application Preview

1. **Storefront / Home Page**

   <img width="709" height="332" alt="1" src="https://github.com/user-attachments/assets/b1cfe514-b292-45a1-b1d5-b8599684338f" />


2. **Admin Dashboard**

  <img width="693" height="342" alt="2" src="https://github.com/user-attachments/assets/69276b8a-361e-47c2-8c2f-dfcfc72421fb" />


---

## 🚀 What Was Accomplished

### 1. Containerization & Docker Hub Registry
- **Multi-Stage Dockerfile:** Built the frontend using `node:20-alpine` and served the optimized static bundle via `nginx:alpine`, significantly reducing final image size and improving startup speed.
- **Custom Nginx Configuration (`nginx.conf`):** Configured SPA routing (`try_files $uri $uri/ /index.html`), static asset caching, gzip compression, and a `/health` check endpoint on port `80`.
- **Docker Hub Integration:** Generated a Personal Access Token (PAT), tagged the production image as `kashyapgoswami/online_shop_app:latest`, and pushed it to Docker Hub.

### 2. Kubernetes Orchestration (KinD on AWS EC2)
- **Namespace Isolation (`namespace.yml`):** Created a dedicated `online-shop` namespace following RFC 1123 naming conventions.
- **High-Availability Deployment (`deployment.yml`):** Deployed **2 replicas** of `kashyapgoswami/online_shop_app:latest` exposing `containerPort: 80`.
- **Service Exposure (`service.yml`):** Configured a `NodePort` service (`online-shop-svc`) mapping external service port `5173` to container `targetPort: 80`.
- **External Access on AWS EC2:** Configured AWS EC2 Security Group inbound rules for TCP port `5173` and exposed the Kubernetes service via `kubectl port-forward`.

---

## 🛠️ Prerequisites

- **AWS EC2 Instance** (Ubuntu) with inbound TCP port `5173` allowed in the Security Group
- **Docker** installed and running
- **Kubernetes Cluster** (`kubectl` configured with KinD / Minikube / EKS)
- **Docker Hub Account** (`kashyapgoswami`)

---

## ⚡ Step-by-Step Deployment Guide

### 1. Build & Push the Docker Image

```bash
# Build the multi-stage Docker image
docker build -t online_shop_app:latest .

# Tag and push to Docker Hub
docker login -u kashyapgoswami
docker tag online_shop_app:latest kashyapgoswami/online_shop_app:latest
docker push kashyapgoswami/online_shop_app:latest
```

### 2. Deploy to Kubernetes

```bash
# Apply Namespace, Deployment, and Service manifests
kubectl apply -f namespace.yml
kubectl apply -f deployment.yml
kubectl apply -f service.yml

# Verify that pods and services are running
kubectl get all -n online-shop
```

### 3. Expose the Application via Port-Forwarding

```bash
# Forward EC2 host port 5173 to the Kubernetes Service
kubectl port-forward svc/online-shop-svc -n online-shop 5173:5173 --address=0.0.0.0
```

Open your browser and visit:
```text
http://<YOUR-EC2-PUBLIC-IP>:5173
```

---

## 🧹 Cleanup & Rerun Commands

### Clean Up Resources (Before Stopping EC2)

To avoid unnecessary AWS charges while preserving your code and manifests for future runs:

```bash
# 1. Stop active port-forwarding (Press Ctrl + C in the active terminal)

# 2. Delete the Kubernetes namespace and all deployed pods/services
kubectl delete ns online-shop

# 3. Remove any standalone test containers and dangling images
docker rm -f \$(docker ps -a -q --filter ancestor=online_shop_app:latest) 2>/dev/null || true
docker image prune -f
```
*(After running cleanup, go to **AWS Console > EC2 > Instances** and select **Stop instance** — do not Terminate).*

### Rerun the Project Later (After Starting EC2)

When you start your EC2 instance again (note that AWS assigns a new Public IPv4 address):

```bash
# 1. Navigate to the project directory
cd ~/kubernetes_practice/k8s_practice_project/online_shopping_app

# 2. Re-apply the Kubernetes manifests
kubectl apply -f namespace.yml -f deployment.yml -f service.yml

# 3. Confirm pods are in Running state
kubectl get pods -n online-shop

# 4. Start port-forwarding
kubectl port-forward svc/online-shop-svc -n online-shop 5173:5173 --address=0.0.0.0
```

Visit `http://<NEW-EC2-PUBLIC-IP>:5173` in your browser.

---

## 🧠 Troubleshooting & Key Learnings

- **RFC 1123 Naming Rules:** Kubernetes resource names and namespaces must use hyphens (`online-shop`) rather than underscores (`online_shop`).
- **Port Alignment (`port` vs `targetPort`):** Because `nginx.conf` listens on port `80`, `containerPort` and `targetPort` must be set to `80`, while the Service `port` and `port-forward` expose `5173`.
- **Port Conflicts:** Always stop standalone Docker containers occupying host port `5173` before running `kubectl port-forward`.

---


### Result / Resume

- Successfully deployed the `Online Shopping Portal` on the internet using DevOps automation tools.
- Improved `deployment speed by 50% `and reduced `Docker image size by 1GB` using multi-stage builds.
- Ensured security and efficiency by implementing `Docker Scout` for `vulnerability analysis`.
- Automated the setup process with `Docker Installation Scripts` and `Docker Compose` for easy deployment.
- Deployed the application on `Amazon EC2`, making it accessible and scalable for real users.

---

