# TLS bootstrapping

## reference
https://kubernetes.io/docs/reference/command-line-tools-reference/kubelet-tls-bootstrapping/
https://kubernetes.io/docs/reference/access-authn-authz/authentication/
https://kubernetes.io/docs/reference/access-authn-authz/bootstrap-tokens/
https://lingxiankong.github.io/2018-09-18-kubelet-bootstrap-process.html
https://kubernetes.io/docs/tasks/tls/certificate-rotation/

## 证书基础知识

## 

## Initialization Process
https://kubernetes.io/docs/reference/command-line-tools-reference/kubelet-tls-bootstrapping/#initialization-process

## configuration
https://kubernetes.io/docs/reference/command-line-tools-reference/kubelet-tls-bootstrapping/#configuration

### kube-apiserver
### kube-controller-manager
While the apiserver receives the requests for certificates from the kubelet and authenticates those requests, 
the controller-manager is responsible for issuing actual signed certificates.
In order for the controller-manager to sign certificates, it needs the following:
- access to the “Kubernetes CA key and certificate” that you created and distributed
- enabling CSR signing
### kubelet
