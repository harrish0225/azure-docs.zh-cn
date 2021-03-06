---
title: 备份 Azure VM 中的 SQL Server 数据库
description: 本文介绍如何使用 Azure 备份在 Azure 虚拟机上备份 SQL Server 数据库。
ms.reviewer: vijayts
ms.topic: conceptual
ms.date: 09/11/2019
ms.openlocfilehash: 887f15deed74330cf132e0574d166c074d2c7cad
ms.sourcegitcommit: 849bb1729b89d075eed579aa36395bf4d29f3bd9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/28/2020
ms.locfileid: "81685718"
---
# <a name="back-up-sql-server-databases-in-azure-vms"></a>备份 Azure VM 中的 SQL Server 数据库

SQL Server 数据库属于关键工作负荷，要求较低的恢复点目标 (RPO) 和长期保留。 可以使用 [Azure 备份](backup-overview.md)来备份 Azure 虚拟机 (VM) 上运行的 SQL Server 数据库。

本文介绍如何将 Azure VM 上运行的 SQL Server 数据库备份到 Azure 备份恢复服务保管库。

本文介绍如何执行以下操作：

> [!div class="checklist"]
>
> * 创建并配置保管库。
> * 发现数据库并设置备份。
> * 为数据库设置自动保护。

>[!NOTE]
>Azure vm**中的 SQL Server 软删除和 AZURE vm 工作负荷中 SAP HANA 的软删除**现已在预览版中提供。<br>
>若要注册预览版，请向我们写信AskAzureBackupTeam@microsoft.com

## <a name="prerequisites"></a>必备条件

在备份 SQL Server 数据库之前，请检查以下条件：

