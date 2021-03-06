---
title: 在 Visual Studio 中删除 Application Insights-Azure Monitor
description: 如何在 Visual Studio 中删除 ASP.NET 和 ASP.NET Core Application Insights SDK。
ms.topic: conceptual
ms.date: 04/06/2020
ms.openlocfilehash: 1c9ff8d3d305645ac7d113421e2c6c5f8451bd2b
ms.sourcegitcommit: 849bb1729b89d075eed579aa36395bf4d29f3bd9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/28/2020
ms.locfileid: "80805100"
---
# <a name="how-to-remove-application-insights-in-visual-studio"></a>如何在 Visual Studio 中删除 Application Insights

本文介绍如何在 Visual Studio 中删除 ASP.NET 和 ASP.NET Core Application Insights SDK。

若要删除 Application Insights，需要从应用程序中的 API 删除 NuGet 包和引用。 你可以使用包管理控制台卸载 NuGet 包，或在 Visual Studio 中管理 NuGet 解决方案。 以下各节将介绍两种删除 NuGet 包的方法以及项目中自动添加的内容。 请确保已在您自己的代码中删除已添加的文件，以及您在其中创建了对 API 的调用的区域。

## <a name="uninstall-using-the-package-management-console"></a>使用包管理控制台卸载

# <a name="net"></a>[.NET](#tab/net)

1. 若要打开包管理控制台，请在顶部菜单中选择 "工具" > "NuGet 包管理器 > 程序包管理器控制台"。
     
    ![在顶部菜单中，单击 "工具" > NuGet 包管理器 > 程序包管理器控制台](./media/remove-application-insights/package-manager.png)

    > [!NOTE]
    > 如果启用了跟踪收集，则需要先卸载 Applicationinsights.config。 TraceListener。 输入`Uninstall-package Microsoft.ApplicationInsights.TraceListener` ，然后执行以下步骤以删除 applicationinsights.config。

1. 输入以下命令： `Uninstall-Package Microsoft.ApplicationInsights.Web -RemoveDependencies`

    输入该命令后，将从项目中卸载 Application Insights 包及其所有依赖项。
    
    ![在控制台中输入命令](./media/remove-application-insights/package-management-console.png)

# <a name="net-core"></a>[.NET Core](#tab/netcore)

1. 若要打开包管理控制台，请在顶部菜单中选择 "工具" > "NuGet 包管理器 > 程序包管理器控制台"。

    ![在顶部菜单中，单击 "工具" > NuGet 包管理器 > 程序包管理器控制台](./media/remove-application-insights/package-manager.png)

1. 输入以下命令： ` Uninstall-Package Microsoft.ApplicationInsights.AspNetCore -RemoveDependencies`

    输入该命令后，将从项目中卸载 Application Insights 包及其所有依赖项。

---

## <a name="uninstall-using-the-visual-studio-nugetui"></a>使用 Visual Studio NuGet UI 卸载

# <a name="net"></a>[.NET](#tab/net)

1. 在右侧的 " *解决方案资源管理器* 中，右键单击" **解决方案** "，然后选择" **管理解决方案的 NuGet 包**"。

    然后，你将看到一个屏幕，该屏幕允许你编辑属于项目的所有 NuGet 包。
    
     ![右键单击 "解决方案"，然后在 "解决方案资源管理器" 中，选择 "管理解决方案的 NuGet 包"](./media/remove-application-insights/manage-nuget-framework.png)

    > [!NOTE]
    > 如果启用了跟踪收集，则需要先卸载 Applicationinsights.config，并选择 "删除依赖项"，然后按照以下步骤卸载 Applicationinsights.config，并选中 "删除依赖项"。
    
1. 单击 "Applicationinsights.config" 包。在右侧选中 " *项目* " 旁边的复选框以选择所有项目。
    
1. 若要在卸载时删除所有依赖项，请在选择了 "项目" 的部分下选择 " **选项** " 下拉按钮。

    在 " *卸载选项*" 下，选中 " *删除依赖项*" 旁边的复选框。

1. 选择“卸载”  。
    
    ![选中 "删除依赖项"，然后选择 "卸载"](./media/remove-application-insights/uninstall-framework.png)

    将显示一个对话框，显示要从应用程序中删除的所有依赖项。选择 **"确定"** 以卸载。
    
    ![选中 "删除依赖项"，然后选择 "卸载"](./media/remove-application-insights/preview-uninstall-framework.png)
    
1.  卸载所有内容后，你可能仍会在 *解决方案资源管理器*中看到 "applicationinsights.config" 和 "AiHandleErrorAttribute.cs"。您可以手动删除这两个文件。

# <a name="net-core"></a>[.NET Core](#tab/netcore)

1. 在右侧的 " *解决方案资源管理器* 中，右键单击" **解决方案** "，然后选择" **管理解决方案的 NuGet 包**"。

    然后，你将看到一个屏幕，该屏幕允许你编辑属于项目的所有 NuGet 包。

    ![右键单击 "解决方案"，然后在 "解决方案资源管理器" 中，选择 "管理解决方案的 NuGet 包"](./media/remove-application-insights/manage-nuget-core.png)

1. 单击 "Applicationinsights.config. AspNetCore" 包。 在右侧选中 "*项目*" 旁边的复选框以选择所有项目，然后选择 "**卸载**"。

    ![选中 "删除依赖项"，然后选择 "卸载"](./media/remove-application-insights/uninstall-core.png)

---

## <a name="what-is-created-when-you-add-application-insights"></a>添加 Application Insights 时创建的内容

将 Application Insights 添加到项目时，它将创建文件并将代码添加到一些文件中。 仅卸载 NuGet 包并不会总是丢弃文件和代码。 若要完全删除 Application Insights，你应该检查并手动删除添加的代码或文件以及你在项目中添加的任何 API 调用。

