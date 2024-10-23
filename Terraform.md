
                                                    # Terraform

## What is Terraform?

Terraform is a HashiCorp tool that is cloud-agnostic, which means you can use the same logic to deploy resources across multiple clouds, including AWS, Azure, and GCP.

## Why Terraform?

Terraform offers greater flexibility and multi-cloud support compared to cloud-native tools like CloudFormation (CFT) and Azure Resource Manager (ARM). It simplifies resource management through modules, reusable code, and a powerful state management system.

### Key Differences between CNT (CFT, ARM) & Terraform:

| Feature                          | CFT & ARM                           | Terraform                      |
|-----------------------------------|--------------------------------------|---------------------------------|
| Language                          | JSON or YAML (All configs in one file) | HashiCorp Configuration Language (HCL) |
| Complexity                        | Learning JSON/YAML is difficult       | HCL is simpler and modular     |
| Cloud Compatibility               | AWS (CFT), Azure (ARM) only          | Multi-cloud (AWS, Azure, GCP)  |
| Module Support                    | No                                  | Yes, with reusable modules     |
| Workspace Support                 | No                                  | Yes, supports multiple workspaces |
| Dry-Run Capability                | Limited                             | `terraform plan` for effective dry-run |
| Importing Resources               | Complex in AWS, not available in ARM | Simple with `terraform import` |

## Data sources:

Data sources in Terraform is used to fetch information about existing resources, such as VPC IDs, subnet IDs, security group IDs, etc. Once you retrieve this information, you can use it in your Terraform configurations to create or manage other resources.

### Example:
Fetching AZ'S using data source and creating subnets in that AZ
``` 
data "aws_availability_zones" "available_zones" {
  state = "available"
}

#public subnets 
resource "aws_subnet" "pubsubnet1a" {
    vpc_id     = aws_vpc.myvpc.id
    availability_zone = data.aws_availability_zones.available_zones.names[0]
    cidr_block = var.pubsub1a_cidr_block
    map_public_ip_on_launch = true



  tags = {
    Name = "pubsubnet_us_east_1a"
  }
}
```
## What is Terraform import:

The terraform import command in Terraform is used to bring existing infrastructure under Terraform's management. It allows you to take resources that were created manually (or by another tool) and incorporate them into your Terraform state, without the need to recreate them.

## Why  terraform import?

#### Manage existing resources: 
If you have infrastructure that wasn’t created with Terraform, you can start managing it using Terraform by importing the resources.
#### Avoid downtime:
It helps you bring existing resources into Terraform without destroying or re-creating them, so there’s no downtime for critical services.
#### Infrastructure consolidation:
When migrating from manual cloud setups to Infrastructure as Code (IaC), terraform import allows for a smooth transition.

 ### Work-flow:

#### Step1:   Import the existing infrastructure:

You run the terraform import command to import the manually created infrastructure into Terraform's state file. This step only updates the state file but does not update or create any configuration in your .tf files.

```
terraform import aws_instance.my_instance i-1234567890abcdef
```
After running this command, Terraform will know about this resource (aws_instance.my_instance) and track its state.

#### Step 2:  Write the corresponding Terraform configuration

You need to manually write the Terraform configuration for the imported resource in your .tf files. The configuration must match the existing resource exactly

```
resource "aws_instance" "my_instance" {
  instance_type = "t2.micro"
  ami           = "ami-0abcdef1234567890"
  # Other attributes that reflect the current state
}
```
After writing the configuration, Terraform will manage this resource as part of its lifecycle.

Then run ``` terraform plan```  to verify.

## Difference between Data Source and Terraform import:

| Feature               | Terraform Import                          | Data Source                             |
|-----------------------|------------------------------------------|-----------------------------------------|
| **Use case**          | Manage an existing resource with Terraform | Fetch information from an existing resource |
| **Modifies state?**   | Yes, adds the resource to Terraform's state | No, only fetches data                   |
| **Manages lifecycle?**| Yes, Terraform manages the resource's lifecycle | No, Terraform doesn't control the resource |
| **Typical usage**     | Transitioning existing infra to Terraform | Referencing attributes like ID, region, status, etc. |

## State file:

The state file is a JSON file that stores the current state of your infrastructure. It acts as a source of truth for Terraform, keeping track of all resources

## Remote state file:

