# 🔐 AWS Cross-Account IAM Role Access Guide

This comprehensive guide demonstrates how to set up cross-account IAM roles in AWS for both **IAM users** and **AWS services**. It enables secure access between different AWS accounts using trust relationships and `sts:AssumeRole`.

---

## 📋 Table of Contents

- [User Cross-Account Access](#user-cross-account-access)
- [Service Cross-Account Access](#service-cross-account-access)
- [Security Best Practices](#security-best-practices)
- [Troubleshooting](#troubleshooting)
- [Additional Resources](#additional-resources)

---

## 👤 User Cross-Account Access

### 📘 Scenario

We have two AWS accounts:

- **Account A** (Trusted Account)
  - IAM user: `crossAccountUser`
  - AWS Account ID: `111111111111`

- **Account B** (Trusting Account)
  - IAM role: `CrossAccountAccessRole`
  - AWS Account ID: `222222222222`

Account A's user will assume the IAM role in Account B.

### 🛠️ Step-by-Step Setup

#### 1️⃣ Create the IAM Role in Account B

Create a trust policy (`trust-policy.json`) to allow Account A's user to assume this role:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::111111111111:user/crossAccountUser"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

Create the role:

```bash
aws iam create-role \
  --role-name CrossAccountAccessRole \
  --assume-role-policy-document file://trust-policy.json
```

#### 2️⃣ Attach Permissions to the Role in Account B

For example, allow the role to list all S3 buckets:

Create `role-policy.json`:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "s3:ListAllMyBuckets",
      "Resource": "*"
    }
  ]
}
```

Attach the policy:

```bash
aws iam put-role-policy \
  --role-name CrossAccountAccessRole \
  --policy-name S3ReadPolicy \
  --policy-document file://role-policy.json
```

#### 3️⃣ Assume the Role from Account A

From Account A, run:

```bash
aws sts assume-role \
  --role-arn arn:aws:iam::222222222222:role/CrossAccountAccessRole \
  --role-session-name dev-session
```

The output will include temporary security credentials (AccessKeyId, SecretAccessKey, and SessionToken).

Export these credentials:

```bash
export AWS_ACCESS_KEY_ID=...
export AWS_SECRET_ACCESS_KEY=...
export AWS_SESSION_TOKEN=...
```

Test it:

```bash
aws s3 ls
```

---

## 🔧 Service Cross-Account Access

To set up cross-account access for AWS services, not just IAM users, you follow the same core principle:
- ✅ Create a trust relationship in the target role in Account B
- ✅ Allow specific services from Account A to assume that role

### 📘 Example Scenario

- **Account A** – Where a service like Lambda, EC2, or EKS is running
- **Account B** – Where the role lives, and which has the resources the service needs to access (e.g., S3 bucket, DynamoDB, etc.)

You want a Lambda in Account A to access resources (e.g., S3) in Account B.

### 🛠️ Step-by-Step Guide

#### Step 1: Create IAM Role in Account B

This role must include:
- A trust policy that allows the AWS service (via a role in Account A) to assume it
- The necessary permissions to access the AWS resources in Account B

**🔐 Trust Policy Example (Account B Role)**

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::111111111111:role/LambdaExecutionRoleFromAccountA"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

🔹 Replace `111111111111` with Account A's ID and `LambdaExecutionRoleFromAccountA` with the role used by the Lambda.

#### Step 2: Attach Permissions to Role in Account B

For example, if the Lambda should read from an S3 bucket:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::account-b-bucket-name/*"
    }
  ]
}
```

#### Step 3: In Account A – Assume Role from Service

Ensure the service (Lambda, EC2, etc.) in Account A uses an IAM role that allows it to call:

```json
{
  "Effect": "Allow",
  "Action": "sts:AssumeRole",
  "Resource": "arn:aws:iam::222222222222:role/CrossAccountServiceAccessRole"
}
```

Then, from within the Lambda or EC2 application code (e.g., in Python/Boto3):

```python
import boto3

sts = boto3.client('sts')
assumed_role = sts.assume_role(
    RoleArn="arn:aws:iam::222222222222:role/CrossAccountServiceAccessRole",
    RoleSessionName="CrossAccountSession"
)

credentials = assumed_role['Credentials']

s3 = boto3.client(
    's3',
    aws_access_key_id=credentials['AccessKeyId'],
    aws_secret_access_key=credentials['SecretAccessKey'],
    aws_session_token=credentials['SessionToken']
)

response = s3.list_objects_v2(Bucket='account-b-bucket-name')
```

### 🔧 Common Service Examples

#### Lambda Cross-Account Access

**Trust Policy for Lambda Service:**

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "lambda.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    },
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::111111111111:role/LambdaExecutionRole"
      },
      "Action": "sts:AssumeRole",
      "Condition": {
        "StringEquals": {
          "sts:ExternalId": "unique-external-id"
        }
      }
    }
  ]
}
```

#### EC2 Cross-Account Access

**Trust Policy for EC2 Service:**

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "ec2.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    },
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::111111111111:role/EC2CrossAccountRole"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

#### CodePipeline Cross-Account Access

**Trust Policy for CodePipeline Service:**

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "codepipeline.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    },
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::111111111111:role/CodePipelineServiceRole"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

---

## 📊 Summary Tables

### User Cross-Account Access

| Task | Location | Notes |
|------|----------|-------|
| Create IAM Role | Account B | Define trust policy |
| Define Trust Policy | Account B | Allow specific user from Account A |
| Attach Permissions to the Role | Account B | Grant necessary resource access |
| User Assumes Role via STS | Account A | Use temporary credentials |

### Service Cross-Account Access

| Task | Account | Notes |
|------|---------|-------|
| Create IAM Role with Trust Policy | Account B | Trust the role from Account A |
| Add Resource Access Permissions | Account B | e.g., S3, DynamoDB, etc. |
| Use sts:AssumeRole from Service | Account A | Service role must allow this |
| Use temporary credentials | Account A | Use them to call services in Account B |

---

## 🔒 Security Best Practices

### General Best Practices

- **Use least privilege principle** when defining permissions
- **Regularly rotate temporary credentials**
- **Monitor cross-account access** through CloudTrail
- **Consider using external ID** for additional security
- **Implement MFA requirements** for sensitive role assumptions

### Service-Specific Best Practices

- **Use External IDs**: Add external ID conditions for additional security
- **Time-based Access**: Use condition keys to limit access windows
- **Monitor Access**: Use CloudTrail to track cross-account activities
- **Regular Audits**: Review and rotate access keys regularly
- **Resource-based Policies**: Combine with resource policies for defense in depth

### Example Security Conditions

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::111111111111:role/ServiceRole"
      },
      "Action": "sts:AssumeRole",
      "Condition": {
        "StringEquals": {
          "sts:ExternalId": "unique-external-id"
        },
        "DateGreaterThan": {
          "aws:CurrentTime": "2024-01-01T00:00:00Z"
        },
        "DateLessThan": {
          "aws:CurrentTime": "2024-12-31T23:59:59Z"
        },
        "IpAddress": {
          "aws:SourceIp": "203.0.113.0/24"
        }
      }
    }
  ]
}
```

