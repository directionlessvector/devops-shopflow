# Shopflow-Lite Architecture & Flow Documentation

## 📋 Table of Contents
1. [API Flow](#api-flow)
2. [DevOps/CI-CD Flow](#devopsci-cd-flow)
3. [System Architecture](#system-architecture)
4. [Components Overview](#components-overview)

---

## 🔄 API Flow

### Frontend Routes & Endpoints

The application is a Flask-based e-commerce platform with the following API endpoints:

| Route | Method | Description | Response |
|-------|--------|-------------|----------|
| `/` | GET | Home page | HTML - index.html |
| `/products` | GET | Fetch all products from Supabase | HTML - products.html with products list |
| `/add-to-cart/<product_id>` | GET | Add product to session cart | Redirect to /cart |
| `/decrease/<product_id>` | GET | Decrease item quantity | Redirect to /cart |
| `/remove/<product_id>` | GET | Remove item from cart | Redirect to /cart |
| `/cart` | GET | Display current cart | HTML - cart.html with items & total |
| `/checkout` | GET | Create order & clear cart | HTML - checkout.html |
| `/orders` | GET | View order history | HTML - orders.html |
| `/health` | GET | Health check endpoint | JSON `{"status": "running"}` |

### Data Flow Diagram

```
User Browser
    ↓
    ├─→ / (Home) → index.html
    ├─→ /products → Supabase Query → products.html
    ├─→ /add-to-cart/<id> → Session Storage → /cart redirect
    ├─→ /decrease/<id> → Session Update → /cart redirect
    ├─→ /remove/<id> → Session Delete → /cart redirect
    ├─→ /cart → Session Cart → cart.html
    ├─→ /checkout → Order Creation + Session Clear → checkout.html
    ├─→ /orders → Session Order History → orders.html
    └─→ /health → Health Check → JSON response
```

### Session-Based State Management

- **Cart**: Stored in Flask session as `session["cart"]` (dict: {product_id: quantity})
- **Order History**: Stored in Flask session as `session["order_history"]` (list, max 25 orders)
- **Session Data**: Persisted in browser cookies (Flask session secret: "supersecretkey")

### Database Interactions

**Supabase** integration points:
- Read: `products` table (retrieves all products with id, name, price, image_url)
- Requires credentials: `SUPABASE_URL` and `SUPABASE_KEY` (env variables)

---

## 🚀 DevOps/CI-CD Flow

### CI/CD Pipeline (Jenkinsfile)

The pipeline consists of 5 stages executed in sequence:

```
Stage 1: Clone Check
    ↓
Stage 2: SonarQube Analysis
    ↓
Stage 3: Docker Check
    ↓
Stage 4: Build Docker Image
    ↓
Stage 5: Deploy to Kubernetes
```

### Detailed Pipeline Stages

#### Stage 1: Clone Check
- **Command**: `ls`
- **Purpose**: Verify repository is cloned correctly
- **Output**: Lists directory contents

#### Stage 2: SonarQube Analysis
- **Tool**: SonarQube Scanner
- **Config Source**: sonar-project.properties
- **Projects Scanned**: `app/` directory
- **Credentials**: Requires Jenkins credential `sonarqube-token`
- **Purpose**: Code quality analysis, security scanning
- **Key Properties**:
  - `sonar.projectKey=shopflow`
  - `sonar.projectName=shopflow`
  - `sonar.sources=app`

#### Stage 3: Docker Check
- **Command**: `docker --version`
- **Purpose**: Verify Docker is available on Jenkins agent
- **Output**: Docker version info

#### Stage 4: Build Docker Image
- **Command**: `docker build -t shopflow-test .`
- **Dockerfile**: Uses `Dockerfile` (Python 3.9-slim base)
- **Image Details**:
  - Base: `python:3.9-slim`
  - Working Dir: `/app`
  - Port: 5001 (Flask)
  - Entrypoint: `python app.py`
  - Dependencies: Flask, Supabase, python-dotenv, Gunicorn
- **Image Tag**: `shopflow-test`

#### Stage 5: Deploy to Kubernetes
- **Command**: 
  ```bash
  kubectl apply -f k8s/deployment.yaml -n default
  kubectl apply -f k8s/service.yaml -n default
  ```
- **Namespace**: default
- **Resources Deployed**:
  - Deployment (2 replicas)
  - Service (NodePort: 30007)

---

## 🏗️ System Architecture

### High-Level Architecture Diagram

```
┌─────────────────────────────────────────────────────────────┐
│                     DEVELOPMENT ENVIRONMENT                  │
├─────────────────────────────────────────────────────────────┤
│                                                               │
│  GitHub Repository                                           │
│  ├── app/                                                    │
│  ├── k8s/                                                    │
│  ├── Dockerfile                                              │
│  └── Jenkinsfile                                             │
│                                                               │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│                    JENKINS CI/CD SERVER                      │
├─────────────────────────────────────────────────────────────┤
│                                                               │
│  Namespace: jenkins                                          │
│  ├── Jenkins Container (Custom image with Docker + kubectl)  │
│  ├── Docker Sock Mount (for DinD)                           │
│  ├── PersistentVolume (jenkins-home)                        │
│  └── ServiceAccount (jenkins-deployer with K8s access)     │
│                                                               │
│  Stages:                                                     │
│  1. Clone Check ────→ 2. SonarQube Analysis                 │
│  3. Docker Check ────→ 4. Build Image                       │
│  5. Deploy to K8s                                            │
│                                                               │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│              KUBERNETES CLUSTER (K8s)                        │
├─────────────────────────────────────────────────────────────┤
│                                                               │
│  Namespaces:                                                 │
│  • default → Shopflow Application                           │
│  • jenkins → Jenkins CI/CD                                  │
│  • kube-prometheus → Monitoring (Prometheus/Grafana)        │
│                                                               │
│  Default Namespace:                                          │
│  ├── Deployment: shopflow                                   │
│  │   ├── 2 Replicas                                         │
│  │   ├── Container: shopflow:latest                         │
│  │   ├── Port: 5001                                         │
│  │   ├── Env: SUPABASE_URL, SUPABASE_KEY (from Secret)     │
│  │   ├── Liveness Probe: /health (10s delay, 5s period)    │
│  │   └── Readiness Probe: /health                           │
│  │                                                           │
│  └── Service: shopflow-service                              │
│      ├── Type: NodePort                                     │
│      ├── Port: 80                                           │
│      ├── TargetPort: 5001                                   │
│      └── NodePort: 30007                                    │
│                                                               │
│  Secrets:                                                    │
│  └── shopflow-secret                                        │
│      ├── SUPABASE_URL                                       │
│      └── SUPABASE_KEY                                       │
│                                                               │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│              EXTERNAL SERVICES                               │
├─────────────────────────────────────────────────────────────┤
│                                                               │
│  Supabase (PostgreSQL)                                       │
│  ├── Database: products table                               │
│  ├── Columns: id, name, price, image_url, ...              │
│  └── Auth: API Key (SUPABASE_KEY)                           │
│                                                               │
│  SonarQube (Code Quality)                                   │
│  └── Analysis of: app/ directory                            │
│                                                               │
└─────────────────────────────────────────────────────────────┘
```

---

## 📦 Components Overview

### 1. Flask Application (`app/app.py`)

**Core Dependencies**:
- `flask`: Web framework
- `supabase`: Database client
- `python-dotenv`: Environment variable management
- `gunicorn`: Production WSGI server (available but not used in dev)

**Key Features**:
- Session-based cart management
- Supabase integration for product data
- RESTful routing
- Health check endpoint for K8s probes
- Order history tracking (max 25 orders)

### 2. Docker Container

**Dockerfile** build process:
```
python:3.9-slim (base image)
    ↓
Install requirements.txt
    ↓
Copy app/ directory
    ↓
Expose port 5001
    ↓
Run: python app.py
```

**Container Configuration**:
- Base Image: `python:3.9-slim` (lightweight)
- Python Environment: Unbuffered output (`PYTHONUNBUFFERED=1`)
- Port: 5001
- Command: `python app.py`

### 3. Kubernetes Deployment

**Shopflow Deployment** (`k8s/deployment.yaml`):
- **Replicas**: 2 (for HA)
- **Image**: `shopflow:latest` (local, imagePullPolicy: Never)
- **Container Port**: 5001
- **Probes**: 
  - Liveness: Checks `/health` endpoint every 5s (after 10s initial delay)
  - Readiness: Checks `/health` endpoint
- **Secrets Injection**: SUPABASE_URL, SUPABASE_KEY

**Service** (`k8s/service.yaml`):
- **Type**: NodePort
- **Exposure**: Port 80 → Pod 5001
- **NodePort**: 30007 (access: `http://node-ip:30007`)

### 4. Jenkins CI/CD

**Dockerfile.jenkins**:
- Base: `jenkins/jenkins:lts`
- Added Tools: Docker, kubectl v1.30.0
- Privileges: Runs Docker-in-Docker (DinD)

**Jenkins Deployment** (`jenkins-deployment.yaml`):
- **Namespace**: jenkins
- **Replicas**: 1
- **Volumes**:
  - Docker socket mount (for DinD)
  - PersistentVolume for /var/jenkins_home
- **ServiceAccount**: jenkins-deployer (K8s cluster access)
- **Security**: Privileged container, runAsUser: 0

### 5. Kubernetes RBAC

**jenkins-rbac.yaml** components:
- **ServiceAccount**: jenkins-deployer
- **Role**: Permissions for deployments, services in default namespace
- **RoleBinding**: Binds jenkins-deployer SA to Role

### 6. Supporting Resources

**Secrets** (`k8s/secret.yaml`):
- Stores Supabase credentials (base64 encoded)
- Referenced by deployment env vars

**PVC** (`k8s/jenkins-pvc.yaml`):
- Persistent storage for Jenkins home directory
- Survives pod restarts

**Monitoring** (`kube-prometheus/`):
- Prometheus + Grafana setup
- Metrics collection from K8s cluster

---

## 🔌 Key Integration Points

### Supabase Connection
```
Flask App
    ↓
Environment Variables (SUPABASE_URL, SUPABASE_KEY)
    ↓
Supabase Python Client
    ↓
PostgreSQL Database (products table)
```

### K8s to Jenkins Communication
```
Jenkins Pod (jenkins-deployer SA)
    ↓
Kube API Server
    ↓
kubectl commands → Create/Update K8s resources
```

### CI/CD to Deployment
```
Git Commit
    ↓
Jenkins Webhook Trigger
    ↓
Pipeline Execution (5 stages)
    ↓
Docker Image Build
    ↓
kubectl apply → K8s Deployment
    ↓
Service exposes app on NodePort 30007
```

---

## 🎯 Necessary Components for Architecture Diagram

### Compute Layer
- ✅ Flask Application (Python)
- ✅ Docker Container
- ✅ Kubernetes Pods (2 replicas)
- ✅ Jenkins Pod

### Networking Layer
- ✅ Kubernetes Service (NodePort)
- ✅ Internal K8s DNS
- ✅ Container networking

### Storage Layer
- ✅ PersistentVolumeClaim (Jenkins)
- ✅ Docker volume mounts
- ✅ Supabase Database

### External Services
- ✅ Supabase (PostgreSQL)
- ✅ SonarQube (Code Analysis)
- ✅ Docker Registry (image storage)

### CI/CD Pipeline
- ✅ Git Repository
- ✅ Jenkins Server
- ✅ Jenkinsfile (pipeline definition)
- ✅ Docker (build tool)
- ✅ kubectl (deployment tool)

### Monitoring (Optional)
- ⚠️ Prometheus & Grafana (kube-prometheus folder exists)
- ⚠️ Kubernetes metrics
- ⚠️ Health check endpoints