1. 在与托管 SQL Server 实例的 VM 相同的区域和订阅中标识或创建[恢复服务保管库](backup-sql-server-database-azure-vms.md#create-a-recovery-services-vault)。
2. 验证 VM 是否具有[网络连接](backup-sql-server-database-azure-vms.md#establish-network-connectivity)。
3. 确保 SQL Server 数据库遵循 [Azure 备份的数据库命名准则](#database-naming-guidelines-for-azure-backup)。
4. 检查是否未为该数据库启用了其他任何备份解决方案。 在备份数据库之前，请禁用其他所有 SQL Server 备份。

> [!NOTE]
> 可以同时针对某个 Azure VM 以及该 VM 上运行的 SQL Server 数据库启用 Azure 备份，这不会发生冲突。

### <a name="establish-network-connectivity"></a>建立网络连接

对于所有操作，SQL Server VM 都需要连接到 Azure 公共 IP 地址。 如果未连接到 Azure 公共 IP 地址，VM 操作（数据库发现、配置备份、计划备份、还原恢复点等）将失败。

使用以下选项之一建立连接：

#### <a name="allow-the-azure-datacenter-ip-ranges"></a>允许 Azure 数据中心 IP 范围

此选项允许已下载文件中的 [IP 范围](https://www.microsoft.com/download/details.aspx?id=41653)。 若要访问网络安全组 (NSG)，请使用 Set-AzureNetworkSecurityRule cmdlet。 如果安全收件人列表仅包含特定于区域的 IP，则还需更新 Azure Active Directory (Azure AD) 服务标记的安全收件人列表以启用身份验证。

#### <a name="allow-access-using-nsg-tags"></a>允许使用 NSG 标记进行访问

如果使用 NSG 来限制连接，则应使用 AzureBackup 服务标记以允许对 Azure 备份进行出站访问。 此外，还应允许使用 Azure AD 和 Azure 存储的[规则](https://docs.microsoft.com/azure/virtual-network/security-overview#service-tags)，在连接后进行身份验证和数据传输。 这可以通过 Azure 门户或 PowerShell 来完成。

若要使用门户创建规则，请执行以下操作：

  1. 在“所有服务”**** 中转到“网络安全组”****，然后选择“网络安全组”。
  2. 在“设置”下，选择“出站安全规则”。********
  3. 选择 **添加** 。 输入创建新规则所需的所有详细信息，如[安全规则设置](https://docs.microsoft.com/azure/virtual-network/manage-network-security-group#security-rule-settings)中所述。 请确保将选项“目标”**** 设置为“服务标记”****，将“目标服务标记”**** 设置为 **AzureBackup**。
  4. 单击“添加”****，保存新创建的出站安全规则。

若要使用 PowerShell 创建规则，请执行以下操作：

 1. 添加 Azure 帐户凭据并更新国家/地区云<br/>
      `Add-AzureRmAccount`<br/>

 2. 选择 NSG 订阅<br/>
      `Select-AzureRmSubscription "<Subscription Id>"`

 3. 选择 NSG<br/>
    `$nsg = Get-AzureRmNetworkSecurityGroup -Name "<NSG name>" -ResourceGroupName "<NSG resource group name>"`

 4. 为 Azure 备份服务标记添加允许出站规则<br/>
    `Add-AzureRmNetworkSecurityRuleConfig -NetworkSecurityGroup $nsg -Name "AzureBackupAllowOutbound" -Access Allow -Protocol * -Direction Outbound -Priority <priority> -SourceAddressPrefix * -SourcePortRange * -DestinationAddressPrefix "AzureBackup" -DestinationPortRange 443 -Description "Allow outbound traffic to Azure Backup service"`

 5. 为 Azure 存储服务标记添加允许出站规则<br/>
    `Add-AzureRmNetworkSecurityRuleConfig -NetworkSecurityGroup $nsg -Name "StorageAllowOutbound" -Access Allow -Protocol * -Direction Outbound -Priority <priority> -SourceAddressPrefix * -SourcePortRange * -DestinationAddressPrefix "Storage" -DestinationPortRange 443 -Description "Allow outbound traffic to Azure Backup service"`

 6. 为 AzureActiveDirectory 服务标记添加允许出站规则<br/>
    `Add-AzureRmNetworkSecurityRuleConfig -NetworkSecurityGroup $nsg -Name "AzureActiveDirectoryAllowOutbound" -Access Allow -Protocol * -Direction Outbound -Priority <priority> -SourceAddressPrefix * -SourcePortRange * -DestinationAddressPrefix "AzureActiveDirectory" -DestinationPortRange 443 -Description "Allow outbound traffic to AzureActiveDirectory service"`

 7. 保存 NSG<br/>
    `Set-AzureRmNetworkSecurityGroup -NetworkSecurityGroup $nsg`

**允许使用 Azure 防火墙标记进行访问**。 如果使用 Azure 防火墙，请使用 AzureBackup [FQDN 标记](https://docs.microsoft.com/azure/firewall/fqdn-tags)创建应用程序规则。 这允许对 Azure 备份进行出站访问。

**部署用于路由流量的 HTTP 代理服务器**。 在 Azure VM 中备份 SQL Server 数据库时，该 VM 上的备份扩展将使用 HTTPS API 将管理命令发送到 Azure 备份，并将数据发送到 Azure 存储。 备份扩展还使用 Azure AD 进行身份验证。 通过 HTTP 代理路由这三个服务的备份扩展流量。 Azure 备份使用的通配符域不能添加到代理规则的允许列表。 你将需要使用 Azure 提供的这些服务的公共 IP 范围。 该扩展是为了访问公共 Internet 而配置的唯一组件。

连接性选项包括以下优点和缺点：

**选项** | **优点** | **缺点**
--- | --- | ---
允许 IP 范围 | 无额外成本 | 管理起来很复杂，因为 IP 地址范围随时会变化 <br/><br/> 允许访问整个 Azure，而不只是 Azure 存储
使用 NSG 服务标记 | 由于范围更改会自动合并，因此更易于管理 <br/><br/> 无额外成本 <br/><br/> | 仅可与 NSG 配合使用 <br/><br/> 提供对整个服务的访问权限
使用 Azure 防火墙 FQDN 标记 | 自动管理必需的 FQDN，因此更易于管理 | 仅可与 Azure 防火墙配合使用
使用 HTTP 代理 | 对 VM 进行单点 Internet 访问 <br/> | 通过代理软件运行 VM 带来的额外成本 <br/> 无已发布的 FQDN 地址，允许规则将受到 Azure IP 地址更改的限制

#### <a name="private-endpoints"></a>专用终结点

[!INCLUDE [Private Endpoints](../../includes/backup-private-endpoints.md)]

### <a name="database-naming-guidelines-for-azure-backup"></a>Azure 备份的数据库命名准则

避免在数据库名称中使用以下元素：

* 尾部和前导空格
* 尾部感叹号 (!)
* 右方括号 (])
* 分号“;”
* 正斜杠“/”

可对不支持的字符使用别名，但我们建议避免这样做。 有关详细信息，请参阅 [Understanding the Table Service Data Model](https://docs.microsoft.com/rest/api/storageservices/Understanding-the-Table-Service-Data-Model)（了解表服务数据模型）。

>[!NOTE]
>不支持为名称中包含特殊字符（如 "+" 或 "&"）的数据库**配置保护**操作。 可以更改数据库名称，也可以启用**自动保护**，这样可以成功保护这些数据库。

[!INCLUDE [How to create a Recovery Services vault](../../includes/backup-create-rs-vault.md)]

## <a name="discover-sql-server-databases"></a>发现 SQL Server 数据库

如何发现 VM 上运行的数据库：

1. 在 [Azure 门户](https://portal.azure.com)中，打开用于备份数据库的恢复服务保管库。

2. 在“恢复服务保管库”仪表板中选择“备份”。********

   ![选择“+备份”打开“备份目标”菜单](./media/backup-azure-sql-database/open-backup-menu.png)

3. 在“备份目标”中，将“工作负荷的运行位置”设置为“Azure”。************

4. 在“要备份哪些内容”中，选择“Azure VM 中的 SQL Server”。********

    ![选择“Azure VM 中的 SQL Server”进行备份](./media/backup-azure-sql-database/choose-sql-database-backup-goal.png)

5. 在 "**备份目标** > " 中**发现虚拟机中**的数据库，选择 "**启动发现**" 搜索订阅中未受保护的 vm。 此搜索过程需要花费一段时间，具体取决于订阅中不受保护的 VM 数量。

   * 发现后，未受保护的 VM 应会按名称和资源组列在列表中。
   * 如果某个 VM 未按预期列出，请查看它是否已保管库中备份。
   * 可能有多个 VM 同名，但它们属于不同的资源组。

     ![在搜索 VM 中的数据库期间，备份将会挂起。](./media/backup-azure-sql-database/discovering-sql-databases.png)

6. 在 VM 列表中，选择运行 SQL Server 数据库的VM，然后选择“发现数据库”。****

7. 在“通知”跟踪数据库发现。**** 完成此操作所需的时间取决于 VM 数据库的数量。 发现选定的数据库后，会显示一条成功消息。

    ![部署成功消息](./media/backup-azure-sql-database/notifications-db-discovered.png)

8. Azure 备份将发现该 VM 上的所有 SQL Server 数据库。 在发现期间，后台将发生以下情况：

    * Azure 备份将 VM 注册到用于备份工作负荷的保管库。 已注册 VM 上的所有数据库只能备份到此保管库。
    * Azure 备份在 VM 上安装 AzureBackupWindowsWorkload 扩展。 不会在 SQL 数据库中安装任何代理。
    * Azure 备份在 VM 上创建服务帐户 NT Service\AzureWLBackupPluginSvc。
      * 所有备份和还原操作使用该服务帐户。
      * NT Service\AzureWLBackupPluginSvc 需要 SQL sysadmin 权限。 在市场中创建的所有 SQL Server VM 已预装 SqlIaaSExtension。 AzureBackupWindowsWorkload 扩展使用 SQLIaaSExtension 自动获取所需的权限。
    * 如果 VM 不是从市场创建的，或者使用的是 SQL 2008 和 2008 R2，则该 VM 上可能未安装 SqlIaaSExtension，并且发现操作将会失败并出现错误消息 UserErrorSQLNoSysAdminMembership。 若要解决此问题，请按照[设置 VM 权限](backup-azure-sql-database.md#set-vm-permissions)中的说明操作。

        ![选择 VM 和数据库](./media/backup-azure-sql-database/registration-errors.png)

## <a name="configure-backup"></a>配置备份  

1. 在 "**备份目标** > "**步骤2：配置备份**中，选择 "**配置备份**"。

   ![选择“配置备份”](./media/backup-azure-sql-database/backup-goal-configure-backup.png)

2. 在“选择要备份的项”中，可以看到所有已注册的可用性组和独立的 SQL Server 实例。**** 选择行左侧的箭头，展开该实例或 Always On 可用性组中所有不受保护的数据库列表。  

    ![显示包含独立数据库的所有 SQL Server 实例](./media/backup-azure-sql-database/list-of-sql-databases.png)

3. 选择要保护的所有数据库，然后选择“确定”。****

   ![保护数据库](./media/backup-azure-sql-database/select-database-to-protect.png)

   为了优化备份负载，Azure 备份会将一个备份作业中的最大数据库数目设置为 50。

     * 若要对 50 个以上的数据库提供保护，请配置多个备份。
     * 若要[启用](#enable-auto-protection)整个实例或 Always On 可用性组，请在“自动保护”下拉列表中选择“打开”，然后选择“确定”。************

    > [!NOTE]
    > [自动保护](#enable-auto-protection)功能不仅可以一次性针对所有现有数据库启用保护，而且还会自动保护添加到该实例或可用性组的所有新数据库。  

4. 选择“确定”打开“备份策略”。********

    ![针对 Always On 可用性组启用自动保护](./media/backup-azure-sql-database/enable-auto-protection.png)

5. 在“备份策略”中选择一个策略，然后选择“确定”。********

   * 选择“HourlyLogBackup”作为默认策略。
   * 选择前面为 SQL 创建的现有备份策略。
   * 根据 RPO 和保留范围定义新策略。

     ![选择“备份策略”](./media/backup-azure-sql-database/select-backup-policy.png)

6. 在“备份”中，选择“启用备份”。********

    ![启用选定的备份策略](./media/backup-azure-sql-database/enable-backup-button.png)

7. 在门户的“通知”区域跟踪配置进度。****

    ![通知区域](./media/backup-azure-sql-database/notifications-area.png)

### <a name="create-a-backup-policy"></a>创建备份策略

备份策略定义备份创建时间以及这些备份的保留时间。

* 策略是在保管库级别创建的。
* 多个保管库可以使用相同的备份策略，但必须向每个保管库应用该备份策略。
* 创建备份策略时，每日完整备份是默认设置。
* 可以添加差异备份，但仅在将完整备份配置为每周发生时，才能这样做。
* 了解[不同类型的备份策略](backup-architecture.md#sql-server-backup-types)。

创建备份策略：

1. 在保管库中，选择 "**备份策略** > " "**添加**"。
2. 在“添加”中，选择“Azure VM 中的 SQL Server”以定义策略类型。********

   ![为新的备份策略选择策略类型](./media/backup-azure-sql-database/policy-type-details.png)

3. 在“策略名称”处输入新策略的名称****。
4. 在“完整备份策略”中选择“备份频率”********。 选择“每日”或“每周”。********

   * 如果选择“每日”，请选择备份作业开始时的小时和时区。****
   * 如果选择“每周”，请选择备份作业开始时的星期、小时和时区。****
   * 运行完整备份，因为无法禁用“完整备份”选项。****
   * 选择“完整备份”以查看策略。****
   * 对于每日完整备份，无法创建差异备份。

     ![新备份策略字段](./media/backup-azure-sql-database/full-backup-policy.png)  

5. 对于“保留范围”，默认已选择所有选项。**** 清除任何不需要的保留范围限制，然后设置要使用的间隔。

    * 任何类型的备份（完整、差异和日志）的最短保留期均为七天。
    * 恢复点已根据其保留范围标记为保留。 例如，如果选择每日完整备份，则每天只触发一次完整备份。
    * 根据每周保留范围和每周保留设置，将会标记并保留特定日期的备份。
    * 每月和每年保留范围的行为类似。

       ![保留范围间隔设置](./media/backup-azure-sql-database/retention-range-interval.png)

6. 在“完整备份策略”菜单中，选择“确定”接受设置。********
7. 若要添加差异备份策略，请选择“差异备份”。****

   ![保留范围间隔设置](./media/backup-azure-sql-database/retention-range-interval.png)
   ![打开差异备份策略菜单](./media/backup-azure-sql-database/backup-policy-menu-choices.png)

8. 在“差异备份策略”中，选择“启用”打开频率和保留控件。********

    * 每天只能触发一次差异备份。
    * 差异备份最多可以保留 180 天。 如需更长的保留期，请使用完整备份。

9. 选择“确定”保存策略，并返回“备份策略”主菜单。********

10. 若要添加事务日志备份策略，请选择“日志备份”。****
11. 在“日志备份”中选择“启用”，并设置频率和保留控件。******** 日志备份最多可以每隔 15 分钟发生一次，最多可以保留 35 天。
12. 选择“确定”保存策略，并返回“备份策略”主菜单。********

    ![编辑日志备份策略](./media/backup-azure-sql-database/log-backup-policy-editor.png)

13. 在“备份策略”菜单中，选择是否启用“SQL 备份压缩”********。 默认情况下禁用此选项。 如果启用，SQL Server 会向 VDI 发送压缩的备份流。  请注意，Azure 备份将根据此控件的值，使用 COMPRESSION/NO_COMPRESSION 子句替代实例级别的默认值。

14. 完成备份策略的编辑后，选择“确定”。****

> [!NOTE]
> 每个日志备份都链接到上一个完整备份，以形成恢复链。 此完整备份将一直保留到最后一个日志备份的保留期结束为止。 这可能意味着完整备份会多保留一段时间，以确保所有日志都可以恢复。 假设用户具有每周完整备份、每日差异和2小时的日志。 这些备份都保留 30 天。 但是，只有在下一个完整备份可用后（即 30 + 7 天后），才能真正清除/删除这个每周完整备份。 假设每周完整备份发生在 11 月 16 日。 根据保留策略，它应保留到 12 月 16 日。 该完整备份的最后一次日志备份发生在下一次计划的完整备份之前，即 11 月 22 日。 必须等到 12 月 22 日此日志备份可用后，才能删除 11 月 16 日的完整备份。 因此，11 月 16 日的完整备份会保留到 12 月 22 日。

## <a name="enable-auto-protection"></a>启用自动保护  

可以启用自动保护，以便自动将所有现有数据库和将来的数据库备份到独立 SQL Server 实例或 Always On 可用性组。

* 可以一次性选择进行自动保护的数据库数目没有限制。
* 启用自动保护时，无法选择在实例中保护数据库，或者在实例中排除对其的保护。
* 如果实例已包含某些受保护的数据库，即使在启用自动保护后，它们也仍会根据相应的策略受到保护。 以后添加的所有不受保护的数据库只会应用在启用自动保护时定义的、在“配置备份”下列出的单个策略。**** 但是，以后可以更改与自动保护的数据库关联的策略。  

若要启用自动保护：

  1. 在“要备份的项”中，选择要为其启用自动保护的实例。****
  2. 选择“自动保护”下的下拉列表，选择“打开”，然后选择“确定”。************

      ![针对可用性组启用自动保护](./media/backup-azure-sql-database/enable-auto-protection.png)

  3. 备份是针对所有数据库统一配置的，可以在“备份作业”中跟踪备份。****

如果需要禁用自动保护，请选择“配置备份”下的实例名称，然后针对该实例选择“禁用自动保护”。******** 所有数据库将继续备份，但将来的数据库不会自动受到保护。

![针对该实例禁用自动保护](./media/backup-azure-sql-database/disable-auto-protection.png)

## <a name="next-steps"></a>后续步骤

了解如何操作：

* [还原已备份的 SQL Server 数据库](restore-sql-database-azure-vm.md)
* [管理已备份的 SQL Server 数据库](manage-monitor-sql-database-backup.md)
