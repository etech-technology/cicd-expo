# GitHub Actions Helm Deployment Pipeline

## Overview
This project implements a **CI/CD pipeline using GitHub Actions** to deploy applications to **AWS EKS clusters** using **Helm**. The pipeline automates deployments based on Git branch activity.

### Deployment Strategy
| **Branch**       | **AWS Environment** | **AWS Region** | **Trigger**               | **Deployment Method** |
|------------------|--------------------|----------------|---------------------------|----------------------|
| `dev`           | Dev EKS Cluster     | `us-west-1`    | Push to `dev` branch      | Helm |
| `uat`           | UAT EKS Cluster     | `us-east-1`    | Push to `uat` branch      | Helm |
| `release/*`     | Prod EKS Cluster    | `us-east-2`    | New `release/*` branch    | Helm |

## Prerequisites
- **AWS IAM Credentials** with permissions to manage EKS
- **Helm installed** (`curl -fsSL https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash`)
- **AWS CLI configured** (`aws configure`)
- **GitHub Secrets** setup for AWS authentication:
  - `AWS_ACCESS_KEY_ID`
  - `AWS_SECRET_ACCESS_KEY`

## GitHub Actions Workflow (`.github/workflows/deploy.yml`)
```yaml
name: Helm Deployment to EKS

on:
  push:
    branches:
      - dev
      - uat
  create:
    branches:
      - "release/*"

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout Repository
        uses: actions/checkout@v4

      - name: Set Environment Variables
        run: |
          if [[ "${{ github.ref }}" == "refs/heads/dev" ]]; then
            echo "AWS_REGION=us-west-1" >> $GITHUB_ENV
            echo "EKS_CLUSTER=dev-cluster" >> $GITHUB_ENV
            echo "NAMESPACE=dev" >> $GITHUB_ENV
            echo "HELM_RELEASE_NAME=dev-app" >> $GITHUB_ENV
          elif [[ "${{ github.ref }}" == "refs/heads/uat" ]]; then
            echo "AWS_REGION=us-east-1" >> $GITHUB_ENV
            echo "EKS_CLUSTER=uat-cluster" >> $GITHUB_ENV
            echo "NAMESPACE=uat" >> $GITHUB_ENV
            echo "HELM_RELEASE_NAME=uat-app" >> $GITHUB_ENV
          elif [[ "${{ github.ref }}" == refs/heads/release/* ]]; then
            echo "AWS_REGION=us-east-2" >> $GITHUB_ENV
            echo "EKS_CLUSTER=prod-cluster" >> $GITHUB_ENV
            echo "NAMESPACE=prod" >> $GITHUB_ENV
            echo "HELM_RELEASE_NAME=prod-app" >> $GITHUB_ENV
          fi

      - name: Configure AWS Credentials
        uses: aws-actions/configure-aws-credentials@v3
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ${{ env.AWS_REGION }}

      - name: Update Kubeconfig for EKS
        run: |
          aws eks update-kubeconfig --region $AWS_REGION --name $EKS_CLUSTER

      - name: Install Helm
        run: |
          curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
          helm version

      - name: Deploy Helm Chart to EKS
        run: |
          helm upgrade --install $HELM_RELEASE_NAME ./helm-chart \
            --namespace $NAMESPACE \
            --create-namespace \
            --set image.repository=my-docker-repo/my-app \
            --set image.tag=latest \
            --set replicaCount=2 \
            --set service.port=80
```

## Helm Chart Setup
Run the following command to create a Helm chart:
```sh
helm create helm-chart
```

### `helm-chart/Chart.yaml`
```yaml
apiVersion: v2
name: my-app
description: A Helm chart for my Kubernetes application
type: application
version: 1.0.0
appVersion: "1.0"
```

### `helm-chart/values.yaml`
```yaml
replicaCount: 2

image:
  repository: nginx
  tag: latest
  pullPolicy: IfNotPresent

service:
  type: LoadBalancer
  port: 80

resources:
  limits:
    cpu: 500m
    memory: 256Mi
  requests:
    cpu: 250m
    memory: 128Mi
```

### `helm-chart/templates/deployment.yaml`
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Release.Name }}
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      app: {{ .Release.Name }}
  template:
    metadata:
      labels:
        app: {{ .Release.Name }}
    spec:
      containers:
      - name: app
        image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
        imagePullPolicy: {{ .Values.image.pullPolicy }}
        ports:
        - containerPort: {{ .Values.service.port }}
```

## Deployment Testing
### Deploy to Dev
```sh
git checkout dev
git add .
git commit -m "Update app"
git push origin dev
```
✅ Deploys to **Dev AWS EKS cluster** using Helm

### Deploy to UAT
```sh
git checkout uat
git add .
git commit -m "Update for UAT"
git push origin uat
```
✅ Deploys to **UAT AWS EKS cluster** using Helm

### Deploy to Prod
```sh
git checkout main
git checkout -b release/v1.0
git push origin release/v1.0
```
✅ Deploys to **Prod AWS EKS cluster** using Helm

## Conclusion
✅ **Uses Helm for Kubernetes deployments**  
✅ **Automates AWS EKS deployments**  
✅ **Multi-region, multi-environment CI/CD**  

---
Would you like to add **Canary Deployments, Rollbacks, or Secrets Management?** 🚀

