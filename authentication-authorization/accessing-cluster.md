# 访问 Kubernetes 集群

> https://jimmysong.io/kubernetes-handbook/guide/access-kubernetes-cluster.html

根据用户部署和暴露服务的方式不同，有很多种方式可以用来访问 kubernetes 集群：  

- 最简单也是最直接的方式是使用 kubectl 命令；
- 其次可以使用 kubeconfig 文件来认证授权访问 API server；
- 通过各种 proxy 经过端口转发访问 kubernetes 集群中的服务；
- 使用 Ingress，在集群外访问 kubernetes 集群内的 service；

> https://jimmysong.io/kubernetes-handbook/guide/access-cluster.html  
> https://kubernetes.io/docs/tasks/access-application-cluster/access-cluster/  

## kubectl

为了访问集群，您需要知道集群的地址，并且需要有访问它的凭证。集群的信息保存的`kubeconfig`文件中，使用下面的命令检查 kubectl 已知的集群的地址和凭证：  
```
kubectl config view
```

## 直接访问 REST API

Kubectl handles locating and authenticating to the apiserver. If you want to directly access the REST API with an http client like curl or wget, or a browser, there are several ways to locate and authenticate:  

- Run kubectl in proxy mode.
  - Recommended approach.
  - Uses stored apiserver location.
  - Verifies identity of apiserver using self-signed cert. No MITM possible.
  - Authenticates to apiserver.
  - In future, may do intelligent client-side load-balancing and failover.

- Provide the location and credentials directly to the http client.
  - Alternate approach.
  - Works with some types of client code that are confused by using a proxy.
  - Need to import a root cert into your browser to protect against MITM.
  
### Using kubectl proxy
  
The following command runs kubectl in a mode where it acts as a reverse proxy. It handles locating the apiserver and authenticating. Run it like this:  
```
kubectl proxy --port=8080 &
```

Then you can explore the API with curl, wget, or a browser, replacing localhost with [::1] for IPv6, like so:  
```
$ curl http://localhost:8080/api/
{
  "versions": [
    "v1"
  ]
}
```

### Programmatic access to the API

Kubernetes officially supports Go and Python client libraries.  

> https://kubernetes.io/docs/reference/client-libraries/

#### Go client
#### Python client

### Accessing the API from a Pod

When accessing the API from a pod, locating and authenticating to the apiserver are somewhat different.

The recommended way to locate the apiserver within the pod is with the kubernetes DNS name, which resolves to a Service IP which in turn will be routed to an apiserver.  

The recommended way to authenticate to the apiserver is with a service account credential.  

```
/var/run/secrets/kubernetes.io/serviceaccount/token
/var/run/secrets/kubernetes.io/serviceaccount/ca.crt
/var/run/secrets/kubernetes.io/serviceaccount/namespace
```

From within a pod the recommended ways to connect to API are:  

- run kubectl proxy in a sidecar container in the pod, or as a background process within the container. This proxies the Kubernetes API to the localhost interface of the pod, so that other processes in any container of the pod can access it.  

- use the Go client library, and create a client using the rest.InClusterConfig() and kubernetes.NewForConfig() functions. They handle locating and authenticating to the apiserver.  
> https://github.com/kubernetes/client-go/blob/master/examples/in-cluster-client-configuration/main.go  

In each case, the credentials of the pod are used to communicate securely with the apiserver.  

## 访问集群中运行的 service

In Kubernetes, the nodes, pods and services all have their own IPs. In many cases, the node IPs, pod IPs, and some service IPs on a cluster will not be routable, so they will not be reachable from a machine outside the cluster, such as your desktop machine.  

### Ways to connect

You have several options for connecting to nodes, pods and services from outside the cluster:  

- Access services through public IPs.
  - Use a service with type NodePort or LoadBalancer to make the service reachable outside the cluster. See the services and kubectl expose documentation.  
  - Depending on your cluster environment, this may just expose the service to your corporate network, or it may expose it to the internet. Think about whether the service being exposed is secure. Does it do its own authentication?  
  - Place pods behind services. To access one specific pod from a set of replicas, such as for debugging, place a unique label on the pod and create a new service which selects this label.  
  - In most cases, it should not be necessary for application developer to directly access nodes via their nodeIPs.  

- 通过 Proxy 规则访问 service、node、pod
  - 在访问远程服务之前，请执行 apiserver 认证和授权。 如果服务不够安全，无法暴露给互联网，或者为了访问节点 IP 上的端口或进行调试，请使用这种方式。
  - 代理可能会导致某些 Web 应用程序出现问题。
  - 仅适用于 HTTP/HTTPS。
  - `https://kubernetes.io/docs/tasks/access-application-cluster/access-cluster/#manually-constructing-apiserver-proxy-urls`
  
- 在集群内访问 node 和 pod
  - 运行一个 pod，然后使用 kubectl exec 命令连接到 shell。从该 shell 中连接到其他 node、pod 和 service。
  - 有些集群可能允许 ssh 到集群上的某个节点。 从那个节点您可以访问到集群中的服务。这是一个非标准的方法，它可能将在某些集群上奏效，而在某些集群不行。这些节点上可能安装了浏览器和其他工具也可能没有。群集 DNS 可能无法正常工作。  
  
