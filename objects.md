# Kubernetes Objects
> https://kubernetes.io/docs/concepts/  

Kubernetes contains a number of abstractions that represent the state of your system:   
- deployed containerized applications and workloads,  
- their associated network and disk resources, 
- and other information about what your cluster is doing.  

These abstractions are represented by objects in the Kubernetes API; see the Kubernetes Objects overview for more details.  
**The basic Kubernetes objects include:**  
- Pod
- Service
- Volume
- Namespace  
In addition, Kubernetes contains a number of **higher-level abstractions called Controllers**. 
Controllers build upon the basic objects, and provide additional functionality and convenience features. They include:
- ReplicaSet
- Deployment
- StatefulSet
- DaemonSet
- Job
