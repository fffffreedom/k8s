# ingress 高可用

## 背景

> https://github.com/kubernetes/contrib/tree/master/keepalived-vip

kubernetes v1.6 offers 3 ways to expose a service:

- L4 LoadBalancer  
Available only on cloud providers such as GCE and AWS

- Service via NodePort  
The NodePort directive allocates a port on every worker node, which proxy the traffic to the respective Pod.

- L7 Ingress  
The Ingress is a dedicated loadbalancer (eg. nginx, HAProxy, traefik, vulcand) that redirects incoming HTTP/HTTPS 
traffic to the respective endpoints

## 为什么要使用keepalived

要对外暴露集群内部的服务，对于`L7 Ingress`，我们要怎么保证它的高可用呢？

我们知道，ingress是一种loadbalancer，它由ingress对象（yaml文件）和ingress controller组成，ingress对象中指定7层的转发规则：

```
域名/服务名 -> 集群中的服务
```

看下ingress对象（yaml文件）：

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

从上面可以看到，ingress对象通过**域名**来给外部用户暴露服务，例如：

```
foo.bar.com/s1 -> s1 service in cluster
```

而域名则对应一个外网IP，如果该IP所在的服务器挂了，那么这个Ingress loadbalance就失效了，所以需要在多台服务器上部署ingress，
而这样会有多个物理IP对应一个域名，这显然是有问题的。

所以需要引入VIP，通过keepalived来生成VIP，使用VIP来对应域名，这样一个即使有服务器挂了，也能使域名和IP保持不变！

```
                                                  ___________________
                                                 |                   |
                                           |-----| Host IP: 10.4.0.3 |
                                           |     |___________________|
                                           |
                                           |      ___________________
                                           |     |                   |
Public ----(example.com = vip)-------------|-----| Host IP: 10.4.0.4 |
                                           |     |___________________|
                                           |
                                           |      ___________________
                                           |     |                   |
                                           |-----| Host IP: 10.4.0.5 |
                                                 |___________________|
```

## 边缘节点

首先解释下什么叫边缘节点（Edge Node），所谓的边缘节点即集群内部用来向集群外暴露服务能力的节点，
集群外部的服务通过该节点来调用集群内部的服务，边缘节点是集群内外交流的一个Endpoint。

边缘节点要考虑两个问题：

- 边缘节点的高可用，不能有单点故障，否则整个kubernetes集群将不可用  
- 对外的一致暴露端口，即只能有一个外网访问IP和端口  

**边缘节点需要有能访问外网的能力，在使用时，需要用label打上标记，然后在DaemonSet的yaml文件中指定nodeSelector：**

```
apiVersion: extensions/v1beta1
kind: DaemonSet
metadata:
...
spec:
  template:
  ...
    spec:
      ...
      nodeSelector:
        edgenode: "true"
```

实际例子可见 参考资料。

## 参考资料

[边缘节点配置](https://jimmysong.io/kubernetes-handbook/practice/edge-node-configuration.html)  
[kube-keepalived-vip](https://github.com/kubernetes/contrib/tree/master/keepalived-vip)  
