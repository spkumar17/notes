# Kubernetes Node Assignment Concepts

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
