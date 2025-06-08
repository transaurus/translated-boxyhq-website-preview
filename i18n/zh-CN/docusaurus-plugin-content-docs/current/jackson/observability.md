---
title: Observability for Enterprise SSO
description: Enterprise SSO provides first-class observability on the back of OpenTelemetry.
sidebar_label: Observability
---

# 可观测性

Jackson基于OpenTelemetry提供一流的可观测性支持。当前我们支持以下指标，链路追踪和日志功能即将推出。

请查阅[环境变量](./deploy/env-variables.md#opentelemetry-configuration)配置以启用此功能。

## 指标

| Name                        | Description                              | Type  | Tags |
| --------------------------- | ---------------------------------------- | ----- | ---- |
| `jackson.connection.create` | Number of SSO Connection create requests | Count | ---- |
| `jackson.connection.get`    | Number of SSO Connection get requests    | Count | ---- |
| `jackson.connection.delete` | Number of SSO Connection delete requests | Count | ---- |
| `jackson.oauth.authorize`   | Number of oauth authorize requests       | Count | ---- |
| `jackson.oauth.token`       | Number of oauth token requests           | Count | ---- |
| `jackson.oauth.userinfo`    | Number of oauth user info requests       | Count | ---- |

自`v1.35.1`版本起新增以下指标：

| Name                        | Description                              | Type  | Tags |
| --------------------------- | ---------------------------------------- | ----- | ---- |
| `jackson.oauth.authorize.error`   | Number of oauth authorize errors         | Count | **protocol**: `saml`,`oidc`,`saml-federated`,`oidc-federated` ; **login_type**: `sp-initiated`,`idp-initiated` |
| `jackson.oauth.token.error`       | Number of oauth token errors             | Count | **protocol**: `saml`,`oidc`,`saml-federated`,`oidc-federated` ; **login_type**: `sp-initiated`,`idp-initiated` |
| `jackson.oauth.userinfo.error`    | Number of oauth user info errors         | Count | **protocol**: `saml`,`oidc`,`saml-federated`,`oidc-federated` ; **login_type**: `sp-initiated`,`idp-initiated` |
| `jackson.oauth.response.error`    | Number of oauth response errors          | Count | **protocol**: `saml`,`oidc`,`saml-federated`,`oidc-federated` ; **login_type**: `sp-initiated`,`idp-initiated` |

自`v1.39.0`版本起新增以下指标：

| Name                        | Description                              | Type  | Tags |
| --------------------------- | ---------------------------------------- | ----- | ---- |
| `jackson.idfed.authorize`   | Number of identity federation authorize requests         | Count | **protocol**: `saml`,`oidc`,`saml-federated`,`oidc-federated` ; **login_type**: `sp-initiated`,`idp-initiated` |
| `jackson.idfed.authorize.error`       | Number of errors in identity federation authorize requests             | Count | **protocol**: `saml`,`oidc`,`saml-federated`,`oidc-federated` ; **login_type**: `sp-initiated`,`idp-initiated` |
| `jackson.idfed.response`    | Number of identity federation response requests         | Count | **protocol**: `saml`,`oidc`,`saml-federated`,`oidc-federated` ; **login_type**: `sp-initiated`,`idp-initiated` |
| `jackson.idfed.response.error`    | Number of errors in identity federation response requests          | Count | **protocol**: `saml`,`oidc`,`saml-federated`,`oidc-federated` ; **login_type**: `sp-initiated`,`idp-initiated` |

自`v1.41.2`版本起新增以下指标：

| Name                             | Description                              | Type  | Tags                   |
| -------------------------------- | ---------------------------------------- | ----- | ---------------------- |
| `jackson.db.connections.max`     | Maximum number of db connections         | Gauge | **db_name**: `jackson` |
| `jackson.db.connections.total`   | Total number of db connections           | Gauge | **db_name**: `jackson` |
| `jackson.db.connections.idle`    | Number of idle db connections            | Gauge | **db_name**: `jackson` |
| `jackson.db.connections.waiting` | Number of waiting db connections         | Gauge | **db_name**: `jackson` |