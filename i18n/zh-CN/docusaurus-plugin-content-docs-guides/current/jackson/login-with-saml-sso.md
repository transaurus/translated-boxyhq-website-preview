---
title: UI Best Practices for Log in with SAML Single Sign-On
description: UI Best Practices for Log in with SAML Single Sign-On
sidebar_label: Log in with SAML SSO
---

# 使用SAML单点登录

让我们看看企业应用如何将SAML SSO集成到其认证流程中。

## 服务提供商发起的SSO

在实现应用的SP发起的SAML SSO登录时，您可采用多种方式。

- 专设SAML SSO页面
- 动态密码字段
- 租户子域名
- 分离邮箱与密码字段

### 专设SAML SSO页面

此方式会在登录页面设置"使用SAML SSO登录"按钮。用户点击后将跳转至SAML SSO专属页面。

**Postman**即采用此方式为其企业用户启用SAML SSO。

<img src="/images/guides/sso/postman-1.png" style={{ border: '1px solid black' }} />

<img src="/images/guides/sso/postman-2.png" style={{ border: '1px solid black' }} />

用户流程大致如下：

- 用户访问登录页
- 点击"使用SAML SSO登录"按钮
- 应用重定向至SAML SSO页面
- 用户输入邮箱或租户ID
- 应用校验域名/租户ID是否启用SSO
- 若启用，则重定向至对应身份提供商(IdP)

### 动态密码字段

此方式会在域名启用SSO时隐藏密码字段。

**Dropbox**的登录表单即采用此方案。

<img src="/images/guides/sso/dropbox-1.png" style={{ border: '1px solid black' }} />

<img src="/images/guides/sso/dropbox-2.png" style={{ border: '1px solid black' }} />

用户流程大致如下：

- 用户访问登录页
- 输入邮箱地址
- 应用校验域名是否启用SSO
- 若启用则隐藏密码字段
- 用户提交登录表单
- 应用重定向至身份提供商(IdP)

### 租户子域名

可为每个租户配置子域名。先展示租户ID输入表单，随后将用户重定向至子域名页面，显示该租户启用的认证机制。

**Freshchat/Freshworks**的登录表单即采用此方案。

<img src="/images/guides/sso/freshchat-1.png" style={{ border: '1px solid black' }} />

<p align="center">
  <img
    src="/images/guides/sso/freshchat-2.png"
    width="70%"
    style={{ border: '1px solid black' }}
  />
</p>

用户流程大致如下：

- 用户访问登录页
- 输入租户名称
- 应用验证租户存在性并重定向至租户子域名页
- 子域名页面展示该租户启用的认证机制
- 若启用SSO，用户可选择"使用SAML SSO登录"

### 分离邮箱与密码字段

此方式将登录流程分阶段进行：先展示无密码字段的邮箱表单，用户提交后，应用检查该用户启用的认证机制。

**Box**即采用此认证流程。

<img src="/images/guides/sso/box-1.png" style={{ border: '1px solid black' }} />

用户流程大致如下：

- 用户访问登录页
- 输入邮箱并提交表单
- 应用检查邮箱是否启用SSO
- 若启用则直接重定向至身份提供商(IdP)，而非进入密码流程