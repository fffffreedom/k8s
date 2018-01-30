# container-image-secret-lifecycle-hooks

https://kubernetes.io/docs/concepts/containers/images/  
https://kubernetes.io/docs/concepts/containers/container-environment-variables/  
https://kubernetes.io/docs/concepts/containers/container-lifecycle-hooks/  

## Images
You create your Docker image and push it to a registry before referring to it in a Kubernetes pod.  
我们先创建docker image，将它push到镜像中心，然后在kubernetes的Pod spec中引用镜像。  

### Updating Images（重要！）
The default pull policy is `IfNotPresent` which causes the Kubelet to skip pulling an image if it already exists. 
If you would like to always force a pull, you can do one of the following:  
- set the `imagePullPolicy` of the container to `Always`;  
- use **:latest** as the tag for the image to use;  
- enable the `AlwaysPullImages` admission controller.  
**If you did not specify tag of your image, it will be assumed as `:latest`, with pull image policy of `Always` correspondingly.**
Note that you should **avoid** using `:latest` tag, see Best Practices for Configuration for more information.  

### Using a Private Registry
有很多私有镜像仓库，Google, AWS, Azure, 一般都搭建企业内部的镜像中心。一般是选用`docker registry`或者`harbor`。  

那k8s怎么去拉取harbor中的image内？

#### k8s对接harbor
k8s对接harbor，需要创建一个secret对象:  
```
kubectl create secret docker-registry myregistrykey --docker-server=DOCKER_REGISTRY_SERVER \
      --docker-username=DOCKER_USER --docker-password=DOCKER_PASSWORD --docker-email=DOCKER_EMAIL
```
然后在yaml文件里指定`imagePullSecrets`:  
```
apiVersion: v1
kind: Pod
metadata:
  name: foo
  namespace: awesomeapps
spec:
  containers:
    - name: foo
      image: janedoe/awesomeapp:v1
  imagePullSecrets: <--
    - name: myregistrykey  <--
```

**需要注意的是，Pod只能引用它所属namespace内的imagePullSecrets，所以如果使用此方法，需要根据Pod所属namespace创建相应的`imagePullSecrets`。**  

使用secret对象来对接harbor，需要在每个Pod的yaml文件里指定`imagePullSecrets`，有没有一次性的方法? YES

#### Service Accounts
https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/  

```
kubectl patch serviceaccount default -p '{"imagePullSecrets": [{"name": "myregistrykey"}]}'
```
Now, any new pods created in the current namespace will have this added to their spec:  
```
spec:
  imagePullSecrets:
  - name: myregistrykey
```

## Container Environment Variables
The Kubernetes Container environment provides several important resources to Containers:  
- A filesystem, which is a combination of an image and one or more volumes.  
- Information about the Container itself.  
- Information about other objects in the cluster.  

### Container information
container的hostname就是它所在Pod的Pod name。  
The Pod name and namespace are available as environment variables through the downward API.  
用户在Pod spec里定义的变量，在container中也是可以用的，就像在Docker镜像中静态指定的任何环境变量一样。  

### Cluster information

Container创建时运行的所有service列表都可作为该的容器环境变量。  

对于一个名为bar的绑定到名为foo的service的container，以下变量会被定义：  
```
FOO_SERVICE_HOST=<the host the service is running on>
FOO_SERVICE_PORT=<the port the service is running on>
```

Services have dedicated IP addresses and are available to the Container via DNS, if DNS addon is enabled.  

## Container Lifecycle Hooks
