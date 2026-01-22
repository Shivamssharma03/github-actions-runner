# 🚀 Self-Hosted GitHub Actions Runner on AWS EC2

This document explains how to set up a **secure, production-ready self-hosted GitHub Actions runner** on an **AWS EC2 (Ubuntu)** instance using **CLI commands**, best practices, and proper Linux permissions.

---

## 📌 Why Use a Self-Hosted Runner?

- Full control over build environment
- Faster builds (no cold starts)
- Access to private infrastructure
- Custom tooling (Docker, AWS CLI, etc.)
- Cost-effective for heavy workloads

---

## 🏗️ Architecture Overview


---

## 🔐 Security Best Practices (IMPORTANT)

- ❌ Never run the runner as `root`
- ✅ Use a dedicated user (`github-runner`)
- ✅ Grant permissions via groups (`sudo`, `docker`)
- ✅ Use IAM Role on EC2 (no AWS keys)
- ✅ Use repo-scoped runners
- ✅ Clean workspace after every job

---

## 🧱 Prerequisites

- AWS account
- EC2 instance (Ubuntu 22.04 recommended)
- GitHub repository (Admin access)
- Internet access from EC2

---

## 1️⃣ Launch EC2 Instance

### Recommended Configuration
- AMI: **Ubuntu Server 22.04**
- Instance type: `t3.medium` (minimum)
- Storage: **30 GB**
- Security Group:
  - SSH (22) → your IP only

---

## 2️⃣ Connect to EC2

```bash
ssh -i your-key.pem ubuntu@<EC2_PUBLIC_IP>
sudo apt update && sudo apt upgrade -y


## 3️⃣ Create Dedicated Runner User
sudo adduser github-runner
sudo usermod -aG sudo github-runner

Switch user:

su - github-runner


4️⃣ Install Required System Packages

sudo apt install -y \
  curl \
  git \
  unzip \
  tar \
  ca-certificates \
  build-essential

5️⃣ Install Docker (REQUIRED)

sudo apt install -y docker.io
sudo systemctl enable docker
sudo systemctl start docker

Allow runner user to run Docker:

sudo usermod -aG docker github-runner
newgrp docker

Verify:

docker run hello-world



6️⃣ (Optional) Install AWS CLI v2

Needed only if you upload artifacts to S3 or use ECR.

cd /tmp
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o awscliv2.zip
sudo apt install -y unzip
unzip awscliv2.zip
sudo ./aws/install


Verify:

aws --version



7️⃣ Download GitHub Actions Runner

mkdir ~/actions-runner
cd ~/actions-runner


Download runner:
curl -L -o actions-runner.tar.gz \
https://github.com/actions/runner/releases/download/v2.331.0/actions-runner-linux-x64-2.331.0.tar.gz

tar xzf actions-runner.tar.gz



8️⃣ Register Runner (Repo-Scoped)

In GitHub:

Repository → Settings → Actions → Runners → New self-hosted runner




./config.sh \
  --url https://github.com/<ORG>/<REPO> \
  --token <TOKEN> \
  --name ec2-runner \
  --labels self-hosted,linux,docker \
  

  9️⃣ Run Runner as a Service (IMPORTANT)

Exit runner if running:

exit


Install and start service:

cd /home/github-runner/actions-runner
sudo ./svc.sh install github-runner
sudo ./svc.sh start


Check status:

sudo ./svc.sh status


Expected:

Active: active (running)

🔎 Verify Runner in GitHub
Repository → Settings → Actions → Runners


You should see:

ec2-runner  ● Idle





