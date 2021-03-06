---
title: 教程：Azure Active Directory 单一登录 (SSO) 与 Teamphoria 集成 | Microsoft Docs
description: 了解如何在 Azure Active Directory 和 Teamphoria 之间配置单一登录。
services: active-directory
documentationCenter: na
author: jeevansd
manager: mtillman
ms.reviewer: barbkess
ms.assetid: d569c705-6f0f-4ec1-b485-ba82526b5d32
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.date: 10/09/2019
ms.author: jeedes
ms.collection: M365-identity-device-management
ms.openlocfilehash: 2631b34f5658c9d4f76ca26d378bc63fe59ad156
ms.sourcegitcommit: 0947111b263015136bca0e6ec5a8c570b3f700ff
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 03/24/2020
ms.locfileid: "72373267"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-teamphoria"></a>教程：Azure Active Directory 单一登录 (SSO) 与 Teamphoria 集成

本教程介绍如何将 Teamphoria 与 Azure Active Directory (Azure AD) 集成。 将 Teamphoria 与 Azure AD 集成后，可以：

* 在 Azure AD 中控制谁有权访问 Teamphoria。
* 让用户使用其 Azure AD 帐户自动登录到 Teamphoria。
* 在一个中心位置（Azure 门户）管理帐户。

若要了解有关 SaaS 应用与 Azure AD 集成的详细信息，请参阅 [Azure Active Directory 的应用程序访问与单一登录是什么](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)。

## <a name="prerequisites"></a>先决条件

若要开始操作，需备齐以下项目：

