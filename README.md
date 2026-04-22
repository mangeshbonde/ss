# 🚀 StaticSite CI/CD Pipeline (Docker → Jenkins → Kubernetes)

## 📌 Project Overview

This project demonstrates a complete **CI/CD pipeline** for deploying a static website using:

* Docker (Containerization)
* Jenkins (CI Automation)
* Kubernetes (Deployment)
* AWS EC2 (Infrastructure)

The pipeline automates building, packaging, and deploying a static HTML application.

---

## 🎯 Objective

To build an automated pipeline where:

```
Code → Docker Image → Docker Hub → Kubernetes Deployment
```

---

## 🏗️ Project Structure

```
ss-cicd-devops-pipeline/
│
├── app/
│   └── index.html
│
├── Dockerfile
│
├── jenkins/
│   └── Jenkinsfile
│
├── k8s/
│   ├── deployment.yaml
│   ├── service.yaml
│   └── ingress.yaml
│
├── README.md
```

---

# ⚙️ Prerequisites Setup (EC2 + Docker + Kind + Jenkins)

This document covers all the required setup steps to prepare the environment for running the **StaticSite CI/CD Pipeline project**.

---

# ☁️ 1. Update System

```bash
sudo apt update
```

---

# 🐳 2. Install Docker

```bash
sudo apt install -y docker.io
```

---

## ▶️ Start & Enable Docker

```bash
sudo systemctl start docker
sudo systemctl enable docker
```

---

## 👤 Run Docker Without sudo

```bash
sudo usermod -aG docker $USER
newgrp docker
```

---

# 📦 3. Clone Setup Repository

```bash
git clone https://github.com/mangeshbonde/k8s.git
cd k8s
```

---

# ⚙️ 4. Run Installation Script

```bash
chmod 777 install.sh
./install.sh
```

👉 This script installs:

* Docker (if not installed)
* kubectl
* Kind (Kubernetes in Docker)

---

# ⚠️ 5. Fix Docker Permission Issue (IMPORTANT)

If you see error:

```bash
permission denied while trying to connect to the docker API
```

---

## ✅ Fix Steps

### 1️⃣ Add user to Docker group

```bash
sudo usermod -aG docker ubuntu
```

---

### 2️⃣ Apply group changes

```bash
newgrp docker
```

OR logout and login again

---

### 3️⃣ Test Docker

```bash
docker ps
```

👉 This must work without sudo

---

# ☸️ 6. Create Kubernetes Cluster (Kind)

```bash
mkdir kind-cluster
cd kind-cluster
```

---

## Create Cluster

```bash
kind create cluster --name=tws-cluster --config=kind-config.yaml
```

---

## Verify Cluster

```bash
kubectl cluster-info --context kind-tws-cluster
kubectl get nodes
```

👉 Nodes should be in **Ready** state

---

# 🌐 7. Install NGINX Ingress Controller

```bash
kubectl apply -f https://kind.sigs.k8s.io/examples/ingress/deploy-ingress-nginx.yaml
```

---

## Verify Namespace

```bash
kubectl get ns
```

👉 You should see:

```bash
ingress-nginx
```

---

# 🤖 8. Install Jenkins

Follow official documentation:

👉 https://www.jenkins.io/doc/book/installing/linux/#debianubuntu

---

## Start Jenkins

```bash
sudo systemctl start jenkins
sudo systemctl enable jenkins
```

---

# 🔐 9. Give Jenkins Sudo Access

```bash
sudo vim /etc/sudoers
```

Add:

```bash
jenkins ALL=(ALL:ALL) NOPASSWD:ALL
```

---

# ☸️ 10. Allow Jenkins to Access Kubernetes

---

## Copy kubeconfig

```bash
sudo mkdir -p /var/lib/jenkins/.kube
sudo cp ~/.kube/config /var/lib/jenkins/.kube/config
sudo chown -R jenkins:jenkins /var/lib/jenkins/.kube
```

---

## Test as Jenkins user

```bash
sudo su - jenkins
kubectl get nodes
```

👉 If this works → Jenkins can deploy to Kubernetes

---

# 🌍 11. Access Application (Temporary) Eun After CICD Success.

## Using Port Forward

```bash
kubectl port-forward -n nginx service/nginx-service 8081:80 --address 0.0.0.0
```

---

## Open in Browser

```bash
http://<EC2-PUBLIC-IP>:8081
```

---

## 🔓 Security Group Rule

Allow:

```bash
Port: 8081
Source: 0.0.0.0/0
```

---

# 🧠 Notes

* Kind cluster runs inside Docker
* Cluster does not persist after EC2 restart
* Ingress requires additional configuration for external access
* Port-forward is used for quick testing

---

# ✅ Prerequisites Completed

You are now ready to:

* Build Docker images
* Run Jenkins CI/CD pipeline
* Deploy applications to Kubernetes

---

