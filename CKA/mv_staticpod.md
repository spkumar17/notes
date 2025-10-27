# 🧩 Moving a Static Pod to Another Node in Kubernetes

## Overview
Static Pods are managed directly by the **kubelet** on each node and defined as YAML manifests in `/etc/kubernetes/manifests/`.
If you want to move a Static Pod from one node to another, follow the steps below.

---

## 🪜 Steps to Move a Static Pod

### 1️⃣ Identify the Manifest File
Static Pod manifests are stored in:

```bash
/etc/kubernetes/manifests/
```
Example:
```bash
ls /etc/kubernetes/manifests/
```
Suppose the file you want to move is:
```
resource-pod.yaml
```

---

### 2️⃣ Copy the Manifest from the Source Node
If you are on the **control-plane node** and want to copy from **node01**:
```bash
scp node01:/etc/kubernetes/manifests/resource-pod.yaml .
```
This copies the manifest from node01 to your **current directory** (`.`).

---

### 3️⃣ Move the File to the Destination Node
If the destination node is the **control-plane node**:
```bash
sudo mv resource-pod.yaml /etc/kubernetes/manifests/
```

If the destination node is another worker node (e.g., `node02`):
```bash
scp resource-pod.yaml node02:/etc/kubernetes/manifests/
```
Once the file is placed in the `/etc/kubernetes/manifests/` directory on the new node,
the **kubelet** on that node will automatically start the Pod.

---

### 4️⃣ Delete the Static Pod from the Old Node
SSH into the source node (where the Pod was originally running):
```bash
ssh node01
```
Then delete the manifest file:
```bash
sudo rm /etc/kubernetes/manifests/resource-pod.yaml
```
The kubelet will detect the file removal and terminate the Pod automatically.

---

### 5️⃣ Verify the Pod Location
Check where the Pod is now running:
```bash
kubectl get pods -A -o wide
```
You should see the `resource-pod` running on the **new node**.

---

## ✅ Summary

| Step | Description | Command |
|------|--------------|----------|
| 1 | Find manifest file | `ls /etc/kubernetes/manifests/` |
| 2 | Copy from source node | `scp node01:/etc/kubernetes/manifests/resource-pod.yaml .` |
| 3 | Move to destination node | `scp resource-pod.yaml node02:/etc/kubernetes/manifests/` |
| 4 | Delete from old node | `ssh node01 && rm /etc/kubernetes/manifests/resource-pod.yaml` |
| 5 | Verify | `kubectl get pods -A -o wide` |
