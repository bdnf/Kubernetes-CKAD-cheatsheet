
# Troubleshooting

```
kubectl create deployment troubleshoot --image=nginx
$ kubectl exec -ti troubleshoot- -- /bin/sh
```

If the Pod is running, use `kubectl logs pod-name` to view the standard out of the container.

# Debugging
```
kubectl describe pod pod-name

# Check for status i the output
<output_omitted>
Conditions:
Type           Status
Initialized    True
Ready          True
PodScheduled   True
<output_omitted>

# Check if there are any events with errors or warnings
Events:Type    Reason   Age                From                  Message----    ------   ----               ----                  -------Normal  Pulling  34m (x50 over 2d)  kubelet, ckad-2-wdrq  pulling
<output_omitted>

# Check the container logs
kubectl logs pod-name

# Check to make sure the container is able to use DNS
kubectl exec  -it  pod-name -c container-name -- sh
# Inside the container
$ nslookup example.com
$ cat /etc/resolv.conf

# Make sure traffic is being sent to the correct Pod.
kubectl get svc

# Verify an endpoint for the service exists and has expected values, including namespaces, ports and protocols
kubectl get endpoint

# If the containers, services and endpoints are working the issue may be with an infrastructure service likekube-proxy.Ensure it’s running, then look for errors in the logs.

ps -elf |grep kube-proxy
journalctl -a | grep proxy

# Look at both of the proxy logs. Lines which begin with the characterIareinfo,Eareerrors. In this example the  #lastmessage says access to listing an endpoint was denied by RBAC. It was because a default installation via Helm #wasn’tRBAC aware. If not using command line completion, view the possible pod names first.

kubectl -n kube-system get pod
kubectl -n kube-system logs kube-proxy-abdcs

# Check expected rules are created
sudo iptables-save |grep secondapp

``
