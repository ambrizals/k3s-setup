# AI Context: Application Deployment & CI/CD

This directory is a template for application development and deployment. Use this context when helping with backend deployment, service scaling, or GitHub Actions troubleshooting.

## 🔄 Deployment Flow
1. Code push to `main` branch.
2. GitHub Action triggers.
3. **Infisical Auth**: Pipeline uses `INFISICAL_CLIENT_ID`/`SECRET` to fetch application secrets.
4. **Build**: Docker image built and tagged as `:latest`.
5. **Push**: Image pushed to `registry.bmd.ambrizal.net`.
6. **Deploy**: `kubectl apply` runs against the `deployment/` folder.
7. **Rollout**: `kubectl rollout restart` triggers a rolling update to pull the new image.

## 🏠 Kubernetes Resources
- **`backend-deployment.yaml`**: Uses `envFrom` to pull ALL environment variables from a synced secret called `backend-secrets`.
- **`backend-service.yaml`**: Exposes the app on port 80 internally.
- **`backend-ingress.yaml`**: Handles public routing and SSL for `api.bmd.ambrizal.net`.

## 🛠️ Troubleshooting Tips
- **Secrets missing?** Check the Infisical Operator pods in the cluster or the `InfisicalSecret` CRD status.
- **Image pull fails?** Verify `registries.yaml` is on the K3s server and K3s has been restarted.
- **SSL not issuing?** Check `cert-manager` logs or the `Challenge` resource.
