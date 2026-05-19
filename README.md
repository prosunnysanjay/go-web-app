# Go Web Application

This is a simple website written in Golang. It uses the `net/http` package to serve HTTP requests.

## Running the server

To run the server, execute the following command:

```bash
go run main.go
```

The server will start on port 8080. You can access it by navigating to `http://localhost:8080/courses` in your web browser.

## Looks like this

![Website](static/images/golang-website.png)


# Go Web App - DevOps Project

## Overview

This project demonstrates end-to-end DevOps implementation for a Go web application using:

- Docker
- Kubernetes
- AKS (Azure Kubernetes Service)
- Helm
- GitHub Actions
- ArgoCD
- NGINX Ingress Controller

The application was containerized and deployed into AKS using GitOps methodology.


## Project Architecture

```
Developer
   ↓
GitHub Repository
   ↓
GitHub Actions (CI)
   ↓
Docker Image Build
   ↓
Docker Hub
   ↓
Helm Chart Update
   ↓
ArgoCD Watches Repository
   ↓
AKS Cluster Deployment
   ↓
NGINX Ingress Controller
   ↓
DNS Mapping
   ↓
Users Access Application
```


# Prerequisites

Before starting the project, install the following tools in your local system:

- Git
- Visual Studio Code
- Docker Desktop
- Kubernetes CLI (kubectl)
- Helm
- Azure CLI
- Go Language
- GitHub Account
- Docker Hub Account
- Microsoft Azure Account

# Start with
 Step 1 - Fork and Run Application Locally

### Fork Repository
Fork the existing Go web application repository into your GitHub account.

