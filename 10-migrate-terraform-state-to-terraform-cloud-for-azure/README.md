# Migrate Terraform State to Terraform Cloud for Azure

Import existing Azure resources into Terraform, inspecting local state, and migrating that state to Terraform Cloud.

## Import Existing Azure Resources into Terraform
### Log in to Azure
```bash
az login
```

### Initialize Terraform
```bash
terraform init
```

---

## Import Azure Resource Group into state
terraform import azurerm_resource_group.rg <resource_id>
Find the resource ID in the Azure Portal under Resource Group → Properties → Resource ID

### Inspect the Local State File
```bash
terraform show
```
This displays the imported resource group and confirms the state file was created locally.

## Configure Terraform Cloud Workspace
Log in to Terraform Cloud

Navigate to Projects & Workspaces → New → Workspace

Select CLI-driven workflow

Name the workspace: remotestate

Copy the generated cloud block and paste it into your main.tf:

```hcl
terraform {
  cloud {
    organization = "your-org-name"

    workspaces {
      name = "remotestate"
    }
  }
}
```

## Authenticate & Migrate State
```bash
# Authenticate with Terraform Cloud
terraform login
```

### Re-initialize to trigger migration
```bash
terraform init
```

When prompted:

Code
Do you want to migrate your existing state to Terraform Cloud?
Type yes.

Verify Migration in Terraform Cloud
Go to your workspace in Terraform Cloud

Navigate to States

Confirm the state file is present and versioned

Clean Up Local State
```bash
rm terraform.tfstate*
This removes the local state file and backup to avoid conflicts.
```

---

## Summary
You’ve successfully:
Imported Azure resources into Terraform
Migrated local state to Terraform Cloud
Secured and centralized your infrastructure state
This setup enables collaboration, auditability, and secure automation for enterprise-grade cloud operations.

## Lab Output Screenshots

![1](https://github.com/user-attachments/assets/2b6bcd15-bd73-4ec9-aaf9-24079688e005)
![2](https://github.com/user-attachments/assets/d14772e7-b032-42ba-9819-51154ad85ea7)
![3](https://github.com/user-attachments/assets/276694a3-013f-46af-a517-57745b1f81c8)
![4](https://github.com/user-attachments/assets/74efda74-3e1c-4a3e-b325-12c3b8d8e656)
![5](https://github.com/user-attachments/assets/f09e28a8-a656-460c-9dff-70bb8f1f5aae)
![6](https://github.com/user-attachments/assets/3c702b5b-1054-4554-9ec2-3f87614bbd02)
![7](https://github.com/user-attachments/assets/fa3dd13b-c79f-4479-b1e6-ff6f9b4657c4)
![8](https://github.com/user-attachments/assets/fc832047-9420-426d-92e7-199b7c71ff27)
![9](https://github.com/user-attachments/assets/92d6a861-f3bd-4f5e-a271-179da907e12c)
![10](https://github.com/user-attachments/assets/5788ecc4-668c-49bc-877e-d25cecba1189)
![11](https://github.com/user-attachments/assets/560da715-d457-4664-9a15-17d252633db9)
![12](https://github.com/user-attachments/assets/f4a9a3bc-a9df-433b-9d46-919041e2e21d)
![13](https://github.com/user-attachments/assets/7ede414f-6f94-4d91-8c30-6ca5c758f231)
![14](https://github.com/user-attachments/assets/19a392a8-5062-47ab-811e-b2e06545c1cd)
![16](https://github.com/user-attachments/assets/4a923258-024f-4e5c-afb3-b5cc5138732f)
![15](https://github.com/user-attachments/assets/911c441d-79e8-4f69-8473-4f636c4848c7)
![17](https://github.com/user-attachments/assets/166acbba-896c-445e-a537-0e6cea8dd76f)
![18](https://github.com/user-attachments/assets/bee01edc-8454-4ce5-8377-f84326a8026b)
![19](https://github.com/user-attachments/assets/532086e3-c54b-437d-b9d2-f0f01e69ed15)
![21](https://github.com/user-attachments/assets/093e70b1-d039-4c9b-b65a-5b9a397e86b1)
![22](https://github.com/user-attachments/assets/0fcd5ce5-164a-4b36-9aed-1c78e63236dc)
![23](https://github.com/user-attachments/assets/d5de4548-8a59-4356-a7f1-91b7529fcfbf)
![24](https://github.com/user-attachments/assets/f3c766c1-df32-4eef-80c3-58a374ed9310)
![25](https://github.com/user-attachments/assets/dc46f8c6-0fc8-46e5-9fdb-b02a0a37bb96)
![26](https://github.com/user-attachments/assets/f5a7b5de-21aa-4470-8fa9-4f23a03632e5)
![27](https://github.com/user-attachments/assets/fe450731-631f-49aa-a781-cd3cd4e62154)
![33](https://github.com/user-attachments/assets/c0b3bc4d-4295-4fc8-a743-94a6194f4038)
![28](https://github.com/user-attachments/assets/df3eff40-0a50-4560-a1bc-dddab8eb12a8)
![29](https://github.com/user-attachments/assets/2d54052d-1c97-4a94-9767-d58cf02b152f)
![30](https://github.com/user-attachments/assets/3a371d79-5bcd-4d61-8c65-ea6fe1e953af)
![34](https://github.com/user-attachments/assets/6e44e974-d402-45af-abaa-5a2f6fdb696d)
![31](https://github.com/user-attachments/assets/5ad3c750-08f2-41ae-b106-4891cacdcc87)
![32](https://github.com/user-attachments/assets/d33e7de6-f17c-4ad4-8d06-b3722aba7bfc)




