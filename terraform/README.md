# Terraform Configuration for Azure AKS

This directory contains Terraform configuration to provision Azure infrastructure for the microservices application.

## Prerequisites

1. Azure CLI installed and authenticated
2. Terraform installed (>= 1.0)
3. Azure subscription with appropriate permissions

## Resources Created

- Azure Resource Group
- Azure Container Registry (ACR)
- Azure Kubernetes Service (AKS) cluster with 2 nodes
- Virtual Network and Subnet
- Role assignment (AKS to ACR)

## Usage

1. **Login to Azure:**
   ```bash
   az login
   ```

2. **Initialize Terraform:**
   ```bash
   cd terraform
   terraform init
   ```

3. **Review the plan:**
   ```bash
   terraform plan
   ```

4. **Apply the configuration:**
   ```bash
   terraform apply
   ```

5. **Get AKS credentials:**
   ```bash
   az aks get-credentials --resource-group pwc-microservices-rg --name pwc-aks-cluster
   ```

6. **Verify connection:**
   ```bash
   kubectl get nodes
   ```

## Customization

Copy `terraform.tfvars.example` to `terraform.tfvars` and modify values as needed:

```bash
cp terraform.tfvars.example terraform.tfvars
# Edit terraform.tfvars with your values
```

## Cleanup

To destroy all resources:

```bash
terraform destroy
```

## Outputs

After applying, Terraform will output:
- ACR login server URL
- AKS cluster name
- Command to get kubectl credentials
