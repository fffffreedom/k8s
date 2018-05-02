# volume

容器中的磁盘文件是短暂的，这会给一些特别的应用带来不便：  
- 当容器crash了，Kubelet会重新创建一个容器，之前容器里的文件就都丢失了；
- 当一个Pod包含多个容器时，它们之间要共享文件；

kubernetes用volume来解决这些问题。volume的生命周期和Pod保持一致，当Pod销毁时，volume也就被销毁或者umount。kubernetes支持多种volume，
Pod可以同时使用它们。  

volume只是一个目录，可能包含一些数据，Pod里的容器可以访问这些数据。volume的属性由volume类型决定。  

## 如何使用volume

To use a volume, a pod specifies what volumes to provide for the pod (the `spec.volumes` field) and where to mount those into containers (the `spec.containers.volumeMounts` field).  
在Pod的`spec.volumes`中指定volume的属性，在container的`spec.containers.volumeMounts`中使用volume.  

## volume

要查看container相关的volume路径，可以使用`docker inspect -f`来查看。  

### emptyDir

```
apiVersion: v1
kind: Pod
metadata:
  name: emptydir
spec:
  volumes:
  - name: emptydir
    emptyDir: {}
  containers:
  - image: busybox
    name: emptydir
    command:
    - "sleep"
    - "3600"
    imagePullPolicy: IfNotPresent
    volumeMounts:
    - mountPath: /data
      name: emptydir
```

查看Volume情况：  
```
docker inspect -f '{{json .Mounts}}' 624175c09a00 | jq .
[
  {
    "Source": "/var/lib/kubelet/pods/e9d5d076-2010-11e8-99b6-fa163e4314ee/containers/emptydir/4ffa69d4",
    "Destination": "/dev/termination-log",
    "Mode": "",
    "RW": true,
    "Propagation": "rprivate"
  },
  {
    "Source": "/var/lib/kubelet/pods/e9d5d076-2010-11e8-99b6-fa163e4314ee/volumes/kubernetes.io~empty-dir/emptydir",
    "Destination": "/data",
    "Mode": "",
    "RW": true,
    "Propagation": "rprivate"
  },
  {
    "Source": "/var/lib/kubelet/pods/e9d5d076-2010-11e8-99b6-fa163e4314ee/volumes/kubernetes.io~secret/default-token-fl97p",
    "Destination": "/var/run/secrets/kubernetes.io/serviceaccount",
    "Mode": "ro",
    "RW": false,
    "Propagation": "rprivate"
  },
  {
    "Source": "/var/lib/kubelet/pods/e9d5d076-2010-11e8-99b6-fa163e4314ee/etc-hosts",
    "Destination": "/etc/hosts",
    "Mode": "",
    "RW": true,
    "Propagation": "rprivate"
  }
]

```

当pod终止后，/var/lib/kubelet/pods/e9d5d076-2010-11e8-99b6-fa163e4314ee/volumes/kubernetes.io~empty-dir/emptydir目录将被删除，
里面的文件自然也就不存在了！  

### hostPath

```
apiVersion: v1
kind: Pod
metadata:
  name: hostpath
spec:
  volumes:
  - name: data
    hostPath:
      path: /var/lib/docker/
  containers:
  - image: busybox
    name: hostpath
    command:
    - "sleep"
    - "3600"
    imagePullPolicy: IfNotPresent
    volumeMounts:
    - mountPath: /data
      name: data
```

### rbd
### cephfs
### glusterfs
### nfs
### secret

## subPath

Sometimes, it is useful to share one volume for multiple uses in a single pod. The volumeMounts.subPath property can be used to specify a sub-path inside the referenced volume instead of its root.

## Resources

emptyDir类型的volume，使用何种存储介质（SSD,SATA等），由于`/var/lib/kubelet`路径所在目录的存储介质决定；emptyDir and hostPath volume的容量大小也没有规定，由其所在的系统的磁盘空间决定。

In the future, we expect that emptyDir and hostPath volumes will be able to request a certain amount of space using a resource specification, and to select the type of media to use, for clusters that have several media types.

## Out-of-Tree Volume Plugins

The Out-of-tree volume plugins include the Container Storage Interface (CSI) and FlexVolume. They enable storage vendors to create custom storage plugins without adding them to the Kubernetes repository.

Before the introduction of CSI and FlexVolume, all volume plugins (like volume types listed above) were “in-tree” meaning they were built, linked, compiled, and shipped with the core Kubernetes binaries and extend the core Kubernetes API.

- CSI  
- FlexVolume  

## Mount propagation

Mount propagation allows for sharing volumes mounted by a Container to other Containers in the same Pod, or even to other Pods on the same node.

If the “MountPropagation” feature is disabled or a Pod does not explicitly specify specific mount propagation, volume mounts in the Pod’s Containers are not propagated. 

### Configuration

Edit your Docker’s systemd service file. Set MountFlags as follows:
```
MountFlags=shared
```
Or, remove `MountFlags=slave` if present. Then restart the Docker daemon:
```
$ sudo systemctl daemon-reload
$ sudo systemctl restart docker
```
