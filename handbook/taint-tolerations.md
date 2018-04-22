# taint and toleration

Node affinity（节点亲合性）是吸引Pod调度到Node的一种属性，可以硬性要求或者是建议性要求；而taint与其相反，是阻止pod调度到相应node的一种机制！

taint和toleration是污点和容忍的含义，当给node打上taint（就是一个键值对及影响），只有具有相应toleration配置的pod，才可能调度到这些Node。

## taint

### 组成

taint由三部分组成，格式为`key=value:taint effect`，key和value是一个键值对，taint effect是具体的影响：

 - key  
 - value  
 - taint effect  
  - NoSchedule：表示pod不能调度到node  
  - NoExecute：表示pod不能在node上运行  

### 打污点和去除污点

以下命令给node1节点打上了一个taint，这意味着，只有pod有key为type，值为ssd的pod能调度到node1节点，
```
kubectl taint nodes node1 type=ssd:NoSchedule
kubectl taint nodes node1 type=ssd:NoSchedule-
```

## toleration

### 组成

toleration由四部分组成（并不是都必须的），`key, operator, value, effect`，见如下两个例子：

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
```

上面两个例子表明了说明了如何去配置pod的toleration，且并不是所有字段都是必须的！

## 参考资料

[Taints and Tolerations](https://kubernetes.io/docs/concepts/configuration/taint-and-toleration/)
