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

**授权有多个模块，如下，你可以选择多个授权模块。优先极按出现的顺序决定！  

修改apiserver的启动参数： `--authorization-mode=MODE`  

### AlwaysAllow ( apiserver的默认配置 )
### AlwaysDeny
### Node

### ABAC
### RBAC
### Webhook







