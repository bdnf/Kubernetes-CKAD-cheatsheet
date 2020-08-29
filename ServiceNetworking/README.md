# Services

Basic step to access a new service is to use to expose ports of previously created deployment.
```
kubectl expose deployment/nginx --port=80 --type=NodePort
```
This service used port 80 and generated a random port on all the nodes. A particular port and targetPort can also be passed during object creation to avoid random values.

`kubectl get svc` command lists all the existing services.

A bit more extended command may look like:
```
kubectl expose deployment/simple-webapp-deployment --port=8080 \
    --target-port=8080 --type=NodePort --labels name=simple-webapp --name=webapp
```

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
A newer service is ExternalName, which is a bit different. It has no selectors, nor does it define ports or endpoints. It allows the return of an alias to an external service. The redirection happens at the DNS level, not via a proxy or forward. This object can be useful for services not yet brought into the Kubernetes cluster or for connecting to a remote database. Also one can use a service without selectors to direct the service to another service, in a different namespace or cluster. ExternalName returns a CNAME record and is handy when using a resource external to the cluster.

```
spec:
  Type: ExternalName
  externalName: ext.db.example.com
```

# Ingress Resources

An ingress resource is an API object containing a list of rules matched against all incoming requests. Only HTTP rules are currently supported. In order for the controller to direct traffic to the backend, the HTTP request must match both the host and the path declared in the ingress.

Fore resources to forward traffic to other Deployment, first an ingress Controller should be created.
Great examples are Nginx-ingress-controller, HAProxy, Traefik.

Ingress controller manages ingress rules to route traffic to existing services.
Ingress can be used for load balancing, fan out to services or name-based hosting. Another useful feature may be to expose low-numbered ports with help of Ingress Controllers.


# Network Policies

Pods become isolated by having a NetworkPolicy that selects them. Once there is any NetworkPolicy in a namespace selecting a particular pod, that pod will reject any connections that are not allowed by any NetworkPolicy. (Other pods in the namespace that are not selected by any NetworkPolicy will continue to accept all traffic).

```
kubectl get networkpolicy
```
More detailed example can be found in: `network-policy.yaml`

Simple Network Policy that is applied on `redis` pods and allow access from `dev` pods only may look like the following:
```
kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  name: redis-access
spec:
  # policy is applied on these pods
  podSelector:
    matchLabels:
      app: redis
  ingress: # allow ingress traffic from pods with specific labels only
  - from:
    - podSelector: 
        matchLabels: 
          environment: dev
    # [Optional] allow access for them only on this port
    ports:
    - protocol: TCP
      port: 6379
```
