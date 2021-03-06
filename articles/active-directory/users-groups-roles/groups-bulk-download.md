---
title: 在 Azure Active Directory 门户中下载组的列表 |Microsoft Docs
description: 在 Azure Active Directory 的 Azure 管理中心中批量下载组属性。
services: active-directory
author: curtand
ms.author: curtand
manager: mtillman
ms.date: 04/16/2020
ms.topic: conceptual
ms.service: active-directory
ms.subservice: users-groups-roles
ms.workload: identity
ms.custom: it-pro
ms.reviewer: jeffsta
ms.collection: M365-identity-device-management
ms.openlocfilehash: 59983678c1b14d6aa87a7b500605e3abeb6a9b85
ms.sourcegitcommit: 849bb1729b89d075eed579aa36395bf4d29f3bd9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/28/2020
ms.locfileid: "81533526"
---
# <a name="bulk-download-a-list-of-groups-in-azure-active-directory"></a>批量下载 Azure Active Directory 中的组列表

使用 Azure Active Directory （Azure AD）门户，你可以将组织中所有组的列表批量下载到逗号分隔值（CSV）文件中。

## <a name="to-download-a-list-of-groups"></a>下载组列表

1. 使用组织中的管理员帐户登录到[Azure 门户](https://portal.azure.com)。
1. 在 Azure AD 中，选择 "**组** > " "**下载组**"。
1. 在 "**组下载**" 页上，选择 "**启动**" 以接收列出组的 CSV 文件。

   !["所有组" 页上的 "下载组" 命令](./media/groups-bulk-download/bulk-download.png)

## <a name="check-download-status"></a>检查下载状态

你可以在 "**批量操作结果**" 页中查看所有挂起的批量请求的状态。

[![](media/groups-bulk-download/bulk-center.png "Check status in the Bulk Operations Results page")](media/groups-bulk-download/bulk-center.png#lightbox)

## <a name="bulk-download-service-limits"></a>大容量下载服务限制

下载组列表的每个批量活动最多可以运行一小时。 这使你可以下载至少300000个组的列表。

## <a name="next-steps"></a>后续步骤

- [批量删除组成员](groups-bulk-remove-members.md)
- [下载组的成员](groups-bulk-download-members.md)
