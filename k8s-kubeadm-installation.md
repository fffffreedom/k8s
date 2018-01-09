# k8s-kubeadm-installation
## 安装过程
- 设置服务器或者虚拟机的hostname  
- 配置/etc/hosts  
- 关闭防火墙
```
systemctl stop firewalld
systemctl disable firewalld
```
- 禁用SELINUX
```
setenforce 0
sed -i 's/^SELINUX=.*/SELINUX=disabled/g' /etc/selinux/config
```
- 配置网络（master only）
```
cat <<EOF >  /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sysctl --system
```
- 安装docker
配置yum源：  
```
cat << EOF > /etc/yum.repos.d/docker.repo
[dockerrepo]
name=Docker Repository
baseurl=https://yum.dockerproject.org/repo/main/centos/7/
enabled=1
gpgcheck=1
gpgkey=https://yum.dockerproject.org/gpg
EOF
yum install -y docker-engine.x86_64-1.12.6-1.el7.centos
```
- 安装kubelet kubeadm kubectl   
配置kubernets yum源：  
```
cat << EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
EOF

# 安装kubelet kubeadm kubectl
yum install -y kubelet kubeadm kubectl
systemctl enable kubelet && systemctl start kubelet
```
- 提前下载好安装所需要的镜像
```
#!/bin/bash

images=(etcd-amd64:3.0.17 \
        pause-amd64:3.0 \
        kube-proxy-amd64:v1.7.2 \
        kube-scheduler-amd64:v1.7.2 \
        kube-controller-manager-amd64:v1.7.2 \
        kube-apiserver-amd64:v1.7.2 \
        kubernetes-dashboard-amd64:v1.6.1 \
        k8s-dns-sidecar-amd64:1.14.4 \
        k8s-dns-kube-dns-amd64:1.14.4
        k8s-dns-dnsmasq-nanny-amd64:1.14.4)
for imageName in ${images[@]} ; do
  docker pull cloudnil/$imageName
  docker tag cloudnil/$imageName gcr.io/google_containers/$imageName
  docker rmi cloudnil/$imageName
done

dockre pull quay.io/calico/node:v2.4.1
docker pull quay.io/calico/cni:v1.10.0
docker pull quay.io/coreos/etcd:v3.1.10
```
- kubeadm (master only)
```
kubeadm init --kubernetes-version=v1.7.2 \
    --pod-network-cidr=192.168.0.0/16 \
    --apiserver-advertise-address=0.0.0.0 \
    --apiserver-cert-extra-sans=10.101.17.74,10.101.17.75,10.101.17.76,k8s-master,k8s-node-1,k8s-node-2
```
如果已经运行过该命令，则添加--skip-preflight-checks参数来跳过已执行过的初始化。  
如果正常的话，会输出如下打印：
```
[kubeadm] WARNING: kubeadm is in beta, please do not use it for production clusters.
[init] Using Kubernetes version: v1.7.2
[init] Using Authorization modes: [Node RBAC]
[preflight] Running pre-flight checks
[preflight] Starting the kubelet service
[kubeadm] WARNING: starting in 1.8, tokens expire after 24 hours by default (if you require a non-expiring token use --token-ttl 0)
[certificates] Generated CA certificate and key.
[certificates] Generated API server certificate and key.
[certificates] API Server serving cert is signed for DNS names [k8s-master kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local k8s-master k8s-node-1 k8s-node-2] and IPs [10.101.17.74 10.101.17.75 10.101.17.76 10.96.0.1 10.101.17.74]
[certificates] Generated API server kubelet client certificate and key.
[certificates] Generated service account token signing key and public key.
[certificates] Generated front-proxy CA certificate and key.
[certificates] Generated front-proxy client certificate and key.
[certificates] Valid certificates and keys now exist in "/etc/kubernetes/pki"
[kubeconfig] Wrote KubeConfig file to disk: "/etc/kubernetes/admin.conf"
[kubeconfig] Wrote KubeConfig file to disk: "/etc/kubernetes/kubelet.conf"
[kubeconfig] Wrote KubeConfig file to disk: "/etc/kubernetes/controller-manager.conf"
[kubeconfig] Wrote KubeConfig file to disk: "/etc/kubernetes/scheduler.conf"
[apiclient] Created API client, waiting for the control plane to become ready
[apiclient] All control plane components are healthy after 30.511805 seconds
[token] Using token: bbe302.4f8c0f0f740c052e
[apiconfig] Created RBAC rules
[addons] Applied essential addon: kube-proxy
[addons] Applied essential addon: kube-dns
Your Kubernetes master has initialized successfully!

To start using your cluster, you need to run (as a regular user):

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  http://kubernetes.io/docs/admin/addons/

You can now join any number of machines by running the following on each node
as root:
  kubeadm join --token bbe302.4f8c0f0f740c052e 10.101.17.74:6443
```
按照上面的提示，运行：
```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
安装网络插件：
```
kubectl apply -f http://docs.projectcalico.org/v2.4/getting-started/kubernetes/installation/hosted/kubeadm/1.6/calico.yaml
```
运行命令查看master是否安装完成：
```
[root@k8s-master ~]# kubectl get pods --all-namespaces
NAMESPACE     NAME                                       READY     STATUS    RESTARTS   AGE
default       nginx-controller-g67g8                     1/1       Running   0          31m
kube-system   calico-etcd-4k7k3                          1/1       Running   0          53m
kube-system   calico-node-2f8ht                          2/2       Running   0          42m
kube-system   calico-node-rcd48                          2/2       Running   0          43m
kube-system   calico-node-s5dzs                          2/2       Running   0          53m
kube-system   calico-policy-controller-336633499-m0d1f   1/1       Running   0          53m
kube-system   etcd-k8s-master                            1/1       Running   0          54m
kube-system   kube-apiserver-k8s-master                  1/1       Running   0          53m
kube-system   kube-controller-manager-k8s-master         1/1       Running   0          54m
kube-system   kube-dns-2425271678-nbbxh                  3/3       Running   0          54m
kube-system   kube-proxy-817dl                           1/1       Running   0          42m
kube-system   kube-proxy-r8xc0                           1/1       Running   0          54m
kube-system   kube-proxy-wxwjr                           1/1       Running   0          43m
kube-system   kube-scheduler-k8s-master                  1/1       Running   0          54m
```
- node加入集群
master正常之后，到节点云运行加入集群的命令：
```
kubeadm join --token bbe302.4f8c0f0f740c052e 10.101.17.74:6443
```
查看所有节点是否正常：
```
kubectl get node
```
- 部署dashboard  
下载yaml文件，1.7版本之后的dashboard提升了安全性，我们找到1.6.3版本的yaml文件：  
https://github.com/kubernetes/dashboard/tree/v1.6.3/src/deploy/kubernetes-dashboard.yaml  
要想在window下登录dashboard的web界面，需要将服务暴露出来：  
```
# 这里的image版本原为1.6.3，修改为本地已有的dashboard版本
image: gcr.io/google_containers/kubernetes-dashboard-amd64:v1.6.1
# 暴露服务
kind: Service
apiVersion: v1
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard
  namespace: kube-system
