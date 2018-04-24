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

node affinitry包含两种类型：  
- requiredDuringSchedulingIgnoredDuringExecution: 硬性(hard)要求  
- preferredDuringSchedulingIgnoredDuringExecution: 建议性(soft)要求  

两种类型的后缀都是 `IgnoredDuringExecution`，其含义是：  
if labels on a node change at runtime such that the affinity rules on a pod are no longer met, 
the pod will still continue to run on the node.

以后会实现：requiredDuringSchedulingRequiredDuringExecution

### rules

```
apiVersion: v1
kind: Pod
metadata:
  name: with-node-affinity
spec:
  affinity:
    nodeAffinity: <== 并不会有anti-nodeAffinity,而是由opertator来决定nodeAffinity的类型,见下面
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: kubernetes.io/e2e-az-name
            operator: In
            values:
            - e2e-az1
            - e2e-az2
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 1
        preference:
          matchExpressions:
          - key: another-node-label-key
            operator: In
            values:
            - another-node-label-value
  containers:
  - name: with-node-affinity
    image: k8s.gcr.io/pause:2.0
```

You can see the operator In being used in the example. The new node affinity syntax supports the following operators:  
**In, NotIn, Exists, DoesNotExist, Gt, Lt.** 

**There is no explicit “node anti-affinity” concept, but NotIn and DoesNotExist give that behavior.**

If you specify both nodeSelector and nodeAffinity, both must be satisfied for the pod to be scheduled onto a candidate node.

If you specify multiple nodeSelectorTerms associated with nodeAffinity types, then the pod can be scheduled onto a node if one of the nodeSelectorTerms is satisfied.  
如果为nodeAffinity指定了多个nodeSelectorTerms，则pod只要满足了nodeSelectorTerms中的一个，就可以被调度到相应node.  

If you specify multiple matchExpressions associated with nodeSelectorTerms, then the pod can be scheduled onto a node only if all matchExpressions can be satisfied.  
如果一个nodeSelectorTerms包含多个matchExpressions，则pod需要满足所有，才能被调度到相应node.  

**affinity selection works only at the time of scheduling the pod.**

The `weight` field in `preferredDuringSchedulingIgnoredDuringExecution` is in the range 1-100. For each node that meets all of the scheduling requirements (resource request, RequiredDuringScheduling affinity expressions, etc.), the scheduler will compute a sum by iterating through the elements of this field and adding “weight” to the sum if the node matches the corresponding MatchExpressions. This score is then combined with the scores of other priority functions for the node. The node(s) with the highest total score are the most preferred.

## Inter-pod affinity and anti-affinity (beta feature)

Inter-pod affinity and anti-affinity were introduced in Kubernetes 1.4.

Inter-pod affinity and anti-affinity allow you to constrain which nodes your pod is eligible to be scheduled **based on labels on pods that are already running on the node rather than based on labels on nodes.** The rules are of the form “this pod should (or, in the case of anti-affinity, should not) run in an X if that X is already running one or more pods that meet rule Y”. Y is expressed as a LabelSelector with an associated list of namespaces (or “all” namespaces); unlike nodes, because pods are namespaced (and therefore the labels on pods are implicitly namespaced), a label selector over pod labels must specify which namespaces the selector should apply to. Conceptually X is a topology domain like node, rack, cloud provider zone, cloud provider region, etc. 

You express it using a **topologyKey** which is the key for the node label that the system uses to denote such a topology domain, e.g. see the label keys listed above in the section Interlude: built-in node labels.

Note: Inter-pod affinity and anti-affinity require substantial amount of processing which can slow down scheduling in large clusters significantly. We do not recommend using them in clusters larger than several hundred nodes.  
比较消耗性能，大集群不不建议使用！  

pod affinitry and anti-affinity 包含两种类型： 
- requiredDuringSchedulingIgnoredDuringExecution: 硬性(hard)要求  
- preferredDuringSchedulingIgnoredDuringExecution: 建议性(soft)要求  

The legal operators for pod affinity and anti-affinity are **In, NotIn, Exists, DoesNotExist.**

labelSelector and topologyKey 的取值是有要求，参见官方文档！

## 实际用例

### Always co-located in the same node

### Never co-located in the same node

## 参考资料

[Assigning Pods to Nodes](https://kubernetes.io/docs/concepts/configuration/assign-pod-node/)
