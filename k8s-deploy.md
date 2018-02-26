# k8s-deploy
## Picking the Right Solution
k8s安装有好多种方法，官网给出了不同情况下，应该选用的安装方式，值得细读！  
https://kubernetes.io/docs/setup/pick-right-solution/  
```
Local-machine Solutions
Hosted Solutions
Turnkey Cloud Solutions
On-Premises turnkey cloud solutions
Custom Solutions
  Universal
  Cloud
  On-Premises VMs
  Bare Metal <-- 物理机
  Integrations
Table of Solutions
  Definition of columns
```
官网给出的例子中，centos 7.x上安装的没有，不靠谱啊~  
## Creating a Custom Cluster from Scratch
https://kubernetes.io/docs/getting-started-guides/scratch/  

## ansible deployment
https://github.com/kubernetes-incubator/kubespray  
https://github.com/kubernetes/contrib  
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
## 如何快速启动一个Kubernetes集群（minikube）
https://feisky.xyz/2016/08/24/%E5%A6%82%E4%BD%95%E5%BF%AB%E9%80%9F%E5%90%AF%E5%8A%A8%E4%B8%80%E4%B8%AAKubernetes%E9%9B%86%E7%BE%A4/
