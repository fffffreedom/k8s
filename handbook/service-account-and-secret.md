# service account and secret

A service account provides an identity for processes that run in a Pod.

service accout 是给运行在 Pod 里的进程用的，用来在访问 kubernetes API 时提供身份认证！

secret 对象用来保存一些私密数据，如密码等。

## service accout 组成

当一个 Pod 启动时，会自动 mount 三个文件到 Pod 里：  
```
root@nginx:/# ls /run/secrets/kubernetes.io/serviceaccount/
ca.crt	namespace  token
```

三个文件就是组成一个 secret，被 service account 引用。通过如下命令来看下每个 namespace 下默认的 default service account:  

```
jonnydeMacBook-Pro:volume jonny$ kubectl describe sa default
Name:                default
Namespace:           default
Labels:              <none>
Annotations:         <none>
Image pull secrets:  <none>
Mountable secrets:   default-token-9xv6l
Tokens:              default-token-9xv6l
Events:              <none>
```

来看下 default-token-9xv6l secret 的容：

```
jonnydeMacBook-Pro:volume jonny$ kubectl describe secrets default-token-9xv6l
Name:         default-token-9xv6l
Namespace:    default
Labels:       <none>
Annotations:  kubernetes.io/service-account.name=default
              kubernetes.io/service-account.uid=59dc740a-49a9-11e8-a932-08002759ad26

Type:  kubernetes.io/service-account-token

Data
====
ca.crt:     1066 bytes
namespace:  7 bytes
token:      ...
```

从上可知，secrets 包含了：ca.crt, namespace and token

## 创建 service account

```
$ cat > /tmp/serviceaccount.yaml <<EOF
apiVersion: v1
kind: ServiceAccount
metadata:
  name: build-robot
EOF
$ kubectl create -f /tmp/serviceaccount.yaml
serviceaccount "build-robot" created
```

## 获取已有 service account 的 token

Suppose we have an existing service account named “build-robot” as mentioned above, and we create a new secret manually.

```
$ cat > /tmp/build-robot-secret.yaml <<EOF
apiVersion: v1
kind: Secret
metadata:
  name: build-robot-secret
  annotations:
    kubernetes.io/service-account.name: build-robot
type: kubernetes.io/service-account-token
EOF
$ kubectl create -f /tmp/build-robot-secret.yaml
secret "build-robot-secret" created
```

创建好后，你会发现 build-robot-secret 的 token 和 build-robot service account 的 token 的值是一样的！

## 创建 secret

[secret](https://kubernetes.io/docs/concepts/configuration/secret/)

## 创建一个 ImagePullSecrets secret

在 k8s 集群的 node 要拉取 harbor 的镜像时，需要创建一个 secret 来保存 harbor 的用户名和密码，以便可以正常拉取镜像。
     
```
$ kubectl create secret docker-registry myregistrykey --docker-server=DOCKER_REGISTRY_SERVER \ 
        --docker-username=DOCKER_USER --docker-password=DOCKER_PASSWORD --docker-email=DOCKER_EMAIL
```

然后再将其 patch 到 default service account:  
```
kubectl patch serviceaccount default -p '{\"imagePullSecrets\": [{\"name\": \"myregistrykey\"}]}'
```

## 参考资料

[configure-service-account](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/)  
[pull-image-private-registry](https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/)  
