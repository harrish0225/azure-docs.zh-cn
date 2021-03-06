---
title: 教程：使用 Azure Active Directory 配置新的 Relic 按组织进行自动用户预配 |Microsoft Docs
description: 了解如何自动将用户 Azure AD 帐户预配到 Relic，并取消其预配。
services: active-directory
documentationcenter: ''
author: zchia
writer: zchia
manager: beatrizd
ms.assetid: 7fd0d976-4f70-4ce5-992e-3ea4ed8e5d60
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 04/14/2020
ms.author: Zhchia
ms.openlocfilehash: 3197ac3033c6550b72bdfdf39fd23c55dda20d90
ms.sourcegitcommit: 849bb1729b89d075eed579aa36395bf4d29f3bd9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/28/2020
ms.locfileid: "82134229"
---
# <a name="tutorial-configure-new-relic-by-organization-for-automatic-user-provisioning"></a>教程：按组织为自动用户预配配置新的 Relic

本教程介绍了需要在新的 Relic by 组织和 Azure Active Directory （Azure AD）中执行的步骤，以配置自动用户预配。 配置后，Azure AD 会使用 Azure AD 预配服务自动设置用户和组，并将其预配到[新 Relic](https://newrelic.com/) 。 有关此服务的功能、工作原理以及常见问题的重要详细信息，请参阅[使用 Azure Active Directory 自动将用户预配到 SaaS 应用程序和取消预配](../manage-apps/user-provisioning.md)。 


## <a name="capabilities-supported"></a>支持的功能
> [!div class="checklist"]
> * 在 New Relic 中按组织创建用户
> * 当新 Relic 中的用户不再需要访问权限时，将其删除
> * 使用户属性在 Azure AD 和组织中的新 Relic 之间保持同步
> * 按组织在新 Relic 中预配组和组成员身份
> * [单一登录](https://docs.microsoft.com/azure/active-directory/saas-apps/new-relic-limited-release-tutorial)到新 Relic （按组织）（推荐）

## <a name="prerequisites"></a>必备条件

本教程中概述的方案假定你已具有以下先决条件：

* [Azure AD 租户](https://docs.microsoft.com/azure/active-directory/develop/quickstart-create-new-tenant) 
* Azure AD 中的一个用户帐户，有权配置预配（例如，应用程序管理员、云应用程序管理员、应用程序所有者或全局管理员）的[权限](https://docs.microsoft.com/azure/active-directory/users-groups-roles/directory-assign-admin-roles)。 
* 你希望用户有权访问的新 Relic 中的一个或多个帐户。 

## <a name="step-1-plan-your-provisioning-deployment"></a>步骤 1。 规划预配部署
1. 了解[预配服务的工作原理](https://docs.microsoft.com/azure/active-directory/manage-apps/user-provisioning)。
2. 确定将处于[预配范围内的](https://docs.microsoft.com/azure/active-directory/manage-apps/define-conditional-rules-for-provisioning-user-accounts)用户。
3. 确定要[按组织 Azure AD 和新的 Relic 之间映射的](https://docs.microsoft.com/azure/active-directory/manage-apps/customize-application-attributes)数据。 

## <a name="step-2-configure-new-relic-by-organization-to-support-provisioning-with-azure-ad"></a>步骤 2。 配置新的 Relic by 组织以支持设置 Azure AD

联系你的客户代表或在 support.newrelic.com 上获得支持，为你的组织配置 SCIM 和 SSO。 你将需要为你的帐户代表提供：

- 你的组织名称
- 要与组织关联的新 Relic 帐户 Id 的列表

使用此信息，你的帐户代表会在新系统中为你创建一个组织记录，并将你的帐户关联到组织。

你的帐户代表提供以下信息，你将需要为标识提供者配置新的 Relic SCIM/SSO 应用程序：

- SCIM 终结点（租户 URL）
- SCIM 持有者令牌（机密令牌）

SCIM 持有者令牌允许在新 Relic 上预配用户，因此请保持此值的安全性。 你的帐户代表会以安全的方式将 SCIM 持有者令牌传输给你。

## <a name="step-3-add-new-relic-by-organization-from-the-azure-ad-application-gallery"></a>步骤 3. 通过 Azure AD 应用程序库中的组织添加新的 Relic

通过 Azure AD 应用程序库中的 "组织" 添加新的 Relic，以开始按组织管理预配到新 Relic。 如果以前已通过组织为 SSO 设置了新的 Relic，则可以使用相同的应用程序。 但建议您在最初测试集成时创建一个单独的应用程序。 在[此处](https://docs.microsoft.com/azure/active-directory/manage-apps/add-gallery-app)了解有关从库中添加应用程序的详细信息。 

## <a name="step-4-define-who-will-be-in-scope-for-provisioning"></a>步骤 4. 定义将在设置范围内的人员 

Azure AD 预配服务允许你确定将根据分配给应用程序的人员，或基于用户/组属性进行预配的用户的范围。 如果选择将根据分配预配到你的应用的用户的范围，则可以使用以下[步骤](../manage-apps/assign-user-or-group-access-portal.md)将用户和组分配给应用程序。 如果选择仅根据用户或组的属性设置的作用域，则可以使用[此处](https://docs.microsoft.com/azure/active-directory/manage-apps/define-conditional-rules-for-provisioning-user-accounts)所述的范围筛选器。 

* 将用户和组分配到组织的新 Relic 时，必须选择 "**默认" 访问权限**以外的其他角色。 具有默认访问角色的用户将从预配中排除，并在预配日志中被标记为不有效。 如果应用程序上唯一可用的角色是默认访问角色，则可以[更新应用程序清单](https://docs.microsoft.com/azure/active-directory/develop/howto-add-app-roles-in-azure-ad-apps)来添加其他角色。 

* 从小开始。 在向所有人推出之前，请使用少量的用户和组进行测试。 如果设置的作用域设置为 "分配的用户和组"，则可以通过将一个或两个用户或组分配到应用来对此进行控制。 当作用域设置为 "所有用户和组" 时，可以指定[基于属性的范围筛选器](https://docs.microsoft.com/azure/active-directory/manage-apps/define-conditional-rules-for-provisioning-user-accounts)。 


## <a name="step-5-configure-automatic-user-provisioning-to-new-relic-by-organization"></a>步骤 5。 按组织配置新 Relic 的自动用户预配 

本部分将指导你完成以下步骤：配置 Azure AD 预配服务，以便基于 Azure AD 中的用户和/或组分配在 TestApp 中创建、更新和禁用用户和/或组。

### <a name="to-configure-automatic-user-provisioning-for-new-relic-by-organization-in-azure-ad"></a>若要为 Azure AD 中的组织配置自动用户预配，请执行以下操作：

1. 登录 [Azure 门户](https://portal.azure.com)。 选择 "**企业应用程序**"，并选择 "**所有应用程序**"。

    ![“企业应用程序”边栏选项卡](common/enterprise-applications.png)

2. 在应用程序列表中，选择 "**新建 Relic By 组织**"。

    ![应用程序列表中的 New Relic 链接](common/all-applications.png)

3. 选择“预配”**** 选项卡。

    ![设置选项卡](common/provisioning.png)

4. 将**预配模式**设置为 "**自动**"。

    ![设置选项卡](common/provisioning-automatic.png)

5. 在 "**管理员凭据**" 部分中`https://scim-provisioning.service.newrelic.com/scim/v2` ，输入 "租户 URL"。 输入先前在**机密令牌**中检索到的 SCIM authentication 令牌值。 单击 "**测试连接**" 以确保 Azure AD 可以连接到新 Relic。 如果连接失败，请确保新 Relic 帐户具有管理员权限，然后重试。

    ![预配](./media/new-relic-by-organization-provisioning-tutorial/provisioning.png)

6. 在 "**通知电子邮件**" 字段中，输入应接收预配错误通知的人员或组的电子邮件地址，并选中 "**发生故障时发送电子邮件通知**" 复选框。

    ![通知电子邮件](common/provisioning-notification-email.png)

7. 选择“保存”  。

8. 在 "**映射**" 部分下，选择 "**将 Azure Active Directory 用户同步到组织的新 Relic**"。

9. 在 "**属性映射**" 部分中，查看从 Azure AD 同步到新 Relic 的用户属性。 选为 "**匹配**" 属性的属性用于匹配 New Relic by 组织中的用户帐户以执行更新操作。 如果选择更改[匹配的目标属性](https://docs.microsoft.com/azure/active-directory/manage-apps/customize-application-attributes)，将需要确保新的 Relic BY 组织 API 支持基于该属性筛选用户。 选择“保存”按钮以提交任何更改****。

   |Attribute|类型|
   |---|---|
   |userName|字符串|
   |externalId|字符串|
   |活动|布尔值|
   |emails[type eq "work"].value|字符串|
   |name.givenName|字符串|
   |名称。已设置格式|字符串|
   |timezone|字符串|

10. 在 "**映射**" 部分下，选择 "**按组织将 Azure Active Directory 组同步到新 Relic**"。

11. 在 "**属性映射**" 部分中，查看从 Azure AD 同步到新 Relic 的组属性。 选为 "**匹配**" 属性的属性用于匹配 New Relic by groups 中的组以执行更新操作。 选择“保存”按钮以提交任何更改****。

      |Attribute|类型|
      |---|---|
      |displayName|字符串|
      |externalId|字符串|
      |members|参考|

12. 若要配置范围筛选器，请参阅[范围筛选器教程](../manage-apps/define-conditional-rules-for-provisioning-user-accounts.md)中提供的以下说明。

13. 若要为新的 Relic 按组织启用 Azure AD 预配服务，请在 "**设置**" 部分中将 "**预配状态**" 更改为 **"打开**"。

    ![预配状态已打开](common/provisioning-toggle-on.png)

14. 通过在 "**设置**" 部分的 "**范围**" 中选择所需的值，定义要按组织预配到新 Relic 的用户和/或组。

    ![预配范围](common/provisioning-scope.png)

15. 已准备好预配时，单击“保存”****。

    ![保存预配配置](common/provisioning-configuration-save.png)

此操作将启动 "**设置**" 部分的 "**范围**" 中定义的所有用户和组的初始同步循环。 初始周期比后续循环长，只要 Azure AD 预配服务正在运行，就大约每40分钟执行一次。 

## <a name="step-6-monitor-your-deployment"></a>步骤 6。 监视部署
配置预配后，请使用以下资源来监视部署：

* 使用[预配日志](https://docs.microsoft.com/azure/active-directory/reports-monitoring/concept-provisioning-logs)来确定哪些用户已成功设置或失败
* 检查[进度栏](https://docs.microsoft.com/azure/active-directory/manage-apps/application-provisioning-when-will-provisioning-finish-specific-user)，查看设置周期的状态以及完成操作的方式
* 如果预配配置似乎处于不正常状态，则应用程序将进入隔离区。 [在此处](https://docs.microsoft.com/azure/active-directory/manage-apps/application-provisioning-quarantine-status)了解有关隔离状态的详细信息。  


## <a name="additional-resources"></a>其他资源

* [管理企业应用的用户帐户预配](../manage-apps/configure-automatic-user-provisioning-portal.md)
* [Azure Active Directory 的应用程序访问与单一登录是什么？](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>后续步骤

* [了解如何查看日志并获取有关预配活动的报告](../manage-apps/check-status-user-account-provisioning.md)