Storing the Terraform state file locally is not a best practice, especially in scenarios where multiple engineers work on the same infrastructure. Keeping the state file locally can lead to duplicate infrastructure, as each engineer has their own version of the state file, resulting in potential conflicts and inconsistencies. In contrast, storing the state file in a remote location facilitates easier monitoring and management of the infrastructure's state, ensuring that all team members have access to the most up-to-date information and reducing the risk of discrepancies.
versioning the state file helps to retrieve the state if there is any issues in the infrastructure.

## Dependencies in Terraform

### 1.Implicit Dependencies:
	
An implicit dependency occurs when one resource refers to the attribute of another resource. For example, when creating a VPC and then an Internet Gateway, the Internet Gateway doesn't inherently know that it must wait for the VPC to be created. However, when you reference the VPC ID in the Internet Gateway resource, Terraform understands that the VPC must be created first.

- **Example:**  When you declare a **VPC**, its ID is generated only after it is created. Any resource, like a **subnet** or **Internet Gateway**, that references this **VPC ID** creates an implicit dependency.

### 2. Explicit Dependencies
Sometimes, implicit dependencies aren’t enough. For example, if we want the **S3 bucket** to be created only after the VPC is created, we need to use explicit dependencies. This is done using the `depends_on` argument in Terraform.

- **Example:**  
  A **NAT Gateway** should only be created after a **Route Table** has been established. If the NAT Gateway is created before the route table, it won’t function as expected. This is where **explicit dependencies** come into play using `depends_on`.

## Terraform WorkSpaces:

Terraform workspaces provide a way to manage different environments like` DEV , UAT , PROD ` within the same Terraform configuration. They help maintain multiple copies of the infrastructure without needing separate directories or configuration files.

Need to deploy the infrastructure for each environment using the appropriate `.tfvars` file.

First need to create Workspaces to have different State files.
```
terraform workspace new DEV
terraform workspace new UAT
terraform workspace new PROD
```
List the workspace using the below command

`terraform workspace list`

Then Switch to required Workspace and create the infrastructure using the .tfvar file
```
terraform workspace select DEV
terraform apply -var-file=dev.tfvars

terraform workspace select UAT
terraform apply -var-file=uat.tfvars

terraform workspace select PROD
terraform apply -var-file=prod.tfvars
```

Based on the Workspace we are in infrastructure will be created and State will be updated in the respective state file. 

## Terraform state locking:

Terraform state locking using DynamoDB is essential in preventing concurrent operations that can corrupt the Terraform state. When multiple users or processes attempt to run terraform apply or terraform plan simultaneously, locking ensures that only one process can modify the state at a time.

**Example**:

You are managing a Terraform configuration that provisions AWS infrastructure (like EC2 instances, S3 buckets, etc.). Multiple team members are working on the infrastructure, and they could potentially run Terraform commands simultaneously, which might cause conflicts or corrupt the state. To prevent this, you implement state locking using a DynamoDB table.

## Terraform Modules

Terraform, modules are reusable blocks of code. They allow you to group related infrastructure resources and reuse that configuration across multiple projects.

A module is essentially a directory that contains .tf files defining resources, variables, outputs, and optionally, input variables and outputs. By encapsulating infrastructure configurations within modules, you can write them once and reuse them many times, making your Terraform code more modular, manageable, and consistent.

## Enabling tf_logs:

Terraform provides the `TF_LOG` environment variable for controlling log verbosity. You can choose from different levels like `TRACE`, `DEBUG`, `INFO`, `WARN`, and `ERROR`.

**Example**:
#### 1.Set TF_LOG for detailed trace logs:
```
powershell:

$env:TF_LOG = "TRACE"
terraform destroy or apply

linux:
TF_LOG="TRACE"
terraform destroy
```

#### 2.Set TF_LOG for error-level logging:

```
$env:TF_LOG = "ERROR"
terraform destroy or apply
```
We can store this logs in file using `$env:TF_LOG_PATH= /path` environmental variable
```
$env:TF_LOG = "ERROR"
$env:TF_LOG_PATH = "./logs/terraform.log"  # use this variable to store the logs.
terraform apply
```
**Reminder**
we need to define environment variable  `$env:TF_LOG` every time when session get expire.

