# rc-rs-deployment
## 关系
rc is short for replication controller  
rs is short for replica set    

A ReplicationController ensures that a specified number of pod replicas are running at any one time.  
ReplicaSet is the next-generation Replication Controller.  

它们在Label selector上的的区别是： 
- rc 只支持基于等式的Label selector(equality-based selector)  
- rs 即支持基于等式的Label selector，又支持基于集合的Label selector(set-based selector)  

还有就是rs不支持rolling-update命令，如果你需要升级你的应用，那么应该使用deployment！  
  
## 何时使用deployment
如果你需要升级你的应用，那么应该使用deployment！  
如果你需要升级你的应用，那么应该使用deployment！ 
如果你需要升级你的应用，那么应该使用deployment！ 

deployment是一个更高层的资源，它使用rs来形成一整套Pod创建、删除、更新的编排机制。
deployment相对于rc一个最大升级是我们可以随时知道当前Pod“部署”的进度。  

当我们使用deployment时，无须关心deployment是如何创建和维护rs的，这一切都是自动发生的！  

## 何时使用rs
您需要自定义更新编排或根本不需要更新。  

# ReplicaSet
## write a rs spec
```
apiVersion: extensions/v1beta1
kind: ReplicaSet
metadata:
  name: frontend
  labels:
    app: guestbook
    tier: frontend
spec:
  replicas: 3
  selector:
    matchLabels:
      tier: frontend
    matchExpressions:
      - {key: tier, operator: In, values: [frontend]}
  template:
    metadata:
      labels:
        app: guestbook
        tier: frontend
      spec:
        containers:
          - name: busy
            images: busybox
            imagePullPolicy: IfNotPresent
            restartPolicy: Never
            env:
            - name: AUTHOR
              value: jonny
            command: ['sh', '-c', 'echo this is $AUTHOR && sleep 10 && exit 0']
```

### label和selector的关系？
  - `.metadata.labels` AND `.spec.template.metadata.labels`  
  The ReplicaSet can itself have labels (`.metadata.labels`).  
  Typically, you would set these the same as the `.spec.template.metadata.labels`.  
  However, they are allowed to be different, and the `.metadata.labels` do not affect the behavior of the ReplicaSet.  

  - `.spec.template.metadata.labels` and `.spec.selector`  
  **The `.spec.template.metadata.labels` must match the `.spec.selector`, or it will be rejected by the API.**  

### 能不能手动创建带有相同label的Pod?
Also you should not normally create any pods whose labels match this selector:  
  - either directly, 
  - with another ReplicaSet, 
  - or with another controller such as a Deployment.  
If you do so, the ReplicaSet thinks that it created the other pods. Kubernetes does not stop you from doing this.  

If you do end up with multiple controllers that have overlapping selectors, you will have to manage the deletion yourself.  

## Working with ReplicaSets
### Deleting a ReplicaSet and its Pods

To delete a ReplicaSet and all its pods, use `kubectl delete`.   
Kubectl will scale the ReplicaSet to zero and wait for it to delete each pod before deleting the ReplicaSet itself. 
If this kubectl command is interrupted, it can be restarted.

When using the REST API or go client library, you need to do the steps explicitly (scale replicas to 0, wait for pod deletions, 
then delete the ReplicaSet).  

### Deleting just a ReplicaSet

You can delete a ReplicaSet without affecting any of its pods, using kubectl delete with the `--cascade=false` option.  
When using the REST API or go client library, simply delete the ReplicaSet object.  

Once the original is deleted, you can create a new ReplicaSet to replace it. 
As long as the old and new `.spec.selector` are the same, then the new one will adopt the old pods. 
However, it will not make any effort to make existing pods match a new, different pod template. 
To update pods to a new spec in a controlled way, use a rolling update.

### Isolating pods from a ReplicaSet
Pods may be removed from a ReplicaSet’s target set by changing their labels. This technique may be used to remove pods from service for debugging, data recovery, etc.  

## ReplicaSet as an Horizontal Pod Autoscaler Target
A ReplicaSet can also be a target for Horizontal Pod Autoscalers (HPA). That is, a ReplicaSet can be auto-scaled by an HPA.   

针对上面的rs，可以创建一个`HorizontalPodAutoscaler`资源对象：  
```
apiVersion: autoscaling/v1
kind: HorizontalPodAutoscaler
metadata:
  name: frontend-scaler
spec:
  scaleTargetRef:
    kind: ReplicaSet
    name: frontend
  minReplicas: 3
  maxReplicas: 10
  targetCPUUtilizationPercentage: 50
```  

也可使用命令来完成同样的任务：  
```
kubectl autoscale rs frontend
```

# ReplicationController
A ReplicationController ensures that a specified number of pod replicas are running at any one time.  

To list all the pods that belong to the ReplicationController in a machine readable form.  
```
$ pods=$(kubectl get pods --selector=app=nginx --output=jsonpath={.items..metadata.name})
echo $pods
nginx-3ntk0 nginx-4ok8v nginx-qrm3m
```

## Writing a ReplicationController Spec
除了kind字段，和rs的spec基本上一样。

## rc的具备的功能
  - Rescheduling
  - Scaling
  - Rolling updates
The two ReplicationControllers would need to create pods with at least one differentiating label, such as the image tag of the primary container of the pod, since it is typically image updates that motivate rolling updates.  

## Using ReplicationControllers with Services
Multiple ReplicationControllers can sit behind a single service, so that, for example, some traffic goes to the old version, and some goes to the new version.  

A ReplicationController will never terminate on its own, but it isn’t expected to be as long-lived as services. Services may be composed of pods controlled by multiple ReplicationControllers, and it is expected that many ReplicationControllers may be created and destroyed over the lifetime of a service.  

## Writing programs for Replication
## Responsibilities of the ReplicationController

# Deployments

