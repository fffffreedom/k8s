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

## reference
http://ju.outofmemory.cn/entry/366843  
https://colobu.com/2015/10/12/create-minimal-golang-docker-images/  

