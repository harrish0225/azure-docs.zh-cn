---
title: 响应 Azure Blob 存储事件 | Microsoft Docs
description: 使用 Azure 事件网格订阅 Blob 存储事件。
author: normesta
ms.author: normesta
ms.date: 04/06/2020
ms.topic: conceptual
ms.service: storage
ms.subservice: blobs
ms.reviewer: cbrooks
ms.openlocfilehash: d9c666fd6fcf020908b6fc5bdd639261853ad9c6
ms.sourcegitcommit: 849bb1729b89d075eed579aa36395bf4d29f3bd9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/28/2020
ms.locfileid: "80811541"
---
# <a name="reacting-to-blob-storage-events"></a>响应 Blob 存储事件

Azure 存储事件允许应用程序响应事件，例如 Blob 的创建和删除。 为此，它无需复杂的代码或高价低效的轮询服务。 最大好处是，使用多少就付多少费用。

使用[Azure 事件网格](https://azure.microsoft.com/services/event-grid/)将 Blob 存储事件推送到订阅服务器（例如 Azure Functions、Azure 逻辑应用，甚至是自己的 http 侦听器）。 事件网格通过丰富的重试策略和死信，为应用程序提供可靠的事件传递。

若要查看 Blob 存储支持的事件的完整列表，请参阅[blob 存储事件架构](../../event-grid/event-schema-blob-storage.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json)一文。

常见的 Blob 存储事件方案包括图像或视频处理、搜索索引或任何面向文件的工作流。 异步文件上传十分适合事件。 基于事件的体系结构对于鲜少更改，但要求立即响应的情况尤为有效。

若要尝试 blob 存储事件，请参阅以下任一快速入门文章：

|若要使用此工具：    |请参阅此文： |
|--|-|
|Azure 门户    |[快速入门：利用 Azure 门户将 Blob 存储事件路由到 Web 终结点](https://docs.microsoft.com/azure/event-grid/blob-event-quickstart-portal?toc=%2fazure%2fstorage%2fblobs%2ftoc.json)|
|PowerShell    |[快速入门：通过 PowerShell 将存储事件路由到 web 终结点](https://docs.microsoft.com/azure/storage/blobs/storage-blob-event-quickstart-powershell?toc=%2fazure%2fstorage%2fblobs%2ftoc.json)|
|Azure CLI    |[快速入门：将存储事件路由到具有 Azure CLI 的 web 终结点](https://docs.microsoft.com/azure/storage/blobs/storage-blob-event-quickstart?toc=%2fazure%2fstorage%2fblobs%2ftoc.json)|

若要深入了解使用 Azure 函数对 Blob 存储事件做出反应的详细示例，请参阅以下文章：

- [教程：使用 Azure Data Lake Storage Gen2 事件更新 Databricks 增量表](data-lake-storage-events.md)。
- [教程：使用事件网格自动调整上传图像的大小](https://docs.microsoft.com/azure/event-grid/resize-images-on-storage-blob-upload-event?tabs=dotnet)

>[!NOTE]
> 只有类型**StorageV2 （常规用途 v2）**、 **BlockBlobStorage**和**BlobStorage**的存储帐户支持事件集成。 “存储(常规用途 v1)”  不  支持与事件网格集成。

## <a name="the-event-model"></a>事件模型

事件网格使用[事件订阅](../../event-grid/concepts.md#event-subscriptions)将事件消息路由到订阅服务器。 此图展示了事件发布者、事件订阅和事件处理程序之间的关系。

![事件网格模型](./media/storage-blob-event-overview/event-grid-functional-model.png)

首先，将终结点订阅到事件。 然后，在触发某个事件时，事件网格服务会将有关该事件的数据发送到终结点。

请参阅 [Blob 存储事件架构](../../event-grid/event-schema-blob-storage.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json)一文，以便查看：

> [!div class="checklist"]
> * Blob 存储事件的完整列表，以及如何触发每个事件。
> * 事件网格会针对每个此类事件发送的数据的示例。
> * 显示在数据中的每个键值对的用途。

## <a name="filtering-events"></a>筛选事件

可以按事件类型、容器名称或已创建/删除的对象的名称来[筛选 Blob 事件](/cli/azure/eventgrid/event-subscription?view=azure-cli-latest)。 事件网格中的筛选器与主题的开头或结尾匹配，因此具有匹配的主题的事件会转到订阅服务器。

若要详细了解如何应用筛选器，请参阅[筛选事件网格的事件](https://docs.microsoft.com/azure/event-grid/how-to-filter-events)。

Blob 存储事件使用者使用的格式：

```
/blobServices/default/containers/<containername>/blobs/<blobname>
```

要匹配存储帐户的所有事件，可将主题筛选器留空。

要匹配在一组共享前缀的容器中创建的 blob 的事件，请使用 `subjectBeginsWith` 筛选器，如下所示：

```
/blobServices/default/containers/containerprefix
```

要匹配在特定容器中创建的 blob 的事件，请使用 `subjectBeginsWith` 筛选器，如下所示：

```
/blobServices/default/containers/containername/
```

要匹配在共享 blob 名称前缀的特定容器中创建的 blob 的事件，请使用 `subjectBeginsWith` 筛选器，如下所示：

```
/blobServices/default/containers/containername/blobs/blobprefix
```

要匹配在共享 blob 后缀的特定容器中创建的 blob 事件，请使用 `subjectEndsWith` 筛选器，例如“.log”或“.jpg”。 有关详细信息，请参阅 [事件网格概念](../../event-grid/concepts.md#event-subscriptions)。

## <a name="practices-for-consuming-events"></a>使用事件的做法

处理 Blob 存储事件的应用程序应遵循以下建议的做法：
> [!div class="checklist"]
> * 由于可将多个订阅配置为将事件路由至相同的事件处理程序，因此请勿假定事件来自特定的源，而是应检查消息的主题，确保它来自所期望的存储帐户。
> * 同样，检查 eventType 是否为准备处理的项，并且不假定所接收的全部事件都是期望的类型。
> * 当消息可以在延迟后到达后，请使用 etag 字段来了解有关对象的信息是否仍然是最新的。 若要了解如何使用 etag 字段，请参阅[在 Blob 存储中管理并发](https://docs.microsoft.com/azure/storage/common/storage-concurrency?toc=%2fazure%2fstorage%2fblobs%2ftoc.json#managing-concurrency-in-blob-storage)。 
> * 当消息可以不按顺序到达时，使用 sequencer 字段来了解任何特定对象上事件的顺序。 Sequencer 字段是一个字符串值，表示任何特定 blob 名称的事件的逻辑顺序。 您可以使用标准字符串比较来了解同一 blob 名称上的两个事件的相对顺序。
> * 存储事件保证至少一次传递到订阅服务器，这可确保输出所有消息。 但是，由于订阅重试或可用性，可能偶尔会出现重复消息。 若要详细了解消息传递和重试，请参阅[事件网格消息传递和重试](../../event-grid/delivery-and-retry.md)。
> * 使用 blobType 字段可了解 blob 中允许何种类型的操作，以及应当使用哪种客户端库类型来访问该 blob。 有效值为 `BlockBlob` 或 `PageBlob`。 
> * 将 URL 字段与 `CloudBlockBlob` 和 `CloudAppendBlob` 构造函数配合使用，以访问 blob。
> * 忽略不了解的字段。 此做法有助于适应将来可能添加的新功能。
> * 如果你想要确保仅在完全提交块 Blob 时触发**BlobCreated**事件`CopyBlob`，请筛选`PutBlob`、 `PutBlockList`或`FlushWithClose` REST API 调用的事件。 这些 API 调用仅在数据已完全提交到块 Blob 后才触发 **Microsoft.Storage.BlobCreated** 事件。 若要了解如何创建筛选器，请参阅[筛选事件网格的事件](https://docs.microsoft.com/azure/event-grid/how-to-filter-events)。


## <a name="next-steps"></a>后续步骤

了解关于事件网格的详细信息，尝试一下 Blob 存储事件：

- [关于事件网格](../../event-grid/overview.md)
- [Blob 存储事件架构](../../event-grid/event-schema-blob-storage.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json)
- [将 Blob 存储事件路由到自定义 web 终结点](storage-blob-event-quickstart.md)
