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
## 🧱 Detailed Implementation Guide

### PHASE 1 – Infrastructure Setup (AWS)
Launch EC2 instances:

<img width="940" height="416" alt="image" src="https://github.com/user-attachments/assets/e797d59b-6d8c-4805-a9bf-5cff5b53ddee" />

1. 1 Control Plane (t3.medium)
2. 2 Worker Nodes (t3.small)
3. Storage must bhi 25GB to each node 

Configure Security Groups:
1. 22 (SSH)
2. 6443 (Kubernetes API)
3. 30000-32767 (NodePort)
4. 80, 443 (Ingress)

Add the "Self-Referencing" Rule
1.	Go to the EC2 Dashboard > Security Groups.
2.	Select the Security Group that is attached to all your nodes.
3.	Click on the Inbound rules tab and click Edit inbound rules.
4.	Click Add rule and set it up as follows:
o	Type: All traffic
o	Protocol: All
o	Port range: 0 - 65535
o	Source: Click the search box and select the ID of the current Security Group (e.g., search for sg- and pick the one you are currently editing).
5.	Click Save rules.

<img width="940" height="419" alt="image" src="https://github.com/user-attachments/assets/11d6f495-ab25-43d5-ab52-e0570369f460" />


Verify Internet Gateway and route table configuration.

<img width="940" height="419" alt="image" src="https://github.com/user-attachments/assets/54d356af-f372-431f-abc2-80d6c4d50d5f" />

#### Steps to Attach an Elastic IP via AWS Console
1. Allocate the IP:

Log in to the AWS Management Console.

Navigate to EC2 > Network & Security > Elastic IPs.

Click Allocate Elastic IP address.

Choose the Amazon pool and click Allocate.

2. Associate the IP with the Node:

Select the newly created Elastic IP from the list.

Click Actions > Associate Elastic IP address.

Resource type: Select "Instance".

Instance: Search for and select your master-node (Control Plane).

Private IP address: Select the internal IP of the master node.

Click Associate.

3. Update Security Groups:

Ensure the Security Group attached to your Control Plane allows traffic on port 6443 (Kubernetes API) and your NodePorts (like the one for Argo CD or SonarQube) from your specific Elastic IP or your local machine's IP.
<img width="940" height="418" alt="image" src="https://github.com/user-attachments/assets/2b51200a-7d4f-4c13-a5e6-b59ec0d31dd7" />

### PHASE 2 – Kubernetes Cluster Setup (kubeadm)

#### PART 1 – COMMON SETUP (Run on ALL 3 Nodes)
SSH into each node one by one

##### Step 1 – Update System
```
sudo apt update && sudo apt upgrade -y
```
##### Step 2 – Disable Swap (Mandatory)
Kubernetes will fail if swap is enabled.
```
sudo swapoff -a
```
Make it permanent:
```
sudo sed -i '/ swap / s/^/#/' /etc/fstab
```
Verify:
```
free -h
```
Swap should show 0.

##### Step 3 – Install containerd
```
sudo apt install -y containerd
```

Create config:
```
sudo mkdir -p /etc/containerd
sudo containerd config default | sudo tee /etc/containerd/config.toml
```

Edit file:
```
sudo nano /etc/containerd/config.toml
```
Find: Search= ctrl+W, in feild serch for ```runc.options``` 
```
SystemdCgroup = false
```
Change to:
```
SystemdCgroup = true
```
Save and exit.

Restart containerd:
```
sudo systemctl restart containerd
sudo systemctl enable containerd
```
##### Step 4 – Enable Required Kernel Modules
```
sudo modprobe overlay
sudo modprobe br_netfilter
```
Make permanent:
```
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF
```
##### Step 5 – Enable Networking Settings
```
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF
```
Apply:
```
sudo sysctl --system
```
##### Step 6 – Install Kubernetes Components
Add Kubernetes repo:
```
sudo apt install -y apt-transport-https ca-certificates curl
sudo curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.29/deb/Release.key | sudo gpg --dearmor -o /usr/share/keyrings/kubernetes-archive-keyring.gpg
echo 'deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.29/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
```

Update:
```
sudo apt update
```

Install:
```
sudo apt install -y kubelet kubeadm kubectl
```
Hold versions:
```
sudo apt-mark hold kubelet kubeadm kubectl
```

#### PART 2 – CONTROL PLANE SETUP
Now SSH into control plane only.

