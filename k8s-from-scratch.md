https://kubernetes.io/docs/getting-started-guides/scratch/  
## Software Binaries
You will need binaries for:  
- etcd
- docker
- Kubernetes
  - kubelet
  - kube-proxy
  - kube-apiserver
  - kube-controller-manager
  - kube-scheduler
## Selecting Images
You will run docker, kubelet, and kube-proxy outside of a container,   
the same way you would run any system daemon, so you just need the bare binaries.  
For etcd, kube-apiserver, kube-controller-manager, and kube-scheduler,  
we recommend that you run these as containers, so you need an image to be built.  
You have several choices for Kubernetes images:  

