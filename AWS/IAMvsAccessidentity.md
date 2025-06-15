# AWS IAM User vs AWS Identity Center User — Key Concepts and Takeaways

## 🔍 Overview

This document outlines the differences between **IAM users** and **AWS Identity Center (formerly AWS SSO)** users, explains the concept of **temporary credentials**, and highlights best practices for managing access in AWS.

---

## 👤 IAM User

- Created and managed **within a single AWS account**
- Uses **static access keys** and optionally a **username/password** for AWS Console
- Credentials **do not expire** unless manually rotated or deleted
- Can be used for:
  - Console access (via username/password)
  - CLI/API access (via access keys)
- Not ideal for large-scale, multi-account setups
- ⚠️ Security risk: long-lived credentials can be exploited if leaked

---

## 🧑‍💼 AWS Identity Center (SSO) User

- Managed **centrally** via AWS Identity Center
- Can integrate with **external IdPs** (e.g., Okta, Azure AD, Active Directory)
- One login gives access to **multiple AWS accounts**
- Login URL format:
https://yourcompany.awsapps.com/start

- Uses **temporary credentials** for CLI/API access
- ✅ Recommended for human access in modern AWS orgs

---

## 🔐 Temporary Access Credentials

- Issued automatically by AWS when:
- You log in via AWS SSO
- You assume a role via STS
- Includes:
- `AccessKeyId`
- `SecretAccessKey`
- `SessionToken`
- Expiration timestamp (e.g., 1 hour)
- Stored locally in: `~/.aws/cli/cache/`
- Automatically expire and require re-login to refresh

### ✅ Benefits:
- Short-lived → reduces risk of misuse
- Auto-rotated → no need to manage key rotation manually
- Secure for human access and automation using IAM roles

---

## 🔁 Difference Between Username/Password and Temporary Credentials

| Item                        | Purpose                            | Used For                  | Validity         |
|----------------------------|------------------------------------|---------------------------|------------------|
| **Username/Password**      | To log in via web console or SSO   | Human login (GUI/browser) | Until changed    |
| **Temporary Credentials**  | Auto-generated for CLI/API access  | Machine access (CLI/SDK)  | Short (1–12 hrs) |

- ✅ Your **username/password stays the same**.
- 🔁 Your **temporary credentials change every login** and **expire** automatically.

---

## 📘 Best Practices

- ✅ Use **IAM roles + STS or Identity Center** instead of static IAM users.
- ✅ Prefer **Identity Center** for managing **multi-account user access**.
- 🚫 Avoid using **long-lived IAM user keys** for human users.
- ✅ Use `aws sso login` for secure CLI/API access via temporary credentials.

---

## 🛠 Example AWS CLI Profile for Identity Center

```ini
[profile dev-admin]
sso_start_url = https://yourcompany.awsapps.com/start
sso_region = us-east-1
sso_account_id = 123456789012
sso_role_name = AdministratorAccess
region = us-east-1
output = json
```
**Use:**
```
aws sso login --profile dev-admin
aws s3 ls --profile dev-admin
   ```

## 🔚 Conclusion

* Use IAM users sparingly (only for automation with roles).

* For human users, always prefer AWS Identity Center with temporary credentials.

* This approach improves security, scalability, and operational manageability.   
