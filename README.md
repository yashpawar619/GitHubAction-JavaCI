# Java CI Pipeline with GitHub Actions

**This project sets up a Java-based CI pipeline using GitHub Actions with Maven, Trivy, Gitleaks, SonarQube, and Docker.**

## Contents

If you are looking for a template, go to Actions in GitHub, where you will see default templates. You can also create one from scratch as per requirements.

Added different steps in the YAML file:
Compile & Build (Maven)
Security Scans (Trivy + Gitleaks)
Unit Tests
SonarQube Analysis & Quality Gate
Docker Build & Push
All the above was performed on GitHub shared runners initially, then moved to self-hosted runners for private control.

## 🖥️ Adding a Private Runner

Go to Settings → Actions → Runner → New Self-Hosted Runner.
GitHub provides commands for setup (download, configure, and start runner).
Create a VM (EC2 or Azure VM) and add appropriate inbound rules.Example: Allow inbound 9000 for SonarQube.
Then, Install Maven on the VM (used in build steps):
sudo apt install maven -y

Create two VMs:
Runner VM – where workflows execute.
SonarQube VM – install Docker and run SonarQube:

sudo apt install docker.io -y
sudo usermod -aG docker $USER
newgrp docker
docker run -d --name sonar -p 9000:9000 sonarqube:lts-community

Add sonar-project.properties file in repo.

## 🔍 Adding SonarQube

Once SonarQube is up, access it at:
http://<VM_PUBLIC_IP>:9000

Setup admin user → Security → Users → Generate Token.
Use the SonarQube GitHub Action Template from GitHub Marketplace
.Add the token as a secret in:
Settings → Secrets and variables → Actions → New repository secret
Example: SONAR_TOKEN, SONAR_HOST_URL.
Added Quality Gate check step from SonarQube official documentation.

## 🛡️ Security Checks

Trivy for scanning vulnerabilities:
trivy fs --format table -o fx-report.json .
Gitleaks for scanning secrets in repo:
gitleaks detect source . -r gitleaks-report.json -f json

## 🐳 Docker on Runner

Install Docker on the Runner:
sudo apt-get update
sudo apt-get install docker.io -y
sudo usermod -aG docker $USER
newgrp docker

Workflow uploads the JAR artifact from build stage → downloads it in Docker build step.
Docker login step uses GitHub Secrets:
DOCKERHUB_USERNAME (variable)
DOCKERHUB_TOKEN (secret)
Final Docker image pushed to Docker Hub:
xyz/bankapp:latest

## ⚠️ Issues Faced

Forgot to add inbound rules → SonarQube page was not loading (fixed by updating security group).
Needed to install unzip on the runner for artifact steps.

## ❓ Questions
_Will you be able to answer it_?

Why is Maven needed on private runners but not on GitHub-hosted runners?

Why do we use docker.io install for SonarQube VM but Docker GitHub Actions for Runner?

To run workflows manually, you can add:
on:
  workflow_dispatch:
