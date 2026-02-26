# ☸️ Kubernetes End-to-End Project on AWS EKS

> A hands-on DevOps project deploying a containerized application on AWS Elastic Kubernetes Service (EKS) — from cluster setup to live traffic routing.

---

## 📌 Project Overview

This project demonstrates a complete Kubernetes workflow on **AWS EKS**, covering everything from cluster architecture to exposing an application via an AWS Application Load Balancer (ALB). Built as part of real-world DevOps practice to bridge the gap between theoretical Kubernetes knowledge and cloud-based implementation.

---

## 🎯 What I Worked On

| Area | Details |
|------|---------|
| **EKS Cluster Architecture** | Understood node groups, control plane, and worker node communication |
| **Containerized App Deployment** | Deployed a Dockerized application onto the Kubernetes cluster |
| **Kubernetes Resources** | Created and managed `Deployment`, `Service`, and `Ingress` resources |
| **AWS ALB Ingress** | Exposed the application externally using the AWS Load Balancer Controller |
| **Networking & Traffic Flow** | Observed how traffic flows through EKS — from the internet to pods |

---

## 🏗️ Architecture

```
Internet
   │
   ▼
AWS ALB (Application Load Balancer)
   │
   ▼
Kubernetes Ingress
   │
   ▼
Kubernetes Service (ClusterIP / NodePort)
   │
   ▼
Kubernetes Pods (Deployment)
   │
   ▼
Docker Container (Application)
```

---

## 🛠️ Tech Stack

- **Cloud Provider:** AWS
- **Kubernetes:** Amazon EKS (Elastic Kubernetes Service)
- **Ingress Controller:** AWS Load Balancer Controller (ALB)
- **Containerization:** Docker
- **Kubernetes Resources:** Deployment, Service, Ingress
- **CLI Tools:** `kubectl`, `eksctl`, `aws cli`

---

## 📂 Key Kubernetes Resources

### Deployment
Manages the desired state of the application pods — ensuring the correct number of replicas are running at all times.

### Service
Provides a stable internal endpoint to route traffic to the appropriate pods, abstracting away pod IP changes.

### Ingress
Defines routing rules for external HTTP/HTTPS traffic and integrates with AWS ALB to expose the application publicly.

---

## 🚀 Getting Started

### Prerequisites

- AWS Account with appropriate IAM permissions
- `kubectl` installed
- `eksctl` installed
- `aws cli` configured

### Steps

1. **Create EKS Cluster**
   ```bash
   eksctl create cluster --name my-cluster --region us-east-1 --nodegroup-name standard-workers --node-type t3.medium --nodes 2
   ```

2. **Configure kubectl**
   ```bash
   aws eks update-kubeconfig --name my-cluster --region us-east-1
   ```

3. **Install AWS Load Balancer Controller**
   ```bash
   # Follow AWS documentation to install the ALB controller via Helm
   helm install aws-load-balancer-controller eks/aws-load-balancer-controller -n kube-system
   ```

4. **Deploy the Application**
   ```bash
   kubectl apply -f deployment.yaml
   kubectl apply -f service.yaml
   kubectl apply -f ingress.yaml
   ```

5. **Verify Resources**
   ```bash
   kubectl get pods
   kubectl get svc
   kubectl get ingress
   ```

---

## 📁 Repository

🔗 **GitHub:** [https://lnkd.in/daQxVPnuU](https://lnkd.in/daQxVPnuU)

---

## 📚 Key Learnings

- How AWS EKS manages the Kubernetes control plane on your behalf
- The relationship between Pods, Deployments, Services, and Ingress
- How the AWS ALB Ingress Controller provisions and manages load balancers automatically
- How traffic flows from the internet → ALB → Ingress → Service → Pods
- The importance of IAM roles, service accounts, and OIDC for EKS integrations

---

## 🌱 What's Next

- [ ] Add Horizontal Pod Autoscaler (HPA)
- [ ] Set up CI/CD pipeline with GitHub Actions
- [ ] Implement monitoring with Prometheus & Grafana
- [ ] Explore Helm charts for templated deployments
- [ ] Configure SSL/TLS with AWS Certificate Manager (ACM)

---

## 👤 Author

Still learning, still building — and focusing on doing real work. 🚀

---

## 📄 License

This project is open source and available under the [MIT License](LICENSE).
