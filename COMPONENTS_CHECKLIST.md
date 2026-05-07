# Shopflow-Lite Architecture Diagram - Components Checklist

## 📐 Architecture Diagram Components

### Core Application Components
- [x] **Flask Application**
  - Type: Web Framework (Python)
  - Port: 5001
  - Features: Routing, Session Management, Templating
  
- [x] **HTML Templates**
  - index.html (Home)
  - products.html (Product Listing)
  - cart.html (Shopping Cart)
  - checkout.html (Order Confirmation)
  - orders.html (Order History)
  - base.html (Base Template)
  
- [x] **Static Assets**
  - CSS: static/css/shopflow.css
  - Images: static/images/

### Data Layer Components
- [x] **Supabase PostgreSQL Database**
  - Table: products (id, name, price, image_url, ...)
  - Authentication: API Key
  
- [x] **Session Storage**
  - Medium: HTTP Cookies (encrypted with Flask secret key)
  - Data: Cart items & Order history
  
- [x] **Environment Variables**
  - SUPABASE_URL
  - SUPABASE_KEY

### Container Components
- [x] **Docker Image**
  - Base: python:3.9-slim
  - Container Registry: Local/DockerHub
  - Image Name: shopflow:latest
  - Build Command: docker build -t shopflow-test .
  
- [x] **Dockerfile**
  - Multi-stage build considerations: No (simple single stage)
  - Dependencies: Flask, Supabase, python-dotenv, Gunicorn
  - Exposed Port: 5001
  
- [x] **Docker Compose** (Optional)
  - Not currently implemented
  - Could include: Flask app + PostgreSQL + Redis

### Kubernetes Components
- [x] **Deployment (default namespace)**
  - Name: shopflow
  - Replicas: 2
  - Selector: app=shopflow
  - Strategy: Rolling Update (default)
  
- [x] **Pod Specifications**
  - Container: shopflow:latest
  - Port: 5001
  - ImagePullPolicy: Never (local image)
  
- [x] **Liveness Probe**
  - Type: HTTP
  - Path: /health
  - Port: 5001
  - Initial Delay: 10s
  - Period: 5s
  
- [x] **Readiness Probe**
  - Type: HTTP
  - Path: /health
  - Port: 5001
  
- [x] **Service (default namespace)**
  - Name: shopflow-service
  - Type: NodePort
  - Port: 80
  - TargetPort: 5001
  - NodePort: 30007
  - Selector: app=shopflow
  
- [x] **Secret (default namespace)**
  - Name: shopflow-secret
  - Type: Opaque
  - Data: SUPABASE_URL, SUPABASE_KEY
  - Mounted as: Environment variables in pods

### Jenkins/CI-CD Components
- [x] **Jenkins Deployment (jenkins namespace)**
  - Name: jenkins
  - Replicas: 1
  - Image: custom-jenkins (Dockerfile.jenkins)
  - Port: 8080
  
- [x] **Custom Jenkins Image**
  - Base: jenkins/jenkins:lts
  - Added Tools: Docker, kubectl v1.30.0
  - Security: DinD enabled, privileged mode
  
- [x] **ServiceAccount (jenkins namespace)**
  - Name: jenkins-deployer
  - Permissions: Create/Update deployments, services
  
- [x] **Role & RoleBinding**
  - rbac.yaml: Define K8s permissions for Jenkins
  
- [x] **Volumes**
  - Docker Socket: /var/run/docker.sock (hostPath)
  - Jenkins Home: PersistentVolumeClaim (jenkins-pvc)
  
- [x] **PersistentVolumeClaim (jenkins namespace)**
  - Name: jenkins-pvc
  - Purpose: Store Jenkins configuration & build history
  - Access Mode: ReadWriteOnce

### CI/CD Pipeline Components
- [x] **Git Repository**
  - Trigger: Webhook
  - Branches: Main (assumed)
  
- [x] **Jenkinsfile**
  - Stages: 5 (Clone Check, SonarQube, Docker Check, Build, Deploy)
  - Agent: Any
  - Credentials: sonarqube-token
  
- [x] **SonarQube Integration**
  - sonar-project.properties: Project configuration
  - projectKey: shopflow
  - Sources: app/
  - Server: sonarqube (Jenkins configured)
  
- [x] **Docker Build**
  - Tool: Docker CLI on Jenkins agent
  - Registry: Local (imagePullPolicy: Never in K8s)
  
- [x] **kubectl Deployment**
  - Tool: kubectl on Jenkins agent
  - Manifests: k8s/deployment.yaml, k8s/service.yaml

### Monitoring & Observability
- [x] **Health Check Endpoint**
  - Path: /health
  - Response: {"status": "running"}
  - Purpose: K8s liveness/readiness probes
  
- [ ] **Prometheus** (kube-prometheus exists, not fully configured)
  - Namespace: kube-prometheus
  - Metrics: K8s cluster metrics, pod metrics
  - Scrape Targets: /metrics endpoints
  
- [ ] **Grafana** (kube-prometheus exists, not fully configured)
  - Namespace: kube-prometheus
  - Dashboards: K8s cluster, application performance
  - Data Source: Prometheus
  
