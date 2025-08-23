# EKS Cluster Update

## **Why you should test before upgrading Prodution Cluster**

* Every new Kubernetes version may bring changes in APIs, behavior, or features.

* Some older APIs get deprecated or removed.

* If you upgrade directly in production, your apps may break unexpectedly.

* That’s why AWS recommends you set up a CI/CD workflow where you test your app on a newer Kubernetes version before updating your production cluster.

## **What happens during an EKS cluster upgrade**

**Amazon EKS launches new control plane (API server) nodes with the updated version**

* The control plane is the “brain” of your Kubernetes cluster.

* These new servers are added alongside the old ones first.

## **EKS runs health checks**

* Checks if the new API server nodes are handling traffic properly.

* Verifies that the networking and cluster communication still work.

**If everything is fine**
* Old control plane nodes are replaced.

* Your cluster now runs on the new Kubernetes version.

**If something goes wrong**

* EKS stops the upgrade and rolls back to the old version.

Your apps keep running on the old control plane.

The cluster never gets stuck in a broken state.

## **What about your workloads (apps)?**

* The upgrade process only affects the control plane (API server), not your running apps.

* Your apps in worker nodes (Pods, Deployments, Services, etc.) keep running.

* But: if your app uses APIs that are deprecated or changed in the new version, you might see issues after the upgrade.

**What are APIs referred above?**

When you write YAML files (like Deployment, Ingress, StatefulSet), you’re using Kubernetes APIs.
Example:
```
apiVersion: apps/v1
kind: Deployment
```

**What does "deprecated or changed" mean?**

Kubernetes evolves over time:

Deprecated = The API still works now, but will be removed in a future version.

Removed/Changed = That API no longer works in the newer version (you must use the new one).

**Example**:

In Kubernetes v1.15, people used:
```
apiVersion: extensions/v1beta1
kind: Deployment

```
From Kubernetes v1.16 onward, this API was removed. You must use:

```
apiVersion: apps/v1
kind: Deployment
```

If you upgrade to a version where your old API is gone, your manifests may fail to apply or your app may stop working if it relies on that API.
If your app’s YAML, Helm charts, or operators still use old APIs, upgrading the cluster could cause:
* **kubectl apply failing with errors like:**
```
error: unable to recognize "deployment.yaml": no matches for kind "Deployment" in version "extensions/v1beta1"

```
* Controllers (like Ingress controllers) not working because their CRDs changed.

* Monitoring/logging agents breaking if their APIs changed.

**That’s why AWS tells you:**

👉 Before upgrading your production cluster, test your app on the new Kubernetes version (e.g., in a staging or CI/CD pipeline) to make sure your manifests and APIs are compatible.

**1. Why EKS needs extra IPs during an upgrade**

* When you upgrade the control plane (API server) in EKS, AWS doesn’t overwrite the existing servers immediately.

* Instead, it creates new control plane nodes (with the new Kubernetes version) alongside the old ones → this requires new network interfaces (ENIs) in your VPC.

* For those ENIs, up to 5 free IP addresses are needed in your subnets.

So if your subnets are already “full” (no free IPs left), the upgrade can’t proceed.

**Where EKS places these ENIs**

* When you created the cluster, you gave EKS a list of subnets.

* EKS can create these control plane ENIs in any of those subnets.

* It doesn’t necessarily reuse the same subnet(s) where the old control plane ENIs are.

**3. Why security groups matter**

The ENIs for the control plane must be able to communicate with worker nodes (and vice versa).

If the subnet chosen for the new ENIs has a security group that blocks traffic, then worker nodes may not be able to talk to the new control plane → the upgrade fails.

Example:

* Suppose your worker nodes are in subnet A.

* EKS creates new ENIs in subnet B.

* If your security group rules don’t allow communication between A and B, the cluster control plane won’t come online.

**Before upgrade:**
Your control plane runs on existing ENIs in the subnets you chose.

**During upgrade:**

* Amazon EKS creates new control plane ENIs with the upgraded Kubernetes version.

