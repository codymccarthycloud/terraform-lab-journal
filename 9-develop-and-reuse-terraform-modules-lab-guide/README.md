## Develop and Reuse Terraform Modules 

Getting Started
Getting to VS Code
At the AWS console home screen click on EC2 to go to the EC2 dashboard.

  Click on Instances (running) to show the EC2 instance which has been set up for this lab.

  Click on the i- Instance ID link for the EC2.

  Copy the Public IPv4 address.

  Precede the address with https://, and append a port :8080.

  In a new browser tab, navigate to that address: https://<Public IPv4 Address>:8080

- You will receive a warning about the site being insecure. This is due to the server using a self-signed certificate. You can ignore this, click through the warning, and proceed to the site. During testing, this at times required two or three attempts.
- Once the page loads you'll be prompted for a basic password to authenticate. The password is Linux4All!
- The site will load and you will have a full version of VS Code running in your browser.
- Set up the Solution Directory
- Inside VS Code, click on the Search bar at the top.
- Click on Show and Run Commands.
- Type in New Terminal
- Click on Terminal: Create New Terminal.
- This will launch a shell TERMINAL at the bottom of VS Code, which is running commands directly on the EC2 instance.
- In the TERMINAL type mkdir src && cd src && mkdir terraform && mkdir lambda (and press enter) to scaffold a directory structure for your solution files.
- If you're pasting in commands, and get a clipboard warning, you can click Allow.
- On the left hand navigation of VS Code click on the Explorer icon to open word up the file explorer.
- Click on Open Folder and select the src folder you just created.
- Click OK and wait for a Do you trust pop-up to appear.
- This may take one or two minutes. (You can ignore an Error loading webview pop-up at the bottom, if it appears; this may've appeared prior, and ignoring it then too is fine.)
- If after a minute or two it doesn't appear, Reload the browser tab. It will then appear.
- Check the Trust the authors of all files in the parent folder 'cloud_user', and click Yes, I trust the authors.
- Using Public Modules
- Provision an S3 Bucket
- Hover over the EXPLORER on the left, click on the terraform folder, and then click the icon for New File...

Name the file main.tf (then press enter).

Set up the AWS and Random Provider by creating the terraform configuration block and aws provider configuration; paste in the following:

```Hcl
 terraform {
      required_providers {
        aws = {
          source = "hashicorp/aws"
          version = "6.0.0"
        }
        random = {
          source = "hashicorp/random"
          version = "3.7.2"
        }
   }
}

provider aws {
   region = "us-east-1"
}
Add (at the very end of the file) a random_id resource block to ensure the S3 Bucket name is unique:

resource "random_id" "s3_suffix" {
  byte_length = 4
}
```

Add an S3 Bucket utilizing the public modules provided by AWS:

```Hcl
data "aws_caller_identity" "current" {}
data "aws_iam_policy_document" "s3_bucket_access" {
  statement {
    sid       = "AllowEc2RoleReadWrite"
    effect    = "Allow"

    principals {
      type        = "AWS"
      identifiers = [data.aws_caller_identity.current.arn]
    }

    actions = [
      "s3:*"
    ]

    resources = [
      module.s3_bucket.s3_bucket_arn,
      "${module.s3_bucket.s3_bucket_arn}/*",
    ]
  }
}

module "s3_bucket" {
    source = "terraform-aws-modules/s3-bucket/aws"

    bucket            = "ps-tf-bucket-${random_id.s3_suffix.hex}"
    versioning = {
       status = "Enabled"
       mfa_delete = "Disabled"
    }

    force_destroy = true
    attach_policy = true
    policy = data.aws_iam_policy_document.s3_bucket_access.json
}
```

Note: When defining the bucket name, you used the random_id resource defined earlier to ensure the bucket name is unique. The hex property of the random_id resource gives a hexadecimal value of the id and will always return twice as many characters as the specified byte length. In this example it will be an 8 character suffix. Additionally the aws_iam_policy_document and aws_caller_identity configuration is giving the current executing caller, the EC2 instance in this case, the ability to read and write to the S3 bucket. The aws_iam_policy_document is just a convenient way to format the policy JSON rather than having to write it by hand.

- Open a New Terminal in VS Code by clicking on the Search bar at the top. Click on Show and Run Commands, and then click on Terminal: Create New Terminal.
- In the TERMINAL run the cd terraform command, then run the terraform init command to initialize Terraform and download the providers you configured.
- You will see output describing, finding, and installing each the hashicorp/random and hashicorp/aws provider. When terraform is finished initializing you will see a message Terraform has been successfully initialized
- In the TERMINAL run the terraform plan command to preview what actions terraform will take.
- Terraform will output all of the properties of the two resources it's going to create. At the very end of the terraform plan output you should see Plan: 5 to add, 0 to change, 0 to destroy.. A warning for Deprecated Attribute may appear, if it does, it is safe to ignore.
- Run the terraform apply command to provision the resources specified in main.tf
- Terraform will run another plan before prompting you with Do you want to perform these actions?. Enter yes and terraform will begin provisioning the resources.
- Once the resources are provisioned you will see a message saying: Apply Complete! Resources 5 added, 0 changed, 0 destroyed..
- Go to the other browser tab which is at the AWS Console's EC2 page
- In the search bar type S3
- Select the S3 Scalable storage in the Cloud item.
- You should now see the bucket which terraform provisioned for you: The name will be something like ps-tf-bucket-<random id> and notice how the random suffix was added based on the random_id resource you created.
- Switch back to the browser tab containing VS Code
- Provision a Lambda Function
- Hover over the EXPLORER on the left, click on the lambda folder, and then click the icon for New File...

Name the file handler.py

Add the following handler code:

```python
def lambda_handler(event, context):
    print(f"Received S3 Bucket Notification")
    print(event)
```

Go back to main.tf, and add the following terraform configuration to provision a Lambda Function:

```Hcl
data "aws_iam_policy_document" "assume_role" {
  statement {
    effect = "Allow"

    principals {
      type        = "Service"
      identifiers = ["lambda.amazonaws.com"]
    }

    actions = ["sts:AssumeRole"]
  }
}

resource "aws_iam_role" "lambda_role" {
  name               = "lambda_execution_role"
  assume_role_policy = data.aws_iam_policy_document.assume_role.json
}

resource "aws_iam_role_policy_attachment" "attach-basic-execution" {
    role = aws_iam_role.lambda_role.name
    policy_arn = "arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole"
}

data "archive_file" "function" {
  type        = "zip"
  source_file = "../lambda/handler.py"
  output_path = "../lambda/function.zip"
}

resource "aws_lambda_function" "s3_handler" {
  filename         = data.archive_file.function.output_path
  function_name    = "s3_handler_lambda_function"
  role             = aws_iam_role.lambda_role.arn
  handler          = "handler.lambda_handler"
  source_code_hash = data.archive_file.function.output_base64sha256

  runtime = "python3.12"
}

resource "aws_cloudwatch_log_group" "lambda_group" {
  name = "/aws/lambda/${aws_lambda_function.s3_handler.function_name}"
}
```

```bash
terraform init -upgrade
terraform apply
```


Note: The configuration here is adding an AWS lambda function which is the last resource defined. The other resources above it are pre-requisites to utilizing a lambda properly on AWS. Lambdas require a deployment .zip file which is what the archive_file resource is creating. The Other two resources aws_iam_policy_document and aws_iam_role are granting an identity to the lambda function. Lastly you also provision a Cloudwatch Log Group for this Lambda function to write its logs to.

- In the TERMINAL run the terraform init -upgrade command as the archive_file resource used requires this to be re-initialized
- Run the terraform apply command. The apply will show you the plan first and at the end of the plan output you should see Plan: 4 to add, 0 to change, 0 to destroy..
- When prompted with Do you want to perform these actions?. Enter yes and terraform will begin provisioning the resources.
- (A warning for Deprecated Attribute may appear, and if it does, it is safe to ignore.)
- Once the resources are provisioned you will see a message saying: Apply Complete! Resources 4 added, 0 changed, 0 destroyed..
- Go to the other browser tab which is at the AWS Console's S3 page
- In the search bar type Lambda
- Select the Lambda Run code without thinking about servers item.
- You should now see the lambda function named s3_handler_lambda_function which terraform provisioned for you
- Click on the s3_handler_lambda_function item and verify the handler.py file shows in the lambda file explorer, and the code matches what you created earlier.
- On the left side of the lambda window click on the Test button.
- Click the search item that appears, Create new test event.
- Click Invoke to send a "hello world" test event to your lambda.
- The output window will display some system logs, between the START RequestId: and END RequestId: messages will be the logs generated by your lambda function code. It should read:

Received S3 Bucket Notification
{'key1': 'value1', 'key2': 'value2', 'key3': 'value3'}
Provision an S3 Notification
Switch back to the browser tab containing VS Code

In the main.tf file add the following terraform configuration to allow the S3 bucket to invoke the lambda function

```Hcl
resource "aws_lambda_permission" "allow_bucket" {
  statement_id  = "AllowExecutionFromS3Bucket"
  action        = "lambda:InvokeFunction"
  function_name = aws_lambda_function.s3_handler.arn
  principal     = "s3.amazonaws.com"
  source_arn    = module.s3_bucket.s3_bucket_arn
}
Add a resource for the S3 Bucket notification:

resource "aws_s3_bucket_notification" "bucket_notification" {
  bucket = module.s3_bucket.s3_bucket_id

  lambda_function {
    lambda_function_arn = aws_lambda_function.s3_handler.arn
    events              = ["s3:ObjectCreated:*"]
  }

  depends_on = [aws_lambda_permission.allow_bucket]
}
```

```bash
terraform apply --auto-approve
```


- In the TERMINAL run the terraform apply --auto-approve command to provision the resources and skip the prompt.
- Terraform will run its plan outputting all of the properties of what it's going to create, then immediately begin provisioning the infrastructure due to the --auto-approve flag. Once the resources are provisioned you will see a message saying: Apply Complete! Resources 2 added, 0 changed, 0 destroyed..
- Go to the other browser tab which is at the AWS Console's Lambda page for your s3_handler_lambda_function.
- Reload the browser tab, and you will see the S3 icon, which is now a trigger for the lambda function.
- Click on the S3 bucket icon, which will open the Configuration tab and show the S3 bucket details.
- Click on the ps-tf-bucket-<random id> link to take you to the S3 bucket in a new browser tab.
- Copy the full bucket name, which begins with ps-tf-bucket-.
- Switch back to the browser tab containing VS Code and in the TERMINAL enter the following command to save your bucket name to an environment variable export BUCKET_NAME=ps-tf-bucket-<random id>
- Make sure the command has the actual full bucket name, like ps-tf-bucket-d475529f, and does not literally contain <random id>.
- Run the following AWS CLI command to copy a file to S3 and trigger the lambda aws s3 cp ../lambda/handler.py s3://$BUCKET_NAME
- Switch back to the browser tab containing the AWS Console's ps-tf-bucket- Page.
- Click the Refresh icon, and observe the handler.py file is now uploaded.
- Navigate to the Lambda page with your lambda function.
- Select the Monitor tab and observe there is now a dot or line in the Invocations panel, indicating the lambda was invoked.
- Click on View CloudWatch logs to take you to the logs of the function in a new browser tab.
- Click the the most recent Log stream (there will likely only be one). The name will look something like YYYY/MM/DD/[$LATEST]<random id>.
- You'll see the logs printed from the Lambda function showing Received S3 Bucket notification, as well as the S3 notification object itself.
- Develop a Custom Terraform Module
- Create an S3 Notification Module
- Switch back to the browser tab containing VS Code
- Hover over the EXPLORER on the left, click on the terraform folder, and then click the icon for New Folder.... Name the folder modules.
- Create another new folder within the modules folder called s3_notification
- Within the s3_notification folder, create two files: main.tf and variables.tf
- Start by copying the aws_lambda_permission resource and aws_s3_bucket_notification resource from your existing main.tf file, into the new modules/s3_notification/main.tf file. This is the terraform code for reference:

```Hcl
resource "aws_lambda_permission" "allow_bucket" {
    statement_id  = "AllowExecutionFromS3Bucket"
    action        = "lambda:InvokeFunction"
    function_name = aws_lambda_function.s3_handler.arn
    principal     = "s3.amazonaws.com"
    source_arn    = module.s3_bucket.s3_bucket_arn
}

resource "aws_s3_bucket_notification" "bucket_notification" {
    bucket = module.s3_bucket.s3_bucket_id

    lambda_function {
        lambda_function_arn = aws_lambda_function.s3_handler.arn
        events              = ["s3:ObjectCreated:*"]
    }

    depends_on = [aws_lambda_permission.allow_bucket]
}
Next, move to the variables.tf file to begin defining the variables of the module to make it more dynamic. Define an input variable for the S3 bucket properties:

variable "bucket" {
    description = "S3 Bucket Identifiers"
    type = object({
        arn = string
        id = string
    })
}
Define an input variable for the Lambda configuration of the S3 Notification (add it at the end of the file):

variable "lambda_notification" {
    description = "Lambda Function notification configuration object"
    type = object({
        arn = string
        events = list(string)
        filter_prefix = optional(string)
        filter_suffix = optional(string)
    })
}
```


In /s3_notification/main.tf modify the terraform configuration to use the variables, rather than hard coding the values:

Within the aws_lambda_permission resource:

Change function_name = aws_lambda_function.s3_handler.arn to function_name = var.lambda_notification.arn
Change source_arn = module.s3_bucket.s3_bucket_arn to source_arn = var.bucket.arn
Within the aws_s3_bucket_notification resource

Change bucket = module.s3_bucket.s3_bucket_id to bucket = var.bucket.id
Change the properties of the lambda_function block to the following:
lambda_function {
    lambda_function_arn = var.lambda_notification.arn
    events              = var.lambda_notification.events
    filter_prefix       = try(var.lambda_notification.filter_prefix, null)
    filter_suffix       = try(var.lambda_notification.filter_suffix, null)
}


The resulting terraform configuration should look like this:

```Hcl
resource "aws_lambda_permission" "allow_bucket" {
  statement_id  = "AllowExecutionFromS3Bucket"
  action        = "lambda:InvokeFunction"
  function_name = var.lambda_notification.arn
  principal     = "s3.amazonaws.com"
  source_arn    = var.bucket.arn
}

resource "aws_s3_bucket_notification" "bucket_notification" {
  bucket = var.bucket.id

  lambda_function {
    lambda_function_arn = var.lambda_notification.arn
    events              = var.lambda_notification.events
    filter_prefix       = try(var.lambda_notification.filter_prefix, null)
    filter_suffix       = try(var.lambda_notification.filter_suffix, null)
  }    

  depends_on = [aws_lambda_permission.allow_bucket]
}
```

In the TERMINAL run 

```bash
cd modules/s3_notification && zip -r s3_notification.zip . && cd ../../ 
aws s3 cp modules/s3_notification/s3_notification.zip s3://$BUCKET_NAME/modules/
echo $BUCKET_NAME
```

- Command to zip the contents of the module folder into an archive.
- Command to copy your module to the S3 bucket.
- Command to print the name of your bucket.

- You'll use the full name in the next step.
- In the main.tf file of your primary terraform configuration (not the module), modify it to utilize the remote notification module.
- Replace the aws_lambda_permission and aws_s3_bucket_notification resources with the following module configuration:

```Hcl
moved {
    from = aws_lambda_permission.allow_bucket
    to = module.s3_notification.aws_lambda_permission.allow_bucket
}   

moved {
    from = aws_s3_bucket_notification.bucket_notification
    to = module.s3_notification.aws_s3_bucket_notification.bucket_notification
}

module "s3_notification" {
    source = "s3::https://s3-us-east-1/ps-tf-bucket-<random id>/modules/s3_notification.zip"
    bucket = {
        arn = module.s3_bucket.s3_bucket_arn
        id = module.s3_bucket.s3_bucket_id
    }
    lambda_notification = {
        arn = aws_lambda_function.s3_handler.arn
        events = ["s3:ObjectCreated:*"]
    }
}
```

```bash 
terraform init
terraform plan
terraform apply --auto-approve
```

Replace ps-tf-bucket-<random id> with the full bucket name you echoed earlier.

Note: It is important that the source property of the module use the exact bucket name. Unfortunately Terraform does not allow the use of variables in a module source which is why it must be hard coded to the exact name here.

- Run the terraform init command (in the TERMINAL) to initialize the new module.
- You will see a Terraform has been successfully initialized! message once the module has been successfully initialized.
- Run the terraform plan command to see Terraform's plan for the new module, entering yes when prompted.
- The plan will output all of the properties and resources and end with stating that you moved the two resources into the module and end with Plan: 0 to add, 0 to change, 0 to destroy..

Note: This is because you removed the two resources directly from the configuration and moved them to the module. Terraform would normally want to destroy and re-create these resources because their identifiers changed, however since you specified where they moved to, terraform can reconcile the state file without needing to alter the infrastructure.

- Run the terraform apply --auto-approve command to execute the move.
- Terraform will complete the move of the resources and output Apply complete! Resources: 0 added, 0 changed, 0 destroyed.
- Use Module Versioning
- You will modify the module to instead use a list, which will be a new module version.
- In the variables.tf file, change the lambda_notification variable to

```Hcl
variable "lambda_notification" {
    description = "Lambda Function notification configuration object"
    type = list(object({
        arn = string
        events = list(string)
        filter_prefix = optional(string)
        filter_suffix = optional(string)
    }))
}
Add a new variable, sqs_notification:

variable "sqs_notification" {
    description = "SQS notification configuration object"
    type = list(object({
        arn = string
        events = list(string)
        filter_prefix = optional(string)
        filter_suffix = optional(string)
    }))
}
```

In modules/s3_notification/main.tf, alter aws_lambda_permission:

```Hcl
resource "aws_lambda_permission" "allow_bucket" {
  for_each = var.lambda_notification

  statement_id  = "AllowExecutionFromS3Bucket"
  action        = "lambda:InvokeFunction"
  function_name = each.value.arn
  principal     = "s3.amazonaws.com"
  source_arn    = var.bucket.arn
}
Change the aws_bucket_notification to

resource "aws_s3_bucket_notification" "bucket_notification" {
    bucket = var.bucket.id

  dynamic "lambda_function" {
    for_each = var.lambda_notification

    content {
        lambda_function_arn = lambda_function.value.arn
        events              = lambda_function.value.events
        filter_prefix       = try(lambda_function.value.filter_prefix, null)
        filter_suffix       = try(lambda_function.value.filter_suffix, null)
    }
  }

  dynamic "queue" {
    for_each = var.sqs_notification

    content {
        lambda_function_arn = queue.value.arn
        events              = queue.value.events
        filter_prefix       = try(queue.value.filter_prefix, null)
        filter_suffix       = try(queue.value.filter_suffix, null)
    }
  }
  depends_on = [aws_lambda_permission.allow_bucket]
}
```

```bash
terraform init
```

- In the TERMINAL run the cd modules/s3_notification && rm s3_notification.zip followed by the zip -r s3_notification.zip . && cd ../../ command. To create a new version of the module package
- Run the aws s3 cp modules/s3_notification/s3_notification.zip s3://$BUCKET_NAME/modules/ command to upload the package to S3.
- Go back to the AWS Console S3 ps-tf-bucket- browser tab, click the Refresh button, then click modules/.
- Click s3_notification.zip, and then select the Versions tab.
- You should see 2 versions of your module, the first one you created earlier, and the latest version which supports a list and SQS Notifications.
- Click the older version's link (it's the only one with a link).
- Copy from the Object URL its suffix starting with ?versionId=<id>.
- Switch back to the browser tab containing VS Code.
- In the main.tf file of your infrastructure (not the module), modify the s3_notification's source property:
- Append the ?versionId=<id> string you copied to the end of the existing url on the source property.
- When completed, it will look something like
- source = "s3::https//s3-us-east-1/ps-tf-bucket-2d58532e/modules/s3_notification.zip?versionId=ynMgmb2O1IK9cT3gigukq0TStwHqEpYN"
- The bucket name's suffix and the versionId's value will be different, of course.
- In the TERMINAL run the terraform init command.
- Part of the output will show terraform Downloading the specific version you requested.
- Note: Specifying the version explicitly will protect you from accidentally incorporating changes you are not ready for.

