# DevSecOps GitOps Platform on Kubernetes (kubeadm) – AWS EC2

## Project Description

This project demonstrates a complete DevSecOps + GitOps architecture built from scratch using a multi-node Kubernetes cluster (kubeadm) on AWS EC2.

Instead of using managed Kubernetes services like EKS, this setup intentionally uses kubeadm to gain deep understanding of:

Control plane mechanics

Cluster networking (Calico CNI)

Ingress routing

NodePort behavior

GitOps reconciliation

CI/CD automation

Declarative deployments

A dummy React application is used to simulate real-world application deployment and automation.

The system follows production-style practices including:

Immutable image tagging (commit SHA)

Automated image updates in GitOps repo

ArgoCD self-healing

Vulnerability scanning

Code quality analysis

Monitoring stack integration
