---
title: Environment Variables (Enterprise SSO)
sidebar_label: Environment Variables
description: Environment Variables for Enterprise SSO
---

# 环境变量

以下环境变量仅适用于Jackson服务。若使用npm库，请参考下方初始化库时的配置选项。

## 通用配置

### **HOSTNAME**

如需绑定特定主机名，可定义`HOSTNAME`环境变量。
例如使用Docker健康检查`wget -q --spider http://localhost:{PORT}/api/health || exit 1`时，需设置`HOSTNAME=0.0.0.0`以监听本地地址。

### **HOST_URL**

服务绑定地址。
默认值：`localhost`

### **PORT**

服务绑定端口。
默认值：`5225`

### **EXTERNAL_URL**

该服务的公开访问URL，用于内部构造SAML/OIDC身份提供商返回授权响应的回调地址。

默认值：`http://{HOST_URL}:{PORT}`

NPM库选项：`externalUrl`

### **ACS_URL**

自定义断言消费服务(ACS)地址，用于指定SAML请求中发送给身份提供商的ACS地址。也可为每个SAML连接单独配置。

自版本`1.40.1`起可用。

NPM库选项：`acsUrl`

### **JACKSON_API_KEYS**

以逗号分隔的API密钥列表，用于验证API请求(`/api/v1/**`)的合法性。

示例：`JACKSON_API_KEYS=key1,key2,key3`

API请求需在`Authorization`头中按此格式提供密钥：`Api-Key key1`

### **SAML_AUDIENCE**

该设置值(即Jackson的SP实体ID)用于验证SAML响应的目标接收方。客户需在其身份提供商端的SAML应用中配置相同值（区分大小写，不必是真实URL）。设置后请勿修改，否则需客户重新配置SAML应用。

默认值：`https://saml.boxyhq.com`

NPM库选项：`samlAudience`

### **samlPath**

> **_注意：_** 仅适用于npm库。