## Index
this fetch the value from the list of elements.
**Example**
```
variable "AZ" {
    type = "string"
}

AZ = ["us-east-1a", "us-east-1b", "us-east-1c"]

index value: [0,               1,              2]
```
## Count Meta-Argument
count is a meta-argument in Terraform that is used to create multiple instances of a resource or module based on a condition or a variable. It controls how many instances of a resource should be created.

### When to Use count:
* When you need to create multiple instances of a resource.
* When you want to conditionally create a resource based on some variable or condition.
* When you need to dynamically control the number of resources based on some list or map.

**Basic example**
```
resource "aws_instance" "example" {
  count = 3

  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"

  tags = {
    Name = "example-instance-${count.index}"
  }
}
```
### 2. Conditional Resource Creation:

Using count with a conditional expression to control whether or not a resource should be created:

```
variable "create_instance" {
  default = true
}

resource "aws_instance" "example" {
  count = var.create_instance ? 1 : 0  # If true, 1 instance will be created; otherwise, 0.

  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"
}
```

### 3.Dynamic Resource Creation with List Length:
You can also use count to dynamically create resources based on a list's length:
```
variable "availability_zones" {
  default = ["us-west-1a", "us-west-1b", "us-west-1c"]
}

resource "aws_subnet" "example" {
  count = length(var.availability_zones)

  vpc_id            = "vpc-123456" 
  cidr_block        = "10.0.${count.index}.0/24"
  availability_zone = var.availability_zones[count.index]
}
```
The number of subnets created will depend on the number of availability zones provided in the list.

## Element Function
element retrieves a single element from a list.

`element(list, index)`

**example:**

```
variable "AZ" {
  type    = list(string)
  default = ["us-west-1a", "us-west-1b", "us-west-1c"]
}

element(var.AZ,${count.index})
```
Every time it iterate it will fetch a value from the list of AZ's


The index is zero-based. This function produces an error if used with an empty list. The index must be a non-negative integer.


## Terraform Lookup 

The lookup function in Terraform is used to retrieve a value from a map based on a key, and it can also return a default value if the key isn't found.

**Example**
```
# Variable definitions
variable "region" {
  default = "us-east-1"
}

variable "ami" {
  default = {
    us-east-1 = "ami.isfcinrnci3423"
    us-east-2 = "ami.isfcinrnci3444343"
  }
}

# Lookup function to fetch the AMI ID based on the region
ami_id = lookup(var.ami, var.region, "ami.default")
```

## For each:
n Terraform, for_each is a meta-argument that allows you to iterate over a collection (such as a list, map, or set) to create multiple instances of a resource or nested blocks. It’s commonly used when you want to define multiple resources or blocks dynamically based on the elements in the collection.


**for_each with Lists:** Terraform will iterate over each element in a list. The current value in the iteration can be accessed using .value.

**for_each with Maps:** When using a map, the key-value pair of the map is available during the iteration, where .key holds the key and .value holds the associated value.

`for_each = <collection>`  # List or map

## Splat Function:

splat expression is a shorthand way to extract values from a list or set of resources in Terraform.

**Example:**


Let's say you create multiple subnets using the count meta-argument
```
resource "aws_subnet" "public-subnet" {
  count             = length(var.public_cidr_block)
  vpc_id            = aws_vpc.default.id
  cidr_block        = element(var.public_cidr_block, count.index)
  availability_zone = element(var.azs, count.index)
}
```
Here, 3 subnet instances are being created, each with a different CIDR block. Each subnet will have its own attributes, like id, arn, etc.

### Using Splat Expression to Extract IDs:
```
output "subnet_ids" {
  value = aws_subnet.example.*.id  # Using the splat expression to collect all subnet IDs
}

output:

subnet_ids = [
  "subnet-abc123",
  "subnet-def456",
  "subnet-ghi789"
]

```
###  How to associate subnets to route table using splat:
**Creating a Route Table:**

You create a route table resource:
```
resource "aws_route_table" "public" {
  vpc_id = aws_vpc.main.id

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.main.id
  }
}
```
### Using Splat Expression for Route Table Associations:

To associate all the created subnets with this route table, you can use a splat expression to gather all the subnet IDs and create associations for each one. This is where splat expressions come in handy.

