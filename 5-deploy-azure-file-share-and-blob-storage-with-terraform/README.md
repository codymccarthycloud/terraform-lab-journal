# Deploy an Azure File Share and Blob Storage with Terraform

## Overview
This lab demonstrates how to deploy an Azure Storage Account with both a **Blob Container** and a **File Share** using Terraform in the Azure Cloud Shell.

---

## Learning Objectives
- Log in to the Azure portal and launch the Cloud Shell
- Create a `.tf` file with Terraform configuration
- Deploy a blob container and file share using Terraform
- Verify resources in the Azure portal

---

## Steps

### 1. Set Up Azure CLI in Cloud Shell

- Click the **Cloud Shell** icon at the top
- Select **Bash**
- Click **Show Advanced Settings**
  - Use the same location as the lab-provided Resource Group
  - Leave Resource Group and Storage Account as defaults
  - If needed, create a new storage account with a unique name

---

### 2. Create Terraform Configuration

Create a file named `lab.tf` with the following content:

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

resource "azurerm_storage_account" "storage" {
  name                     = "newfileandblob4lab"  # Use a unique name
  resource_group_name      = "183-xxxxxxx"         # Replace with your lab RG name
  location                 = "East US"
  account_tier             = "Standard"
  account_replication_type = "LRS"
}

resource "azurerm_storage_container" "blob" {
  name                  = "blobcontainer4lab"
  storage_account_name = azurerm_storage_account.storage.name
  container_access_type = "private"
}

resource "azurerm_storage_blob" "blob" {
  name                   = "TerraformBlob"
  storage_account_name   = azurerm_storage_account.storage.name
  storage_container_name = azurerm_storage_container.blob.name
  type                   = "Block"
}

resource "azurerm_storage_share" "fileshare" {
  name                  = "terraformshare"
  storage_account_name = azurerm_storage_account.storage.name
  quota                 = 50
}
```

---

## 3. Deploy Resources with Terraform
In the Cloud Shell:

```bash
terraform init
terraform plan
terraform apply
Review the output
```

Confirm with yes when prompted

---

## 4. Verify Deployment
In the Azure portal:
- Go to Storage Accounts
- Open the newly created account
- Check Containers for blobcontainer4lab
- Check File shares for terraformshare

### Completion Checklist
-Cloud Shell configured
- lab.tf file created and uploaded
- Terraform commands executed
- Blob container and file share verified

## Lab Output Screenshots

![1](https://github.com/user-attachments/assets/496f2d3e-5b49-4c9a-a83f-a197dec503a2)
![2](https://github.com/user-attachments/assets/f44e1ea6-2919-47fc-928b-a09058c860fe)
![3](https://github.com/user-attachments/assets/372c3dc5-ac0c-43e0-9c52-cb2385b3d5e5)
![4](https://github.com/user-attachments/assets/0dbfe273-696d-4774-9b23-0d2c9a1f74e8)
![5](https://github.com/user-attachments/assets/51d2aad2-0aec-4609-83d4-b23b50e955d9)
![6](https://github.com/user-attachments/assets/a23bc7a4-0dfd-42ba-b602-905306c4cf82)
![7](https://github.com/user-attachments/assets/774856d3-a2b4-4a5e-8f9f-560f3f206852)
![8](https://github.com/user-attachments/assets/70a02fa7-4a4f-4d31-9964-83a3aed6a5fd)
![9](https://github.com/user-attachments/assets/e60a6758-5d5d-48ab-ae56-5f12f566b5f3)
