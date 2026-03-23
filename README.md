
**Note:** This project is a fork of [opentelemetry-demo](https://github.com/open-telemetry/opentelemetry-demo/tree/main). Thanks to the team and contributors for opensourcing this wonderful demo project. Definitely one of the best on internet.

<!-- markdownlint-disable-next-line -->
# 🚀 Ultimate DevOps Project — End-to-End AWS Cloud Deployment

> **A production-grade, end-to-end DevOps implementation** of the OpenTelemetry Astronomy Shop — a polyglot microservices e-commerce application — deployed on AWS EKS using Docker, Terraform, Kubernetes, Helm, GitHub Actions, ArgoCD, and full OpenTelemetry observability.


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
- [Phase 1: AWS Setup & Prerequisites](#-phase-1-aws-setup--prerequisites)
  - [AWS Account & IAM Setup](#aws-account--iam-setup)
  - [EC2 Instance Setup](#ec2-instance-setup)
  - [Tool Installation](#tool-installation)
- [Phase 2: Local Development with Docker Compose](#-phase-2-local-development-with-docker-compose)
  - [Running the Project Locally](#running-the-project-locally)
  - [Containerizing the Services](#containerizing-the-services)
- [Phase 3: Infrastructure as Code with Terraform](#-phase-3-infrastructure-as-code-with-terraform)
  - [Terraform Concepts & Lifecycle](#terraform-concepts--lifecycle)
  - [Remote State with S3 & DynamoDB](#remote-state-with-s3--dynamodb)
  - [Provisioning VPC with Terraform](#provisioning-vpc-with-terraform)
  - [Provisioning EKS with Terraform](#provisioning-eks-with-terraform)
- [Phase 4: Kubernetes on AWS EKS](#-phase-4-kubernetes-on-aws-eks)
  - [EKS Cluster Overview](#eks-cluster-overview)
  - [Core Kubernetes Concepts Used](#core-kubernetes-concepts-used)
  - [Deploying the Application to EKS](#deploying-the-application-to-eks)
  - [Kubernetes Ingress & Load Balancing](#kubernetes-ingress--load-balancing)
  - [SSL/TLS & External DNS](#ssltls--external-dns)
- [Phase 5: Secrets Management](#-phase-5-secrets-management)
  - [AWS Secrets Manager Integration](#aws-secrets-manager-integration)
  - [EKS Pod Identity & IRSA](#eks-pod-identity--irsa)
- [Phase 6: AWS Managed Database Services](#-phase-6-aws-managed-database-services)
- [Phase 7: Helm Charts](#-phase-7-helm-charts)
  - [Why Helm?](#why-helm)
  - [Helm Chart Structure](#helm-chart-structure)
  - [Working with Helm](#working-with-helm)
- [Phase 8: CI/CD Pipeline](#-phase-8-cicd-pipeline)
  - [GitHub Actions — Continuous Integration](#github-actions--continuous-integration)
  - [ArgoCD — Continuous Deployment (GitOps)](#argocd--continuous-deployment-gitops)
  - [End-to-End CI/CD Flow](#end-to-end-cicd-flow)
- [Phase 9: Autoscaling with Karpenter](#-phase-9-autoscaling-with-karpenter)
  - [Why Karpenter?](#why-karpenter)
  - [Spot Instance Handling](#spot-instance-handling)
- [Phase 10: Observability](#-phase-10-observability)
  - [OpenTelemetry Overview](#opentelemetry-overview)
  - [Distributed Tracing with Jaeger / AWS X-Ray](#distributed-tracing-with-jaeger--aws-x-ray)
  - [Metrics with Prometheus & Grafana](#metrics-with-prometheus--grafana)
  - [Logs with CloudWatch / Fluent Bit](#logs-with-cloudwatch--fluent-bit)
- [Phase 11: Security](#-phase-11-security)
- [Project Directory Layout](#-project-directory-layout)
- [Getting Started](#-getting-started)
- [Cost Optimization Tips](#-cost-optimization-tips)
- [Troubleshooting](#-troubleshooting)
- [Prerequisites Summary](#-prerequisites-summary)
- [Author & Credits](#-author--credits)

---

## 🔭 Project Overview

This project is an **end-to-end DevOps implementation** built on top of the **OpenTelemetry Astronomy Shop** — a realistic, near-production-grade, microservice-based distributed system originally created by the OpenTelemetry community (forked from [open-telemetry/opentelemetry-demo](https://github.com/open-telemetry/opentelemetry-demo)).

> **What is the Astronomy Shop?**
> It simulates an online e-commerce store where users browse products (telescopes, lenses, accessories), add items to a shopping cart, and check out. Behind the scenes, over a dozen microservices — written in 10+ different programming languages — collaborate via gRPC and HTTP to handle every step of that flow.

On top of this real-world application, the project layers a **full DevOps pipeline** covering every major domain a modern cloud engineer needs to master:

| Domain | Tools Used |
|---|---|
| Containerization | Docker, Docker Compose, ECR |
| Infrastructure as Code | Terraform |
| Cloud Platform | AWS (EKS, VPC, RDS, ElastiCache, DynamoDB, SQS, S3, IAM) |
| Container Orchestration | Kubernetes (AWS EKS) |
| Package Management | Helm |
| CI Pipeline | GitHub Actions |
| CD / GitOps | ArgoCD |
| Autoscaling | Karpenter + Spot Instances |
| Observability | OpenTelemetry, Jaeger, Prometheus, Grafana, AWS X-Ray, CloudWatch |
| Secrets & Security | AWS Secrets Manager, RBAC, IAM Roles, Pod Identity |

---

## 📁 Repository Structure
