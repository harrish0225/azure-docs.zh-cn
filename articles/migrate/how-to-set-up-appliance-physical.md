---
title: 为物理服务器设置 Azure Migrate 设备
description: 了解如何设置 Azure Migrate 设备进行物理服务器评估。
ms.service: azure-migrate
ms.topic: article
ms.date: 04/15/2020
ms.openlocfilehash: ddc70ee9430d3a767ce01191824c150a4dbd5e6f
ms.sourcegitcommit: 849bb1729b89d075eed579aa36395bf4d29f3bd9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/28/2020
ms.locfileid: "81538267"
---
# <a name="set-up-an-appliance-for-physical-servers"></a>为物理服务器设置设备

本文介绍了如何设置 Azure Migrate 设备，前提是你要用 Azure Migrate： Server 评估工具评估物理服务器。

Azure Migrate 设备是一种轻型设备，由 Azure Migrate 服务器评估用于执行以下操作：

- 发现本地服务器。
- 将发现的服务器的元数据和性能数据发送到 Azure Migrate Server 评估。

[了解](migrate-appliance.md)Azure Migrate 设备的详细信息。


## <a name="appliance-deployment-steps"></a>设备部署步骤

若要设置该设备，请执行以下操作：
- 从 Azure 门户下载带有 Azure Migrate 安装程序脚本的压缩文件。
- 从压缩文件中提取内容。 使用管理权限启动 PowerShell 控制台。
- 执行 PowerShell 脚本以启动设备 Web 应用程序。
- 完成设备的首次配置，并将其注册到 Azure Migrate 项目。

## <a name="download-the-installer-script"></a>下载安装程序脚本

下载设备的压缩文件。

1. 在“迁移目标” > “服务器” > “Azure Migrate:    服务器评估”中，单击“发现”。 
2. 在“发现计算机” > “计算机是否已虚拟化?”中，单击“未虚拟化/其他”。   
3. 单击“下载”以下载压缩文件  。

    ![下载 VM](./media/tutorial-assess-physical/download-appliance.png)


### <a name="verify-security"></a>验证安全性

在部署压缩文件之前检查其安全性。

1. 在下载文件的计算机上，打开管理员命令窗口。
2. 运行以下命令以生成 zip 文件的哈希：
    - ```C:\>CertUtil -HashFile <file_location> [Hashing Algorithm]```
    - 公有云的示例用法：```C:\>CertUtil -HashFile C:\Users\administrator\Desktop\AzureMigrateInstaller.zip SHA256 ```
    - 政府云的示例用法：```  C:\>CertUtil -HashFile C:\Users\administrator\Desktop\AzureMigrateInstaller-Server-USGov.zip MD5 ```
3.  验证哈希值：
 
    - 对于公有云（适用于最新设备版本）：

        **算法** | **哈希值**
          --- | ---
          MD5 | 1e92ede3e87c03bd148e56a708cdd33f
          SHA256 | a3fa78edc8ff8aff9ab5ae66be1b64e66de7b9f475b6542beef114b20bfdac3c

    - 对于 Azure 政府云（适用于最新设备版本）：

        **算法** | **哈希值**
          --- | ---
          MD5 | f81c155fc4a1409901caea948713913f


## <a name="run-the-azure-migrate-installer-script"></a>运行 Azure Migrate 安装程序脚本
此安装程序脚本执行以下操作：

