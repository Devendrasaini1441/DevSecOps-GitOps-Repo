# DevSecOps GitOps Platform on Kubernetes (kubeadm) – AWS EC2

## Project Description

This project demonstrates a complete DevSecOps + GitOps architecture built from scratch using a multi-node Kubernetes cluster (kubeadm) on AWS EC2.
Instead of using managed Kubernetes services like EKS, this setup intentionally uses kubeadm to gain deep understanding of:

1. Control plane mechanics
2. Cluster networking (Calico CNI)
3. Ingress routing
4. NodePort behavior
5. GitOps reconciliation
6. CI/CD automation
7. Declarative deployments

A dummy React application is used to simulate real-world application deployment and automation.
The system follows production-style practices including:

1. Immutable image tagging (commit SHA)
2. Automated image updates in GitOps repo
3. ArgoCD self-healing
4. Vulnerability scanning
5. Code quality analysis
6. Monitoring stack integration

## Architecture Overview

![DevSecOps-GitOps_page-0001](https://github.com/user-attachments/assets/eb9991af-13d3-426f-9082-8e3e2985b658)

Core Principles:

1. Git as Single Source of Truth
2. Immutable container versioning
3. Declarative deployments
4. Drift detection & self-healing
5. CI-driven automation

## 🛠️ Tech Stack

☁️ Cloud
1. AWS EC2 (Free Tier)
2. VPC

☸️ Kubernetes

1. kubeadm (Multi-node cluster)
2. Calico CNI
3. NGINX Ingress Controller

🔁 CI/CD

1. GitHub Actions
2. Docker
3. DockerHub

🔐 DevSecOps Tools

1. SonarQube (SAST / Code Quality)
2. Trivy (Container Vulnerability Scan)

🚀 GitOps

1. ArgoCD
2. Separate GitOps Repository

📊 Monitoring

1. Prometheus
2. Grafana (kube-prometheus-stack)

integration

## 📦 Repository Structure
### Application Repository:- https://github.com/Devendrasaini1441/DevSecOps-GitOps-App.git
```
DevSecOps-GitOps-App/
 ├── src/
 ├── Dockerfile
 ├── .github/workflows/ci.yml
```

### GitOps Repository:- https://github.com/Devendrasaini1441/DevSecOps-GitOps-App.git
```
DevSecOps-GitOps-Repo/
 ├── deployment.yaml
 ├── service.yaml
 ├── ingress.yaml
```

