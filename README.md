# KFC Static Site — DevOps Capstone Project

A static website deployed through a complete, automated DevOps pipeline: **GitHub → Jenkins → Docker Hub → Ansible → Kubernetes**, exposed to the internet via Minikube tunnel + Ngrok.

## Objective

Deploy a static KFC-themed website using an end-to-end CI/CD pipeline, demonstrating source control, containerization, build automation, configuration management, and container orchestration working together.

## Architecture
Developer push → GitHub repo
↓
Jenkins pipeline (Checkout → Build → Push)
↓
Docker Hub (image registry)
↓
Ansible playbook (SSH deploy trigger)
↓
Kubernetes on Minikube (Deployment + Service)
↓
Minikube tunnel + Ngrok (public exposure)

## Infrastructure

Two CentOS 10 VMs, separating build responsibilities from runtime responsibilities:

- **Jenkins Controller** — runs Jenkins and Ansible. Builds the Docker image and orchestrates deployment.
- **Deployment-VM** — runs Docker and Minikube. Hosts the actual running application.

## Components

### 1. Source code
`index.html`, `style.css` — the static site content.

### 2. Dockerfile
Builds the site into an Nginx-served image (`nginx:1.27-alpine` base). Includes a custom `nginx.conf` (gzip, security headers), `.dockerignore`, and a `HEALTHCHECK`.

### 3. Jenkinsfile
Declarative pipeline with four stages:
- **Checkout** — pulls latest code from GitHub
- **Build Image** — `docker build`
- **Push to Docker Hub** — pushes `abdullahamjad2129/kfc-static:latest`
- **Deploy** — runs the Ansible playbook

### 4. Ansible (`deploy.yml`)
Runs against Deployment-VM over SSH:
1. Pulls the latest image from Docker Hub
2. Applies the Kubernetes Deployment manifest
3. Applies the Kubernetes Service manifest
4. Restarts the deployment to force pods to pick up the new image

### 5. Kubernetes manifests
- `deployment.yaml` — runs 2 replica pods of the container
- `service.yaml` — `LoadBalancer` type, exposes port 80, gives pods a stable address

## How it works end to end

1. Code is pushed to GitHub.
2. Jenkins pipeline is triggered manually (`Build Now`).
3. Jenkins checks out the code, builds the Docker image, and pushes it to Docker Hub.
4. Jenkins runs the Ansible playbook, which SSHes into Deployment-VM.
5. Ansible pulls the new image and applies/restarts the Kubernetes resources.
6. Kubernetes runs 2 healthy pods behind a LoadBalancer Service.
7. `minikube tunnel` assigns the Service a reachable IP; Ngrok tunnels that out to a public URL.

## Setup summary

- SSH key-based passwordless authentication between Jenkins Controller and Deployment-VM
- Docker installed on both VMs (Jenkins Controller needs it to build; Deployment-VM needs it to run Minikube's driver)
- Jenkins Docker Hub credentials stored securely (ID: `dockerhub-creds`)
- Ansible inventory (`inventory.ini`) pointing at Deployment-VM
- Minikube cluster running with the `docker` driver, under a dedicated non-root user

## Known limitations / production considerations

- **Image tagging**: currently pushes only `:latest`, which is overwritten on every build — no rollback path exists. A production setup would tag with the Jenkins build number as well.
- **No automated tests**: the pipeline builds and ships without verifying the app actually runs first (a smoke test stage would fix this).
- **Manual trigger**: the pipeline currently runs on manual `Build Now` clicks rather than an automatic trigger (Poll SCM or a GitHub webhook).
- **Root usage**: for simplicity, root is used across both VMs; production practice would use dedicated least-privilege service accounts.

## Author

Abdullah — DevOps Internship Capstone Project
