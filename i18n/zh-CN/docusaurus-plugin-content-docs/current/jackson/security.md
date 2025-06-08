# 安全性

一般而言，大多数OAuth 2.0安全准则同样适用于Jackson，但仍存在一些细微差异。

## 多租户特性

OAuth 2.0通常是单一实例，所有用户均可通过同一实例登录。而SAML基于特定实例，需要客户在其身份提供商处单独配置。这意味着SAML实例的访问权限仅限已配置的租户，从而为登录流程增添了额外的安全层。

## 客户端密钥

配置API会返回`client_id`和`client_secret`，您可为每个客户保存这些凭证以初始化OAuth 2.0流程。为简化操作，我们也允许直接使用`tenant`和`product`参数，这样无需存储额外数据。采用此方式时，建议配置[CLIENT_SECRET_VERIFIER](./deploy/env-variables.md#client_secret_verifier)环境变量，避免使用默认的`dummy`值。虽然`client_secret`实际被滥用的风险较低（如下节所述，令牌有效期极短），但仍建议将其设置为真正的机密值。

对于客户端登录场景，强烈建议使用Jackson支持的PKCE流程。该机制可在不暴露`client_secret`的前提下确保登录安全。

## 令牌机制

Jackson生成的令牌有效期极短（默认为5分钟），因此即使`client_secret`泄露，实际滥用风险也极低。您的SAML提供商将确保不存在未授权的访问。