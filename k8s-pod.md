# pod
## overview  

> https://kubernetes.io/docs/concepts/workloads/pods/pod-overview/  

- **pod is the smallest and simplest unit in the Kubernetes object model that you create or deploy. 
A Pod represents a running process on your cluster（是一个正在运行的进程）.**  
- A Pod encapsulates an application container (or, in some cases, multiple containers), 
storage resources, a unique network IP, and options that govern how the container(s) should run.  
- 一个pod可以包含一个或者多个container。  
- k8s通过副本控制器来管理pod的数目。  
- 一个pod里的container共享pod的网络和存储。  
- The Pod itself does not run, but is an environment the containers run in and persists until it is deleted.  
- Pods do not, by themselves, self-heal. If a Pod is scheduled to a Node that fails, 
or if the scheduling operation itself fails, the Pod is **deleted**;
likewise, a Pod won’t survive an eviction due to a lack of resources or Node maintenance.  
- pod还不支持migration.(In the future, a higher-level API may support pod migration.)  
- 各种controller通过Pod Templates来创建pod.
```
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
spec:
  containers:
  - name: myapp-container
    image: busybox
    command: ['sh', '-c', 'echo Hello Kubernetes! && sleep 3600']
```
## pod  

> https://kubernetes.io/docs/concepts/workloads/pods/pod/  

- A pod (as in a pod of whales or pea pod) is a group of one or more containers (such as Docker containers), 
with shared storage/network, and a specification for how to run the containers.  
pod是一组一个或多个容器（如Docker容器），共享存储/网络，以及如何运行容器的规范。  
- In general, users shouldn’t need to create pods directly. 
They should almost always use controllers even for singletons, for example, Deployments). 
Controllers provide self-healing with a cluster scope, as well as replication and rollout management. 
Controllers like StatefulSet can also provide support to stateful pods.  
使用controller，如Deployment，来部署或管理pod，会带来众多好处！ 
- 如果kubelet或者容器管理器在等待pod进程终止时重启，它会在一定时间后重试。
```

```
- Force deletion of pods  
不用等待一定的时间！  
- Privileged mode for pod containers  
From Kubernetes v1.1, any container in a pod can enable privileged mode, 
using the privileged flag on the SecurityContext of the container spec. 
This is useful for containers that want to use linux capabilities like manipulating the network stack and accessing devices. 
Processes within the container get almost the same privileges that are available to processes outside a container.  
注意master和node节点上的kubernetes版本是否匹配！  
## pod-lifecycle

> https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/  

- Pod phase  
  - Pending  
  - Running  
  - Succeeded  
  - Failed  
  - Unknown  
- Pod status  
Pod 有一个 PodStatus 对象，其中包含一个 PodCondition 数组。 PodCondition 数组的每个元素都有一个 type 字段和一个 status 字段。
type 字段是字符串，可能的值有 PodScheduled、Ready、Initialized 和 Unschedulable。  
status 字段是一个字符串，可能的值有 True、False 和 Unknown。  
https://kubernetes.io/cn/docs/concepts/workloads/pods/pod-lifecycle/  
- Container probes  
A Probe is a diagnostic performed periodically by the kubelet on a Container.   
To perform a diagnostic, the kubelet calls a Handler implemented by the Container. There are three types of **handlers**:  
  - ExecAction  
  - TCPSocketAction  
  - HTTPGetAction  
Each probe has one of three results:  
  - Success: The Container passed the diagnostic.  
  - Failure: The Container failed the diagnostic.  
  - Unknown: The diagnostic failed, so no action should be taken.  

容器状态探测方式：  
  - livenessProbe  
Indicates whether the Container is running. If the liveness probe fails, the kubelet kills the Container, and the Container is subjected to its restart policy. If a Container does not provide a liveness probe, the default state is Success.
  - readinessProbe  
Indicates whether the Container is ready to service requests. If the readiness probe fails, the endpoints controller removes the Pod’s IP address from the endpoints of all Services that match the Pod. The default state of readiness before the initial delay is Failure. If a Container does not provide a readiness probe, the default state is Success.  
### When should you use liveness or readiness probes?  
If the process in your Container is able to crash on its own whenever it encounters an issue or becomes unhealthy, you do not necessarily need a liveness probe; the kubelet will automatically perform the correct action in accordance with the Pod’s restartPolicy. 即当程序可以在碰到问题或者不健康时自动crash，那就不需要liveness probe了，因为kubelet会根据pod的restartPolicy自动做出正常的行为。  

If you’d like your Container to be killed and restarted if a probe fails, then specify a liveness probe, and specify a restartPolicy of Always or OnFailure.  

If you’d like to start sending traffic to a Pod only when a probe succeeds, specify a readiness probe.  
readiness probe in the spec means that the Pod will start without receiving any traffic and only start receiving traffic after the probe starts succeeding.  

If you want your Container to be able to take itself down for maintenance, you can specify a readiness probe.  
Note that if you just want to be able to drain requests when the Pod is deleted, you do not necessarily need a readiness probe.  

### Restart policy
A PodSpec has a restartPolicy field with possible values, The default value is Always:  
  - Always
  - OnFailure
  - Never.   

restartPolicy applies to all Containers in the Pod.   
restartPolicy only refers to restarts of the Containers by the kubelet on the same node.  
Failed Containers that are restarted by the kubelet are restarted with an exponential back-off delay (10s, 20s, 40s …) capped at five minutes, and is reset after ten minutes of successful execution. **As discussed in the Pods document, once bound to a node, a Pod will never be rebound to another node.**  

### Pod lifetime  
In general, Pods do not disappear until someone destroys them. This might be a human or a controller.  
The only exception to this rule is that Pods with a phase of Succeeded or Failed for more than some duration (determined by the master) will expire and be automatically destroyed.(Pod GC)  
If a node dies or is disconnected from the rest of the cluster, Kubernetes applies a policy for setting the phase of all Pods on the lost node to Failed.  

### pod状态转换
TODO  

## Init Containers
1.6 beta版本之前，可以通过annotations来指定init container的信息；且annotations指定的值会覆盖PodSpec field该在1.6版本之后，这种方式已经不支持了，只支持在PodSpec中指定，具体格式，下面给出。  

### Understanding Init Containers  
一个Pod里可以包含一个或者多个init container，它们在app container运行之前运行，进行初始化工作。  

init和app container相似，除了：  
- init总是运行到结束；  
- init必须串行执行，即只有在上一个init container成功运行完成，下一个init才能开始运行。  

如果pod的init container执行失败，k8s会不断地重启pod，直到init container执行成功；但如果pod的`restartPolicy`为`Never`，则不会被重启。  

### 返回状态
The status of the init containers is returned in `status.initContainerStatuses` field as an array of the container statuses (similar to the `status.containerStatuses` field).  




