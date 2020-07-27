# CLI commands

```
# Create Service from CLI

kubectl expose deployment/simple-webapp-deployment --port=8080 \
    --target-port=8080 --type=NodePort --labels name=simple-webapp --name=webapp
```

# Network Policies
```
kubectl get networkpolicy
```

```yaml
# allow incoming to payroll from internal on port 8080
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: external-policy
spec:
  ingress:
  - from:
    - podSelector:
        matchLabels:
          name: internal
    ports:
    - port: 8080
      protocol: TCP
  podSelector:
    matchLabels:
      name: payroll
  policyTypes:
  - Ingress
---
# Allow outgoing from internal to payroll and mysql only
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: internal-policy
spec:
  egress:
  - to:
    - podSelector:
        matchLabels:
          name: payroll
    ports:
    - port: 8080
      protocol: TCP
  - to:
    - podSelector:
        matchLabels:
         name: mysql
    ports:
    - port: 3306
  podSelector:
    matchLabels:
      name: internal
  policyTypes:
  - Egress
```
