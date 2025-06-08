---
title: Deploy Enterprise SSO on Vercel
sidebar_label: Vercel
---

# Vercel

本指南将引导您在Vercel上配置和部署SAML Jackson。我们将使用Vercel的**一键部署按钮**，该功能允许用户无需离开浏览器且几乎无需配置即可将应用部署至Vercel。

若您尚未拥有Vercel账户，请先[注册](https://vercel.com/signup)。

## 部署SAML Jackson

- Fork SAML Jackson的[GitHub仓库](https://github.com/boxyhq/jackson/fork)。
- 进入您fork的仓库，点击README部分的**Deploy**按钮。
- 在**GIT SCOPE**下选择您的用户名。
- 为项目输入**名称**，选择仓库可见性（`private`或`public`），点击**Create**。
- 根据[文档](/docs/jackson/deploy/env-variables)填写**环境变量**。
- 填写完成后点击**Deploy**按钮开始部署。

### 验证部署

您可以通过访问应用路径`/api/hello`来验证Jackson是否成功启动。若部署成功，页面将显示如下输出内容。

```javascript
{
  name: 'Jules Winnfield';
}
```

## 后续步骤

至此，您已在服务器上成功运行了一个功能完备的SAML Jackson实例！

- 遇到问题？[在此提问](https://discord.gg/uyb7pYt4Pa)
- [登录管理门户](/docs/admin-portal/overview#authentication-methods)
- [添加SAML配置](/docs/admin-portal/enterprise-sso)