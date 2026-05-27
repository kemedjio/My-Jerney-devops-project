# 🛤️ Jerney — Blog Platform

A Gen-Z vibe blog platform built with a 3-tier architecture — React frontend, Node.js backend, and PostgreSQL database.

![Tech Stack](https://img.shields.io/badge/React-18-61DAFB?style=flat-square&logo=react)
![Tech Stack](https://img.shields.io/badge/Node.js-20-339933?style=flat-square&logo=node.js)
![Tech Stack](https://img.shields.io/badge/PostgreSQL-16-4169E1?style=flat-square&logo=postgresql)

---

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
My-Jerney-devops-project/
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
└── terraform/
│   ├── bastion-ec2.tf      # Bastion host
│   ├── data.tf             # retrieve ubuntu AMI
│   ├── eks.tf              # AWS EKS cluster module configuration
│   ├── outputs.tf          # Output bastion public IP, cluster name, vpc id
│   ├── terraform.tf        # terraform provider config
│   ├── terraform.tf        # terraform provider config
│   ├── vpc.tf              # AWS VPC module for the infra
└── argocd
│   ├── jerney-app.yaml     # jerney argocd application file
│
└── k8s
    ├── jerney.yaml         # jerney kubernetes manifest
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
CI.yaml
sim-security-scan.yml

## Create our Argocd App

Create the Argo app manifest and place it inside the `argocd/` directory.

Now apply the file:

```bash
kubectl apply -f boutique-app.yaml
```

Check the ArgoCD UI; you should see the app visible there. And all Synced.

# Observability

We  don't manage the observability stack with ArgoCD. Because anyone having access to Argocd can modify it.

## 1. Monitoring

Create a namespace:

```bash
kubectl create ns monitoring
```

## Setup Slack

Create a dedicated channel where you want to receive the alerts.

**`#alertmanager`**

After this is done 

Give name and choose the workspace and create.


Head to **`Incoming Webhook`**

Turn it ON

Scroll down then Click on **`Add New Webhook`**

Select the channel and then allow.

Copy the Webhook and keep it somewhere pasted.

### Create a Kubernetes Secret for Slack Webhook

On your cluster:

```bash
kubectl create secret generic alertmanager-slack-webhook \
  --from-literal=slack-webhook-url="<Webhook FQDN>" \
  -n monitoring
```

Verify:

```bash
kubectl get secret alertmanager-slack-webhook -n monitoring
```

### Kube-Prometheus-Stack

Add `kube-prometheus-stack`  repo:

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
```

get the helm values and save it in a file:

```bash
helm show values prometheus-community/kube-prometheus-stack --version 81.6.3 > observability/helm-values/kube-prom-stack-81.6.3.yaml 
```

Edit in `vi`

Attach the secret in `alertmanagerSpec:` section. So that it will mount to the pod.

```bash
alertmanager:
  alertmanagerSpec:
        secrets:
            - alertmanager-slack-webhook
```

Go to `alertmanger` section and find its `config` block:

```bash
config:
    global:
      resolve_timeout: 5m
    route:
      group_by: ['namespace']
      group_wait: 30s
      group_interval: 5m
      repeat_interval: 12h
      receiver: 'slack-notification'
      routes:
      - receiver: 'slack-notification'
        matchers:
          - severity = "critical"
    receivers:
    - name: 'slack-notification'
      slack_configs:
          - api_url_file: /etc/alertmanager/secrets/alertmanager-slack-webhook/slack-webhook-url
           channel: '#alerts'
           send_resolved: true
    templates:
    - '/etc/alertmanager/config/*.tmpl'
```

>### Slack Config
>
>```yaml
>api_url:'https://hooks.slack.com/services/...'channel:'#alerts'send_resolved:true
>```
>
># 5. Templates
>
>```yaml
>templates:-'/etc/alertmanager/config/*.tmpl'
>```

>---
>
># How It Actually Works (End-to-End Flow)
>
>### 1. Application exposes metrics
>
>Example:
>
>```
>http_requests_total
>pod_memory_usage_bytes
>up
>```
>---
>
>### 2. Prometheus scrapes those metrics
>
>From:
>
>- Pods
>- Services
>- Nodes
>- Kubernetes API
>- etc.
>
>---
>
>### 3. Alert Rules Define When Something Is Critical
>
>Inside kube-prometheus-stack, there are many alert rules like:
>
>```yaml
>-alert:PodCrashLoopingexpr:kube_pod_container_status_restarts_total>5for:5mlabels:severity:criticalannotations:description:Podisrestartingfrequently
>```

>
>### 4. Prometheus Sends Alert to Alertmanager
>
>When the condition becomes true:
>
>```json
>{"alertname":"PodCrashLooping","severity":"critical","namespace":"production"}
>```
>
>Prometheus pushes this to Alertmanager.
>
>---
>
>### 5. Alertmanager Routes Based on Labels
>
>Now your config says:
>
>```yaml
>matchers:-severity="critical"
>```
>

Install Prometheus:

```bash
helm upgrade -i kube-prometheus-stack prometheus-community/kube-prometheus-stack --version 81.6.3 -f helm-values/kube-prom-stack-81.6.3.yaml -n monitoring
```

Check if all the pods are running:

```bash
kubectl get po -n monitoring
```

Check the Services:

```bash
kubectl get svc -n monitoring


```| Branch | Purpose |
|--------|---------|
| `main` | Source code + EC2 bare-metal deployment |
| `devops` | Full DevSecOps — Docker, Kubernetes (EKS), Terraform, CI/CD pipeline, security scanning |

---

