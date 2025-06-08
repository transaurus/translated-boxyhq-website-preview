---
title: Deploy Enterprise SSO on Heroku
sidebar_label: Heroku
---

# Heroku

本指南将引导您在Heroku上配置和部署SAML Jackson。我们将使用**一键部署到Heroku**按钮，该功能允许用户无需离开网页浏览器且几乎无需配置即可将应用部署至Heroku。

若您尚无Heroku账户，请先[注册](https://signup.heroku.com/)。

## 部署SAML Jackson

- Fork SAML Jackson的[GitHub代码库](https://github.com/boxyhq/jackson/fork)。
- 进入fork后的代码库，点击README部分的**Deploy to Heroku**按钮。
- 为您的应用指定一个**名称**。
- 根据[文档](/docs/jackson/deploy/env-variables)填写**Config Vars**配置变量。
- 填写完成后点击**Deploy app**部署按钮。

### 验证部署

访问应用路径`/api/hello`可验证Jackson是否成功启动。若部署成功，您将在页面看到如下输出。

```javascript
{
  name: 'Jules Winnfield';
}
```

## 后续步骤

至此您已在服务器上成功运行SAML Jackson实例！

- 遇到问题？[在此提问](https://discord.gg/uyb7pYt4Pa)
- [登录管理门户](/docs/admin-portal/overview#authentication-methods)
- [添加SAML配置](/docs/admin-portal/enterprise-sso)