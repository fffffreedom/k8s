# k8s-deploy
## ansible deployment
https://github.com/kubernetes-incubator/kubespray
## Kubernetes单机安装部署
https://www.cnblogs.com/edisonxiang/p/6911994.html
## 在本机启动单机版k8s
https://www.cnblogs.com/opama/p/5931515.html
## Kubernetes单机版安装方法
https://jingyan.baidu.com/article/e73e26c09c074824adb6a736.html
```
yum install -y etcd kubernetes
systemctl start etcd
systemctl start docker
systemctl start kube-apiserver
systemctl start kube-controller-manager
systemctl start kube-scheduler
systemctl start kubelet
systemctl start kube-proxy
bug fix
http://blog.csdn.net/jinzhencs/article/details/51435020
http://blog.csdn.net/a632189007/article/details/78730903
Docker中配置国内镜像
http://blog.csdn.net/zzy1078689276/article/details/77371782
```
