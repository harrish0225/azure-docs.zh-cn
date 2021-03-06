---
title: 所有 Azure 安全中心警报的引用表
description: 本文列出了 Azure 安全中心的威胁防护模块中可见的安全警报。
services: security-center
documentationcenter: na
author: memildin
manager: rkarlin
ms.service: security-center
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 02/25/2020
ms.author: memildin
ms.openlocfilehash: 9b96547b841c26f12aff2db781ce55c49ef9cde4
ms.sourcegitcommit: e0330ef620103256d39ca1426f09dd5bb39cd075
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/05/2020
ms.locfileid: "82790620"
---
# <a name="security-alerts---a-reference-guide"></a>安全警报-参考指南

本文列出了你可能在 Azure 安全中心的威胁防护模块中看到的安全警报。 环境中显示的警报取决于要保护的资源和服务，以及自定义的配置。

若要了解如何响应这些警报，请参阅[管理和响应 Azure 安全中心的安全警报](security-center-managing-and-responding-alerts.md)。

若要了解如何导出警报（和建议），请参阅[导出安全警报和建议（预览版）](continuous-export.md)。

警报表下面是一个表，描述用于对这些警报的目的进行分类的 Azure 安全中心终止链。 



## <a name="alerts-for-windows-machines"></a><a name="alerts-windows"></a>Windows 计算机的警报

