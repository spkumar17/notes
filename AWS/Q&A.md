# AWS Q&A

## Question 1

### If Control Tower already has a Management Account, why do we need a separate AFT Management Account?

**Answer:**

Although the Control Tower Management Account is responsible for setting up and governing the landing zone (e.g., creating OUs, applying SCPs, and managing guardrails), it is **not recommended** to run automation workloads in this account.

AWS recommends provisioning a **separate AFT Management Account** to host the Account Factory for Terraform (AFT) pipelines. This follows the **principle of least privilege** and ensures a **clear separation of concerns**.

### The AFT Management Account is used to:

- Run the Terraform automation pipelines  
- Host resources like **CodePipeline**, **CodeBuild**, **Lambda**, etc.  
- Trigger account creation and customizations based on Git workflows  

This separation improves **security**, **auditability**, and **operational clarity**, ensuring the Control Tower Management Account remains focused purely on **governance**, not **deployment logic**.

## Question 1
### What are Service Control Policies (SCPs) and IAM Policies in AWS?

**Answer:**
**Service Control Policies (SCPs)**

Service Control Policies (SCPs) are a feature of AWS Organizations that help you manage permissions across multiple AWS accounts in an organization. SCPs allow you to set permission guardrails to control the maximum available permissions for IAM roles, users, or services within the accounts in your AWS Organization.

**Key points about SCPs:**

* Organization-Wide Control: SCPs work at the organization or organizational unit (OU) level, which means they can be applied to multiple accounts simultaneously.

* Max Permissions: They do not grant permissions directly; instead, they restrict what permissions can be granted by IAM policies, ensuring that accounts can only perform actions allowed by the SCP.

* Permissions Boundaries: SCPs define what actions are allowed or denied in terms of services, resources, and actions across your accounts. They don’t replace IAM policies but limit the scope of what an IAM policy can do within a given account.

Two Types of SCPs:

* **Allow SCP:** Permissions are allowed within the specified boundary.

* **Deny SCP:** Denies actions across the organization or specific accounts, even if IAM policies grant access.

**Examples of Use:** You might want to restrict certain accounts from using specific AWS services (e.g., EC2 or S3), or perhaps only allow access to particular services in a certain region.

**IAM Policies**
IAM Policies are used to define permissions for individual users, groups, and roles within a single AWS account. These policies are written in JSON format and specify what actions (like s3:PutObject, ec2:StartInstances) a user or role can perform on which resources (e.g., EC2 instances, S3 buckets).

**Key points about IAM Policies:**

* **User-Specific Permissions:** IAM policies can be attached to users, groups, or roles, granting them the necessary permissions to access AWS services and resources.

* **Granular Permissions:** IAM policies are used to define specific actions on specific resources. For example, an IAM policy might allow a user to only read from one S3 bucket but not modify anything in it.

* Structure of IAM Policy: 
An IAM policy is composed of the following elements

* **Effect:** Can be Allow or Deny (decides whether the action is allowed or not).

* **Action:** Defines what actions (operations) are allowed or denied (e.g., ec2:DescribeInstances).

* **Resource:** Specifies which resources the actions apply to (e.g., specific S3 bucket or EC2 instance).

* **Condition:** Optional, allows you to specify conditions for when the policy is applied (e.g., only during a certain time of day).

* **Examples of Use:** If you want to allow a developer to manage EC2 instances but not delete them, you can create an IAM policy that grants ec2:StartInstances and ec2:StopInstances permissions but denies ec2:TerminateInstances.

