---
title: 教程：实现 IoT 空间分析 | Microsoft Azure Maps
description: 将 IoT 中心与 Microsoft Azure Maps 服务 API 集成。
author: philmea
ms.author: philmea
ms.date: 11/12/2019
ms.topic: tutorial
ms.service: azure-maps
services: azure-maps
manager: philmea
ms.custom: mvc
ms.openlocfilehash: cfea9aa7bfcc9a9698bb93bdf54797481b8539ce
ms.sourcegitcommit: 58faa9fcbd62f3ac37ff0a65ab9357a01051a64f
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/29/2020
ms.locfileid: "80333962"
---
# <a name="tutorial-implement-iot-spatial-analytics-using-azure-maps"></a>教程：使用 Azure Maps 实现 IoT 空间分析

在 IoT 场景中，通常需要捕获和跟踪时空中发生的相关事件。 示例场景包括机群管理、资产跟踪、物流和智能城市应用。 本教程提供有关使用 Azure Maps API 的解决方案模式的指导。 相关事件是 IoT 中心使用事件网格提供的事件订阅模型捕获的。

在本教程中，将：

> [!div class="checklist"]
> * 创建 IoT 中心。
> * 使用数据上传 API 在 Azure Maps 数据服务中上传地域隔离区。
> * 在 Azure Functions 中创建一个函数，基于 Azure Maps 空间分析实现业务逻辑。
> * 通过事件网格订阅 Azure 函数发出的 IoT 设备遥测事件。
> * 使用 IoT 中心消息路由筛选遥测事件。
> * 创建一个存储帐户用于存储相关的事件数据。
> * 模拟车内 IoT 设备。
    

## <a name="use-case"></a>用例

此解决方案演示以下场景：出租汽车公司打算监视并记录其出租汽车的事件。 出租汽车公司通常面向特定的地理区域出租其汽车。 它们需要跟踪租出的汽车的行程。 必须记录汽车离开所选地理区域的事件实例。 记录数据可确保正确处理政策、费用和其他业务方面的事宜。