## Lab Output Screenshots

![1](https://github.com/user-attachments/assets/f42282e5-2279-44f7-ad5f-d8353bf24e6d)
![2](https://github.com/user-attachments/assets/057c897f-2ddc-424a-90fd-725ec2f69fc3)
![3](https://github.com/user-attachments/assets/fe42f68f-9ef9-4cf2-a3e9-b8763f678cdd)
![4](https://github.com/user-attachments/assets/92e3ba86-4947-4aaf-8bc2-c4a6b1374e20)
![5](https://github.com/user-attachments/assets/d6bc6d25-04a5-40bf-ad3e-f33491a3751b)
![6](https://github.com/user-attachments/assets/0ef6dcd2-cb68-4456-a8d5-1efdc151ba8a)
![7](https://github.com/user-attachments/assets/711e39c2-1156-442f-8d1d-df78a4567981)
![8](https://github.com/user-attachments/assets/4037678e-2acf-40a1-a87d-86e646cdd0ac)
![9](https://github.com/user-attachments/assets/c4cf358c-4cb7-484e-8165-0f6bbeb03f7d)
![10](https://github.com/user-attachments/assets/427e56d3-0625-4836-aadb-424fde81cf37)
![11](https://github.com/user-attachments/assets/98a3edef-091f-4dc6-a49d-f6cc9f5ae8cf)
![12](https://github.com/user-attachments/assets/4e643b33-ce81-48b2-84fb-94b7edfad879)
![13](https://github.com/user-attachments/assets/08f45d0c-e94b-4764-b1fe-df366565992c)
![14](https://github.com/user-attachments/assets/2d02211d-63da-477a-8247-10e93c24846b)
![15](https://github.com/user-attachments/assets/b1fe1c2c-4692-44d3-a962-7dd2ab5831e9)
![16](https://github.com/user-attachments/assets/10fcae18-eb0e-47dd-b583-50acc886bf80)
![17](https://github.com/user-attachments/assets/586a0fe9-e26c-40c1-b7c9-13ac6ec06d1c)
![18](https://github.com/user-attachments/assets/df41aecd-e3ff-4808-81c0-e4d47a70d422)
![19](https://github.com/user-attachments/assets/82f75972-ca34-4420-b22d-ea88154a7343)
![20](https://github.com/user-attachments/assets/ac6862e5-7e05-4976-bd7a-5c0227c67e7a)
![21](https://github.com/user-attachments/assets/f2e6f904-7cf4-4b78-af99-51f39a4080e3)
![22](https://github.com/user-attachments/assets/d6190ae6-f3ea-4b5c-b3c4-1dcc88ba8196)
![23](https://github.com/user-attachments/assets/a497fe38-a124-4dcf-bb90-8bfd18d237a0)
![24](https://github.com/user-attachments/assets/295ba2e3-da2f-4ecd-9829-75cc2e788526)
![25](https://github.com/user-attachments/assets/08aa6ec7-5230-47d6-a43a-7fd2d9e75208)
![26](https://github.com/user-attachments/assets/6279a137-2e8b-4a6d-a059-0a905c0bc183)
![27](https://github.com/user-attachments/assets/d9c3f09b-7ba0-42a0-ae16-fad74acb0631)
![28](https://github.com/user-attachments/assets/a5d9a84a-1c77-48ad-aa54-b6016f32c3a6)
![29](https://github.com/user-attachments/assets/d97cf1a7-7006-4111-895a-1ec540ab46ce)
![30](https://github.com/user-attachments/assets/8fabde0b-e27f-4097-aa53-d03dc42ed5e1)
![31](https://github.com/user-attachments/assets/af326009-4cca-44c7-99cb-01586e681799)
![32](https://github.com/user-attachments/assets/29fd44ea-49f8-487a-bdb7-708fe6857f13)
![33](https://github.com/user-attachments/assets/96d11d33-bc77-400d-bf27-50c87b970e22)
![34](https://github.com/user-attachments/assets/8d6f2113-3e4b-4a52-8bc9-0ed97fd0a9dc)
![35](https://github.com/user-attachments/assets/ccf6673d-372a-49cd-9c03-eecfc9eb8a40)
![36](https://github.com/user-attachments/assets/0952ff6d-4a99-40d3-aba2-6344c9c161e0)
![37](https://github.com/user-attachments/assets/7f15a594-aa67-4bb3-beda-02d0d12832e6)
![38](https://github.com/user-attachments/assets/55189b4c-3381-4904-90bd-ba8d7b11a5c9)
![39](https://github.com/user-attachments/assets/766ea101-c602-42ef-b600-0bbca7becebd)
![40](https://github.com/user-attachments/assets/4d641dbd-aebe-457c-b169-f3a44aa7c616)
![41](https://github.com/user-attachments/assets/6654ea92-8632-498c-8ef5-ced9167caca7)
![42](https://github.com/user-attachments/assets/ad1c2b06-88d1-4b8b-9840-78bce231d366)
![43](https://github.com/user-attachments/assets/6e4e9764-2d20-47b3-9695-4180ac5eded3)
![44](https://github.com/user-attachments/assets/3e5a5030-3cc0-42e8-84ef-13ef1f35a96e)
![45](https://github.com/user-attachments/assets/eaa78460-ab91-4929-9f71-9ee1ec38547b)
![46](https://github.com/user-attachments/assets/9b3fea32-5626-4bc5-a31f-e3a07e393001)
![47](https://github.com/user-attachments/assets/6ae6b1a2-1f4a-464c-a97f-47290e4cf359)
![48](https://github.com/user-attachments/assets/334962fa-2e0a-4ce0-b672-fbca27b2d98f)
![49](https://github.com/user-attachments/assets/7c70c157-18b1-447d-aad5-9b177b6b40f5)
![50](https://github.com/user-attachments/assets/9a20a967-523a-4412-a567-216d0a48aa10)
![51](https://github.com/user-attachments/assets/055bac8e-c550-4331-a185-7d3dbbd73ae4)
![52](https://github.com/user-attachments/assets/bd4dbdee-3f5d-4fb9-9d26-a79d59d11882)
![53](https://github.com/user-attachments/assets/56f54c46-51ac-41f3-873f-f1c9115c3fd7)

