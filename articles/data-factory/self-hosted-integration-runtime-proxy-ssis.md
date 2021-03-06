---
title: 将自承载集成运行时配置为 SSIS 的代理
description: 了解如何将自承载集成运行时配置为 Azure-SSIS Integration Runtime 的代理。
services: data-factory
documentationcenter: ''
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
author: swinarko
ms.author: sawinark
ms.reviewer: douglasl
manager: mflasko
ms.custom: seo-lt-2019
ms.date: 04/15/2020
ms.openlocfilehash: 4cb5b84f3889dcf4e0f28d525afb42cfeac5b54c
ms.sourcegitcommit: 849bb1729b89d075eed579aa36395bf4d29f3bd9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/28/2020
ms.locfileid: "81605498"
---
# <a name="configure-a-self-hosted-ir-as-a-proxy-for-an-azure-ssis-ir-in-azure-data-factory"></a>自承载 IR 配置为 Azure 数据工厂中 Azure-SSIS IR 的代理

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

本文介绍如何在将某个自承载集成运行时（自承载 IR）配置为代理的情况下，在 Azure 数据工厂中的 Azure-SSIS Integration Runtime (Azure-SSIS IR) 上运行 SQL Server Integration Services (SSIS) 包。 

使用此功能可在本地访问数据，而无需[将 Azure-SSIS IR 加入虚拟网络](https://docs.microsoft.com/azure/data-factory/join-azure-ssis-integration-runtime-virtual-network)。 当企业网络的配置过于复杂，或者采用过于严格的策略，以致你很难在此网络中注入 Azure-SSIS IR 时，此功能将很有用。

此功能会将具有本地数据源的任何 SSIS 数据流任务拆分为两个临时任务： 
* 第一个任务在自承载 IR 中运行，它首先将数据从本地数据源移入 Azure Blob 存储中的暂存区域。
* 第二个任务在 Azure-SSIS IR 中运行，它随后将数据从暂存区域移入预期的数据目标。

举例而言，此功能的其他优势和功能使你能够在 Azure-SSIS IR 尚不支持的区域中设置自承载 IR，并在数据源的防火墙中允许自承载 IR 的公共静态 IP 地址。

## <a name="prepare-the-self-hosted-ir"></a>准备自承载 IR

若要使用此功能，请先创建一个数据工厂，然后在其中设置 Azure-SSIS IR。 如果尚未执行此操作，请按照[设置 Azure-SSIS IR](https://docs.microsoft.com/azure/data-factory/tutorial-deploy-ssis-packages-azure) 中的说明操作。

然后，在设置了 Azure-SSIS IR 的同一数据工厂中设置自承载 IR。 为此，请参阅[创建自承载 IR](https://docs.microsoft.com/azure/data-factory/create-self-hosted-integration-runtime)。

最后，按如下所述，在本地计算机或 Azure 虚拟机 (VM) 上下载并安装最新版本的自承载 IR 以及其他驱动程序和运行时：
- 下载并安装最新版本的[自承载 IR](https://www.microsoft.com/download/details.aspx?id=39717)。
- 如果使用包中的对象链接与嵌入数据库 (OLEDB) 连接器，请在安装了自承载 IR 的同一台计算机上下载并安装相关的 OLEDB 驱动程序（如果尚未这样做）。  

  如果使用早期版本的用于 SQL Server 的 OLEDB 驱动程序 (SQL Server Native Client [SQLNCLI])，请[下载 64 位版本](https://www.microsoft.com/download/details.aspx?id=50402)。  

  如果使用最新版本的用于 SQL Server 的 OLEDB 驱动程序 (MSOLEDBSQL)，请[下载 64 位版本](https://www.microsoft.com/download/details.aspx?id=56730)。  
  
  如果使用用于其他数据库系统（例如 PostgreSQL、MySQL、Oracle 等）的 OLEDB 驱动程序，可以从其网站下载 64 位版本。
- 如果尚未这样做，请在安装了自承载 IR 的同一台计算机上[下载并安装 64 位版本的 Visual C++ (VC) 运行时](https://www.microsoft.com/download/details.aspx?id=40784)。

## <a name="prepare-the-azure-blob-storage-linked-service-for-staging"></a>准备用于暂存的 Azure Blob 存储链接服务

如果尚未这样做，请在设置了 Azure-SSIS IR 的同一数据工厂中创建一个 Azure Blob 存储链接服务。 为此，请参阅[创建 Azure 数据工厂链接服务](https://docs.microsoft.com/azure/data-factory/quickstart-create-data-factory-portal#create-a-linked-service)。 确保执行以下操作：
- 对于“数据存储”，请选择“Azure Blob 存储”。    
- 对于 "**通过集成运行时连接**"，请选择 " **AutoResolveIntegrationRuntime** " （而不是你的 AZURE-SSIS IR 和自承载 IR），因为我们使用默认 Azure IR 来提取 Azure Blob 存储的访问凭据。
- 对于“身份验证方法”，请选择“帐户密钥”、“SAS URI”或“服务主体”。      

    >[!TIP]
    >如果选择**服务主体**方法，请至少向服务主体授予 " *存储 Blob 数据参与者* " 角色。 有关详细信息，请参阅  [Azure Blob 存储连接器](connector-azure-blob-storage.md#linked-service-properties)。

![准备用于暂存的 Azure Blob 存储链接服务](media/self-hosted-integration-runtime-proxy-ssis/shir-azure-blob-storage-linked-service.png)

## <a name="configure-an-azure-ssis-ir-with-your-self-hosted-ir-as-a-proxy"></a>配置使用自承载 IR 作为代理的 Azure-SSIS IR 配置

准备好用于暂存的自承载 IR 和 Azure Blob 存储链接服务后，接下来可以在数据工厂门户或应用中，配置使用自承载 IR 作为代理的新的或现有 Azure-SSIS IR。 不过，在执行此操作之前，如果现有的 Azure-SSIS IR 已运行，请停止再重启它。

1. 在“集成运行时设置”窗格中，选择“下一步”跳过“常规设置”和“SQL 设置”部分。     

1. 在“高级设置”部分执行以下操作： 

   1. 选中“将自承载集成运行时设置为 Azure-SSIS Integration Runtime 的代理”复选框。  

   1. 在“自承载集成运行时”下拉列表中，选择现有的自承载 IR 作为 Azure-SSIS IR 的代理。 

   1. 在“暂存存储链接服务”下拉列表中，选择现有的 Azure Blob 存储链接服务，或创建新的服务用于暂存。 

   1. 在“暂存路径”框中，指定所选 Azure Blob 存储帐户中的某个 Blob 容器，或将其留空以使用默认容器进行暂存。 

   1. 选择“继续”。 

   ![自承载 IR 的高级设置](./media/tutorial-create-azure-ssis-runtime-portal/advanced-settings-shir.png)

还可以使用 PowerShell 来配置使用自承载 IR 作为代理的新的或现有 Azure-SSIS IR。

```powershell
$ResourceGroupName = "[your Azure resource group name]"
$DataFactoryName = "[your data factory name]"
$AzureSSISName = "[your Azure-SSIS IR name]"
# Self-hosted integration runtime info - This can be configured as a proxy for on-premises data access 
$DataProxyIntegrationRuntimeName = "" # OPTIONAL to configure a proxy for on-premises data access 
$DataProxyStagingLinkedServiceName = "" # OPTIONAL to configure a proxy for on-premises data access 
$DataProxyStagingPath = "" # OPTIONAL to configure a proxy for on-premises data access 

# Add self-hosted integration runtime parameters if you configure a proxy for on-premises data accesss
if(![string]::IsNullOrEmpty($DataProxyIntegrationRuntimeName) -and ![string]::IsNullOrEmpty($DataProxyStagingLinkedServiceName))
{
    Set-AzDataFactoryV2IntegrationRuntime -ResourceGroupName $ResourceGroupName `
        -DataFactoryName $DataFactoryName `
        -Name $AzureSSISName `
        -DataProxyIntegrationRuntimeName $DataProxyIntegrationRuntimeName `
        -DataProxyStagingLinkedServiceName $DataProxyStagingLinkedServiceName

    if(![string]::IsNullOrEmpty($DataProxyStagingPath))
    {
        Set-AzDataFactoryV2IntegrationRuntime -ResourceGroupName $ResourceGroupName `
            -DataFactoryName $DataFactoryName `
            -Name $AzureSSISName `
            -DataProxyStagingPath $DataProxyStagingPath
    }
}
Start-AzDataFactoryV2IntegrationRuntime -ResourceGroupName $ResourceGroupName `
    -DataFactoryName $DataFactoryName `
    -Name $AzureSSISName `
    -Force
```

## <a name="enable-ssis-packages-to-connect-by-proxy"></a>使 SSIS 包能够通过代理进行连接

使用适用于 Visual Studio 的最新 SSDT 和 SSIS Projects 扩展或使用独立安装程序，可以发现“OLEDB”或“平面文件”连接管理器中已添加一个新的 `ConnectByProxy` 属性。
* [下载适用于 Visual Studio 的 SSDT 和 SSIS Projects 扩展](https://marketplace.visualstudio.com/items?itemName=SSIS.SqlServerIntegrationServicesProjects)
* [下载独立安装程序](https://docs.microsoft.com/sql/ssdt/download-sql-server-data-tools-ssdt?view=sql-server-2017#ssdt-for-vs-2017-standalone-installer)   

使用“OLEDB”或“平面文件”源（可用于在本地访问数据库或文件）设计包含数据流任务的新包时，可以通过在相关连接管理器的“属性”窗格中，将此属性设置为 *True* 来启用此属性。 

![启用 ConnectByProxy 属性](media/self-hosted-integration-runtime-proxy-ssis/shir-connection-manager-properties.png)

还可以在运行现有包时启用此属性，而无需逐个手动更改其设置。  有两个选项：
- **选项 A**：打开、重新生成并重新部署包含这些包的项目。这些包中包含要在 Azure-SSIS IR 上运行的最新 SSDT。 然后，可以针对相关的连接管理器，通过将此属性设置为 *True* 来启用此属性。 从 SSMS 运行包时，这些连接管理器将显示在“执行包”弹出窗口的“连接管理器”选项卡上。  

  ![启用 ConnectByProxy 属性 2](media/self-hosted-integration-runtime-proxy-ssis/shir-connection-managers-tab-ssms.png)

  在数据工厂管道中运行包时，对于“[执行 SSIS 包活动](https://docs.microsoft.com/azure/data-factory/how-to-invoke-ssis-package-ssis-activity)”的“连接管理器”选项卡上显示的相关连接管理器，还可以通过将此属性设置为 True 来启用此属性。  
  
  ![启用 ConnectByProxy 属性 3](media/self-hosted-integration-runtime-proxy-ssis/shir-connection-managers-tab-ssis-activity.png)

- **选择 B**：重新部署包含这些包的项目，以在 SSIS IR 上运行。 然后，可以在从 SSMS 运行包时，通过在“执行包”弹出窗口的“高级”选项卡上提供此属性的属性路径 `\Package.Connections[YourConnectionManagerName].Properties[ConnectByProxy]`，并将其设置为 *True* 作为属性重写，来启用此属性。  

  ![启用 ConnectByProxy 属性 4](media/self-hosted-integration-runtime-proxy-ssis/shir-advanced-tab-ssms.png)

  在数据工厂管道中运行包时，还可以通过在“[执行 SSIS 包活动](https://docs.microsoft.com/azure/data-factory/how-to-invoke-ssis-package-ssis-activity)”的“属性重写”选项卡上提供此属性的属性路径 `\Package.Connections[YourConnectionManagerName].Properties[ConnectByProxy]`，并将其设置为 True 作为属性重写，来启用此属性。  
  
  ![启用 ConnectByProxy 属性 5](media/self-hosted-integration-runtime-proxy-ssis/shir-property-overrides-tab-ssis-activity.png)

## <a name="debug-the-first-and-second-staging-tasks"></a>调试第一和第二个暂存任务

在自承载 IR 上，可以在 *C:\ProgramData\SSISTelemetry* 文件夹中找到运行时日志，并在 *C:\ProgramData\SSISTelemetry\ExecutionLog* 文件夹中找到第一个暂存任务的执行日志。  可以在 SSISDB 或指定的日志路径中找到第二个暂存任务的执行日志，具体取决于是将包存储在 SSISDB、文件系统、文件共享还是 Azure 文件存储中。 还可以在第二个暂存任务的执行日志中找到第一个暂存任务的唯一 ID。 

![第一个暂存任务的唯一 ID](media/self-hosted-integration-runtime-proxy-ssis/shir-first-staging-task-guid.png)

## <a name="use-windows-authentication-in-staging-tasks"></a>在暂存任务中使用 Windows 身份验证

如果自承载 IR 上的暂存任务需要 Windows 身份验证，请[将 SSIS 包配置为使用相同的 Windows 身份验证](https://docs.microsoft.com/sql/integration-services/lift-shift/ssis-azure-connect-with-windows-auth?view=sql-server-ver15)。 

将使用自承载 IR 服务帐户（默认为 *NT SERVICE\DIAHostService*）调用暂存任务，将使用 Windows 身份验证帐户访问数据存储。 需要向这两个帐户分配特定的安全策略。 在自承载 IR 计算机上，转到“本地安全策略” > “本地策略” > “用户权限分配”，然后执行以下操作：   

1. 向自承载 IR 服务帐户分配“调整进程的内存配额”和“替换进程级令牌”策略。   当你安装具有默认服务帐户的自承载 IR 时，会自动发生这种情况。 如果不是，则手动分配这些策略。 如果使用不同的服务帐户，则为其分配相同的策略。

1. 将 "*作为服务登录*" 策略分配给 Windows 身份验证帐户。

## <a name="billing-for-the-first-and-second-staging-tasks"></a>第一和第二个过渡任务的计费

在自承载 IR 上运行的第一个暂存任务将单独计费，就像在自承载 IR 上运行的任何数据移动活动计费一样。 这是在[Azure 数据工厂数据管道定价](https://azure.microsoft.com/pricing/details/data-factory/data-pipeline/)一文中指定的。

不会单独对 Azure-SSIS IR 上运行的第二个暂存任务计费，但正在运行的 Azure-SSIS IR 按[Azure-SSIS IR 定价](https://azure.microsoft.com/pricing/details/data-factory/ssis/)文章中指定的方式计费。

## <a name="enabling-tls-12"></a>启用 TLS 1.2

如果需要使用强加密/更安全的网络协议（TLS 1.2）并在自承载 IR 上禁用旧版 SSL/TLS 版本，你可以下载并运行可在公共预览版容器的*CustomSetupScript/UserScenarios/tls 1.2*文件夹中找到的*主 .cmd*脚本。  使用[Azure 存储资源管理器](https://storageexplorer.com/)，可以通过输入以下 SAS URI 连接到公共预览版容器：

`https://ssisazurefileshare.blob.core.windows.net/publicpreview?sp=rl&st=2020-03-25T04:00:00Z&se=2025-03-25T04:00:00Z&sv=2019-02-02&sr=c&sig=WAD3DATezJjhBCO3ezrQ7TUZ8syEUxZZtGIhhP6Pt4I%3D`

## <a name="current-limitations"></a>当前限制

- 目前仅支持具有开放式数据库连接（ODBC）/OLEDB/Flat 文件源或 OLEDB 目标的数据流任务。 
- 目前仅支持使用*帐户密钥*、*共享访问签名（SAS） URI*或*服务主体*身份验证配置的 Azure Blob 存储链接服务。
- 尚不支持 OLEDB 源中的*ParameterMapping* 。 作为一种解决方法，请使用*变量中的 SQL 命令*作为*AccessMode* ，并使用*表达式*在 SQL 命令中插入变量/参数。 如图所示，请参见公共预览版容器的*SelfHostedIRProxy/限制*文件夹中的*ParameterMappingSample. .dtsx*包。 使用 Azure 存储资源管理器，可以通过输入上述 SAS URI 连接到公共预览版容器。

## <a name="next-steps"></a>后续步骤

将自承载 IR 配置为你的 Azure-SSIS IR 的代理后，你可以部署和运行你的包，以便在数据工厂管道中作为执行 SSIS 包活动访问本地数据。 若要了解如何操作，请参阅[在数据工厂管道中运行 ssis 包作为执行 Ssis 包活动](https://docs.microsoft.com/azure/data-factory/how-to-invoke-ssis-package-ssis-activity)。