---

## 🧠 Important Notes

- The trust relationship must be manually added to the role in Account B
- The IAM user/service in Account A must have permission to call `sts:AssumeRole`
- Temporary credentials have a limited lifetime (default 1 hour, max 12 hours)
- Cross-account access can be combined with resource-based policies for additional security layers

---

## 🔍 Troubleshooting

### Common Issues

1. **Access Denied when assuming role**
   - Check trust policy in target role
   - Verify source principal has `sts:AssumeRole` permission
   - Confirm account IDs are correct

2. **Temporary credentials expired**
   - Re-assume the role to get fresh credentials
   - Consider implementing automatic credential refresh

3. **Service can't assume role**
   - Verify service principal is included in trust policy
   - Check that service role has necessary permissions
   - Ensure external ID matches if required

### Debugging Commands

```bash
# Test role assumption
aws sts get-caller-identity

# Decode STS error messages
aws sts decode-authorization-message --encoded-message <encoded-message>

# List assumable roles
aws iam list-roles --query 'Roles[?contains(AssumeRolePolicyDocument, `sts:AssumeRole`)]'
```

---


## 🚀 Advanced Usage

For more complex scenarios, consider:
- Using condition keys in trust policies for enhanced security
- Implementing MFA requirements for role assumption
- Setting up cross-account access for multiple AWS services
- Automating role creation with Infrastructure as Code (Terraform, CloudFormation)
- Implementing cross-account resource sharing with AWS Resource Access Manager (RAM)

---