### Clone Repository into Local System
```bash
git clone <repository-url>
```
Run it locally (go run . and access it in browser: http://localhost:8080/courses)



Step2: - Docker Setup and Containerization
### Create Dockerfile
Create a multi-stage Dockerfile in the project root directory.

### Install Docker Desktop
Install and start Docker Desktop in local system.

### Build Docker Image
```bash
docker build -t sanjaysunny296/go-web-app:v1 .
```

### Run Docker Container
```bash
docker run -p 8080:8080 -it sanjaysunny296/go-web-app:v1
```

Access application in browser:
```text
http://localhost:8080/courses
```

### Login into DockerHub
```bash
docker login
```

### Push Docker Image into DockerHub
```bash
docker push sanjaysunny296/go-web-app:v1
```

Step 3: Kubernetes Deployment and AKS Setup

### Create Kubernetes Manifest Files
Inside project folder create:
kubernetes/manifests folder -> files: deployment/service/ingress.yaml
These manifest files are used for:

- Deployment → Application deployment
- Service → Internal service exposure
- Ingress → External access routing

### Login into Azure
```bash
az login
```

### Create AKS Cluster
AKS cluster can be created using:

- Azure CLI
- Terraform
- Azure Portal

Example using Azure CLI:

```bash
az aks create --resource-group demo-rg --name demo-cluster --node-count 1 --generate-ssh-keys
```
### Configure kubectl Access
```bash
az aks get-credentials --resource-group demo-rg --name demo-cluster
```

Verify cluster connection:
```bash
kubectl get nodes
```

### Deploy Kubernetes Manifests
```bash
kubectl apply -f kubernetes/manisfests
```
Verify resources:
```bash
kubectl get pods
kubectl get svc
kubectl get ingress
```

## NGINX Ingress Controller Setup
Initially, ingress resource will not get external IP because ingress controller is not installed.
Install NGINX Ingress Controller:
```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.11.1/deploy/static/provider/cloud/deploy.yaml
```

Verify ingress controller pods:
```bash
kubectl get pods -n ingress-nginx
```

## Verify Ingress External IP
```bash
kubectl get ingress
```

After ingress controller installation:
- External IP will be assigned
- DNS name can be mapped to ingress IP

## DNS Mapping

Verify DNS resolution:

```bash
nslookup <dns-name>
```

If DNS is not configured properly in local system:
Linux:
```bash
sudo vi /etc/hosts
```

Windows:
```text
C:\Windows\System32\drivers\etc\hosts
```
Add entry:
```text
<EXTERNAL-IP> go-web-app.local

Notes:ingress resource itself does NOT create traffic handling
ingress controller actually processes ingress rules
NGINX ingress controller creates LoadBalancer service
LoadBalancer gets external IP


Step 4: - Helm Setup and Deployment

### Install Helm
Windows:
```bash
winget install Helm.Helm
```

Verify installation:
```bash
helm version
```

## Create Helm Chart
Generate Helm chart structure:

```bash
helm create go-web-app-chart
```

This command automatically creates:
go-web-app-chart/
├── charts/
├── templates/
├── values.yaml
├── Chart.yaml
└── charts/
```

## Modify Helm Templates

Default Helm templates were removed and replaced with custom Kubernetes manifests:

- deployment.yaml
- service.yaml
- ingress.yaml

These files were copied into:
go-web-app-chart/templates/



## Parameterize Docker Image Details
Instead of hardcoding image values directly inside deployment manifest:
Docker image repository and tag were managed dynamically using:

```yaml
values.yaml
```

Example:
```yaml
image:
  repository: sanjaysunny296/go-web-app
  tag: latest
```

Deployment template reads values dynamically using:
```yaml
{{ .Values.image.repository }}
{{ .Values.image.tag }}
```

This allows reusable deployments across multiple environments.


## Remove Existing Kubernetes Deployment
Delete previously deployed Kubernetes resources:
kubectl delete -f kubernetes/



## Deploy Application using Helm
Install Helm chart:

```bash
helm install go-web-app ./go-web-app-chart
```

Verify Helm release:
```bash
helm list
```

Verify Kubernetes resources:

```bash
kubectl get pods
kubectl get svc
kubectl get ingress
```

## Upgrade Deployment using Helm
Whenever values.yaml or templates are modified:
```bash
helm upgrade go-web-app ./go-web-app-chart
```

## Uninstall Helm Release
helm uninstall go-web-app

## Notes: GitHub Actions automatically updates values.yaml
## ArgoCD detects changes
## deployment happens automatically

## This is the bridge between:

## CI pipeline
## GitOps deployment

Step 5: CI/CD
    
    # CI/CD Pipeline using GitHub Actions

GitHub Actions is used to implement the Continuous Integration (CI) pipeline for the Go web application.

The workflow automatically triggers whenever code is pushed into the `main` branch.

The pipeline performs the following activities:

## CI/CD Workflow Trigger
The workflow runs on:

- push to main branch

The workflow ignores changes made only to:

- Helm charts
- Kubernetes manifests
- README file

```yaml
paths-ignore:
  - 'helm/**'
  - 'k8s/**'
  - 'README.md'
```

This prevents unnecessary CI pipeline execution when only deployment configuration files are modified.

---

## Build Stage

In the build stage:

- Repository code is checked out
- Go language is installed
- Application binary is generated
- Unit testing is executed

```bash
go build -o go-web-app
go test ./...
```

This stage validates that the application builds successfully and passes unit tests.

---

## Code Quality Stage

The pipeline performs static code analysis using:

- golangci-lint

This helps identify:

- code quality issues
- linting problems
- potential coding standard violations

---

## Docker Build and Push Stage

In this stage:

- Docker Buildx is configured
- DockerHub authentication is performed using GitHub Secrets
- Docker image is built
- Docker image is pushed to DockerHub

```bash
<dockerhub-username>/go-web-app:${{github.run_id}}
```

The image tag uses:

```yaml
github.run_id
```

This ensures every pipeline execution generates a unique image tag.

---

## Automatic Helm Chart Update

After the Docker image is pushed successfully:

- Helm values.yaml file is updated automatically
- New Docker image tag is replaced dynamically

```yaml
tag: "${{github.run_id}}"
```

The updated Helm chart is then committed and pushed back into the GitHub repository.

This enables GitOps-based deployment automation.

---

# GitOps Continuous Delivery using ArgoCD

ArgoCD is used for Continuous Delivery (CD).

ArgoCD continuously watches the Helm chart repository.

Whenever the Helm chart tag is updated:

- ArgoCD detects changes automatically
- Pulls the latest Helm chart
- Deploys updated application into AKS cluster

This creates a fully automated GitOps deployment workflow.

---

# CI/CD Flow

```text
Developer Pushes Code
        ↓
GitHub Actions Triggered
        ↓
Build Application
        ↓
Run Unit Tests
        ↓
Run Code Quality Checks
        ↓
Build Docker Image
        ↓
Push Image to DockerHub
        ↓
Update Helm values.yaml
        ↓
Push Updated Helm Chart
        ↓
ArgoCD Detects Change
        ↓
Deploys into AKS Cluster
```