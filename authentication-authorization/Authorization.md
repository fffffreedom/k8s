# Authorization

> https://kubernetes.io/docs/admin/authorization/  

在k8s中，你的请求在被授权（authorized ）之前，你需要先登录，即先通过认证（authenticated）。  

## Determine Whether a Request is Allowed or Denied - apiserver

k8s用apiserver来给API授权。它基于所有的policies，判断请求的属性(Request Attributes, see bellow)是符合，以作出允许或者拒绝该请求的决定。
一个请求的所有部分必须被一些policy允许，才可以被进一步处理。这意味着，对于请求默认是被拒绝的！  

**Although Kubernetes uses the API server, access controls and policies that depend on specific fields of specific kinds of objects 
are handled by Admission Controllers.**  

When multiple authorization modules are configured, each is checked in sequence. If any authorizer approves or denies a request, 
that decision is immediately returned and no other authorizer is consulted. If all modules have no opinion on the request, 
then the request is denied. A deny returns an HTTP status code 403.  

## Request Attributes

Kubernetes reviews only the following API request attributes:  

- user  
The user string provided during authentication.  

- group  
The list of group names to which the authenticated user belongs.  

- "extra"  
A map of arbitrary string keys to string values, provided by the authentication layer.  

- API  
Indicates whether the request is for an API resource.  

- Request path  
Path to miscellaneous non-resource endpoints like /api or /healthz.  

- API request verb  
API verbs get, list, create, update, patch, watch, proxy, redirect, delete, and deletecollection are used for resource requests.
To determine the request verb for a resource API endpoint, see Determine the request verb below.  

- HTTP request verb
HTTP verbs get, post, put, and delete are used for non-resource requests.  

- Resource  
The ID or name of the resource that is being accessed (for resource requests only) – For resource requests using get, update, patch, 
and delete verbs, you must provide the resource name.  

- Subresource  
The subresource that is being accessed (for resource requests only).  

- Namespace  
The namespace of the object that is being accessed (for namespaced resource requests only).  

- API group  
The API group being accessed (for resource requests only). An empty string designates the core API group.

## Determine the Request Verb

To determine the request verb for a resource API endpoint, review the HTTP verb used and whether or not the request acts 
on an individual resource or a collection of resources:  

|HTTP verb|request verb|
|:---|:---|
|POST|create|
|GET, HEAD|get (for individual resources), list (for collections)|
|PUT|update|
|PATCH|patch|
|DELETE|delete (for individual resources), deletecollection (for collections)|

Kubernetes sometimes checks authorization for additional permissions using specialized verbs. For example:  

- PodSecurityPolicy checks for authorization of the use verb on podsecuritypolicies resources in the extensions API group.  

- RBAC checks for authorization of the bind verb on roles and clusterroles resources in the rbac.authorization.k8s.io API group.  

- Authentication layer checks for authorization of the impersonate verb on users, groups, and serviceaccounts in the core API group, 
and the userextras in the authentication.k8s.io API group.  

## Authorization Modules

**授权有多个模块，如下，你可以选择多个授权模块。优先极按出现的顺序决定！**  

修改apiserver的启动参数： `--authorization-mode=MODE`  

### AlwaysAllow ( apiserver的默认配置 )

### AlwaysDeny

### Node

> https://kubernetes.io/docs/admin/authorization/node/  

Node authorization is a special-purpose authorization mode that specifically authorizes API requests made by kubelets.  

To enable the Node authorizer, start the apiserver with **--authorization-mode=Node**.  

### ABAC (Attribute-Based Access Control, 基于属性的访问控制)

> https://kubernetes.io/docs/admin/authorization/abac/  

For mode ABAC, also specify **--authorization-policy-file=SOME_FILENAME.**  

The file format is one JSON object per line. There should be no enclosing list or map, just one map per line.  

### RBAC ( Role-Based Access Control )

> https://kubernetes.io/docs/admin/authorization/  

Role-based access control (RBAC) is a method of regulating access to **computer or network resources** based on the roles of individual users within an enterprise.   

从上面可知，RBAC主要给企业内部的用户对计算和网络资源访问的授权！  

> https://kubernetes.io/docs/admin/authorization/rbac/  

Role-Based Access Control (“RBAC”) uses the “rbac.authorization.k8s.io” API group to drive authorization decisions, allowing admins to dynamically configure policies through the Kubernetes API.  

