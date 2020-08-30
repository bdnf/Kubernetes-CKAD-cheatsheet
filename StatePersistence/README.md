# Persistent Volumes

A Kubernetes volume shares at least the Pod lifetime, not the containers within. Should a container terminate, the data would continue to be available to the new container. A volume can persist longer than a Pod, and can be accessed by multiple Pods, using PersistentVolumeClaims. This allows for state persistency.
```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv0003
spec:
  capacity:
    storage: 5Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Recycle
  storageClassName: slow
```

# PersistentVolumeClaims

```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: myclaim
spec:
  accessModes:
    - ReadWriteOnce
  volumeMode: Filesystem
  resources:
    requests:
      storage: 8Gi
  storageClassName: slow
```

# Volume Options

For storage lifetime to be longer that lifetime of a Pod, you can use Persistent Volumes. These allow for empty or pre-populated volumes to be claimed by a Pod using a Persistent Volume Claim, then outlive the Pod. Data inside the volume could then be used by another Pod.

Another options are to use `Secrets` or `ConfigMaps`. Encoded data can be passed using a Secret and non-encoded data can be passed with a ConfigMap. These can be used to pass important data like SSH keys, passwords, or even a configuration (or other) files.

`emptyDir` option will create the directory in the container, but not mount any storage. Any data created is written to the shared container space. As a result, it would not be persistent storage. When the Pod is destroyed, the directory would be deleted along with the container.

```
spec:
  ...
    volumeMounts:
        - mountPath: /scratch
          name: scratch-volume
  volumes:
  - name: scratch-volume
          emptyDir: {}
```
`hostPath` mounts a resource from the host node filesystem. The resource could be a directory, file socket, character, or block device. These resources must already exist on the host to be used.
Options to create the resources on the host if they are not exist:
- `DirectoryOrCreate`
- `FileOrCreate`

```
volumes:
- name: local-dir
  hostPath:
     path: /etc/volume
```
# Volume affinity

Host volume can be used with nodeAffinity rules:

```
apiVersion: v1
kind: PersistentVolume
...
spec:
  capacity:
    storage: 8Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/home/ubuntu/volume/data/prometheus-server"
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - key: kubernetes.io/hostname
          operator: In
          values:
          - node02
```

# Cloud Volumes

`GCEpersistentDisk` or `awsElasticBlockStore` allows you to mount GCE and EBS disks in your Pods.

`NFS (Network File System)` and `iSCSI (Internet Small Computer System Interface)` are straightforward choices for multiple readers scenarios.

`rbd` for block storage or `CephFS` and `GlusterFS`, if available in your Kubernetes cluster, can be a good choice for multiple writer needs.
