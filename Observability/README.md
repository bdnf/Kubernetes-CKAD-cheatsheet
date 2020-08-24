# Multi-Container Pods
By adding a second container, each container can still be optimized and developed independently, and both can scale and be repurposed to best meet the needs of the workload. there are main patterns actively used in kubernetes environment.

## Sidecar pattern

An extra container in your pod to enhance or extend the functionality of the main container.
Without changing the business logic (the main container), one can deploy a logging-agent as a sidecar container.
Example: log processor for the web server, the log processor could be a different container reading logs from the web server.

## Ambassador pattern

A container that proxy the network connection to the main container.
A good example is that you deploy ambassador container which has credentials to Kubernetes API, so you don't have to use authentication from your client. One more example is to use it as application level DNS service discovery. Another good example is using Ambassador as proxy to the Redis caching cluster.

## Adapter pattern

A container that transform output of the main container. Using the adapter pattern means keeping communication between containers consistent. Having a standard way of communicating via a set of contracts helps you to always make requests in the same way, and lets you expect the same response format.
Example: receive and transform raw logs from a container.

# Probes

## Readiness and liveness probes

Example of a `readinessProbe` may look like:
```
containers:
- name: liveness
  image: k8s.gcr.io/busybox
  readinessProbe:
     exec:
       command:
       - cat
       - /tmp/healthy
     initialDelaySeconds: 5
     periodSeconds: 5
```

To perform a probe, the kubelet executes the command `cat /tmp/healthy` in the target container. If the command succeeds, it returns 0, and the kubelet considers the container to be alive and healthy. If the command returns a non-zero value, the kubelet kills the container and restarts it.

Another example may include checking if HTTP GET request is successful:
```
livenessProbe:
      httpGet:
        path: /healthz
        port: 8080
        # optional
        httpHeaders:
        - name: Custom-Header
          value: Awesome
```

### Example 1
Adding a LivenessProbe and a ReadinessProbe on port 80 can be described on full example below.
```
kubectl create deploy nginx --image=nginx --dry-run -o yaml > nginx-deploy.yaml
# define Probes
kubectl expose deploy nginx --name=nginx-svc --port=80 --type=NodePort
```
Answer can be found in `nginx-deploy.yaml`

### Example 2
Create a multi-container pod running both Nginx Server and proxy on port 8080.
Answer can be found in `nginx-proxy-multi-container.yaml`

# Logging

While custom-built tools may be best at testing a deployment, there are some built-in kubectl arguments to begin the process. The first one is describe and the next would be logs.

You can see the details, conditions, volumes and events for an object with describe.
`kubectl describe pod some-pod`
A next step in testing may be to look at the output of containers within a pod.
`kubectl logs some-pod`

## Print Logs for a Container in a Pod
```
kubectl logs nginx-deployment-649897645-jpnmj
# Print logs from exited container
kubectl logs nginx-deployment-649897645-jpnmj -p
# Print the logs that occurred after an absolute time.
kubectl logs nginx-78f5d695bd-czm8z --since-time=2018-11-01T15:00:00Z
# Print logs for the past hour
kubectl logs nginx-78f5d695bd-czm8z --since=1h
```
## Print the logs for all Pods in a Deployment
```
kubectl logs -l app=nginx
# Print logs with timestamps
kubectl logs -l app=nginx --timestamps
```
