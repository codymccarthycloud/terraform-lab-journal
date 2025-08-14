## Verifying Terraform Installation


Run the following command to confirm Terraform is available:

```bash
terraform
```

## Build Infrastructure
2.1 Create Configuration File
In Cloud Shell, create a new file named instance.tf:

```bash
touch instance.tf
```

Open instance.tf in the editor and add:

```hcl
resource "google_compute_instance" "terraform" {
  project      = "qwiklabs-gcp-00-beb3b9b58f34"
  name         = "terraform"
  machine_type = "e2-medium"
  zone         = "us-west1-b"

  boot_disk {
    initialize_params {
      image = "debian-cloud/debian-11"
    }
  }

  network_interface {
    network       = "default"
    access_config {}
  }
}
```

2.2 Initialize Terraform
```bash
terraform init
```

2.3 Preview Execution Plan
```bash
terraform plan
```

2.4 Apply Configuration
```bash
terraform apply
```

## Inspect Terraform State
Run the following to view the state file contents:

```bash
terraform show
```

## Conclusion

Verified Terraform installation in Cloud Shell
Written a Terraform configuration to provision a VM
Initialized, planned, and applied infrastructure changes
Inspected the Terraform state file

## Lab Output Screenshots

![1](https://github.com/user-attachments/assets/b30e3718-217c-4044-b474-35433992b71d)
![2](https://github.com/user-attachments/assets/c872b98e-d9e5-44bf-abd6-aba8a904c18a)
![3](https://github.com/user-attachments/assets/67445dd7-6621-4f3c-82bd-a65e0d8e6cd9)
![4](https://github.com/user-attachments/assets/72f5ca9e-66c2-4eb8-be38-b1127e0f10ad)
![5](https://github.com/user-attachments/assets/d22bd133-a743-49b2-9c5e-082c76e60070)
![6](https://github.com/user-attachments/assets/6c324f29-1cac-4d6e-9a76-58312a148706)
![7](https://github.com/user-attachments/assets/7cdc0911-7c46-41ca-99e0-8aa4ac2cc98e)
![8](https://github.com/user-attachments/assets/0ed58b8a-405a-4f5e-ace1-bf5062832142)
![9](https://github.com/user-attachments/assets/aab7057a-112b-4192-a0d0-01838b3a3543)




