# Rolling Update
The preferred way to create a replicated application is to use a **Deployment**.  
官网资料  
https://tachingchen.com/tw/blog/kubernetes-rolling-update-with-deployment/  
透過 KUBERNETES DEPLOYMENTS 實現滾動升級  
https://kubernetes.io/docs/concepts/workloads/controllers/deployment/  

## Creating a Deployment
cat nginx-deployment.yaml
```
# 注意apiVersion, 使用kubectl api-versions查看当前支持api
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.7.9
        ports:
        - containerPort: 80
```
```
[root@k8s-master deployment]# kubectl create -f nginx-deployment.yaml
[root@k8s-master deployment]# kubectl get deploy
NAME               DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   3         3         3            3           25s 3         3            3           25s
[root@k8s-master deployment]# kubectl get rs --show-labels
NAME                          DESIRED   CURRENT   READY     AGE       LABELS
nginx-deployment-431080787    3         3         3         26m       app=nginx,pod-template-hash=431080787
[root@k8s-master deployment]# kubectl get pods --show-labels
NAME                               READY     STATUS    RESTARTS   AGE       LABELS
nginx-deployment-431080787-kh0xs   1/1       Running   0          49s       app=nginx,pod-template-hash=431080787
nginx-deployment-431080787-m9b6b   1/1       Running   0          49s       app=nginx,pod-template-hash=431080787
nginx-deployment-431080787-sdmp5   1/1       Running   0          49s       app=nginx,pod-template-hash=431080787
```
- pod的输出输出格式  
`[DEPLOYMENT-NAME]-[POD-TEMPLATE-HASH-VALUE]`  
The pod-template-hash label is added by the Deployment controller to every ReplicaSet that a Deployment creates or adopts. 
Do not change this label !!!  
## Updating a Deployment
- 更新Deployment的方法
可以使用多种方法更新一个Deployment：  
```
kubectl set image deployment/nginx-deployment nginx=nginx:1.9.1  
kubectl edit deployment/nginx-deployment  
```
- 查看升级的进度及状态  
```
[root@k8s-master deployment]# kubectl rollout status deployment/nginx-deployment
Waiting for rollout to finish: 2 out of 3 new replicas have been updated...
deployment "nginx-deployment" successfully rolled out
[root@k8s-master deployment]# kubectl get rs
NAME                          DESIRED   CURRENT   READY     AGE
nginx-deployment-2078889897   3         3         3         17m
nginx-deployment-431080787    0         0         0         1h
可以看到，旧的rs（431080787）已经全部被替换成新的rs（2078889897）了
[root@k8s-master deployment]# kubectl get po
NAME                                READY     STATUS    RESTARTS   AGE
nginx-deployment-2078889897-5gw3r   1/1       Running   0          17m
nginx-deployment-2078889897-gf40q   1/1       Running   0          17m
nginx-deployment-2078889897-w71nc   1/1       Running   0          17m
```
- 查看升级过程  
```
# kubectl describe deployments
  FirstSeen	LastSeen	Count	From			SubObjectPath	Type		Reason			Message
  ---------	--------	-----	----			-------------	--------	------			-------
  28m		28m		1	deployment-controller			Normal		ScalingReplicaSet	Scaled up replica set nginx-deployment-2078889897 to 1
  28m		28m		1	deployment-controller			Normal		ScalingReplicaSet	Scaled down replica set nginx-deployment-431080787 to 2
  28m		28m		1	deployment-controller			Normal		ScalingReplicaSet	Scaled up replica set nginx-deployment-2078889897 to 2
  28m		28m		1	deployment-controller			Normal		ScalingReplicaSet	Scaled down replica set nginx-deployment-431080787 to 1
  28m		28m		1	deployment-controller			Normal		ScalingReplicaSet	Scaled up replica set nginx-deployment-2078889897 to 3
  28m		28m		1	deployment-controller			Normal		ScalingReplicaSet	Scaled down replica set nginx-deployment-431080787 to 0
```
## Checking Rollout History of a Deployment
```
$ kubectl rollout history deployment/nginx-deployment
deployments "nginx-deployment"
REVISION    CHANGE-CAUSE
1           kubectl create -f docs/user-guide/nginx-deployment.yaml --record
2           kubectl set image deployment/nginx-deployment nginx=nginx:1.9.1
3           kubectl set image deployment/nginx-deployment nginx=nginx:1.91
这里我显示出来的都为none，因为命令没有指定选项`--record`
[root@k8s-master deployment]# kubectl rollout history deployment/nginx-deployment
deployments "nginx-deployment"
REVISION	CHANGE-CAUSE
1		<none>
2		<none>
```
## Rolling Back to a Previous Revision
回滚可以回滚到上一版本，也可以回滚到指定版本!  
```
# 回滚到上一版本
kubectl rollout undo deployment/nginx-deployment
# 回滚到指定版本
kubectl rollout undo deployment/nginx-deployment --to-revision=2
```
回滚具体过程：  
```
[root@k8s-master deployment]# kubectl rollout undo deployment/nginx-deployment
deployment "nginx-deployment" rolled back
[root@k8s-master deployment]# kubectl get rs --show-labels
NAME                          DESIRED   CURRENT   READY     AGE       LABELS
nginx-deployment-2078889897   1         1         1         41m       app=nginx,pod-template-hash=2078889897
nginx-deployment-431080787    3         3         2         1h        app=nginx,pod-template-hash=431080787
[root@k8s-master deployment]# kubectl get rs --show-labels
NAME                          DESIRED   CURRENT   READY     AGE       LABELS
nginx-deployment-2078889897   0         0         0         41m       app=nginx,pod-template-hash=2078889897
nginx-deployment-431080787    3         3         3         1h        app=nginx,pod-template-hash=431080787
[root@k8s-master deployment]# kubectl get rs --show-labels
NAME                          DESIRED   CURRENT   READY     AGE       LABELS
nginx-deployment-2078889897   0         0         0         41m       app=nginx,pod-template-hash=2078889897
nginx-deployment-431080787    3         3         3         1h        app=nginx,pod-template-hash=431080787

[root@k8s-master deployment]# kubectl get po --show-labels
NAME                               READY     STATUS    RESTARTS   AGE       LABELS
nginx-deployment-431080787-0p20v   1/1       Running   0          4m        app=nginx,pod-template-hash=431080787
nginx-deployment-431080787-57xcr   1/1       Running   0          4m        app=nginx,pod-template-hash=431080787
nginx-deployment-431080787-bjtxz   1/1       Running   0          4m        app=nginx,pod-template-hash=431080787
```
## Scaling a Deployment
pod扩缩容：  
```
[root@k8s-master deployment]# kubectl scale deployment nginx-deployment --replicas=6
deployment "nginx-deployment" scaled
[root@k8s-master deployment]# kubectl get po --show-labels
NAME                               READY     STATUS    RESTARTS   AGE       LABELS
nginx-deployment-431080787-0p20v   1/1       Running   0          19m       app=nginx,pod-template-hash=431080787
nginx-deployment-431080787-1wv60   1/1       Running   0          3m        app=nginx,pod-template-hash=431080787
nginx-deployment-431080787-57xcr   1/1       Running   0          19m       app=nginx,pod-template-hash=431080787
nginx-deployment-431080787-bjtxz   1/1       Running   0          19m       app=nginx,pod-template-hash=431080787
nginx-deployment-431080787-l6wxt   1/1       Running   0          3m        app=nginx,pod-template-hash=431080787
nginx-deployment-431080787-pgphq   1/1       Running   0          3m        app=nginx,pod-template-hash=431080787
```
## Proportional scaling（比例缩放）
RollingUpdate Deployments支持同时运行多个版本的app。当你或者一个autoscaler对一个处于rollout（正在进行或暂停）的Deployment扩缩容时，
那么Deployment控制器将把缩放副本数（replicas）按比例地分配现有活动的ReplicaSets（ReplicaSets with Pods），以降低风险。  
说白了就是，当一个Deployment正在执行时，你或者autoscaler又要去更新pod的replica数，假定是扩容，从10扩到15，那这个时候，该怎么分配pod数？  
假如此时，deployment下的rs是这个状态：  
```
$ kubectl get rs
NAME                          DESIRED   CURRENT   READY     AGE
nginx-deployment-1989198191   5         5         0         9s
nginx-deployment-618515232    8         8         8         1m
```
在没有proportional scaling功能的情况下，5个增加的replica数会全部分配到新的rs，也即上面的nginx-deployment-1989198191；  
在有proportional scaling功能的情况下，5个增加的replica数会**按rs比例**分配到不同的rs上去；  
如果存在为0的rs，它不会分配replica数！  
按照上面的情况，大多数会分配到nginx-deployment-618515232，小部分会分配到nginx-deployment-1989198191；  
```
$ kubectl get deploy
NAME                 DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment     15        18        7            8           7m
$ kubectl get rs
NAME                          DESIRED   CURRENT   READY     AGE
nginx-deployment-1989198191   7         7         0         7m
nginx-deployment-618515232    11        11        11        7m
```
## Pausing and Resuming a Deployment
You can pause a Deployment before triggering one or more updates and then resume it. 
This will allow you to apply multiple fixes in between pausing and resuming without triggering unnecessary rollouts.  
意思是说，在升级之前，我们可以先暂停Deployment，做一系列的升级更新，然后再恢复Deployment；  
在做一系列的更新时，并不会触发Deployment工作，只有在resume后，才会开始进行更新！  
```
$ kubectl rollout pause deployment/nginx-deployment
$ kubectl set resources deployment nginx-deployment -c=nginx --limits=cpu=200m,memory=512Mi
```
## Deployment status
- Progressing Deployment  
Kubernetes marks a Deployment as progressing when one of the following tasks is performed:  
  - The Deployment creates a new ReplicaSet.
  - The Deployment is scaling up its newest ReplicaSet.
  - The Deployment is scaling down its older ReplicaSet(s).
  - New Pods become ready or available (ready for at least MinReadySeconds).
