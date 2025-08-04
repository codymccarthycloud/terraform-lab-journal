# Deploy an Azure Storage Account with Terraform

## Lab Overview
This lab demonstrates how to provision an Azure Storage Account using Terraform inside the Azure Cloud Shell (Bash). 
You'll also configure tagging for better resource tracking.

---

## Azure CLI Environment Setup
1. Open the Azure Portal and select the **Command Line** button.
2. Choose **Bash** when prompted.
3. Click **Show Advanced Settings**:
   - Use the lab-provided location.
   - Leave the Resource Group and Storage Account as **defaults**.
   - Name the File Share (e.g., `cloudshell`).
4. Click **Create Storage** to launch the Bash environment.

---

## Create Terraform Configuration File
Create a file named `lab.tf` with the following code (replace placeholders):

```hcl
terraform {
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "2.93.0"
    }
    azuread = {
      source = "hashicorp/azuread"
    }
  }
}

provider "azurerm" {
  features {}
  skip_provider_registration = true
}

resource "azurerm_storage_account" "lab" {
  name                     = "storage4terraformlab"         # Replace with unique name
  resource_group_name      = "156-21b1c17f-deploy-lab"      # Replace with lab’s RG
  location                 = "East US"
  account_tier             = "Standard"
  account_replication_type = "LRS"
  tags = {
    environment = "Terraform Storage"
    CreatedBy   = "Admin"
  }
}
```
---

## Upload & Deploy

1. Use **Upload/Download** above the Azure CLI session to upload `lab.tf`.  
2. Initialize Terraform:  
```bash
   terraform init
   terraform plan
   terraform apply
```

## Verify Deployment

1. In the Azure Portal, go to Resource Groups.
  - Click Refresh.
   - Open the newly created Storage Account.
   - Confirm the tags: environment = Terraform Storage
   - CreatedBy = Admin

---

## Conclusion: Deployment Records Live in Azure CLI Activity Logs 
### **Enable Diagnostics with a Log Analytics Workspace to view in Monitor > Activity Log**
  
  - CLI Cloud Shell itself doesn’t maintain a record of what resources you deployed. It only keeps your files and command history.
  - To audit deployments, check the Azure Activity Log in the portal:
  - Navigate to Monitor > Activity Log.
  - Filter by the subscription, resource group, or “Create” operations to see your Terraform-provisioned resources.

## Lab Output Screenshots

![1 tf file creation edit tf file configure cli in azure](https://github.com/user-attachments/assets/493e1e05-c5a7-4a0e-8622-f3eb87ca2968)
![2 upload tf storage account file ](https://github.com/user-attachments/assets/1d0a5bfc-1af4-4afd-b055-a8cd0c9d279e)
![3 check for the uploaded tf file using ls in cli run terraform init](https://github.com/user-attachments/assets/38d51929-df6a-426a-9d14-62d8800702c6)
![4 terraform plan view confirmation of the tf file contents](https://github.com/user-attachments/assets/d601cef8-6a54-4cbc-aa15-5e4635a12c79)
![5 terraform apply deployment](https://github.com/user-attachments/assets/53697528-eabd-414b-8373-238f55760c92)
![6 custom tf file name tfdeployedstorage showing in the resource group after deployment through the cli](https://github.com/user-attachments/assets/532143f9-9370-47ed-b256-466e53ff6b32)


