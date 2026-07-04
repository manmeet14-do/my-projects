# Employee Portal Application on Kubernetes (KIND)

## Project Overview

This project demonstrates deploying a Python Flask application on a Kubernetes cluster created using KIND (Kubernetes IN Docker).

The application is containerized using Docker, deployed using Kubernetes Deployment and exposed using a NodePort Service with KIND extraPortMappings.

---

# Architecture

Browser
    ↓
AWS EC2
    ↓
extraPortMappings
    ↓
KIND Cluster
    ↓
NodePort Service
    ↓
Deployment
    ↓
ReplicaSet
    ↓
Pods
    ↓
Flask Application

---

# Tech Stack

- Python 3.11
- Flask
- Docker
- Kubernetes
- KIND
- AWS EC2
- Git/GitHub

---

# Project Structure

```bash
employee-portal/
│
├── app.py
├── requirements.txt
├── Dockerfile
├── kind-config.yml
├── namespace.yml
├── Deployment.yml
├── service.yml
└── templates/
    └── index.html
```

---

# Step 1: Create Flask Application

## app.py

```python
from flask import Flask, render_template

app = Flask(__name__)

@app.route('/')
def home():
    return render_template('index.html')

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000)
```

---

# Step 2: Create requirements.txt

```text
flask
```

---

# Step 3: Create Dockerfile

```dockerfile
FROM python:3.11-slim

WORKDIR /app

COPY requirements.txt .

RUN pip install -r requirements.txt

COPY . .

CMD ["python","app.py"]
```

---

# Step 4: Build Docker Image

```bash
docker build -t employee-portal:latest .
```

Verify:

```bash
docker images
```

---

# Step 5: Create KIND Cluster

## kind-config.yml

```yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
name: employee-cluster

nodes:
- role: control-plane
  image: kindest/node:v1.29.0

  extraPortMappings:
  - containerPort: 30082
    hostPort: 30082
    protocol: TCP

- role: worker
  image: kindest/node:v1.29.0

- role: worker
  image: kindest/node:v1.29.0
```

Create cluster:

```bash
kind create cluster --config kind-config.yml
```

Verify:

```bash
kubectl get nodes
```

---

# Step 6: Load Docker Image into KIND

```bash
kind load docker-image employee-portal:latest --name employee-cluster
```

---

# Step 7: Create Namespace

```bash
kubectl apply -f namespace.yml
```

Verify:

```bash
kubectl get ns
```

---

# Step 8: Create Deployment

```bash
kubectl apply -f Deployment.yml
```

Verify:

```bash
kubectl get deploy -n employee-app
kubectl get pods -n employee-app
```

---

# Step 9: Create Service

```bash
kubectl apply -f service.yml
```

Verify:

```bash
kubectl get svc -n employee-app
```

---

# Step 10: Open AWS Security Group

Allow:

```text
TCP 30082
```

---

# Application Access

```text
http://<EC2-PUBLIC-IP>:30082
```

---

# Kubernetes Networking Flow

```text
Browser
    ↓
EC2:30082
    ↓
extraPortMappings
    ↓
KIND Node:30082
    ↓
NodePort Service
    ↓
Service Port:80
    ↓
targetPort:5000
    ↓
Flask Application
```

---

# Understanding Ports

| Component | Port |
|-----------|------|
| Flask App | 5000 |
| containerPort | 5000 |
| targetPort | 5000 |
| Service Port | 80 |
| NodePort | 30082 |
| extraPortMappings | 30082 |

---

# Commands Used

```bash
docker build -t employee-portal:latest .

kind create cluster --config kind-config.yml

kind load docker-image employee-portal:latest --name employee-cluster

kubectl apply -f namespace.yml

kubectl apply -f Deployment.yml

kubectl apply -f service.yml

kubectl get nodes

kubectl get pods -n employee-app

kubectl get deploy -n employee-app

kubectl get svc -n employee-app
```

---

# Learning Outcomes

- Docker Image Creation
- KIND Cluster Creation
- extraPortMappings
- Namespace
- Deployment
- ReplicaSet
- Pods
- Labels and Selectors
- NodePort Service
- Kubernetes Networking
- AWS Security Groups
- Browser Access
