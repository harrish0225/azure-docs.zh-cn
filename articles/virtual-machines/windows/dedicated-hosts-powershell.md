---
title: 使用 Azure PowerShell 部署 Azure 专用主机
description: 使用 Azure PowerShell 将 Vm 部署到专用主机。
author: cynthn
ms.service: virtual-machines-windows
ms.topic: article
ms.workload: infrastructure
ms.date: 08/01/2019
ms.author: cynthn
ms.reviewer: zivr
ms.openlocfilehash: b90189c6ba5e51a24d0c248b5aa08e9a5e4bbd9b
ms.sourcegitcommit: 849bb1729b89d075eed579aa36395bf4d29f3bd9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/28/2020
ms.locfileid: "82082843"
---
# <a name="deploy-vms-to-dedicated-hosts-using-the-azure-powershell"></a>使用 Azure PowerShell 将 Vm 部署到专用主机

本文介绍如何创建 Azure[专用主机](dedicated-hosts.md)来托管虚拟机（vm）。 

请确保已安装 Azure PowerShell 版本2.8.0 或更高版本，并且已使用`Connect-AzAccount`登录到 Azure 帐户。 

## <a name="limitations"></a>限制

- 虚拟机规模集目前在专用主机上不受支持。
- 专用主机可用的大小和硬件类型因区域而异。 若要了解详细信息，请参阅 "主机[定价" 页](https://aka.ms/ADHPricing)。

## <a name="create-a-host-group"></a>创建主机组

**主机组**是代表专用主机集合的资源。 在区域和可用性区域中创建主机组，并向其添加主机。 规划高可用性时，有其他选项可供选择。 你可以将以下一个或两个选项与专用主机一起使用： 
- 跨多个可用性区域。 在这种情况下，你需要在要使用的每个区域中都有一个主机组。
- 跨多个容错域，这些容错域映射到物理机架。 
 
在任一情况下，都需要提供主机组的容错域计数。 如果你不希望跨组中的容错域，请使用容错域计数1。 

您还可以决定使用可用性区域和容错域。 此示例将在区域1中创建具有2个容错域的主机组。 


```azurepowershell-interactive
$rgName = "myDHResourceGroup"
$location = "EastUS"

New-AzResourceGroup -Location $location -Name $rgName
$hostGroup = New-AzHostGroup `
   -Location $location `
   -Name myHostGroup `
   -PlatformFaultDomain 2 `
   -ResourceGroupName $rgName `
   -Zone 1
```

## <a name="create-a-host"></a>创建主机

现在，让我们在主机组中创建一个专用主机。 除了主机名称外，还需要提供主机的 SKU。 主机 SKU 捕获受支持的 VM 系列以及专用主机的硬件生成。

有关主机 Sku 和定价的详细信息，请参阅[Azure 专用主机定价](https://aka.ms/ADHPricing)。

如果为主机组设置了容错域计数，系统会要求你为主机指定容错域。 在此示例中，我们将主机的容错域设置为1。


```azurepowershell-interactive
$dHost = New-AzHost `
   -HostGroupName $hostGroup.Name `
   -Location $location -Name myHost `
   -ResourceGroupName $rgName `
   -Sku DSv3-Type1 `
   -AutoReplaceOnFailure 1 `
   -PlatformFaultDomain 1
```

## <a name="create-a-vm"></a>创建 VM

在专用主机上创建虚拟机。 

如果在创建主机组时指定了可用性区域，则需要在创建虚拟机时使用同一区域。 在此示例中，由于我们的主机组在区域1中，因此需要在区域1中创建 VM。  


```azurepowershell-interactive
$cred = Get-Credential
New-AzVM `
   -Credential $cred `
   -ResourceGroupName $rgName `
   -Location $location `
   -Name myVM `
   -HostId $dhost.Id `
   -Image Win2016Datacenter `
   -Zone 1 `
   -Size Standard_D4s_v3
```

> [!WARNING]
> 如果在没有足够资源的主机上创建虚拟机，则虚拟机将创建为失败状态。 

## <a name="check-the-status-of-the-host"></a>检查主机的状态

你可以使用[GetAzHost](/powershell/module/az.compute/get-azhost)和`-InstanceView`参数检查主机运行状况状态，以及你仍可以将多个虚拟机部署到主机。

```azurepowershell-interactive
Get-AzHost `
   -ResourceGroupName $rgName `
   -Name myHost `
   -HostGroupName $hostGroup.Name `
   -InstanceView
```

输出与此类似：

```
ResourceGroupName      : myDHResourceGroup
PlatformFaultDomain    : 1
AutoReplaceOnFailure   : True
HostId                 : 12345678-1234-1234-abcd-abc123456789
ProvisioningTime       : 7/28/2019 5:31:01 PM
ProvisioningState      : Succeeded
InstanceView           : 
  AssetId              : abc45678-abcd-1234-abcd-123456789abc
  AvailableCapacity    : 
    AllocatableVMs[0]  : 
      VmSize           : Standard_D2s_v3
      Count            : 32
    AllocatableVMs[1]  : 
      VmSize           : Standard_D4s_v3
      Count            : 16
    AllocatableVMs[2]  : 
      VmSize           : Standard_D8s_v3
      Count            : 8
    AllocatableVMs[3]  : 
      VmSize           : Standard_D16s_v3
      Count            : 4
    AllocatableVMs[4]  : 
      VmSize           : Standard_D32-8s_v3
      Count            : 2
    AllocatableVMs[5]  : 
      VmSize           : Standard_D32-16s_v3
      Count            : 2
    AllocatableVMs[6]  : 
      VmSize           : Standard_D32s_v3
      Count            : 2
    AllocatableVMs[7]  : 
      VmSize           : Standard_D64-16s_v3
      Count            : 1
    AllocatableVMs[8]  : 
      VmSize           : Standard_D64-32s_v3
      Count            : 1
    AllocatableVMs[9]  : 
      VmSize           : Standard_D64s_v3
      Count            : 1
  Statuses[0]          : 
    Code               : ProvisioningState/succeeded
    Level              : Info
    DisplayStatus      : Provisioning succeeded
    Time               : 7/28/2019 5:31:01 PM
  Statuses[1]          : 
    Code               : HealthState/available
    Level              : Info
    DisplayStatus      : Host available
Sku                    : 
  Name                 : DSv3-Type1
Id                     : /subscriptions/10101010-1010-1010-1010-101010101010/re
sourceGroups/myDHResourceGroup/providers/Microsoft.Compute/hostGroups/myHostGroup/hosts
/myHost
Name                   : myHost
Location               : eastus
Tags                   : {}
```

## <a name="add-an-existing-vm"></a>添加现有 VM 

可以将现有 VM 添加到专用主机，但必须先 Stop\Deallocated. VM 将 VM 移到专用主机之前，请确保支持 VM 配置：

- VM 大小必须与专用主机在同一大小系列中。 例如，如果专用主机是 DSv3，则可以 Standard_D4s_v3 VM 大小，但不能是 Standard_A4_v2。 
- VM 需要与专用主机位于同一区域。
- VM 不能是邻近位置组的一部分。 将 VM 移到专用主机之前，请先从邻近感应放置组中将其删除。 有关详细信息，请参阅[将 VM 移出邻近位置组](https://docs.microsoft.com/azure/virtual-machines/windows/proximity-placement-groups#move-an-existing-vm-out-of-a-proximity-placement-group)
- VM 不能位于可用性集中。
- 如果 VM 位于可用性区域中，则它必须是与主机组相同的可用性区域。 VM 和主机组的可用性区域设置必须匹配。

将变量的值替换为自己的信息。

```azurepowershell-interactive
$vmRGName = "movetohost"
$vmName = "myVMtoHost"
$dhRGName = "myDHResourceGroup"
$dhGroupName = "myHostGroup"
$dhName = "myHost"

$myDH = Get-AzHost `
   -HostGroupName $dhGroupName `
   -ResourceGroupName $dhRGName `
   -Name $dhName
   
$myVM = Get-AzVM `
   -ResourceGroupName $vmRGName `
   -Name $vmName
   
$myVM.Host = New-Object Microsoft.Azure.Management.Compute.Models.SubResource

$myVM.Host.Id = "$myDH.Id"

Stop-AzVM `
   -ResourceGroupName $vmRGName `
   -Name $vmName -Force
   
Update-AzVM `
   -ResourceGroupName $vmRGName `
   -VM $myVM -Debug
   
Start-AzVM `
   -ResourceGroupName $vmRGName `
   -Name $vmName
```


## <a name="clean-up"></a>清除

即使部署了虚拟机，也需要为你的专用主机付费。 应删除当前未使用的任何主机以节省成本。  

只有在没有任何更长的虚拟机使用它时，才能删除该主机。 使用[new-azvm](/powershell/module/az.compute/remove-azvm)删除 vm。

```azurepowershell-interactive
Remove-AzVM -ResourceGroupName $rgName -Name myVM
```

删除 Vm 后，可以使用[AzHost](/powershell/module/az.compute/remove-azhost)删除该主机。

```azurepowershell-interactive
Remove-AzHost -ResourceGroupName $rgName -Name myHost
```

删除所有主机后，可以使用[AzHostGroup](/powershell/module/az.compute/remove-azhostgroup)删除该主机组。 

```azurepowershell-interactive
Remove-AzHost -ResourceGroupName $rgName -Name myHost
```

还可以使用[AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup)在单个命令中删除整个资源组。 这会删除在组中创建的所有资源，包括所有 Vm、主机和主机组。
 
```azurepowershell-interactive
Remove-AzResourceGroup -Name $rgName
```


## <a name="next-steps"></a>后续步骤

- [这里](https://github.com/Azure/azure-quickstart-templates/blob/master/201-vm-dedicated-hosts/README.md)有一个示例模板，它使用区域和容错域实现了区域中的最大复原能力。

- 你还可以使用[Azure 门户](dedicated-hosts-portal.md)部署专用主机。