##### Step 7 – Initialize Cluster
Run:
```
sudo kubeadm init --node-name my-master-node --pod-network-cidr=10.244.0.0/16
```
You see something like below: Copy this on notepade we use it later.
```
⚠ This will take 2–3 minutes.
After success, you will see:
kubeadm join <IP> --token <token> ...
⚠ Copy that join command somewhere safe.

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join <control-plane-ip>:6443 --token <token> \
        --discovery-token-ca-cert-hash sha256:<hash>
```
##### Step 8 – Configure kubectl Access
Run:
```
mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
Test:
```
kubectl get nodes
```
You should see control-plane in NotReady state. 

That’s normal.

#### PART 3 – Install CNI (Calico)
On control plane only:
```
kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.26.1/manifests/calico.yaml
```
Wait 1–2 minutes.

Check:
```
kubectl get pods -n kube-system
```
All calico pods should be Running.

Now check nodes:
```
kubectl get nodes
```
Control plane should become:

Ready

#### PART 4 – Join Worker Nodes
SSH into Worker 1.

Paste the join command copied earlier:

Example:
```
sudo kubeadm join <control-plane-ip>:6443 --token <token> \
    --discovery-token-ca-cert-hash sha256:<hash> \
    --node-name worker-01
```
Do same on Worker 2.

#### PART 5 – Verify Cluster
Go back to control plane.

Run:
```
kubectl get nodes
```
You should see:
```
control-plane   Ready
worker-01        Ready
worker-02        Ready
```
#### PART 6 – Remove Control Plane Taint (If you want to schedule pods on your control plane-Do NOT remove taint in Production)
If you want to allow pods on control plane:
```
kubectl taint nodes --all node-role.kubernetes.io/control-plane-
```
(Production: Do NOT remove taint)

### PHASE 4 – Install Monitoring Stack

#### STEP 1 – Install Helm (On Control Plane Only)
Download Helm:
```
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```
Verify:
```
helm version
```
You should see Helm version output.

#### STEP 2 – Add Prometheus Community Repo
```
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
```
Update repo:
```
helm repo update
```
#### STEP 3 – Create Monitoring Namespace
Better practice (production style):
```
kubectl create namespace monitoring
```
#### STEP 4 – Install kube-prometheus-stack
Now install:
```
helm install monitoring prometheus-community/kube-prometheus-stack -n monitoring
```
Wait 2–3 minutes.

#### STEP 5 – Verify Installation
Check pods:
```
kubectl get pods -n monitoring
```
You should see:
```
•	monitoring-kube-prometheus-prometheus
•	monitoring-grafana
•	monitoring-alertmanager
•	monitoring-kube-state-metrics
•	node-exporter pods
```
All should eventually be:

Running

#### STEP 6 – Check Services
```
kubectl get svc -n monitoring
```
You will see:
```
monitoring-grafana
monitoring-kube-prometheus-prometheus
```

#### STEP 7 - Change the Service to NodePort
```
kubectl patch svc monitoring-grafana -n monitoring -p '{"spec": {"type": "NodePort"}}'
```
Find the assigned port
```
kubectl get svc -n monitoring monitoring-grafana
```
Access UI:
```
http://<MASTER_ELASTIC_IP>:<Port>
```
#### STEP 8 - Login in Grafana
```
Username: admin
```
Password
```
kubectl get secret -n monitoring monitoring-grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo
```
<img width="940" height="455" alt="image" src="https://github.com/user-attachments/assets/a5f45c27-b6bc-47cb-a627-2a41e809cac6" />

Verify Metrics

Inside Grafana:

Go to:

Dashboards → Browse

You should see:

•	Kubernetes / Compute Resources
•	Node Exporter dashboards
•	Cluster monitoring dashboards

#### STEP 9 - Change Prometheus to NodePort
Run this command on your master-node to permanently switch the service type:
```
kubectl patch svc monitoring-kube-prometheus-prometheus -n monitoring -p '{"spec": {"type": "NodePort"}}'
```
Identify the Permanent Port

Now, check your services again to find the newly assigned port:
```
kubectl get svc -n monitoring monitoring-kube-prometheus-prometheus
```
Access the Prometheus Dashboard

Open your browser and enter the following URL:
```
http://<control-plane-ip>:<port>
```
Prometheus Dashboard:

<img width="940" height="472" alt="image" src="https://github.com/user-attachments/assets/50a7d2ed-9972-4df9-a0ba-531b64297e47" />

#### STEP 10 - Add Promethesu Data Source in Grafana

<img width="940" height="455" alt="image" src="https://github.com/user-attachments/assets/a842591e-df57-4211-b722-cb35c21fd902" />

Add Data Source Prometheus:

<img width="940" height="472" alt="image" src="https://github.com/user-attachments/assets/80ea3c5e-17cb-4f40-a62f-7c7a2f1b5e30" />

In Connection Paste Prometheus URL:

<img width="940" height="485" alt="image" src="https://github.com/user-attachments/assets/73b4a337-0ea9-424e-8645-2b81a3eea53e" />

Save and Test

<img width="940" height="479" alt="image" src="https://github.com/user-attachments/assets/bf94bf7a-988c-4634-ad77-8cb20efe1a37" />





































