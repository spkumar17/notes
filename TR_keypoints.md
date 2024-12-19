# TERRAFORM 

## IAC
* IaC is written using a **declarative** approach (**not imperative**)
* IaC allows you to commit your configurations to version control to safely collaborate on infrastructure(**versioned, shared, and reused**)
* IaC code can be used to manage infrastructure on multiple cloud platforms (**platform agnostic**)

* IaC uses a human-readable configuration language to help you write infrastructure code quickly
* Terraform is an **immutable, declarative, Infrastructure as Code** provisioning language based on Hashicorp Configuration Language **HCL**, or optionally **JSON**.
* Terraform is primarily designed for **macOS, FreeBSD, OpenBSD, Linux, Solaris, and Windows**
* In Terraform, the variable type `float` is not a valid type. Terraform supports variable types such as `string`, `map`, `bool`, and `number`, but not `float`.

## Points on state file
* Terraform relies on state files to track the current state of infrastructure resources and compare it to the desired state declared in configuration files. Without a state file, Terraform would not be able to accurately determine the changes needed to align real-world resources with the desired state.

## PLUGINS
* Terraform **< 0.13** before 0.13 we need to atomatically install the pugins in provider block but after **> 0.13** version when we run terraform init terraform will atomatically install all the required providers 

* Provider requirements are declared in a **required_providers** block.


```
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}
## here hashicorp is the namespace (who manages the provider)
## Provider Name: aws (the specific provider for AWS resources)

terraform {
  required_providers {
    mycloud = {  # mycloud is an alias or local name more details below 
      source  = "mycorp/mycloud"
      version = "~> 1.0"
    }
  }
}


```
####  Giving local name for plugin 

* the local name in Terraform is a user-defined alias for a provider. It allows you to reference a provider with any name in your configuration while keeping its source intact.

```
terraform {
  required_providers {
    mycloud = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "mycloud" {
  region = "us-east-1"
}

resource "mycloud_instance" "example" {
  ami           = "ami-12345678"
  instance_type = "t2.micro"
}
```
* Exact Version: **version = "1.0"** No other versions will be allowed.
* wild card version: **version = "~> 1.0"** Allows any version ≥ 1.0.0 but < 2.0.0.
```
terraform {
  required_version = ">= 1.9.2"
}
```
specifies the minimum version of Terraform required to run the configuration, ensuring compatibility and preventing potential issues that may arise from using older versions.
* In Terraform, the **alias parameter** is used to create an alternative name for a provider instance. This is especially useful when you need to configure multiple instances of the same provider (for example, multiple AWS regions).
* The provider plugins are downloaded and stored in the **`.terraform/providers`**  directory within the current working directory. This directory is specifically used by Terraform to manage provider plugins.


### Environment Variables

* **export TF_LOG=trace** ---> to set the log 
* **export TF_LOG=off** ---> to diable them
* Terraform Logging Levels (TF_LOG)

| Log Level  | Description                                               |
|------------|-----------------------------------------------------------|
| `TRACE`    | Most detailed logging, logs every internal step.          |
| `DEBUG`    | Logs detailed information, useful for in-depth debugging. |
| `INFO`     | Default level, provides important operational details.    |
| `WARN`     | Logs warnings and errors only.                            |
| `ERROR`    | Logs errors only.                                         |
| `NONE`     | No logging.                                               |

* **TF_LOG_PATH** This specifies where the log should persist its output to. Note that even when TF_LOG_PATH is set, TF_LOG must be set in order for any logging to be enabled.

**export TF_LOG_PATH=./terraform.log**
* TF_INPUT is an environment variable in Terraform that controls whether or not Terraform will prompt for user input during execution (if set to false, it disables prompts).
* **export TF_WORKSPACE=your_workspace** For multi-environment deployment, in order to select a workspace, instead of doing terraform workspace select your_workspace, it is possible to use this environment variable. Using TF_WORKSPACE allow and override workspace selection.


### Scenario based:

1) When multiple arguments with single-line values appear on consecutive lines at the same nesting level, HashiCorp recommends that you:

```
ami = var.aws_ami
instance_type = var.instance_size
subnet_id = "subnet-0bb1c79de3EXAMPLE"
tags = {
  Name = "HelloWorld"
}
```

align the equals signs
```
ami           = "abc123" 
instance_type = "t2.micro"
```
HashiCorp recommends aligning the equals signs for better readability and consistency in the code. This helps in quickly identifying and understanding the key-value pairs in the configuration.

2) When writing Terraform code, how many spaces between each nesting level does HashiCorp recommend that you use?

2 
HashiCorp recommends using 2 spaces between each nesting level in Terraform code for better readability and maintainability.

### More on commands:

* **terraform apply -replace:** When you use the -replace option, Terraform will plan to destroy and recreate the resource (in this case, aws_instance.web). **terraform taint:** Before Terraform 0.13, this command would manually mark a resource for replacement, and the next time terraform apply was run, the tainted resource would be replaced.
So, terraform taint is now effectively replaced by the -replace flag in Terraform 0.13 and later.

1) **terraform state show** Displays the current state of a specific resource in the Terraform state file.

2) **terraform state mv aws_instance.web aws_instance.new_web** The terraform state mv command moves a resource in the Terraform state, updating the internal tracking of that resource, but it does not make any changes to the actual infrastructure.

