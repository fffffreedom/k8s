# Concepts

## [what is Kubernetes](https://kubernetes.io/)

Kubernetes is an open-source system for automating deployment, scaling, and management of containerized applications.

### Why do I need Kubernetes

Kubernetes has a number of features. It can be thought of as:  

- a container platform  
- a microservices platform  
- a portable cloud platform and a lot more.  

Kubernetes provides a container-centric management environment. 
It orchestrates computing, networking, and storage infrastructure on behalf of user workloads. 
This provides much of the simplicity of Platform as a Service (PaaS) with the flexibility of Infrastructure as a Service (IaaS), 
and enables portability across infrastructure providers.

### What Kubernetes is not

- Does not limit the types of applications supported  
- Does not deploy source code and does not build your application  
- Does not provide application-level services  
- Does not dictate logging, monitoring, or alerting solutions  
- Does not provide nor mandate a configuration language/system  
- Does not provide nor adopt any comprehensive machine configuration, maintenance, management, or self-healing systems.  

### Summary of container benefits

- Agile application creation and deployment  
- Continuous development, integration, and deployment  
- Dev and Ops separation of concerns  
- Observability  
- Environmental consistency across development, testing, and production  
- Cloud and OS distribution portability  
- Application-centric management  
- Loosely coupled, distributed, elastic, liberated micro-services  
- Resource isolation  
- Resource utilization  

## concept overview

To work with Kubernetes, you use **Kubernetes API objects** to describe your cluster’s desired state: 
what applications or other workloads you want to run, what container images they use, the number of replicas, 
what network and disk resources you want to make available, and more.

 You set your desired state by creating objects using the Kubernetes API, typically via the command-line interface, kubectl. 
 You can also use the Kubernetes API directly to interact with the cluster and set or modify your desired state.
 
 Once you’ve set your desired state, the Kubernetes Control Plane works to make the cluster’s current state match the desired state. 
 To do so, Kubernetes performs a variety of tasks automatically–such as starting or restarting containers, scaling the number of 
 replicas of a given application, and more. The Kubernetes Control Plane consists of a collection of processes running on your cluster:
 
 - The Kubernetes Master is a collection of three processes that run on a single node in your cluster, which is designated as the master 
 node. Those processes are: kube-apiserver, kube-controller-manager and kube-scheduler.
 
 - Each individual non-master node in your cluster runs two processes:  
  - kubelet, which communicates with the Kubernetes Master.  
  - kube-proxy, a network proxy which reflects Kubernetes networking services on each node.  
  
## Kubernetes Objects

Kubernetes contains a number of abstractions that represent the state of your system: deployed containerized applications and workloads, 
their associated network and disk resources, and other information about what your cluster is doing. 
**These abstractions are represented by objects in the Kubernetes API; **  

The basic Kubernetes objects include:  

- pod  
- service  
- volume  
- namespace  

In addition, Kubernetes contains a number of higher-level abstractions called Controllers. Controllers build upon the basic objects, 
and provide additional functionality and convenience features. They include:

- ReplicaSet  
- Deployment  
- StatefulSet  
- DaemonSet  
- Job  
- Kube  

## Kubernetes Control Plane

The various parts of the Kubernetes Control Plane, such as the Kubernetes Master and kubelet processes, 
govern how Kubernetes communicates with your cluster. 

### Kubernetes Master

The Kubernetes master is responsible for maintaining the desired state for your cluster. 
When you interact with Kubernetes, such as by using the kubectl command-line interface, 
you’re communicating with your cluster’s Kubernetes master.

The “master” refers to a collection of processes managing the cluster state. 
Typically these processes are all run on a single node in the cluster, 
and this node is also referred to as the master. The master can also be replicated for availability and redundancy.

### Kubernetes Nodes

The nodes in a cluster are the machines (VMs, physical servers, etc) that run your applications and cloud workflows. 
The Kubernetes master controls each node; you’ll rarely interact with nodes directly.

## 参考资料
 

[what-is-kubernetes](https://kubernetes.io/docs/concepts/overview/what-is-kubernetes/)  
[concepts](https://kubernetes.io/docs/concepts/)  
 