SAML身份提供商返回[saml响应](./npm-library#saml-response)的ACS路径。使用npm包时需设置此值。

NPM库选项：`samlPath`

示例：`/api/oauth/saml`

### **oidcPath**

> **_注意：_** 仅适用于npm库。

OpenID Connect身份提供商返回授权响应的`redirect_uri`。使用npm包时需设置此值。

NPM库选项：`oidcPath`

示例：`/api/oauth/oidc`

### **IDP_DISCOVERY_PATH**

该路径用于在同一个`tenant`和`product`下存在多个SSO连接时展示身份提供商（IdP）选择界面。在IdP发起的SAML登录场景中（多个应用使用同一个SAML IdP），该路径也用于展示应用选择界面。使用npm包时需要设置此路径。

NPM库选项：`idpDiscoveryPath`

例如：`/idp/select` —— 您可以在[这里](https://github.com/boxyhq/jackson/blob/main/pages/idp/select.tsx)找到IdP/应用选择的实现示例。

### **CLIENT_SECRET_VERIFIER**

当SAML流程中使用`tenant`和`product`（且未使用PKCE）时，系统会使用`dummy`作为`client_id`和`client_secret`的占位符。这不会引发安全问题，因为SAML具有租户隔离特性，您的身份提供商将阻止任何尝试登录您SAML租户的行为。但为了增强安全性，您应设置`CLIENT_SECRET_VERIFIER`为一个随机密钥，并在OAuth 2.0流程中使用该值作为`client_secret`。

默认值：`dummy`

NPM库选项：`clientSecretVerifier`

### **IDP_ENABLED**

设置为true可启用SAML的IdP发起登录。虽然SP发起登录是推荐流程，但有时仍需支持IdP登录。

默认值：`false`

NPM库选项：`idpEnabled`

### **PUBLIC_KEY**

这是用于签署SAML请求的私钥对应的公钥。Jackson要求公钥采用base64编码格式。

NPM库选项：`certs.publicKey`

### **PRIVATE_KEY**

这是用于签署SAML请求的私钥。Jackson要求私钥采用base64编码格式。

NPM库选项：`certs.privateKey`

可通过以下命令生成公私钥对：

```bash
openssl req -x509 -newkey rsa:2048 -keyout key.pem -out public.crt -sha256 -days 365 -nodes

# Convert the public key to base64
cat public.crt | base64

# Convert the private key to base64
cat key.pem | base64
```

### **FLATTEN_RAW_CLAIMS**

如需将所有原始声明（raw claims）平铺到用户档案中（而非分组到profile.raw对象里），可设置此配置。某些不支持嵌套对象的OIDC提供商需要此配置。

默认值：`false`

NPM库选项：`flattenRawClaims`

## OpenID 配置

为支持OpenID流程，需设置用于签署ID令牌JWT的算法和密钥。

### **OPENID_JWS_ALG**

用于签署id_token的算法。Jackson使用[jose](https://github.com/panva/jose)生成ID令牌，支持的算法详见[此处](https://github.com/panva/jose/issues/114#digital-signatures)。

默认值：`RS256`

NPM库选项：`openid.jwsAlg`

### **OPENID_RSA_PRIVATE_KEY**

私钥的base64编码值。生成方法：

```bash
openssl genrsa -out private-key.pem 3072
openssl pkcs8 -topk8 -inform PEM -outform PEM -nocrypt -in private-key.pem -out private_key.pem
cat private_key.pem | base64
```

NPM库选项：`openid.jwtSigningKeys.private`

### **OPENID_RSA_PUBLIC_KEY**

公钥的base64编码值。可通过私钥生成公钥，如下所示：

```bash
openssl rsa -in private_key.pem -pubout -out public_key.pem
cat public_key.pem | base64
```

NPM库选项：`openid.jwtSigningKeys.public`

### **OPENID_REQUEST_PROFILE_SCOPE**

将此选项设为`false`后，授权请求中将不再自动包含profile范围参数发送至OIDC提供商。这使得应用能够仅向登录用户请求最小必需的授权权限。应用可显式地在请求中向Jackson发送profile范围参数，该参数将被转发至OIDC提供商。

默认值：`true`

NPM库选项：`openid.requestProfileScope`

### **OPENID_SUBJECT_PREFIX**

启用此选项后，OIDC声明中的`id`和`subject`将添加`${tenant}:${product}:`前缀，以防止潜在的标识冲突或恶意身份提供商攻击。

默认值：`true`

NPM库选项：`openid.subjectPrefix`

## 数据库配置

### **DB_ENGINE**

支持取值：`redis`、`sql`、`mongo`、`mem`、`planetscale`、`dynamodb`

默认值：`sql`

NPM库选项：`db.engine`

> **_注意：_** 使用DynamoDB时需额外设置`AWS_ACCESS_KEY_ID`和`AWS_SECRET_ACCESS_KEY`。区域与容量单位等高级选项请参阅[此章节](#db_dynamodb_region)

> **_注意：_** `mem`数据库（内存数据库）仅适用于本地测试Jackson配置，不可用于生产环境。在Vercel等无服务器部署中，由于每次API调用都是全新的lambda调用上下文，内存数据库无法跨请求持久化数据。

### **DB_TYPE**

仅当DB_ENGINE为sql时需配置。支持取值：`postgres`、`mysql`、`mariadb`、`mssql`

默认值：`postgres`

NPM库选项：`db.type`

### **DB_URL**

数据库连接URL。若使用自签名证书，需将`sslmode=require`替换为`sslmode=no-verify`，因为严格模式下自签名证书会被视为未授权。同时需设置`DB_SSL=true`和`DB_SSL_REJECT_UNAUTHORIZED=false`（详见下方环境变量说明）。

示例：`postgres://postgres:postgres@localhost:5432/postgres` 或 `postgres://postgres:postgres@localhost:5432/postgres?sslmode=no-verify`

`mssql`连接URL格式：`sqlserver://localhost:1433;database=<数据库名>;username=<用户名>;password=<密码>;encrypt=true`

> **_注意：_** 使用DynamoDB时需额外设置`AWS_ACCESS_KEY_ID`和`AWS_SECRET_ACCESS_KEY`。区域与容量单位等高级选项请参阅[此章节](#db_dynamodb_region)

NPM库选项：`db.url`

### **DB_SSL**

当数据库需通过SSL连接时必须设为`true`（若需通过公网访问数据库，强烈建议启用SSL）。

默认值：`false`

### **DB_SSL_REJECT_UNAUTHORIZED**

使用自签名证书时需设为`false`，否则会因证书权威机构校验失败被拒绝连接。

默认值：`true`

### **DB_MANUAL_MIGRATION**

使用Jackson Docker镜像时，设为true可触发迁移脚本执行。

也可选择运行`./npm/package.json`中的npm脚本来执行特定数据库的迁移操作。

请确保已正确设置 `DB_ENGINE` 环境变量，因为迁移脚本会检查该值以执行正确的命令和迁移文件。

NPM库选项: `db.manualMigration`
默认值: `false`

### **DB_TTL**

代码、会话和令牌存储的生存时间（单位：秒）

默认值: `300`

NPM库选项: `db.ttl`

### **DB_CLEANUP_LIMIT**

限制TTL条目清理的数量

默认值: `1000`

NPM库选项: `db.cleanupLimit`

### **DB_ENCRYPTION_KEY**

如需静态数据加密，请指定32字节密钥（base64编码）

可使用openssl生成随机32字节密钥：

```bash
openssl rand -base64 32
```

NPM库选项: `db.encryptionKey`

### **PGSSLMODE**

如果在Heroku部署Postgres（或使用自签名证书），请将此值设为`no-verify`。详见[Heroku文档](https://devcenter.heroku.com/articles/connecting-heroku-postgres#connecting-in-node-js)

### **DB_DYNAMODB_REGION**

使用DynamoDB时可指定区域

默认值: `us-east-1`

NPM库选项: `db.dynamodb.region`

### **DB_DYNAMODB_RCUS**

使用DynamoDB时可指定读取容量单位。详见[AWS文档](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/HowItWorks.ReadWriteCapacityMode.html)

默认值: 5

NPM库选项: `db.dynamodb.readCapacityUnits`

### **DB_DYNAMODB_WCUS**

使用DynamoDB时可指定写入容量单位

默认值: 5

NPM库选项: `db.dynamodb.writeCapacityUnits`

## Webhook配置

### **WEBHOOK_URL**

指定接收SSO和目录连接事件的webhook URL

NPM库选项: `webhook.endpoint`

### **WEBHOOK_SECRET**

指定用于签名webhook负载的密钥，用于验证负载真实性

NPM库选项: `webhook.secret`

需同时配置`WEBHOOK_URL`和`WEBHOOK_SECRET`才能启用webhook事件

## 预加载连接配置

### **PRE_LOADED_CONNECTION**

若仅需单个租户或少量预配置租户，此配置可帮助读取和加载IdP（包括OpenID和SAML）连接。该功能与mem数据库引擎配合良好，无需配置外部数据库（但也可兼容其他数据库）。需指定包含特定格式文件的目录路径（绝对或相对路径），格式说明详见下一节。更多细节请参阅[此章节](./pre-loaded-connections.md)

NPM库选项: `preLoadedConnection`

## OpenTelemetry配置

Jackson 支持通过 OpenTelemetry 实现可观测性。以下是可配置的环境变量（以及[官方支持的其他变量](https://github.com/open-telemetry/opentelemetry-specification/blob/main/specification/protocol/exporter.md)）：

### **OTEL_EXPORTER_OTLP_ENDPOINT** 或 **OTEL_EXPORTER_OTLP_METRICS_ENDPOINT**

导出器发送指标的目标 URL。

示例：`https://ingest.lightstep.com:443/metrics/otlp/v0.6`

### **OTEL_EXPORTER_OTLP_HEADERS** 或 **OTEL_EXPORTER_OTLP_METRICS_HEADERS**

与端点相关的请求头，可用于指定服务商的身份验证信息。

示例：`lightstep-access-token=<token>,...`

### **OTEL_EXPORTER_OTLP_PROTOCOL** 或 **OTEL_EXPORTER_OTLP_METRICS_PROTOCOL**

传输协议。可选值必须为：`grpc`、`http/protobuf` 或 `http/json`。

### **OTEL_EXPORTER_DEBUG**

设置为 `true` 可启用 OpenTelemetry 的调试日志。仅限本地调试使用。

## 管理门户配置

以下变量用于为管理门户启用[魔法链接](https://next-auth.js.org/providers/email)认证功能。**SMTP_** 变量用于发送包含一次性登录魔法链接的邮件。

### **SMTP_HOST**

SMTP 主机地址，例如 `smtp.example.com`。

### **SMTP_PORT**

SMTP 服务器端口，例如 `587`。

### **SMTP_USER**

SMTP 服务器的用户名。

### **SMTP_PASSWORD**

SMTP 服务器的密码。

### **SMTP_FROM**

发送邮件时使用的发件人地址，例如：`noreply@example.com`。

### **NEXTAUTH_URL**

本地运行时指向本地服务器：`http://localhost:5225`。部署到生产环境时，请设置为站点的正式 URL。详情参见[此处](https://next-auth.js.org/configuration/options#nextauth_url)。

### **NEXTAUTH_SECRET**

设置为随机字符串。可通过 `openssl rand -base64 32` 生成。该密钥用于加密 JWT 和哈希邮件验证令牌。详情参见[此处](https://next-auth.js.org/configuration/options#nextauth_secret)。

### **NEXTAUTH_ACL**

设置为逗号分隔的电子邮件地址或通配符模式，例如：`tonystark@gmail.com,*@marvel.com`。不匹配的邮箱地址将被拒绝访问。若未指定值，则拒绝所有访问。

### **NEXTAUTH_ADMIN_CREDENTIALS**

设置为 `email:password` 格式的逗号分隔字符串以启用管理门户登录，例如 `NEXTAUTH_ADMIN_CREDENTIALS=deepak@boxyhq.com:Password123`。若未指定值，则拒绝所有访问。

### **ADMIN_PORTAL_HIDE_AUDIT_LOGS**

设置为 `true` 可隐藏侧边栏中的审计日志产品。

默认值：`false`

### **ADMIN_PORTAL_HIDE_IDENTITY_FEDERATION**

设置为 `true` 可隐藏侧边栏中的身份联邦产品。

默认值：`false`

### **ADMIN_PORTAL_SSO_TENANT**

该参数将作为管理门户自身登录所用SSO连接（通过设置页添加）的租户标识。请设置为不易与企业主SSO连接冲突的值。

默认值：`_jackson_boxyhq`

### **ADMIN_PORTAL_SSO_PRODUCT**

该参数将作为管理门户自身登录所用SSO连接（通过设置页添加）的产品标识。请设置为不易与企业主SSO连接冲突的值。

默认值：`_jackson_admin_portal`

### **RETRACED_HOST_URL**

若需通过管理门户操作审计日志服务([Retraced](https://github.com/retracedhq/retraced))，请将此环境变量设置为该服务URL。

### **RETRACED_EXTERNAL_URL**

若需通过管理门户操作审计日志服务([Retraced](https://github.com/retracedhq/retraced))，请将此环境变量设置为该服务的公网URL。若该值与上述`RETRACED_HOST_URL`相同，可忽略此参数，系统将自动沿用`RETRACED_HOST_URL`的值。

默认值：当未定义`RETRACED_EXTERNAL_URL`时，自动设置为`RETRACED_HOST_URL`的值。

### **RETRACED_ADMIN_ROOT_TOKEN**

需设置Retraced的管理员根令牌，以便连接Retraced并获取项目及审计日志。

## 匿名分析

### **BOXYHQ_NO_TELEMETRY** 或 **DO_NOT_TRACK**

将其中任一参数设为`1`或`true`可关闭匿名分析功能。我们仅每天统计一次使用事件，且不包含任何可识别用户身份的信息。

## SSO追踪

### **SSO_TRACES_DISABLE**

设为`true`可禁用SSO追踪收集功能。

NPM库选项：`ssoTraces.disable`

### **SSO_TRACES_REDACT**

设为`true`可对SSO追踪中的个人信息进行脱敏处理。

NPM库选项：`ssoTraces.redact`

### **SSO_TRACES_TTL**

设置SSO追踪数据的存储时长（单位：小时）。默认值：`7天`

NPM库选项：`ssoTraces.ttl`

## 目录同步

以下环境变量用于配置目录同步功能。

`DSYNC_GOOGLE_`前缀的环境变量仅在使用Google Workspace作为目录同步提供商时生效。

### **DSYNC_WEBHOOK_BATCH_SIZE**

启用目录同步事件的批量处理功能。该值表示批量合并的事件数量（替代单事件发送模式）。需配置定时任务来[处理队列中的事件](/docs/directory-sync/api-reference#batch-processing-events)。若未部署在边缘环境，也可直接设置下方的`DSYNC_WEBHOOK_BATCH_CRON_INTERVAL`参数。

### **DSYNC_WEBHOOK_BATCH_CRON_INTERVAL**

设置处理队列事件的定时任务间隔（单位：秒）。默认情况下定时任务处于禁用状态。

NPM库选项：`dsync.webhookBatchCronInterval`

### **DSYNC_GOOGLE_CRON_INTERVAL**

设置Google Workspace同步的定时任务间隔时间（以秒为单位）。默认情况下定时任务处于禁用状态。

NPM库选项：`dsync.providers.google.cronInterval`

### **DSYNC_GOOGLE_CLIENT_ID**

Google身份验证客户端ID。

NPM库选项：`dsync.providers.google.clientId`

### **DSYNC_GOOGLE_CLIENT_SECRET**

Google身份验证客户端密钥。

NPM库选项：`dsync.providers.google.clientSecret`

### **dsync.providers.google.authorizePath**

Google SCIM的授权端点路径。

NPM库选项：`dsync.providers.google.authorizePath`

例如：`/api/scim/oauth/authorize`

### **dsync.providers.google.callbackPath**

完成Google SCIM授权请求后的回调路径。

NPM库选项：`dsync.providers.google.callbackPath`

例如：`/api/scim/oauth/callback`

## 企业级功能

### **BOXYHQ_LICENSE_KEY**

联系我们获取企业级功能许可证（包含门户品牌定制、身份联邦与Ory集成）。

## 企业级功能 - Ory集成

### **ENTERPRISE_ORY_SDK_TOKEN**

您的Ory网络会话令牌。可通过Ory CLI获取并复制会话令牌。

### **ENTERPRISE_ORY_PROJECT_ID**

您的Ory网络项目ID。

## 日志配置选项

### **LOG_FILE**

指定日志输出文件路径

### **LOG_LEVEL**

日志级别 - "fatal" | "error" | "warn" | "info" (默认) | "debug" | "trace"