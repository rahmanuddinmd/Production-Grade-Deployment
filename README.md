<div align="center">

# 🚀 ShopEasy — Production-Style CI/CD & GitOps Deployment on AWS EKS

## Visual Architecture • Step-by-Step Study Guide • Complete Commands

[![Python](https://img.shields.io/badge/Python-Flask-3776AB?logo=python&logoColor=white)](https://www.python.org/)
[![GitHub](https://img.shields.io/badge/GitHub-Source%20Control-181717?logo=github&logoColor=white)](https://github.com/)
[![Jenkins](https://img.shields.io/badge/Jenkins-Multibranch%20CI-D24939?logo=jenkins&logoColor=white)](https://www.jenkins.io/)
[![Docker](https://img.shields.io/badge/Docker-Containerization-2496ED?logo=docker&logoColor=white)](https://www.docker.com/)
[![Kubernetes](https://img.shields.io/badge/Kubernetes-AWS%20EKS-326CE5?logo=kubernetes&logoColor=white)](https://kubernetes.io/)
[![AWS](https://img.shields.io/badge/AWS-EKS%20%7C%20EC2%20%7C%20S3-232F3E?logo=amazonwebservices&logoColor=white)](https://aws.amazon.com/)
[![Argo CD](https://img.shields.io/badge/Argo%20CD-GitOps-EF7B4D?logo=argo&logoColor=white)](https://argo-cd.readthedocs.io/)
[![Prometheus](https://img.shields.io/badge/Prometheus-Monitoring-E6522C?logo=prometheus&logoColor=white)](https://prometheus.io/)
[![Grafana](https://img.shields.io/badge/Grafana-Observability-F46800?logo=grafana&logoColor=white)](https://grafana.com/)

### **Code → Build → Containerize → Publish → GitOps → Deploy → Monitor**

</div>

---

# 🖼️ Project Visual Overview

![ShopEasy Complete DevOps Visual Overview](docs/images/00-project-visual-overview.png)

The visual above provides a quick summary of the project.  
The rest of this README breaks that summary into **separate architecture diagrams**, explains each diagram, and then gives the exact commands/code used to build the environment.

---

# 📌 Project Objective

This project demonstrates an end-to-end DevOps delivery workflow for a Python Flask e-commerce application named **ShopEasy**.

The application is used as the workload, while the real learning focus is the DevOps platform surrounding it:

```text
Application Development
        ↓
Git / GitHub
        ↓
Jenkins Multibranch CI
        ↓
Docker Image Build
        ↓
Container Registry
        ↓
Kubernetes Manifest Update
        ↓
Git Commit
        ↓
Argo CD GitOps
        ↓
Amazon EKS
        ↓
Kubernetes LoadBalancer
        ↓
ShopEasy Application
        ↓
Prometheus + Grafana
```

This README is written as a **study document**, not just a project description.

Every major section follows:

```text
DIAGRAM
   ↓
WHAT IT MEANS
   ↓
HOW IT WORKS
   ↓
COMMANDS / CODE
   ↓
HOW TO VERIFY IT
```

---

# 📚 Table of Contents

1. [High-Level Architecture](#1️⃣-high-level-architecture)
2. [End-to-End CI/CD Pipeline](#2️⃣-end-to-end-cicd-pipeline)
3. [Jenkins Multibranch Pipeline](#3️⃣-jenkins-multibranch-pipeline)
4. [Docker Image Lifecycle](#4️⃣-docker-image-lifecycle)
5. [Argo CD GitOps Flow](#5️⃣-argo-cd-gitops-flow)
6. [Kubernetes Runtime Architecture](#6️⃣-kubernetes-runtime-architecture)
7. [Monitoring Architecture](#7️⃣-monitoring-architecture)
8. [Git Branching Strategy](#8️⃣-git-branching-strategy)
9. [Deployment Sequence](#9️⃣-deployment-sequence-and-rolling-update)
10. [Production Hardening](#🔟-production-hardening-roadmap)
11. [AWS Cloud Architecture: EC2, EKS, LoadBalancer and S3](#1️⃣1️⃣-aws-cloud-architecture)
12. [Step-by-Step Implementation Flow](#1️⃣2️⃣-step-by-step-implementation-flow)
13. [Three Critical Data Paths](#1️⃣3️⃣-three-critical-data-paths)
14. [Repository Structure](#-recommended-repository-structure)
15. [Prerequisites](#-prerequisites)
16. [Complete Installation Guide](#-complete-installation-and-deployment-guide)
17. [Dockerfile](#-dockerfile)
18. [Jenkinsfile](#-jenkinsfile)
19. [Kubernetes YAML](#-kubernetes-manifests)
20. [Argo CD YAML](#-argo-cd-application)
21. [Prometheus and Grafana](#-prometheus-and-grafana)
22. [Validation Checklist](#-full-validation-checklist)
23. [Known Project Corrections](#-important-corrections-found-in-the-original-project)
24. [Troubleshooting](#-troubleshooting)
25. [Cleanup](#-cleanup)
26. [Skills Demonstrated](#-skills-demonstrated)

---

# 1️⃣ High-Level Architecture

![High-Level DevOps Architecture](docs/images/01-high-level-architecture.png)

## What this diagram represents

This is the **main architecture** of the project.

At the highest level, six systems work together:

```text
Developer
GitHub
Jenkins
Container Registry
Argo CD
AWS EKS
```

The application does not jump directly from GitHub to Kubernetes.

Instead, it moves through a controlled delivery process.

## Flow

```text
Developer
   │
   │ git push
   ▼
GitHub
   │
   │ branch discovered
   ▼
Jenkins
   │
   ├── docker build
   ├── docker push
   └── update deployment.yaml
          │
          ▼
      Git Repository
          │
          ▼
       Argo CD
          │
          ▼
       AWS EKS
          │
          ▼
   LoadBalancer Service
          │
          ▼
       ShopEasy
```

## Responsibilities

| Component | Responsibility |
|---|---|
| Developer | Creates feature changes |
| GitHub | Stores source and deployment configuration |
| Jenkins | Builds and publishes application versions |
| Docker Registry | Stores versioned images |
| Argo CD | Watches Git and synchronizes Kubernetes |
| EKS | Runs Kubernetes workloads |
| LoadBalancer | Exposes application externally |
| Prometheus | Collects metrics |
| Grafana | Visualizes metrics |

## Why this architecture matters

The project separates two responsibilities:

### Continuous Integration

Handled by:

```text
Jenkins
```

Jenkins answers:

```text
Can the source be built?
What Docker image should represent this release?
Has the image been pushed?
What deployment version should Git contain?
```

### Continuous Delivery

Handled by:

```text
Argo CD
```

Argo CD answers:

```text
Does the EKS cluster match Git?
Should Kubernetes be synchronized?
Has configuration drift occurred?
```

---

# 2️⃣ End-to-End CI/CD Pipeline

![End-to-End CI/CD Flow](docs/images/02-end-to-end-cicd-flow.png)

## CI/CD lifecycle

The complete release lifecycle is:

```text
1. Developer changes code
2. Feature branch
3. Push to GitHub
4. Pull Request
5. Merge to main
6. Jenkins starts
7. Docker image is built
8. Docker image is pushed
9. Kubernetes image tag is updated
10. Manifest change is committed
11. Argo CD notices Git change
12. EKS is synchronized
13. Kubernetes performs rollout
14. LoadBalancer serves the new version
15. Prometheus observes cluster
16. Grafana displays metrics
```

---

## Stage 1 — Application development

Example:

```bash
git checkout -b feature/wishlist
```

A developer edits:

```text
app.py
```

and possibly:

```text
requirements.txt
templates/
static/
```

---

## Stage 2 — Local validation

Before pushing:

```bash
python app.py
```

Test:

```bash
curl http://localhost:5000
```

---

## Stage 3 — Commit

```bash
git add .

git commit \
  -m "feat: add wishlist functionality"
```

---

## Stage 4 — Push feature branch

```bash
git push \
  -u origin feature/wishlist
```

---

## Stage 5 — Pull Request and merge

Conceptually:

```text
feature/wishlist
       │
       ▼
Pull Request
       │
       ▼
Code Review
       │
       ▼
main
```

---

## Stage 6 — Jenkins build

Jenkins generates:

```text
IMAGE_TAG=build-${BUILD_NUMBER}
```

Example:

```text
build-105
```

---

## Stage 7 — Docker build

```bash
docker build \
  -t yourdockeruser/shopeasy-app:build-105 \
  .
```

---

## Stage 8 — Docker push

```bash
docker push \
  yourdockeruser/shopeasy-app:build-105
```

---

## Stage 9 — Kubernetes manifest update

Before:

```yaml
image: yourdockeruser/shopeasy-app:build-104
```

After:

```yaml
image: yourdockeruser/shopeasy-app:build-105
```

---

## Stage 10 — GitOps commit

```bash
git add k8s/deployment.yaml

git commit \
  -m "chore(deploy): update image to build-105"

git push origin main
```

---

## Stage 11 — Argo CD detects new Git revision

Argo CD is watching:

```text
repoURL
targetRevision
path
```

For example:

```yaml
repoURL: https://github.com/<USER>/<REPO>.git
targetRevision: main
path: k8s
```

---

## Stage 12 — EKS deployment

Argo CD synchronizes:

```text
Git Desired State
       ↓
AWS EKS Actual State
```

---

# 3️⃣ Jenkins Multibranch Pipeline

![Jenkins Multibranch Pipeline](docs/images/03-jenkins-pipeline.png)

## Why Multibranch Pipeline is used

The repository may contain:

```text
main
feature/wishlist
feature/reviews
feature/login
```

A Jenkins Multibranch job can discover each branch automatically.

### Branch behavior

```text
feature branches
      ↓
checkout / validation

main
      ↓
checkout
      ↓
build Docker image
      ↓
push Docker image
      ↓
update Kubernetes YAML
      ↓
commit deployment change
```

---

## Jenkins Stage 1 — Checkout

```groovy
stage('Checkout') {

    steps {

        checkout scm
    }
}
```

Meaning:

> Fetch the source code from the branch Jenkins is currently building.

---

## Jenkins Stage 2 — Main branch condition

```groovy
when {

    branch 'main'
}
```

This prevents a normal feature branch from publishing a production image or modifying the deployment state.

---

## Jenkins Stage 3 — Image version

```groovy
env.IMAGE_TAG = "build-${BUILD_NUMBER}"
```

Example:

```text
BUILD_NUMBER = 105
IMAGE_TAG     = build-105
```

---

## Jenkins Stage 4 — Docker build

```bash
docker build \
  -t ${IMAGE_NAME}:${IMAGE_TAG} \
  .
```

---

## Jenkins Stage 5 — Login to registry

```bash
echo "$DOCKER_PASS" \
  | docker login \
      -u "$DOCKER_USER" \
      --password-stdin
```

Credentials come from Jenkins Credentials Store.

---

## Jenkins Stage 6 — Push

```bash
docker push \
  ${IMAGE_NAME}:${IMAGE_TAG}
```

---

## Jenkins Stage 7 — Manifest change

```bash
sed -i \
  "s|image:.*|image: ${IMAGE_NAME}:${IMAGE_TAG}|" \
  k8s/deployment.yaml
```

---

## Jenkins Stage 8 — Git push

```bash
git add k8s/deployment.yaml

git commit \
  -m "chore(deploy): update image to ${IMAGE_TAG}"

git push
```

This is where CI hands control to GitOps.

---

# 4️⃣ Docker Image Lifecycle

![Docker Image Lifecycle](docs/images/04-docker-image-lifecycle.png)

## Source-to-image process

```text
app.py
requirements.txt
Dockerfile
      │
      ▼
docker build
      │
      ▼
Docker Image
      │
      ▼
build-N tag
      │
      ▼
Container Registry
      │
      ▼
EKS pulls image
```

---

## Dockerfile

```dockerfile
FROM python:3.11-slim

WORKDIR /app

COPY requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt

COPY app.py .

EXPOSE 5000

CMD ["python", "app.py"]
```

---

## Line-by-line explanation

### Base runtime

```dockerfile
FROM python:3.11-slim
```

Provides Python 3.11 using a smaller Linux base image.

### Working directory

```dockerfile
WORKDIR /app
```

Sets `/app` as the working directory inside the container.

### Dependencies

```dockerfile
COPY requirements.txt .

RUN pip install \
  --no-cache-dir \
  -r requirements.txt
```

### Application source

```dockerfile
COPY app.py .
```

### Port

```dockerfile
EXPOSE 5000
```

### Container startup

```dockerfile
CMD ["python", "app.py"]
```

---

## Local build

```bash
docker build \
  -t shopeasy-app:local \
  .
```

---

## Run

```bash
docker run \
  -d \
  --name shopeasy \
  -p 5000:5000 \
  shopeasy-app:local
```

---

## Verification

```bash
docker ps
```

```bash
docker logs shopeasy
```

```bash
curl \
  http://localhost:5000
```

---

# 5️⃣ Argo CD GitOps Flow

![Argo CD GitOps Architecture](docs/images/05-argocd-gitops.png)

## Core GitOps idea

Instead of:

```text
Jenkins
   ↓
kubectl apply
   ↓
EKS
```

this architecture uses:

```text
Jenkins
   ↓
Git
   ↓
Argo CD
   ↓
EKS
```

That means Git becomes the deployment source of truth.

---

## Desired state example

```yaml
replicas: 5

image: yourdockeruser/shopeasy-app:build-105
```

---

## Actual state example

EKS may currently contain:

```text
5 Pods
build-104
```

Argo CD sees a mismatch:

```text
Git    = build-105
Cluster = build-104
```

It synchronizes the cluster.

---

## Argo CD policy

```yaml
syncPolicy:

  automated:

    prune: true

    selfHeal: true
```

### Automated

Git updates can automatically deploy.

### Prune

Objects removed from Git may be removed from Kubernetes.

### Self-heal

Manual drift in Kubernetes can be returned to the state defined in Git.

---

# 6️⃣ Kubernetes Runtime Architecture

![Kubernetes Runtime Architecture](docs/images/06-kubernetes-runtime.png)

## Runtime objects

The project contains:

```text
Deployment
ReplicaSet
Pods
Service
```

---

## Deployment

A Deployment manages application replicas.

```yaml
replicas: 5
```

means:

```text
Desired Pods = 5
```

If a Pod terminates:

```text
Desired = 5
Actual  = 4
```

Kubernetes attempts to restore:

```text
Actual = 5
```

---

## Service

The Service is:

```yaml
type: LoadBalancer
```

and maps:

```text
port:       80
targetPort: 5000
```

Traffic path:

```text
User
 ↓
AWS LoadBalancer
 ↓
Kubernetes Service :80
 ↓
Flask Pod :5000
```

---

## Labels

Pod label:

```yaml
app: shopeasy-app
```

Service selector:

```yaml
app: shopeasy-app
```

These values must match.

---

# 7️⃣ Monitoring Architecture

![Monitoring and Observability Architecture](docs/images/07-monitoring-observability.png)

## Monitoring stack

The project uses:

```text
kube-prometheus-stack
```

to provide:

```text
Prometheus
Grafana
Alertmanager
Node Exporter
kube-state-metrics
```

---

## Node Exporter

Collects node-level metrics:

```text
CPU
Memory
Disk
Filesystem
Network
```

---

## kube-state-metrics

Exposes Kubernetes object-state metrics:

```text
Pods
Deployments
ReplicaSets
Services
Namespaces
```

---

## Prometheus

Prometheus:

```text
Scrapes metrics
Stores time-series data
Evaluates alert rules
Provides PromQL
```

---

## Grafana

Grafana converts metrics into dashboards.

Recommended dashboards:

```text
Kubernetes / Cluster
Kubernetes / Nodes
Kubernetes / Pods
Kubernetes / Deployments
Node Exporter
```

---

# 8️⃣ Git Branching Strategy

![Git Branching Strategy](docs/images/08-git-branching-strategy.png)

## Recommended branches

```text
main
│
├── feature/wishlist
├── feature/reviews
└── feature/login
```

---

## Example branch workflow

Create:

```bash
git checkout \
  -b feature/reviews
```

Modify code.

Check:

```bash
git status
```

Stage:

```bash
git add .
```

Commit:

```bash
git commit \
  -m "feat: add customer review functionality"
```

Push:

```bash
git push \
  -u origin feature/reviews
```

Then:

```text
feature/reviews
      ↓
Pull Request
      ↓
Review
      ↓
Merge
      ↓
main
```

---

# 9️⃣ Deployment Sequence and Rolling Update

![Deployment Sequence Diagram](docs/images/09-deployment-sequence.png)

## Deployment sequence

When Jenkins changes:

```text
build-104
```

to:

```text
build-105
```

the following occurs:

```text
Git Commit
   ↓
Argo CD
   ↓
Kubernetes API
   ↓
Deployment
   ↓
New ReplicaSet
   ↓
New Pods
   ↓
Readiness
   ↓
Old Pods removed
```

---

## Watch rollout

```bash
kubectl rollout status \
  deployment/shopeasy-app
```

---

## View ReplicaSets

```bash
kubectl get rs
```

---

## View image

```bash
kubectl get deployment shopeasy-app \
  -o jsonpath='{.spec.template.spec.containers[0].image}'

echo
```

---

## Rollout history

```bash
kubectl rollout history \
  deployment/shopeasy-app
```

---

# 🔟 Production Hardening Roadmap

![Production Hardening Roadmap](docs/images/10-production-hardening.png)

The project demonstrates a production-style workflow, but additional controls are required for a hardened production environment.

## Recommended additions

### CI quality

```text
Lint
Unit Tests
Integration Tests
Static Analysis
```

### Container security

```text
Trivy
SBOM
Signed Images
Pinned Dependencies
```

### Kubernetes reliability

```text
Readiness Probe
Liveness Probe
Resource Requests
Resource Limits
Horizontal Pod Autoscaler
Pod Disruption Budget
```

### Platform security

```text
RBAC
Secrets Manager
Private networking
TLS
Ingress
Least privilege IAM
```

### Operations

```text
Central logging
Alerts
SLOs
Automated rollback
Canary / Blue-Green deployment
```

---

# 1️⃣1️⃣ AWS Cloud Architecture

![AWS Cloud Deployment Architecture](docs/images/11-aws-cloud-deployment-architecture.png)

This is the AWS-focused view that shows where:

```text
EC2
EKS
LoadBalancer
S3
```

fit into the design.

---

## Jenkins on EC2

A common lab design is:

```text
EC2 Ubuntu Instance
        ↓
Java
        ↓
Jenkins
        ↓
Docker
```

Jenkins performs:

```text
Git checkout
Docker build
Docker push
Manifest update
Git push
```

---

## Amazon EKS

Amazon EKS provides the managed Kubernetes cluster.

It contains:

```text
Argo CD
Application Deployment
ReplicaSets
Pods
Services
Monitoring components
```

---

## AWS LoadBalancer

A Kubernetes Service:

```yaml
type: LoadBalancer
```

requests a cloud load balancer.

The external path becomes:

```text
Internet
   ↓
AWS LoadBalancer
   ↓
Kubernetes Service
   ↓
Application Pods
```

---

## Amazon S3 — important clarification

The supplied project repository **does not currently contain application or Jenkins code that actively uploads to or downloads runtime data from S3**.

The old project notes include an Amazon S3 URL while downloading a historical `kubectl` binary:

```text
amazon-eks.s3...
```

That does **not** mean the ShopEasy application is using S3 as a runtime storage component.

Therefore, the architecture image shows S3 as:

```text
OPTIONAL
```

Possible future uses:

```text
Pipeline artifacts
Build reports
Static website assets
Backup files
Log archives
Terraform state
```

If S3 is later added, the README can be updated from:

```text
S3 = optional supporting service
```

to:

```text
S3 = active application / DevOps dependency
```

This distinction keeps the project technically accurate.

---

## Container Registry

The current project uses:

```text
Docker Hub
```

AWS ECR can also be used in the future:

```text
Jenkins
   ↓
Amazon ECR
   ↓
Amazon EKS
```

---

# 1️⃣2️⃣ Step-by-Step Implementation Flow

![Step-by-Step Implementation Flow](docs/images/12-step-by-step-implementation-flow.png)

This image shows the actual recommended order to build the project.

## Phase 1 — Prepare repository

Create:

```text
app.py
requirements.txt
Dockerfile
Jenkinsfile
k8s/
argocd/
```

---

## Phase 2 — Prepare Jenkins host

Launch:

```text
Ubuntu EC2
```

Install:

```text
Java
Jenkins
Git
Docker
```

---

## Phase 3 — Install Kubernetes/AWS tooling

Install:

```text
AWS CLI
kubectl
eksctl
Helm
```

---

## Phase 4 — Create EKS

Use:

```bash
eksctl create cluster
```

---

## Phase 5 — Test Kubernetes manually

Before GitOps, verify:

```bash
kubectl apply
```

works.

---

## Phase 6 — Configure Jenkins CI

Add:

```text
Docker credentials
GitHub credentials
Multibranch Pipeline
```

---

## Phase 7 — Configure Argo CD

Install Argo CD and apply:

```text
Application CR
```

---

## Phase 8 — Monitoring

Install:

```text
kube-prometheus-stack
```

---

## Phase 9 — Run end-to-end release test

Change application source.

Then validate:

```text
Git commit
↓
Jenkins build
↓
Docker image
↓
Manifest commit
↓
Argo CD sync
↓
New EKS Pods
```

---

## Phase 10 — Cleanup

Remove unused resources to avoid AWS charges.

---

# 1️⃣3️⃣ Three Critical Data Paths

![Source, GitOps and Runtime Paths](docs/images/13-critical-data-and-traffic-paths.png)

Understanding these three paths makes the project much easier to study.

---

## A. Source / CI path

```text
Developer
   ↓
GitHub
   ↓
Jenkins
   ↓
Docker Build
   ↓
Registry
```

This path produces the application artifact.

---

## B. Deployment / GitOps path

```text
Jenkins
   ↓
deployment.yaml
   ↓
Git Commit
   ↓
Argo CD
   ↓
EKS
```

This path controls the deployed version.

---

## C. Runtime traffic / observability path

```text
User
   ↓
AWS LoadBalancer
   ↓
Kubernetes Service
   ↓
Flask Pods
   ↓
Prometheus
   ↓
Grafana
```

This path explains both:

```text
How users reach the application
```

and:

```text
How engineers observe the application/platform
```

---

# 📂 Recommended Repository Structure

The original archive stores branch examples in folders.

For a proper GitHub Multibranch project, use actual Git branches.

Recommended main-branch layout:

```text
ShopEasy-DevOps/
│
├── app.py
├── requirements.txt
├── Dockerfile
├── Jenkinsfile
├── README.md
│
├── docs/
│   └── images/
│       ├── 00-project-visual-overview.png
│       ├── 01-high-level-architecture.png
│       ├── 02-end-to-end-cicd-flow.png
│       ├── 03-jenkins-pipeline.png
│       ├── 04-docker-image-lifecycle.png
│       ├── 05-argocd-gitops.png
│       ├── 06-kubernetes-runtime.png
│       ├── 07-monitoring-observability.png
│       ├── 08-git-branching-strategy.png
│       ├── 09-deployment-sequence.png
│       ├── 10-production-hardening.png
│       ├── 11-aws-cloud-deployment-architecture.png
│       ├── 12-step-by-step-implementation-flow.png
│       └── 13-critical-data-and-traffic-paths.png
│
├── k8s/
│   ├── deployment.yaml
│   └── service.yaml
│
└── argocd/
    └── application.yaml
```

Actual Git branches:

```text
main
feature/wishlist
feature/reviews
feature/login
```

---

# ✅ Prerequisites

Prepare:

```text
AWS account
GitHub account
Docker Hub account
Ubuntu/Linux server for Jenkins
IAM permissions for EKS
Git
Java
Docker
AWS CLI
kubectl
eksctl
Helm
```

---

# 🛠 Complete Installation and Deployment Guide

# Step 1 — Clone the repository

```bash
git clone \
  https://github.com/<YOUR_USERNAME>/<YOUR_REPOSITORY>.git
```

```bash
cd \
  <YOUR_REPOSITORY>
```

Verify:

```bash
git status
```

```bash
git branch -a
```

---

# Step 2 — Run ShopEasy locally

Create Python environment:

```bash
python3 \
  -m venv \
  venv
```

Activate:

```bash
source \
  venv/bin/activate
```

Install:

```bash
pip install \
  -r requirements.txt
```

Run:

```bash
python app.py
```

Test:

```bash
curl \
  http://localhost:5000
```

---

# Step 3 — Build Docker image locally

```bash
docker build \
  -t shopeasy-app:local \
  .
```

List:

```bash
docker images
```

Run:

```bash
docker run \
  -d \
  --name shopeasy \
  -p 5000:5000 \
  shopeasy-app:local
```

Check:

```bash
docker ps
```

Logs:

```bash
docker logs \
  shopeasy
```

Test:

```bash
curl \
  http://localhost:5000
```

---

# Step 4 — Prepare Jenkins EC2 server

Example operating system:

```text
Ubuntu 24.04
```

Update:

```bash
sudo apt update
```

Install Java:

```bash
sudo apt install \
  -y openjdk-17-jre-headless
```

Verify:

```bash
java -version
```

---

# Step 5 — Install Jenkins

Add repository key and Jenkins repository according to the Jenkins version you are using, then install:

```bash
sudo apt update
```

```bash
sudo apt install \
  -y jenkins
```

Start:

```bash
sudo systemctl enable \
  jenkins
```

```bash
sudo systemctl start \
  jenkins
```

Check:

```bash
sudo systemctl status \
  jenkins
```

Initial administrator password:

```bash
sudo cat \
  /var/lib/jenkins/secrets/initialAdminPassword
```

Open:

```text
http://<JENKINS_PUBLIC_IP>:8080
```

---

# Step 6 — Install Docker on Jenkins host

Install prerequisites:

```bash
sudo apt update

sudo apt install \
  -y ca-certificates curl
```

Install Docker Engine using Docker's official Ubuntu repository.

Verify:

```bash
docker --version
```

Add Jenkins to Docker group:

```bash
sudo usermod \
  -aG docker \
  jenkins
```

Restart Jenkins:

```bash
sudo systemctl restart \
  jenkins
```

---

# Step 7 — Install AWS CLI

```bash
sudo apt install \
  -y unzip curl
```

```bash
curl \
  "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" \
  -o awscliv2.zip
```

```bash
unzip \
  awscliv2.zip
```

```bash
sudo ./aws/install
```

Verify:

```bash
aws --version
```

Configure lab credentials if required:

```bash
aws configure
```

Verify identity:

```bash
aws sts \
  get-caller-identity
```

---

# Step 8 — Install kubectl

Install a kubectl version compatible with the EKS/Kubernetes version.

Verify:

```bash
kubectl version \
  --client
```

---

# Step 9 — Install eksctl

Install `eksctl`.

Verify:

```bash
eksctl version
```

---

# Step 10 — Install Helm

Install Helm.

Verify:

```bash
helm version
```

---

# Step 11 — Create Amazon EKS

Set reusable variables:

```bash
export CLUSTER_NAME="shopeasy-eks"
```

```bash
export AWS_REGION="us-east-1"
```

Create cluster:

```bash
eksctl create cluster \
  --name "$CLUSTER_NAME" \
  --region "$AWS_REGION" \
  --nodegroup-name shopeasy-ng \
  --node-type t3.medium \
  --nodes 2 \
  --managed
```

Configure Kubernetes access:

```bash
aws eks update-kubeconfig \
  --region "$AWS_REGION" \
  --name "$CLUSTER_NAME"
```

Verify:

```bash
kubectl get nodes
```

Expected concept:

```text
NODE 1   Ready
NODE 2   Ready
```

---

# Step 12 — Deploy manually before GitOps

This is an important validation step.

Apply Deployment:

```bash
kubectl apply \
  -f k8s/deployment.yaml
```

Apply Service:

```bash
kubectl apply \
  -f k8s/service.yaml
```

Check:

```bash
kubectl get deployment
```

```bash
kubectl get pods \
  -o wide
```

```bash
kubectl get svc
```

---

# Step 13 — Jenkins credentials

Create Docker credentials.

Recommended ID:

```text
dockerhub-creds
```

Create GitHub credentials.

Recommended ID:

```text
github-creds
```

Path:

```text
Manage Jenkins
   ↓
Credentials
   ↓
System
   ↓
Global credentials
```

---

# Step 14 — Jenkins Multibranch Pipeline

Go to:

```text
New Item
   ↓
Multibranch Pipeline
```

Configure:

```text
Branch Sources
   ↓
GitHub
   ↓
Repository URL
   ↓
Credentials
```

Jenkins looks for:

```text
Jenkinsfile
```

in each branch.

---

# Step 15 — Run Jenkins build

After merging to `main`, verify:

```text
Checkout             SUCCESS
Build Docker Image   SUCCESS
Docker Push          SUCCESS
Manifest Update      SUCCESS
Git Push             SUCCESS
```

---

# Step 16 — Install Argo CD

Add Helm repo:

```bash
helm repo add \
  argo \
  https://argoproj.github.io/argo-helm
```

Update:

```bash
helm repo update
```

Create namespace:

```bash
kubectl create namespace \
  argocd
```

Install:

```bash
helm install \
  argocd \
  argo/argo-cd \
  -n argocd
```

Check:

```bash
kubectl get pods \
  -n argocd
```

---

# Step 17 — Access Argo CD

For a lab, patch the service:

```bash
kubectl patch svc \
  argocd-server \
  -n argocd \
  -p '{"spec":{"type":"LoadBalancer"}}'
```

Check:

```bash
kubectl get svc \
  argocd-server \
  -n argocd
```

Admin password:

```bash
kubectl get secret \
  argocd-initial-admin-secret \
  -n argocd \
  -o jsonpath="{.data.password}" \
  | base64 -d
```

---

# Step 18 — Apply Argo CD Application

```bash
kubectl apply \
  -f argocd/application.yaml
```

Check:

```bash
kubectl get applications \
  -n argocd
```

Desired status:

```text
Synced
Healthy
```

---

# Step 19 — Install Prometheus and Grafana

Add:

```bash
helm repo add \
  prometheus-community \
  https://prometheus-community.github.io/helm-charts
```

Update:

```bash
helm repo update
```

Create namespace:

```bash
kubectl create namespace \
  monitoring
```

Install:

```bash
helm install \
  monitoring \
  prometheus-community/kube-prometheus-stack \
  -n monitoring
```

Verify:

```bash
kubectl get pods \
  -n monitoring
```

---

# Step 20 — Access Grafana

For a lab:

```bash
kubectl patch svc \
  monitoring-grafana \
  -n monitoring \
  -p '{"spec":{"type":"LoadBalancer"}}'
```

Get service:

```bash
kubectl get svc \
  monitoring-grafana \
  -n monitoring
```

Get password:

```bash
kubectl get secret \
  monitoring-grafana \
  -n monitoring \
  -o jsonpath="{.data.admin-password}" \
  | base64 -d
```

---

# 🐳 Dockerfile

Current project concept:

```dockerfile
FROM python:3.11-slim

WORKDIR /app

COPY requirements.txt .

RUN pip install \
    --no-cache-dir \
    -r requirements.txt

COPY app.py .

EXPOSE 5000

CMD ["python", "app.py"]
```

---

# ⚙ Jenkinsfile

A cleaned version aligned with this README:

```groovy
pipeline {

    agent any

    options {

        disableConcurrentBuilds()
    }

    environment {

        IMAGE_NAME = "<DOCKERHUB_USERNAME>/shopeasy-app"
    }

    stages {

        stage('Checkout') {

            steps {

                checkout scm
            }
        }

        stage('Build and Push Image') {

            when {

                branch 'main'
            }

            steps {

                script {

                    env.IMAGE_TAG = "build-${BUILD_NUMBER}"

                    withCredentials([

                        usernamePassword(

                            credentialsId: 'dockerhub-creds',

                            usernameVariable: 'DOCKER_USER',

                            passwordVariable: 'DOCKER_PASS'
                        )
                    ]) {

                        sh '''
                            docker build \
                              -t ${IMAGE_NAME}:${IMAGE_TAG} \
                              .

                            echo "$DOCKER_PASS" \
                              | docker login \
                                  -u "$DOCKER_USER" \
                                  --password-stdin

                            docker push \
                              ${IMAGE_NAME}:${IMAGE_TAG}
                        '''
                    }
                }
            }
        }

        stage('Update Kubernetes Manifest') {

            when {

                branch 'main'
            }

            steps {

                script {

                    withCredentials([

                        usernamePassword(

                            credentialsId: 'github-creds',

                            usernameVariable: 'GIT_USERNAME',

                            passwordVariable: 'GIT_TOKEN'
                        )
                    ]) {

                        sh '''
                            set -e

                            git config \
                              user.name \
                              "Jenkins"

                            git config \
                              user.email \
                              "jenkins@automation.local"

                            git fetch \
                              origin

                            git checkout \
                              main

                            git reset \
                              --hard origin/main

                            sed -i \
                              "s|image:.*|image: ${IMAGE_NAME}:${IMAGE_TAG}|" \
                              k8s/deployment.yaml

                            git add \
                              k8s/deployment.yaml

                            git diff \
                              --cached \
                              --quiet \
                              || git commit \
                                   -m "chore(deploy): update image to ${IMAGE_TAG}"

                            git push \
                              https://${GIT_USERNAME}:${GIT_TOKEN}@github.com/<YOUR_USERNAME>/<YOUR_REPOSITORY>.git \
                              main
                        '''
                    }
                }
            }
        }
    }
}
```

---

# ☸ Kubernetes Manifests

## `k8s/deployment.yaml`

```yaml
apiVersion: apps/v1

kind: Deployment

metadata:

  name: shopeasy-app

  namespace: default

spec:

  replicas: 5

  selector:

    matchLabels:

      app: shopeasy-app

  template:

    metadata:

      labels:

        app: shopeasy-app

    spec:

      containers:

        - name: shopeasy-app

          image: <DOCKERHUB_USERNAME>/shopeasy-app:build-1

          ports:

            - containerPort: 5000
```

---

## `k8s/service.yaml`

```yaml
apiVersion: v1

kind: Service

metadata:

  name: shopeasy-service

spec:

  type: LoadBalancer

  selector:

    app: shopeasy-app

  ports:

    - port: 80

      targetPort: 5000
```

---

# 🔄 Argo CD Application

## `argocd/application.yaml`

```yaml
apiVersion: argoproj.io/v1alpha1

kind: Application

metadata:

  name: shopeasy-app

  namespace: argocd

spec:

  project: default

  source:

    repoURL: https://github.com/<YOUR_USERNAME>/<YOUR_REPOSITORY>.git

    targetRevision: main

    path: k8s

  destination:

    server: https://kubernetes.default.svc

    namespace: default

  syncPolicy:

    automated:

      prune: true

      selfHeal: true
```

---

# 📊 Prometheus and Grafana

Install:

```bash
helm repo add \
  prometheus-community \
  https://prometheus-community.github.io/helm-charts
```

```bash
helm repo update
```

```bash
kubectl create namespace \
  monitoring
```

```bash
helm install \
  monitoring \
  prometheus-community/kube-prometheus-stack \
  -n monitoring
```

Check:

```bash
kubectl get pods \
  -n monitoring
```

Expected components include:

```text
Prometheus
Grafana
Alertmanager
Node Exporter
kube-state-metrics
```

---

# ✅ Full Validation Checklist

## Git

```bash
git status
```

```bash
git branch -a
```

```bash
git log \
  --oneline \
  -5
```

---

## Application

```bash
python app.py
```

```bash
curl \
  http://localhost:5000
```

---

## Docker

```bash
docker images
```

```bash
docker ps
```

---

## AWS identity

```bash
aws sts \
  get-caller-identity
```

---

## EKS nodes

```bash
kubectl get nodes
```

---

## Deployment

```bash
kubectl get deployment
```

---

## Pods

```bash
kubectl get pods \
  -o wide
```

---

## Service

```bash
kubectl get svc
```

---

## Current image

```bash
kubectl get deployment shopeasy-app \
  -o jsonpath='{.spec.template.spec.containers[0].image}'

echo
```

---

## Argo CD

```bash
kubectl get applications \
  -n argocd
```

Expected:

```text
Synced
Healthy
```

---

## Monitoring

```bash
kubectl get pods \
  -n monitoring
```

---

# ⚠ Important Corrections Found in the Original Project

The original uploaded project contains several inconsistencies.

These should be corrected before publishing the repository as a polished portfolio project.

---

## 1. `.yaml` vs `.yml`

Actual file:

```text
k8s/deployment.yaml
```

Original Jenkinsfile references:

```text
k8s/deployment.yml
```

Correct Jenkinsfile to:

```text
k8s/deployment.yaml
```

Otherwise:

```text
sed
git add
```

will fail.

---

## 2. Cluster-name mismatch

Original cluster creation:

```text
kastro-cluster
```

Original kubeconfig command:

```text
eks-cluster
```

Use one cluster name consistently.

Recommended:

```text
shopeasy-eks
```

---

## 3. Jenkins and Argo CD repository mismatch

The original Jenkinsfile pushes to one repository.

The Argo CD Application watches another repository.

That can be a valid two-repository GitOps design **only when it is intentional**.

You must decide between:

### Single repository

```text
Application code
+
Kubernetes manifests
```

in one repository.

or:

### Separate repositories

```text
Repository 1
Application source

Repository 2
GitOps deployment state
```

If using two repositories, Jenkins must explicitly clone/update the GitOps repository.

---

## 4. Application naming mismatch

Original Kubernetes resources are named:

```text
movie-app
movie-app-service
```

but the Flask workload is ShopEasy/e-commerce.

Recommended names:

```text
shopeasy-app
shopeasy-service
```

---

## 5. Old owner-specific values

Replace values such as:

```text
Docker namespace
GitHub username
Git repository URLs
Git email
Argo CD repoURL
```

with your own project values.

---

## 6. S3 is not currently an application dependency

Do not describe S3 as an active runtime data store unless you actually add S3 integration.

Correct wording:

```text
S3 = optional supporting AWS service
```

for:

```text
artifacts
static assets
backups
reports
logs
Terraform state
```

---

# 🛠 Troubleshooting

## Jenkins: Docker permission denied

Check:

```bash
id jenkins
```

Add user:

```bash
sudo usermod \
  -aG docker \
  jenkins
```

Restart:

```bash
sudo systemctl restart \
  jenkins
```

---

## Jenkins cannot find deployment file

Error:

```text
k8s/deployment.yml:
No such file or directory
```

Fix:

```text
deployment.yml
```

to:

```text
deployment.yaml
```

---

## `ImagePullBackOff`

Describe Pod:

```bash
kubectl describe pod \
  <POD_NAME>
```

Check image:

```bash
kubectl get deployment shopeasy-app \
  -o jsonpath='{.spec.template.spec.containers[0].image}'

echo
```

Possible causes:

```text
Wrong image
Wrong tag
Private registry
Missing imagePullSecret
```

---

## `CrashLoopBackOff`

Logs:

```bash
kubectl logs \
  <POD_NAME>
```

Previous container:

```bash
kubectl logs \
  <POD_NAME> \
  --previous
```

---

## Argo CD `OutOfSync`

Inspect:

```bash
kubectl describe application shopeasy-app \
  -n argocd
```

Validate Kubernetes YAML:

```bash
kubectl apply \
  --dry-run=client \
  -f k8s/
```

---

## kubectl cannot connect

Reconfigure:

```bash
aws eks update-kubeconfig \
  --region "$AWS_REGION" \
  --name "$CLUSTER_NAME"
```

Verify AWS identity:

```bash
aws sts \
  get-caller-identity
```

---

# 🧹 Cleanup

AWS resources can generate charges.

## Remove application

```bash
kubectl delete \
  -f argocd/application.yaml
```

```bash
kubectl delete \
  -f k8s/service.yaml
```

```bash
kubectl delete \
  -f k8s/deployment.yaml
```

---

## Remove monitoring

```bash
helm uninstall \
  monitoring \
  -n monitoring
```

```bash
kubectl delete namespace \
  monitoring
```

---

## Remove Argo CD

```bash
helm uninstall \
  argocd \
  -n argocd
```

```bash
kubectl delete namespace \
  argocd
```

---

## Delete EKS cluster

```bash
eksctl delete cluster \
  --name "$CLUSTER_NAME" \
  --region "$AWS_REGION"
```

Also inspect AWS for remaining:

```text
EC2 instances
Load Balancers
EBS volumes
Elastic IPs
NAT Gateways
CloudFormation stacks
S3 data if you created any buckets
```

---

# 🧠 Skills Demonstrated

This repository can demonstrate practical knowledge of:

```text
Git
GitHub
Branching
Pull Requests
Jenkins
Jenkinsfile
Multibranch Pipeline
CI/CD
Docker
Container Registry
Linux
AWS
EC2
Amazon EKS
Amazon S3 concepts
Kubernetes
kubectl
eksctl
Helm
Argo CD
GitOps
Prometheus
Grafana
Observability
Troubleshooting
Python
Flask
```

---

# 👨‍💻 Author

## Rahmanuddin MD

**Engineering Automation | AI/ML | Cloud & DevOps**

GitHub:

```text
https://github.com/rahmanuddinmd
```

LinkedIn:

```text
https://www.linkedin.com/in/md-rahmanuddin/
```

---

<div align="center">

# 🚀 Build • Automate • Containerize • Deploy • Observe • Improve

### A complete visual journey from source code to AWS EKS.

</div>
