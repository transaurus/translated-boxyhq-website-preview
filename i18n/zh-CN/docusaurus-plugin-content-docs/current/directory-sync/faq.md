---
title: Faq
sidebar_label: FAQ
hide_table_of_contents: true
---

# 常见问题解答

关于目录同步提供商的常见问题及限制说明。

### 为什么在OneLogin中删除群组时看不到任何事件？

这是OneLogin SCIM的已知问题。OneLogin不会为已删除的群组发送特定的取消配置事件，因此当群组被移除时，您不会看到诸如`group.deleted`或`group.user_removed`之类的事件。建议在从SCIM应用中删除群组本身之前，先移除群组中的用户。

### 为什么在Okta中移除用户时看不到DELETE事件？

Okta不会执行DELETE操作，而是向SCIM应用发送请求将`active`属性设置为`false`。因此，对于从Okta应用中移除的用户，您会观察到`user.updated`事件，其中`active`属性变为`false`。[了解更多](https://developer.okta.com/docs/reference/scim/scim-20/#delete-users)

### 为什么Okta中用户停用时没有事件？

Okta不会为被暂停的用户发送特定的取消配置事件。这种停用事件的缺失是Okta的已知问题。

### Google Workspace是否实时同步？

否，Google Workspace同步并非实时。Jackson会按计划（例如每2小时）同步Google Workspace目录。在自托管部署中，您可以配置cron作业按您选择的时间表同步目录。

### 能否同步Google Workspace中的特定群组？

目前不可行。Jackson会同步Google Workspace中的所有群组。您需要从应用端过滤群组。

### Microsoft Entra ID SCIM默认的同步频率是多少？

Microsoft Entra ID会根据用户和群组分配等情况自动在应用中配置和更新用户账户。同步会按定期计划的时间间隔进行，通常为每20-40分钟一次。[了解更多](https://learn.microsoft.com/en-us/entra/identity/app-provisioning/application-provisioning-when-will-provisioning-finish-specific-user#how-long-will-it-take-to-provision-users)

### 为什么Microsoft Entra ID SCIM中的PATCH请求会失败？

这是Entra ID SCIM实现的已知问题。您必须在SCIM端点URL中添加查询参数`?aadOptscim062020`才能使PATCH请求正常工作。此查询参数会更新PATCH行为并确保SCIM合规性。[了解更多](https://learn.microsoft.com/en-us/entra/identity/app-provisioning/application-provisioning-config-problem-scim-compatibility#flags-to-alter-the-scim-behavior)