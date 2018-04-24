# Assigning Pods to Nodes

通过一些机制，我们能控制pod的调度：只能调度到一类nodes，或者更倾向于调度到一类nodes!

用很几种方法可以达到这一目的，且它们都依赖于[label selector](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/)。

在默认的情况下，scheduler会自动进行合理的安置，即把pod调度到集群中的各个node中（而不是全都在一个node上）；
但在一些特殊情况下，我们需要控制pod的调度策略，比如将需要使用ssd的pod调度到有ssd的node上，或者将两个紧密相关的pod调度到同一个node上。

## nodeSelector

nodeSelector是最基础的方法，通过给node打上label，并配置pod的`spec.nodeSelector`，使得pod调度到相应的node上；

### kubectl 命令

```
kubectl get nodes
kubectl label nodes <node-name> <label-key>=<label-value>
kubectl get nodes --show-labels
```
### pod spec

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    env: test
spec:
  containers:
  - name: nginx
    image: nginx
    imagePullPolicy: IfNotPresent
  nodeSelector: <== 
    disktype: ssd
```

### built-in node labels

node会被打上一些标准的labels，从Kubernetes v1.4开始，这些标签是：  
- kubernetes.io/hostname  
- failure-domain.beta.kubernetes.io/zone  
- failure-domain.beta.kubernetes.io/region  
- beta.kubernetes.io/instance-type  
- beta.kubernetes.io/os  
- beta.kubernetes.io/arch  

## Affinity and anti-affinity

nodeSelector provides a very simple way to constrain pods to nodes with particular labels. The affinity/anti-affinity feature, 
currently in beta, greatly expands the types of constraints you can express. The key enhancements are:  

- the language is more expressive (not just “AND of exact match”)  
- you can indicate that the rule is “soft”/”preference” rather than a hard requirement, so if the scheduler can’t satisfy it, 
the pod will still be scheduled  
- you can constrain against labels on other pods running on the node (or other topological domain), rather than against labels on 
the node itself, which allows rules about which pods can and cannot be co-located  

```
Affinity and anti-affinity
  |-- node: 基于node label
  |-- inter-pod: 基于pod label
```

nodeSelector continues to work as usual, but will eventually be deprecated, as node affinity can express everything that 
nodeSelector can express.  
nodeSelector 将会被node affinity取代!  

## Node affinity (beta feature)

node affinitry




## 参考资料

[Assigning Pods to Nodes](https://kubernetes.io/docs/concepts/configuration/assign-pod-node/)