### Discovering builtin services

通常集群内会有几个在 kube-system 中启动的服务。使用 `kubectl cluster-info` 命令获取该列表：  

```
$ kubectl cluster-info

  Kubernetes master is running at https://104.197.5.247
  elasticsearch-logging is running at https://104.197.5.247/api/v1/namespaces/kube-system/services/elasticsearch-logging/proxy
  kibana-logging is running at https://104.197.5.247/api/v1/namespaces/kube-system/services/kibana-logging/proxy
  kube-dns is running at https://104.197.5.247/api/v1/namespaces/kube-system/services/kube-dns/proxy
  grafana is running at https://104.197.5.247/api/v1/namespaces/kube-system/services/monitoring-grafana/proxy
  heapster is running at https://104.197.5.247/api/v1/namespaces/kube-system/services/monitoring-heapster/proxy
```
这显示了访问每个服务的代理 URL。  

例如，此集群启用了集群级日志记录（使用Elasticsearch），如果传入合适的凭据，可以在该地址 `https://104.197.5.247/api/v1/namespaces/kube-system/services/elasticsearch-logging/proxy/` 访问到，或通过 kubectl 代理，例如：`http://localhost:8080/api/v1/namespaces/kube-system/services/elasticsearch-logging/proxy/`。  

#### 手动构建 apiserver 代理 URL

如上所述，您可以使用 kubectl cluster-info 命令来检索服务的代理 URL。要创建包含服务端点、后缀和参数的代理 URL，您只需附加到服务的代理URL：  

```
http://kubernetes_master_address/api/v1/namespaces/namespace_name/services/service_name[:port_name]/proxy
```
如果您没有指定 port 的名字，那么您不必在 URL 里指定 port_name。  

- Example

- To access the Elasticsearch service endpoint `_search?q=user:kimchy`, you would use:  
`http://104.197.5.247/api/v1/namespaces/kube-system/services/elasticsearch-logging/proxy/_search?q=user:kimchy`  

- To access the Elasticsearch cluster health information `_cluster/health?pretty=true`, you would use:  
```
https://104.197.5.247/api/v1/namespaces/kube-system/services/elasticsearch-logging/proxy/_cluster/health?pretty=true
```

output:  
```
  {
    "cluster_name" : "kubernetes_logging",
    "status" : "yellow",
    "timed_out" : false,
    "number_of_nodes" : 1,
    "number_of_data_nodes" : 1,
    "active_primary_shards" : 5,
    "active_shards" : 5,
    "relocating_shards" : 0,
    "initializing_shards" : 0,
    "unassigned_shards" : 5
  }
```
 
#### 使用 web 浏览器来访问集群中运行的服务
 
您可以将 apiserver 代理网址放在浏览器的地址栏中。 然而：  
 
- Web 浏览器通常不能传递 token，因此您可能需要使用基本（密码）认证。 Apiserver 可以配置为接受基本认证，但您的集群可能未配置为接受基本认证。  
- 某些网络应用程序可能无法正常工作，特别是那些在不知道代理路径前缀的情况下构造 URL 的客户端 JavaScript。  

### 多种代理

在使用 kubernetes 的时候您可能会遇到许多种不同的代理：  

- The kubectl proxy:
  - runs on a user’s desktop or in a pod
  - proxies from a localhost address to the Kubernetes apiserver
  - client to proxy uses HTTP
  - proxy to apiserver uses HTTPS
  - locates apiserver
  - adds authentication headers
  
- The apiserver proxy:
  - is a bastion built into the apiserver
  - connects a user outside of the cluster to cluster IPs which otherwise might not be reachable
  - runs in the apiserver processes
  - client to proxy uses HTTPS (or http if apiserver so configured)
  - proxy to target may use HTTP or HTTPS as chosen by proxy using available information
  - can be used to reach a Node, Pod, or Service
  - does load balancing when used to reach a Service
  
- The kube proxy:
  - runs on each node
  - proxies UDP and TCP
  - does not understand HTTP
  - provides load balancing
  - is just used to reach services
  
- A Proxy/Load-balancer in front of apiserver(s):
  - existence and implementation varies from cluster to cluster (e.g. nginx)
  - sits between all clients and one or more apiservers
  - acts as load balancer if there are several apiservers.
  
- Cloud Load Balancers on external services:
  - are provided by some cloud providers (e.g. AWS ELB, Google Cloud Load Balancer)
  - are created automatically when the Kubernetes service has type LoadBalancer
  - use UDP/TCP only
  - implementation varies by cloud provider.
  
Kubernetes users will typically not need to worry about anything other than the first two types. The cluster admin will typically ensure that the latter types are setup correctly.  

除了前面两种类型，k8s用户不需要去担心其它类型，集群管理员会确保后面几种类型是正确设置的。  