- [ ] **Alertmanager** (kube-prometheus exists, not fully configured)
  - Namespace: kube-prometheus
  - Rules: Pod restart thresholds, error rates

### Network & Routing
- [x] **Kubernetes DNS**
  - Service URL: shopflow-service.default.svc.cluster.local
  - Port: 80
  
- [x] **External Access**
  - NodePort: 30007
  - Access: http://node-ip:30007
  
- [x] **Ingress** (Not currently implemented)
  - Could replace NodePort for better routing
  - SSL/TLS termination capability

### Configuration Management
- [x] **Environment Variables**
  - Source: K8s Secrets
  - Variables: SUPABASE_URL, SUPABASE_KEY
  
- [x] **.env File** (Development only)
  - Used with python-dotenv
  - Not committed to repository
  
- [x] **ConfigMaps** (Not currently used)
  - Could store: Flask configuration, app settings

### Development Tools
- [x] **Python Version**
  - Version: 3.9
  - Base Image: python:3.9-slim
  
- [x] **Dependencies**
  - flask: Web framework
  - supabase: Database client
  - python-dotenv: Environment management
  - gunicorn: WSGI server (optional)
  
- [x] **Linting & Analysis**
  - Tool: SonarQube
  - Triggers: On every CI/CD pipeline run

---

## 🎨 Recommended Architecture Diagram Layers

### Layer 1: User & Access
```
[Users/Browsers] 
    ↓ HTTP Requests (Port 80/30007)
```

### Layer 2: Load Balancing & Ingress
```
[Kubernetes Service - NodePort 30007]
    ↓ Internal Service DNS
    ↓ Port 80 → 5001
```

### Layer 3: Application Layer
```
[Pod 1: Flask App] [Pod 2: Flask App]
    ↓ Session Cookies ↓
    ↓ SQL Queries ↓
```

### Layer 4: Data Layer
```
[Supabase PostgreSQL]    [Session Cookies]
    ↓                        ↓
[Products Table]         [Browser Storage]
```

### Layer 5: CI/CD Layer
```
[GitHub Repo] → [Jenkins] → [SonarQube] → [Docker Build] → [kubectl Deploy]
```

### Layer 6: Kubernetes Infrastructure
```
[K8s Cluster]
├── default namespace (App)
├── jenkins namespace (CI/CD)
└── kube-prometheus namespace (Monitoring)
```

---

## 📊 Data Flow Summary

### API Request Flow
```
Browser → HTTP Request → K8s Service → Load Balancer → Pod
→ Flask Route → Logic (Cart/Checkout/etc) → Response
```

### Database Query Flow
```
Flask App → Supabase Client → Supabase API → PostgreSQL
→ Query Result → Flask → Render Template → Send to Browser
```

### Session Management Flow
```
Request → Flask reads cookie → Deserialize session
→ Access cart/order_history → Modify if needed
→ Serialize → Send back in cookie
```

### Deployment Flow
```
Git Commit → Jenkins Webhook → Pipeline (5 stages)
→ SonarQube Check → Docker Build → kubectl Deploy
→ K8s creates pods → Service routes traffic → App live
```

---

## ✅ Essential Components for Your Architecture Diagram

### Must Include:
1. Browser/Client
2. K8s Service (NodePort)
3. Pods (Flask containers)
4. Deployment controller
5. Supabase database
6. Jenkins server
7. Docker build step
8. Git repository
9. Secrets/ConfigMaps
10. Persistent storage (jenkins-pvc)

### Good to Include:
1. Health check endpoints
2. Environment variables injection
3. Session storage
4. SonarQube integration
5. kubectl commands
6. Docker socket mount (DinD)
7. Namespace boundaries

### Optional:
1. Prometheus/Grafana (not fully used)
2. AlertManager
3. NetworkPolicy
4. Ingress controller
5. Certificate management
6. Log aggregation

---

## 🔍 Component Interconnections

| Component | Connects To | Protocol | Purpose |
|-----------|------------|----------|---------|
| Browser | K8s Service | HTTP | Request app |
| K8s Service | Pods | Internal | Route traffic |
| Pod | Supabase | HTTPS | Query products |
| Pod | Session Cookie | HTTP | Manage cart |
| Jenkins | Git | SSH/HTTPS | Pull code |
| Jenkins | SonarQube | HTTP | Quality check |
| Jenkins | Docker | Unix Socket | Build images |
| Jenkins | K8s API | HTTPS | Deploy |
| K8s Deployment | Secret | Internal | Read env vars |
| Jenkins | PVC | Block Storage | Persistent data |

---

## 🎯 Architecture Diagram Recommendations

### Simple Architecture (3-layer)
```
[Client Layer]
    ↓
[Application Layer (K8s + Flask)]
    ↓
[Data Layer (Database + Storage)]
```

### Detailed Architecture (6-layer)
```
[User & Access]
    ↓
[API Gateway / Service]
    ↓
[Application Pods]
    ↓
[Kubernetes Orchestration]
    ↓
[Data & Storage]
    ↓
[CI/CD Pipeline]
```

### Full Stack Architecture (with CI/CD)
Include all components from this checklist in separate swim lanes:
- User Lane
- Network Lane
- Application Lane
- Storage Lane
- CI/CD Lane
- Infrastructure Lane

