# Kubernetes End-to-End Deployment on AWS EKS

> Containerized application deployed on AWS Elastic Kubernetes Service with ALB-based external traffic routing, namespace isolation, and rolling update strategy.

---

## Overview

This project covers a full Kubernetes deployment lifecycle on AWS EKS — from cluster provisioning and IAM configuration to exposing a containerized application via an AWS Application Load Balancer. The focus was on understanding how Kubernetes primitives (Deployment, Service, Ingress) map to real AWS infrastructure, and working through the operational gaps that don't show up in tutorials.

---

## Tech Stack

| Component | Detail |
|-----------|--------|
| Cloud | AWS (EKS, ALB, IAM, EC2 Node Groups) |
| Kubernetes | Amazon EKS — managed control plane |
| Ingress Controller | AWS Load Balancer Controller (ALB) |
| Containerization | Docker |
| K8s Resources | Deployment, Service (ClusterIP), Ingress |
| CLI Tools | kubectl, eksctl, aws cli, helm |

---

## Architecture

Traffic flows from the internet through an AWS ALB, which the Load Balancer Controller provisions automatically based on Ingress annotations. The ALB forwards requests to the Kubernetes Service, which load-balances across healthy pods managed by the Deployment controller.

```
Internet → AWS ALB → Kubernetes Ingress → Service (ClusterIP) → Pods → Docker Container
```

### Key Design Decisions

- **ClusterIP over NodePort** — ALB controller routes directly to pod IPs via IP target mode, avoiding extra hops through node ports
- **Namespace isolation** — workloads separated by environment to scope RBAC and resource quotas
- **Rolling update strategy** — configured on the Deployment to avoid downtime during redeployment
- **Liveness and readiness probes** — defined per container so the ALB only routes to healthy pods

---

## Cluster Provisioning

### 1. Create EKS Cluster

Cluster created with `eksctl` using a managed node group of 2 x t3.medium workers in us-east-1:

```bash
eksctl create cluster \
  --name my-cluster \
  --region us-east-1 \
  --nodegroup-name standard-workers \
  --node-type t3.medium \
  --nodes 2
```

Configure local kubectl context:

```bash
aws eks update-kubeconfig --name my-cluster --region us-east-1
```

### 2. Install AWS Load Balancer Controller

The ALB controller requires an IAM OIDC provider and a service account with an IAM role before Helm installation:

```bash
# Associate OIDC provider with cluster
eksctl utils associate-iam-oidc-provider --cluster my-cluster --approve

# Create IAM service account
eksctl create iamserviceaccount \
  --cluster my-cluster \
  --namespace kube-system \
  --name aws-load-balancer-controller \
  --attach-policy-arn arn:aws:iam::<ACCOUNT_ID>:policy/AWSLoadBalancerControllerIAMPolicy \
  --approve

# Install via Helm
helm install aws-load-balancer-controller eks/aws-load-balancer-controller \
  -n kube-system \
  --set clusterName=my-cluster \
  --set serviceAccount.create=false \
  --set serviceAccount.name=aws-load-balancer-controller
```

---

## Application Deployment

### Deployment

Manages desired pod state with 2 replicas, rolling update strategy, and health probes:

```bash
kubectl apply -f deployment.yaml
```

Key config:

- `replicas: 2` — two pods for redundancy
- `strategy: RollingUpdate` — `maxUnavailable: 0`, `maxSurge: 1` ensures zero-downtime deploys
- `readinessProbe` — HTTP GET on `/health`, 3s delay, 5s period; pod only receives traffic when ready
- `livenessProbe` — same endpoint, 10s initial delay; restarts container if it becomes unresponsive

### Service

ClusterIP Service provides a stable internal endpoint. The ALB controller uses pod IPs directly (`ip` target type), so ClusterIP is sufficient — no NodePort needed:

```bash
kubectl apply -f service.yaml
```

### Ingress

Ingress resource with ALB annotations triggers the controller to provision an internet-facing ALB in the correct subnets:

```bash
kubectl apply -f ingress.yaml
```

Critical annotations:

- `kubernetes.io/ingress.class: alb` — tells the controller to handle this resource
- `alb.ingress.kubernetes.io/scheme: internet-facing` — provisions a public ALB
- `alb.ingress.kubernetes.io/target-type: ip` — routes to pod IPs, not node ports

---

## Verifying the Deployment

```bash
# Check pods are Running and Ready
kubectl get pods -n <namespace>

# Confirm service endpoint
kubectl get svc -n <namespace>

# Get ALB DNS name from Ingress
kubectl get ingress -n <namespace>

# Describe ingress for event log and ALB provisioning status
kubectl describe ingress <ingress-name> -n <namespace>
```

---

## Issues Encountered & Fixed

| Issue | Root Cause & Fix |
|-------|-----------------|
| ALB not provisioning | Controller pod was not running — OIDC association was missing. Fixed by re-running `eksctl utils associate-iam-oidc-provider` before Helm install. |
| Pods stuck in Pending | Node group didn't have enough capacity for the requested CPU. Lowered resource requests in Deployment spec. |
| Readiness probe failing | App was taking ~8s to start but probe delay was set to 3s. Increased `initialDelaySeconds` to 10. |
| Ingress address blank | Subnets weren't tagged with `kubernetes.io/role/elb=1`. Added tags and ALB provisioned within 2 minutes. |

---

## Planned Improvements

- [ ] Add Horizontal Pod Autoscaler (HPA) based on CPU utilization
- [ ] Configure SSL/TLS termination at the ALB using AWS Certificate Manager
- [ ] Set up Prometheus + Grafana for pod-level metrics and alerting
- [ ] Replace manual `kubectl apply` with a GitHub Actions CI/CD pipeline
- [ ] Explore Helm chart templating to parameterize environment-specific values

---

## Author

**Irfan Ali** · [github.com/irfanjat](https://github.com/irfanjat) · [linkedin.com/in/irfanjat](https://linkedin.com/in/irfanjat)
