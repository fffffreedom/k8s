# PV, PVC and StorageClass

```
https://kubernetes.io/docs/concepts/storage/persistent-volumes/
https://kubernetes.io/docs/tasks/configure-pod-container/configure-persistent-volume-storage/
https://kubernetes.io/docs/concepts/storage/storage-classes/
https://kubernetes.io/docs/concepts/storage/dynamic-provisioning/
```

PersistentVolume types are implemented as plugins.

## PV and PVC

A PersistentVolume (PV) is a piece of storage in the cluster that has been provisioned by an administrator.  
A PersistentVolumeClaim (PVC) is a request for storage by a user.  

Cluster administrators need to be able to offer a variety of PersistentVolumes that differ in more ways than just size and 
access modes, without exposing users to the details of how those volumes are implemented. 
For these needs there is the `StorageClass` resource.  

### 举个例子

```
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: slow
provisioner: kubernetes.io/gce-pd
parameters:
  type: pd-standard
```
```
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast
provisioner: kubernetes.io/gce-pd
parameters:
  type: pd-ssd
```
```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: claim1
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: fast
  resources:
    requests:
      storage: 30Gi
```

### Lifecycle of a volume and claim

Provisioning -> Binding -> Using -> Storage Object in Use Protection -> Reclaiming

#### Provisioning

There are two ways PVs may be provisioned: statically or dynamically.  

- Static

A cluster administrator creates a number of PVs. They carry the details of the real storage which is available for use by cluster users.   

集群管理者事先创建好PV，存储的详细信息，比如`name, capacity, accessModes, type`等，都是由PV决定的。  

- Dynamic

When none of the static PVs the administrator created matches a user’s PersistentVolumeClaim, 
the cluster may try to dynamically provision a volume specially for the PVC.   

This provisioning is based on StorageClasses: the PVC must request a storage class and the administrator must have created 
and configured that class in order for dynamic provisioning to occur. Claims that request the class "" effectively disable 
dynamic provisioning for themselves.  

To enable dynamic storage provisioning based on storage class.  
**动态`Provisioning`依赖于`StorageClasses`！**  

the cluster administrator needs to enable the `DefaultStorageClass` admission controller on the API server.  

```
--enable-admission-plugins
```

#### Binding

A control loop in the master watches for new PVCs, finds a matching PV (if possible), and binds them together.

A PVC to PV binding is a one-to-one mapping.

#### Using

Pods use claims as volumes. The cluster inspects the claim to find the bound volume and mounts that volume for a pod.

#### Storage Object in Use Protection（beta）

该功能防止正在被使用的PVC被删除！

#### Reclaiming（回收）

The reclaim policy for a **PersistentVolume** tells the cluster what to do with the volume after it has been released of its claim

回收策略是针对PV的，包括以下类型：

- Retained（保留）

当和PV绑定的PVC被删除后，PV仍然存在，并处于`released`状态。但它还不能被 claim （被申请或分配），因为还保留着旧上一个 claim 的数据。
管理员通过以下步骤可以手动去回收这个 voloume：

（1）删除PV。和PV相关联的外部存储介质或集群在PV被删除之后，还是存在的。

（2）手动清除相应存储介质上的旧数据。

（3）手动删除相应存储介质上的用来存储数据的池子（如ceph块存储会创建一个image来存储这个数据）。如果想重新利用，可以再创建。

- Recycled（循环）

> Warning: The Recycle reclaim policy is deprecated. Instead, the recommended approach is to use dynamic provisioning.