You can monitor the progress for a Deployment by using `kubectl rollout status`.  
- Complete Deployment  
Kubernetes marks a Deployment as complete when it has the following characteristics:  
  - All of the replicas associated with the Deployment have been updated to the latest version you’ve specified, 
  meaning any updates you’ve requested have been completed.
  - All of the replicas associated with the Deployment are available.
  - No old replicas for the Deployment are running.
You can check if a Deployment has completed by using `kubectl rollout status`. 
If the rollout completed successfully, `kubectl rollout status` returns a zero exit code.  
```
$ kubectl rollout status deploy/nginx-deployment
Waiting for rollout to finish: 2 of 3 updated replicas are available...
deployment "nginx" successfully rolled out
$ echo $?
0
```
- Failed Deployment
Your Deployment may get stuck trying to deploy its newest ReplicaSet without ever completing. 
This can occur due to some of the following factors:  
  - Insufficient quota
  - Readiness probe failures
  - Image pull errors
  - Insufficient permissions
  - Limit ranges
  - Application runtime misconfiguration
One way you can detect this condition is to specify a deadline parameter in your Deployment spec: (spec.progressDeadlineSeconds). 
spec.progressDeadlineSeconds denotes the number of seconds the Deployment controller waits before indicating (in the Deployment status) 
that the Deployment progress has stalled.  
The following kubectl command sets the spec with progressDeadlineSeconds to make the controller report lack of progress 
for a Deployment after 10 minutes:  
```
$ kubectl patch deployment/nginx-deployment -p '{"spec":{"progressDeadlineSeconds":600}}'
deployment "nginx-deployment" patched
```
Once the deadline has been exceeded, the Deployment controller adds a DeploymentCondition with the following attributes 
to the Deployment’s status.conditions:  
  - Type=Progressing
  - Status=False
  - Reason=ProgressDeadlineExceeded
run `kubectl get deployment nginx-deployment -o yaml` to see the deployment status.  
- Operating on a failed deployment
All actions that apply to a complete Deployment also apply to a failed Deployment. 
You can scale it up/down, roll back to a previous revision, 
or even pause it if you need to apply multiple tweaks in the Deployment pod template.  
## Clean up Policy
You can set `.spec.revisionHistoryLimit` field in a Deployment to specify how many old ReplicaSets 
for this Deployment you want to retain. The rest will be garbage-collected in the background. 
By default, all revision history will be kept. In a future version, it will default to switch to 2.  
> Note: Explicitly setting this field to 0, will result in cleaning up all the history 
of your Deployment thus that Deployment will not be able to roll back. (设置为0，将会导致无法回滚！) 
## Writing a Deployment Spec
> https://kubernetes.io/docs/concepts/workloads/controllers/deployment/  




