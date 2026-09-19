<div align="center">

# 🚀 ShopEasy — End-to-End CI/CD & GitOps Deployment on AWS EKS

### A Visual DevOps Study Guide + Portfolio Project

[![Python](https://img.shields.io/badge/Python-Flask-3776AB?logo=python&logoColor=white)](https://www.python.org/)
[![Jenkins](https://img.shields.io/badge/Jenkins-CI%2FCD-D24939?logo=jenkins&logoColor=white)](https://www.jenkins.io/)
[![Docker](https://img.shields.io/badge/Docker-Containerization-2496ED?logo=docker&logoColor=white)](https://www.docker.com/)
[![Kubernetes](https://img.shields.io/badge/Kubernetes-AWS%20EKS-326CE5?logo=kubernetes&logoColor=white)](https://kubernetes.io/)
[![AWS](https://img.shields.io/badge/AWS-EKS-232F3E?logo=amazonwebservices&logoColor=white)](https://aws.amazon.com/eks/)
[![Argo CD](https://img.shields.io/badge/Argo%20CD-GitOps-EF7B4D?logo=argo&logoColor=white)](https://argo-cd.readthedocs.io/)
[![Prometheus](https://img.shields.io/badge/Prometheus-Monitoring-E6522C?logo=prometheus&logoColor=white)](https://prometheus.io/)
[![Grafana](https://img.shields.io/badge/Grafana-Observability-F46800?logo=grafana&logoColor=white)](https://grafana.com/)

**Source Code → CI → Container → Registry → GitOps → Kubernetes → Monitoring**

</div>

---

# 📌 Project Overview

This repository demonstrates an **end-to-end cloud-native CI/CD and GitOps deployment workflow** for a Python Flask e-commerce application called **ShopEasy**.

The main purpose is to explain how all of the DevOps components connect together:

- Git and GitHub for source control
- Jenkins for Continuous Integration
- Docker for containerization
- Docker Hub or another registry for image storage
- Kubernetes for orchestration
- AWS EKS for managed Kubernetes
- Argo CD for GitOps Continuous Delivery
- Prometheus for metrics
- Grafana for visualization

This README is intentionally organized as:

> **Diagram → Detailed Explanation → Code/Commands → Verification**

---

# 📚 Contents

1. High-Level Architecture
2. End-to-End CI/CD Flow
3. Jenkins Multibranch Pipeline
4. Docker Image Lifecycle
5. Argo CD GitOps Architecture
6. Kubernetes Runtime Architecture
7. Monitoring and Observability
8. Git Branching Strategy
9. Deployment Sequence and Rolling Update
10. Production Hardening Roadmap
11. Repository Structure
12. Prerequisites
13. Complete Setup Guide
14. Jenkinsfile
15. Kubernetes YAML
16. Argo CD YAML
17. Validation Checklist
18. Troubleshooting
19. Cleanup

---

# 1️⃣ High-Level Architecture

![High-Level DevOps Architecture](docs/images/01-high-level-architecture.png)

## What this architecture shows

This diagram represents the complete high-level design of the project.

The deployment starts when a developer changes the Flask application. The code is pushed to GitHub and processed by Jenkins. Jenkins builds a Docker image and pushes it to a registry. Jenkins also updates the Kubernetes deployment manifest.

Argo CD continuously watches the Git repository for deployment-state changes. When the image tag changes, Argo CD synchronizes AWS EKS.

The EKS cluster runs the Flask application in Kubernetes Pods and exposes it through a LoadBalancer Service. Prometheus and Grafana provide operational visibility.

## Component responsibilities

| Component | Responsibility |
|---|---|
| Developer | Writes and modifies application code |
| GitHub | Stores source code and deployment files |
| Jenkins | Performs CI automation |
| Docker | Packages the application |
| Container Registry | Stores versioned images |
| Argo CD | Performs GitOps synchronization |
| AWS EKS | Runs the Kubernetes cluster |
| Kubernetes Service | Exposes the application |
| Prometheus | Collects metrics |
| Grafana | Displays dashboards |

## Architecture flow

```text
Developer
   ↓
GitHub
   ↓
Jenkins
   ↓
Docker Build
   ↓
Docker Registry
   ↓
Kubernetes Manifest Update
   ↓
Git Repository
   ↓
Argo CD
   ↓
AWS EKS
   ↓
LoadBalancer
   ↓
ShopEasy Application
   ↓
Prometheus + Grafana
```

---

# 2️⃣ End-to-End CI/CD Flow

![End-to-End CI/CD Flow](docs/images/02-end-to-end-cicd-flow.png)

## Complete process

### Step 1 — Developer changes code

The developer changes application files such as:

```text
app.py
requirements.txt
templates
static files
```

Check the working tree:

```bash
git status
```

### Step 2 — Create a feature branch

```bash
git checkout -b feature/wishlist
```

The feature branch isolates development from `main`.

### Step 3 — Push the branch

```bash
git add .
git commit -m "feat: add wishlist functionality"
git push -u origin feature/wishlist
```

### Step 4 — Pull Request

Create a Pull Request:

```text
feature/wishlist
        ↓
       main
```

The code can be reviewed before merge.

### Step 5 — Jenkins discovers the branch

The Jenkins Multibranch Pipeline scans GitHub for branches containing:

```text
Jenkinsfile
```

### Step 6 — Main branch triggers release stages

Production stages use:

```groovy
when {
    branch 'main'
}
```

Meaning:

```text
feature/* → branch validation
main      → build + push + GitOps update
```

### Step 7 — Jenkins builds the image

```bash
docker build \
  -t <DOCKERHUB_USER>/shopeasy-app:build-105 \
  .
```

### Step 8 — Jenkins pushes the image

```bash
docker push \
  <DOCKERHUB_USER>/shopeasy-app:build-105
```

### Step 9 — Jenkins updates Kubernetes YAML

Before:

```yaml
image: <DOCKERHUB_USER>/shopeasy-app:build-104
```

After:

```yaml
image: <DOCKERHUB_USER>/shopeasy-app:build-105
```

### Step 10 — Jenkins commits the manifest

```bash
git add k8s/deployment.yaml
git commit -m "chore(deploy): update image to build-105"
git push origin main
```

### Step 11 — Argo CD detects the change

Argo CD notices that the desired state stored in Git changed.

### Step 12 — Argo CD synchronizes EKS

Argo CD applies the new desired state to Kubernetes.

### Step 13 — Kubernetes performs a rolling update

New Pods start with:

```text
build-105
```

Old Pods using:

```text
build-104
```

are gradually replaced.

### Step 14 — Application becomes live

Traffic path:

```text
LoadBalancer :80
        ↓
Kubernetes Service
        ↓
Pod :5000
        ↓
Flask
```

### Step 15 — Prometheus collects metrics

Prometheus collects metrics from nodes and Kubernetes objects.

### Step 16 — Grafana visualizes the environment

Grafana dashboards help the DevOps engineer study and monitor cluster health.

---

# 3️⃣ Jenkins Multibranch Pipeline

![Jenkins Pipeline](docs/images/03-jenkins-pipeline.png)

## Why Multibranch Pipeline?

A Multibranch Pipeline can automatically discover:

```text
main
feature/wishlist
feature/reviews
feature/login
```

and create branch-specific pipeline runs.

## Main pipeline stages

### Checkout

```groovy
stage('Checkout') {
    steps {
        checkout scm
    }
}
```

`checkout scm` fetches the branch currently being processed.

### Main-branch gate

```groovy
when {
    branch 'main'
}
```

This prevents a normal feature branch from changing the production deployment state.

### Image tag

```groovy
env.IMAGE_TAG = "build-${BUILD_NUMBER}"
```

Example:

```text
Jenkins Build Number = 105
IMAGE_TAG = build-105
```

### Docker build

```bash
docker build \
  -t ${IMAGE_NAME}:${IMAGE_TAG} \
  .
```

### Registry login

```bash
echo "$DOCKER_PASS" \
  | docker login \
      -u "$DOCKER_USER" \
      --password-stdin
```

### Push image

```bash
docker push \
  ${IMAGE_NAME}:${IMAGE_TAG}
```

### Update manifest

```bash
sed -i \
  "s|image:.*|image: ${IMAGE_NAME}:${IMAGE_TAG}|" \
  k8s/deployment.yaml
```

### Commit and push

```bash
git add k8s/deployment.yaml
git commit -m "chore(deploy): update image to ${IMAGE_TAG}"
git push
```

That Git push is the handoff from **Continuous Integration** to **GitOps Continuous Delivery**.

---

# 4️⃣ Docker Image Lifecycle

![Docker Image Lifecycle](docs/images/04-docker-image-lifecycle.png)

## Why Docker is used

Docker packages:

```text
Application Code
+
Python Runtime
+
Dependencies
+
Startup Command
```

into one portable image.

The same image can run locally, in Jenkins testing, or in AWS EKS.

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

## Dockerfile explanation

`FROM python:3.11-slim` provides a lightweight Python runtime.

`WORKDIR /app` sets the container working directory.

`COPY requirements.txt .` copies the dependency definition.

`RUN pip install ...` installs Flask and other dependencies.

`COPY app.py .` copies application source.

`EXPOSE 5000` documents the application port.

`CMD ["python", "app.py"]` starts the application.

## Build locally

```bash
docker build \
  -t shopeasy-app:local \
  .
```

Check:

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

Verify:

```bash
docker ps
docker logs shopeasy
curl http://localhost:5000
```

## Why versioned image tags?

Prefer:

```text
shopeasy-app:build-101
shopeasy-app:build-102
shopeasy-app:build-103
```

instead of using only:

```text
shopeasy-app:latest
```

Versioned tags improve traceability, rollback, debugging, and auditability.

---

# 5️⃣ Argo CD GitOps Architecture

![Argo CD GitOps](docs/images/05-argocd-gitops.png)

## Traditional model

```text
Jenkins
   ↓
kubectl apply
   ↓
Kubernetes
```

## GitOps model used here

```text
Jenkins
   ↓
Git
   ↓
Argo CD
   ↓
Kubernetes
```

## Desired state

Git defines what Kubernetes should run.

Example:

```yaml
replicas: 5
image: youruser/shopeasy-app:build-105
```

## Actual state

AWS EKS contains what is currently running.

Argo CD compares:

```text
Desired State in Git
        VS
Actual State in EKS
```

## Automated synchronization

```yaml
syncPolicy:
  automated:
    prune: true
    selfHeal: true
```

`automated` enables automatic synchronization.

`prune: true` allows resources removed from Git to be removed from the cluster.

`selfHeal: true` can correct live-cluster drift back to the Git-defined state.

---

# 6️⃣ Kubernetes Runtime Architecture

![Kubernetes Runtime](docs/images/06-kubernetes-runtime.png)

## Objects used

The application primarily uses:

```text
Deployment
ReplicaSet
Pods
Service
```

## Deployment example

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: shopeasy-app

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
        - name: shopeasy
          image: youruser/shopeasy-app:build-105

          ports:
            - containerPort: 5000
```

## What `replicas: 5` means

Kubernetes attempts to maintain five running Pods.

If one Pod fails:

```text
Desired = 5
Actual  = 4
```

the Deployment controller creates a replacement Pod.

## Service example

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
    - protocol: TCP
      port: 80
      targetPort: 5000
```

## Traffic flow

```text
Internet
   ↓
AWS Load Balancer
   ↓
Kubernetes Service :80
   ↓
targetPort :5000
   ↓
Flask Container :5000
```

The Pod label and Service selector must match:

```yaml
app: shopeasy-app
```

---

# 7️⃣ Monitoring and Observability

![Monitoring Architecture](docs/images/07-monitoring-observability.png)

## Why monitoring is required

A successful deployment does not automatically prove that the platform is healthy.

You need visibility into:

- CPU
- memory
- Pods
- nodes
- restarts
- Deployment state
- resource utilization
- alerts

## Stack

This project uses:

```text
kube-prometheus-stack
```

which normally includes:

```text
Prometheus
Grafana
Alertmanager
Node Exporter
kube-state-metrics
```

## Node Exporter

Provides:

```text
CPU
Memory
Filesystem
Network
```

## kube-state-metrics

Provides Kubernetes object-state metrics such as:

```text
Deployment replicas
Pod states
ReplicaSets
Services
Namespaces
```

## Prometheus

Prometheus scrapes metrics, stores time-series data, and evaluates alert rules.

## Grafana

Grafana connects to Prometheus and displays dashboards.

Useful dashboards include:

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

## Recommended branch model

```text
main
│
├── feature/wishlist
├── feature/reviews
└── feature/login
```

Create a feature branch:

```bash
git checkout -b feature/wishlist
```

Stage changes:

```bash
git add .
```

Commit:

```bash
git commit \
  -m "feat: add wishlist functionality"
```

Push:

```bash
git push \
  -u origin feature/wishlist
```

Create:

```text
feature/wishlist → main
```

After review and approval, merge into `main`.

---

# 9️⃣ Deployment Sequence and Rolling Update

![Deployment Sequence](docs/images/09-deployment-sequence.png)

Suppose the cluster currently runs:

```text
build-104
```

Jenkins updates the deployment to:

```text
build-105
```

Argo CD detects the Git commit and synchronizes the new Deployment.

The sequence becomes:

```text
Git
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
Ready
 ↓
Old ReplicaSet scales down
```

## Watch rollout

```bash
kubectl rollout status \
  deployment/shopeasy-app
```

View ReplicaSets:

```bash
kubectl get rs
```

View Pods:

```bash
kubectl get pods
```

View currently deployed image:

```bash
kubectl get deployment shopeasy-app \
  -o jsonpath='{.spec.template.spec.containers[0].image}'

echo
```

View rollout history:

```bash
kubectl rollout history \
  deployment/shopeasy-app
```

For a GitOps workflow, prefer reverting the Git desired state for a durable rollback.

---

# 🔟 Production Hardening Roadmap

![Production Hardening](docs/images/10-production-hardening.png)

The current project demonstrates a strong learning architecture. A hardened production environment should add more controls.

## Quality gates

Add:

```text
Linting
Unit Tests
Integration Tests
Static Analysis
```

## Supply-chain security

Possible tooling:

```text
Trivy
Grype
SBOM generation
Image signing
Dependency pinning
```

Example:

```bash
trivy image \
  youruser/shopeasy-app:build-105
```

## Kubernetes reliability

Add readiness and liveness probes:

```yaml
readinessProbe:

  httpGet:
    path: /
    port: 5000

  initialDelaySeconds: 5
  periodSeconds: 10
```

```yaml
livenessProbe:

  httpGet:
    path: /
    port: 5000

  initialDelaySeconds: 15
  periodSeconds: 20
```

Add resources:

```yaml
resources:

  requests:
    cpu: "100m"
    memory: "128Mi"

  limits:
    cpu: "500m"
    memory: "512Mi"
```

Also consider:

```text
Horizontal Pod Autoscaler
Pod Disruption Budget
Ingress
TLS
RBAC
Secrets Manager
Centralized logging
SLOs and alerts
Terraform/OpenTofu
Dev/Staging/Production separation
Canary or blue/green deployment
```

---

# 📂 Repository Structure

Recommended structure:

```text
shopeasy-devops/
│
├── app.py
├── requirements.txt
├── Dockerfile
├── Jenkinsfile
├── README.md
│
├── docs/
│   └── images/
│       ├── 01-high-level-architecture.png
│       ├── 02-end-to-end-cicd-flow.png
│       ├── 03-jenkins-pipeline.png
│       ├── 04-docker-image-lifecycle.png
│       ├── 05-argocd-gitops.png
│       ├── 06-kubernetes-runtime.png
│       ├── 07-monitoring-observability.png
│       ├── 08-git-branching-strategy.png
│       ├── 09-deployment-sequence.png
│       └── 10-production-hardening.png
│
├── k8s/
│   ├── deployment.yaml
│   └── service.yaml
│
└── argocd/
    └── application.yaml
```

GitHub renders:

```markdown
![High-Level Architecture](docs/images/01-high-level-architecture.png)
```

only when the relative file exists in the repository.

---

# ✅ Prerequisites

Prepare:

```text
AWS account
GitHub account
Docker Hub / registry account
Ubuntu/Linux Jenkins server
Git
Java
Docker
AWS CLI
kubectl
eksctl
Helm
IAM permissions for EKS
```

---

# 🛠 Complete Setup Guide

## Step 1 — Clone

```bash
git clone \
  https://github.com/<YOUR_USERNAME>/<YOUR_REPOSITORY>.git

cd <YOUR_REPOSITORY>
```

Verify:

```bash
git status
git branch -a
```

## Step 2 — Run Flask locally

```bash
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
python app.py
```

Test:

```bash
curl \
  http://localhost:5000
```

## Step 3 — Build Docker image

```bash
docker build \
  -t shopeasy-app:local \
  .
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
docker logs shopeasy
```

## Step 4 — Install Jenkins

```bash
sudo apt update

sudo apt install \
  -y fontconfig openjdk-21-jre
```

```bash
java -version
```

```bash
sudo mkdir -p /etc/apt/keyrings

sudo wget \
  -O /etc/apt/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian/jenkins.io-2026.key
```

```bash
echo \
  "deb [signed-by=/etc/apt/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian binary/" \
  | sudo tee \
      /etc/apt/sources.list.d/jenkins.list \
      > /dev/null
```

```bash
sudo apt update
sudo apt install -y jenkins
sudo systemctl enable jenkins
sudo systemctl start jenkins
```

Initial password:

```bash
sudo cat \
  /var/lib/jenkins/secrets/initialAdminPassword
```

## Step 5 — Install Docker for Jenkins

```bash
sudo apt update
sudo apt install -y ca-certificates curl
```

```bash
sudo install -m 0755 -d /etc/apt/keyrings

sudo curl \
  -fsSL \
  https://download.docker.com/linux/ubuntu/gpg \
  -o /etc/apt/keyrings/docker.asc

sudo chmod a+r /etc/apt/keyrings/docker.asc
```

```bash
sudo tee /etc/apt/sources.list.d/docker.sources > /dev/null <<EOF
Types: deb
URIs: https://download.docker.com/linux/ubuntu
Suites: $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}")
Components: stable
Architectures: $(dpkg --print-architecture)
Signed-By: /etc/apt/keyrings/docker.asc
EOF
```

```bash
sudo apt update

sudo apt install \
  -y \
  docker-ce \
  docker-ce-cli \
  containerd.io \
  docker-buildx-plugin \
  docker-compose-plugin
```

```bash
sudo usermod -aG docker jenkins
sudo systemctl restart jenkins
```

## Step 6 — Install AWS CLI

```bash
sudo apt install -y unzip curl

curl \
  "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" \
  -o awscliv2.zip

unzip awscliv2.zip

sudo ./aws/install
```

```bash
aws --version
aws configure
aws sts get-caller-identity
```

## Step 7 — Install kubectl

```bash
curl \
  -LO \
  "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
```

```bash
chmod +x kubectl

sudo mv \
  kubectl \
  /usr/local/bin/kubectl
```

```bash
kubectl version --client
```

## Step 8 — Install eksctl

```bash
curl \
  --silent \
  --location \
  "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" \
  | tar xz -C /tmp

sudo mv \
  /tmp/eksctl \
  /usr/local/bin
```

```bash
eksctl version
```

## Step 9 — Install Helm

```bash
curl \
  https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 \
  | bash
```

```bash
helm version
```

## Step 10 — Create EKS

```bash
export CLUSTER_NAME="shopeasy-eks"
export AWS_REGION="us-east-1"
```

```bash
eksctl create cluster \
  --name "$CLUSTER_NAME" \
  --region "$AWS_REGION" \
  --nodegroup-name shopeasy-ng \
  --node-type t3.medium \
  --nodes 2 \
  --managed
```

```bash
aws eks update-kubeconfig \
  --region "$AWS_REGION" \
  --name "$CLUSTER_NAME"
```

```bash
kubectl get nodes
```

## Step 11 — Deploy Kubernetes resources

```bash
kubectl apply \
  -f k8s/deployment.yaml

kubectl apply \
  -f k8s/service.yaml
```

```bash
kubectl get deployment
kubectl get pods
kubectl get svc
```

## Step 12 — Configure Jenkins credentials

Create:

```text
dockerhub-creds
github-creds
```

under:

```text
Manage Jenkins
→ Credentials
→ System
→ Global credentials
```

## Step 13 — Create Multibranch Pipeline

```text
New Item
→ Multibranch Pipeline
→ Branch Sources
→ GitHub
→ Select credentials
→ Repository
→ Save
```

## Step 14 — Install Argo CD

```bash
helm repo add \
  argo \
  https://argoproj.github.io/argo-helm

helm repo update
```

```bash
helm install \
  argocd \
  argo/argo-cd \
  --namespace argocd \
  --create-namespace
```

```bash
kubectl get pods \
  -n argocd
```

Apply:

```bash
kubectl apply \
  -f argocd/application.yaml
```

## Step 15 — Install Prometheus and Grafana

```bash
helm repo add \
  prometheus-community \
  https://prometheus-community.github.io/helm-charts

helm repo update
```

```bash
helm install \
  monitoring \
  prometheus-community/kube-prometheus-stack \
  --namespace monitoring \
  --create-namespace
```

```bash
kubectl get pods \
  -n monitoring
```

---

# ⚙ Jenkinsfile

```groovy
pipeline {

    agent any

    environment {

        IMAGE_NAME = "<DOCKERHUB_USERNAME>/shopeasy-app"
    }

    stages {

        stage('Checkout') {

            steps {

                checkout scm
            }
        }

        stage('Build and Push Docker Image') {

            when {

                branch 'main'
            }

            steps {

                script {

                    env.IMAGE_TAG = "build-${BUILD_NUMBER}"
                }

                withCredentials([

                    usernamePassword(

                        credentialsId: 'dockerhub-creds',

                        usernameVariable: 'DOCKER_USER',

                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {

                    sh '''
                        echo "$DOCKER_PASS" \
                          | docker login \
                              -u "$DOCKER_USER" \
                              --password-stdin

                        docker build \
                          -t ${IMAGE_NAME}:${IMAGE_TAG} \
                          .

                        docker push \
                          ${IMAGE_NAME}:${IMAGE_TAG}
                    '''
                }
            }
        }

        stage('Update Kubernetes Manifest') {

            when {

                branch 'main'
            }

            steps {

                sh '''
                    sed -i \
                      "s|image:.*|image: ${IMAGE_NAME}:${IMAGE_TAG}|" \
                      k8s/deployment.yaml

                    git config user.name "Jenkins"

                    git config \
                      user.email \
                      "jenkins@automation.local"

                    git add \
                      k8s/deployment.yaml

                    git commit \
                      -m "chore(deploy): update image to ${IMAGE_TAG}" \
                      || true
                '''

                withCredentials([

                    usernamePassword(

                        credentialsId: 'github-creds',

                        usernameVariable: 'GIT_USER',

                        passwordVariable: 'GIT_TOKEN'
                    )
                ]) {

                    sh '''
                        git push \
                          https://${GIT_USER}:${GIT_TOKEN}@github.com/<YOUR_USERNAME>/<YOUR_REPOSITORY>.git \
                          HEAD:main
                    '''
                }
            }
        }
    }
}
```

---

# ☸ Kubernetes YAML

## `k8s/deployment.yaml`

```yaml
apiVersion: apps/v1

kind: Deployment

metadata:

  name: shopeasy-app

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

        - name: shopeasy

          image: <DOCKERHUB_USERNAME>/shopeasy-app:build-1

          ports:

            - containerPort: 5000
```

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

    - protocol: TCP

      port: 80

      targetPort: 5000
```

---

# 🔄 Argo CD YAML

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

# ✅ Validation Checklist

## Git

```bash
git status
git branch -a
git log --oneline -5
```

## Flask

```bash
python app.py
curl http://localhost:5000
```

## Docker

```bash
docker images
docker ps
```

## AWS

```bash
aws sts get-caller-identity
```

## EKS

```bash
kubectl get nodes
```

## Application

```bash
kubectl get deployment
kubectl get pods
kubectl get svc
```

## Image currently deployed

```bash
kubectl get deployment shopeasy-app \
  -o jsonpath='{.spec.template.spec.containers[0].image}'

echo
```

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

## Monitoring

```bash
kubectl get pods \
  -n monitoring
```

---

# 🛠 Troubleshooting

## Jenkins Docker permission problem

```bash
sudo usermod \
  -aG docker \
  jenkins

sudo systemctl restart jenkins
```

## Wrong deployment filename

Use:

```text
k8s/deployment.yaml
```

not:

```text
k8s/deployment.yml
```

## `ImagePullBackOff`

```bash
kubectl describe pod \
  <POD_NAME>
```

```bash
kubectl get deployment shopeasy-app \
  -o jsonpath='{.spec.template.spec.containers[0].image}'

echo
```

Possible causes:

```text
Wrong image name
Missing image tag
Private registry
Missing pull credentials
```

## `CrashLoopBackOff`

```bash
kubectl logs \
  <POD_NAME>
```

```bash
kubectl logs \
  <POD_NAME> \
  --previous
```

## Argo CD `OutOfSync`

```bash
kubectl describe application shopeasy-app \
  -n argocd
```

```bash
kubectl apply \
  --dry-run=client \
  -f k8s/
```

## kubectl cannot connect

```bash
aws eks update-kubeconfig \
  --region "$AWS_REGION" \
  --name "$CLUSTER_NAME"
```

```bash
aws sts get-caller-identity
```

---

# ⚠ Important Repository Corrections

Before publishing, make the following values consistent.

Use:

```text
k8s/deployment.yaml
```

Use one cluster name, for example:

```text
shopeasy-eks
```

Use consistent Kubernetes names:

```text
shopeasy-app
shopeasy-service
```

Replace old owner-specific values:

```text
Docker username
GitHub username
Repository URL
Git user.name
Git user.email
Argo CD repoURL
Jenkins Git push URL
```

---

# 🧹 Cleanup

Delete Argo CD Application:

```bash
kubectl delete \
  -f argocd/application.yaml
```

Delete application:

```bash
kubectl delete \
  -f k8s/service.yaml

kubectl delete \
  -f k8s/deployment.yaml
```

Delete monitoring:

```bash
helm uninstall \
  monitoring \
  -n monitoring

kubectl delete namespace \
  monitoring
```

Delete Argo CD:

```bash
helm uninstall \
  argocd \
  -n argocd

kubectl delete namespace \
  argocd
```

Delete EKS:

```bash
eksctl delete cluster \
  --name "$CLUSTER_NAME" \
  --region "$AWS_REGION"
```

Also verify that unused AWS Load Balancers, EC2 resources, EBS volumes, Elastic IPs, NAT Gateways, and CloudFormation stacks are not left behind.

---

# 🧠 Skills Demonstrated

`Git` • `GitHub` • `Jenkins` • `Multibranch Pipeline` • `CI/CD` • `Docker` • `Docker Hub` • `Kubernetes` • `AWS EKS` • `Argo CD` • `GitOps` • `Helm` • `Prometheus` • `Grafana` • `Linux` • `Python` • `Flask`

---

# 👨‍💻 Author

## Rahmanuddin MD

**Engineering Automation | AI/ML | Cloud & DevOps**

GitHub: `https://github.com/rahmanuddinmd`

LinkedIn: `https://www.linkedin.com/in/md-rahmanuddin/`

---

<div align="center">

## 🚀 Build → Automate → Containerize → Deploy → Observe → Improve

### From Source Code to Cloud-Native Delivery

</div>
