# taint and toleration

Node affinity（节点亲合性）是吸引Pod调度到Node的一种属性，可以硬性要求或者是建议性要求；而taint与其相反，是阻止pod调度到相应node的一种机制！

taint和toleration是污点和容忍的含义，当给node打上taint（就是一个键值对及影响），只有具有相应toleration配置的pod，才可能调度到这些Node。

## taint

### 组成

我们一般在通过kubectl给Node打taint由，格式为：`key=value:taint effect`；其中key和value是一个键值对，taint effect是具体的影响：

 - key  
 - value  
 - taint effect  

### taint effect
- NoSchedule：表示pod不能调度到node  
- PreferNoSchedule：This is a “preference” or “soft” version of NoSchedule  
- NoExecute：表示pod不能在node上运行  

### 打污点和去除污点

以下命令给node1节点打上了一个taint，这意味着，只有pod有key为type，值为ssd的pod能调度到node1节点，
```
kubectl taint nodes node1 type=ssd:NoSchedule
kubectl taint nodes node1 type=ssd:NoSchedule-
```

当然也可以给node打多个taint，给pod配置多个toleration。

## toleration

### 组成

toleration目前有五个配置项（并不是都必须的），[Toleration v1 core](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.10/#toleration-v1-core)，`key, operator, value, effect, tolerationSeconds`，见如下两个例子：

```
tolerations:
- key: "key"
  operator: "Equal"
  value: "value"
  effect: "NoSchedule"

tolerations:
- key: "key"
  operator: "Exists"
  effect: "NoSchedule"
  
tolerations:
- key: "key"
  operator: "Equal"
  value: "value"
  effect: "NoExecute"
  tolerationSeconds: 3600  # only for NoExecute
```

上面两个例子表明了说明了如何去配置pod的toleration，且并不是所有字段都是必须的！

有两个特殊情况：

- An empty key with operator Exists matches all keys, values and effects which means this will tolerate everything.

```
tolerations:
- operator: "Exists"
```

- An empty effect matches all effects with key key.

```
tolerations:
- key: "key"
  operator: "Exists"
```

## taint and toleration 匹配

A toleration “matches” a taint if the keys are the same and the effects are the same, and:  
- the operator is Exists (in which case no value should be specified), or  
- the operator is Equal and the values are equal  

Operator defaults to Equal if not specified.  

## taint and toleratio 规则

- if there is at least one un-ignored taint with effect NoSchedule then Kubernetes will not schedule the pod onto that node  
如果node存在至少一个不可被忽略的NoSchedule taint，没有配置相应toleration的pod将无法被调度到该node；  

- if there is no un-ignored taint with effect NoSchedule but there is at least one un-ignored taint with effect PreferNoSchedule then Kubernetes will try to not schedule the pod onto the node  
如果只存在PreferNoSchedule taint，没有配置相应toleration的pod将尽量不被调度到该node；  

- if there is at least one un-ignored taint with effect NoExecute then the pod will be evicted from the node (if it is already running on the node), and will not be scheduled onto the node (if it is not yet running on the node).  
如果node存在至少一个不可被忽略的NoExecute taint，没有配置相应toleration的pod将无法在该node上运行；  

## Example Use Cases

### Dedicated Nodes（专用节点）

某些node有特殊的用途，比如要给特定的小组使用，就可以给这些node打上特定的taint，防止小组的pod调度到该node：  
```
kubectl taint nodes nodename dedicated=groupName:NoSchedule
```

通过给相关pod配置好toleration，或者自己写一个[admission controller](https://kubernetes.io/docs/admin/admission-controllers/)，使得相关的pod，可以调度到专用的node。

### Nodes with Special Hardware

可以给具有特殊硬件的node，比如GPU，打上taint，以提供给需要使用GPU的pod：  
```
kubectl taint nodes nodename special=true:NoSchedule
```

目前我们需要手动给pod配置toleration，有没有什么机制可以自动添加toleration配置？

it is recommended to use [Extended Resources](https://kubernetes.io/docs/concepts/configuration/manage-compute-resources-container/#extended-resources) to represent the special hardware, taint your special hardware nodes with the extended resource name and run the [ExtendedResourceToleration](https://kubernetes.io/docs/admin/admission-controllers/#extendedresourcetoleration) admission controller. Now, because the nodes are tainted, no pods without the toleration will schedule on them. But when you submit a pod that requests the extended resource, the ExtendedResourceToleration admission controller will automatically add the correct toleration to the pod and that pod will schedule on the special hardware nodes. This will make sure that these special hardware nodes are dedicated for pods requesting such hardware and you don’t have to manually add tolerations to your pods.

### Taint based Evictions (alpha feature)

当node有问题时，可以配置每个pod被驱逐时的eviction behavior。

## Taint based Evictions

NoExecute的taint effect会影响已经运行在node上的pod，

- pods that do not tolerate the taint are evicted immediately  
- pods that tolerate the taint without specifying `tolerationSeconds` in their toleration specification remain bound forever  
- pods that tolerate the taint with a specified `tolerationSeconds` remain bound for the specified amount of time  

- 不能容忍污点的pod会被立即驱逐  
- 能容忍污点的pod,并且没有指定`tolerationSeconds`配置,pod将会一直运行在该node  
- 能容忍污点的pod,但指定了`tolerationSeconds`配置,pod将会在node上继续运行`tolerationSeconds`秒后,被驱逐出node  

## built-in taints

Kubernetes 1.6 has alpha support for representing node problems. In other words, **the node controller automatically taints a node when certain condition is true**. The built-in taints currently include:  

- node.kubernetes.io/not-ready: Node is not ready. This corresponds to the NodeCondition Ready being “False”.  
- node.alpha.kubernetes.io/unreachable: Node is unreachable from the node controller. This corresponds to the NodeCondition Ready being “Unknown”.  
- node.kubernetes.io/out-of-disk: Node becomes out of disk.  
- node.kubernetes.io/memory-pressure: Node has memory pressure.  
- node.kubernetes.io/disk-pressure: Node has disk pressure.  
- node.kubernetes.io/network-unavailable: Node’s network is unavailable.  
- node.cloudprovider.kubernetes.io/uninitialized: When kubelet is started with “external” cloud provider, it sets this taint on a node to mark it as unusable. When a controller from the cloud-controller-manager initializes this node, kubelet removes this taint.  

When the TaintBasedEvictions alpha feature is enabled (you can do this by including TaintBasedEvictions=true in --feature-gates for Kubernetes controller manager, such as --feature-gates=FooBar=true,TaintBasedEvictions=true), the taints are automatically added by the NodeController (or kubelet) and the normal logic for evicting pods from nodes based on the Ready NodeCondition is disabled. 

**Note: To maintain the existing rate limiting behavior of pod evictions due to node problems, the system actually adds the taints in a rate-limited way. This prevents massive pod evictions in scenarios such as the master becoming partitioned from the nodes.**

This alpha feature, in combination with tolerationSeconds, allows a pod to specify how long it should stay bound to a node that has one or both of these problems.

For example, an application with a lot of local state might want to stay bound to node for a long time in the event of network partition, in the hope that the partition will recover and thus the pod eviction can be avoided. The toleration the pod would use in that case would look like

```
tolerations:
- key: "node.alpha.kubernetes.io/unreachable"
  operator: "Exists"
  effect: "NoExecute"
  tolerationSeconds: 6000
```

Note that Kubernetes automatically adds a toleration for node.kubernetes.io/not-ready with tolerationSeconds=300 unless the pod configuration provided by the user already has a toleration for node.kubernetes.io/not-ready. Likewise it adds a toleration for node.alpha.kubernetes.io/unreachable with tolerationSeconds=300 unless the pod configuration provided by the user already has a toleration for node.alpha.kubernetes.io/unreachable.

These automatically-added tolerations ensure that the default pod behavior of remaining bound for 5 minutes after one of these problems is detected is maintained. The two default tolerations are added by the [DefaultTolerationSeconds admission controller](https://git.k8s.io/kubernetes/plugin/pkg/admission/defaulttolerationseconds).

DaemonSet pods are created with NoExecute tolerations for the following taints with no tolerationSeconds

- node.alpha.kubernetes.io/unreachable  
- node.kubernetes.io/not-ready  

This ensures that DaemonSet pods are never evicted due to these problems, which matches the behavior when this feature is disabled.

## Taint Nodes by Condition

Version 1.8 introduces an alpha feature that causes the node controller to create taints corresponding to Node conditions. When this feature is enabled (you can do this by including TaintNodesByCondition=true in the --feature-gates command line flag to the scheduler, such as --feature-gates=FooBar=true,**TaintNodesByCondition=true**), the scheduler does not check Node conditions; instead the scheduler checks taints. This assures that Node conditions don’t affect what’s scheduled onto the Node. The user can choose to ignore some of the Node’s problems (represented as Node conditions) by adding appropriate Pod tolerations.

To make sure that turning on this feature doesn’t break DaemonSets, starting in version 1.8, the DaemonSet controller automatically adds the following NoSchedule tolerations to all daemons:

- node.kubernetes.io/memory-pressure  
- node.kubernetes.io/disk-pressure  
- node.kubernetes.io/out-of-disk (only for critical pods)  

The above settings ensure backward compatibility, but we understand they may not fit all user’s needs, which is why cluster admin may choose to add arbitrary tolerations to DaemonSets.

## 参考资料

[Taints and Tolerations](https://kubernetes.io/docs/concepts/configuration/taint-and-toleration/)
