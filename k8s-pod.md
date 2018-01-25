# pod
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
> https://kubernetes.io/docs/concepts/workloads/pods/pod/  

- A pod (as in a pod of whales or pea pod) is a group of one or more containers (such as Docker containers), 
with shared storage/network, and a specification for how to run the containers.  
pod是一组一个或多个容器（如Docker容器），共享存储/网络，以及如何运行容器的规范。  
- n general, users shouldn’t need to create pods directly. 
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







