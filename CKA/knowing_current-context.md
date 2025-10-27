# 🧩 Scenario: Finding Current Context Without `kubectl`

## 🧠 Problem

You're on a Kubernetes control plane node where the `kubectl` binary is either **missing**, **corrupted**, or **not accessible due to permission restrictions**. You need to identify the **current context** being used to connect to the Kubernetes cluster.

---

## 🪜 Steps to Solve

### 1️⃣ Locate kubeconfig file

By default, `kubectl` uses:

```
~/.kube/config
```

On kubeadm-based clusters, the admin kubeconfig is often at:

```
/etc/kubernetes/admin.conf
```

---

### 2️⃣ Check the current context manually

Use simple Linux commands to read the kubeconfig file.

#### Example 1: Using `grep`

```bash
grep current-context ~/.kube/config
```

Output:

```
current-context: kubernetes-admin@kubernetes
```

#### Example 2: If using kubeadm config file

```bash
grep current-context /etc/kubernetes/admin.conf
```

---


## ✅ Verification

After restoring access to `kubectl`, confirm:

```bash
kubectl config current-context
```

Expected output:

```
dev-context
```

---

## 🧾 Summary Table

| Step | Description              | Command                               |
| ---- | ------------------------ | ------------------------------------- |
| 1    | Locate kubeconfig file   | `ls ~/.kube/config`                   |
| 2    | Find current context     | `grep current-context ~/.kube/config` |
| 3    | Edit manually (optional) | `vim ~/.kube/config`                  |
