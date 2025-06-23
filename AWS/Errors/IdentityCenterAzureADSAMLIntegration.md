# AWS IAM Identity Center + Azure AD SSO: Login Error Fix

## Issue

After integrating Azure AD with AWS IAM Identity Center, login failed with:

```
Something went wrong
Looks like this code isn't right.
```

## Root Cause

Azure AD was sending `user.userprincipalname` as the **NameID**, but AWS expected `user.mail`.

## Fix

In Azure AD → Enterprise Applications → AWS App → Single sign-on:

- Set **Unique User Identifier (Name ID)** to:
  ```
  user.mail
  ```

- Set **NameID format** to:
  ```
  EmailAddress
  ```

- Ensure `user.mail` is populated and matches the email in AWS IAM Identity Center.

## Result
https://projectk.awsapps.com/start
Login succeeded after correction. ✅

---

## Quick Troubleshooting

| Error | Fix |
|-------|-----|
| "Code isn't right" | Check NameID mapping |
| "Access Denied" | Verify user exists in AWS |
| "Invalid SAML" | Update metadata URLs |

## Best Practice

Use `user.userprincipalname` if `user.mail` is empty - it's always populated in Azure AD.