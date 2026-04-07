# 🛒 E-Commerce DevOps Implementation

> **A production-grade, end-to-end DevOps implementation** of the OpenTelemetry Astronomy Shop — a polyglot microservices e-commerce application — deployed on **AWS EKS** using Docker, Terraform, Kubernetes, Helm, GitHub Actions, and ArgoCD, with full OpenTelemetry observability.

> **Note:** The application source code is forked from [open-telemetry/opentelemetry-demo](https://github.com/open-telemetry/opentelemetry-demo). Full credit to the OpenTelemetry Authors and all contributors for this exceptional open-source reference project.

---

## 📑 Table of Contents

- [Project Overview](#-project-overview)
- [Repository Structure](#-repository-structure)
- [Application Architecture](#-application-architecture)
  - [Microservices](#microservices)
  - [Service Communication](#service-communication)
  - [Infrastructure Services](#infrastructure-services)
- [Technology Stack](#-technology-stack)
- [DevOps Pipeline — Big Picture](#-devops-pipeline--big-picture)
- [Section 1: AWS Setup & Prerequisites](#-section-1-aws-setup--prerequisites)
  - [IAM User Setup](#11-create-an-iam-user-non-root)
  - [EC2 Instance Setup](#13-launch-ec2-instance-jump-server)
  - [Tool Installation](#14-install-required-tools)
- [Section 2: Docker Fundamentals](#-section-2-docker-fundamentals)
  - [Essential Docker Commands](#21-essential-docker-commands)
  - [Working with AWS ECR](#22-working-with-aws-ecr)
- [Section 3: Dockerfile Mastery](#-section-3-dockerfile-mastery)
  - [Dockerfile Instructions](#dockerfile-instructions)
  - [Multi-Stage Build Example](#multi-stage-build-example-go-service)
- [Section 4: Local Development with Docker Compose](#-section-4-local-development-with-docker-compose)
  - [Running the Full Stack](#41-start-the-full-stack)
  - [Accessing the Application](#43-access-the-running-application)
- [Section 5: Containerizing & Pushing Images](#-section-5-containerizing--pushing-images)
- [Section 6: Infrastructure as Code with Terraform](#-section-6-infrastructure-as-code-with-terraform)
  - [Terraform Core Concepts](#61-terraform-core-concepts)
  - [Remote State with S3 & DynamoDB](#63-remote-state-with-s3--dynamodb-locking)
  - [Provisioning VPC with Terraform](#64-provisioning-vpc-with-terraform)
- [Section 7: Provisioning AWS EKS with Terraform](#-section-7-provisioning-aws-eks-with-terraform)
  - [EKS Cluster Configuration](#71-eks-cluster-configuration)
  - [Connect kubectl to EKS](#72-configure-kubectl-to-connect-to-eks)
- [Section 8: Kubernetes on EKS](#-section-8-kubernetes-on-eks)
  - [Core Kubernetes Concepts Used](#core-kubernetes-concepts-used)
  - [Deploying the Application](#86-applying-manifests)
  - [Kubernetes Ingress & Load Balancing](#-kubernetes-ingress--load-balancing)
- [Section 9: Secrets Management](#-section-9-secrets-management)
  - [Kubernetes Secrets Basics](#91-kubernetes-secrets-basics)
  - [AWS Secrets Manager](#92-aws-secrets-manager)
  - [EKS Pod Identity Agent](#93-eks-pod-identity-agent)
- [Section 10: CI/CD Pipeline](#-section-10-cicd-pipeline-github-actions--argocd)
  - [GitHub Actions — CI](#101-github-actions--continuous-integration)
  - [ArgoCD — GitOps CD](#102-argocd--continuous-deployment-gitops)
  - [End-to-End CI/CD Flow](#103-end-to-end-cicd-flow)
- [Deploying via Helm](#-deploying-via-helm)
- [Observability](#-observability)
- [Getting Started](#-getting-started)
- [Troubleshooting](#-troubleshooting)
- [Prerequisites Summary](#-prerequisites-summary)
- [Credits](#-credits)

---

## 🔭 Project Overview

This project takes a real-world, polyglot microservices e-commerce application — the **OpenTelemetry Astronomy Shop** — and builds a complete, production-grade DevOps pipeline around it. The app simulates an online store where users browse astronomy products (telescopes, lenses, accessories), add them to a cart, and check out. Behind the scenes, 15+ microservices written in 10+ programming languages collaborate via gRPC and HTTP to handle every step of that flow.

On top of this application, every major domain of modern cloud-native DevOps is covered:

| Domain | Tools Used |
|---|---|
| Containerization | Docker, Docker Compose, AWS ECR |
| Infrastructure as Code | Terraform |
| Cloud Platform | AWS (EKS, VPC, EC2, S3, DynamoDB, IAM, Secrets Manager) |
| Container Orchestration | Kubernetes (AWS EKS) |
| Package Management | Helm (OpenTelemetry official chart) |
| CI Pipeline | GitHub Actions |
| CD / GitOps | ArgoCD |
| Observability | OpenTelemetry, Jaeger, Prometheus, Grafana |
| Secrets & Security | AWS Secrets Manager, EKS Pod Identity, RBAC |

---

## 📁 Repository Structure

```
📁 E-COMMERCE-DEVOPS-Implementation/
│
├── 📁 .github/
│   └── 📁 workflows/                          ← GitHub Actions CI/CD pipeline definitions
│       └── *.yml                              ← Workflow files (build, push, lint)
│
├── 📁 eks-install/                            ← Terraform code to provision AWS EKS
│   ├── main.tf                                ← EKS cluster, VPC, node groups
│   ├── variables.tf                           ← Input variables (region, instance type, etc.)
│   └── outputs.tf                             ← Cluster endpoint, kubeconfig outputs
│
├── 📁 internal/
│   └── 📁 tools/                              ← Internal developer tooling & helper scripts
│
├── 📁 kubernetes/                             ← Raw Kubernetes manifests
│   └── 📁 productcatalog/
│       ├── deployment.yaml                    ← Deployment spec (replicas, image, resources)
│       └── svc.yaml                           ← ClusterIP Service for internal communication
│
├── 📁 pb/                                     ← Protocol Buffer (.proto) gRPC definitions
│   └── demo.proto                             ← Shared service interface contracts
│
├── 📁 src/                                    ← Microservice source code (10+ languages)
│   ├── 📁 accountingservice/                  ← .NET/C# — order transaction accounting
│   ├── 📁 adservice/                          ← Java — contextual advertisements
│   ├── 📁 cartservice/                        ← .NET/C# — shopping cart state via Redis
│   ├── 📁 checkoutservice/                    ← Go — full checkout orchestration
│   ├── 📁 currencyservice/                    ← C++ — real-time currency conversion
│   ├── 📁 emailservice/                       ← Ruby — order confirmation emails
│   ├── 📁 flagd-ui/                           ← TypeScript — feature flag management UI
│   ├── 📁 frauddetectionservice/              ← Kotlin — Kafka-based fraud detection
│   ├── 📁 frontend/                           ← TypeScript/Next.js — user-facing web UI
│   ├── 📁 frontendproxy/                      ← Envoy/C++ — reverse proxy & routing
│   ├── 📁 imageprovider/                      ← nginx — serves product images
│   ├── 📁 kafka/                              ← Kafka setup for async event streaming
│   ├── 📁 loadgenerator/                      ← Python/Locust — simulates user traffic
│   ├── 📁 otelcollector/                      ← OTel Collector config — aggregates signals
│   ├── 📁 paymentservice/                     ← JavaScript/Node.js — payment processing
│   ├── 📁 productcatalogservice/              ← Go — product database & catalog API
│   ├── 📁 quoteservice/                       ← PHP — shipping cost quote generation
│   ├── 📁 recommendationservice/              ← Python — product recommendation engine
│   └── 📁 shippingservice/                    ← Rust — shipping rate calculation
│
├── 📁 test/                                   ← Integration & end-to-end test suites
│
├── .dockerignore                              ← Files excluded from Docker build context
├── .env                                       ← Default environment variables for all services
├── .env.arm64                                 ← ARM64-specific image tag overrides
├── .env.override                              ← Local override values (not committed)
├── .gitattributes / .gitignore                ← Git configuration files
├── .licenserc.json                            ← Apache-2.0 license header enforcement
├── .markdownlint.yaml / .yamllint             ← Linting rules for docs and YAML
├── buildkitd.toml                             ← BuildKit config for multi-platform builds
├── CHANGELOG.md                               ← Version history & release notes
├── CONTRIBUTING.md                            ← Contribution guidelines
├── docker-compose.yml                         ← Full local stack — all 15+ services
├── docker-compose.minimal.yml                 ← Lightweight stack — core services only
├── docker-compose-tests.yml                   ← Compose config for integration tests
├── docker-gen-proto.sh                        ← Regenerate gRPC stubs via Docker
├── ide-gen-proto.sh                           ← Regenerate gRPC stubs locally in IDE
├── LICENSE                                    ← Apache-2.0 open-source license
├── Makefile                                   ← Developer shortcuts (build, test, run)
├── package.json / package-lock.json           ← Node.js workspace & shared JS tooling
├── README.md                                  ← This file
└── renovate.json5                             ← Automated dependency update config
```

---

## 🏗 Application Architecture

### Microservices

The Astronomy Shop consists of **15+ microservices** written in **10+ programming languages**, making it an authentic polyglot application:

| Service | Language | Responsibility |
|---|---|---|
| **Frontend** | TypeScript (Next.js) | Web UI, API Gateway |
| **Frontend Proxy** | C++ (Envoy) | Reverse proxy & request routing |
| **Ad Service** | Java | Serves contextual advertisements |
| **Cart Service** | .NET (C#) | Manages shopping cart state |
| **Checkout Service** | Go | Orchestrates the full checkout flow |
| **Currency Service** | C++ | Real-time currency conversion |
| **Email Service** | Ruby | Sends order confirmation emails |
| **Fraud Detection** | Kotlin | Kafka-based fraud detection |
| **Payment Service** | JavaScript (Node.js) | Processes payments |
| **Product Catalog** | Go | Product database & catalog API |
| **Recommendation** | Python | Product recommendations |
| **Shipping Service** | Rust | Shipping cost calculation |
| **Quote Service** | PHP | Shipping quote generation |
| **Accounting Service** | .NET (C#) | Transaction accounting |
| **Image Provider** | C++ (nginx) | Hosts product images |
| **Load Generator** | Python (Locust) | Simulates realistic user traffic |

### Service Communication

All services communicate via **gRPC** and **HTTP**, with every request traced end-to-end using the OpenTelemetry SDK for each language.

```
User Browser
    │
    ▼
Frontend Proxy (Envoy)
    │
    ▼
Frontend (TypeScript / Next.js)
    ├──► Product Catalog Service (Go)      ──► PostgreSQL
    ├──► Ad Service (Java)
    ├──► Cart Service (.NET)               ──► Redis (Valkey)
    ├──► Recommendation Service (Python)   ──► Product Catalog
    └──► Checkout Service (Go)
             ├──► Cart Service
             ├──► Currency Service (C++)
             ├──► Payment Service (Node.js)
             ├──► Email Service (Ruby)
             ├──► Shipping Service (Rust)
             │        └──► Quote Service (PHP)
             └──► Product Catalog
                      └──► Fraud Detection (Kafka ──► Kotlin)
                               └──► Accounting (.NET)
```

### Infrastructure Services

In addition to application microservices, the following infrastructure components run as part of the full stack:

| Component | Purpose |
|---|---|
| **OpenTelemetry Collector** | Aggregates traces, metrics, and logs from all services and exports them to backends |
| **Jaeger** | Distributed tracing backend and UI |
| **Prometheus** | Metrics scraping and storage |
| **Grafana** | Dashboards for metrics visualization |
| **Kafka** | Message queue between checkout → fraud detection → accounting |
| **PostgreSQL** | Product catalog persistent store |
| **Redis (Valkey)** | Cart state caching |
| **Flagd** | Feature flag service for runtime toggles |

---

## 🛠 Technology Stack

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                          FULL TECHNOLOGY STACK                              │
├──────────────────────┬──────────────────────────────────────────────────────┤
│ APPLICATION          │ Go, TypeScript, Python, Java, .NET/C#, Rust,         │
│                      │ Ruby, Kotlin, C++, JavaScript, PHP                   │
├──────────────────────┼──────────────────────────────────────────────────────┤
│ CONTAINERIZATION     │ Docker, Docker Compose, Multi-stage Dockerfiles      │
│                      │ AWS ECR (Elastic Container Registry)                 │
├──────────────────────┼──────────────────────────────────────────────────────┤
│ INFRASTRUCTURE       │ Terraform — VPC, EKS, IAM, S3 remote state,          │
│                      │ DynamoDB state locking                               │
├──────────────────────┼──────────────────────────────────────────────────────┤
│ AWS SERVICES         │ EKS, EC2, VPC, IAM, S3, Route53, ACM,                │
│                      │ Secrets Manager, CloudWatch                          │
├──────────────────────┼──────────────────────────────────────────────────────┤
│ KUBERNETES           │ Pods, Deployments, Services (all 5 types),           │
│                      │ ConfigMaps, Secrets, StatefulSets, Ingress, HPA      │
├──────────────────────┼──────────────────────────────────────────────────────┤
│ HELM                 │ Official OpenTelemetry Demo chart (external)         │
├──────────────────────┼──────────────────────────────────────────────────────┤
│ CI/CD                │ GitHub Actions (CI) + ArgoCD (GitOps CD)             │
├──────────────────────┼──────────────────────────────────────────────────────┤
│ OBSERVABILITY        │ OpenTelemetry (OTLP), Jaeger, Prometheus, Grafana    │
├──────────────────────┼──────────────────────────────────────────────────────┤
│ SECRETS & SECURITY   │ AWS Secrets Manager, EKS Pod Identity (IRSA), RBAC   │
└──────────────────────┴──────────────────────────────────────────────────────┘
```

---

## 🔄 DevOps Pipeline — Big Picture

```
Developer pushes code
        │
        ▼
┌───────────────────┐
│  GitHub Actions   │  ← CI Pipeline
│                   │
│  1. Checkout code │
│  2. Build Docker  │
│     image         │
│  3. Run tests     │
│  4. Push image    │
│     to AWS ECR    │
│  5. Update image  │
│     tag in k8s/   │
│     manifests     │
└────────┬──────────┘
         │  Git commit back to repo
         ▼
┌───────────────────┐
│     ArgoCD        │  ← CD / GitOps
│                   │
│  1. Detects new   │
│     commit in     │
│     kubernetes/   │
│  2. Syncs desired │
│     state to EKS  │
│  3. Rolls out new │
│     Deployment    │
│  4. Self-heals    │
│     on any drift  │
└────────┬──────────┘
         │  Deploys to
         ▼
┌──────────────────────────────────┐
│        AWS EKS Cluster           │
│                                  │
│  ┌──────────────────────────┐    │
│  │   Microservice Pods      │    │
│  │  (15+ services, all      │    │
│  │   OTel instrumented)     │    │
│  └──────────────────────────┘    │
└──────────────────────────────────┘
         │
         ▼
┌──────────────────────────────────┐
│      Observability Stack         │
│  OTel Collector → Jaeger         │
│  OTel Collector → Prometheus     │
│                 → Grafana        │
└──────────────────────────────────┘
```

---

## ☁️ Section 1: AWS Setup & Prerequisites

### 1.1 Create an IAM User (non-root)

Never use the root account for daily operations. Create a dedicated IAM user:

1. Go to **IAM → Users → Create User**
2. Attach the following managed policies:
   - `AmazonEKSFullAccess`
   - `AmazonEC2FullAccess`
   - `AmazonS3FullAccess`
   - `IAMFullAccess`
   - `AmazonVPCFullAccess`
   - `AmazonDynamoDBFullAccess`
3. Generate **Access Key ID** and **Secret Access Key**

### 1.2 Configure AWS CLI

```bash
aws configure
# AWS Access Key ID: <your-key>
# AWS Secret Access Key: <your-secret>
# Default region: us-east-1
# Output format: json
```

### 1.3 Launch EC2 Instance (Jump Server)

All commands (Docker, kubectl, Terraform, Helm) are run from this EC2 instance — no local installation needed.

- **AMI:** Amazon Linux 2023
- **Instance Type:** `t2.medium` (minimum 2 vCPU, 4 GB RAM)
- **Storage:** 30 GB gp3
- **Security Group Inbound Rules:**

| Port | Protocol | Source | Purpose |
|---|---|---|---|
| 22 | TCP | Your IP | SSH access |
| 80 | TCP | 0.0.0.0/0 | HTTP |
| 443 | TCP | 0.0.0.0/0 | HTTPS |

```bash
chmod 400 your-key.pem
ssh -i "your-key.pem" ec2-user@<public-ip>
```

### 1.4 Install Required Tools

**Docker:**
```bash
sudo yum update -y
sudo yum install -y docker
sudo systemctl start docker && sudo systemctl enable docker
sudo usermod -aG docker ec2-user
newgrp docker
docker --version
```

**kubectl:**
```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl && sudo mv kubectl /usr/local/bin/
kubectl version --client
```

**Terraform:**
```bash
sudo yum install -y yum-utils
sudo yum-config-manager --add-repo https://rpm.releases.hashicorp.com/AmazonLinux/hashicorp.repo
sudo yum install -y terraform
terraform --version
```

**Helm:**
```bash
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
helm version
```

---

## 🐳 Section 2: Docker Fundamentals

### 2.1 Essential Docker Commands

```bash
# Pull an image
docker pull nginx:latest

# Run a container
docker run -d -p 8080:80 --name my-nginx nginx

# Execute into a running container
docker exec -it my-nginx /bin/sh

# View logs
docker logs my-nginx -f

# Stop, start, remove a container
docker stop my-nginx
docker start my-nginx
docker rm my-nginx

# Remove an image
docker rmi nginx:latest

# Inspect container details
docker inspect my-nginx

# List running containers / all containers
docker ps
docker ps -a
```

### 2.2 Working with AWS ECR

```bash
# Create a repository
aws ecr create-repository --repository-name product-catalog --region us-east-1

# Authenticate Docker to ECR
aws ecr get-login-password --region us-east-1 | \
  docker login --username AWS --password-stdin \
  <account-id>.dkr.ecr.us-east-1.amazonaws.com

# Tag and push
docker tag product-catalog:v1.0 \
  <account-id>.dkr.ecr.us-east-1.amazonaws.com/product-catalog:v1.0
docker push <account-id>.dkr.ecr.us-east-1.amazonaws.com/product-catalog:v1.0
```

---

## 📄 Section 3: Dockerfile Mastery

Every microservice in `src/` has its own `Dockerfile`. Here is the complete set of Dockerfile instructions used across this project:

#### Dockerfile Instructions

| Instruction | Purpose |
|---|---|
| `FROM` | Base image — use minimal images (`distroless`, `alpine`) |
| `LABEL` | Image metadata (author, version) |
| `ARG` | Build-time variables |
| `ENV` | Runtime environment variables |
| `WORKDIR` | Set working directory inside the container |
| `COPY` | Copy files from host to image |
| `ADD` | Like COPY but can extract archives (use sparingly) |
| `RUN` | Execute commands during build — combine with `&&` to reduce layers |
| `EXPOSE` | Document the port the container listens on |
| `USER` | Run as non-root for security |
| `HEALTHCHECK` | Container health check command |
| `ENTRYPOINT` | Fixed startup command |
| `CMD` | Default arguments to `ENTRYPOINT` |

#### Multi-Stage Build Example (Go service)

```dockerfile
# ── Stage 1: Build ─────────────────────────────────────
FROM golang:1.22-alpine AS builder
WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -o service .

# ── Stage 2: Run (minimal production image) ────────────
FROM gcr.io/distroless/static-debian12
WORKDIR /
COPY --from=builder /app/service /service
EXPOSE 8080
USER nonroot:nonroot
ENTRYPOINT ["/service"]
```

Multi-stage builds produce **significantly smaller final images** by leaving the compiler, build tools, and intermediate artifacts entirely in the build stage.

---

## 🐙 Section 4: Local Development with Docker Compose

The `docker-compose.yml` at the root of this repo starts the entire application stack locally — all 15+ services plus the full observability stack (Jaeger, Prometheus, Grafana).

### 4.1 Start the Full Stack

```bash
# Clone and enter the repo
git clone https://github.com/Shyam-Kumar-Khatri/E-COMMERCE-DEVOPS-Implementation.git
cd E-COMMERCE-DEVOPS-Implementation

# Start everything in the background
docker compose up -d

# Check all containers are healthy
docker compose ps

# View logs for a specific service
docker compose logs -f frontend
```

### 4.2 Minimal Stack (Lower Resource Usage)

```bash
docker compose -f docker-compose.minimal.yml up -d
```

### 4.3 Access the Running Application

| Interface | URL |
|---|---|
| 🛒 Astronomy Shop (Frontend) | http://localhost:8080 |
| 📊 Grafana Dashboards | http://localhost:8080/grafana |
| 🔍 Jaeger (Distributed Traces) | http://localhost:8080/jaeger/ui |
| 📈 Prometheus | http://localhost:9090 |
| 🚩 Feature Flags (Flagd UI) | http://localhost:8080/feature |
| ⚡ Load Generator (Locust) | http://localhost:8089 |

### 4.4 Stop & Clean Up

```bash
docker compose down          # Stop containers
docker compose down -v       # Stop & remove volumes
```

### 4.5 Running Integration Tests

```bash
docker compose -f docker-compose-tests.yml up --abort-on-container-exit
```

---

## 📦 Section 5: Containerizing & Pushing Images

Once the application runs locally with Docker Compose, each microservice image is built and pushed to AWS ECR for cloud deployment on EKS.

```bash
# Build a specific service image
docker build -t product-catalog:latest ./src/productcatalogservice/

# Build for multiple platforms (AMD64 + ARM64)
docker buildx build \
  --platform linux/amd64,linux/arm64 \
  -t <ecr-url>/product-catalog:v1.0 \
  --push \
  ./src/productcatalogservice/
```

The `buildkitd.toml` at the root configures the BuildKit daemon for multi-platform builds. The `Makefile` contains shorthand targets to build and push all services at once:

```bash
make build         # Build all service images
make push          # Push all images to ECR
```

---

## 🏗 Section 6: Infrastructure as Code with Terraform

All AWS infrastructure is provisioned using Terraform, stored in the `eks-install/` directory.

### 6.1 Terraform Core Concepts

| Concept | Description |
|---|---|
| `provider` | Plugin to interact with the AWS API |
| `resource` | A cloud object to create (e.g., `aws_eks_cluster`) |
| `variable` | Parameterize configs for reuse across environments |
| `output` | Export values after apply (e.g., cluster endpoint) |
| `module` | Reusable group of resources |
| `state` | Terraform's record of real-world infrastructure |

### 6.2 Terraform Lifecycle

```bash
cd eks-install/

terraform init      # Download providers & initialize backend
terraform plan      # Preview what will be created/changed/destroyed
terraform apply     # Apply changes (type 'yes' to confirm)
terraform destroy   # Tear everything down
```

### 6.3 Remote State with S3 & DynamoDB Locking

Storing state locally works for a single developer, but in any team environment remote state is required to prevent corruption.

**Step 1 — Create the S3 bucket:**
```bash
aws s3api create-bucket \
  --bucket my-devops-tf-state \
  --region us-east-1

aws s3api put-bucket-versioning \
  --bucket my-devops-tf-state \
  --versioning-configuration Status=Enabled
```

**Step 2 — Create DynamoDB table for state locking:**
```bash
aws dynamodb create-table \
  --table-name tf-state-lock \
  --attribute-definitions AttributeName=LockID,AttributeType=S \
  --key-schema AttributeName=LockID,KeyType=HASH \
  --billing-mode PAY_PER_REQUEST
```

**Step 3 — Configure backend in `eks-install/main.tf`:**
```hcl
terraform {
  backend "s3" {
    bucket         = "my-devops-tf-state"
    key            = "eks/terraform.tfstate"
    region         = "us-east-1"
    dynamodb_table = "tf-state-lock"
    encrypt        = true
  }
}
```

### 6.4 Provisioning VPC with Terraform

The VPC separates public-facing resources (load balancers) from internal workloads (EKS nodes).

```hcl
module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "5.0.0"

  name = "eks-vpc"
  cidr = "10.0.0.0/16"

  azs             = ["us-east-1a", "us-east-1b"]
  private_subnets = ["10.0.1.0/24", "10.0.2.0/24"]
  public_subnets  = ["10.0.101.0/24", "10.0.102.0/24"]

  enable_nat_gateway   = true
  single_nat_gateway   = true   # one NAT GW saves cost (use multiple for HA)
  enable_dns_hostnames = true

  # Required tags for EKS to discover subnets
  private_subnet_tags = {
    "kubernetes.io/role/internal-elb" = "1"
  }
  public_subnet_tags = {
    "kubernetes.io/role/elb" = "1"
  }
}
```

---

## ☸️ Section 7: Provisioning AWS EKS with Terraform

The `eks-install/` directory provisions the full EKS cluster with all required IAM roles and node groups.

### 7.1 EKS Cluster Configuration

```hcl
module "eks" {
  source  = "terraform-aws-modules/eks/aws"
  version = "19.0.0"

  cluster_name    = "ecommerce-devops-cluster"
  cluster_version = "1.29"

  vpc_id     = module.vpc.vpc_id
  subnet_ids = module.vpc.private_subnets

  cluster_endpoint_public_access = true

  eks_managed_node_groups = {
    general = {
      desired_size   = 2
      min_size       = 2
      max_size       = 5
      instance_types = ["t3.medium"]
      capacity_type  = "ON_DEMAND"
    }
  }
}
```

### 7.2 Configure kubectl to Connect to EKS

After `terraform apply` completes:

```bash
aws eks update-kubeconfig \
  --region us-east-1 \
  --name ecommerce-devops-cluster

# Verify nodes are ready
kubectl get nodes
kubectl get nodes -o wide
```

### 7.3 IAM Roles Created by Terraform

Terraform automatically creates and attaches the required IAM roles:

- **EKS Cluster Role** — allows the control plane to manage AWS resources
- **EKS Node Group Role** — allows worker nodes to pull ECR images, join the cluster, and use the VPC CNI plugin

---

## ⚙️ Section 8: Kubernetes on EKS

The `kubernetes/` directory contains raw Kubernetes manifests. These are the foundation — applied directly with `kubectl` before moving to Helm for more complex deployments.

### Core Kubernetes Concepts Used

**Pods** — the smallest deployable unit; each microservice runs in one or more pods.

```yaml
# kubernetes/productcatalog/deployment.yaml (simplified)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: productcatalog
spec:
  replicas: 2
  selector:
    matchLabels:
      app: productcatalog
  template:
    metadata:
      labels:
        app: productcatalog
    spec:
      containers:
      - name: productcatalog
        image: <ecr-url>/productcatalog:latest
        ports:
        - containerPort: 3550
        resources:
          requests:
            cpu: "100m"
            memory: "64Mi"
          limits:
            cpu: "200m"
            memory: "128Mi"
        readinessProbe:
          grpc:
            port: 3550
          initialDelaySeconds: 10
        livenessProbe:
          grpc:
            port: 3550
          initialDelaySeconds: 10
```

**Services** — all 5 types are used in this project:

| Type | When Used in This Project |
|---|---|
| `ClusterIP` | Internal service-to-service communication (default for all microservices) |
| `NodePort` | Exposing services during development & testing |
| `LoadBalancer` | Provisioning an AWS ALB/NLB for external production access |
| `ExternalName` | Mapping to external DNS names |
| `Headless` | Kafka and Redis StatefulSets needing direct pod IP access |

```yaml
# kubernetes/productcatalog/svc.yaml
apiVersion: v1
kind: Service
metadata:
  name: productcatalog
spec:
  type: ClusterIP
  selector:
    app: productcatalog
  ports:
  - port: 3550
    targetPort: 3550
```

**ConfigMaps** — inject configuration into pods without rebuilding images:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: otel-config
data:
  OTEL_EXPORTER_OTLP_ENDPOINT: "http://otelcol:4317"
  PRODUCT_CATALOG_SERVICE_ADDR: "productcatalog:3550"
  CART_SERVICE_ADDR: "cartservice:7070"
```

**StatefulSets** — used for Kafka and Redis which need stable network identities and persistent storage.

**HorizontalPodAutoscaler (HPA)** — automatically scales pods based on CPU or memory:

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: productcatalog-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: productcatalog
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
```

### 8.6 Applying Manifests

```bash
# Apply all manifests
kubectl apply -f kubernetes/

# Watch rollout status
kubectl rollout status deployment/productcatalog

# Get all resources
kubectl get pods,svc,deployments

# Debug a pod
kubectl describe pod <pod-name>
kubectl logs <pod-name> -f
kubectl logs <pod-name> --previous     # logs from crashed container
kubectl exec -it <pod-name> -- /bin/sh
```

### Kubernetes Ingress & Load Balancing

The **AWS Load Balancer Controller** is installed on EKS to provision ALBs directly from Kubernetes Ingress resources.

**Install the AWS Load Balancer Controller:**
```bash
helm repo add eks https://aws.github.io/eks-charts
helm repo update

helm install aws-load-balancer-controller eks/aws-load-balancer-controller \
  -n kube-system \
  --set clusterName=ecommerce-devops-cluster \
  --set serviceAccount.create=false \
  --set serviceAccount.name=aws-load-balancer-controller
```

**Ingress manifest to expose the frontend:**
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ecommerce-ingress
  annotations:
    kubernetes.io/ingress.class: alb
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/target-type: ip
    alb.ingress.kubernetes.io/certificate-arn: <acm-cert-arn>
    alb.ingress.kubernetes.io/listen-ports: '[{"HTTP":80},{"HTTPS":443}]'
    alb.ingress.kubernetes.io/ssl-redirect: "443"
spec:
  rules:
  - host: shop.yourdomain.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: frontend-proxy
            port:
              number: 8080
```

---

## 🔐 Section 9: Secrets Management

### 9.1 Kubernetes Secrets Basics

Kubernetes Secrets store sensitive values as base64-encoded data:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-credentials
type: Opaque
data:
  username: YWRtaW4=          # base64 of "admin"
  password: c3VwZXJzZWNyZXQ=   # base64 of "supersecret"
```

> ⚠️ Kubernetes Secrets are **base64-encoded, not encrypted**. For production, always integrate with AWS Secrets Manager.

### 9.2 AWS Secrets Manager

AWS Secrets Manager stores secrets encrypted at rest and supports automatic rotation.

**Create a secret:**
```bash
aws secretsmanager create-secret \
  --name "/ecommerce/database" \
  --secret-string '{"username":"admin","password":"supersecret"}'
```

**Read a secret:**
```bash
aws secretsmanager get-secret-value \
  --secret-id "/ecommerce/database" \
  --query SecretString \
  --output text
```

### 9.3 EKS Pod Identity Agent

EKS Pod Identity lets pods assume IAM roles **without storing any AWS credentials** — the recommended production approach.

**Install the addon:**
```bash
aws eks create-addon \
  --cluster-name ecommerce-devops-cluster \
  --addon-name eks-pod-identity-agent
```

**Associate a role with a Kubernetes service account:**
```bash
aws eks create-pod-identity-association \
  --cluster-name ecommerce-devops-cluster \
  --namespace default \
  --service-account otel-collector-sa \
  --role-arn arn:aws:iam::<account-id>:role/otel-collector-role
```

Any pod using `otel-collector-sa` now automatically receives temporary AWS credentials — no static secrets stored anywhere in code or YAML.

---

## 🔁 Section 10: CI/CD Pipeline (GitHub Actions + ArgoCD)

### 10.1 GitHub Actions — Continuous Integration

The `.github/workflows/` directory contains the CI pipeline. On every push to `main`, it builds Docker images and pushes them to ECR.

```yaml
# .github/workflows/ci.yml
name: CI Pipeline

on:
  push:
    branches: [main]

jobs:
  build-and-push:
    runs-on: ubuntu-latest
    permissions:
      id-token: write     # Required for OIDC auth to AWS
      contents: read

    steps:
    - name: Checkout code
      uses: actions/checkout@v4

    - name: Configure AWS credentials via OIDC (no stored secrets)
      uses: aws-actions/configure-aws-credentials@v4
      with:
        role-to-assume: arn:aws:iam::<account-id>:role/github-actions-role
        aws-region: us-east-1

    - name: Login to ECR
      id: login-ecr
      uses: aws-actions/amazon-ecr-login@v2

    - name: Build & push image
      env:
        ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
        IMAGE_TAG: ${{ github.sha }}
      run: |
        docker build -t $ECR_REGISTRY/productcatalog:$IMAGE_TAG ./src/productcatalogservice/
        docker push $ECR_REGISTRY/productcatalog:$IMAGE_TAG

    - name: Update image tag in Kubernetes manifests
      env:
        IMAGE_TAG: ${{ github.sha }}
      run: |
        sed -i "s|image:.*productcatalog.*|image: $ECR_REGISTRY/productcatalog:$IMAGE_TAG|g" \
          kubernetes/productcatalog/deployment.yaml
        git config user.email "ci@github.com"
        git config user.name "GitHub Actions"
        git add kubernetes/
        git commit -m "ci: update image tag to $IMAGE_TAG"
        git push
```

**OIDC authentication** means no AWS credentials are ever stored as GitHub Secrets. GitHub requests a short-lived token from AWS via OpenID Connect — the most secure approach for any CI pipeline.

### 10.2 ArgoCD — Continuous Deployment (GitOps)

ArgoCD watches this repository and automatically applies any change it detects to the EKS cluster. **Git is the single source of truth for the cluster state.**

**Install ArgoCD:**
```bash
kubectl create namespace argocd

kubectl apply -n argocd -f \
  https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Wait for ArgoCD pods to be ready
kubectl wait --for=condition=available deployment -l app.kubernetes.io/name=argocd-server \
  -n argocd --timeout=120s

# Get the initial admin password
kubectl get secret argocd-initial-admin-secret -n argocd \
  -o jsonpath="{.data.password}" | base64 -d

# Access the UI
kubectl port-forward svc/argocd-server -n argocd 8080:443
# Open https://localhost:8080
```

**Create an ArgoCD Application pointing to this repo:**
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: ecommerce-app
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/Shyam-Kumar-Khatri/E-COMMERCE-DEVOPS-Implementation.git
    targetRevision: main
    path: kubernetes              # ArgoCD watches this exact folder
  destination:
    server: https://kubernetes.default.svc
    namespace: default
  syncPolicy:
    automated:
      prune: true       # Delete resources removed from Git
      selfHeal: true    # Revert any manual cluster changes automatically
    syncOptions:
    - CreateNamespace=true
```

```bash
kubectl apply -f argocd-application.yaml
```

### 10.3 End-to-End CI/CD Flow

```
Developer pushes code to main branch
        │
        ▼
GitHub Actions triggers:
  ✅ Checks out code
  ✅ Authenticates to AWS via OIDC (zero stored credentials)
  ✅ Builds Docker image
  ✅ Pushes image to ECR  ──  tagged with git commit SHA
  ✅ Updates image tag in kubernetes/ manifests
  ✅ Commits & pushes manifest change back to repo
        │
        ▼
ArgoCD detects new commit in repo
        │
        ▼
ArgoCD compares:  desired state (Git)  vs  actual state (EKS)
        │
        ▼
ArgoCD applies the diff
  → Rolling Deployment update on EKS
  → Zero downtime rollout
  → Any manual cluster drift is auto-reverted (self-healing)
        │
        ▼
New version is live ✅
```

---

## 📦 Deploying via Helm

This project uses the **official OpenTelemetry Demo Helm chart** — maintained by the OpenTelemetry community at [open-telemetry/opentelemetry-helm-charts](https://github.com/open-telemetry/opentelemetry-helm-charts). The chart is pulled directly from the official Helm registry; no custom Helm chart is stored in this repo.

**Add the chart and install:**
```bash
# Add the OpenTelemetry Helm repository
helm repo add open-telemetry https://open-telemetry.github.io/opentelemetry-helm-charts
helm repo update

# Create a dedicated namespace and install
kubectl create namespace otel-demo

helm install my-otel-demo open-telemetry/opentelemetry-demo \
  --namespace otel-demo

# Install with a custom values override file
helm install my-otel-demo open-telemetry/opentelemetry-demo \
  --namespace otel-demo \
  --values my-custom-values.yaml
```

**Access the app after install:**
```bash
kubectl port-forward svc/my-otel-demo-frontendproxy 8080:8080 -n otel-demo
# Open http://localhost:8080
```

**Useful Helm commands:**
```bash
helm list -n otel-demo                              # List all releases
helm status my-otel-demo -n otel-demo               # Check release status
helm upgrade my-otel-demo open-telemetry/opentelemetry-demo -n otel-demo
helm rollback my-otel-demo 1 -n otel-demo           # Roll back to previous revision
helm uninstall my-otel-demo -n otel-demo            # Uninstall the release
```

---

## 📊 Observability

Every microservice in `src/` is instrumented with the **OpenTelemetry SDK** for its language. All telemetry flows through the `otelcollector` component defined in `src/otelcollector/`.

### Three Pillars of Observability

| Signal | What It Shows | Backend |
|---|---|---|
| **Traces** | Full request journey across every service | Jaeger |
| **Metrics** | Latency, error rates, throughput over time | Prometheus + Grafana |
| **Logs** | Timestamped events from each service | Captured in OTel spans |

### Telemetry Data Flow

```
Each Microservice  (OTel SDK instrumented)
        │
        │  OTLP gRPC / HTTP
        ▼
OpenTelemetry Collector  (src/otelcollector/)
        │
        ├──────────────────────────────────► Jaeger
        │                                   (Distributed traces)
        │
        ├──────────────────────────────────► Prometheus
        │                                   (Metrics storage)
        │                                        │
        │                                        ▼
        │                                     Grafana
        │                                   (Dashboards)
        │
        └──────────────────────────────────► CloudWatch
                                            (Logs & metrics)
```

### Accessing Observability UIs Locally

```bash
# After docker compose up -d
open http://localhost:8080/jaeger/ui     # Distributed traces
open http://localhost:8080/grafana       # Metrics dashboards  (admin / admin)
open http://localhost:9090               # Raw Prometheus metrics
```

### Feature Flags for Simulating Failures

The `flagd-ui` service provides runtime feature flags to trigger intentional failures — use these to practice debugging distributed systems safely:

```bash
# Access the feature flag UI
open http://localhost:8080/feature
```

| Flag | Effect |
|---|---|
| `productCatalogFailure` | Product catalog returns errors |
| `paymentServiceFailure` | Random payment failures |
| `cartServiceFailure` | Cart failures on add/remove |

Toggle a flag, then open Jaeger and watch error traces populate in real time.

---

## 🚦 Getting Started

Follow these steps in order to go from zero to a fully deployed application:

```bash
# Step 1: Clone the repository
git clone https://github.com/Shyam-Kumar-Khatri/E-COMMERCE-DEVOPS-Implementation.git
cd E-COMMERCE-DEVOPS-Implementation

# Step 2: Run locally with Docker Compose
docker compose up -d
# Access: http://localhost:8080

# Step 3: Provision AWS infrastructure with Terraform
cd eks-install/
terraform init
terraform plan
terraform apply

# Step 4: Configure kubectl to connect to EKS
aws eks update-kubeconfig --region us-east-1 --name ecommerce-devops-cluster
kubectl get nodes

# Step 5a: Deploy with raw Kubernetes manifests
kubectl apply -f kubernetes/

# Step 5b: OR deploy with the official Helm chart
helm repo add open-telemetry https://open-telemetry.github.io/opentelemetry-helm-charts
helm install my-otel-demo open-telemetry/opentelemetry-demo \
  --namespace otel-demo --create-namespace

# Step 6: Set up ArgoCD for GitOps
kubectl create namespace argocd
kubectl apply -n argocd -f \
  https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Step 7: Verify everything is running
kubectl get pods --all-namespaces
kubectl get ingress

# Step 8: Access the deployed app
kubectl port-forward svc/my-otel-demo-frontendproxy 8080:8080 -n otel-demo
# Visit http://localhost:8080
```

---

## 🛠 Troubleshooting

**Pods stuck in `Pending` state:**
```bash
kubectl describe pod <pod-name>
# Insufficient CPU/Memory → scale up node group
# ImagePullBackOff      → check ECR permissions and image tag
```

**Pods in `CrashLoopBackOff`:**
```bash
kubectl logs <pod-name> --previous
kubectl describe pod <pod-name>
# Common causes: bad config, missing env vars, failing health check
```

**ArgoCD showing `OutOfSync`:**
```bash
# Force sync
argocd app sync ecommerce-app --force
# Check for resource conflicts
argocd app diff ecommerce-app
```

**Terraform state lock:**
```bash
# If terraform apply crashes mid-run
terraform force-unlock <lock-id>
```

**EKS nodes not joining cluster:**
```bash
# Verify node group IAM role has required policies
# Verify Security Group allows ports 443 and 10250
aws eks describe-nodegroup \
  --cluster-name ecommerce-devops-cluster \
  --nodegroup-name general
```

**Cannot access Ingress URL:**
```bash
kubectl describe ingress ecommerce-ingress
# Ensure AWS LBC controller is running
kubectl get pods -n kube-system | grep aws-load-balancer
# Check Security Group allows 80/443 from the internet
```

---

## ✅ Prerequisites Summary

Before starting, ensure you have all of the following:

- [ ] AWS Account with appropriate IAM permissions
- [ ] AWS CLI configured (`aws configure`)
- [ ] Git installed
- [ ] Docker and Docker Compose installed
- [ ] kubectl installed
- [ ] Terraform ≥ 1.5 installed
- [ ] Helm ≥ 3.0 installed
- [ ] A registered domain name (for SSL/TLS with Ingress)
- [ ] GitHub account (for CI/CD pipeline workflows)
- [ ] Basic familiarity with Linux command line

---

## 👨‍💻 Credits

**Application:** The Astronomy Shop is forked from [OpenTelemetry Demo](https://github.com/open-telemetry/opentelemetry-demo), originally donated from the Google Microservices Demo. Full credit to the OpenTelemetry Authors and all contributors.

**DevOps Implementation Reference:** [Abhishek Veeramalla](https://github.com/iam-veeramalla) — DevOps Engineer, Educator, and Open Source contributor.

---

<div align="center">

⭐ **If this project helped you, please give it a star!** ⭐

[View Repository](https://github.com/Shyam-Kumar-Khatri/E-COMMERCE-DEVOPS-Implementation)

</div>
