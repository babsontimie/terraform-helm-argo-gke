# terraform-helm-argo-gke

# High-Level Pipeline Architecture

                   ┌─────────────────────────────────────┐
                   │            Git Repository           │
                   │  - Terraform for GKE Infra          │
                   │  - Helm charts for app              │
                   │  - Argo CD app manifests per env    │
                   └─────────────────────────────────────┘
                                │
             ┌──────────────────┼──────────────────┐
             │                  │                  │
        ┌────▼────┐        ┌────▼────┐        ┌────▼────┐
        │   Dev   │        │  Test   │        │  Prod   │
        └────┬────┘        └────┬────┘        └────┬────┘
             │                  │                  │
             ▼                  ▼                  ▼
      GKE Cluster          GKE Cluster        GKE Cluster
     (Dev namespace)      (Test namespace)   (Prod namespace)
             │                  │                  │
             ▼                  ▼                  ▼
         Argo CD           Argo CD            Argo CD
      (watching Git)     (watching Git)      (watching Git)

Pipeline Breakdown

# Step 1: Provision GKE + Argo CD with Terraform

Each environment can be represented by its own Terraform workspace or directory:

```text
infra/
├── dev/
│   └── main.tf
├── test/
│   └── main.tf
└── prod/
    └── main.tf
```

Each main.tf:

Creates a GKE cluster (or shared cluster with separate namespaces)

Installs Argo CD (optional: via Helm provider)

Sets up networking, IAM, etc.

✅ Recommended: Use Terraform modules to avoid repetition.

# Step 2: Prepare Helm Chart

Helm chart for your application:

```text
helm/
  └── my-app/
      ├── Chart.yaml
      ├── values.yaml
      └── templates/
          ├── deployment.yaml
          └── service.yaml
```

You’ll use separate values files per environment:

```text
helm-values/
  ├── dev-values.yaml
  ├── test-values.yaml
  └── prod-values.yaml
```

Step 3: Create Argo CD App Manifests

These define how Argo CD will deploy your app in each environment.

```text
argo-apps/
  ├── dev-app.yaml
  ├── test-app.yaml
  └── prod-app.yaml
```
# Step 4: CI/CD Pipeline

You can use GitHub Actions, GitLab CI, or similar. Here’s a rough flow:

Terraform Plan & Apply (Infra)

On infra/* changes → run Terraform

Argo CD App Sync (optional)

On changes to argo-apps/*.yaml → apply to Argo CD

Helm Values Deployment

On changes to helm-values/*.yaml or Helm chart → Argo CD syncs automatically

# Git Repo Structure Example

```text
your-repo/
├── infra/
│   ├── dev/
│   ├── test/
│   └── prod/
├── helm/
│   └── my-app/
├── helm-values/
│   ├── dev-values.yaml
│   ├── test-values.yaml
│   └── prod-values.yaml
├── argo-apps/
│   ├── dev-app.yaml
│   ├── test-app.yaml
│   └── prod-app.yaml
└── .github/
    └── workflows/
        └── ci.yml
```

# 🔐 GCP Permissions

Ensure:

Terraform service account can create GKE clusters, IAM roles, networks

Argo CD service account (in-cluster) has access to namespaces

CI/CD has deploy permissions (or push access to Git if using GitOps)        

# ✅ Argo CD Setup Tip

Install Argo CD using Helm or kubectl in each cluster:

kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml        

# kubectl port-forward svc/argocd-server -n argocd 8080:443
