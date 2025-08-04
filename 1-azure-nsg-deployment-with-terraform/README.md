# Azure NSG Deployment with Terraform 

## Lab Overview
This lab demonstrates deploying an Azure Network Security Group (NSG) using Terraform through Azure Cloud Shell. 
It covers file creation, uploading to Cloud Shell, verifying contents, executing Terraform commands, and confirming deployment via the Azure Portal.

---

## Step 1: Create the Terraform File Locally

- Open a text editor (Notepad, VS Code, etc.)
- Paste the provided NSG Terraform configuration code
- Save the file as `lab.tf` ( Be sure it's `.tf` not `.txt`)
  - In Notepad: Select "All Files" from the Save dialog
  - Type: `lab.tf`

```Hcl
terraform {
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "=3.0.0"
    }
  }
}

# Configure the Microsoft Azure Provider
provider "azurerm" {
  features {}

  skip_provider_registration = true
}

resource "azurerm_network_security_group" "nsg" {
  name                = "LabNSG"
  location            = "East US"
  resource_group_name = "185-c61e6718-create-azure-nsgs-with-terraform"
}

resource "azurerm_network_security_rule" "example1" {
  name                        = "Web80"
  priority                    = 1001
  direction                   = "Inbound"
  access                      = "Allow"
  protocol                    = "Tcp"
  source_port_range           = "80"
  destination_port_range      = "80"
  source_address_prefix       = "*"
  destination_address_prefix  = "*"
  resource_group_name         = "185-c61e6718-create-azure-nsgs-with-terraform"
  network_security_group_name = azurerm_network_security_group.nsg.name
}

resource "azurerm_network_security_rule" "example2" {
  name                        = "Web8080"
  priority                    = 1000
  direction                   = "Inbound"
  access                      = "Deny"
  protocol                    = "Tcp"
  source_port_range           = "8080"
  destination_port_range      = "8080"
  source_address_prefix       = "*"
  destination_address_prefix  = "*"
  resource_group_name         = "185-c61e6718-create-azure-nsgs-with-terraform"
  network_security_group_name = azurerm_network_security_group.nsg.name
}

  resource "azurerm_network_security_rule" "example4" {
  name                        = "SSH"
  priority                    = 1100
  direction                   = "Inbound"
  access                      = "Allow"
  protocol                    = "Tcp"
  source_port_range           = "*"
  destination_port_range      = "22"
  source_address_prefix       = "*"
  destination_address_prefix  = "*"
  resource_group_name         = "185-c61e6718-create-azure-nsgs-with-terraform"
  network_security_group_name = azurerm_network_security_group.nsg.name
}

  resource "azurerm_network_security_rule" "example3" {
  name                        = "Web80Out"
  priority                    = 1000
  direction                   = "Outbound"
  access                      = "Deny"
  protocol                    = "Tcp"
  source_port_range           = "80"
  destination_port_range      = "80"
  source_address_prefix       = "*"
  destination_address_prefix  = "*"
  resource_group_name         = "185-c61e6718-create-azure-nsgs-with-terraform"
  network_security_group_name = azurerm_network_security_group.nsg.name
}
```

---

## Step 2: Upload the File into Azure Cloud Shell

- Launch **Azure Cloud Shell** from the top menu (select **Bash**)
- When prompted, select **Show Advanced Settings**
  - Use pre-selected Resource Group and Storage Account
  - In "File Share", choose **Create new** and name it `console`
  - Click **Attach Storage**

- After the shell initializes, click the **Upload/Download** button above the terminal
  - Upload your `lab.tf` file from your local machine

---

## Step 3: Verify the Upload

- In the Cloud Shell terminal, confirm the file exists:
```bash
  ls
```

---

## Step 4: Terraform Deployment with CLI

```bash
terraform init
terraform plan
terraform apply
```

---

# Azure NSG Deployment – Lab Completion & Output

##  Conclusion

This Terraform lab successfully deployed an Azure Network Security Group (NSG) using `lab.tf` in Cloud Shell. 
All setup was completed via Bash with manual file upload and confirmation of contents using basic CLI commands.
The NSG was verified in the Azure portal under the designated resource group, with inbound/outbound rules created as defined in the configuration file.

--- 

## Lab Output Screenshots

![creating a lab tf file for NSG with Terraform in azure cli](https://github.com/user-attachments/assets/ef7c620a-ef88-49ac-bcd3-89e823df5f1b)
![upload a file into bash and view it](https://github.com/user-attachments/assets/cbda7d24-0197-43bf-9060-45f158ed76a9)
![terraform init in cli](https://github.com/user-attachments/assets/04cfe555-e350-4855-b7ba-c20caf4ff43d)
![terraform plan view uploaded Lab tf](https://github.com/user-attachments/assets/7e60f44d-6347-4463-ac28-0be1840ffe75)
![terraform apply azure refresh resource group to confirm NSG](https://github.com/user-attachments/assets/ffc3c194-9630-45da-9137-cc8e3bbbd2f2)
