---
title: Azure AD 应用程序代理：版本发布历史记录 |Microsoft Docs
description: 本文列出 Azure AD 应用程序代理的所有版本，并介绍新功能和已修复的问题
services: active-directory
documentationcenter: ''
author: msmimart
manager: CelesteDG
editor: ''
ms.assetid: ''
ms.service: active-directory
ms.devlang: na
ms.topic: reference
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 04/07/2020
ms.subservice: app-mgmt
ms.author: mimart
ms.collection: M365-identity-device-management
ms.openlocfilehash: f027fbce66a73306165a0ad35d1ba3faa7a5c0bc
ms.sourcegitcommit: 849bb1729b89d075eed579aa36395bf4d29f3bd9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/28/2020
ms.locfileid: "80983885"
---
# <a name="azure-ad-application-proxy-version-release-history"></a>Azure AD 应用程序代理：版本发布历史记录
本文列出了已发布的 Azure Active Directory （Azure AD）应用程序代理的版本和功能。 Azure AD 团队定期更新具有新特性和功能的应用程序代理。 发布新版本时，应用程序代理连接器会自动更新。 

建议确保为连接器启用了自动更新，以确保具有最新的功能和 bug 修复。 Microsoft 为最新连接器版本和之前的版本提供直接支持。

下面是相关资源的列表：

资源 |  详细信息
--------- | --------- |
如何启用应用程序代理 | 本[教程](application-proxy-add-on-premises-application.md)介绍了启用应用程序代理和安装和注册连接器的先决条件。
了解 Azure AD 应用程序代理连接器 | 了解有关[连接器管理](application-proxy-connectors.md)以及连接器[自动升级](application-proxy-connectors.md#automatic-updates)方式的详细信息。
Azure AD 应用程序代理连接器下载 |  [下载最新的连接器](https://download.msappproxy.net/subscription/d3c8b69d-6bf7-42be-a529-3fe9c2e70c90/connector/download)。

## <a name="1515260"></a>1.5.1526.0

### <a name="release-status"></a>版本状态

7月7日，2020：已发布以供下载

### <a name="new-features-and-improvements"></a>新增功能和改进
-   连接器仅将 TLS 1.2 用于所有连接。 有关更多详细信息，请参阅[连接器先决条件](application-proxy-add-on-premises-application.md#before-you-begin)。
- 改进了连接器与 Azure 服务之间的信号。 这包括支持用于连接器与 Azure 服务之间的 WCF 通信的可靠会话，以及针对 WebSocket 通信的 DNS 缓存改进。
- 支持在连接器与后端应用程序之间配置代理。 有关详细信息，请参阅[使用现有的本地代理服务器](application-proxy-configure-connectors-with-proxy-servers.md)。

### <a name="fixed-issues"></a>修复的问题
- 删除了到端口8080的通信，以便从连接器连接到 Azure 服务。
- 为 WebSocket 通信添加了调试跟踪。 
- 当在后端应用程序 cookie 上设置时，解析保留 SameSite 属性。

## <a name="156120"></a>1.5.612.0

### <a name="release-status"></a>版本状态

2018年9月20日：已发布以供下载

### <a name="new-features-and-improvements"></a>新增功能和改进

- 添加了对 QlikSense 应用程序的 WebSocket 支持。 若要了解有关如何将 QlikSense 与应用程序代理集成的详细信息，请参阅此[演练](application-proxy-qlik.md)。 
- 改进了安装向导，使其更易于配置出站代理。 
- 将 TLS 1.2 设置为连接器的默认协议。 
- 添加了新的最终用户许可协议（EULA）。  

### <a name="fixed-issues"></a>修复的问题

- 修复了一个 bug，该 bug 导致连接器中的内存泄漏。
- 更新了 Azure 服务总线版本，其中包括针对连接器超时问题的 bug 修复。

## <a name="154020"></a>1.5.402.0

### <a name="release-status"></a>版本状态

2018年1月19日：已发布以供下载

### <a name="fixed-issues"></a>修复的问题

- 添加了对需要 cookie 中的域转换的自定义域的支持。

## <a name="151320"></a>1.5.132.0

### <a name="release-status"></a>版本状态 

5月25日，2017：已发布以供下载 

### <a name="new-features-and-improvements"></a>新增功能和改进 

提高了对连接器的出站连接限制的控制。 

## <a name="15360"></a>1.5.36.0

### <a name="release-status"></a>版本状态

2017年4月15日：已发布以供下载

### <a name="new-features-and-improvements"></a>新增功能和改进

- 简化的载入和管理，所需的端口更少。 应用程序代理现在需要仅打开两个标准出站端口：443和80。 应用程序代理将继续仅使用出站连接，因此，不需要在 DMZ 中使用任何组件。 有关详细信息，请参阅我们的 [配置文档](application-proxy-add-on-premises-application.md)。  
- 如果你的外部代理或防火墙支持，你现在可以通过 DNS 而不是 IP 范围打开网络。 应用程序代理服务要求仅连接到 *. msappproxy.net 和 *. servicebus.windows.net。


## <a name="earlier-versions"></a>早期版本

如果使用的应用程序代理连接器版本低于1.5.36.0，请更新到最新版本，以确保具有最新的完全受支持的功能。

## <a name="next-steps"></a>后续步骤
- 详细了解[如何通过 Azure AD 应用程序代理远程访问本地应用程序](application-proxy.md)。
- 若要开始使用应用程序代理，请参阅[教程：通过应用程序代理添加用于远程访问的本地应用程序](application-proxy-add-on-premises-application.md)。
