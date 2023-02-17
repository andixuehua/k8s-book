# 用户认证

## Kubernetes 中的用户[ ](https://kubernetes.io/zh-cn/docs/reference/access-authn-authz/authentication/#users-in-kubernetes)

所有 Kubernetes 集群都有两类用户：由 Kubernetes 管理的服务账号和普通用户。



## 身份认证策略

你可以同时启用多种身份认证方法，并且你通常会至少使用两种方法：

- 针对服务账号使用服务账号令牌
- 至少另外一种方法对用户的身份进行认证

当集群中启用了多个身份认证模块时，第一个成功地对请求完成身份认证的模块会直接做出评估决定。 API 服务器并不保证身份认证模块的运行顺序。

对于所有通过身份认证的用户，`system:authenticated` 组都会被添加到其组列表中。

与其它身份认证协议（LDAP、SAML、Kerberos、X509 的替代模式等等） 都可以通过使用一个[身份认证代理](https://kubernetes.io/zh-cn/docs/reference/access-authn-authz/authentication/#authenticating-proxy)或[身份认证 Webhoook](https://kubernetes.io/zh-cn/docs/reference/access-authn-authz/authentication/#webhook-token-authentication) 来实现。