如果 volume plugin 支持，该策略会对 volume 执行一个基本的清除动作（rm -rf /thevolume/*)，使得该volume可以重新使用。

当然，也可以通过在Pod的template里指定命令行来删除 volume 上的数据。不过这样需要写死 volume 的挂载路径。

- Deleted（删除）

对于支持通过删除来回收volume的插件，删除操作会删除掉PV和外部相关的存储介质。通过动态分配(dynamically provisioned)的volume，
会默认使用相关StorageClass的回收策略（reclaim policy），默认值为**`delete`**。

管理员需要根据用户的期望来配置StorageClass，否则PV在被创建之后，需要通过 edit 和 patch 来修改它的回收策略！

### 扩展 PVC

k8s的1.8版本添加了 PV 扩展功能（alpha版本）；1.9中添加了以下 volume 类型的 PVC 扩展功能：

- gcePersistentDisk  
- awsElasticBlockStore  
- Cinder  
- glusterfs  
- rbd  

通过设置 ExpandPersistentVolumes feature gate 为 true 来使能 PVC 扩展功能，此外还应该使能 PersistentVolumeClaimResize admission plugin 来对可调整容量的 volume 进行额外的验证。

一旦 PersistentVolumeClaimResize admission plugin 被使能，只有 allowVolumeExpansion 配置被设置为 true 的 storageClass才允许进行
resizing.  

```
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: gluster-vol-default
provisioner: kubernetes.io/glusterfs
parameters:
  resturl: "http://192.168.10.100:8080"
  restuser: ""
  secretNamespace: ""
  secretName: ""
allowVolumeExpansion: true
```

一旦 feature gate 和 前面提到的 admission plug-in 被使能，用户可以通过修改 PVC（如编辑yaml文件）来申请更大的 volume，
这会触发存储集群的动态调整 volume 的大小，而不是重新创建一个新的 volume。

而如果要对已经存有文件系统的 volume 进行扩展，只有在一个新启动的Pod以 ReadWrite 的模式使用 PVC时，才能触发文件系统的resizing！
换句话说，如果一个 volume 已经被 pod 或者 deployment 使用，要想触发文件系统 resizing，需要先删除 pod，再重新创建 pod!
支持 resizing 的文件类型为：  
- XFS  
- Ext3 and Ext4

### Types of Persistent Volumes

Kubernetes currently supports the following plugins:

- rbd  
- cephfs  
- nfs  
- glusterfs  
- cinder  
- ...

## PV

Each PV contains a spec and status, which is the specification and status of the volume.

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
  mountOptions:
    - hard
    - nfsvers=4.1
  nfs:
    path: /tmp
    server: 172.17.0.2
```

下面讲下各配置字段的的聚少含义：  

- capacity

Currently, storage size is the only resource that can be set or requested. Future attributes may include IOPS, throughput, etc.

- volumeMode

|version|value|
|-----|-----|
|1.8|Filesystem|
|1.9|Filesystem, Block|

- accessModes

只要资源提供者（resource provider）支持，PV 能被挂载到主机上的任何路径。不同的资源提供者拥有不同的能力，该配置需要配置 volume 所支持的访问方式！

访问方式类型（命令行缩写）：
- ReadWriteOnce（RWO） – the volume can be mounted as read-write by a single node  
- ReadOnlyMany（ROX） – the volume can be mounted read-only by many nodes  
- ReadWriteMany(RWX) – the volume can be mounted as read-write by many nodes  

volume can only be mounted using one access mode at a time, even if it supports many.

- storageClassName

A PV can have a class, which is specified by setting the storageClassName attribute to the name of a StorageClass. A PV of a particular class can only be bound to PVCs requesting that class. A PV with no storageClassName has no class and can only be bound to PVCs that request no particular class.

In the past, the annotation volume.beta.kubernetes.io/storage-class was used instead of the storageClassName attribute. This annotation is still working, however it will become fully deprecated in a future Kubernetes release.

- persistentVolumeReclaimPolicy

Reclaim Policy: Retain, Recycle and Delete

- mountOptions

A Kubernetes administrator can specify additional mount options for when a Persistent Volume is mounted on a node.

> Note: Not all Persistent volume types support mount options.

Mount options are not validated, so mount will simply fail if one is invalid.

In the past, the annotation `volume.beta.kubernetes.io/mount-options` was used instead of the `mountOptions` attribute. This annotation is still working, however it will become fully deprecated in a future Kubernetes release.

- Phase  

A volume will be in one of the following phases:

  - Available – a free resource that is not yet bound to a claim  
  - Bound – the volume is bound to a claim  
  - Released – the claim has been deleted, but the resource is not yet reclaimed by the cluster  
  - Failed – the volume has failed its automatic reclamation  
  - The CLI will show the name of the PVC bound to the PV.  

## PVC

Each PVC contains a spec and status, which is the specification and status of the claim.

```
kind: PersistentVolumeClaim
apiVersion: v1
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
  selector:
    matchLabels:
      release: "stable"
    matchExpressions:
      - {key: environment, operator: In, values: [dev]}
```

- accessModes  
- volumeMode  
- resources    
- selector  
- storageClassName  

A claim can request a particular class by specifying the name of a StorageClass using the attribute storageClassName. 
Only PVs of the requested class, ones with the same storageClassName as the PVC, can be bound to the PVC.
PVC 通过 storageClassName 属性来指定需要申请的特定的 storageClass，当 PV 的 storageClassName 和 PVC 的一样时，PV 才绑定到 PVC！




## PV支持的读写模式

| Volume Plugin        | ReadWriteOnce | ReadOnlyMany | ReadWriteMany                      |
| -------------------- | ------------- | ------------ | ---------------------------------- |
| AWSElasticBlockStore | ✓             | -            | -                                  |
| AzureFile            | ✓             | ✓            | ✓                                  |
| AzureDisk            | ✓             | -            | -                                  |
| CephFS               | ✓             | ✓            | ✓                                  |
| Cinder               | ✓             | -            | -                                  |
| FC                   | ✓             | ✓            | -                                  |
| FlexVolume           | ✓             | ✓            | -                                  |
| Flocker              | ✓             | -            | -                                  |
| GCEPersistentDisk    | ✓             | ✓            | -                                  |
| Glusterfs            | ✓             | ✓            | ✓                                  |
| HostPath             | ✓             | -            | -                                  |
| iSCSI                | ✓             | ✓            | -                                  |
| PhotonPersistentDisk | ✓             | -            | -                                  |
| Quobyte              | ✓             | ✓            | ✓                                  |
| NFS                  | ✓             | ✓            | ✓                                  |
| RBD                  | ✓             | ✓            | -                                  |
| VsphereVolume        | ✓             | -            | - (works when pods are collocated) |
| PortworxVolume       | ✓             | -            | ✓                                  |
| ScaleIO              | ✓             | ✓            | -                                  |
| StorageOS            | ✓             | -            | -                                  |
