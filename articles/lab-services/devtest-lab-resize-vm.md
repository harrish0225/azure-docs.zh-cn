---
title: 在 Azure 开发测试实验室的实验室中调整 VM 大小
description: 了解如何根据 CPU、网络或磁盘性能需求更改 Azure 开发测试实验室中的虚拟机（VM）的大小。
services: devtest-lab,virtual-machines,lab-services
documentationcenter: na
author: spelluru
manager: femila
editor: ''
ms.assetid: 8460f09e-482f-48ba-a57a-c95fe8afa001
ms.service: lab-services
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 01/24/2020
ms.author: spelluru
ms.openlocfilehash: bf7c425766a97aaa1d143133f04502a0aa3c36cb
ms.sourcegitcommit: 849bb1729b89d075eed579aa36395bf4d29f3bd9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/28/2020
ms.locfileid: "76756171"
---
# <a name="resize-a-vm-in-a-lab-in-azure-devtest-labs"></a>在 Azure 开发测试实验室的实验室中调整 VM 大小
Azure 虚拟机的重要功能之一是，它允许你基于对 CPU、网络或磁盘性能的需求来更改虚拟机 (VM) 大小。 Azure 开发测试实验室现支持实验室中此 VM 功能。 调整大小功能遵循实验室中允许的 VM 大小的实验室策略。 也就是说，可以将 VM 大小更改为只允许在实验室中使用的大小。 


## <a name="steps-to-resize-a-vm-in-a-lab"></a>在实验室调整 VM 大小的步骤 
若要在 Azure 开发测试实验室的实验室中调整 VM 大小，请执行以下步骤： 

> [!NOTE]
> 如果通过远程桌面会话 (RDP) 连接到 VM，请在调整大小之前保存工作，并断开与 VM 的连接。

1. 登录 [Azure 门户](https://portal.azure.com)。
2. 选择“所有服务”****，并从列表中选择“开发测试实验室”****。
3. 从实验室列表中，选择包含要调整大小的 VM 的实验室。  
4. 在左面板中，选择“我的虚拟机”****。 
5. 从 VM 列表中选择 VM。
6. 如果 VM 正在运行，则选择工具栏上的“停止”****。 检查“通知”**** 窗口中的操作状态。 耐心等待直到 VM 停止，然后关闭“通知”**** 窗口。 

    ![停止 VM](media/devtest-lab-resize-vm/stop-vm.png)
1. 在 VM 虚拟机页上，选择左侧菜单“设置”**** 下的“大小”****。

    ![“大小”菜单](media/devtest-lab-resize-vm/size-menu.png)
1. 在“选择大小”**** 窗口中，浏览和选择 VM 大小，然后单击“选择”****。     
1. 检查“通知”**** 窗口中调整大小操作的状态。

    ![调整大小状态](media/devtest-lab-resize-vm/resize-status.png)
10. 调整大小操作成功后，关闭“通知”**** 窗口。 
11. 选择左侧菜单中的“概述”****，然后选择工具栏上的“重新启动”**** 以重启 VM。 

## <a name="next-steps"></a>后续步骤
有关 Azure 虚拟机支持的调整大小功能的详细信息，请参阅[调整虚拟机大小](https://azure.microsoft.com/blog/resize-virtual-machines/)。


