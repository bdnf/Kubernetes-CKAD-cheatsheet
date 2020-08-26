## Security Context

To specify security settings for a Pod, include the securityContext field in the Pod specification.
The security settings that you specify for a Pod apply to all Containers in the Pod.

With Linux capabilities, you can grant certain privileges to a process without granting all the privileges of the root user. To add or remove Linux capabilities for a Container, include the capabilities field in the securityContext section of the Container manifest.

Example context for container in a pod:
```
  containers:
  - name: container1
    ...
    securityContext:
      runAsUser: 1000
      runAsGroup: 3000
      capabilities:
        add: ["NET_ADMIN", "SYS_TIME"]
```
The security settings that you specify for this Pod will apply to only `container1`.

In the configuration file, the runAsUser field specifies that for any Containers in the Pod, all processes run with user ID 1000. The runAsGroup field specifies the primary group ID of 3000 for all processes within any containers of the Pod.

## Taints and Tolerations

```
kubectl get nodes --show-labels

kubectl describe node worker # and check Taints

# Add taints
kubectl taint nodes worker role=core-worker:NoSchedule

# Remove taint from master to allow scheduling
kubectl taint nodes master node-role.kubernetes.io/master:NoSchedule-

# add labels to the node
kubectl label node worker type=data-processor
```

Taints can be handled by adding tolerations to each pod. Taints are triple of values: `key`=`value`:`effect`.
Effect can be of type `NoSchedule`, `NoExecute`, `PreferNoSchedule`
```
spec:
  containers:
  - name: nginx
    image: nginx
  tolerations:
  - key: "key"
    operator: "Equal"
    value: "value"
    effect: "NoSchedule"
```
`NoSchedule` effect means that no pod will be able to schedule onto the node unless it has a matching toleration.

If effect `NoExecute` is added to a node, then any pods that do not tolerate the taint will be evicted immediately, and pods that do tolerate the taint will never be evicted. However, a toleration with NoExecute effect can specify an optional tolerationSeconds field that dictates how long the pod will stay bound to the node after the taint is added.

`PreferNoSchedule` - is a "soft" version of `NoSchedule`. The system will try to avoid placing a pod that does not tolerate the taint on the node, but it is not required. 

Node labels can be used to assign pods directly to specific node (assuming that notolerations are present).
```
spec:
  containers:
  - name: nginx
    image: nginx
  nodeSelector:
    type: data-processor
```
Or node labels and tainst can be combined to add more fine-grained scheduling control.
