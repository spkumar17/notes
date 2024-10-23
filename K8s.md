# Kubernetes

## Service Discovery
**Kube-DNS (or CoreDNS in newer versions)** automatically assigns DNS names to Kubernetes services. When a service is created, it gets a DNS entry in the form of service-name.namespace.svc.cluster.local, allowing other services in the cluster to access it using this DNS name.

If, for example, a service named mysql is in the dev namespace, other pods can communicate with it by simply using mysql.dev.svc.cluster.local.
