# Ingress Introduce

> https://kubernetes.io/docs/concepts/services-networking/ingress/  

## What is Ingress?

通常，service and pods的IP只在集群内可路由，无法被外部访问。而Ingress可以给service配置外部可访问的URL！  
如下图所示，通过Ingress，可以把service暴露给公网。  

```
    internet
        |
   [ Ingress ]
   --|-----|--
   [ Services ]
```

Ingress can be configured to give services:  
- externally-reachable URLs  
- load balance traffic
- terminate SSL
- offer name based virtual hosting  
- and more.  

Ingress通常由两个组件组成：  
- Ingress controller  
  In order for the Ingress resource to work, the cluster must have an Ingress controller running, 
  You need to choose the ingress controller implementation that is the best fit for your cluster, or implement one.  
  k8s已经实现了一个基于nginx的ingress controller, nginx-ingress-controller, [here](https://raw.githubusercontent.com/kubernetes/ingress-nginx)
- Ingress yaml  
  通过yaml文件定义Ingress，它包含一系列rules，rule指定了URL对应的service。
  如下Ingress定义，它规定了对URL: `http://mywebsite.com/web`的访问将被转发到k8s的一个service上：`webapp:80`  

```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: mywebsite-ingress
spec:
  rules: 
  - host: mywebsite.com
    http:
      paths:
      - path: /web
        backend:
          serviceName: webapp
          servicePort: 80
```

## types of Ingress

### Single Service Ingress

如下yaml文件，只定义了一个backend，所有访问该Ingress的流量，都会被转发给testsvc service.(如何使用？)  

```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: test-ingress
spec:
  backend:
    serviceName: testsvc
    servicePort: 80
```

### Simple fanout

之前说过，Pod的IP只在集群网络内可访问，那么外部要访问具体的pod，就需要用到loadbalance。  
```
    internet
        |
   [ loadbalance ]
   --|-------|--
  [ pod ] [ pod ]
```
Ingress可以使你减少使用loadbalance，因为一个Ingress可以指定多个path，即多个服务。  
```
foo.bar.com -> 178.91.123.132 -> / foo    s1:80
                                 / bar    s2:80
```
要实现上面的fanout，可以用如下的Ingress实现：  
```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: test
  annotations:
    ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: foo.bar.com
    http:
      paths:
      - path: /foo
        backend:
          serviceName: s1
          servicePort: 80
      - path: /bar
        backend:
          serviceName: s2
          servicePort: 80
```

### Name based virtual hosting

Name-based virtual hosts use multiple host names for the same IP address.  

通过不同的域名（对应同一个IP）来访问不同的服务！  

```
foo.bar.com --|                 |-> foo.bar.com s1:80
              | 178.91.123.132  |
bar.foo.com --|                 |-> bar.foo.com s2:80
```

The following Ingress tells the backing loadbalancer to route requests based on the [Host header](https://tools.ietf.org/html/rfc7230#section-5.4).  

```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: test
spec:
  rules:
  - host: foo.bar.com
    http:
      paths:
      - backend:
          serviceName: s1
          servicePort: 80
  - host: bar.foo.com
    http:
      paths:
      - backend:
          serviceName: s2
          servicePort: 80
```

## TLS

如果要使能Ingress的https访问，则需要给Ingress指定secret(包含TLS private key and certificate). 
目前，Ingress只支持TLS端口443，并且模式是TLS termination.   

如果Ingress中的TLS配置部分指定了不同的主机，它们将根据通过SNI TLS扩展指定的主机名在同一端口进行多路复用（由支持SNI的Ingress controller提供）。  
  
```
apiVersion: v1
data:
  tls.crt: base64 encoded cert
  tls.key: base64 encoded key
kind: Secret
metadata:
  name: testsecret
  namespace: default
type: Opaque
```

```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: no-rules-map
spec:
  tls:
  - secretName: testsecret
  backend:
    serviceName: s1
    servicePort: 80
```

需要注意的是，不同的Ingress controller支持的TLS功能会有差别，请参见nginx, GCE的文档。  

## Loadbalancing

Ingress controller采用了通用的负载均衡策略，比如负载均衡算法、后端权重方案等，更多高级的负载均衡原理，不没有通过Ingress提供。
你可以通过[service loadbalancer](https://github.com/kubernetes/ingress-nginx/blob/master/docs/catalog.md)来获得这些功能。  

随着时间的推移，我们计划将适用于跨平台的负载平衡模式提取到Ingress资源中。  

还值得注意的是，健康检查不直接通过Ingress公开。因为k8s通过readiness porbes来实现健康检查。  

## Updating an Ingress

```
kubectl edit ing test
or
kubectl replace -f
```