在我们的用例中，出租汽车配备了 IoT 设备，可定期将遥测数据发送到 Azure IoT 中心。 遥测数据包括当前位置，指示汽车引擎是否正在运转。 设备位置架构符合 IoT [地理空间数据的即插即用架构](https://github.com/Azure/IoTPlugandPlay/blob/master/Schemas/geospatial.md)。 出租汽车的设备遥测架构如下所示：

```JSON
{
    "data": {
        "properties": {
            "Engine": "ON"
        },
        "systemProperties": {
            "iothub-content-type": "application/json",
            "iothub-content-encoding": "utf-8",
            "iothub-connection-device-id": "ContosoRentalDevice",
            "iothub-connection-auth-method": "{\"scope\":\"device\",\"type\":\"sas\",\"issuer\":\"iothub\",\"acceptingIpFilterRule\":null}",
            "iothub-connection-auth-generation-id": "636959817064335548",
            "iothub-enqueuedtime": "2019-06-18T00:17:20.608Z",
            "iothub-message-source": "Telemetry"
        },
        "body": { 
            "location": { 
                "type": "Point",
                "coordinates": [ -77.025988698005662, 38.9015330523316 ]
            } 
        } 
    }
}
```

让我们使用车内设备遥测数据来实现目标。 我们想要执行地理围栏规则。 此外，希望在每次收到指示汽车已转移位置的事件时做出响应。 为此，我们将通过事件网格订阅 IoT 中心发出的设备遥测事件。 

可通过多种方式订阅事件网格，本教程将使用 Azure Functions。 Azure Functions 对事件网格中发布的事件做出反应。 它还基于 Azure Maps 空间分析实现汽车出租业务逻辑。 

Azure 函数中的代码将检查车辆是否已离开地理围栏。 如果车辆已离开地理围栏，Azure 函数将收集其他信息，例如与当前位置关联的地址。 该函数还实现相应的逻辑在数据 Blob 存储中存储有意义的事件数据，以帮助提供事件情景的说明。 

事件情景可能对汽车出租公司和租用客户有帮助。 下图从较高层面提供了该系统的概览。

 
  <center>

  ![系统概览](./media/tutorial-iot-hub-maps/system-diagram.png)
  
  </center>

下图以蓝色突出显示地理围栏区域， 以绿线表示出租车的路线。

  ![地理围栏路线](./media/tutorial-iot-hub-maps/geofence-route.png)


## <a name="prerequisites"></a>先决条件 

### <a name="create-a-resource-group"></a>创建资源组

若要完成本教程中的步骤，首先需要在 Azure 门户中创建一个资源组。 若要创建资源组，请执行以下步骤：

1. 登录 [Azure 门户](https://portal.azure.com)。

2. 选择“资源组”  。
    
   ![资源组](./media/tutorial-iot-hub-maps/resource-group.png)

3. 在“资源组”下，选择“添加”。  
    
   ![添加资源组](./media/tutorial-iot-hub-maps/add-resource-group.png) 

4. 输入以下属性值：
    * **订阅：** 选择 Azure 订阅。
    * **资源组：** 输入“ContosoRental”作为资源组名称。
    * **区域：** 选择资源组所在的区域。  

    ![资源组详细信息](./media/tutorial-iot-hub-maps/resource-details.png)

    选择“查看 + 创建”，然后在下一页上选择“创建”。  

### <a name="create-an-azure-maps-account"></a>创建 Azure Maps 帐户 

若要基于 Azure Maps 空间分析实现业务逻辑，需要在创建的资源组中创建一个 Azure Maps 帐户。 按照[创建帐户](quick-demo-map-app.md#create-an-account-with-azure-maps)中的说明，在 S1 定价层中创建一个 Azure Maps 帐户订阅。 按照[获取主密钥](quick-demo-map-app.md#get-the-primary-key-for-your-account)中的步骤获取帐户的主密钥。 有关 Azure Maps 中身份验证的详细信息，请参阅[在 Azure Maps 中管理身份验证](how-to-manage-authentication.md)。



### <a name="create-a-storage-account"></a>创建存储帐户

为了记录事件数据，我们将创建一个常规用途 v2storage，用于访问所有 Azure 存储服务：Blob、文件、队列、表和磁盘  。  我们需要将此存储帐户放置在“ContosoRental”资源组中，以便将数据存储为 Blob。 若要创建存储帐户，请遵照[创建存储帐户](https://docs.microsoft.com/azure/storage/common/storage-quickstart-create-account?toc=%2Fazure%2Fstorage%2Fblobs%2Ftoc.json&tabs=azure-portal)中的说明。 接下来，需要创建一个用于存储 Blob 的容器。 为此，请执行以下步骤：

1. 在“存储帐户 - Blob、文件、表、队列”中导航到“容器”。

    ![Blob](./media/tutorial-iot-hub-maps/blobs.png)

2. 单击左上角的容器按钮，将容器命名为“contoso-rental-logs”，然后单击“确定”。

    ![blob-container](./media/tutorial-iot-hub-maps/blob-container.png)

3. 导航到存储帐户中的“访问密钥”边栏选项卡，并复制“存储帐户名称”和“访问密钥”，  因为在后面的步骤中需要用到。

    ![access-keys](./media/tutorial-iot-hub-maps/access-keys.png)


现在我们已获取存储帐户以及用于记录事件数据的容器。 接下来创建一个 IoT 中心。

### <a name="create-an-iot-hub"></a>创建 IoT 中心

IoT 中心是云中的托管服务。 IoT 中心充当中央消息中心，用于在 IoT 应用程序及其管理的设备之间进行双向通信。 若要将设备遥测消息路由到事件网格，请在“ContosoRental”资源组中创建一个 IoT 中心。 设置消息路由集成，在其中我们基于汽车的发动机状态筛选消息。 此外，每当汽车移动位置时，我们就会将设备遥测消息发送到事件网格。

> [!Note] 
> 在事件网格中发布设备遥测事件的 IoT 中心功能目前以公共预览版提供。 公共预览版功能适用于除**美国东部、美国西部、西欧、Azure 政府、Azure 中国世纪互联**和 **Azure 德国**以外的所有地区。 

遵循[创建 IoT 中心](https://docs.microsoft.com/azure/iot-hub/quickstart-send-telemetry-dotnet#create-an-iot-hub)部分的步骤创建 IoT 中心。


### <a name="register-a-device"></a>注册设备 

若要连接到 IoT 中心，必须注册设备。 若要将设备注册到 IoT 中心，请执行以下步骤：

1. 在 IoT 中心，依次单击“IoT 设备”边栏选项卡、“新建”。

    ![add-device](./media/tutorial-iot-hub-maps/add-device.png)

2. 在“创建设备”页上为 IoT 设备命名，然后单击“保存”。
    
    ![register-device](./media/tutorial-iot-hub-maps/register-device.png)

3. 保存设备的**主连接字符串**，以便在一个需要使用此连接字符串更改占位符的稍后步骤中使用它。

    ![add-device](./media/tutorial-iot-hub-maps/connection-string.png)

## <a name="upload-geofence"></a>上传地理围栏

我们将使用 [Postman 应用程序](https://www.getpostman.com)通过 Azure Maps 数据上传 API [将地理围栏上传](https://docs.microsoft.com/azure/azure-maps/geofence-geojson)到 Azure Maps 服务。 当汽车处于此地理围栏以外时发生的任何事件都会得到记录。

打开 Postman 应用，并遵循以下步骤使用 Azure Maps 数据上传 API 上传地理围栏。  

1. 在 Postman 应用中，单击“新建”|“新建”，然后选择“请求”。 输入“上传地域隔离区数据”的请求名称，选择用于保存该请求的集合或文件夹，然后单击“保存”。

    ![使用 Postman 上传地理围栏](./media/tutorial-iot-hub-maps/postman-new.png)

2. 在生成器选项卡上选择 POST HTTP 方法，并输入以下 URL 发出 POST 请求。

    ```HTTP
    https://atlas.microsoft.com/mapData/upload?subscription-key={subscription-key}&api-version=1.0&dataFormat=geojson
    ```
    
    URL 路径中 `dataFormat` 参数对应的“geojson”值表示正在上传的数据的格式。

3. 单击“参数”，输入用于 POST 请求 URL 的以下键/值对。  将 subscription-key 值替换为 Azure Maps 密钥。
   
    ![Postman 中的键值对参数](./media/tutorial-iot-hub-maps/postman-key-vals.png)

4. 单击“正文”，选择“raw”输入格式，然后从下拉列表中选择“JSON (application/text)”作为输入格式。    打开[此处](https://raw.githubusercontent.com/Azure-Samples/iothub-to-azure-maps-geofencing/master/src/Data/geofence.json?token=AKD25BYJYKDJBJ55PT62N4C5LRNN4)的 JSON 数据文件，将 Json 复制到正文节作为要上传的数据，然后单击“发送”。 
    
    ![发布数据](./media/tutorial-iot-hub-maps/post-json-data.png)
    
5. 查看响应标头。 成功请求后，**Location** 标头将包含状态 URI，可在其中检查上传请求的当前状态。 状态 URI 采用以下格式。 

   ```HTTP
   https://atlas.microsoft.com/mapData/{uploadStatusId}/status?api-version=1.0
   ```

6. 复制状态 URI 并将 `subscription-key` 参数追加到其后。 将 Azure Maps 帐户订阅密钥的值分配给 `subscription-key` 参数。 状态 URI 格式应如下所示，并且 `{Subscription-key}` 应替换为你的订阅密钥。

   ```HTTP
   https://atlas.microsoft.com/mapData/{uploadStatusId}/status?api-version=1.0&subscription-key={Subscription-key}
   ```

7. 若要获取 `udId`，请在 Postman 应用中打开一个新选项卡，在生成器选项卡上选择 GET HTTP 方法，并在状态 URI 中发出 GET 请求。 如果数据上传成功，将在响应正文中收到一个 udId。 请复制该 udId 供稍后使用。

   ```JSON
   {
    "udid" : "{udId}"
   }
   ```


接下来，在“ContosoRental”资源组中创建一个 Azure 函数，然后在 IoT 中心设置一个消息路由用于筛选设备遥测消息。


## <a name="create-an-azure-function-and-add-an-event-grid-subscription"></a>创建 Azure 函数并添加事件网格订阅

Azure Functions 是一个无服务器计算服务，使用它可以按需运行代码，而无需显式预配或管理计算基础结构。 有关 Azure Functions 的详细信息，请参阅 [Azure Functions](https://docs.microsoft.com/azure/azure-functions/functions-overview) 文档。 

在函数中实现的逻辑使用来自车内设备遥测功能的位置数据来评估地理围栏状态。 如果给定车辆处于地理围栏以外，该函数将通过[获取搜索地址反向 API](https://docs.microsoft.com/rest/api/maps/search/getsearchaddressreverse) 收集更多信息（例如位置的地址）。 此 API 可将给定的位置坐标转换为人类可理解的街道地址。 

然后，所有相关事件信息将保存在 Blob 存储中。 以下步骤 5 指向实现此类逻辑的可执行代码。 遵循以下步骤创建一个 Azure 函数用于将数据日志发送到 Blob 存储帐户中的 Blob 容器，并添加对该容器的事件网格订阅。

1. 在 Azure 门户仪表板中，选择“创建资源”。 从可用资源类型列表中选择“计算”，然后选择“函数应用”。  

    ![create-resource](./media/tutorial-iot-hub-maps/create-resource.png)

2. 在“函数应用”创建页上为函数应用命名。  在“资源组”下选择“使用现有项”，然后从下拉列表中选择“ContosoRental”。   选择“.NET Core”作为运行时堆栈。 在“承载”  下，对于“存储帐户”  ，请选择前面的步骤中的存储帐户名称。 在前面的步骤中，我们已将存储帐户命名为 **v2storage**。  然后，选择“查看 + 创建”  。
    
    ![create-app](./media/tutorial-iot-hub-maps/rental-app.png)

2. 查看函数应用详细信息，然后选择“创建”。

3. 创建应用后，需要向其添加一个函数。 转到函数应用。 单击“新建函数”以添加一个函数，然后选择“门户中”作为开发环境。   然后选择“继续”。 

    ![create-function](./media/tutorial-iot-hub-maps/function.png)

4. 选择“更多模板”并单击“完成并查看模板”。   

5. 选择包含 **Azure 事件网格触发器**的模板。 根据提示安装扩展，将函数命名，然后选择“创建”。 

    ![function-template](./media/tutorial-iot-hub-maps/eventgrid-funct.png)
    
    **Azure 事件中心触发器**和 **Azure 事件网格触发器**具有相似的图标。 请确保选择“Azure 事件网格触发器”  。

6. 将 [C# 代码](https://github.com/Azure-Samples/iothub-to-azure-maps-geofencing/blob/master/src/Azure%20Function/run.csx)复制到函数中。
 
7. 在 C# 脚本中替换以下参数。 单击“ **保存**”。 目前还不要单击“运行” 
    * 将 **SUBSCRIPTION_KEY** 替换为 Azure Maps 帐户主订阅密钥。
    * 将 **UDID** 替换为上传的地理围栏的 udId。 
    * 脚本中的 **CreateBlobAsync** 函数将为数据存储帐户中的每个事件创建一个 Blob。 将 **ACCESS_KEY**、**ACCOUNT_NAME** 和 **STORAGE_CONTAINER_NAME** 替换为存储帐户的访问密钥、帐户名称和数据存储容器。

10. 单击“添加事件网格订阅”。 
    
    ![add-event-grid](./media/tutorial-iot-hub-maps/add-egs.png)

11. 填写订阅详细信息，在“事件订阅详细信息”下为你的事件订阅命名。  对于事件架构，请选择“事件网格架构”。 在“主题详细信息”下选择“Azure IoT 中心帐户”作为主题类型。  选择用于创建资源组的同一订阅，选择“ContosoRental”作为“资源组”。 选择创建的 IoT 中心作为“资源”。 选择“设备遥测”作为“事件类型”。  选择这些选项后，“主题类型”会自动更改为“IoT 中心”。

    ![event-grid-subscription](./media/tutorial-iot-hub-maps/af-egs.png)
 

## <a name="filter-events-using-iot-hub-message-routing"></a>使用 IoT 中心消息路由筛选事件

将事件网格订阅添加到 Azure 函数后，可以在 IoT 中心的“消息路由”边栏选项卡中看到事件网格的默认消息路由。  消息路由使你能够将不同的数据类型路由到各个终结点。 例如，可以路由设备遥测消息、设备生命周期事件和设备孪生更改事件。 若要详细了解 IoT 中心消息路由，请参阅[使用 IoT 中心消息路由](https://docs.microsoft.com/azure/iot-hub/iot-hub-devguide-messages-d2c)。

![hub-EG-route](./media/tutorial-iot-hub-maps/hub-route.png)

在示例方案中，我们希望筛选出与出租车辆转移位置相关的所有消息。 为了将此类设备遥测事件发布到事件网格，我们将使用路由查询来筛选 `Engine` 属性为 **"ON"** 的事件。 可通过多种方式查询 IoT 设备到云的消息。若要详细了解消息路由语法，请参阅 [IoT 中心消息路由](https://docs.microsoft.com/azure/iot-hub/iot-hub-devguide-routing-query-syntax)。 若要创建路由查询，请单击“RouteToEventGrid”路由，将 **Routing query** 替换为 **"Engine='ON'"** ，然后单击“保存”。   现在，IoT 中心只会发布引擎处于打开状态时的设备遥测数据。

![hub-EG-filter](./media/tutorial-iot-hub-maps/hub-filter.png)


## <a name="send-telemetry-data-to-iot-hub"></a>将遥测数据发送到 IoT 中心

Azure 函数启动并运行后，我们现在可将遥测数据发送到 IoT 中心，而 IoT 中心会将其路由到事件网格。 让我们使用一个 C# 应用程序来模拟出租汽车的车内设备的定位数据。 若要运行该应用程序，需要在开发计算机上安装 .NET Core SDK 2.1.0 或更高版本。 遵循以下步骤将模拟的遥测数据发送到 IoT 中心。

1. 下载 [rentalCarSimulation](https://github.com/Azure-Samples/iothub-to-azure-maps-geofencing/tree/master/src/rentalCarSimulation) C# 项目。 

2. 在所选的文本编辑器中打开 simulatedCar.cs 文件，将 `connectionString` 的值替换为在注册设备时保存的值，然后保存对该文件所做的更改。
 
3. 确保已在计算机上安装 .NET Core。 在本地终端窗口中，导航到 C# 项目的根文件夹，运行以下命令以安装模拟设备应用程序所需的包：
    
    ```cmd/sh
    dotnet restore
    ```

4. 在同一终端中运行以下命令，以生成并运行出租汽车模拟应用程序：

    ```cmd/sh
    dotnet run
    ```

  本地终端应如下所示。

  ![终端输出](./media/tutorial-iot-hub-maps/terminal.png)

如果现在打开 Blob 存储容器，应该可以看到四个 Blob，它们对应于车辆位于地理围栏以外时所处的位置。

![进入 Blob](./media/tutorial-iot-hub-maps/blob.png)

下图显示了定期记录的、当车辆位于地理围栏以外时所处的四个点。

![违规图](./media/tutorial-iot-hub-maps/violation-map.png)

## <a name="next-steps"></a>后续步骤

若要浏览本教程中使用的 Azure Maps API，请参阅：

* [获取反向搜索地址](https://docs.microsoft.com/rest/api/maps/search/getsearchaddressreverse)
* [获取地理围栏](https://docs.microsoft.com/rest/api/maps/spatial/getgeofence)

有关 Azure Maps REST API 的完整列表，请参阅：

* [Azure Maps REST API](https://docs.microsoft.com/rest/api/maps/spatial/getgeofence)

若要详细了解 IoT 即插即用，请参阅：

* [IoT 即插即用](https://docs.microsoft.com/azure/iot-pnp)

若要获取 Azure 认证的 IoT 设备列表，请访问：

* [Azure 认证的设备](https://catalog.azureiotsolutions.com/)

若要详细了解如何将设备发送到云遥测以及执行反向发送，请参阅：

* [从设备发送遥测数据](https://docs.microsoft.com/azure/iot-hub/quickstart-send-telemetry-dotnet)
