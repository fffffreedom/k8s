# Garbage Collection

The role of the Kubernetes garbage collector is to delete certain objects that once had an owner, but no longer have an owner.  

> Note: Garbage collection is a beta feature and is enabled by default in Kubernetes version 1.4 and later.  

Prior to Kubernetes 1.9, the default garbage collection policy for many controller resources was **orphan**.  
This included:  
  - ReplicationController
  - ReplicaSet
  - StatefulSet
  - DaemonSet
  - Deployment.  
For kinds in the extensions/v1beta1, apps/v1beta1, and apps/v1beta2 group versions, 
unless you specify otherwise, dependent objects are orphaned by default.  

In Kubernetes 1.9, for all kinds in the apps/v1 group version, dependent objects are deleted by default.  

**kubectl also supports cascading deletion. To delete dependents automatically using kubectl, set --cascade to true.**  
To orphan dependents, set --cascade to false. The default value for --cascade is true.  

##  Owners and dependents
Some Kubernetes objects are owners of other objects. For example, a ReplicaSet is the owner of a set of Pods. 
The owned objects are called dependents of the owner object（被拥有的object依赖于拥有者）.
Every dependent object has a `metadata.ownerReferences` field that points to the owning object.  

Sometimes, Kubernetes sets the value of ownerReference automatically. 
In 1.8, Kubernetes automatically sets the value of ownerReference for objects created 
or adopted by ReplicationController, ReplicaSet, StatefulSet, DaemonSet, Deployment, Job and CronJob.  
 
You can also specify relationships between owners and dependents by manually setting the ownerReference field.  

### 查看Pod的拥有者
```
kubectl get pods --output=yaml

apiVersion: v1
kind: Pod
metadata:
  ...
  ownerReferences:
  - apiVersion: extensions/v1beta1
    controller: true
    blockOwnerDeletion: true
    kind: ReplicaSet
    name: my-repset
    uid: d9607e19-f88f-11e6-a518-42010a800195
  ...
```

## Controlling how the garbage collector deletes dependents
### cascading deletion
dependents会自动删除吧？是可以配置的。  

When you delete an object, you can specify whether the object’s dependents are also deleted automatically. 
Deleting dependents automatically is called `cascading deletion`.  
If you delete an object without deleting its dependents automatically, the dependents are said to be orphaned.  

There are two modes of cascading deletion: background and foreground.  

#### foreground cascading deletion
In foreground cascading deletion, the root object first enters a “deletion in progress” state. 
In the “deletion in progress” state, the following things are true:  
  - The object is still visible via the REST API
  - The object’s `deletionTimestamp` is set
  - The object’s metadata.finalizers contains the value “foregroundDeletion”.

Once the “deletion in progress” state is set, the garbage collector deletes the object’s dependents.  
Once the garbage collector has deleted all “blocking” dependents (objects with ownerReference.blockOwnerDeletion=true), 
it delete the owner object.  

Note that in the “foregroundDeletion”, only dependents with `ownerReference.blockOwnerDeletion` block the deletion of the owner object.  


If an object’s ownerReferences field is set by a controller (such as Deployment or ReplicaSet), 
blockOwnerDeletion is set automatically and you do not need to manually modify this field.  

#### Background cascading deletion
In background cascading deletion, Kubernetes deletes the owner object immediately and the garbage collector 
then deletes the dependents in the background.  

### Setting the cascading deletion policy
To control the cascading deletion policy, set the propagationPolicy field on the deleteOptions argument when deleting an Object. 
Possible values include:  
  - “Orphan”
  - “Foreground”
  - “Background”

Prior to Kubernetes 1.9, the default garbage collection policy for many controller resources was **orphan**.  
This included:  
  - ReplicationController
  - ReplicaSet
  - StatefulSet
  - DaemonSet
  - Deployment.
For kinds in the extensions/v1beta1, apps/v1beta1, and apps/v1beta2 group versions, 
unless you specify otherwise, dependent objects are orphaned by default.  
In Kubernetes 1.9, for all kinds in the apps/v1 group version, dependent objects are deleted by default.  

Here’s an example that deletes dependents in background:  
```
kubectl proxy --port=8080

curl -X DELETE localhost:8080/apis/extensions/v1beta1/namespaces/default/replicasets/my-repset \
-d '{"kind":"DeleteOptions","apiVersion":"v1","propagationPolicy":"Background"}' \
-H "Content-Type: application/json"
```

## Additional note on Deployments
**When using cascading deletes with Deployments you must use propagationPolicy: Foreground to delete not only the ReplicaSets created, 
but also their Pods**.  
If this type of propagationPolicy is not used, only the ReplicaSets will be deleted, 
and the Pods will be orphaned. See kubeadm/#149 for more information.  

## Reference 
https://kubernetes.io/docs/concepts/workloads/controllers/garbage-collection/  


