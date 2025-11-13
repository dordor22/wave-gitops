# 🚀 Wave-App GitOps

This repository contains the complete **GitOps configuration** for the **Wave-App** platform.  
It provides automated, declarative, multi-environment **Continuous Delivery (CD)** using **ArgoCD**.  
Git serves as the single source of truth, and ArgoCD ensures that the actual cluster state always matches the desired state defined in this repository.

The repository manages deployments for three environments:
- **dev**
- **stage**
- **prod**

Each environment has its own dedicated Helm values and its own ArgoCD-managed deployment.

---

## 🏗️ GitOps Architecture

This repository is part of a 3-repository DevOps structure:

### 1️⃣ Application Repository (CI)
🔗 https://github.com/dordor22/wave-app  
Contains:
- React frontend
- Flask backend
- Dockerfiles
- Helm chart (`helm/`)
- GitHub Actions (CI → build & push images to ECR)

### 2️⃣ Infrastructure Repository (Terraform)
🔗 https://github.com/dordor22/wave-app-infra  
Contains:
- AWS VPC networking  
- EKS clusters (dev/stage/prod)
- ECR registries
- S3 backup buckets
- Terraform remote backend (S3 + DynamoDB)

### 3️⃣ GitOps Repository (CD) – this repo
🔗 https://github.com/dordor22/wave-gitops  
Contains:
- ArgoCD Project
- ArgoCD ApplicationSet
- Environment-specific Helm values

ArgoCD continuously monitors this repository and applies any changes automatically to the relevant EKS environment.

---

## 📁 Repository Structure

````text
wave-gitops/
│
├── .gitignore
├── README.md
│
├── argocd/
│   ├── project-wave-app.yaml
│   └── applicationset-wave-app.yaml
│
└── env/
    ├── dev/
    │   └── values.yaml
    │
    ├── stage/
    │   └── values.yaml
    │
    └── prod/
        └── values.yaml
````


## 🧩 ArgoCD Components

### 🔹 AppProject
Defines:
- Allowed Git repositories  
- Allowed namespaces  
- Cluster access boundaries  

Purpose:
- Prevent misconfigurations  
- Improve security  
- Limit what ArgoCD can deploy  

File:
`argocd/project-wave-app.yaml`

---

### 🔹 ApplicationSet
Automatically generates ArgoCD Applications for:

- wave-app-dev  
- wave-app-stage  
- wave-app-prod  

Each application:
- Uses the Helm chart from the `wave-app` repository  
- Uses environment values from this GitOps repo  
- Deploys into a dedicated namespace  
- Supports automated syncing and self-healing  

File:
`argocd/applicationset-wave-app.yaml`

---

## 🌍 Environment-Specific Configuration

Each environment has:
- Its own EKS namespace  
- Its own Helm values  
- Its own ECR image tags  
- Its own overrides (resources, replicas, ingress, etc.)

Files:
- `env/dev/values.yaml`
- `env/stage/values.yaml`
- `env/prod/values.yaml`

---

## 🔄 Deployment Workflow

### 🧪 CI – wave-app (build & publish)
- Developer pushes code  
- GitHub Actions builds Docker images  
- Pushes them to the correct ECR repositories  
- (Optional) CI updates the values.yaml in `wave-gitops` with new image tags  

---

### 🚀 CD – wave-gitops (deploy automatically)
- ArgoCD watches this repository  
- Detects any commit  
- Applies the Helm chart to the correct environment  
- Ensures cluster state always matches Git  
- Auto-heals drift  

This creates a full GitOps pipeline from CI → CD.

---

## 🚀 Applying ArgoCD Configuration

Once ArgoCD is installed in your cluster:

```bash
kubectl apply -n argocd -f argocd/project-wave-app.yaml
kubectl apply -n argocd -f argocd/applicationset-wave-app.yaml
```

ArgoCD will automatically create:
- `wave-app-dev`
- `wave-app-stage`
- `wave-app-prod`

---

## 📚 Summary

This repository enables a clean and scalable GitOps model:

- Automated multi-environment deployments  
- ArgoCD-based Continuous Delivery  
- Strong environment isolation  
- Seamless integration with CI (wave-app) and Infra (wave-app-infra)  
- Production-ready structure used in real DevOps/SRE teams  

Wave-App GitOps integrates directly with:
- `wave-app` → source code & CI  
- `wave-app-infra` → infrastructure provisioning  
- `wave-gitops` → continuous delivery  

This completes the full DevOps lifecycle for Wave-App.
