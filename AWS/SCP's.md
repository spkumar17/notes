# 🔐 AWS Service Control Policies (SCPs) vs IAM Policies: Q&A Guide

## 🤔 What are Service Control Policies (SCPs)?

**Answer:** SCPs are organization-wide permission guardrails in AWS Organizations that define the **maximum permissions** available to accounts, users, and roles. They act as a security boundary but don't grant permissions directly.

---

## 🤔 What are IAM Policies?

**Answer:** IAM Policies are JSON documents that define specific permissions for users, groups, and roles within a **single AWS account**. They grant or deny access to AWS services and resources.

---

## 🤔 What's the key difference between SCPs and IAM Policies?

**Answer:**

| Aspect | SCPs | IAM Policies |
|--------|------|-------------|
| **Scope** | Organization/OU level (multiple accounts) | Single account level |
| **Purpose** | Set permission boundaries (what's allowed) | Grant specific permissions (what you can do) |
| **Effect** | Restrict maximum permissions | Grant actual permissions |
| **Inheritance** | Applied to all accounts in OU | Applied to specific users/roles |

**🔑 Key Point:** SCPs define the "ceiling" of permissions, while IAM policies define the "floor" of what users actually get.

---

## 🤔 Do SCPs grant permissions?

**Answer:** **No!** SCPs only **restrict** permissions. They define what actions are **allowed** to be granted by IAM policies, but they don't grant permissions themselves.

**Example:**
- SCP allows `s3:*` actions
- IAM policy grants `s3:GetObject`
- **Result:** User can only perform `s3:GetObject` (intersection of both)

---

## 🤔 What are the two types of SCPs?

**Answer:**

### 1. **Allow SCPs** (Whitelist approach)
- Only explicitly listed actions are allowed
- More restrictive by default
- Example: Only allow EC2 and S3 services

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ec2:*",
        "s3:*"
      ],
      "Resource": "*"
    }
  ]
}
```

### 2. **Deny SCPs** (Blacklist approach)
- All actions allowed except explicitly denied ones
- Less restrictive by default
- Example: Deny access to expensive services

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Deny",
      "Action": [
        "sagemaker:*",
        "redshift:*"
      ],
      "Resource": "*"
    }
  ]
}
```

---

## 🤔 Can you provide a real-world SCP example?

**Answer:** Here's an SCP that prevents accounts from launching expensive EC2 instances:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "DenyExpensiveInstances",
      "Effect": "Deny",
      "Action": [
        "ec2:RunInstances"
      ],
      "Resource": "arn:aws:ec2:*:*:instance/*",
      "Condition": {
        "StringNotEquals": {
          "ec2:InstanceType": [
            "t3.micro",
            "t3.small",
            "t3.medium"
          ]
        }
      }
    }
  ]
}
```

**What this does:** Even if an IAM policy allows launching any EC2 instance, this SCP restricts it to only t3.micro, t3.small, and t3.medium instances.

---

## 🤔 What's a typical IAM Policy structure?

**Answer:** IAM policies have four main components:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::my-bucket/*",
      "Condition": {
        "IpAddress": {
          "aws:SourceIp": "203.0.113.0/24"
        }
      }
    }
  ]
}
```

| Component | Purpose | Example |
|-----------|---------|---------|
| **Effect** | Allow or Deny | `"Allow"` |
| **Action** | What operations | `"s3:GetObject"` |
| **Resource** | Which resources | `"arn:aws:s3:::my-bucket/*"` |
| **Condition** | When it applies | IP address restrictions |

---

## 🤔 How do SCPs and IAM Policies work together?

**Answer:** They work through **permission intersection**:

**Scenario:** Developer needs S3 access

1. **SCP (Organization level):** Allows `s3:*`, `ec2:*`
2. **IAM Policy (User level):** Allows `s3:GetObject`, `dynamodb:*`
3. **Effective Permissions:** Only `s3:GetObject` (intersection)

**🔍 Why?** 
- DynamoDB is blocked by SCP (not in allowed list)
- Other S3 actions blocked by IAM policy (not granted)

