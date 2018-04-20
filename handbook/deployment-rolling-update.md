# deployment滚动更新

## 如何触发deployment进行滚动更新？

A Deployment’s rollout is triggered if and only if the Deployment’s pod template (that is, .spec.template) is changed, 
for example if the labels or container images of the template are updated. Other updates, such as scaling the Deployment, 
do not trigger a rollout.

Deployment的滚动更新何时被触发？
Deployment滚动更新在pod的template (that is, .spec.template)发生改变时被触发。例如，template的labels或者image被更新。
其它的更新，如扩缩容，是不能触发滚动更新的！

## 怎么触发？

- 直接更新image  
```
kubectl set image deployment/nginx-deployment nginx=nginx:1.9.1 --record
```

- kubectl edit，并保存
```
kubectl edit deployment/nginx-deployment
```

- 直接编辑deployment的yaml文件，再kubectl apply
```
kubectl apply -f nginx-deployment.yaml
```

## 常用操作

- 查看滚动更新的状态  
```
kubectl rollout status deployments nginx-deployment
```

- 暂停和恢复更新  
```
kubectl rollout pause deployment/nginx-deployment
kubectl rollout resume deploy/nginx-deployment
```

## rolling update 相关配置参数

```
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: frontend
spec:
  minReadySeconds: 5
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 3
      maxUnavailable: 2
```

- .spec.minReadySeconds  
新创建的Pod状态为Ready持续的时间至少为.spec.minReadySeconds才认为Pod Available(Ready)。默认为0。  

- .spec.strategy.type  
升级的策略，包括 `RollingUpdate` 和 `Recreate`，默认为前者。  

- .spec.strategy.rollingUpdate.maxUnavailable  
指定在升级过程中，一次Unavailable pod的最大数，值可以为整数或者百分比，默认值为25%。 
The value cannot be 0 if maxSurge is 0.  
确保在升级时，不可用的Pod数不超过该值。  

- .spec.strategy.rollingUpdate.maxSurge  
指定升级时，一次可以创建Pod数目或者比例，值可以为整数或者百分比，默认值为25%。  
The value cannot be 0 if MaxUnavailable is 0.  
计算时向上取整。  
确保在升级时，可用的Pod数不超过该值。  

**根据上面两个参数配置，在滚动升级的过程中，available的pod数不小于 `desired pods number - maxUnavailable`，
且不大于 `desired pods number + maxSurge`。**

## 滚动更新过程

## 理解rollout pause和resume

或许很多人至今还会这么觉得：整个滚动更新的过程中，一旦用户执行了`kubectl rollout pause`后，正在执行的滚动流程就会立刻停止，
然后用户执行`kubectl rollout resume`就会继续未完成的滚动更新。那你就大错特错了！

`kubectl rollout pause`只会用来停止触发下一次rollout。什么意思呢？ 上面描述的这个场景，正在执行的滚动历程是不会停下来的，
而是会继续正常的进行滚动，直到完成。等下一次，用户再次触发rollout时，Deployment就不会真的去启动执行滚动更新了，
而是等待用户执行了`kubectl rollout resume`，流程才会真正启动执行。即pasue是用来禁止触发rollout的？（try it）  

kubernetes官网：  
You can pause a Deployment before triggering one or more updates and then resume it. 
This will allow you to apply multiple fixes in between pausing and resuming without triggering unnecessary rollouts.

## --record和rollback（回滚）

默认情况下，所有通过kubectl xxxx –record都会被kubernetes记录到etcd进行持久化，这无疑会占用资源，
最重要的是，时间久了，当你kubectl get rs时，会有成百上千的垃圾RS返回给你，那时你可能就眼花缭乱了。  

上生产时，我们最好通过设置Deployment的`.spec.revisionHistoryLimit`来限制最大保留的revision number，比如15个版本，
回滚的时候一般只会回滚到最近的几个版本就足够了。

执行下面的命令，可以返回某个Deployment的所有record记录：  
```
$ kubectl rollout history deployment/nginx-deployment
deployments "nginx-deployment"
REVISION    CHANGE-CAUSE
1           kubectl create -f docs/user-guide/nginx-deployment.yaml --record
2           kubectl set image deployment/nginx-deployment nginx=nginx:1.9.1
3           kubectl set image deployment/nginx-deployment nginx=nginx:1.91
```
然后执行rollout undo命令就可以回滚到to-revision指定的版本。  
```
kubectl rollout undo deployment/nginx-deployment --to-revision=2
deployment "nginx-deployment" rolled back
```
其实rollout history中记录的revision都和ReplicaSets一一对应。如果手动delete某个ReplicaSet，对应的rollout history就会被删除，
也就是还说你无法回滚到这个revison了。  
rollout history和ReplicaSet的对应关系，可以在kubectl describe rs $RSNAME返回的revision字段中得到，
这里的revision就对应着roolout history返回的revison。  

## 回滚是如何进行的？

使用命令`kubectl get rs -w`来观察这个过程，同样发现，回滚的时候也是按照滚动的机制进行的，同样要遵守maxSurge和maxUnavailable的约束，
并不是一次性将所有的Pods删除，然后再一次性创建新的Pods。

## 参考资料

[kubernetes deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)
[聊聊你可能误解的Kubernetes Deployment滚动更新机制](https://blog.csdn.net/WaltonWang/article/details/77461697?locationNum=5&fps=1)