# <a name="net"></a>[.NET](#tab/net)

将 Application Insights 遥测添加到 Visual Studio ASP.NET 项目时，它将添加以下文件：

- ApplicationInsights.config
- AiHandleErrorAttribute.cs

添加了以下代码片段：

- [项目的名称] .csproj

    ```C#
     <ApplicationInsightsResourceId>/subscriptions/00000000-0000-0000-0000-000000000000/resourcegroups/Default-ApplicationInsights-EastUS/providers/microsoft.insights/components/WebApplication4</ApplicationInsightsResourceId>
    ```

- 包 .config

    ```xml
    <packages>
    ...
    
      <package id="Microsoft.ApplicationInsights" version="2.12.0" targetFramework="net472" />
      <package id="Microsoft.ApplicationInsights.Agent.Intercept" version="2.4.0" targetFramework="net472" />
      <package id="Microsoft.ApplicationInsights.DependencyCollector" version="2.12.0" targetFramework="net472" />
      <package id="Microsoft.ApplicationInsights.PerfCounterCollector" version="2.12.0" targetFramework="net472" />
      <package id="Microsoft.ApplicationInsights.Web" version="2.12.0" targetFramework="net472" />
      <package id="Microsoft.ApplicationInsights.WindowsServer" version="2.12.0" targetFramework="net472" />
      <package id="Microsoft.ApplicationInsights.WindowsServer.TelemetryChannel" version="2.12.0" targetFramework="net472" />
    
      <package id="Microsoft.AspNet.TelemetryCorrelation" version="1.0.7" targetFramework="net472" />
    
      <package id="System.Buffers" version="4.4.0" targetFramework="net472" />
      <package id="System.Diagnostics.DiagnosticSource" version="4.6.0" targetFramework="net472" />
      <package id="System.Memory" version="4.5.3" targetFramework="net472" />
      <package id="System.Numerics.Vectors" version="4.4.0" targetFramework="net472" />
      <package id="System.Runtime.CompilerServices.Unsafe" version="4.5.2" targetFramework="net472" />
    ...
    </packages>
    ```

- Layout。 cshtml

    如果你的项目包含一个布局 cshtml 文件，则将添加以下代码。
    
    ```html
    <head>
    ...
        <script type = 'text/javascript' >
            var appInsights=window.appInsights||function(config)
            {
                function r(config){ t[config] = function(){ var i = arguments; t.queue.push(function(){ t[config].apply(t, i)})} }
                var t = { config:config},u=document,e=window,o='script',s=u.createElement(o),i,f;for(s.src=config.url||'//az416426.vo.msecnd.net/scripts/a/ai.0.js',u.getElementsByTagName(o)[0].parentNode.appendChild(s),t.cookie=u.cookie,t.queue=[],i=['Event','Exception','Metric','PageView','Trace','Ajax'];i.length;)r('track'+i.pop());return r('setAuthenticatedUserContext'),r('clearAuthenticatedUserContext'),config.disableExceptionTracking||(i='onerror',r('_'+i),f=e[i],e[i]=function(config, r, u, e, o) { var s = f && f(config, r, u, e, o); return s !== !0 && t['_' + i](config, r, u, e, o),s}),t
            }({
                instrumentationKey:'00000000-0000-0000-0000-000000000000'
            });
            
            window.appInsights=appInsights;
            appInsights.trackPageView();
        </script>
    ...
    </head>
    ```

- ConnectedService.json

    ```json
    {
      "ProviderId": "Microsoft.ApplicationInsights.ConnectedService.ConnectedServiceProvider",
      "Version": "16.0.0.0",
      "GettingStartedDocument": {
        "Uri": "https://go.microsoft.com/fwlink/?LinkID=613413"
      }
    }
    ```

- FilterConfig.cs

    ```csharp
            public static void RegisterGlobalFilters(GlobalFilterCollection filters)
            {
                filters.Add(new ErrorHandler.AiHandleErrorAttribute());// This line was added
            }
    ```

# <a name="net-core"></a>[.NET Core](#tab/netcore)

将 Application Insights 遥测添加到 Visual Studio ASP.NET Core 模板项目时，它会添加以下代码：

- [项目的名称] .csproj

    ```csharp
      <PropertyGroup>
        <TargetFramework>netcoreapp3.1</TargetFramework>
        <ApplicationInsightsResourceId>/subscriptions/00000000-0000-0000-0000-000000000000/resourcegroups/Default-ApplicationInsights-EastUS/providers/microsoft.insights/components/WebApplication4core</ApplicationInsightsResourceId>
      </PropertyGroup>
    
      <ItemGroup>
        <PackageReference Include="Microsoft.ApplicationInsights.AspNetCore" Version="2.12.0" />
      </ItemGroup>
    
      <ItemGroup>
        <WCFMetadata Include="Connected Services" />
      </ItemGroup>
    ```

- Appsettings：

    ```json
    "ApplicationInsights": {
        "InstrumentationKey": "00000000-0000-0000-0000-000000000000"
    ```

- ConnectedService.json
    
    ```json
    {
      "ProviderId": "Microsoft.ApplicationInsights.ConnectedService.ConnectedServiceProvider",
      "Version": "16.0.0.0",
      "GettingStartedDocument": {
        "Uri": "https://go.microsoft.com/fwlink/?LinkID=798432"
      }
    }
    ```
- Startup.cs

    ```csharp
       public void ConfigureServices(IServiceCollection services)
            {
                services.AddRazorPages();
                services.AddApplicationInsightsTelemetry(); // This is added
            }
    ```

---

## <a name="next-steps"></a>后续步骤

- [Azure Monitor](https://docs.microsoft.com/azure/azure-monitor/overview)