```
resource "aws_route_table_association" "public_association" {
  count          = length(aws_subnet.public.*.id)
  subnet_id      = element(aws_subnet.public.*.id, count.index)
  route_table_id = aws_route_table.public.id
}

```
**If not we need to create route table associations manually as follow:**
```
resource "aws_subnet" "public" {
  count = 3
  vpc_id     = aws_vpc.main.id
  cidr_block = element( var.public_cidr_block, count.index)
}
```
In this case, Terraform will create 3 subnets:
```
aws_subnet.public[0]: First subnet
aws_subnet.public[1]: Second subnet
aws_subnet.public[2]: Third subnet
```
Each instance of `aws_subnet.public` has its own `id`, and you can reference each one separately using its index.

**Manual process:**
```
resource "aws_route_table_association" "public_association_1" {
  subnet_id      = aws_subnet.public[0].id  # Refers to the first subnet's ID
  route_table_id = aws_route_table.public.id
}

resource "aws_route_table_association" "public_association_2" {
  subnet_id      = aws_subnet.public[1].id  # Refers to the first subnet's ID
  route_table_id = aws_route_table.public.id
}

resource "aws_route_table_association" "public_association_3" {
  subnet_id      = aws_subnet.public[1].id  # Refers to the first subnet's ID
  route_table_id = aws_route_table.public.id
}


```

## Local values in terraform:

A local value assigns a name to an expression, so you can use the name multiple times within a module instead of repeating the expression.

#### Declaring a Local Value:

A set of related local values can be declared together in a single locals block:

```
locals {
    service_name = "forum"
    owner        = "Community Team"
}
```

used as

```
tags = {
    owner       = local.owner
    service_name = local.service_name   
}
```

## Dynamic function:

In Terraform, dynamic blocks provide a way to dynamically generate repeated nested blocks within resource, data, provider, and provisioner blocks. They're commonly used in resource blocks to make your configurations more flexible and follow the "Don't Repeat Yourself" (DRY) principle.

**Basic structure**
```
resource "resource_type" "resource_name" {
  # Resource block configuration

  dynamic "label" {
    for_each = collection_to_iterate
    iterator = item

    content {
      # Content of the dynamically generated block
    }
  }
}
```

**Example:**
```
resource "aws_security_group" "allow_all" {
  name        = "${var.vpc_name}-allow-all"
  description = "Allow all Inbound traffic"
  vpc_id      = aws_vpc.default.id

  # Ingress rule block with dynamic iteration over service_ports
  dynamic "ingress" {
    for_each = var.ingress_value
    content {
      from_port   = ingress.value  # ingress.value refers to the value of ingress in present iteration
      to_port     = ingress.value
      protocol    = "tcp"
      cidr_blocks = ["0.0.0.0/0"]  # Allow traffic from any IP
    }
  }

  # Egress rule block
  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]  # Allow outbound traffic to any IP
  }

  # Tags block
  tags = {
    Name        = "${var.vpc_name}-allow-all"
    Owner       = local.Owner
    costcenter  = local.costcenter
    TeamDL      = local.TeamDL
    environment = var.environment
  }
}
```
**Explanation:**

ingress is the name of the block that is being dynamically generated.

ingress.value refers to the current value in the iteration, meaning the value from var.ingress_value during the specific iteration.

**Here's a breakdown:**

dynamic "ingress": This tells Terraform to dynamically create ingress blocks.

for_each = var.ingress_value: This iterates over the values of var.ingress_value, which could be a list of port numbers (e.g., [22, 80, 443]).

ingress.value: Inside the content block, ingress.value refers to the current value in the iteration from var.ingress_value. For example, on the first iteration, ingress.value would be 22 (if the list is [22, 80, 443]), on the second iteration, it would be 80, and so on.

**Key Point:**

ingress is just the name of the dynamic block, and .value is the way to access the current value of the item being iterated over.
So, ingress is the block name you provided in the dynamic block, and ingress.value refers to the value of the current iteration of var.ingress_value.

## Conditional Expressions (ternary operator)


In Terraform, conditions allow you to control resource creation, variable values, and expressions based on logic

`condition ? true_value : false_value `

**Use Case:**
Creating number of instance based on environment

