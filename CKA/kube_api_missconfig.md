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