* 一个 Azure AD 订阅。 如果没有订阅，可以获取一个[免费帐户](https://azure.microsoft.com/free/)。
* 已启用 Teamphoria 单一登录 (SSO) 的订阅。

## <a name="scenario-description"></a>方案描述

本教程在测试环境中配置并测试 Azure AD SSO。

* Teamphoria 支持 **SP** 发起的 SSO

## <a name="adding-teamphoria-from-the-gallery"></a>从库中添加 Teamphoria

要配置 Teamphoria 与 Azure AD 的集成，需要从库中将 Teamphoria 添加到托管的 SaaS 应用列表。

1. 使用工作或学校帐户或个人 Microsoft 帐户登录到 [Azure 门户](https://portal.azure.com)。
1. 在左侧导航窗格中，选择“Azure Active Directory”服务  。
1. 导航到“企业应用程序”，选择“所有应用程序”   。
1. 若要添加新的应用程序，请选择“新建应用程序”  。
1. 在“从库中添加”部分的搜索框中，键入 **Teamphoria**。 
1. 从结果面板中选择“Teamphoria”，然后添加该应用  。 在该应用添加到租户时等待几秒钟。

## <a name="configure-and-test-azure-ad-single-sign-on-for-teamphoria"></a>配置并测试 Teamphoria 的 Azure AD 单一登录

使用名为 **B.Simon** 的测试用户配置和测试 Teamphoria 的 Azure AD SSO。 若要运行 SSO，需要在 Azure AD 用户与 Teamphoria 相关用户之间建立链接关系。

若要配置并测试 Teamphoria 的 Azure AD SSO，请完成以下构建基块：

1. **[配置 Azure AD SSO](#configure-azure-ad-sso)** - 使用户能够使用此功能。
    1. **[创建 Azure AD 测试用户](#create-an-azure-ad-test-user)** - 使用 B. Simon 测试 Azure AD 单一登录。
    1. **[分配 Azure AD 测试用户](#assign-the-azure-ad-test-user)** - 使 B. Simon 能够使用 Azure AD 单一登录。
1. **[配置 Teamphoria SSO](#configure-teamphoria-sso)** - 在应用程序端配置单一登录设置。
    1. **[创建 Teamphoria 测试用户](#create-teamphoria-test-user)** - 在 Teamphoria 中创建 B.Simon 的对应用户，并将其链接到该用户的 Azure AD 表示形式。
1. **[测试 SSO](#test-sso)** - 验证配置是否正常工作。

## <a name="configure-azure-ad-sso"></a>配置 Azure AD SSO

按照下列步骤在 Azure 门户中启用 Azure AD SSO。

1. 在 [Azure 门户](https://portal.azure.com/)的“Teamphoria”应用程序集成页上，找到“管理”部分，选择“单一登录”    。
1. 在“选择单一登录方法”页上选择“SAML”   。
1. 在“使用 SAML 设置单一登录”页上，单击“基本 SAML 配置”的编辑/笔形图标以编辑设置   。

   ![编辑基本 SAML 配置](common/edit-urls.png)

1. 在“基本 SAML 配置”部分，输入以下字段的值  ：

    在“登录 URL”  文本框中，使用以下模式键入 URL：`https://<sub-domain>.teamphoria.com/login`

    > [!NOTE]
    > 此值不是真实值。 请使用实际登录 URL 更新此值。 联系 [Teamphoria 客户端支持团队](https://www.teamphoria.com/)以获取值。 还可以参考 Azure 门户中的“基本 SAML 配置”  部分中显示的模式。

1. 在“使用 SAML 设置单一登录”页的“SAML 签名证书”部分中，找到“证书(Base64)”，选择“下载”以下载该证书并将其保存到计算机上     。

    ![证书下载链接](common/certificatebase64.png)

1. 在“设置 Teamphoria”部分中，根据要求复制相应的 URL。 

    ![复制配置 URL](common/copy-configuration-urls.png)

### <a name="create-an-azure-ad-test-user"></a>创建 Azure AD 测试用户

在本部分，我们将在 Azure 门户中创建名为 B.Simon 的测试用户。

1. 在 Azure 门户的左侧窗格中，依次选择“Azure Active Directory”、“用户”和“所有用户”    。
1. 选择屏幕顶部的“新建用户”  。
1. 在“用户”属性中执行以下步骤  ：
   1. 在“名称”  字段中，输入 `B.Simon`。  
   1. 在“用户名”字段中输入 username@companydomain.extension  。 例如，`B.Simon@contoso.com` 。
   1. 选中“显示密码”复选框，然后记下“密码”框中显示的值。  
   1. 单击“创建”。 

### <a name="assign-the-azure-ad-test-user"></a>分配 Azure AD 测试用户

在本部分中，将通过授予 B.Simon 访问 Teamphoria 的权限，允许其使用 Azure 单一登录。

1. 在 Azure 门户中，依次选择“企业应用程序”、“所有应用程序”。  
1. 在应用程序列表中，选择“Teamphoria”  。
1. 在应用的概述页中，找到“管理”部分，选择“用户和组”   。

   ![“用户和组”链接](common/users-groups-blade.png)

1. 选择“添加用户”，然后在“添加分配”对话框中选择“用户和组”。   

    ![“添加用户”链接](common/add-assign-user.png)

1. 在“用户和组”对话框中，从“用户”列表中选择“B.Simon”，然后单击屏幕底部的“选择”按钮。   
1. 如果在 SAML 断言中需要任何角色值，请在“选择角色”对话框的列表中为用户选择合适的角色，然后单击屏幕底部的“选择”按钮。  
1. 在“添加分配”对话框中，单击“分配”按钮。  

## <a name="configure-teamphoria-sso"></a>配置 Teamphoria SSO

1. 若要在 Teamphoria 中自动完成配置，需要单击“安装扩展”安装“我的应用安全登录浏览器扩展”。  

    ![我的应用扩展](common/install-myappssecure-extension.png)

2. 将扩展添加到浏览器后，单击“设置 Teamphoria”将定向到 Teamphoria 应用程序  。 在此处，请提供管理员凭据以登录到 Teamphoria。 浏览器扩展会自动配置该应用程序，并自动执行步骤 3-6。

    ![设置配置](common/setup-sso.png)

3. 若要手动设置 Teamphoria，请打开新的 Web 浏览器窗口，以管理员身份登录 Teamphoria 公司站点，并执行以下步骤：

4. 转到左侧工具栏中的“管理员设置”  选项，并在“配置”选项卡下单击“单一登录”  ，以打开 SSO 配置窗口。

    ![配置单一登录](./media/teamphoria-tutorial/admin_sso_configure.png)

5. 单击右上角的“添加新的标识提供程序”  选项，以打开用于添加 SSO 设置的窗体。

    ![配置单一登录](./media/teamphoria-tutorial/add_new_identity_provider.png)

6. 在字段中输入详细信息，如下所述 -

    ![配置单一登录](./media/teamphoria-tutorial/Teamphoria_sso_save.png)

    a. **显示名称**：在管理页上输入插件的显示名称。

    b. **按钮名称**：会在用于通过 SSO 登录的登录页上显示的选项卡的名称。

    c. **证书**：在记事本中打开之前从 Azure 门户下载的证书，复制相同的内容并将其粘贴到此处的框中。

    d. **入口点**：粘贴前面从 Azure 门户复制的“登录 URL”。 

    e. 将选项切换为“打开”  ，然后单击“保存”  。

### <a name="create-teamphoria-test-user"></a>创建 Teamphoria 测试用户

要使 Azure AD 用户能够登录到 Teamphoria，必须将其预配到 Teamphoria 中。 对于 Teamphoria，需要手动执行预配。

**若要预配用户帐户，请执行以下步骤：**

1. 以管理员身份登录到 Teamphoria 公司站点。

1. 单击左侧工具栏上的“管理员”  设置，并在“管理”  选项卡下单击“用户”  ，以打开用户的管理页。

    ![添加员工](./media/teamphoria-tutorial/admin_manage_users.png)

1. 单击“手动邀请”  选项。

    ![邀请人员](./media/teamphoria-tutorial/admin_manage_add_users.png)

1. 在此页上，执行以下操作。

    ![邀请人员](./media/teamphoria-tutorial/manual_user_invite.png)

    a. 在“电子邮件地址”文本框中，输入用户的**电子邮件地址**，例如 B.Simon。 

    b. 在“名字”文本框中，输入用户的名字，例如 **B**。 

    c. 在“姓氏”文本框中，输入用户的姓氏，例如 **Simon**。 

    d. 单击“邀请 1 名用户”  。 用户需要接受邀请，才能在系统中创建用户。

## <a name="test-sso"></a>测试 SSO 

在本部分中，使用访问面板测试 Azure AD 单一登录配置。

在访问面板中单击“Teamphoria”磁贴时，应会自动登录到设置了 SSO 的 Teamphoria。 有关访问面板的详细信息，请参阅 [Introduction to the Access Panel](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction)（访问面板简介）。

## <a name="additional-resources"></a>其他资源

- [有关如何将 SaaS 应用与 Azure Active Directory 集成的教程列表](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [什么是使用 Azure Active Directory 的应用程序访问和单一登录？](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [什么是 Azure Active Directory 中的条件访问？](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

- [通过 Azure AD 试用 Teamphoria](https://aad.portal.azure.com/)

