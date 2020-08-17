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

kubectl label node worker color=red
```
