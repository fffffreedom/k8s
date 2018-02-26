# DaemonSet
## What is a DaemonSet? one-pod-per-node

A DaemonSet ensures that all (or some) Nodes run a copy of a Pod. As nodes are added to the cluster, Pods are added to them.   
确保一个或一些node上只运行一个Pod实例。当node加入集群时，Pod就会被绑定到Pod.  

As nodes are removed from the cluster, those Pods are **garbage collected**. Deleting a DaemonSet will clean up the Pods it created.  
当Node被删除时，Pod会被GC；DaemonSet所创建的Pod会随着其被删除而被清理掉！  

## 使用场景：  
- running a cluster storage daemon, such as glusterd, ceph, on each node.  
- running a logs collection daemon on every node, such as fluentd or logstash.  
- running a node monitoring daemon on every node, such as Prometheus Node Exporter, 
collectd, Datadog agent, New Relic agent, or Ganglia gmond.  

在一些简单的使用场景下，一个daemon可以只用一个DaemonSet；但在一些复杂的情况下，可能由于节点的CPU和内存不一样，
在编写DaemonSet的yaml文件时，需要根据情况指定不同的配置（如flag或者resource requests），进而使得一个daemon需要多个DaemonSet来表示！

## Writing a DaemonSet Spec
```
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd-elasticsearch
  namespace: kube-system
  labels:
    k8s-app: fluentd-logging
spec:
  selector:
    matchLabels:
      name: fluentd-elasticsearch
  template:
    metadata:
      labels:
        name: fluentd-elasticsearch
    spec:
      tolerations:
      - key: node-role.kubernetes.io/master
        effect: NoSchedule
      containers:
      - name: fluentd-elasticsearch
        image: gcr.io/google-containers/fluentd-elasticsearch:1.20
        resources:
          limits:
            memory: 200Mi
          requests:
            cpu: 100m
            memory: 200Mi
        volumeMounts:
        - name: varlog
          mountPath: /var/log
        - name: varlibdockercontainers
          mountPath: /var/lib/docker/containers
          readOnly: true
      terminationGracePeriodSeconds: 30
      volumes:
      - name: varlog
        hostPath:
          path: /var/log
      - name: varlibdockercontainers
        hostPath:
          path: /var/lib/docker/containers
```
### Required Fields
  - apiVersion
  - kind
  - metadata fields
  - spec
### Pod Template
  - The `.spec.template` is one of the required fields in .spec.
  - A Pod Template in a DaemonSet must have a RestartPolicy equal to Always, or be unspecified, which defaults to Always.
  - Pod Selector  
  As of Kubernetes 1.8, you must specify a pod selector that matches the labels of the `.spec.template`. 
  The pod selector will no longer be defaulted when left empty. Selector defaulting was not compatible with kubectl apply.
  **Also, once a DaemonSet is created, its spec.selector can not be mutated.** 
  Mutating the pod selector can lead to the unintentional orphaning of Pods, and it was found to be confusing to users.  
  
The spec.selector is an object consisting of two fields:  
  - matchLabels
  - matchExpressions  
When the two are specified the result is ANDed.  

If the `.spec.selector` is specified, it must match the `.spec.template.metadata.labels`. 
Config with these not matching will be rejected by the API.  

不要使用其它方式去创建Label符合DaemonSet Selector的Pod，这样会导致DaemonSet会认为是由它创建的。

## Running Pods on Only Some Nodes
If you specify a `.spec.template.spec.nodeSelector`, then the DaemonSet controller will create Pods on nodes 
which match that node selector.   

Likewise if you specify a `.spec.template.spec.affinity`, then DaemonSet controller will create Pods on nodes 
which match that node affinity. If you do not specify either, then the DaemonSet controller will create Pods on all nodes.  

## How Daemon Pods are Scheduled
Normally, the machine that a Pod runs on is selected by the Kubernetes scheduler.   
However, Pods created by the DaemonSet controller have the machine already selected
（可以通过指定`.spec.nodeName`来选择machine，使Pod忽略scheduler的调度），因此：  
  - DaemonSet controller并不会遵守node节点的`unschedulable`配置（即使你配置了，也还是会调度）
  - The DaemonSet controller can make Pods even when the scheduler has not been started, which can help cluster bootstrap.  

Daemon Pods do respect taints and tolerations，but they are **created with NoExecute tolerations** 
for the following taints with no tolerationSeconds:  
  - `node.kubernetes.io/not-ready`
  - `node.alpha.kubernetes.io/unreachable`

============ TODO after learn taints and tolerations================

## Communicating with Daemon Pods
Some possible patterns for communicating with Pods in a DaemonSet are:  
### Push
Pods in the DaemonSet are configured to send updates to another service, such as a stats database. They do not have clients.  
### NodeIP and Known Port
通过hostPort和nodeIP通信。  
### DNS
Create a headless service with the same pod selector, and then discover DaemonSets 
using the endpoints resource or retrieve multiple A records from DNS.  
### Service
Create a service with the same Pod selector, and use the service to reach a daemon on a random node. (No way to reach specific node.)  

## Updating a DaemonSet
如果节点的label发生变化，DaemonSet会立即添加新Pod到匹配的节点，或删除不匹配节点上的Pod.  

您可以修改DaemonSet创建的Pod，但是，Pod不允许更新所有字段（不能完全更新）。
此外，DaemonSet控制器将在下一次创建节点（即使具有相同名称）时使用原始模板。  

使用带有`--cascade-false`选项的`kubectl delete`来删除DaemonSet，这样的话，所有DaemonSet创建的Pod将会被留在node上。  

创建带有不同template的DaemonSet，它会识别出所有matching label的Pod；虽然新旧Pod template不一样，但DaemonSet并不会修改或者删除这些Pod！
你必须强制删除这些Pod或者node，以使DaemonSet使用新的template创建Pod！

## rolling update
In Kubernetes version 1.6 and later, you can perform a rolling update on a DaemonSet.  

## DaemonSet的替代品
### Init Scripts
可以通过init, upstartd or systemd运行daemon processes来替代DaemonSet。但使用DaemonSet有几个优点：  
  - 可以采用与application监控和管理log相同的方法来监控和管理daemons的日志  
  - Same config language and tools (e.g. Pod templates, kubectl) for daemons and applications.  
  - Running daemons in containers with resource limits increases isolation between daemons from app containers. 
  However, this can also be accomplished by running the daemons in a container but not in a Pod (e.g. start directly via Docker)

### Bare Pods
It is possible to create Pods directly which specify a particular node to run on. 
However, a DaemonSet replaces Pods that are deleted or terminated for any reason, 
such as in the case of node failure or disruptive node maintenance, such as a kernel upgrade. 

For this reason, you should use a DaemonSet rather than creating individual Pods.  

### Static Pods
Unlike DaemonSet, static Pods cannot be managed with kubectl or other Kubernetes API clients. 
Static Pods do not depend on the apiserver, making them useful in cluster bootstrapping cases. 
Also, static Pods may be deprecated in the future!

### Deployments
Use a DaemonSet when it is important that a copy of a Pod always run on all or certain hosts, 
and when it needs to start before other Pods.  
