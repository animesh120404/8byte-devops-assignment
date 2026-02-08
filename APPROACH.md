# 📖 Approach & Design Decisions

> This document explains the methodology, design choices, and implementation approach for the **8byte DevOps intern assignment**.

---

## 📋 Table of Contents

- [1. Overall Methodology](#1-overall-methodology)
- [2. Application Design](#2-application-design)
- [3. Docker Strategy (Multi-Stage Build)](#3-docker-strategy-multi-stage-build)
- [4. Terraform & AWS Design](#4-terraform--aws-design)
- [5. CI/CD (GitHub Actions) Design](#5-cicd-github-actions-design)
- [6. Challenges & Solutions](#6-challenges--solutions)
- [7. Implementation Summary](#7-implementation-summary)

---

## 1. Overall Methodology

The project was executed in a systematic **four-phase lifecycle** to ensure stability at each layer:

1. **Application & Local Run** — Implementing a minimal Node.js app and ensuring it runs locally on Port 3000.

2. **Containerization** — Packaging the application using Docker for consistent and portable deployment across environments.

3. **Infrastructure as Code (IaC)** — Using Terraform to provision a custom AWS VPC, subnet, security groups, and an EC2 instance with Docker pre-installed.

4. **CI/CD Automation** — Automating the build, push, and deployment of the Docker image to Docker Hub and AWS on every push to the `main` branch.

Each phase was validated before moving to the next:
Local ➔ Docker ➔ Terraform ➔ EC2 Deploy ➔ Pipeline

---

## 2. Application Design

- **Stack:** Node.js 18 + Express framework for a simple, high-performance HTTP server.

- **Port:** Fixed to `3000` for consistency across local, Docker, and AWS security group configurations.

- **Scope:** A single route (`/`) returning a success message to satisfy assignment requirements and allow quick verification.

- **Rationale:** The design keeps the focus on DevOps orchestration (Docker, Terraform, CI/CD) rather than unnecessary application complexity.

---

## 3. Docker Strategy (Multi-Stage Build)

I implemented a **Multi-stage Docker Build** to align with industry production standards:

- **Stage 1 (Builder):** Uses `node:18-alpine` to install dependencies with `npm ci`. This ensures deterministic builds by including the `package-lock.json`.

- **Stage 2 (Runner):** Copies only the essential files (`package.json`, `node_modules`, and `app.js`) from the builder stage.

- **Benefits:** This results in a significantly smaller image size, faster pull times, and a reduced attack surface by excluding build tools from the final runtime.

### Key Details:

- `COPY package*.json ./` ensures reproducible builds.

- `USER node` ensures the container process does not run as root for enhanced security.

- `EXPOSE 3000` documents the application port for tooling and transparency.

---

## 4. Terraform & AWS Design

- **Network Layout:** A dedicated VPC (`10.0.0.0/16`) with a Public Subnet (`10.0.1.0/24`) and an Internet Gateway for external connectivity.

- **Security Group:**

  - **Ingress:** Allows SSH (Port `22`) for management and Application Traffic (Port `3000`) from `0.0.0.0/0` for accessibility.

  - **Egress:** All outbound traffic is allowed to enable the instance to pull Docker images and install system updates.

- **EC2 Instance:**

  - **User Data:** The `install_docker.sh` script automates the Docker installation during the first boot, making the instance "Container Ready" instantly.

  - **Variables:** Region, AMI, and instance types are parameterized in `variables.tf` and `terraform.tfvars` for modularity and reusability.

---

## 5. CI/CD (GitHub Actions) Design

- **Trigger:** The pipeline is automated to trigger on every push to the `main` branch.

- **Docker Hub Push:** The workflow handles authentication, building the image from `./app`, and pushing it to `animesh71/8byte-app` automatically.

- **SSH Deployment:** I implemented an SSH-based deployment using `appleboy/ssh-action`. The pipeline securely logs into the EC2 instance, pulls the latest image, stops the old container, and starts the new one.

- **Secrets Management:** All sensitive credentials (`DOCKER_HUB_ACCESS_TOKEN`, `EC2_SSH_KEY`, `EC2_PUBLIC_IP`) are managed via GitHub Secrets to prevent exposure.

---

## 6. Challenges & Solutions

Detailed notes are provided in [`CHALLENGES.md`](./CHALLENGES.md). A quick summary:

- **`npm ci` Failure:** Fixed by ensuring `package-lock.json` was copied into the Docker builder.

- **Terraform Key Error:** Resolved by using the logical AWS Key Pair name instead of a local file path.

---

## 7. Implementation Summary

The project follows a **modular structure** to enforce a clean separation of concerns:

- **`app/`**: Application source and Docker configuration.

- **`terraform/`**: Infrastructure as Code resource definitions.

- **`.github/workflows/`**: Continuous Deployment (CD) automation.

- **`public/`**: Visual evidence of deployment success.

---

<p align="center">
  <b>8byte DevOps Intern Assignment</b> — Node.js, Docker, Terraform, GitHub Actions.
</p>