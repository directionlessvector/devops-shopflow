# 🚀 Shopflow-Lite - Quick Reference Guide

## 📌 Executive Summary

Shopflow-Lite is a containerized e-commerce Flask application with:
- **8 API endpoints** for shopping cart functionality
- **5-stage CI/CD pipeline** with SonarQube + Docker + Kubernetes
- **2-replica Kubernetes deployment** for high availability
- **Session-based cart management** (no database persistence for cart data)
- **External Supabase PostgreSQL** for product data

---

## 🔌 API Flow At A Glance

```
User Actions → Flask Routes → Session/Database → Response

GET /products     → Query Supabase → Display products
GET /add-to-cart  → Update session['cart'] → Redirect
GET /cart         → Calculate items+totals → Display cart
GET /checkout     → Create order → Clear cart → Confirm
GET /orders       → Fetch history → Display orders
```

---

## 🚀 DevOps Flow At A Glance

```
Git Commit
  ↓
Jenkins Webhook Trigger
  ↓
Stage 1: Clone Check
Stage 2: SonarQube Analysis (code quality check)
Stage 3: Docker Check
Stage 4: Build Docker Image (shopflow-test)
Stage 5: Deploy to K8s (kubectl apply)
  ↓
Pods Ready at localhost:30007
```

---

## 🏗️ Architecture Overview

### Layers (6-tier)
1. **User Layer** → Browser (HTTP client)
2. **Frontend Layer** → HTML templates (index, products, cart, checkout, orders)
3. **API Layer** → Flask routes (8 endpoints)
4. **Container Layer** → Docker image (Python 3.9-slim + Flask)
5. **Orchestration Layer** → Kubernetes (2 pod replicas, service, secrets)
6. **Data Layer** → Supabase PostgreSQL + Session cookies

### Key Components

| Component | Details |
|-----------|---------|
| **Container** | Python 3.9-slim, port 5001 |
| **Deployment** | 2 replicas, rolling updates |
| **Service** | NodePort type, port 80 → 5001, NodePort 30007 |
| **Database** | Supabase PostgreSQL (products table) |
| **Session** | Encrypted cookies (cart, orders) |
| **CI/CD** | Jenkins with Docker + kubectl |
| **Quality Gate** | SonarQube analysis required |
| **Secrets** | K8s secret for SUPABASE_URL & SUPABASE_KEY |
| **Health Check** | /health endpoint (liveness & readiness probes) |

---

## 📊 Data Flow Diagram

```
┌──────────────┐
│   Browser    │
└──────┬───────┘
       │ HTTP Requests
       ↓
┌──────────────────────────┐
│   Flask Application      │
│   (Port 5001)            │
│ ┌──────────────────────┐ │
│ │ Routes (8 endpoints) │ │
│ └──────────────────────┘ │
└─────┬────────────┬───────┘
      │            │
      │ SQL Query  │ Session
      ↓            ↓
┌──────────────┐  ┌─────────────┐
│   Supabase   │  │   Cookies   │
│  PostgreSQL  │  │  (Encrypted)│
│ (products)   │  │ (Cart/Order)│
└──────────────┘  └─────────────┘
```

---

## 🚀 CI/CD Pipeline Stages

### Stage 1: Clone Check
- Command: `ls`
- Purpose: Verify repository cloned correctly

### Stage 2: SonarQube Analysis  ⚠️ GATE
- Tool: SonarQube Scanner
- Scope: `app/` directory
- Credential: `sonarqube-token`
- Decision: PASS/FAIL (fails pipeline if issues)

### Stage 3: Docker Check
- Command: `docker --version`
- Purpose: Verify Docker available

### Stage 4: Build Docker Image
- Command: `docker build -t shopflow-test .`
- Output: Docker image tagged `shopflow-test`

### Stage 5: Deploy to Kubernetes
- Commands:
  ```bash
  kubectl apply -f k8s/deployment.yaml -n default
  kubectl apply -f k8s/service.yaml -n default
  ```
- Result: 2 pods running, service exposed on port 30007

---

## 📦 Kubernetes Resources

### Deployment (k8s/deployment.yaml)
```
Name: shopflow
Replicas: 2
Image: shopflow:latest
Port: 5001
Env: SUPABASE_URL, SUPABASE_KEY (from secret)
Liveness Probe: /health (every 5s, 10s delay)
Readiness Probe: /health
```

### Service (k8s/service.yaml)
```
Name: shopflow-service
Type: NodePort
Port: 80
TargetPort: 5001
NodePort: 30007
Selector: app=shopflow
```

### Secret (k8s/secret.yaml)
```
Name: shopflow-secret
Type: Opaque
Keys: SUPABASE_URL, SUPABASE_KEY
```

### Jenkins (jenkins-deployment.yaml)
```
Namespace: jenkins
Name: jenkins
Replicas: 1
Image: custom-jenkins (Dockerfile.jenkins)
Port: 8080
Volumes: 
  - Docker socket (DinD)
  - PVC for jenkins-home
ServiceAccount: jenkins-deployer (K8s access)
```

