# K3s Setup & Continuous Deployment Guide

This repository contains the infrastructure and deployment configuration for a production-grade development environment using **K3s**, **Zot Registry**, **Infisical Secret Manager**, and **Cert-Manager**.

## 📂 Directory Structure

The project is divided into two main parts:

### 1. [server-setup/](./server-setup/)

Contains all the "one-time" infrastructure setup files that must be applied directly to your K3s server.

- **K3s Registry Mirror**: Configures K3s to use your private Zot registry.
- **Cert-Manager**: Automated SSL certificates via Let's Encrypt.
- **Zot Registry**: Private container registry for your Docker images.
- **PostgreSQL**: Internal database for your applications.
- **Infisical**: Self-hosted secret manager to securely store and sync credentials.

### 2. [github-project/](./github-project/)

A template for your application repository. This is what you commit to your GitHub project.

- **GitHub Actions**: Automated workflow to build, push, and deploy your code.
- **Kubernetes Manifests**: Deployment, Service, and Ingress configurations for your app.
- **Infisical Integration**: Pulls secrets from your self-hosted instance at runtime.

---

## 🚀 Getting Started

### Step 0: Manual Pre-setup (Firewall & Ports)
Before anything else, you must manually open ports **80**, **443**, and **6443** on your server's firewall (e.g., `ufw`) or cloud security group. Without this, your Ingress and CI/CD will not be reachable.

### Step 1: Server Infrastructure
Go to the [server-setup README](./server-setup/README.md) and follow Phase 0 through 8 to get your cluster ready.

### Step 2: GitHub Configuration
Go to the [github-project README](./github-project/README.md) to learn how to connect your application repository to your new K3s infrastructure.

---

## 🔒 Security & Secrets Flow

This setup uses a "GitOps-friendly" secret management flow:

1. **Infisical Dashboard**: You manage secrets (API keys, DB passwords) in a web UI on your own server.
2. **Infisical Operator**: Automatically syncs those secrets into native Kubernetes Secrets inside your cluster.
3. **Pods**: Your applications consume these secrets as standard environment variables via `envFrom`.

No sensitive credentials (besides the Infisical Client ID/Secret) are ever stored in GitHub or committed to code.

## Thanks Gemini

So i would thanks for gemini to make this repo, because it help me a lot to setup my k3s cluster and continuous deployment.
