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

### Authorization

> https://kubernetes.io/docs/admin/authorization/

In Kubernetes, you must be authenticated (logged in) before your request can be authorized (granted permission to access).

## reference
http://www.yiichina.com/doc/guide/2.0/rest-authentication  
