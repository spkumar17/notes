# 🔐 Understanding JWT (JSON Web Token)

This provides a simple explanation of how JWT (JSON Web Token) works — especially how **JWT signatures** are used to ensure data integrity and authentication.

---

## 📦 What is a JWT?

A JWT is a compact, URL-safe token used to securely transmit information between parties.

It consists of **three parts**:

1. **Header** – contains metadata (e.g., algorithm used)
2. **Payload** – contains claims/data (e.g., user ID, role)
3. **Signature** – used to verify the token wasn't tampered with

A JWT looks like this:
```
<base64url(header)>.<base64url(payload)>.<signature>

```

---

## 🔍 Example Structure

```json

Header

{
  "alg": "HS256",
  "typ": "JWT"
}

Payload

{
  "user_id": 123,
  "role": "admin"
}

Signature

HMACSHA256(
  base64urlEncode(header) + "." + base64urlEncode(payload),
  secret
)
```


## How JWT Authentication Works

* ✅ User logs in with valid credentials.

* ✅ Server generates a JWT using a secret key and sends it to the user.

* ✅ User stores the token (e.g., in localStorage or a cookie).

* ✅ User sends the token in the Authorization header on future requests.

* ✅ Server does NOT trust the incoming token blindly:

* It recomputes the signature using the header + payload and its own secret key.

* It compares the computed signature with the one in the token.

* If they match, the token is valid.

* If they don't, the token is rejected (because it was likely tampered with).

## 🧪 What Happens If the Token is Modified?

If a malicious user changes the payload (e.g., changing their role from user to admin) and tries to send the token back:

* The signature will no longer match.

* The server will detect that the token was tampered with and reject it.

This is what makes JWTs secure and stateless.


##  Signature validation flow:

* Recompute the signature using the header and payload I sent and its secret key.

* Compare this newly computed signature with the one I sent in the JWT.

* If the signatures match, the server knows the token wasn't tampered with — and the request is authenticated.

* If the signatures don't match, the server rejects the token because it has been modified and can’t be trusted.

## 🔐 Real-World Scenario with Tampering Attempt

Imagine a user logs into a web application and receives a JWT token that includes their user ID and role in the payload. This token is signed by the server using a secret key and sent to the user's browser. Later, a malicious user (or attacker) tries to tamper with the token by changing their role from "user" to "admin" in the payload. They re-encode the token and send it with the original signature, hoping to gain unauthorized access. However, the server never trusts the signature blindly. It recomputes the signature using the modified header and payload along with the secret key it knows. Since the attacker does not have the secret, the newly computed signature does not match the one sent with the token. As a result, the server detects the tampering and rejects the request, denying access. This is how JWT protects against tampering, ensuring that any modification to the token can be immediately detected and blocked.
