# minikube handbook

```
bogon:~ jonny$ minikube -h
Minikube is a CLI tool that provisions and manages single-node Kubernetes clusters optimized for development workflows.

Usage:
  minikube [command]

Available Commands:
  addons         Modify minikube's kubernetes addons
  cache          Add or delete an image from the local cache.
  completion     Outputs minikube shell completion for the given shell (bash or zsh)
  config         Modify minikube config
  dashboard      Access the kubernetes dashboard running within the minikube cluster
  delete         Deletes a local kubernetes cluster
  docker-env     Sets up docker env variables; similar to '$(docker-machine env)'
  help           Help about any command
  ip             Retrieves the IP address of the running cluster
  logs           Gets the logs of the running instance, used for debugging minikube, not user code
  mount          Mounts the specified directory into minikube
  profile        Profile sets the current minikube profile
  service        Gets the kubernetes URL(s) for the specified service in your local cluster
  ssh            Log into or run a command on a machine with SSH; similar to 'docker-machine ssh'
  ssh-key        Retrieve the ssh identity key path of the specified cluster
  start          Starts a local kubernetes cluster
  status         Gets the status of a local kubernetes cluster
  stop           Stops a running local kubernetes cluster
  update-check   Print current and latest version number
  update-context Verify the IP address of the running cluster in kubeconfig.
  version        Print the version of minikube

Flags:
      --alsologtostderr                  log to standard error as well as files
  -b, --bootstrapper string              The name of the cluster bootstrapper that will set up the kubernetes cluster. (default "kubeadm")
  -h, --help                             help for minikube
      --log_backtrace_at traceLocation   when logging hits line file:N, emit a stack trace (default :0)
      --log_dir string                   If non-empty, write log files in this directory
      --logtostderr                      log to standard error instead of files
  -p, --profile string                   The name of the minikube VM being used.  
                                         	This can be modified to allow for multiple minikube instances to be run independently (default "minikube")
      --stderrthreshold severity         logs at or above this threshold go to stderr (default 2)
  -v, --v Level                          log level for V logs
      --vmodule moduleSpec               comma-separated list of pattern=N settings for file-filtered logging

Use "minikube [command] --help" for more information about a command.
```
## IMPORTANT
**start the specific minikube VM:**
```
minikube start -p VM_NAME
```
**set minikube VM profile:**
```
minikube profile yaml
```
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
## minikube docker-env
The command `minikube docker-env` returns a set of Bash environment variable exports to configure your local environment to re-use the Docker daemon inside the Minikube instance.
- **`eval $(minikube docker-env)`**  
- **`eval $(minikube docker-env -u)`**  
reference:
https://stackoverflow.com/questions/52310599/what-does-minikube-docker-env-mean


