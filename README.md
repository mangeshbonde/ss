# 🚀 StaticSite CI/CD Pipeline (Docker → Jenkins → Kubernetes on EC2)

---

## 📌 Project Overview

This project demonstrates a complete **CI/CD pipeline** for deploying a static website using:

* Docker (Containerization)
* Jenkins (CI Automation)
* Kubernetes (Kind Cluster)
* AWS EC2 (Infrastructure)

The pipeline automates the entire lifecycle:

```text
Code → Build → Docker Image → Push → Deploy → Access
```

---

# 🎯 Objective

To build a **fully automated CI/CD pipeline** where:

* Code changes trigger Jenkins
* Docker image is built & pushed
* Kubernetes deployment is updated automatically
* Application is accessible via browser

---

# 🏗️ Project Structure

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

# ⚙️ Step 1: Prerequisites Setup (EC2)

## Update System

```bash
sudo apt update
```

---

## Install Docker

```bash
sudo apt install -y docker.io
sudo systemctl start docker
sudo systemctl enable docker
```

---

## Run Docker Without sudo

```bash
sudo usermod -aG docker $USER
newgrp docker
```

---

## Install Kubernetes Tools (Kind + kubectl)

```bash
git clone https://github.com/mangeshbonde/k8s.git
cd k8s
chmod 777 install.sh
./install.sh
```

---

## Fix Docker Permission Issue

```bash
sudo usermod -aG docker ubuntu
newgrp docker
docker ps
```

---

# ☸️ Step 2: Create Kubernetes Cluster

```bash
kind create cluster --name=tws-cluster --config=kind-config.yaml
```

---

## Verify Cluster

```bash
kubectl get nodes
```

---

# 🌐 Step 3: Install Ingress Controller

```bash
kubectl apply -f https://kind.sigs.k8s.io/examples/ingress/deploy-ingress-nginx.yaml
```

---

## Verify

```bash
kubectl get pods -n ingress-nginx
```

---

# 🐳 Step 4: Docker Setup

## Dockerfile

```dockerfile
FROM nginx:alpine
RUN rm -rf /usr/share/nginx/html/*
COPY app/index.html /usr/share/nginx/html/index.html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

---

## Build Image

```bash
docker build -t <dockerhub-username>/nginx-co:v1 .
```

---

## Push Image

```bash
docker login
docker push <dockerhub-username>/nginx-co:v1
```

---

# 🤖 Step 5: Jenkins Setup

## Install Jenkins

Follow:
https://www.jenkins.io/doc/book/installing/linux/#debianubuntu

---

## Start Jenkins

```bash
sudo systemctl start jenkins
sudo systemctl enable jenkins
```

---

## Add Docker Hub Credentials

* Go to Jenkins → Manage Jenkins → Credentials
* Add:

  * Username
  * Password
  * ID: `dockerhub-creds`

---

## Allow Jenkins to Access Docker

```bash
sudo usermod -aG docker jenkins
sudo systemctl restart jenkins
```

---

## Allow Jenkins to Access Kubernetes

```bash
sudo mkdir -p /var/lib/jenkins/.kube
sudo cp ~/.kube/config /var/lib/jenkins/.kube/config
sudo chown -R jenkins:jenkins /var/lib/jenkins/.kube
```

---

## Test

```bash
sudo su - jenkins
kubectl get nodes
```

---

# ⚙️ Step 6: Kubernetes Manifests

## deployment.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-app
  namespace: nginx
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx-app
  template:
    metadata:
      labels:
        app: nginx-app
    spec:
      containers:
      - name: nginx-container
        image: <dockerhub-username>/nginx-co:latest
        ports:
        - containerPort: 80
```

---

## service.yaml (ClusterIP)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
  namespace: nginx
spec:
  type: ClusterIP
  selector:
    app: nginx-app
  ports:
    - port: 80
      targetPort: 80
```

---

## ingress.yaml

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nginx-ingress
  namespace: nginx
spec:
  rules:
  - http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: nginx-service
            port:
              number: 80
```

---

# 🚀 Step 7: Jenkins Pipeline (Full CI/CD)

```groovy
pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "<dockerhub-username>/nginx-co"
        TAG = "latest"
        K8S_NAMESPACE = "nginx"
    }

    stages {

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $DOCKER_IMAGE:$TAG .'
            }
        }

        stage('Login to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    sh 'echo $PASS | docker login -u $USER --password-stdin'
                }
            }
        }

        stage('Push Image') {
            steps {
                sh 'docker push $DOCKER_IMAGE:$TAG'
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                kubectl apply -f k8s/
                kubectl set image deployment/nginx-app nginx-container=$DOCKER_IMAGE:$TAG -n $K8S_NAMESPACE
                '''
            }
        }
    }
}
```

---

# 🌐 Step 8: Access Application

## Option 1 (Port Forward)

```bash
kubectl port-forward -n nginx service/nginx-service 8081:80 --address 0.0.0.0
```

```
http://<EC2-PUBLIC-IP>:8081
```

---

## Option 2 (Ingress)

```bash
kubectl port-forward -n ingress-nginx service/ingress-nginx-controller 80:80 --address 0.0.0.0
```

```
http://<EC2-PUBLIC-IP>
```

---

## 🔓 Security Group Rules

| Port | Purpose            |
| ---- | ------------------ |
| 8080 | Jenkins            |
| 8081 | App (port-forward) |
| 80   | Ingress            |

---

# 🎉 Final Architecture

```
Developer → GitHub → Jenkins → Docker Hub → Kubernetes → Ingress → Browser
```

---

# 🧠 Key Learnings

* Docker build context handling
* Jenkins credentials management
* CI/CD pipeline automation
* Kubernetes deployment lifecycle
* ClusterIP vs NodePort vs Ingress
* AWS EC2 networking

---
