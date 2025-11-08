# Kubernetes Manifests

This directory contains Kubernetes configuration files to deploy the Flask microservice application.

## Files

- `configmap.yaml` - Configuration values for the application
- `deployment.yaml` - Defines how to run the Flask application (3 replicas)
- `service.yaml` - LoadBalancer service to expose the application externally
- `ingress.yaml` - Alternative Ingress configuration (requires NGINX Ingress Controller)
- `hpa.yaml` - HorizontalPodAutoscaler for automatic scaling based on CPU/memory

## Prerequisites

1. AKS cluster running (created via Terraform)
2. kubectl configured to connect to your cluster
3. Docker image pushed to ACR

## Deployment Steps

### Step 1: Push Docker Image to ACR

```bash
# Get ACR login server from Terraform output
terraform output acr_login_server

# Login to ACR
az acr login --name pwcmicroservicesacr

# Tag the image
docker tag flask-microservice:latest pwcmicroservicesacr.azurecr.io/flask-microservice:latest

# Push to ACR
docker push pwcmicroservicesacr.azurecr.io/flask-microservice:latest
```

### Step 2: Deploy to Kubernetes

```bash
# Apply all manifests
kubectl apply -f k8s/

# Or apply individually
kubectl apply -f k8s/configmap.yaml
kubectl apply -f k8s/deployment.yaml
kubectl apply -f k8s/service.yaml
kubectl apply -f k8s/hpa.yaml
```

### Step 3: Verify Deployment

```bash
# Check pods
kubectl get pods

# Check service
kubectl get svc flask-service

# Get external IP (may take a few minutes)
kubectl get svc flask-service -w
```

### Step 4: Test the Application

```bash
# Get the external IP
EXTERNAL_IP=$(kubectl get svc flask-service -o jsonpath='{.status.loadBalancer.ingress[0].ip}')

# Test endpoints
curl http://$EXTERNAL_IP/users
curl http://$EXTERNAL_IP/products
```

## Using Ingress (Alternative)

If you prefer Ingress over LoadBalancer:

### Step 1: Install NGINX Ingress Controller

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.8.1/deploy/static/provider/cloud/deploy.yaml
```

### Step 2: Change Service Type

Edit `service.yaml` and change:
```yaml
spec:
  type: ClusterIP  # Change from LoadBalancer to ClusterIP
```

### Step 3: Apply Ingress

```bash
kubectl apply -f k8s/ingress.yaml
```

### Step 4: Get Ingress IP

```bash
kubectl get ingress flask-ingress
```

## Monitoring

```bash
# View logs
kubectl logs -l app=flask-microservice -f

# View HPA status
kubectl get hpa flask-hpa

# Describe deployment
kubectl describe deployment flask-microservice
```

## Cleanup

```bash
# Delete all resources
kubectl delete -f k8s/

# Or delete individually
kubectl delete deployment flask-microservice
kubectl delete service flask-service
kubectl delete ingress flask-ingress
kubectl delete hpa flask-hpa
kubectl delete configmap flask-config
```

## Troubleshooting

### Pods not starting?
```bash
kubectl describe pod <pod-name>
kubectl logs <pod-name>
```

### Can't pull image from ACR?
```bash
# Verify ACR integration
az aks check-acr --name pwc-aks-cluster --resource-group pwc-microservices-rg --acr pwcmicroservicesacr.azurecr.io
```

### Service has no external IP?
```bash
# Check service events
kubectl describe svc flask-service

# Verify load balancer
kubectl get svc -A
```
