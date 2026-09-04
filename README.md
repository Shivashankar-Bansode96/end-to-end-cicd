# End-to-End CI/CD Pipeline

An end-to-end DevOps project that automates the complete application delivery lifecycle using GitHub Actions, Docker, Trivy, GitHub Container Registry, and Kubernetes.

The pipeline automatically builds, tests, scans, packages, and deploys a Flask application to a local Kubernetes cluster running on Minikube.

---

## 📌 Project Overview

This project demonstrates a complete CI/CD workflow where a code change pushed to the `main` branch automatically triggers:

1. Application checkout
2. Python dependency installation
3. Automated testing with Pytest
4. Docker image build
5. Container vulnerability scanning with Trivy
6. Docker image push to GitHub Container Registry (GHCR)
7. Kubernetes deployment using a self-hosted GitHub Actions runner
8. Rolling update of Kubernetes Pods
9. Deployment verification
10. Automatic rollback if the Kubernetes rollout fails

The objective is to eliminate manual deployment steps and demonstrate a production-style DevOps workflow.

---

## 🏗️ Architecture

```text
                    Developer
                        │
                        │ git push
                        ▼
                ┌─────────────────┐
                │     GitHub      │
                │   Repository    │
                └────────┬────────┘
                         │
                         ▼
              ┌──────────────────────┐
              │    GitHub Actions    │
              │                      │
              │  1. Checkout         │
              │  2. Install deps     │
              │  3. Run tests        │
              │  4. Docker build     │
              │  5. Trivy scan       │
              │  6. Push image       │
              └──────────┬───────────┘
                         │
                         ▼
              ┌──────────────────────┐
              │        GHCR          │
              │ GitHub Container     │
              │     Registry         │
              └──────────┬───────────┘
                         │
                         │ Docker image
                         ▼
              ┌──────────────────────┐
              │ Self-Hosted Runner   │
              │      Windows         │
              └──────────┬───────────┘
                         │
                         │ kubectl
                         ▼
              ┌──────────────────────┐
              │      Minikube        │
              │     Kubernetes       │
              ├──────────────────────┤
              │ Deployment           │
              │   ├── Pod 1          │
              │   └── Pod 2          │
              │                      │
              │ Service              │
              └──────────────────────┘

🛠️ Technologies Used
| Technology     | Purpose                               |
| -------------- | ------------------------------------- |
| Python         | Application development               |
| Flask          | Web application framework             |
| Pytest         | Automated testing                     |
| Git            | Version control                       |
| GitHub         | Source code repository                |
| GitHub Actions | CI/CD automation                      |
| Docker         | Application containerization          |
| Trivy          | Container vulnerability scanning      |
| GHCR           | Container image registry              |
| Kubernetes     | Container orchestration               |
| Minikube       | Local Kubernetes cluster              |
| kubectl        | Kubernetes CLI                        |
| PowerShell     | Windows automation / deployment shell |

📂 Project Structure
end-to-end-cicd/
│
├── .github/
│   └── workflows/
│       └── ci-cd.yml
│
├── app/
│   ├── app.py
│   └── test_app.py
│
├── k8s/
│   ├── deployment.yaml
│   └── service.yaml
│
├── .dockerignore
├── .gitignore
├── Dockerfile
├── requirements.txt
└── README.md

🚀 Application

The project contains a simple Flask application with two endpoints.

Application endpoint
/

Returns:

Hello from End-to-End CI/CD Pipeline!
Health endpoint
/health

Returns:

OK

The /health endpoint is used by Kubernetes readiness and liveness probes.

🧪 Automated Testing

Tests are written using Pytest.

The test suite verifies:

HTTP status code
Application response
Health endpoint response

Run tests locally:

pytest

Expected result:

2 passed

🐳 Docker

The Flask application is packaged into a Docker image.

Build the image locally:

docker build -t cicd-demo:1.0 .

Run the container:

docker run -d --name cicd-demo-container -p 5000:5000 cicd-demo:1.0

Application:

http://localhost:5000

Health endpoint:

http://localhost:5000/health

🔄 CI Pipeline

The CI pipeline runs on GitHub-hosted Ubuntu runners.

The pipeline performs the following steps:

Checkout
   ↓
Setup Python 3.12
   ↓
Install dependencies
   ↓
Run Pytest
   ↓
Login to GHCR
   ↓
Build Docker image
   ↓
Trivy vulnerability scan
   ↓
Push image to GHCR

🔐 Container Security Scanning

Trivy is used to scan the Docker image for vulnerabilities.

The pipeline checks for:

HIGH
CRITICAL

severity vulnerabilities.

Unfixed vulnerabilities are ignored:

ignore-unfixed: true

The pipeline fails when matching vulnerabilities are detected.

This prevents vulnerable container images from being automatically promoted to deployment.

📦 GitHub Container Registry

Docker images are published to GitHub Container Registry (GHCR).

Repository:

ghcr.io/shivashankar-bansode96/cicd-demo

Images are tagged using Git commit SHAs.

Example:

ghcr.io/shivashankar-bansode96/cicd-demo:2be7eddaf4e49141476d96cede620d477e76e889

The GHCR package is public so that the project image can be inspected and pulled without registry credentials.

☸️ Kubernetes Deployment

The application is deployed to a local Minikube Kubernetes cluster.

The Deployment runs:

2 replicas

This provides two application Pods instead of relying on a single Pod.

The Kubernetes Deployment includes:

Rolling updates
Replica management
Resource requests
Resource limits
Readiness probe
Liveness probe
Deployment revision history
Security context

🔒 Kubernetes Security

The container is configured to run as a non-root user.

The Kubernetes security configuration includes:

runAsNonRoot: true
runAsUser: 1000
runAsGroup: 1000
fsGroup: 1000

Container-level security settings include:

allowPrivilegeEscalation: false

All Linux capabilities are dropped:

capabilities:
  drop:
    - ALL

The running container was verified using:

kubectl exec -it <pod-name> -- id

Expected result:

uid=1000(appuser)

This confirms that the application is not running as root.

🩺 Kubernetes Health Probes

The application exposes:

/health

Kubernetes uses this endpoint for:

Readiness Probe

Determines whether a Pod is ready to receive traffic.

readinessProbe:
  httpGet:
    path: /health
    port: 5000
Liveness Probe

Determines whether the application is still running correctly.

livenessProbe:
  httpGet:
    path: /health
    port: 5000

💻 Resource Management

The Kubernetes Deployment defines CPU and memory requests and limits.

Requests
CPU:    100m
Memory: 128Mi
Limits
CPU:    500m
Memory: 512Mi

This demonstrates basic Kubernetes resource management.

🔁 Continuous Deployment

The CD job runs on a self-hosted Windows GitHub Actions runner.

The runner has access to the local Minikube cluster through kubectl.

The deployment process is:

GitHub Actions
      ↓
Self-hosted Runner
      ↓
kubectl apply
      ↓
Apply Kubernetes configuration
      ↓
kubectl set image
      ↓
Deploy Git SHA image
      ↓
kubectl rollout status
      ↓
Verify deployment

🔄 Rolling Deployment

Kubernetes Deployment performs a rolling update when a new image is deployed.

For example:

Old Pods
  ├── Pod 1
  └── Pod 2

      ↓ New image

New Pods
  ├── Pod 1
  └── Pod 2

The old Pods are terminated after the new Pods become available.

This allows the application to transition between versions without manually deleting Pods.

↩️ Automatic Rollback

The deployment pipeline monitors the Kubernetes rollout.

The workflow runs:

kubectl rollout status deployment/cicd-demo --timeout=120s

If the rollout fails, the pipeline automatically executes:

kubectl rollout undo deployment/cicd-demo

The previous working version is then restored.

Conceptually:

New deployment
      ↓
Rollout
      ↓
 ┌───────────────┐
 │ Successful?   │
 └───────┬───────┘
         │
    ┌────┴────┐
   YES        NO
    │          │
    ▼          ▼
 Continue   Rollback
               │
               ▼
        Previous version

🏷️ Deployment Version Tracking

Each deployment is associated with the Git commit SHA.

Example:

Deploy 2be7eddaf4e49141476d96cede620d477e76e889

Deployment history can be inspected using:

kubectl rollout history deployment/cicd-demo

A specific revision can be inspected using:

kubectl rollout history deployment/cicd-demo --revision=3

⚙️ GitHub Actions Workflow

The workflow is triggered by pushes to the main branch.

on:
  push:
    branches:
      - main

Pull requests also trigger the CI portion:

pull_request:
  branches:
    - main

The deployment job is restricted to pushes to main.

This prevents pull requests from automatically deploying to the Kubernetes environment.

🖥️ Running Kubernetes Locally

Start Minikube:

minikube start --driver=docker

Verify the cluster:

kubectl get nodes

Expected:

NAME       STATUS   ROLES
minikube   Ready    control-plane

Apply Kubernetes resources manually if required for local development:

kubectl apply -f k8s/deployment.yaml
kubectl apply -f k8s/service.yaml

Check the Deployment:

kubectl get deployment cicd-demo

Check Pods:

kubectl get pods -l app=cicd-demo

Check Service:

kubectl get service cicd-demo-service

🌐 Accessing the Application

The Kubernetes Service uses:

NodePort

Access the service through Minikube:

minikube service cicd-demo-service

🔍 Useful Kubernetes Commands

Check Pods:

kubectl get pods

Check Deployment:

kubectl get deployment cicd-demo

Check Service:

kubectl get service cicd-demo-service

Check deployment image:

kubectl get deployment cicd-demo \
  -o=jsonpath='{.spec.template.spec.containers[0].image}'

Check rollout status:

kubectl rollout status deployment/cicd-demo

Check rollout history:

kubectl rollout history deployment/cicd-demo

Rollback:

kubectl rollout undo deployment/cicd-demo

📈 CI/CD Flow Summary
Developer
    │
    │ git push
    ▼
GitHub
    │
    ▼
GitHub Actions
    │
    ├── Test
    │
    ├── Docker Build
    │
    ├── Trivy Scan
    │
    └── Push to GHCR
             │
             ▼
      Self-hosted Runner
             │
             ▼
          Kubernetes
             │
             ├── Apply manifests
             │
             ├── Deploy SHA image
             │
             ├── Rolling update
             │
             ├── Verify rollout
             │
             └── Rollback on failure

🎯 Key DevOps Concepts Demonstrated

This project demonstrates practical knowledge of:

CI/CD
Git-based workflows
Automated testing
Containerization
Docker image versioning
Container vulnerability scanning
Container registries
Kubernetes Deployments
Kubernetes Services
Kubernetes health probes
Kubernetes resource management
Kubernetes security contexts
Non-root containers
Rolling deployments
Deployment history
Automated rollback
GitHub Actions self-hosted runners
Infrastructure automation

🚧 Future Improvements

Possible future enhancements include:

Deploying Kubernetes to AWS EKS
Using Terraform for infrastructure provisioning
Adding Helm charts
Adding Prometheus and Grafana
Adding centralized logging
Adding Kubernetes namespaces
Adding NetworkPolicies
Adding GitHub Environments
Adding deployment approvals for production
Adding automated integration tests
Adding container image signing
Adding dependency scanning
Migrating the self-hosted runner to a dedicated infrastructure

👨‍💻 Author

Shivashankar Bansode

DevOps / Cloud Engineering Portfolio Project

Technologies:

AWS
Linux
Docker
Kubernetes
GitHub Actions
Terraform
Python
Trivy
CI/CD

⭐ Project Goal

The primary goal of this project is to demonstrate how application delivery can be automated from source code commit to Kubernetes deployment while incorporating automated testing, security scanning, immutable container versions, deployment verification, and rollback capabilities.
        
    

              
