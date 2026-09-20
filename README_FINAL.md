# GitHub Actions DevSecOps Pipeline

A hands-on DevOps/DevSecOps project built around a Flask application and GitHub Actions.

The goal of this project was to move beyond a simple **build-and-deploy** pipeline and add automated quality, security, testing, container scanning, image publishing, and deployment gates before the application reaches the production server.

## 🚀 What this project does

On every push to the `main` branch, the pipeline runs a series of checks and deployment stages:

```text
Git Push
   │
   ├── Code Quality + SAST
   │     ├── Flake8
   │     └── Bandit
   │
   ├── Secrets Scan
   │     └── Gitleaks
   │
   ├── Dependency Scan
   │     └── pip-audit
   │
   ├── Dockerfile Scan
   │     └── Hadolint
   │
   ├── Tests
   │     └── Pytest
   │
   └── All required checks pass
             │
             ▼
       Docker Build & Push
             │
             ▼
       Trivy Image Scan
             │
             ▼
       SSH to EC2
             │
             ▼
       Docker Compose Deploy
```

The repository uses **reusable GitHub Actions workflows** with `workflow_call` so individual pipeline stages can be kept separate and then orchestrated from the main DevSecOps workflow.

## 🧰 Tech Stack

| Area | Tools |
|---|---|
| Application | Python, Flask |
| CI/CD | GitHub Actions |
| Code Quality | Flake8 |
| SAST | Bandit |
| Secret Scanning | Gitleaks |
| Dependency Security | pip-audit |
| Dockerfile Linting | Hadolint |
| Testing | Pytest |
| Containerization | Docker, Docker Compose |
| Image Registry | Docker Hub |
| Image Vulnerability Scanning | Trivy |
| Server | AWS EC2 |
| Deployment | SSH, Docker Compose |

## 🔐 DevSecOps Pipeline

### 1. Code Quality + SAST

The `code-quality` workflow validates the Python application across multiple Python versions and runs:

- **Flake8** for Python linting and code-quality checks.
- **Bandit** for static application security testing (SAST).

The pipeline uses a matrix to validate the application against multiple Python versions.

### 2. Secret Scanning

**Gitleaks** scans the repository for accidentally committed secrets and credentials.

### 3. Dependency Scanning

**pip-audit** checks Python dependencies for known security vulnerabilities.

### 4. Dockerfile Security

**Hadolint** checks the Dockerfile for common issues and Dockerfile best-practice violations.

### 5. Automated Tests

The application includes basic route tests that are executed before the build/deployment stages are allowed to continue.

### 6. Docker Build & Push

After the required checks pass, GitHub Actions builds the Docker image and pushes it to Docker Hub.

The deployment process uses a commit SHA as an image tag so the server can deploy a specific build rather than relying only on a mutable `latest` tag.

Example image format:

```text
<dockerhub-username>/github-actions-app:<git-sha>
```

### 7. Container Image Scanning

The published image is scanned with **Trivy** for container vulnerabilities.

The pipeline is configured to treat **HIGH** and **CRITICAL** findings as the security gate for the image-scan stage.

### 8. Deployment to AWS EC2

After the security stages succeed, GitHub Actions connects to the production EC2 server over SSH.

The deployment stage:

1. Installs Docker and Docker Compose on the server when required.
2. Creates the application directory.
3. Copies `docker-compose.yml` to the server.
4. Logs into Docker Hub using a token stored in GitHub Secrets.
5. Pulls/recreates the application container using Docker Compose.

## 🏗️ Repository Structure

```text
.
├── .github/
│   └── workflows/
│       ├── DevSecOps-Pipeline.yml
│       ├── code-quality.yml
│       ├── secrets-scan.yml
│       ├── dependencies-scan.yml
│       ├── docker-scan.yml
│       ├── build.yml
│       ├── image-scan.yml
│       └── deploy.yml
│
├── app.py
├── templates/
│   └── index.html
├── tests/
├── Dockerfile
├── docker-compose.yml
├── requirements.txt
└── README.md
```

> Workflow filenames can vary if you rename them; the important idea is the separation between the reusable stage workflows and the main DevSecOps orchestration workflow.

## 🐍 Application

The project uses a small Flask application with the following routes:

```text
GET /
GET /health
```

The container runs the application with **Gunicorn** rather than Flask's built-in development server.

### Dockerfile

The application is containerized using a Python slim base image and Gunicorn:

