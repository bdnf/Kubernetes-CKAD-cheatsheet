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
