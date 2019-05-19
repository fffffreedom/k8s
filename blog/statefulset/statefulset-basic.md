# statefulset basics

## reference
https://kubernetes.io/docs/tutorials/stateful-application/basic-stateful-set/  
https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/  
https://my.oschina.net/jxcdwangtao/blog/1634309  

## statefulset介绍
statefulset中的pod拥有唯一的索引和稳定的网络标识（hostname），无论怎么调度，每个pod都有一个永久不变的ID。  
StatefulSets有如下特性：
- 稳定的、唯一的网络标识符
- 稳定的、持久的存储
- 有序的、优雅的部署和缩放
- 有序的、自动的滚动更新

### 稳定的、唯一的网络标识符
statefulset中的pod拥有唯一索引和网络标识符，网络标识符指的是什么呢？  
即为pod的hostname：`$(statefulset name)-$(ordinal)`，其中ordinal即为pod的索引；  
稳定体现在：不管pod怎么调度，pod的hostname都会保持不变。

**这里有个问题，pod的IP变不变呢？** 答案是会变的，那怎么固定的访问它？ 该pod的A record是不变的！

|Cluster Domain|Service (ns/name)|StatefulSet (ns/name)|StatefulSet Domain|Pod DNS|Pod Hostname|
|--|--|--|--|--|--|
|cluster.local|default/nginx|default/web|nginx.default.svc.cluster.local|web-{0..N-1}.nginx.default.svc.cluster.local|web-{0..N-1}|
|cluster.local|foo/nginx|foo/web|nginx.foo.svc.cluster.local|web-{0..N-1}.nginx.foo.svc.cluster.local|web-{0..N-1}|
|kube.local|foo/nginx|foo/web|nginx.foo.svc.kube.local|web-{0..N-1}.nginx.foo.svc.kube.local|web-{0..N-1}|

### 稳定的、持久的存储
- 当statefulset或statefulset的pod被删除，它相应的PVC不会被删除，必需手动去删除
- 当Pod发生re-schedule（其实是recreate）后，它所对应的PVC所Bound的PV仍然会自动的挂载到新的Pod中

### 有序的、优雅的部署和缩放
- 当部署有N个副本的StatefulSet应用时，严格按照index从0到N-1的递增顺序创建，当前Pod状态为Running and Ready时，才开始创建下一个
- 当删除有N个副本的StatefulSet应用时，严格按照index从N-1到0的递减顺序删除，当前Pod被完全删除时，才开始删除下一个
- 当扩容StatefulSet应用时，当前新增的Pod状态为Running and Ready时，才开始创建下一个
- 当缩容StatefulSet应用时，当前Pod被完全删除时，才开始删除下一个
- 注意StatefulSet的pod.Spec.TerminationGracePeriodSeconds不要设置为0

### 有序的、自动的滚动更新
- 当更新有N个副本的StatefulSet应用时，严格按照index从N-1到0的递减顺序更新，当前Pod状态为Running and Ready时，才开始更新下一个

## statefulset中的顺序
### 创建
默认情况下，statefulset创建时，pod被顺序创建，索引是从小到大，web-0, web-1, web-2, ...，
后者必需等前者创建完成后，才开始被创建，在此之前，后者处于pending状态。

### 删除
分两种情况：
- Non-Cascading Delete  
  删除statefulset，相应的pod不会被删除
- Cascading Delete  
  删除statefulset，相应的pod会被删除，pod被删除的顺序和创建时相反

## statefulset更新策略
statefulset更新策略由`.spec.updateStrategy`指定，此功能可用于更新StatefulSet中Pod的容器镜像，资源请求和/或限制，标签和注释。  
statefulset支持两种类型的更新策略：`RollingUpdate`和`OnDelete`  
```
...
spec:
  updateStrategy:
    type: RollingUpdate or OnDelete
    rollingUpdate：
      partition: 3  # Partition indicates the ordinal at which the StatefulSet should be partitioned. Default value is 0.
...
```

### 灰度发布 or 金丝雀发布
这里的`partition（分区）`指定了不更新pod的个数，相当于把pod分成两个区：
- pod索引大于等于`partition`的分区：template更新，pod就会更新  
- pod小于`partition`的分区：template更新，pod不会更新（可以Staging an Update）  
通过这个特性，可以用来灰度发布或金丝雀发布。举个例子：  
先给statefulset配置好参数，如上：`partition: 3`，假定一个名为web的statefulset的有3个pod：  
  - 当`partition`为3，pod的模板更新了，pod不会更新  
  - 当`partition`为2，pod的模板更新了，有一个pod会更新（web-2）  

### statefulset的pod管理策略（Pod Management Policy）
statefulset支持两种方式的pod管理策略，通过`.spec.podManagementPolicy`控制：
### OrderedReady
默认方式，pod标识唯一，按顺序启动和删除
### Parallel
pod标识唯一，pod可并行启动和删除

### 强制回滚（Forced Rollback）

## 限制

