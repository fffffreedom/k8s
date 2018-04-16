# 认证 (authentication) 和授权 (authorization)

## 认证

和Web应用不同，RESTful APIs通常是无状态的， 也就意味着不应使用sessions或cookies， 因此每个请求应附带某种授权凭证，
因为用户授权状态可能没通过 sessions 或 cookies维护， 常用的做法是每个请求都发送一个秘密的access token来认证用户， 
由于access token可以唯一识别和认证用户， API 请求应通过HTTPS来防止man-in-the-middle (MitM) 中间人攻击.

下面有几种方式来发送access token：

- HTTP 基本认证: access token 当作用户名发送，应用在access token可安全存在API使用端的场景， 例如，API使用端是运行在一台服务器上的程序。  

- 请求参数: access token 当作API URL请求参数发送，例如 https://example.com/users?access-token=xxxxxxxx， 由于大多数服务器都会保存请求参数到日志，
这种方式应主要用于JSONP 请求，因为它不能使用HTTP头来发送access token.  

- OAuth 2: 使用者从认证服务器上获取基于OAuth2协议的access token， 然后通过HTTP Bearer Tokens发送到API服务器。  

## 授权

在用户认证成功后，你可能想要检查他是否有权限执行对应的操作来获取资源， 这个过程称为 authorization.  

## k8s

### Authenticating

> https://kubernetes.io/docs/admin/authentication/  
> https://k8smeetup.github.io/docs/admin/authentication/  

#### Users in Kubernetes

所有的k8s集群有两类用户：service accounts managed by Kubernetes, and normal users.  
service account（SA）由k8s管理的，而普通用户由独立的外部服务管理。  

Kubernetes does not have objects which represent normal user accounts. Regular users cannot be added to a cluster through an API call.  
**即k8s没有代码普通用户的对象，不能通过API调用来给集群添加普通用户！**  

service account是由k8s API管理的的用户，它们和特定的namespace绑定，可以由API server自动创建，或者通过API调用手动创建。  

Service accounts are tied to a set of credentials stored as Secrets, which are mounted into pods allowing in-cluster processes to talk to the Kubernetes API.  

SA是给Pod提供身份证明，存储在Secrets文件中，pod启动时，Secrets会被挂载pod中，使Pod进程可以访问K8s的API。  

API requests are tied to either a normal user or a service account, or are treated as anonymous requests. This means every process inside or outside the cluster, from a human user typing kubectl on a workstation, to kubelets on nodes, to members of the control plane, must authenticate when making requests to the API server, or be treated as an anonymous user. 

API request分为三类：  
- normal user request
- service account request
- anonymous request（除了上述两种request）

#### Authentication strategies

Kubernetes uses client certificates, bearer tokens, an authenticating proxy, or HTTP basic auth to authenticate API requests through authentication plugins.  

Kubernetes使用客户端证书，承载令牌，**身份验证代理**或HTTP基本身份验证来通过身份验证插件对API请求进行身份验证。  

As HTTP requests are made to the API server, plugins attempt to associate the following attributes with the request:  
- Username 
- UID
- Groups
- Extra fields

All values are opaque to the authentication system and only hold significance when interpreted by an authorizer.  
You can enable multiple authentication methods at once. You should usually use at least two methods:  

- service account tokens for service accounts
- at least one other method for user authentication

当多个认证模块被使用时，如果第一个模块认证成功，就不会再进行认证，API server不能保证最先使用哪个认证模块。  

The system:authenticated group is included in the list of groups for all authenticated users.  

Integrations with other authentication protocols (LDAP, SAML, Kerberos, alternate x509 schemes, etc) can be accomplished using an **authenticating proxy or the authentication webhook.**  

#### 认证方法

- X509 Client Certs
- Static Token File
- Bootstrap Tokens
- Static Password File
- Service Account Tokens
- OpenID Connect Tokens
- Webhook Token Authentication
- Authenticating Proxy
- Keystone Password

#### Anonymous requests

When enabled, requests that are not rejected by other configured authentication methods are treated as anonymous requests, and given a username of system:anonymous and a group of system:unauthenticated.  

apiserver的`--anonymous-auth`选项使能！  

### Authorization

> https://kubernetes.io/docs/admin/authorization/

In Kubernetes, you must be authenticated (logged in) before your request can be authorized (granted permission to access).

## reference
http://www.yiichina.com/doc/guide/2.0/rest-authentication  
