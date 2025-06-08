---
slug: boxyhq-cerbos-merging-sso-and-authorization
title: 'BoxyHQ + Cerbos: Merging SSO and Authorization'
author: Kiran K
author_title: Senior Developer @ BoxyHQ
author_url: https://twitter.com/tokirankrishnan
author_image_url: /images/authors/kiran.jpg
tags_disabled: [enterprise-readiness, saml, saml-jackson, sso]
image: /img/blog/merging-sso-and-authorization.png
---

本文将演示如何为Next.js应用启用基于SAML单点登录协议的Okta企业级SSO登录，并将身份提供者中的基于角色的访问控制直接传递至Cerbos。

我们将使用BoxyHQ的开源[企业级SSO](/enterprise-sso)解决方案（名为SAML Jackson）与Okta对接。本示例应用将遵循最小化界面原则，仅保留必要功能接口。

## SAML单点登录简介

安全断言标记语言(SAML)专为传统Web应用设计于21世纪初，旨在通过将认证联邦至身份提供者(IdP)，为应用提供无缝用户体验。

这使得应用无需维护用户身份体系，仅需将浏览器重定向至IdP。IdP完成用户认证后，会返回关于已登录用户的安全断言。

该断言实质上是种令牌，向应用声明用户已在IdP完成认证，且断言在有效期内持续生效。您可在此处深入了解SAML及其他SSO协议。

当需要为企业级应用提供单点登录(SSO)体验时，SAML仍是极受欢迎的选择。微软提供了详尽的SAML认证解析和深度技术文档。

## 使用SAML的优势

### 增强安全性

SAML本质是安全标准，通过集中式安全环境完成认证，为服务增加企业客户普遍要求的安全防护层。

### 优化用户体验

对用户而言，SAML只需单次登录即可通过仪表盘一键访问所有外部服务，既节省时间又提升产品使用体验。

### 降低成本

非SAML方案需跨服务维护账户信息，而SAML方案中这些均由身份提供者统一管理。

## BoxyHQ简介

