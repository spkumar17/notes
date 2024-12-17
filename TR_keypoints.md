
# TERRAFORM 

## IAC
* IaC is written using a **declarative** approach (**not imperative**)
* IaC allows you to commit your configurations to version control to safely collaborate on infrastructure(**versioned, shared, and reused**)
* IaC code can be used to manage infrastructure on multiple cloud platforms (**platform agnostic**)

* IaC uses a human-readable configuration language to help you write infrastructure code quickly
* Terraform is an **immutable, declarative, Infrastructure as Code** provisioning language based on Hashicorp Configuration Language **HCL**, or optionally **JSON**.
* Terraform is primarily designed for **macOS, FreeBSD, OpenBSD, Linux, Solaris, and Windows**
## Points on state file
* Terraform relies on state files to track the current state of infrastructure resources and compare it to the desired state declared in configuration files. Without a state file, Terraform would not be able to accurately determine the changes needed to align real-world resources with the desired state.

## PLUGINS

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







