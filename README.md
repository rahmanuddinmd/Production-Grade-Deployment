<div align="center">

# 🚀 ShopEasy — Production-Style CI/CD & GitOps Deployment on AWS EKS

### Jenkins • Docker • GitHub • Kubernetes • AWS EKS • Argo CD • Prometheus • Grafana

[![Python](https://img.shields.io/badge/Python-Flask-3776AB?logo=python&logoColor=white)](https://www.python.org/)
[![Jenkins](https://img.shields.io/badge/Jenkins-CI%2FCD-D24939?logo=jenkins&logoColor=white)](https://www.jenkins.io/)
[![Docker](https://img.shields.io/badge/Docker-Containerization-2496ED?logo=docker&logoColor=white)](https://www.docker.com/)
[![Kubernetes](https://img.shields.io/badge/Kubernetes-AWS%20EKS-326CE5?logo=kubernetes&logoColor=white)](https://kubernetes.io/)
[![Argo CD](https://img.shields.io/badge/Argo%20CD-GitOps-EF7B4D?logo=argo&logoColor=white)](https://argo-cd.readthedocs.io/)
[![Prometheus](https://img.shields.io/badge/Prometheus-Monitoring-E6522C?logo=prometheus&logoColor=white)](https://prometheus.io/)
[![Grafana](https://img.shields.io/badge/Grafana-Observability-F46800?logo=grafana&logoColor=white)](https://grafana.com/)

**Build → Automate → Containerize → Deploy → Monitor**

</div>

---

![ShopEasy DevOps Overview](assets/devops-project-overview.png)

> The image above is optional. The architecture diagrams below use **Mermaid**, which GitHub renders directly inside the README.

---

# 📌 Project Overview

This project demonstrates a complete **cloud-native DevOps delivery workflow** for a Python Flask e-commerce application named **ShopEasy**.

The application itself is simple by design. The real focus is the DevOps lifecycle around it:

```text
Developer
   ↓
GitHub
   ↓
Jenkins Multibranch Pipeline
   ↓
Docker Build
   ↓
Docker Registry
   ↓
Kubernetes Manifest Update
   ↓
Git
   ↓
Argo CD
   ↓
AWS EKS
   ↓
Prometheus + Grafana
```

The project demonstrates how source code moves from development to a live Kubernetes environment while keeping the deployment state version-controlled and observable.

---

# 📚 Table of Contents

- [Architecture](#-architecture)
- [End-to-End CI/CD Pipeline](#-end-to-end-cicd-pipeline)
- [GitOps Flow](#-gitops-flow)
- [Kubernetes Architecture](#-kubernetes-architecture)
- [Monitoring Architecture](#-monitoring-architecture)
- [Branching Strategy](#-branching-strategy)
- [Technology Stack](#-technology-stack)
- [Repository Structure](#-repository-structure)
- [Prerequisites](#-prerequisites)
- [Step 1 — Clone the Repository](#1️⃣-step-1--clone-the-repository)
- [Step 2 — Run the Application Locally](#2️⃣-step-2--run-the-application-locally)
- [Step 3 — Build with Docker](#3️⃣-step-3--build-with-docker)
- [Step 4 — Jenkins Setup](#4️⃣-step-4--jenkins-setup)
- [Step 5 — Install Docker for Jenkins](#5️⃣-step-5--install-docker-for-jenkins)
- [Step 6 — AWS CLI, kubectl, eksctl and Helm](#6️⃣-step-6--aws-cli-kubectl-eksctl-and-helm)
- [Step 7 — Create AWS EKS](#7️⃣-step-7--create-aws-eks)
- [Step 8 — Deploy Kubernetes Manifests](#8️⃣-step-8--deploy-kubernetes-manifests)
- [Step 9 — Configure Jenkins Multibranch](#9️⃣-step-9--configure-jenkins-multibranch)
- [Step 10 — Jenkins Pipeline Code](#-step-10--jenkins-pipeline-code)
- [Step 11 — Install Argo CD](#1️⃣1️⃣-step-11--install-argo-cd)
- [Step 12 — Configure GitOps](#1️⃣2️⃣-step-12--configure-gitops)
- [Step 13 — Monitoring](#1️⃣3️⃣-step-13--monitoring)
- [Validation](#-validation-checklist)
- [Troubleshooting](#-troubleshooting)
- [Production Improvements](#-production-improvements)
- [Cleanup](#-cleanup)

---

# 🏗 Architecture

```mermaid
flowchart LR
    DEV["Developer"]
    GH["GitHub Repository"]
    J["Jenkins Multibranch CI"]
    D["Docker Build"]
    REG["Docker Registry"]
    GIT["GitOps Manifest"]
    ARGO["Argo CD"]
    EKS["AWS EKS"]
    LB["LoadBalancer"]
    APP["ShopEasy Flask App"]
    PROM["Prometheus"]
    GRAF["Grafana"]

    DEV -->|"push / PR"| GH
    GH -->|"trigger / scan"| J
    J --> D
    D -->|"push image"| REG
    J -->|"update image tag"| GIT
    GIT -->|"watch"| ARGO
    ARGO -->|"sync"| EKS
    REG -->|"pull image"| EKS
    EKS --> LB
    LB --> APP
    EKS --> PROM
    PROM --> GRAF
```

## Architecture explanation

### 1. Developer
The developer changes application code on a feature branch.

### 2. GitHub
GitHub stores the source code, branches, pull requests, Jenkinsfile, Kubernetes manifests, and Argo CD configuration.

### 3. Jenkins
Jenkins performs Continuous Integration. It checks out the code, builds a Docker image, tags and pushes the image, updates the Kubernetes manifest, and commits the new desired state.

### 4. Docker Registry
The registry stores immutable application versions such as:

```text
yourdockeruser/shopeasy-app:build-101
yourdockeruser/shopeasy-app:build-102
yourdockeruser/shopeasy-app:build-103
```

### 5. Argo CD
Argo CD monitors Git and synchronizes Kubernetes with the desired state stored in the repository.

### 6. AWS EKS
Amazon EKS hosts the Kubernetes workload.

### 7. Prometheus and Grafana
Prometheus collects metrics, while Grafana provides dashboards for infrastructure and workload visibility.

---

# 🔁 End-to-End CI/CD Pipeline

```mermaid
flowchart TD
    A["1. Developer Changes Code"]
    B["2. Feature Branch"]
    C["3. Push to GitHub"]
    D["4. Jenkins Detects Branch"]
    E{"Main Branch?"}
    F["Branch Validation"]
    G["5. Build Docker Image"]
    H["6. Tag Image build-N"]
    I["7. Push Image"]
    J["8. Update deployment.yaml"]
    K["9. Commit Manifest"]
    L["10. Push Git Change"]
    M["11. Argo CD Detects Change"]
    N["12. Sync to AWS EKS"]
    O["13. Kubernetes Rolling Update"]
    P["14. Application Live"]
    Q["15. Prometheus Metrics"]
    R["16. Grafana Dashboard"]

    A --> B --> C --> D --> E
    E -- No --> F
    E -- Yes --> G --> H --> I --> J --> K --> L --> M --> N --> O --> P --> Q --> R
```

## Pipeline stages

| Stage | Tool | Purpose |
|---|---|---|
| Source Change | Git | Track code modifications |
| Branch | GitHub | Isolate feature development |
| CI Trigger | Jenkins | Start automation |
| Build | Docker | Package Flask app |
| Image Version | Jenkins | Generate immutable build tag |
| Registry Push | Docker Hub | Store container image |
| Manifest Update | Git | Change desired image version |
| GitOps Detection | Argo CD | Detect manifest commit |
| Deployment | AWS EKS | Run new application version |
| Monitoring | Prometheus | Collect cluster metrics |
| Visualization | Grafana | Display dashboards |

---

# 🔄 GitOps Flow

```mermaid
flowchart LR
    CI["Jenkins"]
    GIT["Git Repository<br/>Desired State"]
    ARGO["Argo CD"]
    CLUSTER["AWS EKS<br/>Actual State"]

    CI -->|"updates deployment.yaml"| GIT
    GIT -->|"watched by"| ARGO
    ARGO -->|"compare"| CLUSTER
    CLUSTER -. "live state" .-> ARGO
    ARGO -->|"sync / self-heal"| CLUSTER
```

Instead of:

```text
Jenkins → kubectl apply → Kubernetes
```

the project follows:

```text
Jenkins → Git → Argo CD → Kubernetes
```

This makes Git the **source of truth** for deployment configuration.

---

# ☸ Kubernetes Architecture

```mermaid
flowchart TD
    USER["Internet / User"]
    LB["Kubernetes Service<br/>Type: LoadBalancer<br/>Port 80"]
    DEP["Deployment<br/>Replicas: 5"]

    P1["Pod 1<br/>Flask :5000"]
    P2["Pod 2<br/>Flask :5000"]
    P3["Pod 3<br/>Flask :5000"]
    P4["Pod 4<br/>Flask :5000"]
    P5["Pod 5<br/>Flask :5000"]

    USER --> LB
    LB --> P1
    LB --> P2
    LB --> P3
    LB --> P4
    LB --> P5

    DEP --> P1
    DEP --> P2
    DEP --> P3
    DEP --> P4
    DEP --> P5
```

Request path:

```text
Browser
   ↓
AWS Load Balancer
   ↓
Kubernetes Service :80
   ↓
Pod :5000
   ↓
Flask Application
```

---

# 📊 Monitoring Architecture

```mermaid
flowchart LR
    NODE["EKS Worker Nodes"]
    PODS["Kubernetes Pods"]
    NODEEXP["Node Exporter"]
    KSM["kube-state-metrics"]
    PROM["Prometheus"]
    ALERT["Alertmanager"]
    GRAF["Grafana"]
    ENG["DevOps Engineer"]

    NODE --> NODEEXP
    PODS --> KSM
    NODEEXP --> PROM
    KSM --> PROM
    PROM --> ALERT
    PROM --> GRAF
    GRAF --> ENG
```

| Component | Responsibility |
|---|---|
| Node Exporter | Node CPU, memory, filesystem and OS metrics |
| kube-state-metrics | Kubernetes object metrics |
| Prometheus | Metrics collection and storage |
| Alertmanager | Alert routing |
| Grafana | Dashboards and visualization |

---

# 🌿 Branching Strategy

```mermaid
flowchart LR
    MAIN["main"]
    A["feature/wishlist"]
    B["feature/reviews"]
    C["feature/login"]
    PR["Pull Request + Review"]
    MERGE["Merge to main"]
    CI["Production CI/CD"]

    MAIN --> A
    MAIN --> B
    MAIN --> C

    A --> PR
    B --> PR
    C --> PR

    PR --> MERGE
    MERGE --> CI
```

Create a feature branch:

```bash
git checkout -b feature/wishlist
git add .
git commit -m "feat: add wishlist functionality"
git push -u origin feature/wishlist
```

Then:

```text
feature/wishlist
        ↓
Pull Request
        ↓
Code Review
        ↓
Merge to main
        ↓
Jenkins Production Pipeline
```

---

# 🧰 Technology Stack

| Technology | Role |
|---|---|
| Python | Application language |
| Flask | Web application framework |
| Git | Version control |
| GitHub | Source repository and PR workflow |
| Jenkins | CI automation |
| Docker | Containerization |
| Docker Hub | Image registry |
| Kubernetes | Container orchestration |
| AWS EKS | Managed Kubernetes |
| kubectl | Kubernetes CLI |
| eksctl | EKS lifecycle management |
| Helm | Kubernetes package manager |
| Argo CD | GitOps Continuous Delivery |
| Prometheus | Metrics collection |
| Grafana | Monitoring dashboards |
| Linux | Server environment |

---

# 📂 Repository Structure

```text
shopeasy-devops/
│
├── app.py
├── requirements.txt
├── Dockerfile
├── Jenkinsfile
├── README.md
│
├── assets/
│   └── devops-project-overview.png
│
├── k8s/
│   ├── deployment.yaml
│   └── service.yaml
│
└── argocd/
    └── application.yaml
```

Use actual Git branches for feature work rather than long-term folders that imitate branches.

---

# ✅ Prerequisites

- AWS account
- GitHub account
- Docker Hub account
- Ubuntu/Linux Jenkins server
- IAM permissions for EKS
- Git
- Basic Linux knowledge

---

# 1️⃣ Step 1 — Clone the Repository

```bash
git clone https://github.com/<YOUR_USERNAME>/<YOUR_REPOSITORY>.git
cd <YOUR_REPOSITORY>
```

Verify:

```bash
git status
git branch -a
```

---

# 2️⃣ Step 2 — Run the Application Locally

Create a virtual environment:

```bash
python3 -m venv venv
source venv/bin/activate
```

Windows:

```powershell
venv\Scripts\Activate.ps1
```

Install packages:

```bash
pip install -r requirements.txt
```

Run:

```bash
python app.py
```

Test:

```bash
curl http://localhost:5000
```

---

# 3️⃣ Step 3 — Build with Docker

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

| Instruction | Purpose |
|---|---|
| `FROM` | Python base image |
| `WORKDIR` | Application working directory |
| `COPY requirements.txt` | Dependency file |
| `RUN pip install` | Install Python packages |
| `COPY app.py` | Copy application |
| `EXPOSE 5000` | Document application port |
| `CMD` | Start Flask |

Build:

```bash
docker build -t shopeasy-app:local .
```

Run:

```bash
docker run -d \
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

---

# 4️⃣ Step 4 — Jenkins Setup

```bash
sudo apt update
sudo apt upgrade -y
sudo apt install -y fontconfig openjdk-21-jre
java -version
```

Install Jenkins:

```bash
sudo mkdir -p /etc/apt/keyrings

sudo wget -O /etc/apt/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian/jenkins.io-2026.key

echo "deb [signed-by=/etc/apt/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian binary/" \
  | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null

sudo apt update
sudo apt install -y jenkins
sudo systemctl enable jenkins
sudo systemctl start jenkins
```

Get password:

```bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

Open:

```text
http://<JENKINS_SERVER_IP>:8080
```

---

# 5️⃣ Step 5 — Install Docker for Jenkins

```bash
sudo apt update
sudo apt install -y ca-certificates curl

sudo install -m 0755 -d /etc/apt/keyrings

sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg \
  -o /etc/apt/keyrings/docker.asc

sudo chmod a+r /etc/apt/keyrings/docker.asc
```

Repository:

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

Install:

```bash
sudo apt update

sudo apt install -y \
  docker-ce \
  docker-ce-cli \
  containerd.io \
  docker-buildx-plugin \
  docker-compose-plugin
```

Allow Jenkins:

```bash
sudo usermod -aG docker jenkins
sudo systemctl restart jenkins
```

Verify:

```bash
docker --version
sudo docker run hello-world
```

---

# 6️⃣ Step 6 — AWS CLI, kubectl, eksctl and Helm

## AWS CLI

```bash
sudo apt install -y unzip curl

curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" \
  -o "awscliv2.zip"

unzip awscliv2.zip
sudo ./aws/install
aws --version
aws configure
aws sts get-caller-identity
```

## kubectl

```bash
curl -LO \
  "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"

chmod +x kubectl
sudo mv kubectl /usr/local/bin/kubectl
kubectl version --client
```

## eksctl

```bash
curl --silent --location \
  "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" \
  | tar xz -C /tmp

sudo mv /tmp/eksctl /usr/local/bin
eksctl version
```

## Helm

```bash
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
helm version
```

---

# 7️⃣ Step 7 — Create AWS EKS

```bash
export CLUSTER_NAME="shopeasy-eks"
export AWS_REGION="us-east-1"
```

Create:

```bash
eksctl create cluster \
  --name "$CLUSTER_NAME" \
  --region "$AWS_REGION" \
  --nodegroup-name shopeasy-ng \
  --node-type t3.medium \
  --nodes 2 \
  --managed
```

Configure kubectl:

```bash
aws eks update-kubeconfig \
  --region "$AWS_REGION" \
  --name "$CLUSTER_NAME"
```

Verify:

```bash
kubectl get nodes
```

---

# 8️⃣ Step 8 — Deploy Kubernetes Manifests

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

Apply:

```bash
kubectl apply -f k8s/deployment.yaml
kubectl apply -f k8s/service.yaml
```

Check:

```bash
kubectl get deployments
kubectl get pods -o wide
kubectl get svc
```

---

# 9️⃣ Step 9 — Configure Jenkins Multibranch

Install or verify these plugins:

```text
Pipeline
Git
GitHub
GitHub Branch Source
Credentials Binding
Docker Pipeline
```

Create Docker registry credentials:

```text
ID: dockerhub-creds
```

Create GitHub credentials:

```text
ID: github-creds
```

Create the job:

```text
New Item
→ Multibranch Pipeline
→ GitHub Branch Source
→ Repository
→ Credentials
→ Save
```

---

# 🔟 Step 10 — Jenkins Pipeline Code

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
                        echo "$DOCKER_PASS" | docker login \
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
                    git config user.email "jenkins@automation.local"

                    git add k8s/deployment.yaml

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

## Pipeline logic

```text
Checkout
   ↓
Is branch main?
   ├── No → Branch validation
   │
   └── Yes
         ↓
     Build Image
         ↓
     Tag build-N
         ↓
     Push Registry
         ↓
     Update deployment.yaml
         ↓
     Git Commit
         ↓
     Git Push
         ↓
     Argo CD detects change
```

---

# 1️⃣1️⃣ Step 11 — Install Argo CD

```bash
helm repo add argo \
  https://argoproj.github.io/argo-helm

helm repo update
```

Install:

```bash
helm install argocd \
  argo/argo-cd \
  --namespace argocd \
  --create-namespace
```

Verify:

```bash
kubectl get pods -n argocd
```

Lab LoadBalancer:

```bash
kubectl patch svc argocd-server \
  -n argocd \
  -p '{"spec":{"type":"LoadBalancer"}}'
```

Get endpoint:

```bash
kubectl get svc argocd-server -n argocd
```

Password:

```bash
kubectl get secret argocd-initial-admin-secret \
  -n argocd \
  -o jsonpath="{.data.password}" \
  | base64 -d

echo
```

---

# 1️⃣2️⃣ Step 12 — Configure GitOps

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

Apply:

```bash
kubectl apply -f argocd/application.yaml
```

Check:

```bash
kubectl get applications -n argocd

kubectl describe application shopeasy-app \
  -n argocd
```

Expected:

```text
SYNC STATUS: Synced
HEALTH STATUS: Healthy
```

---

# 1️⃣3️⃣ Step 13 — Monitoring

```bash
helm repo add prometheus-community \
  https://prometheus-community.github.io/helm-charts

helm repo update
```

Install:

```bash
helm install monitoring \
  prometheus-community/kube-prometheus-stack \
  --namespace monitoring \
  --create-namespace
```

Verify:

```bash
kubectl get pods -n monitoring
```

Lab Grafana access:

```bash
kubectl patch svc monitoring-grafana \
  -n monitoring \
  -p '{"spec":{"type":"LoadBalancer"}}'
```

Get endpoint:

```bash
kubectl get svc monitoring-grafana \
  -n monitoring
```

Get password:

```bash
kubectl get secret monitoring-grafana \
  -n monitoring \
  -o jsonpath="{.data.admin-password}" \
  | base64 -d

echo
```

Username:

```text
admin
```

Recommended dashboards:

```text
Kubernetes / Cluster
Kubernetes / Nodes
Kubernetes / Pods
Kubernetes / Deployments
Node Exporter
```

---

# ✅ Validation Checklist

## Application

```bash
python app.py
curl http://localhost:5000
```

## Docker

```bash
docker build -t shopeasy-app:test .
docker run -d -p 5000:5000 shopeasy-app:test
curl http://localhost:5000
```

## EKS

```bash
kubectl get nodes
kubectl get deployments
kubectl get pods
kubectl get svc
```

## Argo CD

```bash
kubectl get applications -n argocd
```

## Monitoring

```bash
kubectl get pods -n monitoring
```

---

# 🛠 Troubleshooting

## Jenkins Docker permission error

```bash
sudo usermod -aG docker jenkins
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
kubectl describe pod <POD_NAME>

kubectl get deployment shopeasy-app \
  -o jsonpath='{.spec.template.spec.containers[0].image}'

echo
```

## `CrashLoopBackOff`

```bash
kubectl logs <POD_NAME>
kubectl logs <POD_NAME> --previous
```

## Argo CD OutOfSync

```bash
kubectl describe application shopeasy-app -n argocd

kubectl apply --dry-run=client -f k8s/
```

## kubectl cannot connect

```bash
aws eks update-kubeconfig \
  --region "$AWS_REGION" \
  --name "$CLUSTER_NAME"

aws sts get-caller-identity
```

---

# ⚠ Important Project Corrections

Before publishing:

### Use one manifest extension

```text
deployment.yaml
```

### Use one EKS cluster name

```text
shopeasy-eks
```

### Use consistent app names

```text
shopeasy-app
shopeasy-service
```

### Replace old owner-specific values

Update:

```text
Docker Hub username
GitHub username
GitHub repository
Git email
Argo CD repoURL
Jenkins push URL
```

---

# 🔐 Production Improvements

A hardened production pipeline should add:

- unit tests
- integration tests
- linting
- SonarQube/code-quality analysis
- image vulnerability scanning
- SBOM generation
- readiness probes
- liveness probes
- resource requests and limits
- Kubernetes Secrets
- external secrets manager
- TLS/HTTPS
- Ingress
- autoscaling
- centralized logging
- notification integrations
- Terraform/OpenTofu
- dev/staging/prod separation
- RBAC
- policy-as-code
- automated rollback
- deployment verification
- blue/green or canary delivery

```mermaid
flowchart LR
    A["Checkout"]
    B["Lint"]
    C["Unit Tests"]
    D["Security Scan"]
    E["Docker Build"]
    F["Image Scan"]
    G["Push"]
    H["GitOps Update"]
    I["Argo CD Sync"]
    J["Verify"]

    A --> B --> C --> D --> E --> F --> G --> H --> I --> J
```

---

# 🧹 Cleanup

Application:

```bash
kubectl delete -f argocd/application.yaml
kubectl delete -f k8s/service.yaml
kubectl delete -f k8s/deployment.yaml
```

Monitoring:

```bash
helm uninstall monitoring -n monitoring
kubectl delete namespace monitoring
```

Argo CD:

```bash
helm uninstall argocd -n argocd
kubectl delete namespace argocd
```

EKS:

```bash
eksctl delete cluster \
  --name "$CLUSTER_NAME" \
  --region "$AWS_REGION"
```

Also verify that unused billable AWS resources are removed.

---

# 🧠 Skills Demonstrated

`Git` • `GitHub` • `Jenkins` • `CI/CD` • `Docker` • `Docker Hub` • `Kubernetes` • `AWS EKS` • `Argo CD` • `GitOps` • `Helm` • `Prometheus` • `Grafana` • `Linux` • `Python` • `Flask`

---

# 👨‍💻 Author

## Rahmanuddin MD

**Engineering Automation | AI/ML | Cloud & DevOps**

GitHub: `https://github.com/rahmanuddinmd`

LinkedIn: `https://www.linkedin.com/in/md-rahmanuddin/`

---

<div align="center">

# ⭐ Build • Automate • Deploy • Observe • Improve

### From Source Code to Cloud-Native Delivery

</div>
