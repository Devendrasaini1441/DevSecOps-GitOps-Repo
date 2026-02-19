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

Git as Single Source of Truth
Immutable container versioning
Declarative deployments
Drift detection & self-healing
CI-driven automation

