# Security Context

## Configure a Security Context for a Pod or Container

> https://kubernetes.io/docs/tasks/configure-pod-container/security-context/

security context（安全上下文）定义Pod或Container的权限和访问控制设置。安全上下文设置包括：  

- Discretionary Access Control（自主访问控制）: Permission to access an object, like a file, is based on user ID (UID) and group ID (GID).

- Security Enhanced Linux (SELinux): Objects are assigned security labels.

- Running as privileged or unprivileged.

- Linux Capabilities: Give a process some privileges, but not all the privileges of the root user.

- AppArmor: Use program profiles to restrict the capabilities of individual programs.

- Seccomp: Limit a process’s access to open file descriptors.

- AllowPrivilegeEscalation: Controls whether a process can gain more privileges than its parent process.
This bool directly controls whether the no_new_privs flag gets set on the container process. 
AllowPrivilegeEscalation is true always when the container is: 1) run as Privileged OR 2) has CAP_SYS_ADMIN

### 配置级别

Security Context 的目的是限制不可信容器的行为，保护系统和其他容器不受其影响。  
Kubernetes 提供了三种配置 Security Context 的方法：

- Container-level Security Context：仅应用到指定的容器
- Pod-level Security Context：应用到 Pod 内所有容器以及 Volume
- Pod Security Policies（PSP）：应用到集群内部所有 Pod 以及 Volume

