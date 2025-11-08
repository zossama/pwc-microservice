1. Azure Setup


az login
az account set --subscription "c95cf32b-1582-4708-b079-3a00038c98b8"
az account show


2. Infrastructure with Terraform


cd terraform
terraform init
terraform plan
terraform apply
cd ..

az aks get-credentials --resource-group pwc-microservices-rg --name pwc-aks-cluster --overwrite-existing
kubectl get nodes


3. Docker Image


az acr login --name pwcmicroservicesacr
docker build --platform linux/amd64 -t pwcmicroservicesacr.azurecr.io/flask-microservice:latest .
docker push pwcmicroservicesacr.azurecr.io/flask-microservice:latest


4. Kubernetes Deployment


kubectl apply -f k8s/configmap.yaml
kubectl apply -f k8s/deployment.yaml
kubectl apply -f k8s/service.yaml
kubectl apply -f k8s/hpa.yaml
kubectl apply -f k8s/ingress.yaml

kubectl get pods
kubectl get services
kubectl get deployments


5. Monitoring Stack


helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm install prometheus prometheus-community/kube-prometheus-stack --namespace monitoring --create-namespace

kubectl get pods -n monitoring
kubectl get svc -n monitoring

kubectl --namespace monitoring port-forward svc/prometheus-grafana 3000:80
kubectl get secret --namespace monitoring prometheus-grafana -o jsonpath="{.data.admin-password}" | base64 --decode


6. GitHub CI/CD Setup


az ad sp create-for-rbac --name "github-actions-pwc" --role contributor --scopes /subscriptions/c95cf32b-1582-4708-b079-3a00038c98b8/resourceGroups/pwc-microservices-rg --sdk-auth > AZURE_CREDENTIALS.json

gh repo create pwc-microservice --public --source=. --remote=origin
git remote remove origin
git remote add origin https://github.com/zossama/pwc-microservice.git
git add .
git commit -m "Initial commit"
git push -u origin main

gh secret set AZURE_CREDENTIALS < AZURE_CREDENTIALS.json --repo zossama/pwc-microservice
gh workflow run ci-cd.yaml --repo zossama/pwc-microservice

