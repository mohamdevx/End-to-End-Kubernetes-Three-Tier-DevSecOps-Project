# DevSecOps Three‑Tier Application on Kubernetes

A full DevSecOps pipeline that builds, scans, and deploys a three‑tier application (MongoDB, Node.js backend, React frontend) on Kubernetes. The pipeline uses Jenkins, Trivy, SonarQube, OWASP Dependency‑Check, ArgoCD, Prometheus, and Grafana. Everything is cloud‑agnostic – run it locally with Minikube or on GCP, AWS, or Azure.

**Project Repository:** https://github.com/mohamdevx/End-to-End-Kubernetes-Three-Tier-DevSecOps-Project  
**Working Branch:** `project-polish`

---

## 📐 Architecture

![Architecture Diagram](https://via.placeholder.com/1000x600?text=Your+Architecture+Diagram+Here)

*Placeholder – insert your own diagram. Show the flow: Developer → GitHub → Jenkins (SonarQube, Trivy, OWASP) → Docker Build → Artifact Registry → Update K8s Manifests in Git → ArgoCD → GKE/EKS/AKS/Minikube → Prometheus/Grafana monitoring.*

---

## ⚙️ Technologies & Tools

| Category | Tools |
|----------|-------|
| **Container Runtime** | Docker |
| **Orchestration** | Kubernetes (GKE, EKS, AKS, Minikube) |
| **CI/CD** | Jenkins (Declarative Pipeline), Git |
| **Code Quality** | SonarQube |
| **Software Composition Analysis** | OWASP Dependency‑Check |
| **Container Scanning** | Trivy (Filesystem & Image) |
| **GitOps** | ArgoCD |
| **Monitoring** | Prometheus, Grafana |
| **Artifact Registry** | Google Artifact Registry (GCP) / Amazon ECR (AWS) / Azure Container Registry (Azure) / local Docker registry |
| **Ingress Controller** | NGINX Ingress |

---

## ✅ Prerequisites

- **Docker** (with Kubernetes enabled if using Docker Desktop)
- **kubectl** – configured for your cluster
- **Helm** – for installing Prometheus/Grafana
- **Git**
- **Jenkins** – running with the following plugins:
  - Pipeline
  - Git
  - SonarQube Scanner
  - OWASP Dependency‑Check
  - NodeJS
  - Docker
  - Trivy (must be installed on Jenkins host)
- **A GitHub account** – for storing the repository and triggering pipelines
- **SonarQube server** (optional, can run as Docker container)
- **ArgoCD** (installed inside Kubernetes)

---

## 🚀 Local Deployment (Minikube)

### 1. Start Minikube

```bash
minikube start --cpus=4 --memory=8192
Enable the Ingress addon:

bash
minikube addons enable ingress
2. Clone the repository
bash
git clone https://github.com/mohamdevx/End-to-End-Kubernetes-Three-Tier-DevSecOps-Project.git
cd End-to-End-Kubernetes-Three-Tier-DevSecOps-Project
git checkout project-polish
3. Deploy the Application (without CI/CD)
Create the namespace and apply all Kubernetes manifests:

bash
kubectl create namespace three-tier
kubectl apply -f Kubernetes-Manifests-file/Database/ -n three-tier
kubectl apply -f Kubernetes-Manifests-file/Backend/ -n three-tier
kubectl apply -f Kubernetes-Manifests-file/Frontend/ -n three-tier
kubectl apply -f Kubernetes-Manifests-file/Ingress/ -n three-tier
Get the application URL (Minikube):

bash
minikube service list -n three-tier
# or port-forward the frontend
kubectl port-forward svc/frontend -n three-tier 3000:3000
Now open http://localhost:3000 in your browser.

4. Local Registry (Optional for CI/CD)
If you want to run the full pipeline locally, set up a local Docker registry:

bash
docker run -d -p 5000:5000 --name registry registry:2
Update the image names in deployment.yaml files to localhost:5000/backend:tag and localhost:5000/frontend:tag, and adjust the Jenkins pipeline accordingly.

☁️ Google Cloud (GCP) Deployment
1. Create a GKE Cluster
bash
gcloud container clusters create my-cluster --zone us-central1-a --num-nodes=2
gcloud container clusters get-credentials my-cluster
2. Set Up Artifact Registry
bash
gcloud artifacts repositories create three-tier-repo --repository-format=docker --location=us-central1
3. Install NGINX Ingress Controller
bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.8.1/deploy/static/provider/cloud/deploy.yaml
4. Deploy the Application
The manifests are already configured for GCP (Artifact Registry URL). If needed, replace the image path in each deployment.yaml:

text
image: us-central1-docker.pkg.dev/PROJECT_ID/three-tier-repo/backend:tag
image: us-central1-docker.pkg.dev/PROJECT_ID/three-tier-repo/frontend:tag
Apply the manifests:

bash
kubectl create namespace three-tier
kubectl apply -f Kubernetes-Manifests-file/Database/ -n three-tier
kubectl apply -f Kubernetes-Manifests-file/Backend/ -n three-tier
kubectl apply -f Kubernetes-Manifests-file/Frontend/ -n three-tier
kubectl apply -f Kubernetes-Manifests-file/Ingress/ -n three-tier
5. Access the Application
Find the Ingress external IP:

bash
kubectl get svc ingress-nginx-controller -n ingress-nginx
Open http://EXTERNAL-IP in your browser. The Ingress file no longer requires a specific host header, so the IP works directly.

☁️ AWS / Azure (High‑Level Instructions)
AWS (EKS)
Create an EKS cluster with eksctl create cluster --name my-cluster --region us-east-1 --node-type t2.medium --nodes 2

Set up Amazon ECR repositories (backend, frontend)

Update image URLs in manifests to your ECR URIs (ACCOUNT_ID.dkr.ecr.REGION.amazonaws.com/backend:tag)

Install the AWS Load Balancer Controller or NGINX Ingress Controller

Apply all Kubernetes manifests as shown above

Azure (AKS)
Create an AKS cluster: az aks create --resource-group myRG --name myAKS --node-count 2 --generate-ssh-keys

Create an Azure Container Registry (az acr create --resource-group myRG --name myACR --sku Basic)

Update image URLs in manifests to myACR.azurecr.io/backend:tag

Install NGINX Ingress Controller (or use Application Gateway)

Apply all Kubernetes manifests

In all clouds, the manifests remain the same. Only the image registry path and Ingress class (nginx vs alb) need adjustment.

🔁 CI/CD Pipeline (Jenkins)
The pipeline files live inside Jenkins-Pipeline-Code/:

Backend Pipeline: Jenkinsfile-Backend

Frontend Pipeline: Jenkinsfile-Frontend

Stages Overview
Checkout – Clone the repository (project-polish branch)

SonarQube Analysis – (enabled/disabled as needed)

Quality Gate – Wait for SonarQube quality gate result

OWASP Dependency‑Check – (enabled/disabled; requires NVD API key)

Trivy FS Scan – Scan source code for vulnerabilities

Docker Build – Build the container image

Push to Artifact Registry – Authenticate using gcloud (GCP) and push image

Trivy Image Scan – Scan the container image

Update Manifest & Push – Replace the image tag in the Kubernetes deployment file, commit, and push to GitHub

Jenkins Credentials Required
Create the following credentials in Manage Jenkins → Credentials → System → Global:

ID	Type	Description
github-creds	Username/Password	GitHub username & personal access token (for checkout)
GITHUB_TOKEN	Secret text	GitHub personal access token (for pushing manifest updates)
GCP_PROJECT_ID	Secret text	Your GCP project ID
AR_LOCATION	Secret text	Artifact Registry region (e.g., us-central1)
AR_REPO	Secret text	Artifact Registry repo name (e.g., three-tier-repo)
sonar-token	Secret text	SonarQube authentication token
GCP_SA_KEY	Secret file	(Optional) GCP service account JSON key if not using VM default credentials
If running locally or on AWS/Azure, replace GCP‑specific variables with your cloud's equivalents.

How to Trigger a Build
In Jenkins, create a Pipeline job pointing to this repository (branch project-polish) and the respective Jenkinsfile.

Click Build Now.

Watch the console output for each stage.

🐙 GitOps with ArgoCD
Installation
bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/v2.4.7/manifests/install.yaml
Expose the Server
bash
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'
Get the admin password:

bash
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
Log in at http://EXTERNAL-IP.

Connect Your Repository
In ArgoCD UI:

Go to Settings → Repositories → Connect Repo using HTTPS

URL: https://github.com/mohamdevx/End-to-End-Kubernetes-Three-Tier-DevSecOps-Project.git

Username: mohamdevx

Password: your GitHub personal access token

Create Applications
Create four applications (one for each manifest directory):

App Name	Path
database	Kubernetes-Manifests-file/Database
backend	Kubernetes-Manifests-file/Backend
frontend	Kubernetes-Manifests-file/Frontend
ingress	Kubernetes-Manifests-file/Ingress
Repository URL: same as above

Revision: project-polish

Cluster URL: https://kubernetes.default.svc

Namespace: three-tier

Sync Policy: Manual (or enable Auto‑Sync for fully automated deployments).

After any pipeline commit updates the manifests, ArgoCD will detect the change and show the app as OutOfSync. Sync it manually, or let auto‑sync do it automatically.

📊 Monitoring with Prometheus & Grafana
Install with Helm
bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update

kubectl create namespace monitoring
helm install prometheus prometheus-community/prometheus -n monitoring
helm install grafana grafana/grafana -n monitoring
Expose Grafana
bash
kubectl patch svc grafana -n monitoring -p '{"spec": {"type": "LoadBalancer"}}'
Wait for the external IP:

bash
kubectl get svc grafana -n monitoring
Get Admin Password
bash
kubectl get secret --namespace monitoring grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo
Add Prometheus Data Source
Login at http://GRAFANA-IP

Configuration → Data Sources → Add data source → Prometheus

URL: http://prometheus-server.monitoring.svc.cluster.local:80

Save & Test

Import Kubernetes Dashboard
Import → ID: 6417 → Load → Select Prometheus data source → Import

You'll see live cluster metrics: pods, nodes, deployments, resource usage.

🎥 Demo Tips for Interviews
Run a Pipeline: Show a Jenkins build from start to finish, pointing out each security stage.

Show ArgoCD Sync: After the pipeline commits a new image tag, open ArgoCD and sync the application (or show auto‑sync).

View Monitoring: Open Grafana and walk through the Kubernetes dashboard, highlighting pod health and resource usage.

Show the App: Prove the application is live by adding/removing a task in the frontend.

Explain the Tools: Mention why each tool (Trivy, SonarQube, OWASP) was chosen and what it protects against.

Take screenshots of each step and include them in your portfolio.

🔒 Security Scanning Explained
Trivy FS Scan: Scans source code and dependencies for known vulnerabilities before building the image.

Trivy Image Scan: Scans the final Docker image for OS and application vulnerabilities.

SonarQube: Static code analysis for code quality, bugs, and code smells.

OWASP Dependency‑Check: Checks project dependencies against the National Vulnerability Database.

All three scanners are integrated into the Jenkins pipeline, but they can be selectively enabled depending on available resources (the SonarQube and OWASP stages are currently commented out to avoid memory issues on small VMs).

🌍 Environment Variables (Cloud Customization)
When moving between clouds, update these variables in the Jenkins pipeline and manifests:

Variable	Purpose	Cloud
GCP_PROJECT_ID	GCP project ID	GCP
AR_LOCATION	Artifact Registry region (e.g., us-central1)	GCP
AR_REPO	Artifact Registry repo name	GCP
AWS_ACCOUNT_ID	AWS account ID	AWS
ECR_REPO_NAME	ECR repository name	AWS
AZURE_REGISTRY_NAME	ACR registry name	Azure
The pipeline uses these to construct the full image path like us-central1-docker.pkg.dev/GCP_PROJECT_ID/AR_REPO/backend.

🧹 Cleanup / Resource Saving
When the GCP trial ends or when you're done, delete resources to avoid charges.

GCP
bash
gcloud container clusters delete my-cluster --zone us-central1-a
gcloud artifacts repositories delete three-tier-repo --location=us-central1
# Delete Jenkins VM if provisioned via Terraform/GCP console
Local
bash
minikube stop
minikube delete
All code is stored in GitHub, so you can recreate everything anytime.

❓ Troubleshooting
Issue	Solution
Jenkins OOM during SonarQube	Increase VM RAM/swap, or lower SonarQube memory (SONAR_ES_JAVA_OPTS, SONAR_CE_JAVA_OPTS, SONAR_WEB_JAVA_OPTS), or add -Dsonar.scanner.memoryCeiling=256
OWASP download takes forever	Provide NVD API key in Jenkins plugin settings
Frontend crash (OpenSSL error)	Add ENV NODE_OPTIONS=--openssl-legacy-provider in Dockerfile
Ingress returns 404	Check Ingress class (nginx) and remove host rule if using IP directly
ArgoCD apps OutOfSync	Trigger a manual sync, or enable auto‑sync in app settings
📜 License
MIT

👤 Author
Mohammad – GitHub Profile

Adapted from Aman Pathak's AWS‑based DevSecOps project.

