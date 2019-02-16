# minikube handbook

## minikube start
run `minikube start -h` to see the useful args of minikube:
```
--kubernetes-version
--container-runtime
--cpus
--dns-domain
--registry-mirror
--memory
```
for example, to run a k8s cluster with the version of v1.10.0:
```
minikube start --kubernetes-version=v1.10.0
```
and more setting with memory and registry:
```
minikube start --memory=4096 --registry-mirror=https://registry.docker-cn.com --kubernetes-version=v1.10.0
```

## minikube ssh