* These new ENIs may be in different subnets than the old ones.

* The old ENIs stay active while the new ones are being tested.

**After upgrade succeeds:**

* Once the new control plane is healthy and passes all readiness checks, EKS deletes the old ENIs.

* This ensures there’s no downtime — your control plane is always available.

**If upgrade fails:**

* New ENIs are removed.

* Old ENIs continue serving the cluster, so rollback is safe.

| Purpose                             | Number of ENIs/IPs | Notes                                                                                           |
| ----------------------------------- | ------------------ | ----------------------------------------------------------------------------------------------- |
| **Testing new control plane nodes** | 1+                 | EKS spins up new API server nodes in one or more subnets to validate they work.                 |
| **Rollback safety**                 | 1+                 | If the new nodes fail, EKS can revert traffic to the old control plane ENIs.                    |
| **High availability / multi-AZ**    | 2–3                | Ensures the control plane remains available across multiple Availability Zones while upgrading. |


**4. What can make the upgrade fail**

* 🚫 One of the subnets you told EKS about was deleted later.

* 🚫 Subnet(s) don’t have at least 5 free IP addresses available.

* 🚫 Security group rules don’t allow cluster communication between control plane ENIs and worker nodes.

If any of these conditions fail, the upgrade cannot finish, and EKS will roll back to the old version.

**What this is saying**

**Highly available control plane**

* Amazon EKS runs your API server (control plane) in a highly available way, usually across multiple Availability Zones (AZs).

* This ensures that if one instance or AZ goes down, the API server remains reachable.

**Rolling updates during upgrade**

* When EKS upgrades your cluster, it updates API server instances one at a time (rolling update).

* This avoids downtime — at least one API server is always serving requests.

**Changing IP addresses**

* Each API server instance may get a different IP address after a rolling update.

* So, clients (like kubectl or applications using Kubernetes client libraries) should handle reconnects automatically.

**Client-side reconnect**

* Modern kubectl versions and official Kubernetes client libraries already handle this transparently .

* That means if an API server instance moves or its IP changes, your commands or app requests automatically retry and reconnect to the new API server.

**What “EKS Auto Mode” means**

It’s essentially a set of EKS managed automation features, where AWS automatically manages node upgrades, scaling, and certain capabilities for you.

**Control Plane vs Nodes**

* When you upgrade the control plane (API server), you normally have to upgrade worker nodes manually.

* In Auto Mode, EKS automatically starts incrementally upgrading managed nodes to match the new Kubernetes version of the control plane.

**Respects Pod Disruption Budgets (PDBs)**

* Even though EKS is upgrading nodes automatically, it respects your PDBs, so workloads are not disrupted beyond the limits you defined.

**Managed Capabilities** 
You don’t have to manually configure or upgrade:

* Compute autoscaling → nodes scale automatically based on demand.

* Block storage (EBS) → storage capabilities are handled automatically.

* Load balancing (ELB/ALB) → traffic routing remains consistent during upgrades or scaling.

**📌 Key Benefits of Auto Mode**

* Less operational overhead → no manual node upgrades.

* Zero-downtime upgrades → respects PDBs.

* Automatic scaling and resource management → compute, storage, and load balancers handled for you.

**What is a Pod Disruption Budget (PDB)?**

A Pod Disruption Budget is a Kubernetes policy that limits how many pods of a deployment, statefulset, or other workload can be unavailable at the same time.

It ensures that during voluntary disruptions (like node upgrades, cluster scaling, or maintenance), your application maintains a minimum level of availability.

* Voluntary disruptions = things you control, e.g., draining a node, updating a deployment.

* Involuntary disruptions = things outside your control, e.g., node crashes — PDBs don’t prevent these.

**The high-level summary of the Amazon EKS cluster upgrade process is as follows:**

1) Ensure your cluster is in a state that will support an upgrade. This includes checking the Kubernetes APIs used by resources deployed into the cluster, ensuring the cluster is free of any health issues. You should use Amazon EKS upgrade insights when evaluating your cluster’s upgrade readiness.

