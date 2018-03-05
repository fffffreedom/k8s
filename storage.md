# volume-pv-pvc

```
https://kubernetes.io/docs/concepts/storage/volumes/
https://kubernetes.io/docs/concepts/storage/persistent-volumes/
https://kubernetes.io/docs/tasks/configure-pod-container/configure-persistent-volume-storage/
https://kubernetes.io/docs/concepts/storage/storage-classes/
https://kubernetes.io/docs/concepts/storage/dynamic-provisioning/
```

## PV and PVC

A PersistentVolume (PV) is a piece of storage in the cluster that has been provisioned by an administrator.  
A PersistentVolumeClaim (PVC) is a request for storage by a user.  

Cluster administrators need to be able to offer a variety of PersistentVolumes that differ in more ways than just size and 
access modes, without exposing users to the details of how those volumes are implemented. 
For these needs there is the StorageClass resource.  

### Lifecycle of a volume and claim

Provisioning ->　Binding　-> Using -> Reclaiming

#### Provisioning

There are two ways PVs may be provisioned: statically or dynamically.  

##### Static

A cluster administrator creates a number of PVs. They carry the details of the real storage which is available for use by cluster
users.   

集群管理者事先创建好PV，存储的详细信息，比如`name, capacity, accessModes, type`等，都是由PV决定的。  

##### Dynamic

When none of the static PVs the administrator created matches a user’s PersistentVolumeClaim, 
the cluster may try to dynamically provision a volume specially for the PVC.   

This provisioning is based on StorageClasses: the PVC must request a storage class and the administrator must have created 
and configured that class in order for dynamic provisioning to occur. Claims that request the class "" effectively disable 
dynamic provisioning for themselves.  

To enable dynamic storage provisioning based on storage class.  
动态`Provisioning`依赖于`StorageClasses`！  

the cluster administrator needs to enable the `DefaultStorageClass` admission controller on the API server.  

#### Binding