---

## 🤔 When should I use SCPs vs IAM Policies?

**Answer:**

### Use **SCPs** when:
- ✅ Managing multiple AWS accounts
- ✅ Enforcing organization-wide security policies
- ✅ Preventing access to specific services/regions
- ✅ Compliance requirements across accounts
- ✅ Cost control (blocking expensive services)

### Use **IAM Policies** when:
- ✅ Granting specific permissions to users/roles
- ✅ Fine-grained access control within an account
- ✅ Application-specific permissions
- ✅ Temporary access (assume roles)
- ✅ Resource-level permissions

---

## 🤔 Can SCPs override IAM policies?

**Answer:** **Yes, SCPs always win!** If an SCP denies an action, no IAM policy can override it.

**Example:**
- SCP: Denies `ec2:TerminateInstances`
- IAM Policy: Allows `ec2:*`
- **Result:** User cannot terminate instances (SCP denial takes precedence)

---

## 🤔 What are common SCP use cases?

**Answer:**

### 1. **Region Restriction**
```json
{
  "Effect": "Deny",
  "Action": "*",
  "Resource": "*",
  "Condition": {
    "StringNotEquals": {
      "aws:RequestedRegion": ["us-east-1", "us-west-2"]
    }
  }
}
```

### 2. **Prevent Root User Actions**
```json
{
  "Effect": "Deny",
  "Action": "*",
  "Resource": "*",
  "Condition": {
    "StringEquals": {
      "aws:PrincipalType": "Root"
    }
  }
}
```

### 3. **Block Expensive Services**
```json
{
  "Effect": "Deny",
  "Action": [
    "sagemaker:*",
    "redshift:*",
    "emr:*"
  ],
  "Resource": "*"
}
```

---

## 🤔 What happens if there's no SCP attached?

**Answer:** **Full permissions are available** (within IAM policy limits). SCPs are "opt-in" restrictions - without them, IAM policies have full effect within the account.

---

## 🤔 Can I apply SCPs to the master account?

**Answer:** **No!** SCPs cannot be applied to the AWS Organizations master (management) account. The master account always has full permissions to manage the organization.

---

## 🤔 How do I troubleshoot SCP permission issues?

**Answer:** Use these steps:

### 1. **Check CloudTrail Logs**
Look for `AccessDenied` errors with `errorCode: "AccessDenied"` and `errorMessage` containing "SCP"

### 2. **Use IAM Policy Simulator**
- Test specific actions against your policies
- Shows both IAM and SCP evaluation results

### 3. **Review SCP Hierarchy**
- Check SCPs at Organization root level
- Check SCPs at OU level
- Check SCPs at account level

### 4. **Common Issues**
| Issue | Cause | Solution |
|-------|-------|----------|
| "Access Denied" despite IAM permissions | SCP blocking action | Review SCP deny statements |
| Unexpected service restrictions | Inherited SCP from parent OU | Check OU hierarchy |
| Root user blocked | SCP denying root actions | Modify SCP or use IAM user |

---

## 🎯 Key Takeaways

1. **SCPs = Organization-wide guardrails** that set maximum permissions
2. **IAM Policies = Account-specific permissions** that grant actual access
3. **SCPs don't grant permissions** - they only restrict what can be granted
4. **Effective permissions = IAM Policy ∩ SCP** (intersection)
5. **SCPs always override IAM policies** when there's a conflict
6. **Use SCPs for compliance and governance** across multiple accounts
7. **Use IAM policies for day-to-day access management** within accounts

---

## 📚 Quick Reference

| Need | Use | Example |
|------|-----|---------|
| Block service organization-wide | SCP Deny | Deny `sagemaker:*` |
| Allow only specific regions | SCP Allow | Allow only `us-east-1` |
| Grant S3 read access | IAM Policy | Allow `s3:GetObject` |
| Temporary elevated access | IAM Assume Role | Cross-account access |
| Prevent accidental deletions | SCP Deny | Deny `*:Delete*` actions |

This comprehensive guide helps you understand when and how to use SCPs vs IAM Policies effectively! 🚀