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
### Container hooks
There are two hooks that are exposed to Containers:  

#### PostStart
This hook executes immediately after a container is created. However, there is **no guarantee** that the hook will execute before the container ENTRYPOINT. No parameters are passed to the handler.  

#### PreStop
This hook is called immediately before a container is terminated. It is blocking, meaning it is synchronous, so it must complete before the call to delete the container can be sent. No parameters are passed to the handler.  
A more detailed description of the termination behavior can be found in [Termination of Pods](https://kubernetes.io/docs/concepts/workloads/pods/pod/#termination-of-pods).  

### Hook handler implementations
Containers can access a hook by implementing and registering a handler for that hook. There are two types of hook handlers that can be implemented for Containers:  
- Exec  
Executes a specific command, such as pre-stop.sh, inside the cgroups and namespaces of the Container. Resources consumed by the command are counted against the Container.  
- HTTP  
Executes an HTTP request against a specific endpoint on the Container.  

### Hook handler execution
`PostStart hook`和`ENTRYPOINT`是异步执行，如果hook执行时间太久，container不会达到`running`状态；  
`PreStop hook`也一样，如果hook执行时间太久，container会一直处于`Terminating`状态，直到超时`terminationGracePeriodSeconds`，被杀掉。  
如果hook运行失败，container都将被杀。  

### Hook delivery guarantees
hook至少被掉用一次，这意味着对于任何给定的事件可以多次调用一个钩子，对于PostStart or PreStop也是一样，在于hook是否正确处理了事件。  

### Debugging Hook handlers
The logs for a Hook handler are not exposed in Pod events, If a handler fails for some reason, it broadcasts an event.  
就是说出错时广播事件！  

**For PostStart, this is the FailedPostStartHook event, and for PreStop, this is the FailedPreStopHook event.**  
