# Container Runtime Interface
> http://blog.kubernetes.io/2016/12/container-runtime-interface-cri-in-kubernetes.html
## What is the CRI and why does Kubernetes need it?
## Overview of CRI
## Pod and container lifecycle management
```
service RuntimeService {
    // Sandbox operations.
    rpc RunPodSandbox(RunPodSandboxRequest) returns (RunPodSandboxResponse) {}
    rpc StopPodSandbox(StopPodSandboxRequest) returns (StopPodSandboxResponse) {}
    rpc RemovePodSandbox(RemovePodSandboxRequest) returns (RemovePodSandboxResponse) {}
    rpc PodSandboxStatus(PodSandboxStatusRequest) returns (PodSandboxStatusResponse) {}
    rpc ListPodSandbox(ListPodSandboxRequest) returns (ListPodSandboxResponse) {}
    // Container operations.
    rpc CreateContainer(CreateContainerRequest) returns (CreateContainerResponse) {}
    rpc StartContainer(StartContainerRequest) returns (StartContainerResponse) {}
    rpc StopContainer(StopContainerRequest) returns (StopContainerResponse) {}
    rpc RemoveContainer(RemoveContainerRequest) returns (RemoveContainerResponse) {}
    rpc ListContainers(ListContainersRequest) returns (ListContainersResponse) {}
    rpc ContainerStatus(ContainerStatusRequest) returns (ContainerStatusResponse) {}
    ...
}
```
A Pod is composed of a group of application containers in an isolated environment with resource constraints.   
In CRI, this environment is called PodSandbox. We intentionally leave some room for the container runtimes to  
interpret the PodSandbox differently based on how they operate internally. For hypervisor-based runtimes,  
PodSandbox might represent a virtual machine. For others, such as Docker, it might be Linux namespaces.  
The PodSandbox must respect the pod resources specifications. this is achieved by launching all the processes  
within the pod-level cgroup that kubelet creates and passes to the runtime.  
Before starting a pod, kubelet calls RuntimeService.RunPodSandbox to create the environment.  
This includes setting up networking for a pod (e.g., allocating an IP). Once the PodSandbox is active,  
individual containers can be created/started/stopped/removed independently. To delete the pod,   
kubelet will stop and remove containers before stopping and removing the PodSandbox.  
Kubelet is responsible for managing the lifecycles of the containers through the RPCs,  
exercising the container lifecycles hooks and liveness/readiness checks, while adhering to the restart policy of the pod.
## Exec/attach/port-forward requests
