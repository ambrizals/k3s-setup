# Kubernetes Deployment Repository Template

This folder contains the scaffolding for deploying your application to a K3s cluster with automated GitHub Actions and a private Zot container registry.

## Directory Structure
- **`.github/workflows/build-and-push.yml`**: GitHub Actions pipeline that builds your Docker image, pushes it to your private Zot registry, and applies the Kubernetes YAML manifests.
- **`deployment/`**: This folder contains your Kubernetes deployment configuration, which manages how your application runs inside K3s.
    - `backend-deployment.yaml`: Defines your Docker container and its resource limits.
    - `backend-service.yaml`: Creates an internal networking endpoint for your deployment.
    - `backend-ingress.yaml`: Points your public domain to the internal service and provides SSL via Let's Encrypt.

## Setup Instructions

This project uses **[Infisical](https://infisical.com)** (self-hosted at `https://secrets.bmd.ambrizal.net`) as a central secret manager. Secrets are no longer stored directly in GitHub — they are fetched at runtime by the GitHub Actions pipeline.

### Step 1: Add Secrets to Infisical
Log in to your Infisical dashboard and create the following secrets inside your project:

| Secret Key | Description |
|---|---|
| `REGISTRY_USER` | Username for your private Zot registry |
| `REGISTRY_PASS` | Password for your private Zot registry |
| `KUBECONFIG_DATA` | Full content of your `~/.kube/config` for K3s access |

### Step 2: Create a Machine Identity in Infisical
Go to **Infisical > Project Settings > Machine Identities** and create a new identity with `read` access. Copy the **Client ID** and **Client Secret**.

### Step 3: Add Only 2 GitHub Secrets
Go to **GitHub > Settings > Secrets and variables > Actions** and add:

| Secret Name | Value |
|---|---|
| `INFISICAL_CLIENT_ID` | The Client ID from the Machine Identity you created |
| `INFISICAL_CLIENT_SECRET` | The Client Secret from the Machine Identity you created |

That's it! All other credentials are managed securely inside your self-hosted Infisical.

## Deployment Process
A deployment is triggered automatically every time code is pushed to the `main` branch. 
The workflow will:
1. Build your Docker image based on a `Dockerfile` located in `./backend` (update the `context` in the workflow if your code lives somewhere else).
2. Authenticate with your private `registry.bmd.ambrizal.net` registry.
3. Push the new image.
4. Assume your K3s server context.
5. Apply all configurations found within the `deployment/` directory to update your cluster.
