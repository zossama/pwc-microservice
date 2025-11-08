# GitHub Actions CI/CD Workflows

This directory contains GitHub Actions workflows for automating the build, test, and deployment process.

## Workflows

### 1. CI/CD Pipeline (`ci-cd.yaml`)
**Triggers:** Push to main, Pull Request, Manual dispatch

**What it does:**
- Builds Docker image
- Pushes to Azure Container Registry (ACR)
- Deploys to Azure Kubernetes Service (AKS)
- Verifies deployment success

**Jobs:**
1. **build-and-push**: Builds and pushes Docker image to ACR
2. **deploy-to-aks**: Deploys the application to AKS cluster

### 2. Terraform Infrastructure (`terraform.yaml`)
**Triggers:** Manual dispatch only

**What it does:**
- Manages Azure infrastructure using Terraform
- Supports: plan, apply, destroy actions

**Usage:**
1. Go to Actions tab in GitHub
2. Select "Terraform Infrastructure"
3. Click "Run workflow"
4. Choose action (plan/apply/destroy)

### 3. Pull Request Checks (`pr-checks.yaml`)
**Triggers:** Pull Request to main

**What it does:**
- Validates Docker build
- Tests Docker image locally
- Validates Kubernetes manifests
- Validates Terraform configuration

## Prerequisites

### 1. Azure Service Principal

Create an Azure Service Principal for GitHub Actions:

```bash
# Login to Azure
az login

# Create Service Principal
az ad sp create-for-rbac \
  --name "github-actions-pwc-microservices" \
  --role contributor \
  --scopes /subscriptions/{subscription-id}/resourceGroups/pwc-microservices-rg \
  --sdk-auth
```

This will output JSON credentials. Copy the entire JSON output.

### 2. GitHub Secrets

Add the following secrets to your GitHub repository:

**Settings → Secrets and variables → Actions → New repository secret**

#### Required Secret:

**`AZURE_CREDENTIALS`**
```json
{
  "clientId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "clientSecret": "your-client-secret",
  "subscriptionId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "tenantId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
}
```

## Workflow Usage

### First-Time Setup

1. **Create Azure Infrastructure:**
   ```bash
   # Run locally first
   cd terraform
   terraform init
   terraform apply
   ```

   Or use the Terraform workflow:
   - Go to Actions → Terraform Infrastructure
   - Run workflow with action: "apply"

2. **Deploy Application:**
   - Push code to main branch
   - CI/CD workflow runs automatically
   - Application deploys to AKS

### Continuous Deployment

Once setup is complete:

1. **Make code changes**
2. **Create Pull Request**
   - PR Checks workflow runs automatically
   - Validates Docker build, K8s manifests, Terraform
3. **Merge to main**
   - CI/CD workflow runs automatically
   - Builds new Docker image
   - Deploys to AKS

### Manual Deployment

To manually trigger deployment:

1. Go to Actions tab
2. Select "CI/CD Pipeline"
3. Click "Run workflow"
4. Select branch and run

## Environment Variables

The workflows use these environment variables (defined in workflow files):

| Variable | Value | Description |
|----------|-------|-------------|
| `ACR_NAME` | `pwcmicroservicesacr` | Azure Container Registry name |
| `AKS_CLUSTER_NAME` | `pwc-aks-cluster` | AKS cluster name |
| `AKS_RESOURCE_GROUP` | `pwc-microservices-rg` | Azure Resource Group |
| `IMAGE_NAME` | `flask-microservice` | Docker image name |

**To customize:** Edit these values in the workflow files.

## Workflow Outputs

### CI/CD Pipeline
- Docker image SHA and tags
- Kubernetes deployment status
- Service LoadBalancer IP
- Pod status

### Terraform
- Terraform plan output
- Created resource details
- ACR login server
- AKS credentials command

## Troubleshooting

### Workflow fails with authentication error
- Verify `AZURE_CREDENTIALS` secret is set correctly
- Ensure Service Principal has correct permissions

### Docker build fails
- Check Dockerfile syntax
- Verify requirements.txt is valid

### Deployment fails
- Ensure AKS cluster exists
- Verify ACR integration with AKS
- Check kubectl credentials

### View workflow logs
1. Go to Actions tab
2. Click on failed workflow run
3. Expand failed job/step to see logs

## Security Best Practices

1. **Never commit secrets** - Use GitHub Secrets
2. **Limit Service Principal permissions** - Use least privilege
3. **Review PR checks** - Don't merge failing PRs
4. **Monitor workflow runs** - Check for suspicious activity
5. **Rotate credentials** - Update Service Principal regularly

## Customization

### Change deployment strategy

Edit `ci-cd.yaml`:

```yaml
# Rolling update (default)
kubectl apply -f k8s/

# Blue-green deployment
kubectl apply -f k8s/ --record
kubectl set image deployment/flask-microservice flask-app=image:new-tag

# Canary deployment
# Create separate canary deployment with fewer replicas
```

### Add testing stage

Add to `ci-cd.yaml` before deploy:

```yaml
test:
  name: Run Tests
  runs-on: ubuntu-latest
  needs: build-and-push
  steps:
    - name: Run integration tests
      run: |
        # Add your tests here
        pytest tests/
```

### Add notifications

Add to end of jobs:

```yaml
    - name: Notify Slack
      if: always()
      uses: slackapi/slack-github-action@v1
      with:
        webhook-url: ${{ secrets.SLACK_WEBHOOK }}
        payload: |
          {
            "text": "Deployment ${{ job.status }}"
          }
```

## Cost Optimization

To save costs during development:

1. **Destroy infrastructure when not in use:**
   ```bash
   terraform destroy
   ```
   Or use Terraform workflow with "destroy" action

2. **Use smaller AKS nodes:**
   - Edit `terraform/variables.tf`
   - Change `aks_node_size` to `Standard_B2s`

3. **Reduce replica count:**
   - Edit `k8s/deployment.yaml`
   - Change `replicas: 3` to `replicas: 1`

## Next Steps

1. Add automated tests to PR checks
2. Implement proper secrets management (Azure Key Vault)
3. Add monitoring/alerting integration
4. Implement multi-environment deployments (dev/staging/prod)
5. Add security scanning (Trivy, Snyk)
