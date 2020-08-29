# Common Concepts and Commands

# Basic commands
```
# create Pod 
kubectl run pod --image=nginx --port=80
# create Pod with Service
kubectl run nginx --image=nginx --restart=Never --port=80 --expose --dry-run -o yaml --labels app=dev,tier=web
kubectl run pod --image=nginx --port=80 --expose

# create Deployment and Service
kubectl create deployment nginx --image=nginx --namespace=dev
kubectl expose deployment nginx --name=nginx-service --port=80 --serviceType=ClusterIP

# create Service seprately. labels won't be inherited, but nodePort can be specified
kubectl create service nodeport nginx --tcp=80:80 --node-port=30080

kubectl scale deployment nginx --replicas=3

kubectl set image deployment nginx nginx=nginx:alpine

# add labels
kubectl label pods my-pod new-label=awesome
```

# Common CLI Commands
```
# Describe is a higher level printing operation that may aggregate data from other sources
kubectl describe deployment  nginx

kubectl get deployment nginx
kubectl get -o=wide deployments nginx
kubectl get deployments -o custom-columns="Name:metadata.name,Replicas:spec.replicas,Strategy:spec.strategy.type"
kubectl get deployments -L=app # Print out specific labels each as their own columns

kubectl get deployment --show-labels
kubectl get deployments -l app=nginx

kubectl get deployments --all-namespaces

# Print the JSON representation of the first Deployment in the list on a single line.
kubectl get deployment.v1.apps -o=jsonpath='{.items[0]}{"\n"}'

# Print the metadata.name field for the first Deployment in the list.
kubectl get deployment.v1.apps -o=jsonpath='{.items[0].metadata.name}{"\n"}'
```

# Cluster Info
```
kubectl version
# The kubectl cluster-info prints information about the control plane and add-ons.
kubectl cluster-info

# `kubectl top node` and `kubectl top pod` print information about the top nodes and pods.
kubectl top node

# Print the Resource Types available in the cluster.
kubectl api-resources

# Print the API versions available in the cluster.
kubectl api-versions
```

# Watch Resources for changes

Use `--watch` or `-w` to continuously watch for changes to objects, and print the objects when they are changed or when the watch is reestablished.
```
kubectl get deployments --watch
```

It is possible to have kubectl get continuously watch for changes to objects without fetching them first using the --watch-only flag.
```
kubectl get deployments --watch-only
```

# Explain resources
The `kubectl explain` command can be used to print metadata about specific Resource types. This is useful for learning about the type.
```
kubectl explain deployment --api-version apps/v1
```
# Logging

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
## Print the logs for all Pods for a Workload
```
kubectl logs -l app=nginx
# Print logs with timestamps
kubectl logs -l app=nginx --timestamps
```

# Copying Container Files
Copy requires that tar be installed in the container image.

## Local to Remote

Copy a local file to a remote Pod in a cluster.

Local file format is <path>
Remote file format is <pod-name>:<path>
```
kubectl cp /tmp/foo_dir <some-pod>:/tmp/bar_dir
```

## Remote to Local

Copy a remote file from a Pod to a local file.

Local file format is <path>
Remote file format is <pod-name>:<path>
```
kubectl cp <some-pod>:/tmp/foo /tmp/bar
```

## Specify the Container

Specify the Container within a Pod running multiple containers.

-c <container-name>
```
kubectl cp /tmp/foo <some-pod>:/tmp/bar -c <specific-container>
```

## Namespaces

Set the Pod namespace by prefixing the Pod name with <namespace>/ .

<pod-namespace>/<pod-name>:<path>
```
kubectl cp /tmp/foo <some-namespace>/<some-pod>:/tmp/bar
```

## Exec Shell
```
kubectl exec -it nginx-78f5d695bd-czm8z --sh
```

# Port Forwarding

## Forward Multiple Ports

Listen on ports 5000 and 6000 locally, forwarding data to/from ports 5000 and 6000 in the pod
```
kubectl port-forward pod/mypod 5000 6000
```

## Pod in a Workload

Listen on ports 5000 and 6000 locally, forwarding data to/from ports 5000 and 6000 in a pod selected by the deployment
```
kubectl port-forward deployment/mydeployment 5000 6000
```

## Different Local and Remote Ports

Listen on port 8888 locally, forwarding to 5000 in the pod
```
kubectl port-forward pod/mypod 8888:5000
```