[BoxyHQ](https://boxyhq.com/)通过开源项目实现并维护SAML的服务提供者功能，将前文所述的所有复杂技术细节封装在更普及的OAuth 2.0协议之后。

## Cerbos简介

[Cerbos](https://cerbos.dev/)是开源核心、语言无关、可扩展的授权解决方案，通过编写上下文感知的访问控制策略，简化应用资源的用户权限管理与实现。

本教程使用BoxyHQ作为示例Next.js应用与Okta身份提供者之间的服务提供层，展示SSO登录期间同步角色和权限的优势。

## 示例应用概览

该示例应用是使用[SAML Jackson实现SAML SSO认证](https://boxyhq.com/docs/jackson/overview)和[Cerbos进行授权](https://docs.cerbos.dev/cerbos/latest/quickstart.html)的简易Next.js应用。

应用包含两个页面：

- `/` - 显示已认证用户档案及Cerbos API返回的授权决策
- `/saml-connection` - 更新SAML连接配置

示例应用运行于3000端口。SAML Jackson和Cerbos分别运行在Docker容器的5225和3593端口。

您可选择使用Cerbos云服务版本，相关配置指令变化不大，但为保持文章简洁未予赘述。

示例应用配置了两种角色：`app-admin` 和 `app-user`。

- `app-admin` 角色拥有所有资源的访问权限。
- `app-user` 角色仅能访问用户自身拥有的资源。

查看 `cerbos/policies/contact.yaml` 文件可了解示例应用的策略定义。您可以尝试修改策略，观察应用中授权决策的变化。

下图展示了示例应用的架构。

![示例应用架构](/img/blog/cerbos-boxyhq/1.png)

### 1. 配置示例应用

克隆示例应用仓库：

```bash
git clone https://github.com/boxyhq/jackson-cerbos
```

进入示例应用目录并运行 `npm install` 安装依赖项。

```bash
cd jackson-cerbos
```

```bash
npm install
```

在浏览器中访问 `http://localhost:3000`，您将看到以下页面。

![示例应用首页](/img/blog/cerbos-boxyhq/2.png)

这表明示例应用已成功运行，您可以继续下一步操作。

仓库中的[README](https://github.com/boxyhq/jackson-cerbos/blob/main/README.md)文件包含示例应用的所有详细信息。若本文有任何不清楚之处，可参考该文件。

### 2. 在Okta中配置SAML应用

下一步是在Okta中设置SAML应用。若您没有Okta账户，可创建免费开发者账户[https://developer.okta.com/signup/](https://developer.okta.com/signup/)。

您也可以使用其他SAML身份提供商（如Azure AD、OneLogin、Auth0等），配置步骤与Okta类似。

本文将使用Okta作为身份提供商。

若不熟悉如何在Okta创建SAML应用，可按照[此处](https://boxyhq.com/docs/jackson/sso-providers/okta)的说明操作。

您需要使用来自[http://localhost:5225/.well-known/saml-configuration](http://localhost:5225/.well-known/saml-configuration)的**断言消费者服务URL**和**实体ID**。

暂时无需配置角色属性，待确保SSO登录流程正常工作后，我们将在后续步骤中配置。

接着从Okta应用下载**SAML元数据XML文件**，下一步骤将需要此文件。

### 3. 在SAML Jackson中创建SAML连接

完成Okta的SAML应用配置后，需在SAML Jackson中创建SAML连接。

在浏览器中访问[http://localhost:3000/saml-connection](http://localhost:3000/saml-connection)，将上一步下载的SAML元数据XML文件内容粘贴至**XML元数据**字段，点击**创建SAML连接**。

若页面显示**SAML SSO已启用**状态，则表示SAML连接创建成功。

### 4. 通过SAML SSO登录

完成SAML连接配置后，即可尝试通过SAML SSO登录。

在浏览器中访问[http://localhost:3000/login](http://localhost:3000/login)，输入电子邮件地址并点击**通过SAML SSO继续**。

您将被重定向至Okta登录页面。输入凭据后点击**登录**。

您将被重定向回示例应用首页，并看到已认证用户的个人资料。

这表示您已成功通过SAML SSO登录。

由于尚未配置角色属性，所有用户默认将被分配**app-user**角色。

### 5. 在Okta应用中配置群组属性

现在我们已经确保SSO登录流程正常工作，可以在Okta中配置群组属性。

群组属性允许您将身份提供商中的群组映射到应用中的角色。

#### 在Okta中创建群组

首先需要在Okta中创建群组。从左导航菜单进入目录 > 群组。

点击**添加群组**，输入群组**名称**和**描述（可选）**

接着创建两个新群组：**app-admin**和**app-user**。

![创建okta群组](/img/blog/cerbos-boxyhq/3.png)

创建完成后，您将在群组页面看到列出的群组。

接下来需要将用户添加到群组。进入刚创建的群组，点击**人员**标签页。

点击**分配人员**按钮，选择要添加到群组的用户。

对第二个群组重复相同步骤。在Okta中配置群组属性

现在Okta中已创建群组，需要配置群组属性。

进入Okta应用的**SAML设置**标签页，点击**编辑**。

点击**配置SAML**标签页，滚动至**群组属性声明**部分。

点击**添加其他**并输入以下值

- 名称：groups
- 名称格式：未指定
- 过滤器：以**app-**开头

![okta群组声明](/img/blog/cerbos-boxyhq/4.png)

注意过滤器中的**app-**前缀。这将确保Okta的SAML响应中仅返回以**app-**开头的群组。如果您在Okta中有其他群组，它们不会出现在SAML响应中。当您只想将Okta中的部分群组映射到应用角色时，这非常有用。

群组属性声明部分还提供其他过滤器选项，您可以在Okta文档中了解更多信息。

### 6. 测试授权

现在已配置**群组**属性，可以测试授权功能。

在浏览器中访问[http://localhost:3000](http://localhost:3000/)，点击**退出登录**按钮。

让我们尝试用具有**app-admin**角色的用户登录。

您将被重定向至Okta登录页面。输入凭据后点击**登录**。

您将被重定向回示例应用首页，并看到已认证用户的个人资料。

#### 访问经Cerbos授权的API

在**访问经Cerbos授权的API**部分，我们添加了表格来显示Cerbos对示例应用中资源的授权决策。

以下是`cerbos/policies/contact.yaml`文件中定义的策略概览。

- 具有 **app-admin** 角色的用户可以执行对所有资源的所有操作。
- 具有 **app-user** 角色的用户只能对自身拥有的资源执行读取和更新操作。

```yaml
resourcePolicy:
  version: default
  resource: contact
  rules:
    - actions: ['read', 'create', 'update', 'delete']
      effect: EFFECT_ALLOW
      roles:
        - app-admin

    - actions: ['read', 'update']
      effect: EFFECT_ALLOW
      roles:
        - app-user
      condition:
        match:
          expr: request.resource.attr.author == request.principal.id
```

#### 受保护路由

在页面底部的"受保护路由"部分，我们添加了3条路由来演示如何用Cerbos实现应用路由的权限控制。

- _管理员角色可访问的路由_：仅限具有app-admin角色的用户访问。
- _用户和管理员角色均可访问的路由_：允许具有app-user或app-admin角色的用户访问。
- _用户无权限访问的路由_：仅限资源所有者用户访问。

## 结语

本文演示了如何通过Jackson和Cerbos集成SAML单点登录功能，实现了基于SAML SSO的身份认证与授权体系。

这为使用Jackson和Cerbos的企业级应用开启了全新可能。如果您基于本文构建了创新应用，欢迎联系Jackson或Cerbos团队分享成果。

本文完整源代码请访问：[https://github.com/boxyhq/jackson-cerbos](https://github.com/boxyhq/jackson-cerbos)

## 了解更多

关于SAML Jackson和Cerbos的更多资源：

- [BoxyHQ官网](https://boxyhq.com/)
- [Cerbos官网](https://cerbos.dev/)
- [SAML Jackson文档](https://boxyhq.com/docs/jackson/overview)
- [Cerbos文档](https://docs.cerbos.dev/cerbos/latest/index.html)