---

## 🛒 Shopping Cart Flow (Session-Based)

```
1. GET /products
   ├─ Query: SELECT * FROM products
   └─ Render: List all products

2. GET /add-to-cart/123
   ├─ Update: session['cart']['123'] += 1
   └─ Store: Encrypted cookie sent to browser

3. GET /cart
   ├─ Read: session['cart']
   ├─ Loop: For each {product_id: qty}
   ├─ Query: Get product details from Supabase
   └─ Calculate: subtotal = qty × price, total = sum

4. GET /checkout
   ├─ Create: order = {id, placed_at, items, total}
   ├─ Append: session['order_history'].push(order)
   ├─ Limit: Keep max 25 orders
   └─ Clear: Delete session['cart']

5. GET /orders
   ├─ Read: session['order_history']
   └─ Display: All previous orders
```

---

## 🔐 Security & Configuration

### Environment Variables
- `SUPABASE_URL` - Supabase project URL
- `SUPABASE_KEY` - Supabase API key
- **Storage**: K8s Secret (mounted as env vars in pods)

### Session Management
- **Secret Key**: "supersecretkey" (Flask)
- **Storage**: HTTP Cookies (encrypted)
- **Scope**: Per-browser session
- **Data**: Cart items, order history

### Health Checks
- **Endpoint**: `/health`
- **Response**: `{"status": "running"}`
- **Uses**: K8s liveness & readiness probes

---

## 📋 Essential Diagram Components

### Must-Have
✅ Browser/Client  
✅ K8s Service (NodePort 30007)  
✅ Pods (2x Flask containers)  
✅ Supabase Database  
✅ Deployment Controller  
✅ Jenkins Pipeline  
✅ Docker Build  
✅ SonarQube Analysis  

### Should-Have
✅ Secrets/ConfigMaps  
✅ Liveness/Readiness Probes  
✅ Session Storage (Cookies)  
✅ Health Check Endpoint  
✅ PersistentVolumeClaim (Jenkins)  

### Nice-to-Have
⚠️ Prometheus/Grafana  
⚠️ AlertManager  
⚠️ Ingress Controller  

---

## 🎯 How Everything Connects

```
Developer Code
    ↓ Commits to Git
↓
Jenkins Webhook Triggered
    ↓ Stage 1-3: Checks
    ↓ Stage 4: Build Docker
↓
Docker Image (shopflow-test)
    ↓ Stage 5: Deploy
↓
K8s Applies Deployment + Service
    ↓ Pulls image (local)
    ↓ Creates 2 pods
    ↓ Exposes service
↓
User Accesses localhost:30007
    ↓ Browser HTTP request
    ↓ K8s Service load balances
    ↓ Request hits Flask pod
↓
Flask Routes:
    ├─ Query Supabase (products)
    ├─ Store in session (cart)
    └─ Render HTML templates
↓
Browser Receives Response + Cookie
    ↓ Displays page
    ↓ Stores encrypted cookie
↓
User Adds Items, Checks Out
    ↓ Session persists via cookie
    ↓ Orders stored in session
```

---

## 🔍 File Locations

```
app/
├── app.py              (Flask app, 8 routes)
├── requirements.txt    (Dependencies)
├── templates/          (HTML files)
│   ├── base.html
│   ├── index.html
│   ├── products.html
│   ├── cart.html
│   ├── checkout.html
│   └── orders.html
└── static/
    ├── css/shopflow.css
    └── images/

k8s/
├── deployment.yaml    (2 replicas)
├── service.yaml       (NodePort 30007)
├── secret.yaml        (Supabase creds)
├── jenkins-deployment.yaml
├── jenkins-rbac.yaml
├── jenkins-service.yaml
├── jenkins-pvc.yaml
└── jenkins-admin.yaml

Dockerfile                (Python 3.9-slim + Flask)
Dockerfile.jenkins        (Jenkins + Docker + kubectl)
Jenkinsfile              (5-stage pipeline)
sonar-project.properties (SonarQube config)
```

---

## 🚦 Access Points

| URL | Purpose | Response |
|-----|---------|----------|
| `localhost:30007/` | Home | HTML |
| `localhost:30007/products` | Product listing | HTML + products |
| `localhost:30007/cart` | Shopping cart | HTML + cart items |
| `localhost:30007/checkout` | Order confirmation | HTML |
| `localhost:30007/orders` | Order history | HTML + orders |
| `localhost:30007/health` | Health check | JSON `{status: running}` |

---

## ✨ Key Takeaways

1. **Stateless Pods**: No persistent cart data in database (session-only)
2. **High Availability**: 2 replicas with automatic failover
3. **CI/CD Gates**: SonarQube quality analysis required before deploy
4. **Container Optimization**: Slim Python image for faster pulls
5. **Infrastructure as Code**: All K8s resources defined in YAML
6. **Health Monitoring**: Probes ensure pod health, service readiness
7. **External Database**: Supabase handles product data persistence
8. **DinD Setup**: Jenkins can build Docker images in K8s

