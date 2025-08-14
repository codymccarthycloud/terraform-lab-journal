# Validating Policies for Terraform on Google Cloud

In this lab, you’ll apply and modify a constraint using `gcloud beta terraform vet` to enforce domain restrictions in IAM policies. You will:

- Apply a constraint that enforces a domain restriction  
- Test a constraint to intentionally throw a validation error  
- Modify the constraint so that it passes validation  

---

## Activate Cloud Shell 

   - Click **Activate Cloud Shell** (top-right of the console).  
   - Click **Continue**, then **Authorize** when prompted.  
   - Verify your active project:
     ```bash
     gcloud config list project
     ```
   - (Optional) List the active account:
     ```bash
     gcloud auth list
     ```
   - (Optional) If you need to switch accounts:
     ```bash
     gcloud config set account ACCOUNT_EMAIL
     ```

---

## Validate a Constraint

### 1. Clone the Policy Library
```bash
git clone https://github.com/GoogleCloudPlatform/policy-library.git
cd policy-library/
```

### 2. Copy the Sample Constraint
```bash
cp samples/iam_service_accounts_only.yaml policies/constraints/
```

### 3. Inspect the Constraint
```bash
cat policies/constraints/iam_service_accounts_only.yaml
```

You should see:
```Yaml 
apiVersion: constraints.gatekeeper.sh/v1alpha1
kind: GCPIAMAllowedPolicyMemberDomainsConstraintV2
metadata:
  name: service_accounts_only
  annotations:
    description: Checks that members that have been granted IAM roles belong to allowlisted domains.
spec:
  severity: high
  match:
    target:
      - "organizations/**"
  parameters:
    domains:
      - gserviceaccount.com
```

### 4. Create and Populate a Terraform File
```bash
touch main.tf
```

Paste the following into main.tf (replace placeholders):
```Hcl
terraform {
  required_providers {
    google = {
      source  = "hashicorp/google"
      version = "~> 3.84"
    }
  }
}

resource "google_project_iam_binding" "sample_iam_binding" {
  project = "<YOUR_PROJECT_ID>"
  role    = "roles/viewer"
  members = [
    "user:<USER_EMAIL>"
  ]
}
```
### 5. Initialize Terraform
```bash
terraform init
```

### 6. Export and Convert Terraform Plan
```bash
terraform plan -out=test.tfplan
terraform show -json test.tfplan > tfplan.json
```

### 7. Install Terraform Vet Tools
```bash
sudo apt-get update && sudo apt-get install -y google-cloud-sdk-terraform-tools
```

### 8. Validate Against the Constraint 
```bash
gcloud beta terraform vet tfplan.json --policy-library=.
```

You should see a violation because the <USER_EMAIL> domain is not gserviceaccount.com

## Modify the Constraint 

### 1. Edit the Constraints File
Open policies/constraints/iam_service_accounts_only.yaml and update the domains list:
```Yaml
parameters:
  domains:
    - gserviceaccount.com
    - qwiklabs.net
```

### 2. Re-export and Re-validate
```bash 
terraform plan -out=test.tfplan
terraform show -json test.tfplan > tfplan.json
gcloud beta terraform vet tfplan.json --policy-library=.
```
You should see no violations now that qwiklabs.net is allowlisted.

### 3. Apply the Terraform Plan
```bash
terraform apply test.tfplan
```

---

## Lab Output Screenshots

![1](https://github.com/user-attachments/assets/415eceee-2a7b-4f5a-ad1a-2ab1462cf669)
![2](https://github.com/user-attachments/assets/b7ae933a-8b5e-46b9-a796-119052ad7ed3)
![4](https://github.com/user-attachments/assets/d42a8bbe-4a32-4b55-8712-c7505216da27)
![6](https://github.com/user-attachments/assets/f5800798-22ba-4264-a3ba-1a8aac41bc34)
![7](https://github.com/user-attachments/assets/2cbbd519-6563-4d4a-b6c9-3994c743d094)
![8](https://github.com/user-attachments/assets/9b32f746-fa15-4ca3-8112-750cb8aa75dc)
![9](https://github.com/user-attachments/assets/acf917d6-28e3-49cd-9213-da341de75620)
![10](https://github.com/user-attachments/assets/ad28904c-bd59-466d-a056-9c0894d408a9)
![5](https://github.com/user-attachments/assets/3f1946de-f3d8-48bc-9574-4731c851348f)

