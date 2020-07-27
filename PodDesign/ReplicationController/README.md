# Replication Controller

Jobs are complementary to Replication Controllers.

A Replication Controller manages Pods which are not expected to terminate (e.g. web servers),
and a Job manages Pods that are expected to terminate (e.g. batch tasks).

# Single Job starts controller Pod
[Full example can be found here (running Spark)](https://github.com/kubernetes/examples/tree/master/staging/spark/README.md)

Another pattern is for a single Job to create a Pod which then creates other Pods,
acting as a sort of custom controller for those Pods.
This allows the most flexibility, but may be somewhat complicated to get started with and offers less integration with Kubernetes.

One example of this pattern would be a Job which starts a Pod which runs a script that in turn starts a Spark master controller (see [spark example](https://github.com/kubernetes/examples/tree/master/staging/spark/README.md)), runs a spark driver, and then cleans up.

An advantage of this approach is that the overall process gets the completion guarantee of a Job object, but maintains complete control over what Pods are created and how work is assigned to them.
