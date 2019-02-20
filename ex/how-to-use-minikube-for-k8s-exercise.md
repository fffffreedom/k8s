# how-to-use-minikube-for-k8s-exercise

## 步骤
- 安装集群（minikube）
- 编写应用
- 打包镜像
- 部署应用
- 应用管理

## 编写应用
- 编写代码
```
package main

import (
	"net/http"
	"fmt"
)

func HelloHandle(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintln(w, "hello http")
}

func main ()  {
	http.HandleFunc("/", HelloHandle)
	http.ListenAndServe(":8080", nil)
}
```
- 静态编译并检查
```
CGO_ENABLED=0 GOOS=linux go build -a -ldflags '-extldflags "-static"' -o hello_http .
bogon:http jonny$ file hello_http
hello_http: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), statically linked, with debug_info, not stripped
```
## 打包镜像
使用scratch来减小镜像的大小：
```
FROM scratch
ADD hello_http /
CMD ["/hello_http"]
```
**运行命令：`eval $(minikube docker-env)`切换到minikube VM的docker环境！，再编译镜像，这样镜像就在minikube的VM中，可以直接使用！**
```
docker build -t hello-http:v1 .
```
## 创建deployment
```
kubectl run hello-http --image=hello-http:v1 --port=8080
```
这个时候查看一下delpyment和service：
```
bogon:hello jonny$ kubectl get svc 
NAME         TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
kubernetes   ClusterIP      10.96.0.1       <none>        443/TCP          3d

bogon:hello jonny$ minikube service list
|-------------|----------------------|--------------|
|  NAMESPACE  |         NAME         |     URL      |
|-------------|----------------------|--------------|
| default     | hello-http           | No node port | <== 没有暴露服务
| default     | kubernetes           | No node port |
| kube-system | kube-dns             | No node port |
| kube-system | kubernetes-dashboard | No node port |
|-------------|----------------------|--------------|
```
## 创建service并暴露服务
```
kubectl expose deployment hello-http --type=LoadBalancer

bogon:hello jonny$ minikube service list
|-------------|----------------------|-----------------------------|
|  NAMESPACE  |         NAME         |             URL             |
|-------------|----------------------|-----------------------------|
| default     | hello-http           | http://192.168.99.100:32718 | <== 已暴露服
| default     | kubernetes           | No node port                |
| kube-system | kube-dns             | No node port                |
| kube-system | kubernetes-dashboard | No node port                |
|-------------|----------------------|-----------------------------|
```
## 打开服务（前面暴露了服务）
```
minikube service hello-http
```
## reference
http://ju.outofmemory.cn/entry/366843  
https://colobu.com/2015/10/12/create-minimal-golang-docker-images/  

