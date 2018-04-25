# 理解Kubernetes object(对象)

kubernetes中的对象是一些持久化的实体，kubernetes使用这些对象实体来表示群集的状态，包括：

- 集群中哪些node上运行了哪些容器化应用；  
- 应用的资源是否满足使用；  
- 应用的执行策略，例如重启策略、更新策略、容错策略等。  

kubernetes的对象是一种意图（期望）的记录，kubernetes会始终保持预期创建的对象存在和保持集群运行在预期的状态下。

操作kubernetes对象（增删改查）需要通过kubernetes API，一般有以下几种方式：  
- kubectl命令工具  
- Client library的方式，例如 client-go  

## Objects

以下列举的内容都是 kubernetes 中的 Object，这些对象都可以在 yaml 文件中作为一种 API 类型来配置。

```
Pod
Node
Namespace
Service
Volume
PersistentVolume
Deployment
Secret
StatefulSet
DaemonSet
ServiceAccount
ReplicationController
ReplicaSet
Job
CronJob
SecurityContext
ResourceQuota
LimitRange
HorizontalPodAutoscaling
Ingress
ConfigMap
Label
ThirdPartyResources
```

我将它们简单的分类为以下几种资源对象：

|类别|名称|
|:--|:--|
|资源对象|Pod、ReplicaSet、ReplicationController、Deployment、StatefulSet、DaemonSet、Job、CronJob、HorizontalPodAutoscaling|
|配置对象|Node、Namespace、Service、Secret、ConfigMap、Ingress、Label、ThirdPartyResource、 ServiceAccount|
|存储对象|Volume、Persistent Volume|
|策略对象|SecurityContext、ResourceQuota、LimitRange|

## object's Spec and Status

每个kubernetes对象的结构描述都包含spec和status两个部分：  
- spec：该内容由用户提供，描述用户期望的对象特征及集群状态。  
- status：该内容由kubernetes集群提供和更新，描述kubernetes对象的实时状态。  

任何时候，kubernetes都会控制集群的实时状态status与用户的预期状态spec一致。

例如：当你定义Deployment的描述文件，指定集群中运行3个实例，那么kubernetes会始终保持集群中运行3个实例，
如果任何实例挂掉，kubernetes会自动重建新的实例来保持集群中始终运行用户预期的3个实例。

## 对象描述文件

当你要创建一个kubernetes对象的时候，需要提供该对象的描述信息spec，来描述你的对象在kubernetes中的预期状态。

一般使用kubernetes API来创建kubernetes对象，其中spec信息可以以JSON的形式存放在request body中，也可以以.yaml文件的形式通过kubectl工具创建。

例如，以下为Deployment对象对应的yaml文件：  

```
apiVersion: apps/v1beta2 # for versions before 1.8.0 use apps/v1beta1
kind: Deployment
metadata:
  name: nginx-deployment
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

执行kubectl create的命令：  
```
#create command
kubectl create -f https://k8s.io/docs/user-guide/nginx-deployment.yaml --record
#output
deployment "nginx-deployment" created
```

## 必须字段

在对象描述文件.yaml中，必须包含以下字段。

- apiVersion：kubernetes API的版本。  
- kind：kubernetes对象的类型。  
- metadata：唯一标识该对象的元数据，包括name，UID，可选的namespace。  
- spec：标识对象的详细信息，不同对象的spec的格式不同，可以嵌套其他对象的字段。  

## 参考资料

[Objects](https://jimmysong.io/kubernetes-handbook/concepts/objects.html)  
[理解Kubernetes对象](https://blog.csdn.net/huwh_/article/details/79068190)  
[Understanding Kubernetes Objects](https://kubernetes.io/docs/concepts/overview/working-with-objects/kubernetes-objects/)  
