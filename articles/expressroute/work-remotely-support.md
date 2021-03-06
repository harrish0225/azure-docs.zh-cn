---
title: 使用 Azure ExpressRoute 支持远程用户
description: 本页介绍了如何利用 Azure ExpressRoute 来远程工作，因为 COVID-19 流行病。
services: expressroute
author: cherylmc
ms.service: expressroute
ms.topic: article
ms.date: 03/22/2020
ms.author: ajitbhu
ms.openlocfilehash: 2fe990fd9c5922dd2e059dbb212cd8bb1cd3726e
ms.sourcegitcommit: 849bb1729b89d075eed579aa36395bf4d29f3bd9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/28/2020
ms.locfileid: "80336655"
---
# <a name="using-azure-expressroute-to-create-hybrid-connectivity-to-support-remote-users"></a>使用 Azure ExpressRoute 创建混合连接来支持远程用户

本文介绍如何使用 ExpressRoute、Azure、Microsoft 网络和 Azure 合作伙伴生态系统以远程方式工作。

## <a name="connecting-to-azure-services-with-expressroute"></a>通过 ExpressRoute 连接到 Azure 服务

>[!NOTE]
>本文介绍了如何利用 ExpressRoute、Azure、Microsoft 网络和 Azure 合作伙伴生态系统远程工作，并减少因 COVID-19 而面临的网络问题。
>

[ExpressRoute](expressroute-introduction.md)是一项 Azure 服务，允许在 Microsoft 数据中心与你的本地或归置设施中的基础结构之间建立专用连接。 ExpressRoute 连接不通过公共 Internet 。 与通过 Internet 的典型连接相比，它们提供安全的连接、可靠性和速度，并具有更低的一致性延迟。

## <a name="useful-links"></a>有用链接

* [如何增加现有 ExpressRoute 线路的带宽](expressroute-howto-circuit-portal-resource-manager.md#modify)
* [ExpressRoute 监视，指标和警报](expressroute-monitoring-metrics-alerts.md#expressroute-gateway-connections-in-bitsseconds)
* [经由 ExpressRoute 的路由优化](expressroute-optimize-routing.md)
* [适用于 O365 的 Azure ExpressRoute](https://docs.microsoft.com/office365/enterprise/azure-expressroute?redirectSourcePath=%252farticle%252f6d2534a2-c19c-4a99-be5e-33a0cee5d3bd)
* [非对称路由注意事项](expressroute-asymmetric-routing.md)
* [如何在 Azure 门户中打开支持请求](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/overview)

## <a name="troubleshoot"></a>疑难解答

* [验证 ExpressRoute 连接](expressroute-troubleshooting-expressroute-overview.md)
* 获取[资源管理器](expressroute-troubleshooting-arp-resource-manager.md)和[经典](expressroute-troubleshooting-arp-classic.md)的 ARP 表
* [重置失败的线路](reset-circuit.md)
* [网络性能故障排除](expressroute-troubleshooting-network-performance.md)

## <a name="next-steps"></a>后续步骤

* [了解 ExpressRoute 连接模型](expressroute-connectivity-models.md)
* 了解 ExpressRoute 连接和路由域。 请参阅[ExpressRoute 线路和路由域](expressroute-circuit-peerings.md)
* 查找服务提供商。 请参阅[ExpressRoute 合作伙伴和对等位置](expressroute-locations.md)
* 确保符合所有先决条件。 请参阅 [ExpressRoute 先决条件](expressroute-prerequisites.md)。
* [创建和修改 ExpressRoute 线路](expressroute-howto-circuit-portal-resource-manager.md)
* [创建和修改 ExpressRoute 线路的对等互连](expressroute-howto-routing-portal-resource-manager.md)
* [将虚拟网络连接到 ExpressRoute 线路](expressroute-howto-linkvnet-portal-resource-manager.md)
