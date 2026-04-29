### Multi-Cluster Microservices Deployment with ArgoCD (EKS + ECR + GitOps)

### Introduction

This project demonstrates how to deploy and manage microservices across multiple Kubernetes clusters using GitOps principles with ArgoCD.

It uses:
- Amazon EKS (multi-cluster)
- ArgoCD ApplicationSet
- AWS ECR (container registry)
- GitHub Actions (CI/CD)

Architecture:

ArgoCD acts as the control plane, continuously syncing application state from GitHub to multiple Kubernetes clusters.

Each service is deployed automatically to all clusters using ApplicationSet.

### Create ECR Repositories

Run:

```
Create ECR Repositories

aws ecr create-repository --repository-name service-a --region us-east-1
aws ecr create-repository --repository-name service-b --region us-east-1
```

#### Verify

```
aws ecr describe-repositories
```

### Authenticate Docker to ECR

```
aws ecr get-login-password --region us-east-1 | \
docker login --username AWS --password-stdin <ACCOUNT_ID>.dkr.ecr.us-east-1.amazonaws.com
```

**Build Docker Images**
```
docker build -t service-a ./service-a
docker build -t service-b ./service-b
```

Tag Images for ECR:

```
docker tag service-a:latest <ACCOUNT_ID>.dkr.ecr.us-east-1.amazonaws.com/service-a:latest
docker tag service-b:latest <ACCOUNT_ID>.dkr.ecr.us-east-1.amazonaws.com/service-b:latest
```

Push Images to ECR:

```
docker push <ACCOUNT_ID>.dkr.ecr.us-east-1.amazonaws.com/service-a:latest
docker push <ACCOUNT_ID>.dkr.ecr.us-east-1.amazonaws.com/service-b:latest
```
### Use ECR Images in Kubernetes

Update your deployment:

```
containers:
  - name: service-a
    image: <ACCOUNT_ID>.dkr.ecr.us-east-1.amazonaws.com/service-a:latest
```    
Note:Your EKS nodes must have permission to pull images.

Attach policy:
```
AmazonEC2ContainerRegistryReadOnly
```

#### Challenges & Fixes (ECR-Specific)

- Docker Build Path Error
  ```
  path "./service-a" not found
  ```
  **Fix**

- Ensured correct repo structure:
```
service-a/
service-b/
```

#### Using latest Tag (Bad Practice)
**Problem**

- Hard to track versions
ArgoCD may not detect updates
 **Fix**

- Use commit SHA:

```
service-a:$GITHUB_SHA
```

**Apps Not Showing**
```
No resources found
```
**Root Cause**

- Git repo not updated with deployment.yaml
- ApplicationSet couldn’t find valid manifests

**Only One Cluster Deploying**
**Root Cause**

- cluster1 not registered in ArgoCD
 
  **Fix**

```
argocd cluster add cluster1
```

**Image Updates Not Reflecting**
**Root Cause**
- Using latest
- ArgoCD doesn’t track registry changes

**Fix**
Use:
```
service-a:<commit-sha>
```