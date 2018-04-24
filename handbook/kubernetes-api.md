# kubernetes api

kubernetes 是一个管理跨主机容器化的开源系统，实现了包括应用部署、高可用管理和弹性伸缩等一系列基础功能，
**并封装成为一套完整、简单易用的 RESTful API 对外提供服务。**  

## [Kubernetes API Overview](https://kubernetes.io/docs/reference/api-overview/)

k8s集群组件之间的所有操作和通信以及外部用户命令都是由apiservice处理的REST API调用。所以，Kubernetes平台中的所有内容都被视为API对象。
API对象定义在[这里](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.10/)。

kubectl和kubeadm都是 kubernetes api 的命令行工具。

### API versioning

Kubernetes支持多个API版本，通过不同的路径表示，例如：`/api/v1` or `/apis/extensions/v1beta1`.  

api分为三个等级：Alpha, Beta and Stable，分别对应：v1alpha1, v1beta1, v1.  

### API groups

通过不同的路径来表示不同的 API version，为了更容易地扩展Kubernetes API，实现了
[API groups](https://github.com/kubernetes/community/blob/master/contributors/design-proposals/api-machinery/api-group.md)。  

- core group /api/v1 对应 apiVersion: v1  
- named groups are at REST path /apis/$GROUP_NAME/$VERSION 对应 apiVersion: apiVersion: $GROUP_NAME/$VERSION  

还有两种方式来定义 [custom resources](https://kubernetes.io/docs/concepts/api-extension/custom-resources/):  

- [CustomResourceDefinition](https://kubernetes.io/docs/tasks/access-kubernetes-api/extend-api-custom-resource-definitions/) 
is for users with very basic CRUD needs.  
- Coming soon: users needing the full set of Kubernetes API semantics can implement their own apiserver and use the 
[aggregator](https://github.com/kubernetes/community/blob/master/contributors/design-proposals/api-machinery/aggregated-api-servers.md)
to make it seamless for clients.  

### 使能 API groups

They can be enabled or disabled by setting --runtime-config on apiserver. 例如：  
```
--runtime-config=batch/v1=false
```

Enabling or disabling groups or resources requires restarting apiserver and controller-manager to pick up the --runtime-config changes.

### Enabling resources in the groups

DaemonSets, Deployments, HorizontalPodAutoscalers, Ingress, Jobs and ReplicaSets are enabled by default. 
Other extensions resources can be enabled by setting --runtime-config on apiserver. 

--runtime-config accepts comma separated values. For example: to disable deployments and ingress, set  
```
--runtime-config=extensions/v1beta1/deployments=false,extensions/v1beta1/ingress=false
```

## [API Client Libraries](https://kubernetes.io/docs/reference/client-libraries/)

client libraries 分为两类：  
- Officially-supported Kubernetes client libraries  
- Community-maintained client libraries  

## kubernetes API 访问控制

用户使用 kubectl, client libraries [访问 kubernetes API](https://kubernetes.io/docs/tasks/access-application-cluster/access-cluster/)，
当然也可以直接通过REST请求。用户和Kubernetes Service Accounts 都可以被授权进行API访问，当请求到达API时，它经历了几个阶段，如下图所示：  

![access-control-overview](https://github.com/fffffreedom/Pictures/blob/master/handbook/access-control-overview.svg)

### API Server Ports and IPs

默认情况下，Kubernetes apiserver 在2个端口上提供HTTP服务：  
- Localhost Port: 用作测试，默认为8080端口，不会进行authentication and authorization！  
- Secure Port: 尽可能使用，默认为6443端口，**会进行authentication and authorization！**  

## 参考资料

[Kubernetes API Overview](https://kubernetes.io/docs/reference/api-overview/)  
[Controlling Access to the Kubernetes API](https://kubernetes.io/docs/admin/accessing-the-api/)  
