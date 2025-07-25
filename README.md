# Step-by-Step Guide: Deploying an Azure Storage Account with Terraform and GitHub Actions  

This guide will walk you through creating an **Azure Storage Account** using Terraform and deploying it via a **GitHub Actions Pipeline**.  

---

## **Prerequisites**  
Before starting, ensure you have:  
1. An **Azure account** ([Sign up for free](https://azure.microsoft.com/en-us/free/)).  
2. **Azure CLI** installed ([Installation guide](https://learn.microsoft.com/en-us/cli/azure/install-azure-cli)).  
3. A **GitHub account** ([Sign up here](https://github.com/)).  
4. **Terraform** installed ([Download Terraform](https://developer.hashicorp.com/terraform/downloads)).  

---

## **Step 1: Set Up Azure Authentication**  
We need to authenticate Terraform with Azure using a **Service Principal**.  

### **1.1 Login to Azure CLI**  
Run in terminal:  
```bash
az login
```  
(A browser window will open for authentication.)  

### **1.2 Create a Service Principal**  
Run:  
```bash
az ad sp create-for-rbac --name "Terraform-GitHub-Actions" --role Contributor --scopes /subscriptions/<SUBSCRIPTION_ID> --sdk-auth
```  
(Replace `<SUBSCRIPTION_ID>` with your Azure Subscription ID.)  

### **1.3 Save the JSON Output**  
This contains credentials (`clientId`, `clientSecret`, `tenantId`, etc.). You’ll need them later.  

---

## **Step 2: Create a Terraform Configuration**  
We’ll write a simple Terraform file to deploy an Azure Storage Account.  

### **2.1 Initialize a Terraform Project**  
Create a new folder and files:  
```bash
mkdir azure-storage-terraform
cd azure-storage-terraform
touch main.tf variables.tf outputs.tf
```  

### **2.2 Edit `main.tf`**  
```hcl
terraform {
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 3.0"
    }
  }
}

provider "azurerm" {
  features {}
}

resource "azurerm_resource_group" "rg" {
  name     = "tf-storage-demo-rg"
  location = "eastus"
}

resource "azurerm_storage_account" "storage" {
  name                     = "tfstorageaccountdemo123" # Must be globally unique
  resource_group_name      = azurerm_resource_group.rg.name
  location                 = azurerm_resource_group.rg.location
  account_tier             = "Standard"
  account_replication_type = "LRS"
}
```  

### **2.3 (Optional) Add `variables.tf`**  
```hcl
variable "location" {
  description = "Azure region for resources"
  default     = "eastus"
}
```  

### **2.4 (Optional) Add `outputs.tf`**  
```hcl
output "storage_account_name" {
  value = azurerm_storage_account.storage.name
}
```  

---

## **Step 3: Set Up GitHub Secrets**  
We’ll store Azure credentials securely in GitHub.  

### **3.1 Go to Your GitHub Repository**  
- Navigate to **Settings** → **Secrets and variables** → **Actions**.  
- Click **New repository secret**.  

### **3.2 Add These Secrets**  
Use the JSON output from Step 1.2:  
- `AZURE_CLIENT_ID` → `clientId`  
- `AZURE_CLIENT_SECRET` → `clientSecret`  
- `AZURE_TENANT_ID` → `tenantId`  
- `AZURE_SUBSCRIPTION_ID` → `subscriptionId`  

---

## **Step 4: Create GitHub Actions Workflow**  
We’ll automate Terraform deployment using GitHub Actions.  

### **4.1 Create Workflow File**  
In your repo, create:  
```
.github/workflows/terraform-deploy.yml
```  

### **4.2 Edit the Workflow File**  
```yaml
name: 'Terraform Azure Storage Deployment'

on:
  push:
    branches: [ "main" ]
  workflow_dispatch:

env:
  ARM_CLIENT_ID: ${{ secrets.AZURE_CLIENT_ID }}
  ARM_CLIENT_SECRET: ${{ secrets.AZURE_CLIENT_SECRET }}
  ARM_TENANT_ID: ${{ secrets.AZURE_TENANT_ID }}
  ARM_SUBSCRIPTION_ID: ${{ secrets.AZURE_SUBSCRIPTION_ID }}

jobs:
  terraform:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Setup Terraform
        uses: hashicorp/setup-terraform@v2

      - name: Terraform Init
        run: terraform init

      - name: Terraform Plan
        run: terraform plan

      - name: Terraform Apply
        run: terraform apply -auto-approve
```  

---

## **Step 5: Deploy via GitHub Actions**  
1. **Commit and Push** your code to GitHub.  
2. Go to **Actions** tab in your repo.  
3. Select the workflow and click **Run workflow**.  
4. Monitor the execution—it will apply Terraform and create the storage account.  

---

## **Step 6: Verify Deployment**  
- Check the **Azure Portal** → **Resource Groups** → `tf-storage-demo-rg`.  
- You should see the storage account created.  

---

## **Cleanup (Optional)**  
To destroy resources:  
1. Manually run:  
```bash
terraform destroy
```  
Or modify the GitHub workflow to include a `destroy` step.  

---

## **Conclusion**  
You’ve successfully:  
✅ Created a Terraform config for Azure Storage.  
✅ Automated deployment using GitHub Actions.  
✅ Verified the deployment in Azure.  

This demo is ready for your session! 🚀 Let me know if you need any refinements.