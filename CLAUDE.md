# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Repo Is

This is a **GitOps configuration repository** for three microservices deployed to Kubernetes via ArgoCD. Changes pushed to this repo are automatically picked up by ArgoCD and applied to the cluster. There is no application source code here — only Kubernetes manifests.

## ArgoCD Setup

```bash
# Install the ApplicationSet into ArgoCD (run from argocd-root/)
make install
# or directly:
kubectl apply -f argocd-root/applicationset.yaml -n argocd
```

The ApplicationSet uses a **Git directory generator** matching `environments/*/*`. Any subdirectory added under `environments/<env>/<app>/` will automatically become a new ArgoCD Application named `<env>-<app>`.

## Repository Structure

```
environments/
  dev/                  # Only environment with active manifests
    orders-api/         # Backend API (container port 8001)
    recipe-api/         # Backend API (container port 8000)
    ramen-ui/           # Frontend (container port 80)
  prod/ qa/ stage/      # Defined but empty — ready for promotion
argocd-root/
  applicationset.yaml   # Master GitOps orchestrator
```

Each application directory contains three manifests: `deployment.yaml`, `service.yaml`, `ingress.yaml`.

## How CI Image Updates Work

An upstream CI pipeline automatically commits image digest updates to this repo with commit messages like:
```
ci: update <app> image to <sha256-digest>
```
The image field in `deployment.yaml` uses the full GHCR digest (`ghcr.io/therealdavidbour/<app>@sha256:...`). When promoting an image to a new environment, copy the `dev/` manifests into the target environment directory and update the image digest.

## Key Infrastructure Details

- **Cluster target:** `https://kubernetes.default.svc` (in-cluster ArgoCD)
- **Namespace:** matches the environment name (`dev`, `prod`, etc.) — auto-created by ArgoCD
- **Ingress:** nginx-ingress with cert-manager TLS using `k3d-ca-issuer`
- **DNS:** nip.io wildcard (`<app>.10-0-0-200.nip.io`)
- **Sync policy:** auto-prune + self-heal enabled on all apps

## Adding a New Environment

1. Create `environments/<env>/<app>/` with the three standard manifests
2. Update ingress hostnames and any environment-specific values
3. Push — ArgoCD picks it up automatically via the directory generator
