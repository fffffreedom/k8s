# TLS bootstrapping

https://kubernetes.io/docs/admin/kubelet-tls-bootstrapping/  
https://k8smeetup.github.io/docs/admin/kubelet-tls-bootstrapping/  
https://kubernetes.io/docs/tasks/tls/managing-tls-in-a-cluster/  

Kubernetes TLS bootstrapping 那点事  
https://mritd.me/2018/01/07/kubernetes-tls-bootstrapping-note/  

https://github.com/opsnull/follow-me-install-kubernetes-cluster/blob/master/06-%E9%83%A8%E7%BD%B2Master%E8%8A%82%E7%82%B9.md  

```
--cluster-signing-cert-file
--cluster-signing-key-file
cluster-signing-* 指定的证书和私钥文件用来签名为 TLS BootStrap 创建的证书和私钥；

--root-ca-file
用来对 kube-apiserver 证书进行校验，指定该参数后，才会在Pod 容器的 ServiceAccount 中放置该 CA 证书文件；
```