If environment is prod then create 3 instances else create 1 instance
```
resource "aws_instance" "private-server" {
  count                  = var.environment == "Prod" ? 3 : 1
  ami                    = lookup(var.amis, var.aws_region)
  instance_type          = "t2.micro"
  key_name               = var.key_name
  subnet_id              = element(aws_subnet.private-subnet.*.id, count.index)
  vpc_security_group_ids = [aws_security_group.allow_all.id]
}
```
### **File provisioner:**

This allow as to copy file from local to remote (CREATED  VM's) then we can use remote exec to run the commands on it.
```
provisioner "file" {
  source      = "user_data.sh"                             # Local file path to copy
  destination = "/tmp/user_data.sh"                       # Destination path on the remote instance

  connection {
    type        = "ssh"                                   # Connection type
    user        = "ubuntu"                                # SSH user for the connection
    private_key = file("LaptopKey.pem")                   # Path to the SSH private key
    host        = element(aws_instance.public-servers.*.public_ip, count.index)  # Remote instance's public IP
  }
}

```

## **Local Exec:**
The local-exec provisioner in Terraform allows you to run a command locally on the machine where Terraform is being run. It is useful when you need to execute local scripts, commands, or any other kind of shell commands as part of your Terraform infrastructure management.

**Example of local-exec:**

```
resource "null_resource" "example" {
  provisioner "local-exec" {
    command = "echo 'Hello, World!' > hello.txt"
  }
}
```
### **Remote-exec:**

The remote-exec provisioner in Terraform allows you to run a command remotely on a machine provisioned by Terraform, such as an EC2 instance. This is useful when you need to execute commands or scripts on the actual instance (or resource) after it is created, to configure it or deploy software.

**Example of remote-exec:**

```
provisioner "remote-exec" {
  inline = [
    "sudo chmod 777 /tmp/userdata.sh",   # Changes permissions to allow execution
    "sudo /tmp/userdata.sh",               # Executes the script
    "sudo apt update",                      # Updates the package list
    "sudo apt install jq unzip -y",        # Installs jq and unzip without prompting for confirmation
  ]

  connection {
    type        = "ssh"                    # Uses SSH to connect
    user        = "ubuntu"                 # User for the connection
    private_key = file("SecOps-Key.pem")  # Path to the private key for SSH authentication
    host        = element(aws_instance.public-server.*.public_ip, count.index)  # Host's public IP
  }
}

```
## **Null Resource:**
The null_resource in Terraform is a placeholder resource that doesn’t represent any actual cloud infrastructure but is used for:

Executing provisioners (like local-exec or remote-exec).

**Example of null_resource:**
```
resource "null_resource" "example" {
  provisioner "local-exec" {
    command = "echo 'Resource triggered!'"
  }

  triggers = {
    resource_id = aws_instance.example.id
  }
}
```
In this example:

The null_resource doesn’t create any infrastructure, but it will run the local-exec command echo 'Resource triggered!'.

The triggers argument ensures that the null_resource is recreated and the provisioner re-executed whenever the aws_instance.example.id changes.
## Taint:

taint is a command used to mark a resource for recreation during the next terraform apply. When a resource is tainted, Terraform treats it as needing to be replaced, even if its configuration has not changed. This is typically used when a resource is in a bad state or when you want to force the re-execution of provisioners associated with that resource.
##  **usecase on provisions, taints :**

if we give user data in resource block of ec2 instance every time we modfiy the script instances already created sing that block need's to be deleted and new instaces will created wil updated script.

in order to change this we can use null Resource with file provisioner, remote exec and taints to do the operation effectively 

## **Steps:**
**Create the Instance:** Provision an EC2 instance using the aws_instance resource.

**Use File Provisioner:** Utilize the file provisioner to transfer the user_data.sh script to the EC2 instance.

**Use Remote Exec:** Use the remote-exec provisioner to execute the script on the EC2 instance.

**Use Null Resource Block:** Encapsulate the file transfer and script execution in a null_resource block.

**First Run:** On the first run, you successfully install and run the application with the original user data script.

**Update User Data:** If you modify the user_data.sh script and run terraform apply, Terraform will not recreate the instance because it checks for changes in the instance itself, not in the provisioner scripts.

**Tainting the Resource:** To force the null_resource to run again, you can taint it. it marks the resource as needing to be replaced during the next terraform apply.( it won't remove the state of the resource )

**Run Terraform Apply:** After tainting the null_resource, run terraform apply again. You should see the modified data reflected on the EC2 instance.


