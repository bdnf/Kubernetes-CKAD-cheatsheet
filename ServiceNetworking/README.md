# Service Types

## ClusterIP
The ClusterIP service type is the default, and only provides access internally (except if manually creating an external endpoint). The range of ClusterIP used is defined via an API server startup option.

The kubectl proxy command creates a local service to access a ClusterIP. This can be useful for troubleshooting or development work.

```
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: MyApp
  ports:
    - protocol: TCP
      port: 80
      targetPort: 9376
```

## Services without selectors
Services most commonly abstract access to Kubernetes Pods, but they can also abstract other kinds of backends. For example:

You want to have an external database cluster in production, but in your test environment you use your own databases.
You want to point your Service to a Service in a different Namespace or on another cluster.
You are migrating a workload to Kubernetes. Whilst evaluating the approach, you run only a proportion of your backends in Kubernetes.

```
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  ports:
    - protocol: TCP
      port: 80
      targetPort: 9376
```
Next, map the Service to the network address and port where it's running, by adding an Endpoint object manually:
```
apiVersion: v1
kind: Endpoints
metadata:
  name: my-service
subsets:
  - addresses:
      - ip: 192.0.2.42
    ports:
      - port: 9376
```

## NodePort
The NodePort type is great for debugging, or when a static IP address is necessary, such as opening a particular address through a firewall. The NodePort range is defined in the cluster configuration.

```
spec:
  type: NodePort
  selector:
    app: MyApp
  ports:
      # By default and for convenience, the `targetPort` is set to the same value as the `port` field.
    - port: 80
      targetPort: 80
      # Optional field
      # By default and for convenience, the Kubernetes control plane will allocate a port from a range (default: 30000-32767)
      nodePort: 30007
```
## LoadBalancer
The LoadBalancer service was created to pass requests to a cloud provider like GKE or AWS. Private cloud solutions also may implement this service type if there is a cloud provider plugin, such as with CloudStack and OpenStack. Even without a cloud provider, the address is made available to public traffic, and packets are spread among the Pods in the deployment automatically.

Creating a LoadBalancer service generates a NodePort, which then creates a ClusterIP. It also sends an asynchronous call to an external load balancer, typically supplied by a cloud provider. The External-IP value will remain in a <Pending> state until the load balancer returns. Should it not return, the NodePort created acts as it would otherwise.

```
spec:
  clusterIP: 10.0.171.239
  type: LoadBalancer
  selector:
    app: MyApp
  ports:
    - protocol: TCP
      port: 80
      targetPort: 9376
```

## ExternalName
A newer service is ExternalName, which is a bit different. It has no selectors, nor does it define ports or endpoints. It allows the return of an alias to an external service. The redirection happens at the DNS level, not via a proxy or forward. This object can be useful for services not yet brought into the Kubernetes cluster. A simple change of the type in the future would redirect traffic to the internal objects.

The use of an ExternalName service, which is a special type of service without selectors, is to point to an external DNS server. Use of the service returns a CNAME record. Working with the ExternalName service is handy when using a resource external to the cluster, perhaps prior to full integration.
```
spec:
  Type: ExternalName
  externalName: ext.db.example.com
```

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
