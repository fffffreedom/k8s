# ingress实践

本文主要记录通过ingress来访问kubernetes-dashboard的实现过程。主要的yaml文件可见ingress的github.  

> https://github.com/kubernetes/ingress-nginx/tree/master/deploy  

## 实践环境

环境是使用kubeadm搭建的测试环境，k8s的版本为v1.7.5(kubectl version)，集群一共三台机器。  

## 简介

> https://mritd.me/2017/03/04/how-to-use-nginx-ingress/#21%E9%83%A8%E7%BD%B2%E9%BB%98%E8%AE%A4%E5%90%8E%E7%AB%AF  

Ingress只有两大组件：Ingress Controller(如nginx)和Ingress，当我们创建一个新的Ingress，
它会动态更新Ingress Controller(如nginx)的配置，非常方便。  

## 部署方式

### 域名访问

组件及部署形式：  
```
kubernetes-dashboard service (NodePort)
nginx-ingress-controller deployment (hostNetwork=true)
ingress
```

kubernetes-dashboard的service如果是以NodePort的类型创建，那么已经可以直接通过`NodeIP+NodePort`来访问了；  
通过ingress可以直接使用ingress中指定的域名访问。  

### 域名:NodePort访问

组件及部署形式：  
```
kubernetes-dashboard service (clusterIP)
nginx-ingress-controller deployment 
nginx-ingress-controller service (Nodeport)
ingress
```

与上一种不同，这是通过`域名:NodePort`的方式来访问的。  

## 部署过程

```
curl https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/namespace.yaml \
    | kubectl apply -f -

//不使用该后端服务
//curl https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/default-backend.yaml \
//    | kubectl apply -f -

curl https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/configmap.yaml \
    | kubectl apply -f -

curl https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/tcp-services-configmap.yaml \
    | kubectl apply -f -

curl https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/udp-services-configmap.yaml \
    | kubectl apply -f -
    
curl https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/rbac.yaml \
    | kubectl apply -f -

//nginx-ingress-controller deployment，根据情况添加hostNetwork=true
curl https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/with-rbac.yaml \
    | kubectl apply -f -
```

对于`域名:NodePort`的访问方式，需要部署nginx-ingress-controller的service，dashbod-ingress.yml  

```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: dashboard-ingress
  namespace: kube-system
spec:
  rules:
  - host: dashboard.work.me
    http:
      paths:
      - backend:
          serviceName: kubernetes-dashboard
          servicePort: 80
```

## window访问

hosts配置：  
```
10.101.17.75 dashboard.work.me
```
使用的是worker节点IP，而不是master节点的IP，因为master上没有kube-proxy.  

### 域名访问

```
dashboard.work.me
```

访问的组件过程：  
```
nginx-ingress-controller pod ->　kubernetes-dashboard service -> kubernetes-dashboard pod
```

### 域名访问:NodePort

```
dashboard.work.me:NodePort
```

访问的组件过程：  
```
nginx-ingress-controller service -> nginx-ingress-controller pod ->　kubernetes-dashboard service -> kubernetes-dashboard pod
```

## other

Kubernetes Ingress实战  
https://blog.frognew.com/2017/04/kubernetes-ingress.html  
