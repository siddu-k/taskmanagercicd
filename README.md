# Task Manager CICD - Complete Documentation

## Project Overview

**Repository:** [siddu-k/taskmanagercicd](https://github.com/siddu-k/taskmanagercicd)  
**Language Composition:** Java (94.2%), Dockerfile (5.8%)  
**Type:** Spring Boot Application with CI/CD Pipeline  
**Description:** A Task Management System built for DevOps practices, featuring a complete CI/CD pipeline with containerization, security scanning, and GitOps deployment.

---

## Architecture & Technology Stack

### Backend Framework
- **Spring Boot 3.2.3** - REST API framework
- **Java 17** - Programming language
- **PostgreSQL** - Database
- **Spring Data JPA** - ORM and data persistence
- **Spring Boot Actuator** - Application monitoring
- **Micrometer Prometheus** - Metrics collection

### DevOps & Infrastructure
- **Docker** - Containerization (multi-stage builds)
- **Docker Compose** - Local development orchestration
- **Kubernetes** - Container orchestration
- **Kind (Kubernetes in Docker)** - Local K8s cluster
- **Jenkins** - CI/CD pipeline automation
- **ArgoCD** - GitOps deployment tool
- **Trivy** - Container security scanning
- **Gitleaks** - Secret scanning
- **SonarQube** - Code quality analysis
- **AWS ECR** - Docker registry

---

## Project Structure

```
taskmanagercicd/
├── src/
│   └── main/
│       ├── java/com/example/taskmanager/
│       │   ├── TaskManagementApplication.java    # Spring Boot entry point
│       │   ├── controller/
│       │   │   └── TaskController.java           # REST API endpoints
│       │   ├── entity/
│       │   │   └── Task.java                     # JPA entity model
│       │   └── repository/
│       │       └── TaskRepository.java           # Data access layer
│       └── resources/
│           └── application.properties            # Application configuration
├── k8s/
│   ├── deployment.yaml                           # Kubernetes deployment
│   └── service.yaml                              # Kubernetes service
├── pom.xml                                       # Maven build configuration
├── Dockerfile                                    # Multi-stage Docker build
├── docker-compose.yml                            # Local development stack
├── Jenkinsfile                                   # CI/CD pipeline definition
├── kind-config.yaml                              # Local K8s cluster config
├── argocd-app.yaml                               # ArgoCD application manifest
├── .gitignore                                    # Git ignore patterns
└── .gitleaks.toml                                # Secret scanning config
```

---

## Core Application Components

### 1. **Task Entity** (`src/main/java/com/example/taskmanager/entity/Task.java`)

JPA entity representing a task with the following properties:

| Field | Type | Details |
|-------|------|---------|
| `id` | Long | Primary key (auto-generated) |
| `title` | String | Task title (required) |
| `description` | String | Task description (TEXT) |
| `status` | String | Task status (required, default: "PENDING") |
| `createdAt` | LocalDateTime | Timestamp (auto-set on creation) |

### 2. **Task Repository** (`src/main/java/com/example/taskmanager/repository/TaskRepository.java`)

Spring Data JPA repository providing CRUD operations:
- Extends `JpaRepository<Task, Long>`
- Automatic database query generation

### 3. **Task Controller** (`src/main/java/com/example/taskmanager/controller/TaskController.java`)

REST API endpoints at `/tasks`:

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/tasks` | Create a new task |
| `GET` | `/tasks` | Get all tasks |
| `GET` | `/tasks/{id}` | Get task by ID |
| `PUT` | `/tasks/{id}` | Update task |
| `DELETE` | `/tasks/{id}` | Delete task |
| `PATCH` | `/tasks/{id}/complete` | Mark task as completed |

**Features:**
- Automatic status initialization (defaults to "PENDING")
- Proper HTTP status codes (201 Created, 200 OK, 404 Not Found, 204 No Content)
- Complete CRUD operations with error handling

---

## Configuration

### Application Properties (`src/main/resources/application.properties`)

```properties
# Application metadata
spring.application.name=taskmanager

# Database configuration (environment variable overrides)
spring.datasource.url=jdbc:postgresql://${DB_HOST:localhost}:${DB_PORT:5432}/${DB_NAME:taskdb}
spring.datasource.username=${DB_USER:postgres}
spring.datasource.password=${DB_PASSWORD:postgres}

# Hibernate/JPA configuration
spring.jpa.hibernate.ddl-auto=update              # Auto schema creation
spring.jpa.show-sql=true                          # Log SQL queries
spring.jpa.properties.hibernate.dialect=...PostgreSQLDialect

# Server configuration
server.port=8080

# Actuator/Prometheus monitoring
management.endpoints.web.exposure.include=health,info,prometheus
management.metrics.tags.application=taskmanager
```

---

## Containerization

### Dockerfile (Multi-stage Build)

**Stage 1: Build**
- Base image: `maven:3.9.6-eclipse-temurin-17`
- Compiles the Maven project
- Skips tests during build for speed

**Stage 2: Runtime**
- Base image: `eclipse-temurin:17-jre-jammy`
- Copies only the compiled JAR
- Exposes port 8080
- Lightweight production image

### Docker Compose (`docker-compose.yml`)

Orchestrates local development with two services:

**PostgreSQL Service:**
- Image: `postgres:15-alpine`
- Port: 5432
- Database: `taskdb`
- Includes health checks
- Persistent volume: `postgres_data`

**Application Service:**
- Image: `taskmanager:latest`
- Port: 8081 (mapped to container 8080)
- Environment variables for database connection
- Depends on healthy PostgreSQL service

---

## Kubernetes Deployment

### Kind Cluster Configuration (`kind-config.yaml`)

**Cluster Setup:**
- 1 control-plane node + 2 worker nodes
- Port mappings for external access (8082 → 30080, 8443 → 30443)

### Kubernetes Manifests

**Deployment** (`k8s/deployment.yaml`):
- Replicas: 1
- Container image: `taskmanager:latest`
- Resource requests: 256Mi memory, 250m CPU
- Resource limits: 512Mi memory, 500m CPU
- Environment variables for database configuration
- Prometheus monitoring annotations

**Service** (`k8s/service.yaml`):
- Type: ClusterIP
- Port: 80 → 8080 (container)
- Label selector: `app: taskmanager`

---

## CI/CD Pipeline

### Jenkins Pipeline (`Jenkinsfile`)

**Pipeline Stages:**

1. **Checkout**
   - Clones the repository

2. **Build**
   - `mvn clean package -DskipTests`
   - Compiles and packages the application

3. **Test**
   - `mvn test`
   - Runs unit tests

4. **SonarQube Analysis**
   - Performs code quality analysis
   - Checks code standards and vulnerabilities

5. **Secret Scan**
   - `gitleaks detect --source . -v`
   - Scans for exposed secrets and credentials

6. **Build Docker Image**
   - Creates Docker image with build number tag
   - Tags with both build number and "latest"
   - Command: `docker build -t taskmanager:${BUILD_NUMBER} -t taskmanager:latest .`

7. **Trivy Scan**
   - `trivy image taskmanager:${IMAGE_TAG}`
   - Scans container for vulnerabilities

8. **Push to ECR**
   - Authenticates to AWS ECR (ap-south-1 region)
   - Tags image: `484907501702.dkr.ecr.ap-south-1.amazonaws.com/taskmanager:${BUILD_NUMBER}`
   - Pushes to AWS Elastic Container Registry

9. **Update GitOps Repo**
   - Clones separate `taskmanager-gitops` repository
   - Updates Helm chart values with new image tag
   - Commits and pushes changes to trigger ArgoCD deployment

**Environment Variables:**
- `IMAGE_NAME`: taskmanager
- `IMAGE_TAG`: ${BUILD_NUMBER}

**Tools:**
- JDK 17
- Maven 3

---

## GitOps & ArgoCD

### ArgoCD Application (`argocd-app.yaml`)

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: taskmanager-app
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/siddu-k/taskmanager-gitops.git
    path: helm/taskmanager-chart
    targetRevision: HEAD
  destination:
    server: https://kubernetes.default.svc
    namespace: default
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

**Features:**
- **Automated sync** with prune and self-heal enabled
- **GitOps repository**: Separate `taskmanager-gitops` repo for Helm charts
- **Helm deployment**: Uses Helm chart for Kubernetes deployment
- **Pull-based deployment**: ArgoCD continuously monitors Git for changes

---

## Deployment Flow

```
Code Commit
    ↓
Jenkins Checkout
    ↓
Build & Test (Maven)
    ↓
Code Quality (SonarQube)
    ↓
Secret Scanning (Gitleaks)
    ↓
Build Docker Image
    ↓
Container Scanning (Trivy)
    ↓
Push to AWS ECR
    ↓
Update GitOps Repo (Helm values)
    ↓
ArgoCD Detects Changes
    ↓
Deploy to Kubernetes
```

---

## Security Features

1. **Secret Scanning**: Gitleaks configuration prevents credential leaks
2. **Container Scanning**: Trivy scans Docker images for vulnerabilities
3. **Code Quality**: SonarQube analyzes code for security issues
4. **Environment Variables**: Sensitive data (passwords, credentials) passed via environment variables
5. **AWS IAM Integration**: Secure authentication to ECR

---

## Build & Deployment Instructions

### Local Development (Docker Compose)

```bash
# Build and start services
docker-compose up --build

# Application will be available at http://localhost:8081
# Database: localhost:5432
```

### Kubernetes Deployment (Kind)

```bash
# Create Kind cluster
kind create cluster --config kind-config.yaml

# Apply ArgoCD manifest
kubectl apply -f argocd-app.yaml

# Check deployment status
kubectl get deployments
kubectl get pods
```

### Jenkins Pipeline Trigger

The pipeline automatically triggers on commits to the main branch and:
1. Runs all quality checks
2. Builds and scans Docker image
3. Pushes to ECR
4. Updates GitOps repository
5. ArgoCD automatically deploys changes to Kubernetes

---

## Monitoring

### Prometheus Metrics

Application exposes metrics at `/actuator/prometheus`:
- Health checks: `/actuator/health`
- Application info: `/actuator/info`
- Custom metrics via Micrometer

Kubernetes deployment includes Prometheus annotations for automatic scraping.

---

## Dependencies

**Maven Dependencies:**
- `spring-boot-starter-data-jpa` - Data persistence
- `spring-boot-starter-web` - REST API
- `postgresql` - Database driver
- `spring-boot-starter-test` - Testing framework
- `spring-boot-starter-actuator` - Application monitoring
- `micrometer-registry-prometheus` - Prometheus metrics

---

## Summary

This is a **production-ready Task Management System** with:
- ✅ Clean Spring Boot REST API
- ✅ PostgreSQL database integration
- ✅ Multi-stage Docker builds for efficiency
- ✅ Comprehensive CI/CD pipeline with security scanning
- ✅ Kubernetes-ready deployment manifests
- ✅ GitOps workflow with ArgoCD
- ✅ Monitoring and metrics collection
- ✅ AWS ECR integration

The project demonstrates DevOps best practices including containerization, infrastructure-as-code, automated testing, security scanning, and continuous deployment.
