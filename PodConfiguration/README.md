## Running commands in a container

Run echo command on container start.
Additioanlly ENV variables can be set in CLI command
```
kubectl run nginx --image=nginx --restart=Never --port=80 --dry-run -o yaml --labels app=dev,tier=web --env="HOST"="$(hostname)" --command -- sh echo "Hello from Kubernetes Cluster"
```

## Create a ConfigMap

```
kubectl create configmap config --from-literal=TIME_FREQ=10

kubectl create configmap config --from-file=app.properties

kubectl create configmap config --from-env-file=conf.env
```
Use configuration create from a Pod with:
```
   image: busybox
   env:
    - name: TIME_FREQ  # name of the ENV to be created in a Pod
      valueFrom:
        configMapKeyRef:
          name: config # name of ConfigMap
          key: TIME_FREQ # name of the ENV in config map
```
Inject all the defined variables in a ConfigMap at once into the Pod with:
```
   image: busybox
   envFrom: 
   - configMapRef:
        name: config
```
Config map can be used to inject config file as a volume.
This is useful for example when ConfigMap is created from file (`kubectl create configmap config --from-file=app.properties`). 
The file then will be available inside of a container inside a specified by `mountPath` path,
and can be easily references like `/etc/config/app.properties` using example below.
```
spec:
  containers:
  - name: nginx
    ...
    volumeMounths:
    - name: myvolume # should match name below
      mouthPath: /etc/config
  volumes:
  - name: myvolume
    configMap:
      name: config
```

## Secrets
Secrets are similar somewhat to ConfigMaps but with the difference that they store sensitive values in Base64 format. Secrets optionally can be encrypted using third part tools.

```
kubectl create secret generic app-secret --from-literal=TIME_FREQ=10

kubectl create secret generic app-secret --from-file=app.secrets

kubectl create secret generic app-private-key --from-file=client-key=${LOCAL_PATH}/client.key
```

Inject into a Pod as an environmental variable or a volume:
```
   image: busybox
   env: 
    - name: frequecy 
      valueFrom:
        secretKeyRef: 
          name: app-secret
          key: TIME_FREQ
```

Using Secrets as from a Volume example:
```
spec:
  containers:
  - name: flask-app
    ...
    volumeMounts:
    - mountPath: /etc/key   # key will be available in file /etc/key/client-key, as --from-file=client-key=
      readOnly: true
      name: private-key
  volumes:
  - name: private-key
    secret:
      secretName: app-private-key
```

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

# add labels to the node
kubectl label node worker type=data-processor
```

Taints can be handled by adding tolerations to each pod. Taints are triple of values: `key`=`value`:`effect`.
Effect can be of type `NoSchedule`, `NoExecute`, `PreferNoSchedule`
```
spec:
  containers:
  - name: nginx
    image: nginx
  tolerations:
  - key: "key"
    operator: "Equal"
    value: "value"
    effect: "NoSchedule"
```
`NoSchedule` effect means that no pod will be able to schedule onto the node unless it has a matching toleration.

If effect `NoExecute` is added to a node, then any pods that do not tolerate the taint will be evicted immediately, and pods that do tolerate the taint will never be evicted. However, a toleration with NoExecute effect can specify an optional tolerationSeconds field that dictates how long the pod will stay bound to the node after the taint is added.

`PreferNoSchedule` - is a "soft" version of `NoSchedule`. The system will try to avoid placing a pod that does not tolerate the taint on the node, but it is not required. 

Node labels can be used to assign pods directly to specific node (assuming that notolerations are present).
```
spec:
  containers:
  - name: nginx
    image: nginx
  nodeSelector:
    type: data-processor
```
Or node labels and taints can be combined to add more fine-grained scheduling control.

To prevent from other pods to be scheduled on the node of interest, a combination ot Taints/Tolerations and NodeAffinity is used.
This also ensures that current pod won't be placed on some other node without this particular toleration specified. Pod will be placed on the node that exactly matches both affinity and toleration rules.
```
    spec:
      containers: 
         ...
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: role
                operator: In
                values:
                - core-worker
      tolerations:
      - key: "role"
        operator: "Equal"
        value: "core-worker"
        effect: "NoSchedule"
```
