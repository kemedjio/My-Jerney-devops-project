# 🛤️ Jerney — Blog Platform

A Gen-Z vibe blog platform built with a 3-tier architecture — React frontend, Node.js backend, and PostgreSQL database.

![Tech Stack](https://img.shields.io/badge/React-18-61DAFB?style=flat-square&logo=react)
![Tech Stack](https://img.shields.io/badge/Node.js-20-339933?style=flat-square&logo=node.js)
![Tech Stack](https://img.shields.io/badge/PostgreSQL-16-4169E1?style=flat-square&logo=postgresql)

---

> [!IMPORTANT]
> **Looking for the full DevSecOps implementation?**
> Switch to the [`devops`](../../tree/devops) branch for Docker, Kubernetes (EKS Auto Mode), Terraform, CI/CD with GitHub Actions, container security scanning, and more.
>
> ```bash
> git checkout devops
> ```

---

## ✨ Features

- 📝 Create blog posts with emoji vibes
- ✏️ Edit your existing posts
- 🗑️ Delete posts you're not feeling anymore
- 💬 Comment on posts
- 🎨 Gen-Z dark UI with glassmorphism and gradients

## 🏗️ Architecture

```
┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│   Frontend   │────▶│   Backend    │────▶│  PostgreSQL   │
│   (React +   │◀────│  (Node.js +  │◀────│              │
│    Nginx)    │     │   Express)   │     │              │
│   Port 80    │     │  Port 5000   │     │  Port 5432   │
└──────────────┘     └──────────────┘     └──────────────┘
```

## 📁 Project Structure

```
Jerney/
├── frontend/                # React (Vite) frontend
│   ├── src/                 # React components & pages
│   ├── nginx.conf           # Nginx config for serving the app
│   └── package.json
├── backend/                 # Node.js Express API
│   ├── src/                 # Routes, DB connection
│   └── package.json
├── deploy/                  # EC2 deployment scripts
│   ├── setup.sh             # One-click EC2 setup script
│   └── jerney-nginx.conf    # Nginx reverse proxy config
└── README.md
```

---


## 📡 API Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/health` | Health check |
| GET | `/api/posts` | Get all posts |
| GET | `/api/posts/:id` | Get single post with comments |
| POST | `/api/posts` | Create a new post |
| PUT | `/api/posts/:id` | Update a post |
| DELETE | `/api/posts/:id` | Delete a post |
| GET | `/api/comments/post/:postId` | Get comments for a post |
| POST | `/api/comments` | Create a comment |
| DELETE | `/api/comments/:id` | Delete a comment |

---
## Install tools in Local Machine

- AWS CLI
- Terraform on your local machine
- Create an IAM user and create an access key and secret access key for the user, and do `aws configure.`

## Terraform Run to create the infrastructure

Clone the repo, `cd` to the `terraform` directory. 

```bash
terraform init
Terraform plan 
terraform apply
```
## Bastion Host Configuration

SSH to bastion and  install the following tools:

- AWS CLI
- kubectl client
- HELM
- eksctl

# update the kube config file and check
aws eks update-kubeconfig --region <your-region> --name <your-cluster-name> 

kubectl get nodes

# Install the AWS load balancer controller

Create an IAM OIDC provider. 

```
eksctl utils associate-iam-oidc-provider \
    --region us-east-1 \
    --cluster ecommerce-cluster \
    --approve
```

**Create IAM role using `eksctl`.**

1. Download an IAM policy for the AWS Load Balancer Controller 
    ```bash
    curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.14.1/docs/install/iam_policy.json
    ```
    
2. Create an IAM policy using the policy downloaded in the previous step.
    
    ```bash
    aws iam create-policy \
        --policy-name AWSLoadBalancerControllerIAMPolicy \
        --policy-document file://iam_policy.json
    ```
    
3. Replace the values for cluster name, region code, and account ID.
    
    ```bash
    eksctl create iamserviceaccount \
        --cluster=jerney-cluster \
        --namespace=kube-system \
        --name=aws-load-balancer-controller \
        --attach-policy-arn=arn:aws:iam::<AWS_ACCOUNT_ID>:policy/AWSLoadBalancerControllerIAMPolicy \
        --override-existing-serviceaccounts \
        --region us-east-1 \
        --approve
    ```
    

**Install AWS Load Balancer Controller**

1. Add the `eks-charts` Helm chart repository. 
    ```bash
    helm repo add eks https://aws.github.io/eks-charts
    ```
    

        helm upgrade -i aws-load-balancer-controller eks/aws-load-balancer-controller \
          -n kube-system \
          --set clusterName=jerney-cluster \
          --set region=us-east-1 \
          --set vpcId=vpc-043ed20a9ec883107 \
          --set serviceAccount.create=false \
          --set serviceAccount.name=aws-load-balancer-controller \
          --set controllerConfig.featureGates.NLBGatewayAPI=true \
          --set controllerConfig.featureGates.ALBGatewayAPI=true \
          --version 3.0.0
        ```
        

**Verify that the controller is installed**

    
    ```bash
    kubectl get deployment -n kube-system aws-load-balancer-controller
    ```## Set up Terraform Remote Backend (Optional)

Create a bucket using Console or AWS CLI.

```bash
aws s3api create-bucket \
  --bucket ecommerce-terraform-backend-bucket234 \
  --region us-east-1
## 🌿 Branch Strategy


## Deploy ArgoCD


**Add ArgoCD repo**

```bash
helm repo add argo https://argoproj.github.io/argo-helm
```

Get the values:

```bash
helm show values argo/argo-cd --version 9.4.0 > argocd-values-9.4.0.yaml
```

Modify the values file:

Add  ”`server.insecure: true`”  line explicitly :

Add this `kustomize.buildOptions: "--enable-helm` line in the `config` section as Kustomize requires to combine the helm values and manifest file.

- Helm support inside Kustomize is considered an **unsafe plugin**, so it is disabled.
- You must explicitly allow it.

    kustomize.buildOptions: "--enable-helm"


Install the chart:

```bash
helm install argo-cd argo/argo-cd -n argocd -f argocd-values-9.4.0.yaml --version 9.4.0 --create-namespace
##  set up the CI part in GitHub Actions.

```bash
mkdir -p .github/workflows
```
Inside the `workflows,` create two config files.

```| Branch | Purpose |
|--------|---------|
| `main` | Source code + EC2 bare-metal deployment |
| `devops` | Full DevSecOps — Docker, Kubernetes (EKS), Terraform, CI/CD pipeline, security scanning |

---

Built with 💜 by the Jerney team. No cap, this blog platform hits different. 🛤️
