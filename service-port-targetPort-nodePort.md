# nodePort，targetPort，port的区别和意义

## nodePort

服务暴露端口，集群外部用户可访问服务的端口，不能重复使用。  

比如一个Web应用需要被其他用户访问，那么需要配置type=NodePort，而且配置nodePort=30001，那么其他机器就可以通过浏览器访问scheme://node:30001
访问到该服务，例如http://node:30001 。例如MySQL数据库可能不需要被外界访问，只需被内部服务访问，那么不必设置NodePort.  

## targetPort

容器或者pod提供服务的端口，相同的程序使用相同的端口，它们是可重复的，会映射到不同的物理端口上。  

与制作容器时暴露的端口一致（DockerFile中EXPOSE），例如docker.io官方的nginx暴露的是80端口。 
docker.io官方的nginx容器的DockerFile参考https://github.com/nginxinc/docker-nginx  

## port

k8s集群内部使用的端口（服务之间访问），尽管mysql容器暴露了3306端口(参考https://github.com/docker-library/mysql/ 的DockerFile) ,
但是集群内其他容器需要通过33306端口访问该服务，集群外部不能访问mysql服务，因为他没有配置NodePort类型。  

## port、nodePort总结

总的来说，port和nodePort都是service的端口，前者暴露给集群内客户访问服务，后者暴露给集群外客户访问服务。从这两个端口到来的数据都需要经过反向代理kube-proxy流入后端pod的targetPod，从而到达pod上的容器内。  

## yaml

```
apiVersion: v1
kind: Service
metadata:
 name: nginx-service
spec:
 type: NodePort
 ports:
 - port: 30080
   targetPort: 80
   nodePort: 30001
 selector:
  name: nginx-pod
```

```
apiVersion: v1
kind: Service
metadata:
 name: mysql-service
spec:
 ports:
 - port: 33306
   targetPort: 3306
 selector:
  name: mysql-pod
```

## reference
Kubernetes中的nodePort，targetPort，port的区别和意义  
http://blog.csdn.net/u013760355/article/details/70162242  
kubernetes中port、target port、node port的对比分析，以及kube-proxy代理(TOREAD)  
http://blog.csdn.net/xinghun_4/article/details/50492041  