3) **terraform state list** this will list all the resource in that state file
4) **terraform state rm** this will remove the resource from the state file with out effecting the infrastructure
terraform will stop managing the resource if we remove the state of resource from the state file

5) **terraform state import** Imports an existing resource into the Terraform state.


### Terraform Workspace Commands


| **Command**               | **Purpose**                                                      | **Example Usage**                       |
|---------------------------|------------------------------------------------------------------|-----------------------------------------|
| `terraform state show`     | Display the current state of a resource.                         | `terraform state show aws_instance.web` |
| `terraform state mv`       | Move a resource within the Terraform state.                      | `terraform state mv aws_instance.web aws_instance.new_web` |
| `terraform state rm`       | Remove a resource from the state file.                           | `terraform state rm aws_instance.web`   |
| `terraform state pull`     | Retrieve the current state file from the backend.                | `terraform state pull`                  |
| `terraform state push`     | Upload a local state file to the remote backend.                 | `terraform state push terraform.tfstate`|
| `terraform state import`   | Import an existing resource into Terraform's state.              | `terraform state import aws_instance.web i-1234567890abcdef0` |


This document provides an overview of commonly used Terraform workspace commands and their examples.

| **Command**                      | **Purpose**                                                       | **Example Usage**                           |
|-----------------------------------|-------------------------------------------------------------------|---------------------------------------------|
| `terraform workspace new`         | Create a new workspace.                                           | `terraform workspace new dev`               |
| `terraform workspace select`      | Select an existing workspace.                                     | `terraform workspace select dev`            |
| `terraform workspace list`        | List all available workspaces.                                    | `terraform workspace list`                  |
| `terraform workspace show`        | Show the current workspace.                                       | `terraform workspace show`                  |
| `terraform workspace delete`      | Delete a workspace.                                               | `terraform workspace delete dev`            |


* Terraform Community (Free) stores the local state for workspaces in a directory called `terraform.tfstate.d/`. This directory structure allows for separate state files for each workspace, making it easier to manage and maintain the state data.


## Explanation of Commands:

### `terraform workspace new`
- **Purpose**: Creates a new workspace in Terraform. Each workspace corresponds to a distinct environment or configuration.
- **Example**: To create a new workspace called `dev`, run:
  ```bash
  terraform workspace new dev


### Modules 
Modules can be stored in several places, depending on use case and preferences

**Local Paths**:Store your modules as a directory in your local filesystem and refer to them using a relative or absolute path.

```
module "example" {
  source = "./modules/example-module"
}
```
**Terraform Registry (Public):** Use publicly available modules from the Terraform Registry or publish your own modules for reuse.
```
module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "3.5.0"
}
```
**Private Module Registries:** Use Terraform Cloud or Terraform Enterprise's private module registry.
```
module "example" {
  source = "app.terraform.io/my-organization/my-module/aws"
  version = "1.0.0"
}
```
If the module source begins with `app.terraform.io`, it is a private module.

### **How to downloaded modules** 

The two Terraform commands used to download and update modules are:



**terraform init:** This command downloads and updates the required modules for the Terraform configuration. It also sets up the backend for state storage if specified in the configuration.



**terraform get:** This command is used to download and update modules for a Terraform configuration. It can be used to update specific modules by specifying the module name and version number, or it can be used to update all modules by simply running the command without any arguments.

It's important to note that `terraform init` is typically run automatically when running other Terraform commands, so you may not need to run terraform get separately. However, if you need to update specific modules, running terraform get can be useful.

#### **To publish a module on the Terraform Registry, follow these requirements:**

* GitHub: Must be a public repo (for public registry only).
* Naming: Use terraform-<PROVIDER>-<NAME> format (e.g., terraform-aws-vpc).
* Description: Add a simple, clear repository description.
* Structure: Follow the standard module structure.
* Version Tags: Use semantic version tags (e.g., v1.0.0). only should start with V

## Commands

**terraform init:** Initializes a Terraform working directory.

* Downloads and installs provider plugins (e.g., AWS, Azure, Google Cloud).
* Configures the backend (e.g., local, remote).
* Prepares the directory for Terraform commands.

**terraform fmt:** Formats Terraform configuration files.
(more on styling and readability)
* Automatically fixes spacing, indentation, and other style issues.
* Ensures consistent code formatting.

**terraform validate:** Validates the syntax and structure of Terraform files.

* Checks for configuration syntax errors.
* Verifies provider requirements.

**terraform plan** Previews changes without applying them.

* Compares the current state with your configuration.
* Displays the resources that will be added, changed, or destroyed.
* Does not modify any infrastructure.

**terraform apply** Executes the changes required to reach the desired state.

* Applies the configuration changes to the infrastructure.
* Prompts for confirmation unless the -auto-approve flag is used.
* Terraform by default provisions 10 resources concurrently during a `terraform apply` command to speed up the provisioning process and reduce the overall time taken.

**terraform destroy** Deletes all resources managed by the Terraform configuration.

* Reads the configuration and destroys all associated resources.
* Prompts for confirmation unless -auto-approve is specified.

**terraform show** Displays the current state or a state file.

* Displays the current state or a state file.

Vedio on terraform Cloud : https://www.youtube.com/watch?v=w4YPGjAWmDc