- 安装用于物理服务器发现和评估的代理和 Web 应用程序。
- 安装 Windows 角色，包括 Windows 激活服务、IIS 和 PowerShell ISE。
- 下载并安装 IIS 可重写模块。 [了解详细信息](https://www.microsoft.com/download/details.aspx?id=7435)。
- 更新 Azure Migrate 的注册表项 (HKLM) 和永久性设置详细信息。
- 在路径下创建以下文件：
    - **配置文件**：%Programdata%\Microsoft Azure\Config
    - **日志文件**：%Programdata%\Microsoft Azure\Logs

按如下所示运行脚本：

1. 将压缩文件解压缩到托管设备的服务器上的某个文件夹中。  请确保不要在现有 Azure Migrate 设备上的计算机中运行该脚本。
2. 使用管理（提升）权限在上述服务器上启动 PowerShell。
3. 将 PowerShell 目录更改为从下载的压缩文件中提取内容的文件夹。
4. 通过运行以下命令，运行名为“AzureMigrateInstaller.ps1”的脚本  ：

    - 对于公有云：``` PS C:\Users\administrator\Desktop\AzureMigrateInstaller> AzureMigrateInstaller.ps1 ```
    - 对于 Azure 政府云：``` PS C:\Users\Administrators\Desktop\AzureMigrateInstaller-Server-USGov>AzureMigrateInstaller.ps1 ```

    脚本将在成功完成时启动设备 Web 应用程序。

如果遇到任何问题，可以访问位于 C:\ProgramData\Microsoft Azure\Logs\AzureMigrateScenarioInstaller_<em>Timestamp</em>.log 的脚本日志来进行故障排除。



### <a name="verify-appliance-access-to-azure"></a>验证设备的 Azure 访问权限

确保设备 VM 可以连接到[公有云](migrate-appliance.md#public-cloud-urls)和[政府云](migrate-appliance.md#government-cloud-urls)的 Azure URL。

## <a name="configure-the-appliance"></a>配置设备

首次设置设备。

1. 在可连接到该 VM 的任一计算机上打开浏览器，然后打开设备 Web 应用的 URL：**https://*设备名称或 IP 地址*:** 44368。

   或者，可以在桌面上单击应用快捷方式打开该应用。
2. 在 Web 应用 >“设置必备组件”中执行以下操作： 
    - **许可证**：接受许可条款，并阅读第三方信息。
    - **连接**：应用将检查 VM 是否可访问 Internet。 如果 VM 使用代理：
        - 单击“代理设置”，并以 http://ProxyIPAddress 或 http://ProxyFQDN 格式指定代理地址和侦听端口。 
        - 如果代理需要身份验证，请指定凭据。
        - 仅支持 HTTP 代理。
    - **时间同步**：将验证时间。 设备上的时间应与 Internet 时间同步，这样才能正常发现 VM。
    - **安装更新**：Azure Migrate 服务器评估将检查设备上是否安装了最新更新。

### <a name="register-the-appliance-with-azure-migrate"></a>将设备注册到 Azure Migrate

1. 单击“登录”。  如果未显示该按钮，请确保已在浏览器中禁用弹出窗口阻止程序。
2. 在新的标签页中，使用 Azure 凭据登录。
    - 使用用户名和密码登录。
    - 不支持使用 PIN 登录。
3. 成功登录后，返回到 Web 应用。
4. 选择在其中创建了 Azure Migrate 项目的订阅。 然后选择该项目。
5. 指定设备的名称。 该名称应是字母数字，长度为 14 个或更少的字符。
6. 单击“注册”  。


## <a name="start-continuous-discovery"></a>启动持续发现

从设备连接到物理服务器，然后启动发现。

1. 单击“添加凭据”以指定设备用于发现服务器的帐户凭据  。  
2. 指定操作系统  、凭据的友好名称以及用户名和密码。 然后单击“添加”  。
你可以为 Windows 和 Linux 服务器分别添加一组凭据。
4. 单击“添加服务器”，然后指定服务器详细信息 - FQDN/IP 地址和凭据的友好名称（每行一个条目）以连接到服务器  。
3. 单击 **“验证”** 。 验证后，将显示可发现的服务器列表。
    - 如果服务器验证失败，请将鼠标悬停在“状态”列中的图标上以查看错误。  解决问题并再次验证。
    - 若要删除服务器，请选择 >“删除”  。
4. 验证之后，单击“保存并启动发现”以启动发现过程。 

随即会启动发现。 大约 15 分钟后，已发现的 VM 的元数据将显示在 Azure 门户中。

## <a name="verify-servers-in-the-portal"></a>验证门户中的服务器

发现完成后，可以验证服务器是否显示在门户中。

1. 打开 Azure Migrate 仪表板。
2. 在“Azure Migrate - 服务器” > “Azure Migrate:   服务器评估”页中，单击显示了**已发现服务器**计数的图标。


## <a name="next-steps"></a>后续步骤

试用 Azure Migrate 服务器评估[的物理服务器评估](tutorial-assess-physical.md)。
