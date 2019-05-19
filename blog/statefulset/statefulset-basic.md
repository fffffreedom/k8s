# statefulset basics

## reference
https://kubernetes.io/docs/tutorials/stateful-application/basic-stateful-set/

## statefulset的pod管理策略（Pod Management Policy）
statefulset支持两种方式的pod管理策略，通过`.spec.podManagementPolicy`控制：

### OrderedReady  
默认方式，pod标识唯一，按顺序启动和删除

### Parallel  
pod标识唯一，pod可并行启动和删除

## statefulset升级
statefulset升级策略由`.spec.updateStrategy`指定，此功能可用于升级StatefulSet中Pod的容器镜像，资源请求和/或限制，标签和注释。  
statefulset升级支持：`RollingUpdate`和`OnDelete`
```
...
spec:
  updateStrategy:
    type: RollingUpdate or OnDelete
    rollingUpdate：
      partition: 3  # Partition indicates the ordinal at which the StatefulSet should be partitioned. Default value is 0.
...
```

### 灰度更新 or 金
这里的`partition`指定了不升级pod的个数，当pod的索引大于等于`partition`时，pod就会更新，可以用来做金丝雀部署。
举个例子：  
先给statefulset配置好参数，如上：`partition: 3`，假定一个名为web的statefulset的有3个pod：  
- 当`partition`为3，pod的模板更新了，pod不会更新  
- 当`partition`为2，pod的模板更新了，有一个pod会更新（web-2）  
