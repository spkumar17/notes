# 🔐 Cross-Account IAM Role Access in AWS

This guide demonstrates how to set up a cross-account IAM role in AWS. It enables an IAM user or role in **Account A** to assume an IAM role in **Account B** using a trust relationship and `sts:AssumeRole`.

---

## 📘 Scenario

We have two AWS accounts:

- **Account A** (Trusted Account)
  - IAM user: `crossAccountUser`
  - AWS Account ID: `111111111111`

- **Account B** (Trusting Account)
  - IAM role: `CrossAccountAccessRole`
  - AWS Account ID: `222222222222`

Account A's user will assume the IAM role in Account B.

---

## 🛠️ Step-by-Step Setup

### 1️⃣ Create the IAM Role in Account B

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

### 2️⃣ Attach Permissions to the Role in Account B

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

### 3️⃣ Assume the Role from Account A

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

## 🧠 Notes

- The trust relationship must be manually added to the role in Account B.
- The IAM user in Account A must have permission to call `sts:AssumeRole`.

---

## ✅ Summary

| Task | Location |
|------|----------|
| Create IAM Role | Account B |
| Define Trust Policy | Account B |
| Attach Permissions to the Role | Account B |
| User Assumes Role via STS | Account A |

---


## 🚀 Advanced Usage

For more complex scenarios, consider:
- Using condition keys in trust policies
- Implementing MFA requirements for role assumption
- Setting up cross-account access for AWS services (not just users)
