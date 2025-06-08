# 升级指南

## 升级至 v1.44.0

请查阅关于ESM库和动态导入问题的说明[此处](./deploy/npm-library.md#)

## 升级至 v1.25.0

`namespace`列的长度已增至256个字符以支持更大值。

此变更影响所有使用`sql`作为数据库引擎的实例。若未执行手动迁移，namespace列将被删除后重新创建。强烈建议在升级前备份数据。

## 升级至 v1.21.0

Google目录同步连接将基于以下授权相关变更获得支持。这些变更将简化Google SCIM的OAuth端点配置流程。

1. 移除了Jackson服务中的`DSYNC_GOOGLE_REDIRECT_URI`、`GOOGLE_REDIRECT_URI`环境变量
2. 移除了npm中的`dsync.providers.google.callbackUrl`配置项
3. npm新增配置选项`dsync.providers.google.authorizePath`和`dsync.providers.google.callbackPath`，这些将与`externalUrl`共同构建完整路径

## 升级至 v1.14.0

我们修改了数据库模式，新增`namespace`列以加速扫描。若您手动管理迁移（例如PlanetScale因MySQL与其差异不支持自动迁移），需运行迁移脚本。

## 升级至 v1.11.0

我们已修复SSO连接(`/api/v1/connections`)的DELETE处理器，现支持通过查询参数传递有效载荷(client/Secret或tenant/product/strategy)。此前要求将有效载荷放在请求体中，这不符合标准且Next.js已不再支持。

## 升级至 v1.9.7

我们发现并修复了v1.9.1版本引入的数据库连接泄漏问题。若您正在使用1.9.1至1.9.6版本，请立即升级至v1.9.7。

## 升级至 v1.8.0

1. 我们新增了自带数据库(BYOD)支持功能。由于需要适配多种数据库，我们在模式设计上进行了多次迭代。最新版MySQL和MariaDB存在字段长度限制问题，为此我们将关键字段长度从1500调整为250。此变更将影响MySQL、MariaDB和MS SQL Server用户的升级流程。若遇问题请联系我们获取指导。若尚未投入生产环境，请在部署前升级至v1.8.0。

## 升级至 v1.3.10

1. 对于npm库用户，原`directorySync`已重命名为`directorySyncController`

## 升级至 v1.3.6

1. `samlSPConfig.get()`方法现改为异步，因为我们引入了全局证书替代租户级证书以简化SAML配置流程

## 升级至 v1.3.0

1. 新增OpenID身份提供商支持
2. 新增API端点(`/api/v1/connections`)用于配置SAML/OIDC SSO连接

> **注意：** 原SAML配置接口`/api/v1/saml/config`已被弃用，将在未来版本中彻底移除

3. 将Jackson服务的环境变量`PRE_LOADED_CONFIG`重命名为[`PRE_LOADED_CONNECTION`](deploy/env-variables.md#pre_loaded_connection)。对于npm用户，`preLoadedConfig`参数更名为`preLoadedConnection`。
4. 若使用npm包，请确保设置[`oidcPath`](deploy/env-variables.md#oidcpath)参数值。

## 备份指南

升级Jackson前请务必进行数据备份。此步骤至关重要——尽管我们会尽力避免缺陷，但偶发问题仍可能出现。备份不仅能帮助您从这些罕见事件中恢复，更应成为灾难恢复流程的常规环节。如需协助，请随时联系我们。