spec:
  type: NodePort  <--
  ports:
  - port: 80
    targetPort: 9090
    nodePort: 30000  <--
  selector:
    k8s-app: kubernetes-dashboard
```
然后运行命令， 创建dashboard：  
```
kubectl create -f kubernetes-dashboard.yaml
kubectl get svc --all-namespaces
```

## 问题
- 在node运行kubectl命令出错
`The connection to the server localhost:8080 was refused - did you specify the right host or port?`
需要在所有节点上运行下面的命令（admin.conf只会在运行了kubeadm init的节点上生成，需要将其拷贝到其它节点）
```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
#要在(id -u):(id -g)括号前加上$，否则会报错
chown $(id -u):$(id -g) $HOME/.kube/config
```
## 参考
官方  
https://kubernetes.io/docs/setup/independent/install-kubeadm/  
https://kubernetes.io/docs/reference/setup-tools/kubeadm/kubeadm-init/  
https://v1-7.docs.kubernetes.io/docs/admin/kubeadm/  
使用kubeadm安装Kubernetes 1.7 --步骤清晰  
https://blog.frognew.com/2017/07/kubeadm-install-kubernetes-1.7.html  
CentOS7.3利用kubeadm安装kubernetes1.7.3完整版(官方文档填坑篇)  
http://www.cnblogs.com/liangDream/p/7358847.html  
Kubeadm安装Kubernetes环境 --查找镜像  
http://www.cnblogs.com/ericnie/p/7749588.html  
Kubeadm部署k8s(资源已有) --如何排错 and stat /etc/kubernetes/kubelet.conf: no such file or directory  
http://www.winseliu.com/blog/2017/08/13/kubeadm-install-k8s-on-centos7-with-resources/  
http://tonybai.com/2017/05/15/setup-a-ha-kubernetes-cluster-based-on-kubeadm-part1/  
## TODO
使用kubeadm在CentOS 7上安装Kubernetes 1.8  
https://www.zybuluo.com/ncepuwanghui/note/953929
