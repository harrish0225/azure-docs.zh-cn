---
title: 教程 - 使用 Azure PowerShell 创建自定义 VM 映像
description: 本教程介绍如何使用 Azure PowerShell 创建一个存储在 Azure 共享映像库中的 Windows 自定义虚拟机映像。
author: cynthn
ms.service: virtual-machines-windows
ms.subservice: imaging
ms.topic: tutorial
ms.workload: infrastructure
ms.date: 05/01/2020
ms.author: cynthn
ms.custom: mvc
ms.openlocfilehash: 9061cbbae0b30881fffe1762208216cb8009594a
ms.sourcegitcommit: e0330ef620103256d39ca1426f09dd5bb39cd075
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/05/2020
ms.locfileid: "82791572"
---
# <a name="tutorial-create-windows-vm-images-with-azure-powershell"></a>教程：使用 Azure PowerShell 创建 Windows VM 映像

映像可用于启动部署并确保多个 VM 的一致性。 在本教程中，我们使用 PowerShell 创建自己的 Azure 虚拟机专用化映像，并将其存储在共享映像库中。 学习如何：

> [!div class="checklist"]
> * 创建共享映像库
> * 创建映像定义
> * 创建映像版本
> * 从映像创建 VM 
> * 共享映像库



## <a name="before-you-begin"></a>开始之前

下列步骤详细说明如何将现有 VM 转换为可重用自定义映像，以便将其用于创建新的 VM。

若要完成本教程中的示例，必须具备现有虚拟机。 如果需要，可以参阅 [PowerShell 快速入门](quick-create-powershell.md)来创建本教程所用的 VM。 在学习本教程期间，请根据需要替换资源名称。

## <a name="overview"></a>概述

[共享映像库](shared-image-galleries.md)大大简化了整个组织中的自定义映像共享。 自定义映像类似于市场映像，不同的是自定义映像的创建者是自己。 自定义映像可用于启动配置，例如预加载应用程序、应用程序配置和其他 OS 配置。 

共享映像库可让你与他人共享自定义 VM 映像。 选择要共享哪些映像，要在哪些区域中共享，以及希望与谁共享它们。 

共享映像库功能具有多种资源类型：

[!INCLUDE [virtual-machines-shared-image-gallery-resources](../../../includes/virtual-machines-shared-image-gallery-resources.md)]


## <a name="launch-azure-cloud-shell"></a>启动 Azure Cloud Shell

Azure Cloud Shell 是免费的交互式 shell，可以使用它运行本文中的步骤。 它预安装有常用 Azure 工具并将其配置与帐户一起使用。 