[更多详细信息和说明](threat-protection.md#windows-machines)

|警报|说明|意向（[了解详细信息](#intentions)）|严重性|
|----|----|:----:|--|
|**检测到来自恶意 IP 的登录**|已成功为帐户 "schleining" 和进程 "Advapi" 进行远程身份验证，但登录 IP 地址 [IP 地址] 以前被报告为恶意或非常罕见。 可能会发生成功的攻击。|-|高|
|**检测到来自恶意 IP 的登录。[多次查看]**|已成功为帐户 "IUSR_10001" 和进程 "Advapi" 进行远程身份验证，但登录 IP 地址 [IP 地址] 以前被报告为恶意或非常罕见。 可能会发生成功的攻击。 扩展名为 .scr 的文件是屏幕保护程序文件，通常驻留在 Windows 系统目录中并从其执行。|-|高|
|**向本地管理员组添加来宾帐户**|主机数据分析已检测到% {受害主机} 上的本地管理员组中添加了内置来宾帐户，该帐户与攻击者活动强烈关联。|-|中|
|**已清除事件日志**|计算机日志指出计算机 "% {CompromisedEntity}" 中的用户 "% {user name}" 的可疑事件日志清除操作。 % {Log 通道} 日志已清除。|-|信息|
|**已发现代码注入**|代码注入是将可执行模块插入到正在运行的进程或线程中。 恶意软件使用此方法来访问数据，同时成功地隐藏自身以防止找到和删除。<br>此警报指示故障转储中存在注入模块。 为了区分恶意和非恶意注入的模块，安全中心会检查注入的模块是否符合可疑行为的配置文件。|-|中|
|**检测到的 Petya 勒索软件指标**|分析% {受害主机} 上的主机数据检测到与 Petya 勒索软件相关的指示器。 有关更多信息，请参见https://blogs.technet.microsoft.com/mmpc/2017/06/27/new-ransomware-old-techniques-petya-adds-worm-capabilities/。 查看此警报中关联的命令行，并将此警报升级到安全团队。|-|高|
|**检测到禁用和删除 IIS 日志文件的操作**|分析主机数据检测到的操作，这些操作显示正在禁用和/或删除的 IIS 日志文件。|-|中|
|**检测到命令行中的大小写字符异常混合**|% {受损主机} 上的主机数据分析检测到具有大写和小写字符的异常混合的命令行。 这种模式（尽管可能是良性的）也是攻击者尝试在受损主机上执行管理任务时隐藏区分大小写或基于哈希的规则。|-|中|
|**检测到可以滥用以绕过 UAC 的注册表项更改**|% {受损主机} 上的主机数据分析检测到可滥用 UAC （用户帐户控制）的注册表项已更改。 这种配置（尽管可能是良性的）在尝试从非特权（标准用户）迁移到受损主机上的特权（例如，管理员）访问权限时也是攻击者的典型活动。|-|中|
|**使用内置 certutil 工具检测到可执行文件的解码**|% {已泄露主机} 上的主机数据分析检测到 certutil，内置的管理员实用工具正在用于解码可执行文件，而不是与操作证书和证书数据相关的主流目的。 我们知道，攻击者可以滥用合法的管理员工具的功能来执行恶意操作，例如，使用 certutil.exe 之类的工具来解码恶意的可执行文件，随后执行该文件。|-|高|
|**检测到 WDigest UseLogonCredential 注册表项的启用**|分析主机数据检测到注册表项 HKLM\SYSTEM\CurrentControlSet\Control\SecurityProviders\WDigest\ "UseLogonCredential" 中的更改。 具体而言，此密钥已更新，以允许以明文形式在 LSA 内存中存储登录凭据。 启用后，攻击者可以使用 Mimikatz 等凭据收集工具从 LSA 内存中转储明文密码。|-|中|
|**在命令行数据中检测到已编码的可执行文件**|% {受损主机} 上的主机数据分析检测到64编码的可执行文件。 这以前与尝试通过一系列命令即时构造可执行文件的攻击者相关，尝试通过确保没有个别命令触发警报来规避入侵检测系统。 这可能是合法活动，也可能是损坏的主机的指示。|-|高|
|**检测到模糊的命令行**|攻击者使用越来越复杂的模糊处理技术来避开对基础数据运行的检测。 % {受损主机} 上的主机数据分析检测到命令行上的模糊标记。|-|信息|
|**检测到可能执行 ssh-keygen 可执行文件**|% {受损主机} 上的主机数据分析检测到其名称指示 ssh-keygen 工具的进程的执行;此类工具通常用于击败软件许可机制，但其下载通常与其他恶意软件捆绑在一起。 了解到活动组金牌是为了利用此类 keygens 来获得对其泄露的主机的后门访问权限。|-|中|
|**检测到恶意软件吸管的可能执行**|% {受损主机} 上的主机数据分析检测到以前已与某个活动组黄金方法（在受害者主机上安装恶意软件）关联的文件名。|-|高|
|**检测到可能的本地侦测活动**|% {受损主机} 上的主机数据分析检测到 systeminfo.exe 命令的组合，这些命令先前已与某个活动组黄金的执行侦测活动的方法相关联。  尽管 "systeminfo.exe" 是合法的 Windows 工具，但在此过程中，这种方法的执行时间都很少。|-||
|**检测到 Telegram 工具可能可疑的使用**|主机数据分析显示了 Telegram 的安装，这是适用于移动和桌面系统的免费的基于云的即时消息服务。 众所周知，攻击者滥用此服务将恶意二进制文件传输到任何其他计算机、手机或平板电脑。|-|中|
|**检测到登录时向用户显示的法律公告禁止显示**|% {受损主机} 上的主机数据分析检测到对注册表项进行的更改，这些更改控制在用户登录时是否向用户显示法律公告。 Microsoft 安全分析已确定，这是攻击者在泄露主机之后进行的常见活动。|-|低|
|**检测到 HTA 和 PowerShell 的可疑组合**|攻击者正在使用 mshta （Microsoft HTML 应用程序主机）来启动恶意 PowerShell 命令。  攻击者通常使用带有内嵌 VBScript 的 HTA 文件。  当受害者浏览到 HTA 文件并选择运行它时，将执行它包含的 PowerShell 命令和脚本。 % {受损主机} 上的主机数据分析检测到 mshta 启动 PowerShell 命令。|-|中|
|**检测到可疑的命令行参数**|% {受损主机} 上的主机数据分析检测到可疑的命令行参数，这些参数已与活动组 HYDROGEN 使用的反向 shell 一起使用。|-|高|
|**检测到用于在目录中启动所有可执行文件的可疑命令行**|分析主机数据时检测到在% {受害主机} 上运行的可疑进程。 命令行指示尝试启动可能驻留在目录中的所有可执行文件（* .exe）。 这可能表示有漏洞的主机。|-|中|
|**在命令行中检测到可疑凭据**|% {受损主机} 上的主机数据分析检测到用于按活动组 BORON 执行文件的可疑密码。 已知道此活动组使用此密码在受害者主机上执行 Pirpi 恶意软件。|-|高|
|**检测到可疑文档凭据**|% {受损主机} 上的主机数据分析检测到用于执行文件的恶意软件所使用的可疑公用预计算密码哈希。 已知活动组 HYDROGEN 使用此密码在受害者主机上执行恶意软件。|-|高|
|**检测到可疑的 VBScript 命令执行。**|% {受损主机} 上的主机数据分析检测到执行了 VBScript 命令。 这会将脚本编码为不可读的文本，从而使用户更难检查代码。 Microsoft 威胁研究表明，攻击者通常使用编码 VBscript 文件作为其攻击的一部分，以规避检测系统。 这可能是合法活动，也可能是损坏的主机的指示。|-|中|
|**通过 rundll32.exe 检测到可疑的执行**|分析% {已泄露主机} 上的主机数据检测到 rundll32.exe 正用于执行具有非常见名称的进程，这与在受攻击的主机上安装其第一阶段植入软件时活动组黄金使用的进程命名方案一致。|-|高|
|**检测到可疑的文件清理命令**|分析% {受害主机} 上的主机数据时检测到 systeminfo.exe 命令的组合，这些命令先前已与某个活动组黄金的方法进行关联，这种方法会执行破坏后的自我清除活动。  尽管 "systeminfo.exe" 是合法的 Windows 工具，但连续执行它两次，并在此过程中发生删除命令，但这种情况很少发生。|-|高|
|**检测到可疑文件创建**|% {受损主机} 上的主机数据分析已检测到创建或执行的进程，该进程之前已通过活动组 BARIUM 来指示在受害者主机上执行的泄露后操作。 已知道此活动组使用此方法，在打开仿冒文档中的附件后，将其他恶意软件下载到受攻击的主机。|-|高|
|**检测到可疑的命名管道通信**|% {受损主机} 上的主机数据分析从 Windows 控制台命令写入本地命名管道的数据。 已知命名管道是攻击者使用的通道，并与恶意植入软件通信。 这可能是合法活动，也可能是损坏的主机的指示。|-|高|
|**检测到可疑网络活动**|分析来自% {受害主机} 的网络流量检测到可疑的网络活动。 此类流量尽管可能是良性的，但通常由攻击者用于与恶意服务器通信，以便下载工具、命令和控制和渗透数据。 典型的相关攻击者活动包括将远程管理工具复制到受损主机上，并窃取用户数据。|-|低|
|**检测到可疑的新防火墙规则**|分析主机数据检测到已通过 netsh 添加了新的防火墙规则，以允许来自可疑位置的可执行文件的流量。|-|中|
|**检测到使用 Cacls 来降低系统的安全状态**|攻击者使用多种方法，如暴力破解、鱼叉式的网络钓鱼等。 一旦实现了初始泄露，它们通常会采取措施降低系统的安全设置。 Cacls-更改访问控制列表的简短是 Microsoft Windows native 命令行实用工具，通常用于修改对文件夹和文件的安全权限。 攻击者使用该二进制文件来降低系统的安全设置。 为每个人提供对某些系统二进制文件（如 cluster.exe、net.tcp、wscript.echo 等）的完全访问权限来完成此操作。% {受损主机} 上的主机数据分析检测到可疑的 Cacls 使用以降低系统的安全性。|-|中|
|**检测到 FTP s 交换机的可疑使用**|分析% {受害主机} 的进程创建数据检测到使用了 FTP 的 "-s:filename" 开关。 此开关用于指定要运行的客户端的 FTP 脚本文件。 已知恶意软件或恶意进程使用此 FTP 开关（-s:filename）指向配置为连接到远程 FTP 服务器的脚本文件，并下载其他恶意二进制文件。|-|中|
|**检测到 Pcalua 的可疑使用来启动可执行代码**|% {受损主机} 上的主机数据分析检测到使用 pcalua 启动可执行代码。 Pcalua 是 Microsoft Windows "Program 兼容性助手" 的组件，它检测程序的安装或执行过程中的兼容性问题。 攻击者可能会滥用合法 Windows 系统工具的功能来执行恶意操作，例如，使用 pcalua 和-a 开关在本地或远程共享中启动恶意可执行文件。|-|中|
|**检测到关键服务禁用**|% {受损主机} 上的主机数据分析检测到正在用来停止关键服务（如 Win2k3 或 Windows 安全中心）的 "net.tcp stop" 命令执行。 这两个服务中的任何一个都可能会指示恶意行为。|-|中|
|**检测到数字货币挖掘相关的行为**|% {受损主机} 上的主机数据分析检测到通常与数字货币挖掘关联的进程或命令的执行。|-|高|
|**动态 PS 脚本构造**|% {受损主机} 上的主机数据分析检测到正在动态构造的 PowerShell 脚本。 攻击者有时会使用此方法来逐步构建脚本，以规避 ID 系统。 这可能是合法活动，或者表明某个计算机已经泄露。|-|中|
|**从可疑位置运行的可执行文件**|分析主机数据检测到% {受害主机} 上运行的可执行文件，该文件从常见的已知可疑文件所在的位置运行。 此可执行文件可能是合法活动，也可能是损坏的主机的指示。|-|高|
|**检测到的 Fileless 攻击方法**|指定进程的内存包含 fileless 攻击工具包： [工具包名称]。 Fileless 攻击工具包通常不会在文件系统上存在，这使得传统的防病毒软件难以检测。|DefenseEvasion/执行|高|
|**检测到高风险软件**|分析来自% {受害主机} 的主机数据时检测到过去安装了恶意软件的软件。 在恶意软件分发中使用的一种常用方法是将其打包在其他良性工具中，如此警报中所示。 使用这些工具时，恶意软件可以在后台无提示安装。|-|中|
|**本地管理员组成员已枚举**|计算机日志指示组% {枚举组域名}\%{枚举组名称} 上的成功枚举。 具体而言，% {枚举用户域名}\%{枚举用户名称} 远程枚举了% {枚举组域名}\%{枚举组名称} 组的成员。 此活动可能是合法活动，也可能表示组织中的计算机已遭到破坏并用于侦测% {vmname}。|-|信息|
|**恶意 SQL 活动**|计算机日志指示帐户 "% {user name}" 执行了 "% {process name}"。 此活动被视为恶意活动。|-|高|
|**ZINC server 植入软件创建的恶意防火墙规则 [多次查看]**|使用与已知的执行组件 ZINC 匹配的技术创建防火墙规则。 此规则可能已用于打开% {受害主机} 上的端口，以允许命令 & 控制通信。 今天在下列计算机上出现了 [x] 次此行为： [计算机名称]|-|高|
|**检测到伪装 Windows 模块**|故障转储分析检测到第三方模块的存在从该警报中标识的进程模拟崩溃转储内的 Windows 模块。 出现这种情况可能表示系统遭到破坏。|-|中|
|**已查询多个域帐户**|主机数据分析已确定在短时间内从% {受害主机} 内查询了不寻常数量的不同域帐户。 这种活动可能是合法的，但也可能是折衷的指示。|-|中|
|**检测到可能的凭据转储 [多次查看]**|主机数据分析检测到使用本机 windows 工具（如 sqldumper）的方式允许从内存提取凭据。 攻击者通常会使用这些方法来提取凭据，这些凭据随后会进一步用于横向移动和权限提升。 今天在下列计算机上出现了 [x] 次此行为： [计算机名称]|-|中|
|**检测到可能尝试绕过 AppLocker**|% {受损主机} 上的主机数据分析检测到跳过 AppLocker 限制的可能尝试。 可以将 AppLocker 配置为实施一个策略，该策略限制允许在 Windows 系统上运行的可执行文件。 与此警报中标识类似的命令行模式之前，攻击者使用受信任的可执行文件（AppLocker 策略允许）来执行不受信任的代码，尝试绕过 AppLocker 策略。 这可能是合法活动，也可能是损坏的主机的指示。|-|高|
|**检测到 PsExec 执行**|"主机数据分析" 指示进程% {Process Name} 是由 PsExec 实用工具执行的。 PsExec 可用于远程运行进程。 此方法可能用于恶意目的。|-|信息|
|**检测到的勒索软件指示器 [多次查看]**|主机数据分析指示传统上与锁屏和加密勒索软件相关的可疑活动。 锁屏勒索软件会显示一条全屏消息，阻止交互使用宿主并访问其文件。 加密勒索软件会通过加密数据文件来阻止访问。 在这两种情况下，通常都会显示一条 ransom 消息，请求付款以便还原文件访问。 今天在下列计算机上出现了 [x] 次此行为： [计算机名称]|-|高|
|**检测到的勒索软件指示器**|主机数据分析指示传统上与锁屏和加密勒索软件相关的可疑活动。 锁屏勒索软件会显示一条全屏消息，阻止交互使用宿主并访问其文件。 加密勒索软件会通过加密数据文件来阻止访问。 在这两种情况下，通常都会显示一条 ransom 消息，请求付款以便还原文件访问。|-|高|
|**已执行稀有 SVCHOST 服务组**|系统进程 SVCHOST 已观察到运行罕见的服务组。 恶意软件通常使用 SVCHOST 来伪装其恶意活动。|-|信息|
|**发现 Shellcode**|Shellcode 是在恶意软件利用软件漏洞之后运行的有效负载。<br>此警报指示故障转储分析检测到可执行代码，该代码展示了恶意有效负载通常会执行的行为。 尽管非恶意软件也可以执行此行为，但它不是一般的软件开发实践。|-|中|
|**检测到粘滞键攻击**|主机数据分析表明，攻击者可能会破坏可访问性二进制文件（如粘滞键、屏幕键盘、讲述人），以便为主机% {受害主机} 提供后门访问。|-|中|
|**成功的暴力攻击**|在 Azure 订阅中的多个主机上检测到来自同一源的多个失败的身份验证尝试。 这类似于密码喷涂攻击，攻击者会在多个主机上执行大量的身份验证尝试。 某些身份验证尝试已成功登录到此订阅中的主机。|-|高|
|**置疑的完整性级别指示 RDP 劫持**|分析主机数据已检测到使用系统特权运行的 tscon-这可能表明攻击者滥用了此二进制文件，以便将上下文切换到此主机上的任何其他登录用户;这是一种已知的攻击者技术，可在网络中泄露其他用户帐户并横向移动。|-|中|
|**可疑的服务安装**|主机数据分析已检测到 tscon 作为服务安装：此二进制文件作为服务启动，这可能允许攻击者通过劫持 RDP 连接来完全切换到此主机上的任何其他登录用户;这是一种已知的攻击者技术，可在网络中泄露其他用户帐户并横向移动。|-|中|
|**已发现可疑的 Kerberos 黄金票证攻击参数**|分析主机数据检测到与 Kerberos 黄金票证攻击一致的命令行参数。|-|中|
|**检测到可疑帐户创建**|% {受损主机} 上的主机数据分析检测到创建或使用了本地帐户% {可疑帐户名称}：此帐户名称与标准 Windows 帐户或组名 "% {类似于帐户名称}" 类似。 这可能是由攻击者创建的恶意帐户，因此，命名为以避免人为管理员注意到。|-|中|
|**检测到可疑活动**|分析主机数据时检测到一个或多个在% {machine name} 上运行的进程的序列曾与恶意活动相关联。 虽然单个命令看起来可能是良性的，但警报的评分是基于这些命令的聚合。 这可能是合法活动，也可能是损坏的主机的指示。|-|中|
|**检测到可疑的 PowerShell 活动**|分析主机数据检测到在% {受害主机} 上运行的 PowerShell 脚本，该脚本的功能与已知的可疑脚本相同。 此脚本可以是合法活动，也可以是对主机的指示。|-|高|
|**已执行可疑 PowerShell cmdlet**|主机数据分析指示执行已知的恶意 PowerShell PowerSploit cmdlet。|-|中|
|**可疑的 SQL 活动**|计算机日志指示帐户 "% {user name}" 执行了 "% {process name}"。 此活动与此帐户不常见。|-|中|
|**已执行可疑 SVCHOST 进程**|系统进程 SVCHOST 在异常上下文中运行。 恶意软件通常使用 SVCHOST 来伪装其恶意活动。|-|高|
|**已执行可疑屏幕保护程序进程**|已观察到从非常见位置执行进程 "% {process name}"。 扩展名为 .scr 的文件是屏幕保护程序文件，通常驻留在 Windows 系统目录中并从其执行。|-|中|
|**可疑的卷影复制活动**|主机数据分析在资源上检测到卷影副本删除活动。 卷影复制 (VSC) 是重要的项目，用于存储数据快照。 某些恶意软件和恶意软件，旨在 VSC 破坏备份策略。|-|高|
|**检测到可疑的 WindowPosition 注册表值**|% {受损主机} 上的主机数据分析检测到 WindowPosition 注册表配置更改，这可能表明在桌面的不可见部分隐藏了应用程序窗口。  这可能是合法的活动，或者是一台受损的计算机：此类活动以前与已知的广告软件（或不需要的软件）（如 Win32/OneSystemCare 和 win32/SystemHealer 和恶意软件，如 Win32/Creprote）相关联。 当 WindowPosition 值设置为201329664时，（Hex： 0x0c00 0c00，对应于 X 轴 = 0c00，Y 轴 = 0c00），这会将控制台应用程序的窗口置于用户屏幕的不可见部分，该区域显示在显示在可见的 "开始" 菜单/任务栏下的 "视图" 中。 已知的怀疑十六进制值包括但不限于 c000c000|-|低|
|**可疑的身份验证活动**|虽然它们都没有成功，但主机已识别其中一些使用的帐户。 这类似于字典攻击，在这种攻击中，攻击者使用预定义的帐户名和密码字典执行大量的身份验证尝试，以便查找有效的访问主机的凭据。 这表示某些主机帐户名称可能存在于众所周知的帐户名称字典中。|-|中|
|**检测到可疑的代码段**|指示已使用非标准方法（如反射注入和进程替换所）分配了代码段。 该警报提供代码段的其他特征，这些特征已经过处理，可为所报告代码段的功能和行为提供上下文。|-|中|
|**可疑的命令执行**|计算机日志指出用户% {user name} 的可疑命令行执行。|-||
|**已执行的可疑双扩展文件**|主机数据的分析指示执行具有可疑双精度扩展的进程。 此扩展可能会诱使用户认为文件是安全打开的，并且可能表明系统上存在恶意软件。|-|高|
|**使用 Certutil 检测到可疑下载 [多次查看]**|% {受损主机} 上的主机数据分析检测到 certutil，这是一个内置的管理员实用工具，用于下载二进制文件，而不是与操作证书和证书数据相关的主流目的。 攻击者可能会滥用合法的管理员工具来执行恶意操作，例如，使用 certutil 下载并解码随后将执行的恶意可执行文件。 今天在下列计算机上出现了 [x] 次此行为： [计算机名称]|-|中|
|**检测到使用 Certutil 的可疑下载**|% {受损主机} 上的主机数据分析检测到 certutil，这是一个内置的管理员实用工具，用于下载二进制文件，而不是与操作证书和证书数据相关的主流目的。 攻击者可能会滥用合法的管理员工具来执行恶意操作，例如，使用 certutil 下载并解码随后将执行的恶意可执行文件。|-|中|
|**执行了可疑进程 [多次查看]**|计算机日志表明计算机上运行了可疑进程 "% {可疑进程}"，通常与攻击者尝试访问凭据有关。 今天在下列计算机上出现了 [x] 次此行为： [计算机名称]|-|高|
|**执行了可疑进程**|计算机日志指示可疑进程 "% {可疑进程}" 在计算机上运行，通常与攻击者尝试访问凭据有关。|-|高|
|**检测到可疑的进程名称 [多次查看]**|% {受损主机} 上的主机数据分析检测到其名称为可疑的进程，例如，对应于已知的攻击者工具或以一种暗示攻击者工具（可在无人参与的情况下）进行命名。 此过程可能是合法活动，或者表明某个计算机已泄露。 今天在下列计算机上出现了 [x] 次此行为： [计算机名称]|-|中|
|**检测到可疑的进程名称**|% {受损主机} 上的主机数据分析检测到其名称为可疑的进程，例如，对应于已知的攻击者工具或以一种暗示攻击者工具（可在无人参与的情况下）进行命名。 此过程可能是合法活动，或者表明某个计算机已泄露。|-|中|
|**可疑进程终止突发**|主机数据的分析指示% {Machine Name} 中存在可疑的进程终止突发。 具体而言，% {NumberOfCommands} 个进程已在% {Begin} 和% {结束} 之间终止。|-|低|
|**可疑系统文件执行**|主机数据分析检测到从异常位置运行的% {受损主机} 上的可执行文件。 此可执行文件可能是合法活动，也可能是损坏的主机的指示。|-|高|
|**已执行可疑系统进程**|检测到系统进程% {process name} 在异常上下文中运行。 恶意软件通常使用此进程名称来伪装其恶意活动。|-|高|
|**检测到可疑的命名进程**|% {受损主机} 上的主机数据分析检测到其名称与非常类似但不同于非常常见的进程（% {类似于进程名称}）的进程。 尽管此过程可能为良性攻击者，但有时会通过将其恶意工具命名为类似于合法的进程名称来以无视觉的方式隐藏。|-|中|
|**检测到异常的进程执行**|% {受损主机} 上的主机数据分析检测到由% {User Name} 执行的进程异常。 帐户（如% {User Name}）倾向于执行有限的一组操作，此执行被确定为超出字符，可能可疑。|-|高|
|**检测到 VBScript HTTP 对象分配**|检测到使用命令提示符创建 VBScript 文件。 下面的脚本包含 HTTP 对象分配命令。 此操作可用于下载恶意文件。|-|高|
|**检测到 Windows 注册表持久性方法**|分析主机数据检测到尝试在 Windows 注册表中持久保存可执行文件。 恶意软件通常使用这种方法，因此在计算机启动后仍然存在。|-|低|
|||||


## <a name="alerts-for-linux-machines"></a><a name="alerts-linux"></a>适用于 Linux 计算机的警报

[更多详细信息和说明](threat-protection.md#linux-machines)

|警报|说明|意向（[了解详细信息](#intentions)）|严重性|
|----|----|:----:|--|
|**已加载内核模块**|使用用户% {user} 的命令% {Command} 在主机% {受害主机} 中加载了内核模块。|-|低|
|**已删除内核模块**|使用用户% {user} 的命令% {Command} 在主机% {受害主机} 中删除了内核模块。|-|中|
|**已将新用户添加到 sudoers 组**|主机数据分析检测到已将用户添加到 sudoers 组，从而使其成员能够运行具有高权限的命令。|PrivilegeEscalation|低|
|**检测到 .htaccess 文件的访问权限**|% {受损主机} 上的主机数据分析检测到 .htaccess 文件的可能操作。 .Htaccess 是一个功能强大的配置文件，可让你对运行 Apache Web 软件（包括基本重定向功能）或更高级功能（如基本密码保护）的 web 服务器进行多项更改。 攻击者通常会在其泄露的计算机上修改 .htaccess 文件，以获得持久性。|-|中|
|**已清除历史记录文件**|主机数据的分析指示已清除命令历史记录日志文件。 攻击者可能会这样做来掩盖其踪迹。 用户执行的操作： "% {user name}"。|-|中|
|**尝试停止 apt-每日升级。检测到计时器服务 [多次查看]**|% {受损主机} 上的主机数据分析检测到停止 apt 服务的尝试。在最近的攻击中，攻击者会阻止攻击者停止此服务，下载恶意文件并授予其攻击的执行特权。 今天在下列计算机上出现了 [x] 次此行为： [计算机名称]|-|低|
|**尝试停止 apt-每日升级。检测到计时器服务**|% {受损主机} 上的主机数据分析检测到停止 apt 服务的尝试。在最近的攻击中，攻击者会阻止攻击者停止此服务，下载恶意文件并为攻击者授予执行权限|-|低|
|**类似于检测到的 Fairware 勒索软件的行为 [多次查看]**|% {受损主机} 上的主机数据分析检测到应用于可疑位置的 rm-rf 命令的执行。 由于 rm 会以递归方式删除文件，因此通常在离散文件夹上使用。 在这种情况下，将在可能删除大量数据的位置使用它。 已知 Fairware 勒索软件在此文件夹中执行 rm-rf 命令。 今天在下列计算机上出现了 [x] 次此行为： [计算机名称]|-|中|
|**类似于检测到 Fairware 勒索软件的行为**|% {受损主机} 上的主机数据分析检测到应用于可疑位置的 rm-rf 命令的执行。 由于 rm 会以递归方式删除文件，因此通常在离散文件夹上使用。 在这种情况下，将在可能删除大量数据的位置使用它。 已知 Fairware 勒索软件在此文件夹中执行 rm-rf 命令。|-|中|
|**检测到的行为类似于检测到的常见 Linux bot [多次查看]**|% {受损主机} 上的主机数据分析检测到通常与常见 Linux 僵尸网络相关联的进程的执行。 今天在下列计算机上出现了 [x] 次此行为： [计算机名称]|-|中|
|**类似于检测到常见 Linux bot 的行为**|% {受损主机} 上的主机数据分析检测到通常与常见 Linux 僵尸网络相关联的进程的执行。|-|中|
|**类似于检测到勒索软件的行为 [多次查看]**|% {受损主机} 上的主机数据分析检测到相似性的已知勒索软件的执行文件，这些文件可阻止用户访问其系统或个人文件，并要求 ransom 付款以便重新获得访问权限。 今天在下列计算机上出现了 [x] 次此行为： [计算机名称]|-|高|
|**检测到挖掘器映像的容器**|计算机日志表示执行与数字货币挖掘关联的映像的 Docker 容器的执行。 此行为可能表示资源被攻击者滥用。|-|高|
|**检测到的永久性尝试 [多次查看]**|% {受损主机} 上的主机数据分析检测到安装了单用户模式的启动脚本。 这种情况非常少见，因为任何合法进程都需要在该模式下执行，因此可能表示攻击者已将恶意进程添加到每个运行级别以保证持久性。 今天在下列计算机上出现了 [x] 次此行为： [计算机名称]|-|中|
|**检测到的永久性尝试**|主机数据分析检测到已安装了单用户模式的启动脚本。<br>由于在此模式下运行时，很少需要合法的进程，这可能表明攻击者已向每个运行级别添加了恶意进程以保证持久性。 |持久性|中|
|**检测到命令行中的大小写字符异常混合**|% {受损主机} 上的主机数据分析检测到具有大写和小写字符的异常混合的命令行。 这种模式（尽管可能是良性的）也是攻击者尝试在受损主机上执行管理任务时隐藏区分大小写或基于哈希的规则。|-|中|
|**检测到来自已知恶意源的文件下载 [多次查看]**|主机数据分析已检测到从% {受损主机} 上的已知恶意软件源下载文件。 今天在下列计算机上出现过 [x] 次此行为： [计算机名称]|-|中|
|**从已知的恶意源检测到的文件下载**|主机数据分析已检测到从% {受损主机} 上的已知恶意软件源下载文件。|-|中|
|**检测到可疑文件下载 [多次查看]**|分析主机数据时检测到% {受害主机} 上存在可疑的远程文件下载。 今天在下列计算机上出现此行为10次： [计算机名称]|-|低|
|**检测到可疑文件下载**|分析主机数据时检测到% {受害主机} 上存在可疑的远程文件下载。|-|低|
|**检测到可疑网络活动**|分析来自% {受害主机} 的网络流量检测到可疑的网络活动。 此类流量尽管可能是良性的，但通常由攻击者用于与恶意服务器通信，以便下载工具、命令和控制和渗透数据。 典型的相关攻击者活动包括将远程管理工具复制到受损主机上，并窃取用户数据。|-|低|
|**检测到对 nohup 命令的可疑使用 [多次检测]**|分析主机数据时检测到对% {受害主机} 的 nohup 命令的可疑使用。 攻击者已在临时目录中运行 nohup 命令，以允许其可执行文件在后台运行。 在临时目录中的文件上运行此命令并不是正常的。 今天在下列计算机上出现了 [x] 次此行为： [计算机名称]|-|中|
|**检测到对 nohup 命令的可疑使用**|分析主机数据时检测到对% {受害主机} 的 nohup 命令的可疑使用。 攻击者已在临时目录中运行 nohup 命令，以允许其可执行文件在后台运行。 在临时目录中的文件上运行此命令并不是正常的。|-|中|
|**检测到对 useradd 命令的可疑使用 [多次检测]**|分析主机数据时检测到对% {受害主机} 的 useradd 命令的可疑使用。 今天在下列计算机上出现了 [x] 次此行为： [计算机名称]|-|中|
|**检测到对 useradd 命令的可疑使用**|分析主机数据时检测到对% {受害主机} 的 useradd 命令的可疑使用。|-|中|
|**检测到数字货币挖掘相关的行为**|% {受损主机} 上的主机数据分析检测到通常与数字货币挖掘关联的进程或命令的执行。|-|高|
|**禁用审核日志记录 [多次查看]**|Linux 审核系统提供了一种方法来跟踪系统上的安全相关信息。 它记录了有关系统中发生的事件的信息。 禁用审核日志记录可能会妨碍发现系统上使用的安全策略冲突。 今天在下列计算机上出现了 [x] 次此行为： [计算机名称]|-|低|
|**从可疑位置运行的可执行文件**|分析主机数据检测到% {受害主机} 上运行的可执行文件，该文件从常见的已知可疑文件所在的位置运行。 此可执行文件可能是合法活动，也可能是损坏的主机的指示。|-|高|
|**Xorg 漏洞的利用 [多次]**|% {受损主机} 上的主机数据分析检测到具有可疑参数的 Xorg 的用户。 攻击者可能会在特权升级尝试中使用此技术。 今天在下列计算机上出现了 [x] 次此行为： [计算机名称]|-|中|
|**检测到暴露的 Docker 守护程序**|计算机日志指示 Docker 守护程序（dockerd.exe）公开 TCP 套接字。 默认情况下，当启用 TCP 套接字时，Docker 配置不使用加密或身份验证。 这样，任何有权访问相关端口的人都可以完全访问 Docker 守护程序。|-|中|
|**SSH 暴力攻击失败**|检测到失败的暴力破解攻击的攻击者：% {攻击者}。  攻击者尝试使用以下用户名访问主机：% {登录主机尝试时使用的帐户失败}。|-|中|
|**检测到隐藏的文件执行**|"主机数据分析" 指示隐藏文件是由% {user name} 执行的。 此活动可能是合法活动，也可能是损坏的主机的指示。|-|信息|
|**检测到与 DDOS 工具包相关的指示器 [多次查看]**|% {受损主机} 上的主机数据分析检测到的文件名称是与恶意软件相关的工具包的一部分，该工具包与能够启动 DDoS 攻击，打开端口和服务，并对受感染的系统进行完全控制。 这也可能是合法的活动。 今天在下列计算机上出现了 [x] 次此行为： [计算机名称]|-|中|
|**检测到与 DDOS 工具包关联的指示器**|% {受损主机} 上的主机数据分析检测到的文件名称是与恶意软件相关的工具包的一部分，该工具包与能够启动 DDoS 攻击，打开端口和服务，并对受感染的系统进行完全控制。 这也可能是合法的活动。|-|中|
|**检测到本地主机侦测 [多次查看]**|% {受损主机} 上的主机数据分析检测到通常与常见 Linux 机器人侦测关联的命令的执行。 今天在下列计算机上出现了 [x] 次此行为： [计算机名称]|-|中|
|**检测到本地主机侦测**|% {受损主机} 上的主机数据分析检测到通常与常见 Linux 机器人侦测关联的命令的执行。|-|中|
|**检测到主机防火墙操作 [多次查看]**|% {受损主机} 上的主机数据分析检测到对主机防火墙的可能操作。 攻击者通常会禁止此盗取数据。 今天在下列计算机上出现了 [x] 次此行为： [计算机名称]|-|中|
|**检测到主机防火墙的操作**|% {受损主机} 上的主机数据分析检测到对主机防火墙的可能操作。 攻击者通常会禁止此盗取数据。|-|中|
|**添加了新的 SSH 密钥 [多次查看]**|已将新的 SSH 密钥添加到授权密钥文件。 今天在下列计算机上出现了 [x] 次此行为： [计算机名称]|-|低|
|**添加了新的 SSH 密钥**|已将新的 SSH 密钥添加到授权密钥文件|-|低|
|**检测到可能的日志篡改活动 [多次查看]**|% {受损主机} 上的主机数据分析检测到在其操作过程中跟踪用户活动的文件可能会被删除。 攻击者通常会尝试规避检测，并通过删除此类日志文件不跟踪恶意活动。 今天在下列计算机上出现了 [x] 次此行为： [计算机名称]|-|中|
|**检测到可能的日志篡改活动**|% {受损主机} 上的主机数据分析检测到在其操作过程中跟踪用户活动的文件可能会被删除。 攻击者通常会尝试规避检测，并通过删除此类日志文件不跟踪恶意活动。|-|中|
|**检测到可能的攻击工具 [多次查看]**|计算机日志指示在% {受害主机} 上运行了可疑进程 "% {可疑进程}"。 此工具通常与恶意用户以某种方式攻击其他计算机相关联。 今天在下列计算机上出现了 [x] 次此行为： [计算机名称]|-|中|
|**检测到可能的攻击工具**|计算机日志指示在% {受害主机} 上运行了可疑进程 "% {可疑进程}"。 此工具通常与恶意用户以某种方式攻击其他计算机相关联。|-|中|
|**检测到可能的后门 [多次查看]**|分析主机数据检测到可疑文件正在下载，然后在订阅中的% {受害主机} 上运行。 此活动以前与后门安装相关联。 今天在下列计算机上出现了 [x] 次此行为： [计算机名称]|-|中|
|**检测到可能的凭据访问工具 [多次查看]**|计算机日志表示在由进程 "% {可疑进程}" 启动的% {受害主机} 上运行了可能的已知凭据访问工具。 此工具通常与尝试访问凭据的攻击者相关联。 今天在下列计算机上出现了 [x] 次此行为： [计算机名称]|-|中|
|**检测到可能的凭据访问工具**|计算机日志表示在由进程 "% {可疑进程}" 启动的% {受害主机} 上运行了可能的已知凭据访问工具。 此工具通常与尝试访问凭据的攻击者相关联。|-|中|
|**Hadoop Yarn 的可能利用**|% {受损主机} 上的主机数据分析检测到可能利用 Hadoop Yarn 服务。|-|中|
|**检测到可能丢失的数据 [多次查看]**|% {受损主机} 上的主机数据分析检测到可能存在数据出口情况。 攻击者经常会从他们泄露的计算机中传出数据。 目前在以下计算机上已发现 [x]] 次： [计算机名称]|-|中|
|**检测到可能丢失数据**|% {受损主机} 上的主机数据分析检测到可能存在数据出口情况。 攻击者经常会从他们泄露的计算机中传出数据。|-|中|
|**检测到可能的恶意 web shell [多次查看]**|% {受损主机} 上的主机数据分析检测到可能的 web 外壳。 攻击者通常会将 web shell 上传到已泄露的计算机，以获取持久性或进一步利用。 今天在下列计算机上出现了 [x] 次此行为： [计算机名称]|-|中|
|**检测到可能的恶意 web shell**|% {受损主机} 上的主机数据分析检测到可能的 web 外壳。 攻击者通常会将 web shell 上传到已泄露的计算机，以获取持久性或进一步利用。|-|中|
|**使用 dm-crypt 检测到的密码可能更改 [多次查看]**|% {受损主机} 上的主机数据分析检测到使用 dm-crypt 方法更改的密码更改。 攻击者可以进行此更改以继续访问并在泄露后获取持久性。 今天在下列计算机上出现了 [x] 次此行为： [计算机名称]|-|中|
|**常见文件的潜在重写 [多次查看]**|主机数据分析检测到% {受害主机} 上覆盖了常见可执行文件。 攻击者将覆盖公共文件，以便对其操作或进行暂留进行模糊处理。 今天在下列计算机上出现了 [x] 次此行为： [计算机名称]|-|中|
|**可能替代公共文件**|主机数据分析检测到% {受害主机} 上覆盖了常见可执行文件。 攻击者将覆盖公共文件，以便对其操作或进行暂留进行模糊处理。|-|中|
|**可能端口转发到外部 IP 地址 [多次查看]**|% {受损主机} 上的主机数据分析检测到启动到外部 IP 地址的端口转发。 今天在下列计算机上出现了 [x] 次此行为： [计算机名称]|-|中|
|**可能端口转发到外部 IP 地址**|主机数据分析检测到启动了到外部 IP 地址的端口转发。|渗透/CommandAndControl|中|
|**检测到可能的反向 shell [多次查看]**|% {受损主机} 上的主机数据分析检测到可能的反向 shell。 它们用于获取被泄露的计算机，以回调到攻击者拥有的计算机中。 今天在下列计算机上出现了 [x] 次此行为： [计算机名称]|-|中|
|**检测到可能的反向 shell**|% {受损主机} 上的主机数据分析检测到可能的反向 shell。 它们用于获取被泄露的计算机，以回调到攻击者拥有的计算机中。|-|中|
|**检测到特权容器**|计算机日志指示特权 Docker 容器正在运行。 特权容器对宿主的资源具有完全访问权限。 如果受到侵害，攻击者可以使用特权容器获取对主机的访问权限。|-|低|
|**特权命令在容器中运行**|计算机日志指示已在 Docker 容器中运行特权命令。 特权命令具有主机上的扩展权限。|-|低|
|**与检测到的数字货币挖掘关联的进程 [多次查看]**|% {受损主机} 上的主机数据分析检测到通常与数字货币挖掘关联的进程的执行。 今天在下列计算机上出现此行为超过100次： [计算机名称]|-|中|
|**检测到与数字货币挖掘关联的进程**|主机数据分析检测到通常与数字货币挖掘关联的进程的执行。|利用/执行|中|
|**以异常方式访问 SSH 授权密钥文件的进程**|已在类似于已知恶意软件活动的方法中访问了 SSH 授权密钥文件。 此访问可能表明攻击者正尝试获取计算机的持久访问权限。|-||
|**检测到 Python 编码的下载程序 [多次查看]**|% {受损主机} 上的主机数据分析检测到从远程位置下载并运行代码的编码 Python 的执行。 这可能表示恶意活动。 今天在下列计算机上出现了 [x] 次此行为： [计算机名称]|-|低|
|**SSH 服务器在容器中运行**|计算机日志表示 SSH 服务器在 Docker 容器中运行。 尽管此行为是有意的，但它通常表示容器配置不正确或被破坏。|-|中|
|**在主机上拍摄的屏幕截图 [多次查看]**|% {受损主机} 上的主机数据分析检测到屏幕捕获工具的用户。 攻击者可能会使用这些工具来访问隐私数据。 今天在下列计算机上出现了 [x] 次此行为： [计算机名称]|-|低|
|**检测到脚本扩展不匹配 [多次查看]**|% {受损主机} 上的主机数据分析检测到脚本解释器和作为输入提供的脚本文件的扩展名不匹配。 这通常与攻击者脚本执行相关联。 今天在下列计算机上出现了 [x] 次此行为： [计算机名称]|-|中|
|**检测到脚本扩展不匹配**|% {受损主机} 上的主机数据分析检测到脚本解释器和作为输入提供的脚本文件的扩展名不匹配。 这通常与攻击者脚本执行相关联。|-|中|
|**检测到的外壳 [多次]**|% {受损主机} 上的主机数据分析检测到从命令行生成的外壳代码。 此过程可能是合法活动，或者表明某个计算机已泄露。 今天在下列计算机上出现了 [x] 次此行为： [计算机名称]|-|中|
|**成功的 SSH 暴力攻击**|主机数据分析检测到成功的暴力破解攻击。 出现了 IP% {攻击者源 IP}，导致多次登录尝试。 已成功使用以下用户从该 IP 创建登录名：% {用于成功登录到主机的帐户}。 这意味着主机可能会受到恶意执行组件的破坏和控制。|-|高|
|**检测到可疑帐户创建**|% {受损主机} 上的主机数据分析检测到创建或使用了本地帐户% {可疑帐户名称}：此帐户名称与标准 Windows 帐户或组名 "% {类似于帐户名称}" 类似。 这可能是由攻击者创建的恶意帐户，因此，命名为以避免人为管理员注意到。|-|中|
|**检测到可疑 PHP 执行**|计算机日志指示可疑 PHP 进程正在运行。 操作包含尝试使用 PHP 进程从命令行运行 OS 命令或 PHP 代码。 虽然这种行为是合法的，但在 web 应用程序中，这种行为也会在恶意活动中被发现，如尝试利用 web shell 感染网站。|-|中|
|**检测到可疑编译 [多次查看]**|% {受损主机} 上的主机数据分析检测到可疑的编译。 攻击者通常会在其遭受提升权限的计算机上编译攻击。 今天在下列计算机上出现了 [x] 次此行为： [计算机名称]|-|中|
|**检测到可疑编译**|% {受损主机} 上的主机数据分析检测到可疑的编译。 攻击者通常会在其遭受提升权限的计算机上编译攻击。|-|中|
|**可疑文件时间戳修改**|主机数据分析检测到可疑的时间戳修改。 攻击者经常将时间戳从现有的合法文件复制到新工具，以避免检测到这些新删除的文件。|持久性/DefenseEvasion|低|
|**检测到可疑内核模块 [多次查看]**|% {受损主机} 上的主机数据分析检测到作为内核模块加载的共享对象文件。 这可能是合法活动，或者表明某个计算机已经泄露。 今天在下列计算机上出现了 [x] 次此行为： [计算机名称]|-|中|
|**可疑密码访问 [多次查看]**|分析主机数据时检测到对% {受害主机} 上的加密用户密码的可疑访问。 今天在下列计算机上出现了 [x] 次此行为： [计算机名称]|-|信息|
|**可疑密码访问**|分析主机数据时检测到对% {受害主机} 上的加密用户密码的可疑访问。|-|信息|
|**Kubernetes API 的可疑请求**|计算机日志表示对 Kubernetes API 发出了可疑的请求。 请求是从 Kubernetes 节点发送的，该节点可能来自节点中运行的某个容器。 尽管此行为是有意的，但它可能指示节点正在运行已泄露的容器。|-|中|
|||||


## <a name="alerts-for-azure-app-service"></a><a name="alerts-azureappserv"></a>Azure App Service 的警报

[更多详细信息和说明](threat-protection.md#app-services)

|警报|说明|意向（[了解详细信息](#intentions)）|严重性|
|----|----|:----:|--|
|**尝试在 Windows 应用服务上运行 Linux 命令**|分析应用服务进程检测到尝试在 Windows 应用服务上运行 Linux 命令。 此操作已被 web 应用程序运行。 此行为在市场活动中经常出现，利用公共 web 应用程序中的漏洞。|-|中|
|**在威胁智能中找到连接到 Azure App Service FTP 接口的 IP**|应用服务 FTP 日志分析检测到来自威胁情报源中找到的源地址的连接。 在此连接期间，用户访问了列出的页面。|-|中|
|**检测到异常请求模式**|Azure App Service 活动日志指示应用服务来自% {Source IP} 的异常 HTTP 活动。 此活动类似于模糊化 \ 强力强制活动模式。|-||
|**检测到尝试运行高特权命令**|分析应用服务进程检测到尝试运行需要高权限的命令。 命令在 web 应用程序上下文中运行。 尽管此行为很合理，但在 web 应用程序中，这种行为可能表明存在恶意活动。|-|中|
|**检测到从异常 IP 地址连接到网页**|Azure App Service 活动日志表示从以前从未连接过它的源 IP 地址（% {Source IP Address}）连接到敏感网页。 这可能表示有人正在尝试对你的 web 应用管理页面进行暴力攻击。 它也可能是合法用户使用的新 IP 地址的结果。|-||
|**Azure Webapps 上托管的网络钓鱼内容**|用于在 Azure AppServices 网站上找到的网络钓鱼攻击的 URL。 此 URL 是发送给 O365 客户的仿冒网站的一部分。 内容通常引诱访问者在合法的网站中输入其企业凭据或财务信息。|收集|高|
|**上传文件夹中的 PHP 文件**|Azure App Service 活动日志指示对位于上传文件夹中的可疑 PHP 页面的访问。 这种类型的文件夹通常不包含 PHP 文件。 存在这种类型的文件可能表示利用了任意文件上传漏洞的利用。|-||
|**检测到原始数据下载**|分析应用服务进程检测到从原始-数据网站（如 Pastebin）下载代码的尝试。 此操作是由 PHP 进程运行的。 此行为与尝试将 web shell 或其他恶意组件下载到应用服务的操作相关联。|-|中|
|**检测到磁盘上检测到磁盘的卷输出**|分析应用服务进程检测到正在运行将输出保存到磁盘的卷命令。 虽然这种行为是合法的，但在 web 应用程序中，这种行为也会在恶意活动中被发现，如尝试利用 web shell 感染网站。|-|低|
|**检测到垃圾邮件文件夹引用**|Azure App Service 活动日志指示标识为来自与垃圾邮件活动关联的网站的 web 活动。 如果你的网站遭到入侵并用于垃圾邮件活动，就会发生这种情况。|-||
|**检测到可能易受到攻击的网页的可疑访问**|应用服务活动日志表明访问了看似敏感的网页。<br>此可疑活动源自其访问模式与 web 扫描程序类似的源地址。 这种活动通常与攻击者尝试扫描您的网络以尝试访问敏感或易受攻击的网页有关。 |-|中|
|**检测到可疑 PHP 执行**|计算机日志指示可疑 PHP 进程正在运行。 操作包含尝试使用 PHP 进程从命令行运行操作系统命令或 PHP 代码。 虽然这种行为是合法的，但在 web 应用程序中，这种行为可能表明恶意活动，例如尝试通过 web 外壳感染网站。|执行|中|
|**检测到可疑的用户代理**|Azure App Service 活动日志指示具有可疑用户代理的请求。 此行为可以指示尝试利用应用服务应用程序中的漏洞。|-||
|**检测到可疑的 WordPress 主题调用**|应用服务活动日志表示应用服务资源上可能的代码注入活动。<br>此可疑活动类似于操作 WordPress 主题以支持服务器端代码执行的活动，后跟一个直接 web 请求来调用操作的主题文件。 此类活动可以是通过 WordPress 的攻击活动的一部分。|-|高|
|**检测到漏洞扫描程序**<br>（Joomla/WordPress/CMS）|Azure App Service 活动日志表示应用服务资源上使用了可能的漏洞扫描程序。 检测到的可疑活动类似于面向 Joomla 应用程序/WordPress 应用程序/内容管理系统（CMS）的工具。|-|中|
|**检测到 Web 指纹**<br>（NMAP/盲大象）|应用服务活动日志指出了应用服务资源上可能存在的 web 指纹活动。<br>此可疑活动与名为盲大象的工具相关联。 工具指纹 web 服务器并尝试检测已安装的应用程序及其版本。 攻击者通常使用此工具来探测 web 应用程序以查找漏洞。 |-|中|
|||||


## <a name="alerts-for-containers---azure-kubernetes-service-clusters"></a><a name="alerts-akscluster"></a>容器警报-Azure Kubernetes Service 群集

[更多详细信息和说明](threat-protection.md#azure-containers)

|警报|说明|意向（[了解详细信息](#intentions)）|严重性|
|----|----|:----:|--|
|**检测到敏感卷装载的容器**|Kubernetes 审核日志分析检测到具有敏感卷装入的新容器。 检测到的卷是将敏感文件或文件夹从节点装入容器的 hostPath 类型。 如果容器被泄露，则攻击者可以使用此装载获取对节点的访问权限。|PrivilegeEscalation|中|
|**检测到数字货币挖掘容器**|Kubernetes 审核日志分析检测到一个容器具有与数字货币挖掘工具关联的图像。|执行|高|
|**检测到公开的 Kubernetes 仪表板**|Kubernetes 审核日志分析检测到由 LoadBalancer 服务公开的 Kubernetes 仪表板。 公开的仪表板允许未经身份验证的群集管理访问，并带来安全威胁。|初始访问|高|
|**检测到 kube 命名空间中的新容器**|Kubernetes 审核日志分析检测到 kube 命名空间中的新容器，该容器不在通常在此命名空间中运行的容器中。 Kube 命名空间不应包含用户资源。 攻击者可以使用此命名空间隐藏恶意组件。|持久性|低|
|**检测到新的高特权角色**|Kubernetes 审核日志分析检测到具有高权限的新角色。 与具有高权限的角色的绑定在群集中为用户/组提升的权限。 不必要地提供提升的权限可能会导致群集中的权限提升问题。|持久性|低|
|**检测到特权容器**|Kubernetes 审核日志分析检测到新的特权容器。 特权容器可以访问节点的资源，并打破容器之间的隔离。 如果受到侵害，攻击者可以使用特权容器获取对节点的访问权限。|PrivilegeEscalation|低|
|**已检测到群集的角色绑定-管理员角色**|Kubernetes 审核日志分析检测到群集管理角色的新绑定，导致管理员权限。 不必要地提供管理员权限可能会导致群集中的权限提升问题。|持久性|低|
|||||

## <a name="alerts-for-containers---host-level"></a><a name="alerts-containerhost"></a>容器警报-主机级别

[更多详细信息和说明](threat-protection.md#azure-containers)

|警报|说明|意向（[了解详细信息](#intentions)）|严重性|
|----|----|:----:|--|
|**检测到特权容器**|计算机日志指示特权 Docker 容器正在运行。 特权容器对宿主的资源具有完全访问权限。 如果受到侵害，攻击者可以使用特权容器获取对主机的访问权限。|PrivilegeEscalation/执行|低|
|**特权命令在容器中运行**|计算机日志指示已在 Docker 容器中运行特权命令。 特权命令具有主机上的扩展权限。|PrivilegeEscalation|低|
|**检测到暴露的 Docker 守护程序**|计算机日志指示 Docker 守护程序（dockerd.exe）公开 TCP 套接字。 默认情况下，启用 TCP 套接字后，Docker 配置不使用加密或身份验证。 可以访问相关端口的任何人均可获取对 Docker 守护程序的完全访问权限。|利用/执行|中|
|**SSH 服务器在容器中运行**|计算机日志表示 SSH 服务器在 Docker 容器中运行。 尽管此行为是有意的，但它通常表示容器配置不正确或被破坏。|执行|中|
|**检测到挖掘器映像的容器**|计算机日志表示执行运行与数字货币挖掘关联的映像的 Docker 容器。 此行为可能表示资源被滥用。|执行|高|
|**Kubernetes API 的可疑请求**|计算机日志表示对 Kubernetes API 发出了可疑的请求。 请求是从 Kubernetes 节点发送的，该节点可能来自节点中运行的某个容器。 尽管此行为是有意的，但它可能指示节点正在运行已泄露的容器。|执行|中|
|**向 Kubernetes 仪表板发出可疑请求**|计算机日志指示对 Kubernetes 仪表板发出了可疑的请求。 请求是从 Kubernetes 节点发送的，该节点可能来自节点中运行的某个容器。 尽管此行为是有意的，但它可能指示节点正在运行已泄露的容器。|横向移动|中|
|||||


## <a name="alerts-for-sql-database-and-sql-data-warehouse"></a><a name="alerts-sql-db-and-warehouse"></a>SQL 数据库和 SQL 数据仓库的警报

[更多详细信息和说明](threat-protection.md#data-sql)

|警报|说明|意向（[了解详细信息](#intentions)）|严重性|
|----|----|:----:|--|
|**可能的 SQL 注入漏洞**|应用程序在数据库中生成了错误的 SQL 语句。 这可能表示存在 SQL 注入攻击的漏洞。 错误的语句有两个可能的原因。 应用程序代码中的缺陷可能构造了错误的 SQL 语句。 或者，应用程序代码或存储过程在构造出现错误的 SQL 语句时，不会清理用户输入，这可被用于 SQL 注入。|-|中|
|**尝试由可能有害的应用程序登录**|已使用可能有害的应用程序访问数据库。 在某些情况下，警报会检测操作中的渗透测试。 在其他情况下，警报会检测使用常见工具的攻击。|探测|高|
|**不熟悉的主体登录**|访问模式已更改为 SQL Server。 有人使用异常的主体（用户）登录到服务器。 在某些情况下，警报会检测合法操作（发布新应用程序或开发人员维护）。 在其他情况下，警报会检测恶意操作（以前的员工或外部攻击者）。|受到|中|
|**从异常的 Azure 数据中心登录**|对 SQL Server 的访问模式进行了更改，在这种情况下，有人从异常的 Azure 数据中心登录到了服务器。 在某些情况下，警报会检测合法操作（新应用程序或 Azure 服务）。 在其他情况下，警报会检测恶意操作（攻击者在 Azure 中从违例资源运行）。|探测|低|
|**从异常位置登录**|访问模式已更改为 SQL Server，其中有人从异常的地理位置登录到了服务器。 在某些情况下，警报会检测合法操作（发布新应用程序或开发人员维护）。 在其他情况下，警报会检测恶意操作（以前的员工或外部攻击者）。|受到|中|
|**潜在的 SQL 暴力尝试尝试**|发生了具有不同凭据的异常数量异常的登录失败。 在某些情况下，警报会检测操作中的渗透测试。 在其他情况下，警报会检测暴力破解攻击。|探测|高|
|**潜在 SQL 注入**|对已识别的应用程序进行主动攻击时，易受 SQL 注入攻击。 这意味着攻击者正尝试使用有漏洞的应用程序代码或存储过程注入恶意 SQL 语句。|-|高|
|**可能不安全的操作**|在 SQL Server 中，经常在恶意会话中使用的高特权 SQL 命令已执行。 建议默认禁用这些命令。 在某些情况下，警报会检测合法操作（运行管理脚本）。 在其他情况下，警报会检测恶意操作（攻击者使用 SQL 信任来破坏 Windows 层）。|执行|高|
|**异常导出位置**|SQL 导入和导出操作的导出存储目标发生了更改。 在某些情况下，警报会检测到合法的更改（新的备份目标）。 在其他情况下，警报会检测恶意操作（攻击者很容易将数据 exfiltrated 到文件中）。|泄露|高|
|||||


## <a name="alerts-for-azure-storage"></a><a name="alerts-azurestorage"></a>Azure 存储的警报

[更多详细信息和说明](threat-protection.md#azure-storage)

|警报|说明|意向（[了解详细信息](#intentions)）|严重性|
|----|----|:----:|--|
|**从 Tor exit 节点访问存储帐户**|指示已从称为 Tor 的活动退出节点的 IP 地址（匿名代理）成功访问此帐户。 此警报的严重性考虑使用的身份验证类型（如果有），以及这是否是此类访问的第一种情况。 可能的原因可能是已使用 Tor 访问了你的存储帐户的攻击者，或者是通过使用 Tor 访问了你的存储帐户的合法用户。|探测/利用|高|
|**从异常位置访问存储帐户**|指示访问模式对 Azure 存储帐户进行了更改。 与最近的活动进行比较时，有人从一个 IP 地址访问此帐户，该地址被视为不熟悉。 攻击者已获得帐户的访问权限，或者合法用户已从新的或异常的地理位置进行连接。 后者的一个示例是从新的应用程序或开发人员进行远程维护。|受到|低|
|**匿名访问存储帐户**|指示存储帐户的访问模式发生更改。 例如，帐户已被匿名访问（无需任何身份验证），这与此帐户上的最新访问模式有意外的对比。 可能的原因是攻击者利用了对保存 blob 存储的容器的公共读取访问权限。|受到|高|
|**已将潜在恶意软件上传到存储帐户**|指示包含潜在恶意软件的 blob 已上传到存储帐户。 此警报基于 Microsoft 威胁情报的强大功能，其中包括病毒、特洛伊木马、间谍软件和勒索软件的哈希。 可能的原因包括攻击者上传蓄意的恶意软件，或者合法用户无意中上传了潜在恶意的 blob。 在此处了解有关 Microsoft 威胁智能功能的详细信息：https://go.microsoft.com/fwlink/?linkid=2128684 |LateralMovement|高|
|**存储帐户中的异常访问检查**|表示与此帐户上的最近活动相比，对存储帐户的访问权限的检查方式与此相同。 一个可能的原因是攻击者为将来的攻击执行了侦测。|收集|中|
|**从存储帐户提取的异常数据量**|表示与此存储容器上的最近活动相比，已提取异常大量的数据。 可能的原因是，攻击者从容纳 blob 存储的容器中提取了大量数据。|泄露|中|
|**异常应用程序访问了存储帐户**|指示异常应用程序已访问此存储帐户。 可能的原因是攻击者通过使用新的应用程序访问了你的存储帐户。|受到|中|
|**存储帐户中访问权限的异常更改**|指示此存储容器的访问权限已以异常方式更改。 可能的原因是攻击者已更改容器权限以降低其安全状况或获取持久性。|持久性|中|
|**存储帐户中的异常数据浏览**|表示与此帐户上的最近活动相比，存储帐户中的 blob 或容器已以异常方式进行枚举。 一个可能的原因是攻击者为将来的攻击执行了侦测。|收集|中|
|**存储帐户中的异常删除**|指示存储帐户中发生了一个或多个意外的删除操作，与此帐户上的最近活动相比。 可能的原因是攻击者已从存储帐户中删除数据。|泄露|中|
|**将 .cspkg 的异常上传到存储帐户**|表示 Azure 云服务包（.cspkg 文件）已以异常方式上传到存储帐户，与此帐户上的最近活动相比。 可能的原因是攻击者已准备好将恶意代码从存储帐户部署到 Azure 云服务。|LateralMovement/执行|中|
|**将 .exe 的异常上传到存储帐户**|表示与此帐户上的最近活动相比，.exe 文件已以异常方式上传到存储帐户。 可能的原因是攻击者已将恶意可执行文件上传到存储帐户，或者合法用户已上传可执行文件。|LateralMovement/执行|中|
|||||


## <a name="alerts-for-azure-cosmos-db-preview"></a><a name="alerts-azurecosmos"></a>Azure Cosmos DB 的警报（预览）

[更多详细信息和说明](threat-protection.md#cosmos-db)

|警报|说明|意向（[了解详细信息](#intentions)）|严重性|
|----|----|:----:|--|
|**从异常位置访问 Cosmos DB 帐户**|指示 Azure Cosmos DB 帐户的访问模式发生了更改。 用户已从不熟悉的 IP 地址（与最近的活动）访问了此帐户。 攻击者已访问该帐户，或者合法用户已从新的和不正常的地理位置访问了它。 后者的一个示例是从新的应用程序或开发人员进行远程维护。|受到|中|
|**从 Cosmos DB 帐户提取的异常数据量**|指示 Azure Cosmos DB 帐户的数据提取模式发生了更改。 与最近的活动相比，有人提取了异常的数据量。 攻击者可能已从 Azure Cosmos DB 数据库（例如，数据渗透或泄露或未经授权的数据传输）提取了大量数据。 或者，合法用户或应用程序可能已从容器提取了异常数据量（例如，对于维护备份活动）。|泄露|中|
|||||


## <a name="alerts-for-azure-network-layer"></a><a name="alerts-azurenetlayer"></a>Azure 网络层警报

[更多详细信息和说明](threat-protection.md#network-layer)

|警报|说明|意向（[了解详细信息](#intentions)）|严重性|
|----|----|:----:|--|
|**检测到与恶意计算机的网络通信**|网络流量分析表明你的计算机（IP% {受害者 IP}）已与可能是哪个命令和控制中心通信。 当泄露的资源是负载均衡器或应用程序网关时，可疑的活动可能表示后端池中的一个或多个资源（负载均衡器或应用程序网关）与可能的命令和控制中心通信。|-|中|
|**检测到可能的受攻击计算机**|威胁情报表明你的计算机（在 IP% {Machine IP} 上）可能已被 Conficker 类型的恶意软件破坏。 Conficker 是一种面向 Microsoft Windows 操作系统的计算机蠕虫，在2008年11月首次检测到。 Conficker 感染了数百万台计算机，包括超过200个国家/地区的政府、企业和家庭计算机，使其成为最常见的已知计算机蠕虫病毒，因为 2003 Welchia 蠕虫。|-|中|
|**检测到可能的传入% {Service Name} 次强制尝试**|网络流量分析检测到% {服务名称} 与% {受害者 IP} 的通信与% {被攻击者 IP} 关联的资源% {受害主机}。 当泄露的资源是负载均衡器或应用程序网关时，可疑的传入流量会转发到后端池中的一个或多个资源（负载均衡器或应用程序网关）。 具体而言，采样的网络数据显示在端口% {受害者端口} 的% {开始时间} 和% {结束时间} 之间发生的可疑活动。 此活动与% {Service Name} 服务器的暴力破解尝试一致。|-|中|
|**检测到可能的传入 SQL 暴力尝试次数**|网络流量分析检测到与% {受害主机} 关联的与% {受害者 ip} 的传入 SQL 通信，来自于% {黑客 IP}。 当泄露的资源是负载均衡器或应用程序网关时，可疑的传入流量会转发到后端池中的一个或多个资源（负载均衡器或应用程序网关）。 具体而言，采样网络数据显示在端口% {端口号} （% {SQL 服务类型}）上的% {开始时间} 和% {结束时间} 之间存在可疑活动。 此活动与针对 SQL Server 进行的暴力破解尝试的特征相符。|-|中|
|**检测到可能的传出型拒绝服务攻击**|网络流量分析检测到源自% {受害主机} 的异常传出活动，是部署中的资源。 此活动可能表示资源已遭到破坏，现已对外部终结点进行拒绝服务攻击。 当泄露的资源是负载均衡器或应用程序网关时，可疑的活动可能表示后端池中的一个或多个资源（负载均衡器或应用程序网关）已泄露。 根据连接量，我们相信以下 Ip 可能会成为 DOS 攻击的目标：% {可能的受害人}。  请注意，这些 Ip 的通信可能是合法的。|-|中|
|**检测到可能的传出端口扫描活动**|网络流量分析检测到来自% {受害主机} 的可疑传出流量。 此流量可能是端口扫描活动的结果。 当泄露的资源是负载均衡器或应用程序网关时，可疑的传出流量来自后端池中的一个或多个资源（负载均衡器或应用程序网关）。 如果此行为是有意的，请注意，对 Azure 服务条款执行端口扫描。 如果此行为是意外的，则可能表示资源已泄露。|-|中|
|**来自多个源的可疑传入 RDP 网络活动**|网络流量分析检测到与% {受害者 IP} 关联的异常传入远程桌面协议（RDP）通信，该通信与来自多个源的资源% {受害主机} 关联。 当泄露的资源是负载均衡器或应用程序网关时，可疑的传入流量会转发到后端池中的一个或多个资源（负载均衡器或应用程序网关）。 具体而言，采样的网络数据显示% {连接到您的资源的攻击 Ip 的唯一 Ip 的数目，这对于此环境被视为不正常。 此活动可能表示尝试从多个主机（僵尸网络）暴力破解 RDP 终结点|-|中|
|**可疑的传入 RDP 网络活动**|网络流量分析检测到与% {受害者 IP} 关联的异常传入远程桌面协议（RDP）通信，该通信与% {被攻击者 IP} 关联。 当泄露的资源是负载均衡器或应用程序网关时，可疑的传入流量会转发到后端池中的一个或多个资源（负载均衡器或应用程序网关）。 具体而言，采样的网络数据显示% {个连接的连接} 个传入连接，这对于此环境被视为不正常。 此活动可能表示尝试暴力破解 RDP 终结点|-|中|
|**来自多个源的可疑传入 SSH 网络活动**|网络流量分析检测到与% {受害者 IP} 的异常传入 SSH 通信，与来自多个源的资源% {受害主机} 关联。 当泄露的资源是负载均衡器或应用程序网关时，可疑的传入流量会转发到后端池中的一个或多个资源（负载均衡器或应用程序网关）。 具体而言，采样的网络数据显示% {连接到您的资源的攻击 Ip 的唯一 Ip 的数目，这对于此环境被视为不正常。 此活动可能表示尝试从多个主机（僵尸网络）暴力破解 SSH 终结点|-|中|
|**可疑的传入 SSH 网络活动**|网络流量分析检测到与% {受害主机} 关联的与% {受害者 IP} 的异常传入 SSH 通信，其来源为% {黑客 IP}。 当泄露的资源是负载均衡器或应用程序网关时，可疑的传入流量会转发到后端池中的一个或多个资源（负载均衡器或应用程序网关）。 具体而言，采样的网络数据显示% {个连接的连接} 个传入连接，这对于此环境被视为不正常。 此活动可能表示尝试暴力破解 SSH 终结点|-|中|
|**检测到可疑的传出% {攻击协议} 流量**|网络流量分析检测到从% {受害主机} 到目标端口% {最常见端口} 的可疑传出流量。 当泄露的资源是负载均衡器或应用程序网关时，可疑的传出流量来自后端池中的一个或多个资源（负载均衡器或应用程序网关）。 此行为可能表示资源正在参与% {攻击协议} 次强制尝试或端口扫描攻击。|-|中|
|**到多个目标的可疑传出 RDP 网络活动**|网络流量分析检测到到多个目标的异常传出远程桌面协议（RDP）通信，这些目标源自% {受害主机} （% {攻击者 IP}），这是部署中的资源。 当泄露的资源是负载均衡器或应用程序网关时，可疑的传出流量来自后端池中的一个或多个资源（负载均衡器或应用程序网关）。 具体而言，采样的网络数据会显示你的计算机连接到% {受攻击的 Ip} 个唯一 Ip，这对于此环境被视为不正常。 此活动可能表示资源已泄露，现已用于暴力破解外部 RDP 终结点。 请注意，此类活动可能导致你的 IP 被外部实体标记为恶意 IP。|-|高|
|**可疑的传出 RDP 网络活动**|网络流量分析检测到从% {受害主机} （% {攻击者 IP}）发往% {受害者 IP} 的异常传出远程桌面协议（RDP）通信，部署中的资源。 当泄露的资源是负载均衡器或应用程序网关时，可疑的传出流量来自后端池中的一个或多个资源（负载均衡器或应用程序网关）。 具体而言，采样的网络数据显示资源的% {Connections} 个传出连接，这在此环境中被视为不正常。 此活动可能表明有人入侵了计算机并且正在用它来暴力破解外部 RDP 终结点。 请注意，此类活动可能导致你的 IP 被外部实体标记为恶意 IP。|-|高|
|**到多个目标的可疑传出 SSH 网络活动**|网络流量分析检测到与源自% {受害主机} （% {攻击者 IP}）的多个目标的异常传出 SSH 通信，这是部署中的资源。 当泄露的资源是负载均衡器或应用程序网关时，可疑的传出流量来自后端池中的一个或多个资源（负载均衡器或应用程序网关）。 具体而言，采样的网络数据会显示你的资源连接到% {受攻击的 Ip 的数量} 个唯一 Ip，这对于此环境被视为不正常。 此活动可能表示资源已遭到破坏，现已用于暴力破解外部 SSH 终结点。 请注意，此类活动可能导致你的 IP 被外部实体标记为恶意 IP。|-|中|
|**可疑的传出 SSH 网络活动**|网络流量分析检测到来自% {受害主机} （% {攻击者 IP}）的异常传出 SSH 通信，该通信源自% {受害主机} （% {攻击者 IP}），部署中的资源。 当泄露的资源是负载均衡器或应用程序网关时，可疑的传出流量来自后端池中的一个或多个资源（负载均衡器或应用程序网关）。 具体而言，采样的网络数据显示资源的% {Connections} 个传出连接，这在此环境中被视为不正常。 此活动可能表示资源已遭到破坏，现已用于暴力破解外部 SSH 终结点。 请注意，此类活动可能导致你的 IP 被外部实体标记为恶意 IP。|-|中|
|**建议用于阻止的 IP 地址的流量**|Azure 安全中心检测到来自建议被阻止的 IP 地址的入站流量。 这通常发生在此 IP 地址不会与此资源定期通信的情况下。 或者，安全中心的威胁智能源将 IP 地址标记为 "恶意"。|探测|低|
|||||


## <a name="alerts-for-azure-resource-manager-preview"></a><a name="alerts-azureresourceman"></a>Azure 资源管理器的警报（预览版）

[更多详细信息和说明](threat-protection.md#management-layer)

|警报|说明|意向（[了解详细信息](#intentions)）|严重性|
|----|----|:----:|--|
|**来自匿名 IP 地址的活动**|检测到来自已被标识为匿名代理 IP 地址的 IP 地址的用户活动。<br>要隐藏其设备 IP 地址的用户使用这些代理，并可用于恶意目的。 此检测使用机器学习算法，该算法可减少误报，例如组织中用户广泛使用的、带有错误标记的 IP 地址。|-|中|
|**来自不常用国家/地区的活动**|来自组织中任何用户最近或从未访问过的位置中的活动。<br>此项检测考虑过去的活动位置，以确定新的和不常见的位置。 异常情况检测引擎将存储组织中用户以往用过的位置的相关信息。|-|中|
|**无法实现的活动**|发生了两个用户活动（在单个或多个会话中），源自地理位置遥远的位置。 这种情况发生在短于用户从第一个位置到达第二个位置所需的时间段内。 这表示其他用户正在使用相同的凭据。<br>此检测使用机器学习算法，该算法会忽略导致不可能旅行情况的明显误报，如组织中其他用户定期使用的 Vpn 和位置。 该检测具有7天的初始学习期间，在此期间，它将学习新用户的活动模式。 |-|中|
|**已检测到 Azurite 工具包**|已在你的环境中检测到一个已知的云环境侦测工具包。 攻击者（或渗透测试人员）可以使用工具[Azurite](https://github.com/mwrlabs/Azurite)来映射订阅的资源并标识不安全的配置。|-|高|
|**预览-检测到使用 PowerShell 的可疑管理会话**|订阅活动日志分析检测到可疑行为。 不经常使用 PowerShell 来管理订阅环境的主体现在使用 PowerShell，并执行可保护攻击者持久性的操作。|持久性|中|
|**预览-检测到使用非活动帐户的可疑管理会话**|订阅活动日志分析检测到可疑行为。 一段很长一段时间内未使用的主体现在正在执行可确保攻击者持久性的操作。|持久性|中|
|**预览版–检测到 MicroBurst 工具包 "AzureDomainInfo" 函数运行**|已在你的环境中检测到一个已知的云环境侦测工具包。 工具 "MicroBurst" （请参阅https://github.com/NetSPI/MicroBurst)可由攻击者（或渗透测试人员）用来映射你的订阅资源、标识不安全的配置以及泄露机密信息。|-|高|
|**预览版–检测到 MicroBurst 工具包 "AzurePasswords" 函数运行**|已在你的环境中检测到一个已知的云环境侦测工具包。 工具 "MicroBurst" （请参阅https://github.com/NetSPI/MicroBurst)可由攻击者（或渗透测试人员）用来映射你的订阅资源、标识不安全的配置以及泄露机密信息。|-|高|
|**预览–检测到使用 Azure 门户的可疑管理会话**|分析订阅活动日志已检测到可疑行为。 通常不会使用 Azure 门户（Ibiza）管理订阅环境（尚未使用 Azure 门户来管理过去45天或正在主动管理的订阅）的主体现在正在使用 Azure 门户，并执行可确保攻击者持久性的操作。|-|中|
|**使用高级 Azure 持久性技术**|订阅活动日志分析检测到可疑行为。 自定义角色已被 legitimized 标识实体提供。 这可能会使攻击者在 Azure 客户环境中获得持久性。|-||
|||||


## <a name="alerts-for-azure-key-vault-preview"></a><a name="alerts-azurekv"></a>Azure Key Vault 的警报（预览）

[更多详细信息和说明](threat-protection.md#azure-keyvault)

|警报|说明|意向（[了解详细信息](#intentions)）|严重性|
|----|----|:----:|--|
|**从 TOR exit 节点访问 Key Vault**|使用 TOR IP 匿名系统的用户访问了 Key Vault，以隐藏其位置。当尝试获取对连接到 internet 的资源的未经授权的访问权限时，恶意执行组件通常会尝试隐藏其位置。|-|中|
|**Key Vault 中的大量操作**|与历史数据相比，已经执行了更大量的 Key Vault 操作。 Key Vault 活动通常在一段时间内是相同的。 这可能是活动的合法更改。 或者，你的基础结构可能已泄露，并且需要进一步调查。|-|中|
|**Key Vault 中的可疑策略更改和机密查询**|已进行 Key Vault 策略更改，然后执行列出和/或获取机密操作的操作。 此外，用户通常不会对此保管库执行此操作模式。 这强烈表明 Key Vault 受到威胁，恶意的执行组件已经窃取了中的机密。|-|中|
|**Key Vault 中的可疑机密列表和查询**|密码列表操作后跟许多机密 Get 操作。 此外，用户通常不会对此保管库执行此操作模式。 这表明有人可以转储存储在 Key Vault 中的机密，以实现潜在的恶意目的。|-|中|
|**异常应用程序访问 Key Vault**|通常不能访问 Key Vault 的应用程序已访问了该。这可能是合法的访问尝试（例如，在基础结构或代码更新后）。 这也可能表示你的基础结构受到危害，恶意执行组件正在尝试访问你的 Key Vault。|-|中|
|**Key Vault 中的异常操作模式**|与历史数据相比较，执行了一组异常的 Key Vault 操作。 Key Vault 活动通常在一段时间内是相同的。 这可能是活动的合法更改。 或者，你的基础结构可能已泄露，并且需要进一步调查。|-|中|
|**异常用户访问了 Key Vault**|Key Vault 已被通常不访问的用户访问。这可能是合法的访问尝试（例如，需要访问的新用户已加入组织）。 这也可能表示你的基础结构受到危害，恶意执行组件正在尝试访问你的 Key Vault。|-|中|
|**访问 Key Vault 的异常用户-应用程序对**|Key Vault 已被通常不访问的用户应用程序配对访问。 这可能是合法的访问尝试（例如，在基础结构或代码更新后）。 这也可能表示你的基础结构受到危害，恶意执行组件正在尝试访问你的 Key Vault。|-|中|
|**用户访问了大量的密钥保管库**|与历史数据相比，用户或应用程序访问的保管库的数目已更改。 Key Vault 活动通常在一段时间内是相同的。这可能是活动的合法更改。 或者，你的基础结构可能已泄露，并且需要进一步调查。|-|中|
|||||


## <a name="alerts-for-azure-ddos-protection"></a><a name="alerts-azureddos"></a>Azure DDoS 保护警报

[更多详细信息和说明](threat-protection.md#azure-ddos)

|警报|说明|意向（[了解详细信息](#intentions)）|严重性|
|----|----|:----:|--|
|**检测到公共 IP 的 DDoS 攻击**|检测到公共 IP （IP 地址）的 DDoS 攻击，并被减轻。|探测|高|
|**为公共 IP 缓解 DDoS 攻击**|公共 IP （IP 地址）对 DDoS 攻击的缓解。|探测|低|
|||||

## <a name="intentions"></a>举动

了解攻击的意图可帮助您更轻松地调查和报告事件。 Azure 安全中心警报包括 "目的" 字段来帮助进行这些工作。

描述受到从侦测到数据渗透的进度的一系列步骤通常称为 "终止链"。 

安全中心支持的 kill 链式方法基于[MITRE ATT&&trade; CK framework](https://attack.mitre.org/matrices/enterprise) ，下表对此进行了说明。

|Intent|说明|
|------|-------|
|**PreAttack**|PreAttack 可以是尝试访问某个资源，而不考虑恶意意向，或尝试获取目标系统的访问权限，以在利用之前收集信息。 通常，此步骤被检测为从网络外部发起的尝试，用于扫描目标系统并标识入口点。</br>有关 PreAttack 阶段的更多详细信息，请参阅[MITRE 的页面](https://attack.mitre.org/matrices/pre/)。|
|**InitialAccess**|InitialAccess 是攻击者设法获取据点的受攻击资源的阶段。 此阶段与计算主机和资源（例如用户帐户、证书等）相关。威胁参与者在此阶段之后常常能够控制资源。|
|**持久性**|持久性是对系统的任何访问、操作或配置更改，使威胁参与者在该系统上具有持久的状态。 威胁参与者通常需要通过中断来维护对系统的访问，例如系统重启、丢失凭据或其他可能需要远程访问工具重启的故障，或者提供备用后门来重新获得访问权限。|
|**PrivilegeEscalation**|权限升级是允许攻击者在系统或网络上获取更高级别的权限的操作结果。 某些工具或操作需要更高级别的特权才能工作，并且在操作过程中有可能在多个点执行此操作。 有权访问特定系统或执行攻击者所需的特定功能的用户帐户也可能被视为权限提升。|
|**DefenseEvasion**|防御规避包含一种技巧，攻击者可使用这些技术规避检测或避免其他防御措施。 有时，这些操作与其他类别中的技术（或不同的变体）相同，这些技术具有额外的破坏。|
|**CredentialAccess**|凭据访问表示一项技术，这些技术导致在企业环境中访问或控制系统、域或服务凭据。 攻击者可能会尝试从用户或管理员帐户（本地系统管理员或具有管理员访问权限的域用户）获取合法凭据，以便在网络中使用。 如果网络中有足够的访问权限，则攻击者可以创建帐户以供以后在环境中使用。|
|**发现**|发现包含允许攻击者获取有关系统和内部网络的知识的技术。 当攻击者获得对新系统的访问权限时，他们必须为其本身做出控制，以及从该系统运行的优点，并为其当前目标或在入侵期间的总体目标。 操作系统提供了许多可帮助此入侵后信息收集阶段的本机工具。|
|**LateralMovement**|横向移动包含一种技术，使攻击者能够访问和控制网络上的远程系统，但不一定包括在远程系统上执行工具。 横向移动技术可让攻击者从系统收集信息，而无需其他工具，如远程访问工具。 攻击者可以使用横向移动来实现多种目的，包括远程执行工具、旋转到其他系统、访问特定信息或文件、访问其他凭据或导致效果。|
|**执行**|执行策略表示在本地或远程系统上导致攻击者控制的代码的执行方法。 这一策略通常与横向移动一起使用，以扩展网络上远程系统的访问权限。|
|**收集**|集合包含用于在渗透之前从目标网络标识和收集信息（如敏感文件）的技术。 此类别还介绍了系统或网络上的一些位置，攻击者可能会在其中查找有关盗取的信息。|
|**渗透**|渗透是指使攻击者从目标网络中删除文件和信息的技术和特性。 此类别还介绍了系统或网络上的一些位置，攻击者可能会在其中查找有关盗取的信息。|
|**CommandAndControl**|命令和控制策略表示攻击者如何与目标网络中控制的系统进行通信。|
|**对**|影响事件主要尝试直接降低系统、服务或网络的可用性或完整性;包括对数据的操作以影响业务或操作过程。 这通常是指勒索软件、篡改、数据操作等技术。|
||||


## <a name="next-steps"></a>后续步骤
若要了解有关警报的详细信息，请参阅以下内容：

* [Azure 安全中心的威胁防护](threat-protection.md)
* [Azure 安全中心的安全警报](security-center-alerts-overview.md)
* [管理和响应 Azure 安全中心的安全警报](security-center-managing-and-responding-alerts.md)
* [导出安全警报和建议（预览）](continuous-export.md)