2) Upgrade the control plane to the next minor version (for example, from 1.32 to 1.33).

3) Upgrade the nodes in the data plane to match that of the control plane.

4) Upgrade any additional applications that run on the cluster (for example, cluster-autoscaler).

5) Upgrade the add-ons provided by Amazon EKS, such as those included by default:

* Amazon VPC CNI recommended version

* CoreDNS recommended version

* kube-proxy recommended version

6) Upgrade any clients that communicate with the cluster (for example, kubectl).


**Step 1: Prepare for upgrade**

Compare the Kubernetes version of your cluster control plane to the Kubernetes version of your nodes.

Get the Kubernetes version of your cluster control plane.

```
kubectl version
```
Get the Kubernetes version of your nodes. This command returns all self-managed and managed Amazon EC2, Fargate, and hybrid nodes. Each Fargate Pod is listed as its own node.
```
kubectl get nodes
```
Before updating your control plane to a new Kubernetes version, make sure that the Kubernetes minor version of both the managed nodes and Fargate nodes in your cluster are the same as your control plane’s version. For example, if your control plane is running version 1.29 and one of your nodes is running version 1.28, then you must update your nodes to version 1.29 before updating your control plane to 1.30. We also recommend that you update your self-managed nodes and hybrid nodes to the same version as your control plane before updating the control plane. 
Fargate nodes can’t be upgraded in-place.Delete the pod → it gets recreated on a new node with the correct version.Then upgrade the control plane.After redeploying remaining pods, all pods run on nodes compatible with the new control plane.


**Amazon EKS Cluster Insights** automatically scans your cluster for potential issues that could impact Kubernetes version upgrades, such as deprecated APIs or incompatible configurations. The list of checks is periodically updated to reflect changes in Kubernetes and EKS. This helps you identify and fix problems before upgrading, ensuring a smooth and safe cluster upgrade.

**Detailed considerations**
Assume that your current cluster version is version 1.28 and you want to update it to version 1.30. You must first update your version 1.28 cluster to version 1.29 and then update your version 1.29 cluster to version 1.30.

**Kubernetes Version Skew**
kube-apiserver = the control plane component that manages the cluster.

kubelet = the agent running on each node, responsible for starting/stopping pods and reporting node/pod status to the control plane.

Version skew = the difference between the Kubernetes version of the control plane (kube-apiserver) and the node agents (kubelets).

## **Starting from Kubernetes 1.28**
The kubelet can safely be up to three minor versions older than the kube-apiserver.

**Example:**
kube-apiserver = 1.28

kubelet = 1.25 → compatible

Kubelet version = 1.25

* You can safely upgrade your control plane from 1.25 → 1.26 → 1.27 → 1.28

* Kubelets remain on 1.25 during this time

* This allows control plane upgrades without immediate node upgrades, reducing operational effort and risk.

If the kube-apiserver is on version 1.28, the nodes (kubelets) can be on 1.27, 1.26, or 1.25, since nodes are supported to be up to 3 minor versions older than the control plane, but never newer.
**Best practice is to keep the kubelet (node version) and the kube-apiserver (control plane version) on the same version to avoid compatibility issues and unexpected behavior, even though Kubernetes allows a minor version skew (control plane can be newer).**

**out of Subject**--->Before installing Kubernetes, you need to set up a CNI plugin to enable pod-to-pod networking inside the cluster.

**Update cluster - Using eksctl**

This procedure requires eksctl version 0.212.0 or later. You can check your version with the following command:

```
eksctl version
```
Update the Kubernetes version of your Amazon EKS control plane. Replace <cluster-name> with your cluster name. Replace <version-number> with the Amazon EKS supported version number that you want to update your cluster to
```
eksctl upgrade cluster --name <cluster-name> --version <version-number> --approve
```

## **Update cluster components**

After your cluster update is complete, update your nodes to the same Kubernetes minor version as your updated cluster
