# Infasructure as Code with Terraform


## Build Infrastructure with Terraform

Terraform comes pre-installed in Cloud Shell. Begin by creating a configuration file named `main.tf`. 
Terraform recognizes files ending in `.tf` or `.tf.json` as configuration files and will load them when it runs.

### 1. Create the Configuration File
```bash
touch main.tf
```

### 2. Add Terraform Configuration
Open the file in the Cloud Shell Editor and paste the following:
```hcl
terraform {
  required_providers {
    google = {
      source  = "hashicorp/google"
      version = "3.5.0"
    }
  }
}

provider "google" {
  project = "PROJECT_ID"
  region  = "REGION"
  zone    = "ZONE"
}

resource "google_compute_network" "vpc_network" {
  name = "terraform-network"
}
```
> Replace `PROJECT_ID`, `REGION`, and `ZONE` with your actual lab values.

### 3. Initialize Terraform
```bash
terraform init
```

### 4. Apply Configuration
```bash
terraform apply
```
Type `yes` when prompted.

### 5. Verify Deployment
After a few moments, Terraform will complete the creation of the VPC network. You’ll see output like:
```
google_compute_network.vpc_network: Creation complete after 58s [id=terraform-network]
Apply complete! Resources: 1 added, 0 changed, 0 destroyed.
```

You can verify the network in the Google Cloud Console by navigating to **VPC network**. Also, run:
```bash
terraform show
```
to inspect the current state.

## Outcome
You’ve successfully provisioned a VPC network named `terraform-network` using Terraform.

# Task 2: Change Infrastructure with Terraform

In the previous task, you created a basic VPC network. Now you'll modify your configuration to add a VM instance, update its properties, and explore how Terraform handles changes and destructive updates.

## Step-by-Step Instructions

### 1. Add a Compute Instance to `main.tf`
Append the following block to your existing `main.tf`:
```hcl
resource "google_compute_instance" "vm_instance" {
  name         = "terraform-instance"
  machine_type = "e2-micro"

  boot_disk {
    initialize_params {
      image = "debian-cloud/debian-11"
    }
  }

  network_interface {
    network = google_compute_network.vpc_network.name
    access_config {}
  }
}
```

---

### 2. Apply the Configuration
```bash
terraform apply
```
Type `yes` when prompted.

### 3. Add Tags to the Instance
Update the `vm_instance` block to include tags:
```hcl
tags = ["web", "dev"]
```

Then reapply:
```bash
terraform apply
```
Type `yes` to confirm. The `~` symbol indicates an in-place update.

### 4. Make a Destructive Change
Change the boot disk image to trigger a resource replacement:
```hcl
boot_disk {
  initialize_params {
    image = "cos-cloud/cos-stable"
  }
}
```

Apply again:
```bash
terraform apply
```
Type `yes` to confirm. The `-/+` symbol indicates the resource will be destroyed and recreated.

### 5. Destroy the Infrastructure
To clean up all resources:
```bash
terraform destroy
```
Type `yes` to confirm. Terraform will destroy the VM first, then the VPC network, respecting dependency order.

## Outcome
You’ve successfully modified, updated, and destroyed infrastructure using Terraform. You’ve seen how Terraform handles in-place updates, destructive changes, and dependency-aware deletions.

---

# Task 3: Create Resource Dependencies with Terraform

In this task, you'll learn how to define relationships between resources in Terraform and use resource attributes to configure other resources. This is essential for building real-world infrastructure where components depend on one another.

## Step-by-Step Instructions

### 1. Recreate Your Network and Instance
If you've previously destroyed your infrastructure, recreate it:
```bash
terraform apply
```
Type `yes` when prompted.

### 2. Add a Static IP Resource
Append the following block to `main.tf`:
```hcl
resource "google_compute_address" "vm_static_ip" {
  name = "terraform-static-ip"
}
```

This allocates a reserved external IP address for your project.

### 3. Plan the Change
Run:
```bash
terraform plan
```
You’ll see that Terraform plans to create the static IP resource.

### 4. Attach the Static IP to the VM
Update the `network_interface` block inside your `vm_instance` resource:
```hcl
network_interface {
  network = google_compute_network.vpc_network.self_link
  access_config {
    nat_ip = google_compute_address.vm_static_ip.address
  }
}
```

This configuration ensures:
- The static IP is created before the VM.
- The VM uses the reserved IP for NAT.

### 5. Save and Apply the Plan
Run:
```bash
terraform plan -out static_ip
terraform apply static_ip
```
Type `yes` to confirm.

