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

#### 返回状态
The status of the init containers is returned in `status.initContainerStatuses` field as an array of the container statuses (similar to the `status.containerStatuses` field).  

#### Differences from regular Containers
Init Containers support all the fields and features of app Containers, including resource limits, volumes, and security settings. However, the resource requests and limits for an Init Container are handled slightly differently, which are documented in Resources below. Also, Init Containers do not support readiness probes because they must run to completion before the Pod can be ready.

If multiple Init Containers are specified for a Pod, those Containers are run one at a time in sequential order. Each must succeed before the next can run. When all of the Init Containers have run to completion, Kubernetes initializes the Pod and runs the application Containers as usual.

## init container用来做什么？

> https://kubernetes.io/docs/concepts/workloads/pods/init-containers/  

init container的镜像和app container的镜像是分开的，对于启动相关的代码，init有优势：  

- They can contain and run utilities that are not desirable to include in the app Container image for security reasons.  
- They can contain utilities or custom code for setup that is not present in an app image. For example, there is no need to make an image FROM another image just to use a tool like sed, awk, python, or dig during setup.  
- The application image builder and deployer roles can work independently without the need to jointly build a single app image.  
- They use Linux namespaces so that they have different filesystem views from app Containers. Consequently, they can be given access to Secrets that app Containers are not able to access.  
- They run to completion before any app Containers start, whereas app Containers run in parallel, so Init Containers provide an easy way to block or delay the startup of app Containers until some set of preconditions are met.  

一些例子：  

- Wait for a service to be created with a shell command like:  
```
for i in {1..100}; do sleep 1; if dig myservice; then exit 0; fi; done; exit 1
```
- Register this Pod with a remote server from the downward API with a command like:  
```
curl -X POST http://$MANAGEMENT_SERVICE_HOST:$MANAGEMENT_SERVICE_PORT/register -d 'instance=$(<POD_NAME>)&ip=$(<POD_IP>)'
```
- Wait for some time before starting the app Container with a command like sleep 60.  
- Clone a git repository into a volume.  
- Place values into a configuration file and run a template tool to dynamically generate a configuration file for the main app Container. For example, place the POD_IP value in a configuration and generate the main app configuration file using Jinja.  

more to see:  
https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-initialization/  
https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/  

## init container yaml

init container在各个版本的yaml格式。  

例子为一个POD有两个init container，运行顺序为：myservice, mydb，下面是各个版本的yaml文件定义：  

#### k8s v1.5
```
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
  annotations:
    pod.beta.kubernetes.io/init-containers: '[
        {
            "name": "init-myservice",
            "image": "busybox",
            "command": ["sh", "-c", "until nslookup myservice; do echo waiting for myservice; sleep 2; done;"]
        },
        {
            "name": "init-mydb",
            "image": "busybox",
            "command": ["sh", "-c", "until nslookup mydb; do echo waiting for mydb; sleep 2; done;"]
        }
    ]'
spec:
  containers:
  - name: myapp-container
    image: busybox
    command: ['sh', '-c', 'echo The app is running! && sleep 3600']
```
#### k8s v1.6  
There is a new syntax in Kubernetes 1.6, although the old annotation syntax still works for 1.6 and 1.7. The new syntax must be used for 1.8 or greater. We have moved the declaration of init containers to spec:  
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
    command: ['sh', '-c', 'echo The app is running! && sleep 3600']
  initContainers:
  - name: init-myservice
    image: busybox
    command: ['sh', '-c', 'until nslookup myservice; do echo waiting for myservice; sleep 2; done;']
  - name: init-mydb
    image: busybox
    command: ['sh', '-c', 'until nslookup mydb; do echo waiting for mydb; sleep 2; done;']
```

1.5 syntax still works on 1.6, but we recommend using 1.6 syntax.  In Kubernetes 1.6, Init Containers were made a field in the API. The beta annotation is still respected in 1.6 and 1.7, but is not supported in 1.8 or greater.  

Yaml file below outlines the mydb and myservice services:  
两个init container对应的service的yaml文件：  
```
kind: Service
apiVersion: v1
metadata:
  name: myservice
spec:
  ports:
  - protocol: TCP
    port: 80
    targetPort: 9376
---
kind: Service
apiVersion: v1
metadata:
  name: mydb
spec:
  ports:
  - protocol: TCP
    port: 80
    targetPort: 9377
```

#### 调试命令

```
$ kubectl create -f myapp.yaml
pod "myapp-pod" created
$ kubectl get -f myapp.yaml
NAME        READY     STATUS     RESTARTS   AGE
myapp-pod   0/1       Init:0/2   0          6m
$ kubectl describe -f myapp.yaml
$ kubectl logs myapp-pod -c init-myservice # Inspect the first init container
$ kubectl logs myapp-pod -c init-mydb      # Inspect the second init container
```

### 详细的行为

在POD的启动过程中，在network和volume初始好后，init container是顺序执行的。每个init container必须正确退出后，下面一个才开始执行。
如果init container运行时失败或者失败退出，会根据pod的restartPolicy进行重试。如果Pod的restartPolicy设置为 Always，init container
的restartPolicy则为OnFailure.  

> https://kubernetes.io/cn/docs/concepts/workloads/pods/init-containers/  

除非所有init container成功执行，否则Pod不会处于Ready状态。Init 容器的端口不会在 Service 中进行聚集。 正在初始化中的 Pod 处于 Pending 状态，但应该会将条件 Initializing 设置为 true。（不是太明白）  

如果Pod重启，所有的init container会再次被执行。  

对 Init 容器 spec 的修改，被限制在容器 image 字段中。 更改 Init 容器的 image 字段，等价于重启该 Pod。  

因为 Init 容器可能会被重启、重试或者重新执行，所以 Init 容器的代码应该是幂等的。 特别地，被写到 EmptyDirs 中文件的代码，应该对输出文件可能已经存在做好准备。  

Init 容器具有应用容器的所有字段。 然而 Kubernetes 禁止使用 readinessProbe，因为 Init 容器不能够定义不同于完成（completion）的就绪（readiness）。 这会在验证过程中强制执行。  

在 Pod 上使用 activeDeadlineSeconds，在容器上使用 livenessProbe，这样能够避免 Init 容器一直失败。 这就为 Init 容器活跃设置了一个期限。  

在 Pod 中的每个 app 和 Init 容器的名称必须唯一；与任何其它容器共享同一个名称，会在验证时抛出错误。  

### Pod 重启的原因
Pod 能够重启，会导致 Init 容器重新执行，主要有如下几个原因：  

- 用户更新 PodSpec 导致 Init 容器镜像发生改变。应用容器镜像的变更只会重启应用容器。  
- Pod 基础设施容器被重启。这不多见，但某些具有 root 权限可访问 Node 的人可能会这样做。  
- 当 restartPolicy 设置为 Always，Pod 中所有容器会终止，强制重启，由于垃圾收集导致 Init 容器完成的记录丢失。  

### 支持与兼容性

Apiserver 版本为 1.6 或更高版本的集群，通过使用 spec.initContainers 字段来支持 Init 容器。 之前的版本可以使用 alpha 和 beta 注解支持 Init 容器。 spec.initContainers 字段也被加入到 alpha 和 beta 注解中，所以 Kubernetes 1.3.0 版本或更高版本可以执行 Init 容器，并且 1.6 版本的 apiserver 能够安全的回退到 1.5.x 版本，而不会使存在的已创建 Pod 失去 Init 容器的功能。  



