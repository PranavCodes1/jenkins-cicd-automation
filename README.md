# Jenkins CI/CD Automation with Kubernetes

Automated CI/CD pipeline for deploying a Flask application using **Jenkins**, **Docker**, and **Kubernetes**.

This project demonstrates a complete deployment workflow where application changes pushed to GitHub automatically trigger Jenkins to build a Docker image, push it to Docker Hub, and deploy the updated application to Kubernetes.

---

## Project Overview

This project implements an automated CI/CD pipeline with the following stages:

1. Source code checkout from GitHub
2. Docker image build
3. Docker image push to Docker Hub
4. Kubernetes deployment
5. Application exposure through Kubernetes Service

The deployment process is automated using Jenkins pipelines.

---

## Architecture

```text
Developer
   ↓
GitHub Push
   ↓
Jenkins Pipeline Trigger
   ↓
Docker Image Build
   ↓
Docker Hub Push
   ↓
Kubernetes Deployment
   ↓
Running Application
```

---

## Features

- Automated CI/CD pipeline
- GitHub integration
- Docker image build and push
- Kubernetes deployment automation
- Namespace-based deployment
- Version-based deployments
- Secure credential handling
- Containerized Flask application

---

## Tech Stack

- Jenkins
- Docker
- Kubernetes
- Flask
- GitHub

---

## Repository Structure

```text
.
├── app/
│   ├── app.py
│   └── requirements.txt
│
├── k8s/
│   ├── deployment.yaml
│   └── service.yaml
│
├── Dockerfile
├── Jenkinsfile
├── README.md
```

---

## Pipeline Workflow

### Stage 1 — Checkout

Jenkins fetches the latest source code from GitHub.

---

### Stage 2 — Build Docker Image

The application image is built automatically.

Example:

```bash
docker build -t <image>:<version> .
```

---

### Stage 3 — Push Image

The built image is pushed to Docker Hub.

Example:

```bash
docker push <image>:<version>
```

---

### Stage 4 — Deploy to Kubernetes

Jenkins deploys the updated image to Kubernetes.

Operations include:

- Namespace creation
- Deployment update
- Service creation/update

---

## Kubernetes Resources

### Deployment

Responsible for:

- Pod creation
- Application rollout
- Replica management

### Service

Responsible for:

- Internal/external application access
- Traffic routing

---

## Security

Sensitive information is intentionally excluded.

Not included in this repository:

- kubeconfig files
- cluster addresses
- Docker credentials
- Jenkins credentials
- authentication secrets
- tokens/certificates

Secrets are managed securely outside source control.

---

## Local Development

Install dependencies:

```bash
pip install -r app/requirements.txt
```

Run application:

```bash
python app.py
```

Application runs on:

```text
http://localhost:5000
```

---

## Deployment Verification

Examples:

```bash
kubectl get pods
kubectl get deployment
kubectl get svc
```

Check logs:

```bash
kubectl logs <pod-name>
```

---

## Lessons Learned

This project helped in understanding:

- Jenkins pipelines
- Docker containerization
- Kubernetes deployments
- Kubernetes services
- Namespace isolation
- CI/CD debugging
- Secure credential usage

---

## Future Improvements

- Health checks
- Automatic rollback
- Helm deployment
- Monitoring and observability
- Ingress-based routing
- Production-grade deployment strategy

---

## Author

Pranav Paralkar

GitHub:
https://github.com/PranavCodes1
