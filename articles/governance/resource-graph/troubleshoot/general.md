---
title: 排查常见错误
description: 了解如何在通过 Azure 资源关系图查询 Azure 资源时排查各种 Sdk 的问题。
ms.date: 10/18/2019
ms.topic: troubleshooting
ms.openlocfilehash: f881db4f75bcee8c13221717596442ac29a4b1ac
ms.sourcegitcommit: 849bb1729b89d075eed579aa36395bf4d29f3bd9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/28/2020
ms.locfileid: "74303899"
---
# <a name="troubleshoot-errors-using-azure-resource-graph"></a>排查使用 Azure Resource Graph 时出现的错误

在使用 Azure Resource Graph 查询 Azure 资源时，你可能会遇到错误。 本文描述可能会发生的各种错误及其解决方法。

## <a name="finding-error-details"></a>查找错误详细信息

大多数错误的原因是使用 Azure Resource Graph 运行查询时出现了问题。 如果某个查询失败，SDK 将提供有关失败的查询的详细信息。 此信息会指出存在的问题，以便可以修复问题并使后续查询成功。

## <a name="general-errors"></a>常规错误

### <a name="scenario-too-many-subscriptions"></a><a name="toomanysubscription"></a>方案：订阅过多

#### <a name="issue"></a>问题

如果客户有权访问1000多个订阅（包括使用[Azure Lighthouse](../../../lighthouse/overview.md)的跨租户订阅），则不能在对 Azure 资源关系图的单个调用中跨所有订阅提取数据。

#### <a name="cause"></a>原因

Azure CLI 和 PowerShell 只会将前 1000 个订阅转发到 Azure Resource Graph。 Azure Resource Graph 的 REST API 接受对最大数目的订阅执行查询。

#### <a name="resolution"></a>解决方法

对一部分订阅批处理查询请求，以保持在 1000 个订阅的限制范围内。 解决方法是在 PowerShell 中使用 **Subscription** 参数。

```azurepowershell-interactive
# Replace this query with your own
$query = 'Resources | project type'

# Fetch the full array of subscription IDs
$subscriptions = Get-AzSubscription
$subscriptionIds = $subscriptions.Id

# Create a counter, set the batch size, and prepare a variable for the results
$counter = [PSCustomObject] @{ Value = 0 }
$batchSize = 1000
$response = @()

# Group the subscriptions into batches
$subscriptionsBatch = $subscriptionIds | Group -Property { [math]::Floor($counter.Value++ / $batchSize) }

# Run the query for each batch
foreach ($batch in $subscriptionsBatch){ $response += Search-AzGraph -Query $query -Subscription $batch.Group }

# View the completed results of the query on all subscriptions
$response
```

### <a name="scenario-unsupported-content-type-rest-header"></a><a name="rest-contenttype"></a>方案：不受支持的内容类型 REST 标头

#### <a name="issue"></a>问题

查询 Azure Resource Graph REST API 的客户获得返回的 _500_（内部服务器错误）响应。

#### <a name="cause"></a>原因

Azure Resource Graph REST API 仅支持 **application/json** 的 `Content-Type`。 某些 REST 工具或代理默认为 **text/plain**，REST API 不支持此选项。

#### <a name="resolution"></a>解决方法

验证用于查询 Azure Resource Graph 的工具或代理是否为 **application/json** 配置了 REST API 标头 `Content-Type`。

### <a name="scenario-no-read-permission-to-all-subscriptions-in-list"></a><a name="rest-403"></a>方案：列表中没有对所有订阅的读取权限

#### <a name="issue"></a>问题

使用 Azure Resource Graph 查询显式传递订阅列表的客户将获得 _403_（禁止）响应。

#### <a name="cause"></a>原因

如果客户对所有提供的订阅没有读取权限，则由于缺乏适当的安全权限，请求将被拒绝。

#### <a name="resolution"></a>解决方法

在订阅列表中包括至少一个订阅，运行查询的客户至少具有对该订阅的读取访问权限。 有关详细信息，请参阅 [Azure Resource Graph 中的权限](../overview.md#permissions-in-azure-resource-graph)。

## <a name="next-steps"></a>后续步骤

如果你的问题未在本文中列出，或者无法解决问题，请访问以下渠道之一获取更多支持：

- 通过[Azure 论坛](https://azure.microsoft.com/support/forums/)获取 azure 专家的解答。
- 联系[@AzureSupport](https://twitter.com/azuresupport) –官方 Microsoft Azure 帐户，通过将 Azure 社区连接到适当的资源来改进客户体验：答案、支持和专家。
- 如需更多帮助，可以提交 Azure 支持事件。 转到[Azure 支持站点](https://azure.microsoft.com/support/options/)并选择 "**获取支持**"。