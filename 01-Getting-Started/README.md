# Cloud Foundation Toolkit Lab - Getting Started
Clone this repository and run locally, or use Cloud Shell to walk through the steps:

[![Open in Cloud Shell](https://gstatic.com/cloudssh/images/open-btn.svg)](https://ssh.cloud.google.com/cloudshell/editor?cloudshell_git_repo=https%3A%2F%2Fgithub.com%2Fterraform-google-modules%2Fcloud-foundation-training&cloudshell_git_branch=new-templates&cloudshell_open_in_editor=setup.tf&cloudshell_working_dir=terraform&cloudshell_tutorial=..%2Ftutorials%2F01-setup-backend.md)

## Prerequisites
* Google Cloud Platform project with valid billing account
* [Terraform](https://learn.hashicorp.com/terraform/getting-started/install.html)
* [Google Cloud SDK](https://cloud.google.com/sdk/install)

## Setup
This lab helps you initiate Terraform to work with Google Cloud Platform.

At the end of the exercise, you will have 2 GCS buckets, (**Logs** and **Remote State**) with 4 lines of output value.

### What You'll Learn
* Using Terraform
* [Provider](https://www.terraform.io/docs/configuration/providers.html) block
* [Resource](https://www.terraform.io/docs/configuration/resources.html) block
* [Output](https://www.terraform.io/docs/configuration/outputs.html) block
* [Local](https://www.terraform.io/docs/configuration/locals.html) values
* Initialize Terraform with local state
* Grant IAM Role (**Storage Admin**)
* Random ID generation
* Create and Destroy GCS Bucket
* Create another GCS Bucket for remote state for following exercises

## Task 1. Set Project
Determine your Google Cloud [Project ID](https://cloud.google.com/sdk/gcloud/reference/projects/list), and select a [Region](https://cloud.google.com/compute/docs/regions-zones/#available) to provision resources.

Replace the quoted text with your Project ID and set your Project
```bash
PROJECT_ID="my-project-id"
```
```bash
gcloud config set project $PROJECT_ID
```

## Task 2. Initialize Terraform

### Google Cloud Provider
Determine your Google Cloud [Project ID](https://cloud.google.com/sdk/gcloud/reference/projects/list), and select a [Region](https://cloud.google.com/compute/docs/regions-zones/#available) to provision resources.

Uncomment the `provider` block in <walkthrough-editor-open-file filePath="setup.tf">setup.tf</walkthrough-editor-open-file>, and fill it in to include your Project ID and a Region. Save file.

### Initialize Terraform
[terraform init](https://www.terraform.io/docs/commands/init.html) command is used to initialize a working directory containing Terraform configuration files.

```bash
terraform init
```

A successful `terraform init` will output the following:
```
Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.
```

###  Terraform Plan
[terraform plan](https://www.terraform.io/docs/commands/plan.html) command is used to create an execution plan. Terraform performs a refresh, unless explicitly disabled, and then determines what actions are necessary to achieve the desired state specified in the configuration files.

Run the following to see what Terraform attempts to create.
```bash
terraform plan
```

Because no resources have been created in your `setup.tf` file (all resources are commented out), your successful `terraform plan` output will look like the following:
```

No changes. Infrastructure is up-to-date.
This means that Terraform did not detect any differences between your
configuration and real physical resources that exist. As a result, no
actions need to be performed.
```

## Task 3. Create and assign variables
1. In <walkthrough-editor-open-file filePath="variables.tf">variables.tf</walkthrough-editor-open-file>, create input variables for `project_id` and `region` that you selected in the previous task. This doesn't mean your variables are set yet, this just declares them.
2. In <walkthrough-editor-open-file filePath="terraform.tfvars">terraform.tfvars</walkthrough-editor-open-file>, set your variables by assigning the values you used in Task 1.

We can now use these variables in our file and avoid repeating the Project ID every time.

Run `terraform plan` to make sure your variables were created and assigned correctly.
```bash
terraform plan
```

## Task 4. Create GCS Buckets

### Terraform Configuration
1. In <walkthrough-editor-open-file filePath="main
.tf">main.tf</walkthrough-editor-open-file>, uncomment the [random_id](https://www.terraform.io/docs/providers/random/r/id.html) resource block, which will generate a random character string that you can append to our bucket to help ensure uniqueness.

2. In <walkthrough-editor-open-file filePath="main
.tf">main.tf</walkthrough-editor-open-file>, uncomment and fill in the [google_storage_bucket](https://www.terraform.io/docs/providers/google/r/storage_bucket.html) that you'll create to store Terraform state.

*Note the use of the random_id output in our bucket name. We can re-use this string in other Terraform resources*.

### Terraform Init
Re-run `terraform init` to download the plugin that allows us to use the `random` resource.

```bash
terraform init
```

### Terraform Plan
Validate execution plan will create 1 GCS bucket with Random Suffix

```bash
terraform plan -out=plan.out
```

Review the Terraform plan that was generated.

### Terraform Apply
Execute previous generated execution plan

```bash
terraform apply plan.out
```

If your resources were created successfully, you should see the following message:
```
Apply complete! Resources: 1 added, 0 changed, 0 destroyed.

The state of your infrastructure has been saved to the path
below. This state is required to modify and destroy your
infrastructure, so keep it safe. To inspect the complete state
use the `terraform show` command.
State path: terraform.tfstate
```

### Verify
[gsutil](https://cloud.google.com/storage/docs/gsutil) is a Python application that lets you access Cloud Storage from the command line. It is installed as part of Google Cloud SDK.

Run the following to review your new Google Cloud Storage buckets Note the random string that was appended to your bucket name.

```bash
gsutil ls
```

You will see the newly created GCS bucket listed as `gs://BUCKET_NAME`

Look inside the folder `.terraform/plugins/<YOUR_OS>/` you will see there are now 2 providers, for Google and Random

Inspect `terraform.tfstate` to see resources managed the local Terraform state file

## Task 5. Setup Remote Backend
In the future, you'll want to store state in a GCS bucket instead of the local repository directory, so in this task you'll point Terraform state to the GCS bucket you've created.

In <walkthrough-editor-open-file filePath="backend
.tf">backend.tf</walkthrough-editor-open-file>, uncomment and fill in the [terraform backend](https://www.terraform.io/docs/backends/types/gcs.html) block so that state is stored in the GCS bucket instead of the local directory.

You'll repeat this step through a number of the labs.

### Terraform Init
Run `terraform init` to migrate state to your new remote backend.

```bash
terraform init
```

When prompted, enter `yes` to indicate you'd like to migrate state to the new remote backend.

When successful, the output should indicate the following:
```
Successfully configured the backend "gcs"! Terraform will automatically
use this backend unless the backend configuration changes.
```
