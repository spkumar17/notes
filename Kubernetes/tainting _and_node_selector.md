# Kubernetes Node Assignment Concepts

## Manual Scheduling in Kubernetes

## Overview
Manual scheduling allows direct pod-to-node assignment by bypassing the Kubernetes scheduler.

## Basic Usage
Specify the target node using `nodeName`:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  nodeName: node-1    # Direct node assignment
  containers:
  - name: nginx
    image: nginx
```

## Important Considerations

### Advantages
- Simple and direct
- Useful for debugging
- Good for testing node-specific issues

### Limitations
- No automatic rescheduling if node fails
- Bypasses scheduler's load balancing
- Cannot be changed after pod creation
- Not recommended for production

### When to Use
- Testing and development
- Debugging node-specific issues
- Special cases requiring fixed node placement

### When to Avoid
- Production deployments
- High-availability requirements
- With ReplicaSets/Deployments
## Node Selectors
Node selectors are the simplest way to constrain which nodes your pod can be scheduled on. You add the `nodeSelector` field to your pod specification and specify key-value pairs. For the pod to be eligible to run on a node, the node must have each of the indicated key-value pairs as labels.

Label a node:
```bash
kubectl label nodes <node-name> size=large disk=ssd
```

Use the labels in pod spec:
```yaml
nodeSelector:
  size: large
  disk: ssd
```

## Taints and Tolerations
Taints and tolerations work together to ensure that pods are not scheduled onto inappropriate nodes.

### Taints
- Taints are applied to nodes
- They allow a node to repel certain pods
- Consist of a key, value, and effect
- Common effects:
  - `NoSchedule`: Pods won't be scheduled on the node(existing pods will remain same)
  - `PreferNoSchedule`: System will try to avoid scheduling pods on the node ( not sure it might schedule as well)
  - `NoExecute`: New pods won't be scheduled AND existing pods will be evicted

Example of adding a taint:
```bash
kubectl taint nodes node1 key=value:NoSchedule
```

### Tolerations
- Tolerations are applied to pods
- Allow pods to schedule onto nodes with matching taints
- Must match the taint's key, value, and effect

Example of a toleration in pod spec:
```yaml
tolerations:
- key: "key"
  operator: "Equal"
  value: "value"
  effect: "NoSchedule"
```

## Common Use Cases

1. **Dedicated Nodes**
   - Taint nodes for specific workloads
   - Only pods with matching tolerations can run there

2. **Special Hardware**
   - Use node selectors for pods requiring specific hardware
   - Example: GPU-intensive workloads

3. **Node Maintenance**
   - Taint nodes before maintenance
   - Helps safely drain workloads

## Best Practices

1. Use node selectors for simple node selection requirements
2. Use taints and tolerations for more complex scheduling requirements
3. Document your taints and node labels
4. Regularly review and clean up unused taints and labels
5. Consider using node affinity for more complex node selection scenarios


## Feature Comparison

| Feature | Enforces Exclusivity | Flexible Scheduling | Use Case |
|---------|---------------------|---------------------|-----------|
| Taints & Tolerations | No | Yes | Use to repel Pods from Nodes unless they tolerate specific conditions, like workloads requiring isolation. |
| Node Selector | Yes | No | Use to strictly schedule Pods on specific Nodes (e.g., Nodes with special hardware). |

## Node Affinity Notes

### What is Node Affinity?
Node Affinity is a set of rules used by the Kubernetes scheduler to determine where to place pods. It provides more expressive and flexible ways to define which nodes your pods should run on compared to node selectors.

### Types of Node Affinity
1. **Required Node Affinity (requiredDuringSchedulingIgnoredDuringExecution)**
   - Hard requirement that must be met for pod scheduling
   - Pod will not be scheduled if no node matches the criteria
   - Similar to nodeSelector but with more expressive syntax

2. **Preferred Node Affinity (preferredDuringSchedulingIgnoredDuringExecution)**
   - Soft requirement (preference)
   - Scheduler will try to find matching nodes but will still schedule the pod even if no matches are found
   - Can specify weight (1-100) to prioritize certain rules

### Common Operators
- `In`: Label value must match one of the specified values
- `NotIn`: Label value must not match any of the specified values
- `Exists`: Label must exist (value doesn't matter)
- `DoesNotExist`: Label must not exist
- `Gt`: Greater than
- `Lt`: Less than

### Basic Example
```yaml
affinity:
  nodeAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      nodeSelectorTerms:
      - matchExpressions:
        - key: kubernetes.io/e2e-az-name
          operator: In
          values:
          - e2e-az1
          - e2e-az2
    preferredDuringSchedulingIgnoredDuringExecution:
    - weight: 1
      preference:
        matchExpressions:
        - key: disk-type
          operator: In
          values:
          - ssd
```

### Best Practices
1. Use required affinity for critical placement requirements
2. Use preferred affinity for optimization and better resource utilization
3. Combine multiple rules to create sophisticated scheduling strategies
4. Consider using weights to balance different preferences
5. Keep rules simple and maintainable

# Node Selection in Kubernetes: Node Selector vs Node Affinity

| Feature | Node Selector | Node Affinity |
|---------|--------------|---------------|
| Definition | Simple mechanism for node selection | Advanced, flexible scheduling rules |
| Syntax | Simple key-value pairs | Complex expressions with operators |
| Soft Preferences | Not supported | Supported (preferredDuringScheduling) |
| Hard Constraints | Only hard constraints | Supported (requiredDuringScheduling) |
| Complex Rules | No (only exact matches) | Yes (supports In, NotIn, Exists, etc.) |
| Weighting Nodes | Not supported | Supported (weight for preferences) |
| Primary Use Case | Simple, strict node targeting | Advanced and conditional node targeting |
| Example YAML Size | Small and minimal | Larger, with more configuration options |

## Node Affinity vs Taints and Tolerations

| Aspect | Node Affinity | Taints and Tolerations |
|--------|---------------|------------------------|
| Who Defines the Rules? | Rules are defined in the pod specification (labels on nodes) | Rules are defined both on nodes (taints) and pods (tolerations) |
| Mandatory vs Optional | Affinity can be mandatory (required) or optional (preferred) | Taints strictly repel unless tolerations are specified |
| Primary Goal | Scheduling pods based on node labels | Preventing pods from running on certain nodes |
| Conflict Handling | No; pods simply fail to schedule if required affinity is not met | Pods without tolerations are blocked by taints |
