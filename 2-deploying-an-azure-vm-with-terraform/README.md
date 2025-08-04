# Deploying an Azure VM with Terraform

---

## Step 1: Create and Upload the Terraform File

1. Upload the provided Terraform `.tf` configuration code into Azure Bash.
2. Modify the following line with your lab’s resource group name:
```hcl
   resource_group_name = "187-c756d714-deploying-an-azure-vm-with-terraform"
```

```Hcl
terraform {

  required_version = ">=0.12"

  required_providers {
    azurerm = {
      source = "hashicorp/azurerm"
      version = "~>2.0"
    }
  }
}

provider "azurerm" {
  skip_provider_registration = "true"
  features {}
}

# Create virtual network
resource "azurerm_virtual_network" "TFNet" {
    name                = "TFVnet"
    address_space       = ["10.0.0.0/16"]
    location            = "East US"
    resource_group_name = "187-c756d714-deploying-an-azure-vm-with-terraform"

    tags = {
        environment = "Terraform VNET"
    }
}
# Create subnet
resource "azurerm_subnet" "tfsubnet" {
    name                 = "default"
    resource_group_name  = "187-c756d714-deploying-an-azure-vm-with-terraform"
    virtual_network_name = azurerm_virtual_network.TFNet.name
    address_prefixes     = ["10.0.1.0/24"]
}

#Deploy Public IP
resource "azurerm_public_ip" "example" {
  name                = "pubip1"
  location            = "East US"
  resource_group_name = "187-c756d714-deploying-an-azure-vm-with-terraform"
  allocation_method   = "Dynamic"
  sku                 = "Basic"
}

#Create NIC
resource "azurerm_network_interface" "example" {
  name                = "robot-nic"
  location            = "East US"
  resource_group_name = "187-c756d714-deploying-an-azure-vm-with-terraform"

    ip_configuration {
    name                          = "ipconfig1"
    subnet_id                     = azurerm_subnet.tfsubnet.id
    private_ip_address_allocation = "Dynamic"
    public_ip_address_id          = azurerm_public_ip.example.id
  }
}

#Create Boot Diagnostic Account
resource "azurerm_storage_account" "sa" {
  name                      = "robodiags4tflab"
  resource_group_name       = "187-c756d714-deploying-an-azure-vm-with-terraform"
  location                  = "East US"
   account_tier             = "Standard"
   account_replication_type = "LRS"

   tags = {
    environment = "Boot Diagnostic Storage"
    CreatedBy = "Admin"
   }
  }

#Create Virtual Machine
resource "azurerm_virtual_machine" "example" {
  name                  = "robot"
  location              = "East US"
  resource_group_name   = "187-c756d714-deploying-an-azure-vm-with-terraform"
  network_interface_ids = [azurerm_network_interface.example.id]
  vm_size               = "Standard_B1s"
  delete_os_disk_on_termination = true
  delete_data_disks_on_termination = true

  storage_image_reference {
    publisher = "Canonical"
    offer     = "UbuntuServer"
    sku       = "16.04-LTS"
    version   = "latest"
  }

  storage_os_disk {
    name              = "osdisk1"
    disk_size_gb      = "128"
    caching           = "ReadWrite"
    create_option     = "FromImage"
    managed_disk_type = "Standard_LRS"
  }

  os_profile {
    computer_name  = "robot"
    admin_username = "vmadmin"
    admin_password = "Password12345!"
  }

  os_profile_linux_config {
    disable_password_authentication = false
  }

boot_diagnostics {
        enabled     = "true"
        storage_uri = azurerm_storage_account.sa.primary_blob_endpoint
    }
}
```

---

## Step 2: Deploy the VM Using Terraform CLI

```bash
terraform init
terraform plan
terraform apply
```

- Confirm deployment by typing "yes" when prompted.
- Terraform will provision the following:
  - Virtual Network (VNet)
  - Subnet
  - Public IP
  - Network Interface (NIC)
  - Boot Diagnostics Storage Account
  - Virtual Machine (Ubuntu)

---

## Step 3: Validate Deployment in Azure Portal

- Go to the Azure Portal → Resource Group → VNet
- Confirm the **Subnet** was created under the VNet
- Go to **Network Interface (`robot-nic`)** → IP Configurations:
  - Private IP assigned (e.g., 10.0.0.4)
  - Public IP assigned
  - NIC connected to correct Subnet

---

## Step 4: Confirm VM Boot via Serial Console

- Navigate to the deployed VM → **Serial Console**
- Confirm console output shows Ubuntu login screen or diagnostics prompt
- VM status should display **Running**
- Optional: Check Boot Diagnostics tab for screenshots and log files stored in the associated storage account

---

## Conclusion

Terraform successfully deployed and configured:
- VM with public/private IPs
- Subnet within VNet
- Network Interface with IP configuration
- Boot diagnostics via linked storage account

Deployment confirmed via CLI and Portal.  
Serial Console verifies successful boot.  
Lab completed with clean infrastructure state.
Learn to create .tf file deployments that work across multiple cloud platforms.

---

## Lab Output Screenshots

![test vm tf](https://github.com/user-attachments/assets/fb5451fc-3576-45d9-bf68-9ae472419609)
![confirming the subnet has been added to the VNET](https://github.com/user-attachments/assets/dec0a331-1551-428d-a994-169d7b824519)
![private and public ip assigned to the NIC network interface](https://github.com/user-attachments/assets/d893cadd-fa5d-42a8-8e53-a6a2d593244a)
![vm serial console check if it is running](https://github.com/user-attachments/assets/e75b16c3-f961-4dd2-b118-b35849379111)
