# kubeconfig

> https://kubernetes.io/docs/concepts/configuration/organize-cluster-access-kubeconfig/  
> https://kubernetes.io/docs/tasks/access-application-cluster/configure-access-multiple-clusters/  

Kubeconfig is a file that you can use to switch multiple clusters by switching context.  

使用kubeadm安装好的一个集群，其kubeconfig文件内容如下：  

```
[root@k8s-master ~]# kubectl config view 
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: REDACTED
    server: https://10.101.17.74:6443
  name: kubernetes
contexts:
- context:
    cluster: kubernetes
    user: kubernetes-admin
  name: kubernetes-admin@kubernetes
current-context: kubernetes-admin@kubernetes
kind: Config
preferences: {}
users:
- name: kubernetes-admin
  user:
    client-certificate-data: REDACTED
    client-key-data: REDACTED
```

该文件也可以配置多个 context，并通过 current-context 来指定当前的 context.  

## Organizing Cluster Access Using kubeconfig Files

> https://kubernetes.io/docs/concepts/configuration/organize-cluster-access-kubeconfig/  

Use kubeconfig files to organize information about clusters, users, namespaces, and authentication mechanisms. 
The kubectl command-line tool uses kubeconfig files to find the information it needs to choose a cluster and 
communicate with the API server of a cluster.  

kubeconfig 文件用来保存集群的信息，信息包括：apisever地址、用户信息、认证机制。  

kubectl 命令从 kubeconfig 文件获取集群查询信息，以及与集群中的 API server 进行通信。  

默认情况下，kubectl从`$HOME/.kube`目录中查找名为`config`的文件，也要以设置`KUBECONFIG`环境变量指定kubeconfig文件，或者是设置`--kubeconfig`标志。  