To enable RBAC, start the apiserver with **--authorization-mode=RBAC**.  

The RBAC API declares four top-level types.  

#### Role and ClusterRole

In the RBAC API, a role contains rules that represent a set of permissions. Permissions are purely additive (there are no “deny” rules). A role can be defined within a namespace with a Role, or cluster-wide with a ClusterRole.  

在RBAC的API中，Role包含了代表一组权限的规则。权限全都是正面的，即全都是允许（没有“拒绝”的规则）。Role的作用域为namespace，ClusterRole是作用域为整个集群。  

- Role  

A Role can only be used to grant access to resources within a single namespace.  

```
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  namespace: default  <== 指定Role的有效namespace为default
  name: pod-reader
rules:
- apiGroups: [""] # "" indicates the core API group
  resources: ["pods"]
  verbs: ["get", "watch", "list"]
```

- ClusterRole  

ClusterRole可用于授予与Role相同的权限，但由于它是作用于整个集群的，可以用来授权对以下资源的访问：  
  - cluster-scoped resources (like nodes)
  - non-resource endpoints (like “/healthz”)
  - namespaced resources (like pods) across all namespaces (needed to run kubectl get pods --all-namespaces, for example)  
  
```
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  # "namespace" omitted since ClusterRoles are not namespaced <===
  name: secret-reader
rules:
- apiGroups: [""]
  resources: ["secrets"]
  verbs: ["get", "watch", "list"]
```

#### RoleBinding and ClusterRoleBinding

A role binding grants the permissions defined in a role to a user or set of users. 
It holds a list of subjects(users, groups, or service accounts), and a reference to the role being granted.  

RoleBinding 对应 Role, ClusterRoleBinding 对应 ClusterRole. ( 但RoleBinding也可以绑定ClusterRole )

- RoleBinding绑定Role  

```
# This role binding allows "jane" to read pods in the "default" namespace.
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: read-pods
  namespace: default
subjects:
- kind: User
  name: jane
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

- RoleBinding也可以绑定ClusterRole  

```
# This role binding allows "dave" to read secrets in the "development" namespace.
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: read-secrets
  namespace: development # This only grants permissions within the "development" namespace.
subjects:
- kind: User
  name: dave
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole <== 也可是ClusterRole
  name: secret-reader
  apiGroup: rbac.authorization.k8s.io
```

- ClusterRoleBinding绑定ClusterRole

```
# This cluster role binding allows anyone in the "manager" group to read secrets in any namespace.
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: read-secrets-global
subjects:
- kind: Group <== anyone in the "manager"
  name: manager
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: secret-reader
  apiGroup: rbac.authorization.k8s.io
```

#### Referring to Resources

Most resources are represented by a string representation of their name, such as “pods”, just as it appears in the URL for the relevant API endpoint. However, some Kubernetes APIs involve a “subresource”, such as the logs for a pod. The URL for the pods logs endpoint is:  
```
GET /api/v1/namespaces/{namespace}/pods/{name}/log
```

In this case, “pods” is the namespaced resource, and “log” is a subresource of pods. To represent this in an RBAC role, use a slash to delimit the resource and subresource. To allow a subject to read both pods and pod logs, you would write:  

```
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  namespace: default
  name: pod-and-pod-logs-reader
rules:
- apiGroups: [""]
  resources: ["pods", "pods/log"]
  verbs: ["get", "list"]
```

Resources can also be referred to by name for certain requests through the resourceNames list.   

#### Aggregated ClusterRoles

As of 1.9, ClusterRoles can be created by combining other ClusterRoles using an aggregationRule. 

```
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: monitoring
aggregationRule:
  clusterRoleSelectors:
  - matchLabels:
      rbac.example.com/aggregate-to-monitoring: "true"
rules: [] # Rules are automatically filled in by the controller manager.
```

### Webhook

> https://kubernetes.io/docs/admin/authorization/webhook/  

A WebHook is an HTTP callback: an HTTP POST that occurs when something happens; a simple event-notification via HTTP POST. A web application implementing WebHooks will POST a message to a URL when certain things happen.  

When specified, mode Webhook causes Kubernetes to query an outside REST service when determining user privileges.  

其实就是向外部的认证系统（鉴权，REST服务）发送一个POST请求，来决定用户的权限。  