## Outcome
You’ve successfully created resource dependencies in Terraform by linking a VM instance to a reserved static IP. 
Terraform automatically manages the order of operations and ensures dependent resources are provisioned correctly.

---

## Lab Output Screenshots

![2](https://github.com/user-attachments/assets/ae05b54c-3ae9-42b1-94a7-6c04e324ab63)
![3](https://github.com/user-attachments/assets/a26e7285-940c-4db4-9928-138736e273c9)
![4](https://github.com/user-attachments/assets/3b5c4bf0-833d-4b40-abb2-726a84fabb65)
![5](https://github.com/user-attachments/assets/17a2d1b9-a0bd-4913-af4e-e2760ef435d7)
![6](https://github.com/user-attachments/assets/7d8d1d9c-6bcb-4b35-89e3-f7bce002cc58)
![8](https://github.com/user-attachments/assets/61389c5a-7a99-4f2a-ab87-d741a0d8448c)
![7](https://github.com/user-attachments/assets/e1871da8-fd84-47e4-8273-7a2a59376e9f)
![9](https://github.com/user-attachments/assets/bc3aa2a2-f1ad-4ad5-b0bb-1a20d3cde44f)
![10](https://github.com/user-attachments/assets/cd48c1ca-8aed-4cff-8f07-cb31575aed0a)
![11](https://github.com/user-attachments/assets/23fda0fc-6d30-4745-bb41-bc4ff3e1f7eb)
![12](https://github.com/user-attachments/assets/47adc51d-f0d7-4ff3-8728-e85d52c29303)
![13](https://github.com/user-attachments/assets/c287039e-db3e-4970-a27f-ef468f56196a)
![14](https://github.com/user-attachments/assets/2da1407b-ce28-49ab-bd35-0d25bf0e9a67)
![15](https://github.com/user-attachments/assets/cd7d7c52-ad41-4b41-8607-3bd00c92eeb0)
![16](https://github.com/user-attachments/assets/8bb7b112-d05d-41ed-a1d7-3d39a06786f3)
![17](https://github.com/user-attachments/assets/5656599f-602d-495c-a09e-9db25b4cdbad)
![18](https://github.com/user-attachments/assets/6f40c026-95de-4476-8330-fc439be728e0)
![19](https://github.com/user-attachments/assets/28719709-49b6-47d6-88c3-2f0f67c51977)
![20](https://github.com/user-attachments/assets/8eb748b0-4d2b-441c-b078-93f905ad7689)
![21](https://github.com/user-attachments/assets/395337ec-f9a6-4f38-a342-65ab229735b5)
![22](https://github.com/user-attachments/assets/619036e8-4db7-4d4c-b101-deef9bbe5336)
![23](https://github.com/user-attachments/assets/4d485b63-a0aa-45dc-8d4e-38e9593ac0c1)
![24](https://github.com/user-attachments/assets/59ed3688-2128-4f9f-a300-c68a1cddee4f)
![25](https://github.com/user-attachments/assets/f0c48c9b-e9eb-412b-b4c6-8fe51b28d1e6)
![26](https://github.com/user-attachments/assets/1e6d81e0-cfa0-431a-9d3a-2d6638183968)
![27](https://github.com/user-attachments/assets/2051cc5c-d363-4cf2-a10d-c50fd5cdeb8e)
![28](https://github.com/user-attachments/assets/c7783588-1569-4a00-aa7a-d0283e4a3303)
![29](https://github.com/user-attachments/assets/2124a5e9-e9ad-4358-a338-563fe314e8ab)
![30](https://github.com/user-attachments/assets/9b1ea76f-2766-46f5-9754-85b07ddcf67c)
![31](https://github.com/user-attachments/assets/fd0f9e1b-b05e-47b0-a9b7-ef32f443bea6)
![32](https://github.com/user-attachments/assets/307bc4d2-0473-4271-bbbb-23ef59d5e63e)
![33](https://github.com/user-attachments/assets/e56b39ff-8eaa-477a-ae5b-03af802a5bf5)
![34](https://github.com/user-attachments/assets/57ea73c7-bb82-41ca-a6df-a8bea253d7ec)
![35](https://github.com/user-attachments/assets/ad869bf2-7ef3-435b-a4c9-575cf923cd4a)
![36](https://github.com/user-attachments/assets/50f5c349-e06c-41c5-b54c-869c69a45991)
![37](https://github.com/user-attachments/assets/44dc0396-2b61-4738-8c5e-dc8c04941bda)
![38](https://github.com/user-attachments/assets/b775aa68-56fc-46bc-869f-e7d67a447276)
![40](https://github.com/user-attachments/assets/4870efda-d3f6-4d91-a935-215117cfb694)
