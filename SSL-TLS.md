# 🔐 TLS Handshake Explained – Step by Step

This guide explains the **TLS Handshake** from scratch, using real-world examples and analogies to make every step easy to understand.

---

## 📌 What is TLS?

**TLS (Transport Layer Security)** is a cryptographic protocol used to secure communication over the internet (like when visiting HTTPS websites).

It provides:

- 🔐 **Encryption** – so no one can read your data
- ✅ **Integrity** – so no one can tamper with your data
- 👤 **Authentication** – so you know you're talking to the right server

---

## 🧠 Real-World Use Case

**When you visit a secure site like:**
https://www.mybank.com

Your browser and the server perform a **TLS Handshake** before any sensitive data is exchanged.

---

## 🤝 TLS Handshake Steps (with Explanation)

### 🧑‍💻 1️⃣ Client Hello

The browser (client) sends:

- Supported encryption algorithms (e.g., AES, ChaCha20)
- A 32-byte random number → **Client Random**
- TLS version it supports

**📨 This message is called: `Client Hello`**

> 👉 Think of it like:  
> “Hi! I want to talk securely. Here’s what I support, and here’s a random number to get us started.”

✅ **Note**: The Client Random is sent **unencrypted** — it's not a secret.

---

### 🏦 2️⃣ Server Hello

The server replies with:

- Chosen encryption algorithm
- A 32-byte **Server Random**
- Its **digital certificate** containing:
  - Its **public key**
  - Its domain name
  - The **signature** of a trusted Certificate Authority (CA)

**📨 This message is called: `Server Hello`**

> 👉 Think of it like:  
> “Let’s use this cipher. Here’s my certificate and a random number from me.”

---

### 🔐 3️⃣ Certificate Validation

The browser checks:

- Is the certificate **signed by a trusted CA**?
- Does the **domain match**?
- Is it **valid (not expired)**?

✅ If everything checks out, the handshake continues.  
🛑 If not, you’ll see a browser warning:  
> “⚠️ Your connection is not private.”

---

### 🔏 4️⃣ Pre-Master Secret Creation

- The client creates a new **Pre-Master Secret** (another random number).
- It **encrypts it using the server’s public key** from the certificate.
- It sends the encrypted secret to the server.

Only the server can decrypt it (using its private key).

---

### 🔓 5️⃣ Server Decrypts the Pre-Master Secret

- Server uses its **private key** to decrypt the pre-master secret.

Now both client and server have:

1. Client Random
2. Server Random
3. Pre-Master Secret

---

### 🔑 6️⃣ Session Key Derivation

Both parties run a **Key Derivation Function (KDF)** using: Session Key = KDF(Client Random + Server Random + Pre-Master Secret)


This creates a **shared symmetric key** that will encrypt all further communication.

---

### ✅ 7️⃣ Handshake Completion

Client and server send encrypted “Finished” messages using the session key to confirm the setup worked.

If each can decrypt the other's message — handshake is successful! 🎉

---

### 🔐 8️⃣ Secure Communication Starts

From now on, everything — login info, forms, API calls — is **encrypted** using the symmetric session key.

---

## 🔍 Summary Table

| Step | Description | Encrypted? |
|------|-------------|------------|
| 1️⃣ Client Hello | Client sends random value, cipher list | ❌ No |
| 2️⃣ Server Hello | Server sends random value, certificate | ❌ No |
| 3️⃣ Cert Validation | Client validates server identity | ❌ No |
| 4️⃣ Pre-Master Secret | Client encrypts secret with server's public key | ✅ Yes |
| 5️⃣ Decryption | Server decrypts the pre-master secret | ✅ Yes |
| 6️⃣ Key Derivation | Both derive a shared session key | — |
| 7️⃣ Finished | Both send encrypted “Finished” messages | ✅ Yes |
| 8️⃣ Communication | All app data is encrypted | ✅ Yes |

---

## 🧪 Debug it Yourself (Optional)

You can use tools like [Wireshark](https://www.wireshark.org/) to inspect TLS handshakes in real-time.  
Search for `Client Hello` and `Server Hello` packets to see `Client Random` and `Server Random` in plaintext.




