---
title: 排查备份文件和文件夹时速度缓慢的问题
description: 提供了故障排除指导，帮助你诊断 Azure 备份性能问题的原因
ms.reviewer: saurse
ms.topic: troubleshooting
ms.date: 07/05/2019
ms.openlocfilehash: 5e669a68794a8622bb4a2fa55b206153717fd772
ms.sourcegitcommit: 849bb1729b89d075eed579aa36395bf4d29f3bd9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/28/2020
ms.locfileid: "82187896"
---
# <a name="troubleshoot-slow-backup-of-files-and-folders-in-azure-backup"></a>排查在 Azure 备份中备份文件和文件夹时速度缓慢的问题

本文提供故障排除指导，帮助你诊断使用 Azure 备份来备份文件和文件夹时备份性能缓慢的原因。 使用 Azure 备份代理备份文件时，备份过程花费的时间可能比预期要长。 这种延迟可能由以下一个或多个原因所造成：

* [正在备份的计算机上存在性能瓶颈。](#cause1)
* [其他进程或防病毒软件正在干扰 Azure 备份进程。](#cause2)
* [备份代理在 Azure 虚拟机 (VM) 上运行。](#cause3)  
* [正在备份大量的（数百万个）文件。](#cause4)

在开始排查问题之前，建议下载并安装 [最新的 Azure 备份代理](https://aka.ms/azurebackup_agent)。 我们经常更新备份代理，以修复各种问题、添加功能和改善性能。

此外，我们强烈建议查看 [Azure Backup service FAQ](backup-azure-backup-faq.md)（Azure 备份服务常见问题），确保所遇到的问题并非任何常见配置问题。

[!INCLUDE [support-disclaimer](../../includes/support-disclaimer.md)]

## <a name="cause-backup-job-running-in-unoptimized-mode"></a>原因：备份作业在未优化模式下运行

* MARS 代理可以使用 USN （更新序列号）更改日志或未**优化模式**在**优化模式下**运行备份作业，方法是通过扫描整个卷来检查目录或文件的更改。
* 未优化模式的速度较慢，因为代理必须扫描卷上的每个文件，并与元数据进行比较以确定更改的文件。
* 若要验证这一点，请从 MARS 代理控制台打开**作业详细信息**，并检查状态以查看它是否显示 **"传输数据" （未优化，可能需要更长时间）** ，如下所示：

    ![在未优化模式下运行](./media/backup-azure-troubleshoot-slow-backup-performance-issue/unoptimized-mode.png)

* 以下情况可能会导致备份作业以未优化模式运行：
  * 第一次备份（也称为初始复制）将始终以未优化模式运行
  * 如果上一个备份作业失败，则下一个计划的备份作业将以未优化的方式运行。

<a id="cause1"></a>

## <a name="cause-performance-bottlenecks-on-the-computer"></a>原因：计算机存在性能瓶颈

正在备份的计算机上可能有一些瓶颈导致延迟。 例如，计算机读取或写入到磁盘的能力、用于通过网络发送数据的带宽，都可能会造成瓶颈。

Windows 提供了名为[性能监视器](https://techcommunity.microsoft.com/t5/ask-the-performance-team/windows-performance-monitor-overview/ba-p/375481)（Perfmon）的内置工具来检测这些瓶颈。

下面列出的一些性能计数器和范围可帮助你诊断瓶颈，并获得最佳备份性能。

| 计数器 | 状态 |
| --- | --- |
| 逻辑磁盘(物理磁盘)--空闲率 |* 100% 空闲到50% 空闲 = 正常</br>* 49% 空闲率到20% 空闲率 = 警告或监视</br>* 19% 空闲率到0% 空闲 = 关键或超出规范 |
| 逻辑磁盘（物理磁盘）--平均磁盘秒读取或写入百分比 |* 0.001 毫秒到0.015 毫秒 = 正常</br>* 0.015 ms 到0.025 毫秒 = 警告或监视</br>* 0.026 ms 或更长时间 = 关键或超出规范 |
| 逻辑磁盘(物理磁盘)--当前磁盘队列长度(适用于所有实例) |80 个请求超过 6 分钟 |
| 内存--池未分页字节 |* 小于60% 的池已消耗 = 正常<br>* 61% 80 到使用的池数 = 警告或监视</br>* 大于80% 的池 = 关键或超出规范 |
| 内存--池分页字节 |* 小于60% 的池已消耗 = 正常</br>* 61% 80 到使用的池数 = 警告或监视</br>* 大于80% 的池 = 关键或超出规范 |
| 内存--可用 MB |* 50% 的可用内存，或更多 = 正常</br>* 25% 的可用内存 = 监视器</br>* 10% 的可用内存 = 警告</br>* 小于 100 MB 或5% 的可用内存 = 关键或超出规范 |
| 处理器--\%处理器时间百分比(所有实例) |* 低于 60% = 正常</br>* 61%-已消耗 90% = 监视器或警告</br>* 91%-已消耗 100% = 关键 |

> [!NOTE]
> 如果判定罪魁祸首是基础结构，建议定期进行磁盘碎片整理以提高性能。
>
>

<a id="cause2"></a>

## <a name="cause-another-process-or-antivirus-software-interfering-with-azure-backup"></a>原因：其他进程或防病毒软件正在干扰 Azure 备份

在许多场合中，我们发现 Windows 系统中的其他进程对 Azure 备份代理进程的性能造成负面影响。 例如，如果同时使用 Azure 备份代理和其他程序来备份数据，或者防病毒软件正在运行，因而锁定了要备份的文件，则文件中的多个锁可能会造成资源争用。 在此情况下，备份可能失败，或者作业花费的时间可能长于预期。

在这种情况下，建议的最佳做法是关闭其他备份程序，并观察 Azure 备份代理的备份时间是否有所变化。 一般情况下，确保多个备份作业不在同一时间运行，就足以防止作业彼此干扰。

至于防病毒程序，我们建议排除以下文件和位置：

* C:\Program Files\Microsoft Azure Recovery Services Agent\bin\cbengine.exe（进程）
* C:\Program Files\Microsoft Azure Recovery Services Agent\（文件夹）
* 暂存位置（如果未使用标准位置）

<a id="cause3"></a>

## <a name="cause-backup-agent-running-on-an-azure-virtual-machine"></a>原因：备份代理在 Azure 虚拟机上运行

如果在 VM 上运行备份代理，其性能比在物理机上运行要慢。 这是预期行为，因为存在 IOPS 限制。  但是，可以通过将正在备份的数据驱动器切换到 Azure 高级存储来优化性能。 我们正在努力解决此问题，将来的版本将有这方面的修复。

<a id="cause4"></a>

## <a name="cause-backing-up-a-large-number-millions-of-files"></a>原因：正在备份大量的（数百万个）文件

移动大量数据所花费的时间比移动少量数据要长。 但在某些情况下，备份时间不仅与数据大小相关，也与文件或文件夹数目相关。 在备份几百万个小型文件（几个字节到几 KB）时更是如此。

出现此行为是因为，尽管我们是在备份数据并将它转到 Azure，但我们同时也在为文件创建目录。 在某些罕见的情况下，目录操作花费的时间可能超过预期。

以下迹象可帮助你了解瓶颈，并相应地采取后续步骤：

* **UI 显示数据传输进度**。 数据仍在传输。 网络带宽或数据大小可能导致延迟。
* **UI 未显示数据传输进度**。 打开位于 C:\Program Files\Microsoft Azure Recovery Services Agent\Temp 中的日志，并检查日志中的 FileProvider::EndData 条目。 此条目表示数据传输已完成，正在进行目录操作。 请不要取消备份作业， 而是再等待片刻，让目录操作完成。 如果问题持续出现，请联系 [Azure 支持部门](https://portal.azure.com/#create/Microsoft.Support)。

## <a name="next-steps"></a>后续步骤

* [与对文件和文件夹进行备份相关的常见问题](backup-azure-file-folder-backup-faq.md)
