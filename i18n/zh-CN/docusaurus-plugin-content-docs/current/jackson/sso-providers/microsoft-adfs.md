---
title: Set up your own custom SAML application for Microsoft AD FS
sidebar_label: Microsoft AD FS
---

# Microsoft AD FS SAML 单点登录配置

本节将指导您如何为 Microsoft AD FS 设置自定义 SAML 应用程序。

在 AD FS 中需要配置信赖方 (RP) 信任。您可提供我们的 SP 元数据文件（通常位于 `https://<您的托管地址>/.well-known/sp-metadata`），AD FS 将自动获取配置。

若需手动输入配置信息，请参考[通用SAML配置章节](./generic-saml.md)。公钥证书用于签名验证，通常可通过 `https://<您的托管地址>/.well-known/saml.cer` 获取。

您还需配置声明属性映射，以便获取用户的资料信息。

## 使用联合元数据创建声明感知的信赖方信任

1. 在服务器管理器中点击"工具"，选择"AD FS 管理"
2. 在"操作"面板点击"添加信赖方信任"
   ![Microsoft AD FS SAML 步骤2](/images/docs/jackson/sso-providers/adfs/addtrust2.png)
3. 在欢迎页面选择"声明感知"，点击"开始"
   ![Microsoft AD FS SAML 步骤3](/images/docs/jackson/sso-providers/adfs/addtrust3.png)
4. 在"选择数据源"页面点击"导入在线或本地网络发布的信赖方数据"，在"联合元数据地址"栏输入元数据URL或主机名，点击"下一步"
   ![Microsoft AD FS SAML 步骤4](/images/docs/jackson/sso-providers/adfs/addtrust4.png)
5. 在"指定显示名称"页面输入名称，在"备注"栏添加描述，点击"下一步"
6. 在"选择颁发授权规则"页面选择"允许所有用户访问此信赖方"或"拒绝所有用户访问此信赖方"，点击"下一步"
7. 在"准备添加信任"页面检查设置，点击"下一步"保存信赖方信任信息
8. 在"完成"页面点击"关闭"，系统将自动弹出"编辑声明规则"对话框
9. **属性映射**：
   在"配置声明规则"界面输入规则名称，选择"Active Directory"作为属性存储，添加以下映射：

- 从LDAP属性列选择`E-Mail-Addresses`，在输出声明类型输入`E-Mail Address`
- 从LDAP属性列选择`Given-Name`，在输出声明类型输入`Given Name`
- 从LDAP属性列选择`Surname`，在输出声明类型输入`Surname`
- 从LDAP属性列选择`User-Principal-Name`，在输出声明类型输入`Name ID`

10. **转换规则**：创建转换规则将传入的`Email-Address`映射为输出的`NameID`（类型为`Email`）。若跳过此步骤，ADFS默认发送`Unspecified`类型的`NameID`会导致`InvalidNameIDPolicy`错误。

![转换规则](/images/docs/jackson/sso-providers/adfs/nameid-email.png)

如需使用声明规则语言，可应用以下规则：

```sh
c:[Type == "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress"] => issue(Type = "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/nameidentifier", Issuer = c.Issuer, OriginalIssuer = c.OriginalIssuer, Value = c.Value, ValueType = c.ValueType, Properties["http://schemas.xmlsoap.org/ws/2005/05/identity/claimproperties/format"] = "urn:oasis:names:tc:SAML:1.1:nameid-format:emailAddress");
```

11. 最后以管理员身份打开Windows PowerShell，运行以下命令：

```sh
 Set-ADFSRelyingPartyTrust -TargetName <display-name> -SamlResponseSignature "MessageAndAssertion"
```