# 🚀 Employee Portal Application on Kubernetes (KIND)

## 📌 Project Overview

This project demonstrates deploying a **Python Flask application** on a **Kubernetes cluster created using KIND (Kubernetes IN Docker)**.

The application is:

- Containerized using **Docker**
- Deployed using **Kubernetes Deployment**
- Exposed using a **NodePort Service**
- Accessed externally using **KIND extraPortMappings**
- Hosted on **AWS EC2**

---

# 🏗️ Architecture

```text
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
```

---

# 🛠️ Tech Stack

- Python 3.11
- Flask
- Docker
- Kubernetes
- KIND
- AWS EC2
- Git
- GitHub

---

# 📁 Project Structure

```text
employee-portal/
│
├── app.py
├── requirements.txt
├── Dockerfile
├── kind-config.yml
├── namespace.yml
├── Deployment.yml
├── service.yml
├── README.md
│
└── templates/
      └── index.html
```

---

# 🚀 Step 1: Create Flask Application

## app.py

```python
from flask import Flask, render_template, jsonify

app = Flask(__name__)

employees = [
    {"id": 1, "name": "Manmeet Kaur", "department": "DevOps"},
    {"id": 2, "name": "John Doe", "department": "Development"},
    {"id": 3, "name": "Jane Smith", "department": "Testing"}
]

@app.route('/')
def home():
    return render_template('index.html', employees=employees)

@app.route('/health')
def health():
    return jsonify({
        "status": "UP"
    })

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000)
```

---

# 📦 Step 2: Create requirements.txt

```text
flask
```

---

# 🐳 Step 3: Create Dockerfile

```dockerfile
FROM python:3.11-slim

WORKDIR /app

COPY requirements.txt .

RUN pip install -r requirements.txt

COPY . .

CMD ["python","app.py"]
```

---

# 🔨 Step 4: Build Docker Image

```bash
docker build -t employee-portal:latest .
```

Verify:

```bash
docker images
```

---

# ☸️ Step 5: Create KIND Cluster

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

# 📥 Step 6: Load Docker Image into KIND

```bash
kind load docker-image employee-portal:latest --name employee-cluster
```

---

# 🏢 Step 7: Create Namespace

## namespace.yml

```yaml
kind: Namespace
apiVersion: v1

metadata:
  name: employee-app
```

Create namespace:

```bash
kubectl apply -f namespace.yml
```

Verify:

```bash
kubectl get ns
```

---

# 🚀 Step 8: Create Deployment

## Deployment.yml

```yaml
kind: Deployment
apiVersion: apps/v1

metadata:
  name: employee-app-deployment
  namespace: employee-app

spec:
  replicas: 4

  selector:
    matchLabels:
      app: employee-app

  template:
    metadata:
      labels:
        app: employee-app

    spec:
      containers:
        - name: employee-container
          image: employee-portal:latest
          imagePullPolicy: Never

          ports:
            - containerPort: 5000
```

Deploy:

```bash
kubectl apply -f Deployment.yml
```

Verify:

```bash
kubectl get deploy -n employee-app
kubectl get pods -n employee-app
```

---

# 🌐 Step 9: Create Service

## service.yml

```yaml
kind: Service
apiVersion: v1

metadata:
  name: employee-service
  namespace: employee-app

spec:
  type: NodePort

  selector:
    app: employee-app

  ports:
    - port: 80
      targetPort: 5000
      nodePort: 30082
```

Create service:

```bash
kubectl apply -f service.yml
```

Verify:

```bash
kubectl get svc -n employee-app
```

---

# 🔐 Step 10: Configure AWS Security Group

Allow inbound traffic:

```text
Type      : Custom TCP
Port      : 30082
Source    : Anywhere (0.0.0.0/0)
```

---

# 🌍 Application Access

```text
http://<EC2-PUBLIC-IP>:30082
```

Example:

```text
http://15.207.xxx.xxx:30082
```

---

# 🔄 Kubernetes Networking Flow

```text
Browser
    ↓
EC2 Public IP:30082
    ↓
extraPortMappings
    ↓
KIND Control Plane:30082
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

# ❓ Why extraPortMappings?

KIND runs Kubernetes nodes as Docker containers.

To access a Kubernetes NodePort from outside the KIND cluster, we need to expose the NodePort from the Docker container to the EC2 host.

```text
EC2 Host Port 30082
          ↓
extraPortMappings
          ↓
KIND Node Port 30082
          ↓
NodePort Service
          ↓
Flask Application
```

Without `extraPortMappings`, the application would only be accessible inside the KIND cluster.

---

# 🔢 Understanding Ports

| Component | Port |
|-----------|------|
| Flask Application | 5000 |
| containerPort | 5000 |
| targetPort | 5000 |
| Service Port | 80 |
| NodePort | 30082 |
| extraPortMappings | 30082 |

---

# 💻 Commands Used

```bash
docker build -t employee-portal:latest .

kind create cluster --config kind-config.yml

kind load docker-image employee-portal:latest --name employee-cluster

kubectl apply -f namespace.yml

kubectl apply -f Deployment.yml

kubectl apply -f service.yml

kubectl get nodes

kubectl get deploy -n employee-app

kubectl get pods -n employee-app

kubectl get svc -n employee-app
```

---

# 📚 Key Learnings

✅ Docker Image Creation

✅ KIND Cluster Creation

✅ extraPortMappings

✅ Kubernetes Namespace

✅ Deployment

✅ ReplicaSet

✅ Pods

✅ Labels and Selectors

✅ NodePort Service

✅ Kubernetes Networking

✅ AWS Security Groups

✅ Browser Access

✅ Container Image Loading in KIND

---

# 📸 Screenshots

Add screenshots here:

- Application Homepage
- kubectl get nodes
- kubectl get pods
- kubectl get svc
- Browser Output

---

# 👨‍💻 Author

**Manmeet Kaur**

DevOps Engineer | RHCSA Certified | AWS | Docker | Kubernetes | CI/CD
