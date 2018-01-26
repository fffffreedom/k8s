# EFK
https://kubernetes.io/docs/concepts/cluster-administration/logging/  
https://blog.frognew.com/2017/01/kubernetes-logging-efk.html#kubernetes%E5%92%8Cefk  

- Basic logging in Kubernetes
use `kubectl log` to get the log of an pod or container.  
- Logging at the node level
  - System component logs
- Cluster-level logging architectures  
  - Using a node logging agent  
  While Kubernetes does not provide a native solution for cluster-level logging, there are several common approaches you can consider.
  Here are some options:  
    - Use a node-level logging agent that runs on every node.  
    **(EFK)**  
    However, node-level logging only works for applications’ standard output and standard error.  
    - Include a dedicated sidecar container for logging in an application pod.
    - Push logs directly to a backend from within an application
  - Using a sidecar container with the logging agent
    - Streaming sidecar container
    - Sidecar container with a logging agent
  - Exposing logs directly from the application

