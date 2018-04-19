# helm and charts

## helm intro

Helm is a tool for managing Kubernetes charts. Charts are packages of pre-configured Kubernetes resources.

Helm是一个用来管理Kubernetes charts的工具，charts是预先配置好的 Kubernetes资源包！

Use Helm to:

- Find and use popular software packaged as Kubernetes charts  
- Share your own applications as Kubernetes charts  
- Create reproducible builds of your Kubernetes applications  
- Intelligently manage your Kubernetes manifest files  
- Manage releases of Helm packages  

Helm is a tool that streamlines installing and managing Kubernetes applications. Think of it like apt/yum/homebrew for Kubernetes.  

用来简化Kubernetes应用的部署和管理，可以把Helm比作CentOS的yum工具，charts则类似于rpm包。  

- Helm has two parts: a client (helm) and a server (tiller)  
- Tiller runs inside of your Kubernetes cluster, and manages releases (installations) of your charts.  
- Helm runs on your laptop, CI/CD, or wherever you want it to run.  
- Charts are Helm packages that contain at least two things:  
  - A description of the package (Chart.yaml)  
  - One or more templates, which contain Kubernetes manifest files  
- Charts can be stored on disk, or fetched from remote chart repositories (like Debian or RedHat packages)  

正如上面提到，一个charts至少包含了两类文件，它们是一个应用部署所需要的文件，通过将这些文件打包，上传到charts repository，
就可以docker镜像一样，进行应用的分发！

## helm 产生原因

微服务和容器化给复杂应用部署与管理带来了极大的挑战，一个传统应用可能由多个微服务组成，如果通过手动部署，即耗时又容易出错；
Helm 通过软件打包的形式，支持发布的版本管理和控制，很大程度上简化了Kubernetes应用部署和管理的复杂性。

再进一步说，利用Kubernetes部署一个应用，需要Kubernetes原生资源文件如deployment、replicationcontroller、service或pod等。
而对于一个复杂的应用，会有很多类似上面的资源描述文件，如果有更新或回滚应用的需求，可能要修改和维护所涉及的大量资源文件，
且由于缺少对发布过的应用版本管理和控制，使Kubernetes上的应用维护和更新等面临诸多的挑战，而Helm可以帮我们解决这些问题。

Helm 把Kubernetes资源(比如deployments、services或 ingress等) 打包到一个chart中，而chart被保存到chart仓库，
通过chart仓库可用来存储和分享chart。Helm使发布可配置，支持发布应用配置的版本管理，简化了Kubernetes部署应用的
版本控制、打包、发布、删除、更新等操作。

## helm 架构

helm架构图如下：  
[!helm arch](https://github.com/fffffreedom/Pictures/blob/master/handbook/helm-arch.jpg)

## helm 组成
- Helm has two parts: a client (helm) and a server (tiller)  
- Tiller runs inside of your Kubernetes cluster, and manages releases (installations) of your charts.  
- Helm runs on your laptop, CI/CD, or wherever you want it to run.  

## helm 安装 -- kubeadm安装的k8s cluster

### helm
```
curl https://raw.githubusercontent.com/kubernetes/helm/master/scripts/get > get_helm.sh
sh get_helm.sh
```

### tiller
```
helm init
```

## 搭建私有charts仓库
一个简单的charts仓库，运行如下命令即可搭建起来：  
```
helm serve --address 0.0.0.0:8879 --repo-path .
```
可以通过helm serve所运行主机的IP:8879来访问Charts仓库。  

## chart package 目录结构
使用如下命令可以创建一个名为foo的chart package:  
```
helm create foo
```

## helm command

intro the helm command

## 参考资料

[简化Kubernetes应用部署工具-Helm简介](https://www.kubernetes.org.cn/2700.html)  
[使用Helm管理kubernetes应用](https://jimmysong.io/kubernetes-handbook/practice/helm.html)  
[helm](https://github.com/kubernetes/helm)  
[charts](https://github.com/kubernetes/charts)  
[是时候使用Helm了：Helm, Kubernetes的包管理工具](https://blog.frognew.com/2017/12/its-time-to-use-helm.html)  
[Kubernetes 微讲堂 08：Kubernetes应用部署](http://v.youku.com/v_show/id_XMzIxNTY5NjMxNg==.html?spm=a2h1n.8251843.playList.5!8~5~A&f=51266236&o=1)  
