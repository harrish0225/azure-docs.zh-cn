---
title: 视频索引器故障转移和灾难恢复
titleSuffix: Azure Media Services
description: 了解在发生区域性的数据中心故障或灾难时，如何故障转移到辅助视频索引器帐户。
services: media-services
documentationcenter: ''
author: juliako
manager: femila
editor: ''
ms.service: media-services
ms.subservice: video-indexer
ms.workload: ''
ms.topic: article
ms.custom: ''
ms.date: 07/29/2019
ms.author: juliako
ms.openlocfilehash: 17c21900448fcb6d0a40fe5407f3b8bd62f9e3e4
ms.sourcegitcommit: 849bb1729b89d075eed579aa36395bf4d29f3bd9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/28/2020
ms.locfileid: "79499620"
---
# <a name="video-indexer-failover-and-disaster-recovery"></a>视频索引器故障转移和灾难恢复

如果出现区域数据中心服务中断或故障，Azure 媒体服务视频索引器不会提供服务的即时故障转移。 本文介绍如何配置环境以进行故障转移，以确保应用的最佳可用性，并在发生灾难时将恢复时间降至最低。

建议跨区域对配置业务连续性灾难恢复（BCDR），以便受益于 Azure 的隔离和可用性策略。 有关详细信息，请参阅 [Azure 配对区域](https://docs.microsoft.com/azure/best-practices-availability-paired-regions)。

## <a name="prerequisites"></a>先决条件

Azure 订阅。 如果还没有 Azure 订阅，请注册[azure 免费试用版](https://azure.microsoft.com/free/)。

## <a name="failover-to-a-secondary-account"></a>故障转移到辅助帐户

若要实现 BCDR，您需要有两个视频索引器帐户来处理冗余。

1. 创建连接到 Azure 的两个视频索引器帐户（请参阅[创建视频索引器帐户](connect-to-azure.md)）。 为主要区域创建一个帐户，将另一个帐户创建到配对的 azure 区域。
1. 如果主要区域发生故障，请使用辅助帐户切换到索引编制。

> [!TIP]
> 你可以通过为服务运行状况通知设置活动日志警报来自动执行 BCDR，如针对[服务通知创建活动日志警报](../../service-health/alerts-activity-log-service-notifications.md)。

有关使用多个租户的详细信息，请参阅[管理多个租户](manage-multiple-tenants.md)。 若要实现 BCDR，请选择以下两个选项之一：每个租户的[视频索引器帐户](manage-multiple-tenants.md#video-indexer-account-per-tenant)或[每个租户的 Azure 订阅](manage-multiple-tenants.md#azure-subscription-per-tenant)。

## <a name="next-steps"></a>后续步骤

[管理连接到 Azure 的视频索引器帐户](manage-account-connected-to-azure.md)。
