# ConfigMaps and Secrets

`Secrets` and `ConfigMaps`. Encoded data can be passed using a `Secret` and non-encoded data can be passed with a `ConfigMap`. These can be used to pass important data like SSH keys, passwords, or even a configuration file like.

## Create Secrets from literal
```
kubectl create secret generic db-secret --from-literal=DB_Host=sql01 --from-literal=DB_User=root --from-literal=DB_Password=password123
```

**Create Service Account**
```
kubectl create serviceaccount api-access
```

In Deployment or Pod definition add in Pod spec (before `spec.containers[]`):
```
  serviceAccountName: api-access
```

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