若要打开 Cloud Shell，只需要从代码块的右上角选择“试一试”。  也可以通过转到 [https://shell.azure.com/powershell](https://shell.azure.com/powershell) 在单独的浏览器标签页中启动 Cloud Shell。 选择“复制”以复制代码块，将其粘贴到 Cloud Shell 中，然后按 Enter 来运行它。 

## <a name="get-the-vm"></a>获取 VM

可以使用 [Get-AzVM](https://docs.microsoft.com/powershell/module/az.compute/get-azvm) 查看资源组中可用的 VM 列表。 了解 VM 名称和资源组后，可以再次使用 `Get-AzVM` 来获取 VM 对象并将其存储在变量中，供稍后使用。 此示例从“myResourceGroup”资源组获取名为 sourceVM 的 VM，并将其分配给变量 $vm   。 

```azurepowershell-interactive
$sourceVM = Get-AzVM `
   -Name sourceVM `
   -ResourceGroupName myResourceGroup
```

## <a name="create-a-resource-group"></a>创建资源组

使用 [New-AzResourceGroup](https://docs.microsoft.com/powershell/module/az.resources/new-azresourcegroup) 命令创建资源组。

Azure 资源组是在其中部署和管理 Azure 资源的逻辑容器。 以下示例在“EastUS”区域中创建名为“myGalleryRG”的资源组   ：

```azurepowershell-interactive
$resourceGroup = New-AzResourceGroup `
   -Name 'myGalleryRG' `
   -Location 'EastUS'
```

## <a name="create-an-image-gallery"></a>创建映像库 

映像库是用于启用映像共享的主要资源。 允许用于库名称的字符为大写或小写字母、数字、点和句点。 库名称不能包含短划线。 库名称在你的订阅中必须唯一。 

使用 [New-AzGallery](https://docs.microsoft.com/powershell/module/az.compute/new-azgallery) 创建映像库。 以下示例在“myGalleryRG”资源组中创建名为“myGallery”的库   。

```azurepowershell-interactive
$gallery = New-AzGallery `
   -GalleryName 'myGallery' `
   -ResourceGroupName $resourceGroup.ResourceGroupName `
   -Location $resourceGroup.Location `
   -Description 'Shared Image Gallery for my organization'  
```


## <a name="create-an-image-definition"></a>创建映像定义 

映像定义为映像创建逻辑分组。 映像定义用于管理在其中创建的映像版本的相关信息。 映像定义名称可以由大写或小写字母、数字、点、短划线和句点构成。 有关可为映像定义指定的值的详细信息，请参阅[映像定义](https://docs.microsoft.com/azure/virtual-machines/windows/shared-image-galleries#image-definitions)。

使用 [New-AzGalleryImageDefinition](https://docs.microsoft.com/powershell/module/az.compute/new-azgalleryimageversion) 创建映像定义。 在此示例中，库映像名为 myGalleryImage，它是为专用化映像创建的  。 

```azurepowershell-interactive
$galleryImage = New-AzGalleryImageDefinition `
   -GalleryName $gallery.Name `
   -ResourceGroupName $resourceGroup.ResourceGroupName `
   -Location $gallery.Location `
   -Name 'myImageDefinition' `
   -OsState specialized `
   -OsType Windows `
   -Publisher 'myPublisher' `
   -Offer 'myOffer' `
   -Sku 'mySKU'
```


## <a name="create-an-image-version"></a>创建映像版本

使用 [New-AzGalleryImageVersion](https://docs.microsoft.com/powershell/module/az.compute/new-azgalleryimageversion) 从 VM 创建映像版本。 

允许用于映像版本的字符为数字和句点。 数字必须在 32 位整数范围内。 格式：MajorVersion.MinorVersion.Patch    。

在此示例中，映像版本为 1.0.0，该版本被复制到“美国东部”和“美国中南部”数据中心    。 选择复制的目标区域时，需要将源区域包含为复制目标  。

若要从 VM 创建映像版本，请对 `-Source` 使用 `$vm.Id.ToString()`。

```azurepowershell-interactive
   $region1 = @{Name='South Central US';ReplicaCount=1}
   $region2 = @{Name='East US';ReplicaCount=2}
   $targetRegions = @($region1,$region2)

New-AzGalleryImageVersion `
   -GalleryImageDefinitionName $galleryImage.Name`
   -GalleryImageVersionName '1.0.0' `
   -GalleryName $gallery.Name `
   -ResourceGroupName $resourceGroup.ResourceGroupName `
   -Location $resourceGroup.Location `
   -TargetRegion $targetRegions  `
   -Source $vm.Id.ToString() `
   -PublishingProfileEndOfLifeDate '2020-12-01'
```

可能需要一段时间才能将映像复制到所有目标区域。


## <a name="create-a-vm"></a>创建 VM 

获得专用化映像后，可以创建一个或多个新 VM。 使用 [New-AzVM](https://docs.microsoft.com/powershell/module/az.compute/new-azvm) cmdlet。 若要使用映像，请使用“Set-AzVMSourceImage”并将“-Id”设置为映像定义 ID（在本例中为 $galleryImage.Id），以始终使用最新的映像版本。` and set the ` 

在此示例中，请根据需要替换资源名称。 

```azurepowershell-interactive
# Create some variables for the new VM.
$resourceGroup = "myResourceGroup"
$location = "South Central US"
$vmName = "mySpecializedVM"

# Create a resource group
New-AzResourceGroup -Name $resourceGroup -Location $location

# Create the network resources.
$subnetConfig = New-AzVirtualNetworkSubnetConfig -Name mySubnet -AddressPrefix 192.168.1.0/24
$vnet = New-AzVirtualNetwork -ResourceGroupName $resourceGroup -Location $location `
  -Name MYvNET -AddressPrefix 192.168.0.0/16 -Subnet $subnetConfig
$pip = New-AzPublicIpAddress -ResourceGroupName $resourceGroup -Location $location `
  -Name "mypublicdns$(Get-Random)" -AllocationMethod Static -IdleTimeoutInMinutes 4
$nsgRuleRDP = New-AzNetworkSecurityRuleConfig -Name myNetworkSecurityGroupRuleRDP  -Protocol Tcp `
  -Direction Inbound -Priority 1000 -SourceAddressPrefix * -SourcePortRange * -DestinationAddressPrefix * `
  -DestinationPortRange 3389 -Access Allow
$nsg = New-AzNetworkSecurityGroup -ResourceGroupName $resourceGroup -Location $location `
  -Name myNetworkSecurityGroup -SecurityRules $nsgRuleRDP
$nic = New-AzNetworkInterface -Name $vmName -ResourceGroupName $resourceGroup -Location $location `
  -SubnetId $vnet.Subnets[0].Id -PublicIpAddressId $pip.Id -NetworkSecurityGroupId $nsg.Id

# Create a virtual machine configuration using $imageVersion.Id to specify the image version.
$vmConfig = New-AzVMConfig -VMName $vmName -VMSize Standard_D1_v2 | `
Set-AzVMSourceImage -Id $galleryImage.Id | `
Add-AzVMNetworkInterface -Id $nic.Id

# Create a virtual machine
New-AzVM -ResourceGroupName $resourceGroup -Location $location -VM $vmConfig
```


## <a name="share-the-gallery"></a>共享库

建议在映像库级别共享访问权限。 使用电子邮件地址和 [Get-AzADUser](/powershell/module/az.resources/get-azaduser) cmdlet 获取用户的对象 ID，然后使用 [New-AzRoleAssignment](/powershell/module/Az.Resources/New-AzRoleAssignment) 为用户授予对库的访问权限。 请将此示例中的示例电子邮件地址 alinne_montes@contoso.com 替换为你自己的信息。

```azurepowershell-interactive
# Get the object ID for the user
$user = Get-AzADUser -StartsWith alinne_montes@contoso.com
# Grant access to the user for our gallery
New-AzRoleAssignment `
   -ObjectId $user.Id `
   -RoleDefinitionName Reader `
   -ResourceName $gallery.Name `
   -ResourceType Microsoft.Compute/galleries `
   -ResourceGroupName $resourceGroup.ResourceGroupName
```
   
## <a name="clean-up-resources"></a>清理资源

不再需要时，可以使用 [Remove-AzResourceGroup](https://docs.microsoft.com/powershell/module/az.resources/remove-azresourcegroup) cmdlet 删除资源组和所有相关资源：

```azurepowershell-interactive
# Delete the gallery 
Remove-AzResourceGroup -Name myGalleryRG

# Delete the VM
Remove-AzResourceGroup -Name myResoureceGroup
```

## <a name="azure-image-builder"></a>Azure 映像生成器

Azure 还提供一个基于 Packer 的服务：[Azure VM 映像生成器](https://docs.microsoft.com/azure/virtual-machines/windows/image-builder-overview)。 只需在模板中描述你的自定义设置，然后该模板即会处理映像的创建。 

## <a name="next-steps"></a>后续步骤

在本教程中，你已创建一个专用化 VM 映像。 你已了解如何执行以下操作：

> [!div class="checklist"]
> * 创建共享映像库
> * 创建映像定义
> * 创建映像版本
> * 从映像创建 VM 
> * 共享映像库

请转到下一教程，了解如何创建高度可用的虚拟机。

> [!div class="nextstepaction"]
> [创建高度可用的 VM](tutorial-availability-sets.md)



