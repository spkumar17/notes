# 🧩 Kubernetes Troubleshooting Scenario — kube-apiserver Misconfiguration

This scenario helps you **practice troubleshooting a broken kube-apiserver** in a Kubernetes cluster by intentionally misconfiguring its static Pod manifest and investigating logs using different tools such as `crictl`, `journalctl`, and filesystem logs.

---

## 🎯 Objective

Learn how to:

* Identify issues when the **kube-apiserver** fails to start.
* Check multiple **log sources** for debugging.
* Use **`crictl`** to inspect and view container logs.
* Safely **recover** from a bad configuration.

---

## 🧱 Prerequisites

* A working Kubernetes **control plane node** (e.g., `controlplane` or `master`).
* Root or sudo privileges.
* Basic knowledge of YAML and Linux CLI.
* `crictl` installed (comes by default with most kubeadm-based setups).

---

## ⚙️ Step 1 — Backup the Existing kube-apiserver Manifest

Always make a backup before making any changes:

```bash
sudo cp /etc/kubernetes/manifests/kube-apiserver.yaml ~/kube-apiserver.yaml.ori
```

---

## ⚡ Step 2 — Introduce a Misconfiguration

Edit the manifest and add an invalid flag:

```bash
sudo vim /etc/kubernetes/manifests/kube-apiserver.yaml
```

Add the following under `command:`:

```yaml
- --this-is-very-wrong
```

Once you save and exit, `kubelet` will automatically detect the change and try to restart the kube-apiserver container.

---

## 🔍 Step 3 — Observe the Behavior

Watch for the static Pod restarting:

```bash
watch sudo crictl ps
```

Or check directly via Kubernetes (if `kubectl` is still working):

```bash
kubectl -n kube-system get pods
```

You’ll notice that **kube-apiserver is not coming back up** — this is expected.

---

## 🗾 Step 4 — Check Logs from Different Sources

When the apiserver fails to start, check multiple log locations:

### 📁 1. Filesystem Logs

```bash
ls /var/log/pods/kube-system_kube-apiserver*
cat /var/log/pods/kube-system_kube-apiserver*/kube-apiserver/*.log
```

You should see something like:

```
Error: unknown flag: --this-is-very-wrong
```

### 🧰 2. Using `crictl`

List containers:

```bash
sudo crictl ps -a
```

Get logs for the apiserver container:

```bash
sudo crictl logs <container-id>
```

### 🐳 3. Using Docker (if Docker runtime is used)

```bash
sudo docker ps -a
sudo docker logs <container-id>
```

### 🧩 4. Kubelet Logs

Kubelet manages static pods, so check its logs for related errors:

```bash
sudo journalctl -u kubelet -f
```

or

```bash
sudo cat /var/log/syslog | grep kubelet
```

---

## 🔁 Step 5 — Fix the Misconfiguration

Restore your working backup:

```bash
sudo cp ~/kube-apiserver.yaml.ori /etc/kubernetes/manifests/kube-apiserver.yaml
```

Wait for the kubelet to detect and restart the healthy container:

```bash
watch sudo crictl ps
```

Once it’s back up, confirm it’s running:

```bash
kubectl -n kube-system get pods
```

---

## 🧠 Understanding `crictl`

`crictl` is a **CLI tool for CRI-compatible container runtimes** such as containerd or CRI-O.
It helps to inspect, manage, and debug containers directly on Kubernetes nodes.

Common commands:

| Command                        | Description                        |
| ------------------------------ | ---------------------------------- |
| `crictl ps`                    | List running containers            |
| `crictl ps -a`                 | List all containers                |
| `crictl images`                | Show images                        |
| `crictl pods`                  | List pods                          |
| `crictl logs <id>`             | View container logs                |
| `crictl inspect <id>`          | Inspect container details          |
| `crictl exec -it <id> /bin/sh` | Execute a shell inside a container |

---

## 🧩 Summary

| Task                           | Description                                                  |
| ------------------------------ | ------------------------------------------------------------ |
| Modify kube-apiserver manifest | Introduced a wrong flag                                      |
| Verify Pod behavior            | kube-apiserver fails to start                                |
| Debug logs                     | Checked logs via `/var/log/pods`, `crictl`, and `journalctl` |
| Recover                        | Restored the original manifest                               |

---

## ✅ Expected Outcome

After reverting the change:

* The kube-apiserver Pod should restart successfully.
* `kubectl` commands should start responding again.
* You gain confidence in debugging control plane failures using various tools.

---

## 🏁 Bonus Practice

Try similar exercises with other static Pods:

* `kube-controller-manager`
* `kube-scheduler`
* `etcd`

This will strengthen your real-world troubleshooting skills in cluster administration and certification exams like **CKA**.



## 📂 Understanding Kubernetes Log Directories

When troubleshooting Pods or components such as the **kube-apiserver**, it’s essential to understand where Kubernetes stores log files on the node. Two common log locations are `/var/log/pods` and `/var/log/containers`.

---

## 📁 `/var/log/pods`

**Purpose:**

* Contains the **actual raw log files** for containers managed by kubelet.
* Logs are organized by **namespace**, **pod name**, and **pod UID**.

**Structure:**

```
/var/log/pods/<namespace>_<pod-name>_<pod-uid>/<container-name>/
```

**Example:**

```
/var/log/pods/kube-system_kube-apiserver-controlplane_abc123/kube-apiserver/0.log
```

**Details:**

* Stores real log data written by the container runtime.
* Files are numbered (`0.log`, `1.log`, etc.) representing log rotations.
* Useful for debugging when `kubectl logs` does not work (e.g., API server is down).

---

## 📦 `/var/log/containers`

**Purpose:**

* Contains **symlinks** (shortcuts) that point to the real log files in `/var/log/pods`.
* Provides a **flat structure** for easy access by log collectors and CLI tools.

**Structure:**

```
/var/log/containers/<pod-name>_<namespace>_<container-name>-<container-id>.log
```

**Example:**

```
/var/log/containers/kube-apiserver-controlplane_kube-system_kube-apiserver-88c59e6d2870e.log
```

**Details:**

* Each symlink connects to the corresponding file under `/var/log/pods`.
* Tools like Fluentd, Promtail, and Filebeat commonly read from this directory.
* Easier to index and search across multiple pods.

---

## 🧰 Quick Comparison

| Feature          | `/var/log/pods`             | `/var/log/containers`                       |
| ---------------- | --------------------------- | ------------------------------------------- |
| **Type**         | Real log files              | Symlinks to log files                       |
| **Organized by** | Namespace / Pod / Container | Flat structure (Pod-Namespace-Container-ID) |
| **Used by**      | Admins, kubelet             | Log collectors, CLI tools                   |
| **Example Path** | `/var/log/pods/.../0.log`   | `/var/log/containers/...log`                |

---

## 🕵️‍♂️ Tip

To verify the link between these two directories:

```bash
ls -l /var/log/containers/
```

You’ll see output showing symbolic links pointing back to files in `/var/log/pods`.

Example:

```
lrwxrwxrwx 1 root root 150 Oct 27 08:32 kube-apiserver-controlplane_kube-system_kube-apiserver-abc123.log -> /var/log/pods/kube-system_kube-apiserver-controlplane_abc123/kube-apiserver/0.log
```

---

### 📚 Summary

* `/var/log/pods`: Contains the **real logs** generated by containers.
* `/var/log/containers`: Contains **symlinks** for convenience and log collection.
* Both are important for **troubleshooting** static Pods and system components on a Kubernetes node.

