

-

## 架构

这是istio的安全架构图，

将单一应用程序分解为微服务可提供各种好处，包括更好的灵活性、可伸缩性以及服务复用的能力。但是，微服务也有特殊的安全需求：

- 为了抵御中间人攻击，需要流量加密。
- 为了提供灵活的服务访问控制，需要双向 TLS 和细粒度的访问策略。
- 要确定谁在什么时候做了什么，需要审计工具。

![安全架构](.assets/arch-sec.svg)

## 认证方式

- Peer authentication：用于服务到服务的认证，以验证进行连接的客户端。Istio 提供[双向 TLS](https://en.wikipedia.org/wiki/Mutual_authentication) 作为传输认证的全栈解决方案，无需更改服务代码就可以启用它。这个解决方案：
  - 为每个服务提供强大的身份，表示其角色，以实现跨群集和云的互操作性。
  - 保护服务到服务的通信。
  - 提供密钥管理系统，以自动进行密钥和证书的生成，分发和轮换。
- Request authentication：用于最终用户认证，以验证附加到请求的凭据。 Istio 使用 JSON Web Token（JWT）验证启用请求级认证，并使用自定义认证实现或任何 OpenID Connect 的认证实现（例如下面列举的）来简化的开发人员体验。

### Peer authentication

Peer 认证策略指定 Istio 对目标工作负载实施的双向 TLS 模式。支持以下模式：

- PERMISSIVE：工作负载接受双向 TLS 和纯文本流量。此模式在迁移因为没有 sidecar 而无法使用双向 TLS 的工作负载的过程中非常有用。一旦工作负载完成 sidecar 注入的迁移，应将模式切换为 STRICT。

  默认为PERMISSIVE，所有无需配置也可以访问

- STRICT： 工作负载仅接收双向 TLS 流量。

- DISABLE：禁用双向 TLS。 从安全角度来看，除非您提供自己的安全解决方案，否则请勿使用此模式。

### Request authentication

Request 认证策略指定验证 JSON Web Token（JWT）所需的值。 这些值包括：

- token 在请求中的位置
- 请求的 issuer
- 公共 JSON Web Key Set（JWKS）

## 授权

From 来源

To 目的

When 条件



#### 条件模式

宽容模式

严格模式 STRICT

关闭 DISABLE



## 环境

```
 istioctl manifest apply --set profile=demo \
  --set values.meshConfig.enableAutoMtls=false
```

## 问题

网格验证策略没有crd

```
kubectl get crd |grep policies.authentication.istio.io
```