```dockerfile
FROM python:3.13-slim

WORKDIR /app

COPY . .

RUN pip install --no-cache-dir -r requirements.txt

EXPOSE 80

CMD ["gunicorn", "--bind", "0.0.0.0:80", "app:app"]
```

## 💻 Run Locally

### Prerequisites

Make sure you have:

- Python 3.x
- Docker
- Docker Compose
- Git

### Clone the repository

```bash
git clone https://github.com/Abhishekgupta295/tws-github-actions-learning.git
cd tws-github-actions-learning
```

### Run with Python

```bash
python -m venv .venv
```

Activate the environment:

**Linux/macOS**

```bash
source .venv/bin/activate
```

**Windows PowerShell**

```powershell
.\.venv\Scripts\Activate.ps1
```

Install dependencies:

```bash
pip install -r requirements.txt
```

Run the application:

```bash
python app.py
```

The application can then be accessed locally according to the host/port configured in `app.py`.

### Run with Docker Compose

```bash
docker compose up -d --build
```

Check running containers:

```bash
docker compose ps
```

Stop the application:

```bash
docker compose down
```

## 🔑 GitHub Actions Configuration

The deployment workflow expects Docker Hub and EC2 credentials to be configured in the repository's **Actions variables and secrets**.

### GitHub Actions Variables

```text
DOCKERHUB_USERNAME
```

### GitHub Actions Secrets

```text
DOCKERHUB_TOKEN
EC2_SSH_HOST
EC2_SSH_USERNAME
EC2_SSH_PRIVATE_KEY
```

Do not commit private keys, access tokens, or other credentials to the repository. GitHub recommends storing sensitive values as Actions secrets and following least-privilege practices. citehttps://docs.github.com/en/actions/reference/security/secure-use

## 🔄 Why reusable workflows?

Instead of putting the entire pipeline into one large YAML file, the project separates responsibilities into reusable workflows.

For example:

```yaml
on:
  workflow_call:
```

The main pipeline can then orchestrate individual security, build, scan, and deployment workflows.

This makes the pipeline easier to understand, reuse, and maintain as the project grows.

## 🎯 Key DevSecOps Concepts Practiced

- CI/CD with GitHub Actions
- Reusable workflows with `workflow_call`
- Job dependencies and pipeline gating with `needs`
- Matrix strategy for multi-version testing
- GitHub Actions variables and secrets
- Docker image build and publishing
- Commit-SHA based image tagging
- Static application security testing (SAST)
- Secret scanning
- Dependency vulnerability scanning
- Dockerfile linting
- Container image vulnerability scanning
- SSH-based deployment
- Docker Compose based application deployment
- Troubleshooting failed CI/CD stages

## 🧪 Troubleshooting Lessons

A major part of this project was not just getting a green pipeline, but understanding why individual stages failed and fixing them.

Some of the issues encountered while building and deploying the project included:

- SSH authentication failures during SCP deployment
- GitHub Actions variable-name mismatches in Docker Compose
- Invalid Docker image references caused by missing environment variables
- Flask/Gunicorn port conflicts on port `80`
- Flake8 formatting violations
- Bandit warnings for binding to `0.0.0.0`
- Trivy reporting vulnerabilities in the container image

These failures were useful for understanding the actual interaction between GitHub Actions, Docker, Linux, SSH, and AWS rather than only working with ideal examples.

## 📸 Project Evidence

The repository includes successful GitHub Actions runs showing the pipeline stages and container image publication to Docker Hub.

A typical successful run flows through:

```text
Code Quality
     ↓
Secrets Scan
     ↓
Dependency Scan
     ↓
Dockerfile Scan
     ↓
Tests
     ↓
Build
     ↓
Image Scan
     ↓
Deploy
```

## 📌 What I Learned

This project helped me understand that a DevSecOps pipeline is not simply a collection of security tools. The real value comes from integrating those checks into the software delivery flow so that quality and security become part of the deployment process.

It also gave me practical experience debugging real pipeline failures involving YAML, shell commands, Docker, SSH, Gunicorn, AWS EC2, and security scanners.

## 🔗 Project

**GitHub:** https://github.com/Abhishekgupta295/tws-github-actions-learning

## 🙌 Acknowledgement

This project was built as a hands-on learning project while studying GitHub Actions, CI/CD, Docker and DevSecOps concepts.

A special thanks to **Shubham Londhe** for the practical GitHub Actions learning material that provided the foundation for this work.
