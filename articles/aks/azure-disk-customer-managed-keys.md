---
title: 使用客户管理的密钥加密 Azure Kubernetes Service (AKS) 中的 Azure 磁盘
description: 自带密钥 (BYOK) 来加密 AKS OS 和数据磁盘。
services: container-service
ms.topic: article
ms.date: 01/12/2020
ms.openlocfilehash: bb6ba5e6dd4ace9e33043079c0f435c10baf5cb2
ms.sourcegitcommit: 849bb1729b89d075eed579aa36395bf4d29f3bd9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/28/2020
ms.locfileid: "77596498"
---
# <a name="bring-your-own-keys-byok-with-azure-disks-in-azure-kubernetes-service-aks"></a>对 Azure Kubernetes Service (AKS) 中的 Azure 磁盘使用自带密钥 (BYOK)

Azure 存储对静态存储帐户中的所有数据进行加密。 默认情况下，使用 Microsoft 管理的密钥对数据进行加密。 为了更进一步控制加密密钥，可以提供[客户托管密钥][customer-managed-keys]，将其用于对 AKS 群集的 OS 和数据磁盘进行静态加密。

> [!NOTE]
> BYOK Linux 和基于 Windows 的 AKS 群集在支持 Azure 托管磁盘服务器端加密的[azure 区域][supported-regions]中提供。

## <a name="before-you-begin"></a>在开始之前

* 本文假定你要创建*新的 AKS 群集*。

* 使用密钥保管库加密托管磁盘时，必须为 *Azure 密钥保管库*启用软删除和清除保护。

* 需要 Azure CLI 2.0.79 或更高版本，以及 aks-preview 0.4.26 扩展

> [!IMPORTANT]
> AKS 预览功能是自助式选择加入功能。 预览版“按原样”提供，并且仅在“可用情况下”提供，不包含在服务级别协议和有限保障中。 AKS 预览版的内容部分包含在客户支持中，我们只能尽力提供支持。 因此，这些功能不应用于生产。 有关其他信息，请参阅以下支持文章：
>
> * [AKS 支持策略](support-policies.md)
> * [Azure 支持常见问题](faq.md)

## <a name="install-latest-aks-cli-preview-extension"></a>安装最新的 AKS CLI 预览版扩展

若要使用客户管理的密钥，需要具有 *aks-preview* CLI 扩展版本 0.4.26 或更高版本。 使用 [az extension add][az-extension-add] 命令安装 *aks-preview* Azure CLI 扩展，然后使用 [az extension update][az-extension-update] 命令检查是否有任何可用的更新：

```azurecli-interactive
# Install the aks-preview extension
az extension add --name aks-preview

# Update the extension to make sure you have the latest version installed
az extension update --name aks-preview
```

## <a name="create-an-azure-key-vault-instance"></a>创建 Azure Key Vault 实例

使用 Azure Key Vault 实例存储密钥。  可以通过 Azure 门户[使用 Azure Key Vault 配置客户管理的密钥][byok-azure-portal]

创建一个新的*资源组*，然后创建一个新的*密钥保管库*实例，并启用软删除和清除保护。  确保为每个命令使用同一区域和资源组名称。

```azurecli-interactive
# Optionally retrieve Azure region short names for use on upcoming commands
az account list-locations
```

```azurecli-interactive
# Create new resource group in a supported Azure region
az group create -l myAzureRegionName -n myResourceGroup

# Create an Azure Key Vault resource in a supported Azure region
az keyvault create -n myKeyVaultName -g myResourceGroup -l myAzureRegionName  --enable-purge-protection true --enable-soft-delete true
```

## <a name="create-an-instance-of-a-diskencryptionset"></a>创建 DiskEncryptionSet 的实例

将*myKeyVaultName*替换为你的密钥保管库的名称。  还需要一个存储在 Azure Key Vault 中的*密钥*才能完成以下步骤。  在前面的步骤中创建的 Key Vault 中存储现有密钥，或[生成新密钥][key-vault-generate]，并将下面的*myKeyName*替换为你的密钥的名称。
    
```azurecli-interactive
# Retrieve the Key Vault Id and store it in a variable
keyVaultId=$(az keyvault show --name myKeyVaultName --query [id] -o tsv)

# Retrieve the Key Vault key URL and store it in a variable
keyVaultKeyUrl=$(az keyvault key show --vault-name myKeyVaultName  --name myKeyName  --query [key.kid] -o tsv)

# Create a DiskEncryptionSet
az disk-encryption-set create -n myDiskEncryptionSetName  -l myAzureRegionName  -g myResourceGroup --source-vault $keyVaultId --key-url $keyVaultKeyUrl 
```

## <a name="grant-the-diskencryptionset-access-to-key-vault"></a>授予 DiskEncryptionSet 对 key vault 的访问权限

使用在前面的步骤中创建的 DiskEncryptionSet 和资源组，并授予 DiskEncryptionSet 资源对 Azure 密钥保管库的访问权限。

```azurecli-interactive
# Retrieve the DiskEncryptionSet value and set a variable
desIdentity=$(az disk-encryption-set show -n myDiskEncryptionSetName  -g myResourceGroup --query [identity.principalId] -o tsv)

# Update security policy settings
az keyvault set-policy -n myKeyVaultName -g myResourceGroup --object-id $desIdentity --key-permissions wrapkey unwrapkey get

# Assign the reader role
az role assignment create --assignee $desIdentity --role Reader --scope $keyVaultId
```

## <a name="create-a-new-aks-cluster-and-encrypt-the-os-disk"></a>创建新的 AKS 群集并加密 OS 磁盘

创建**新的资源组**和 AKS 群集，并使用密钥对 OS 磁盘进行加密。 只有1.17 版的 Kubernetes 支持客户管理的密钥。 

> [!IMPORTANT]
> 确保为 AKS 群集创建新的资源组

```azurecli-interactive
# Retrieve the DiskEncryptionSet value and set a variable
diskEncryptionSetId=$(az resource show -n mydiskEncryptionSetName -g myResourceGroup --resource-type "Microsoft.Compute/diskEncryptionSets" --query [id] -o tsv)

# Create a resource group for the AKS cluster
az group create -n myResourceGroup -l myAzureRegionName

# Create the AKS cluster
az aks create -n myAKSCluster -g myResourceGroup --node-osdisk-diskencryptionset-id $diskEncryptionSetId --kubernetes-version 1.17.0 --generate-ssh-keys
```

将新节点池添加到上面创建的群集中时，在创建过程中提供的客户托管密钥用于对 OS 磁盘进行加密。

## <a name="encrypt-your-aks-cluster-data-disk"></a>加密 AKS 群集数据磁盘

还可以使用你自己的密钥对 AKS 数据磁盘进行加密。

> [!IMPORTANT]
> 确保你具有正确的 AKS 凭据。 服务主体将需要具有对部署 diskencryptionset 的资源组的参与者访问权限。 否则，你将收到一条错误消息，指出服务主体没有权限。

```azurecli-interactive
# Retrieve your Azure Subscription Id from id property as shown below
az account list
```

```
someuser@Azure:~$ az account list
[
  {
    "cloudName": "AzureCloud",
    "id": "666e66d8-1e43-4136-be25-f25bb5de5893",
    "isDefault": true,
    "name": "MyAzureSubscription",
    "state": "Enabled",
    "tenantId": "3ebbdf90-2069-4529-a1ab-7bdcb24df7cd",
    "user": {
      "cloudShellID": true,
      "name": "someuser@azure.com",
      "type": "user"
    }
  }
]
```

创建一个名为 **byok-azure-disk.yaml** 的文件，在其中包含以下信息。  将 myAzureSubscriptionId、myResourceGroup 和 myDiskEncrptionSetName 替换为你的值，并应用 yaml。  请确保使用部署 DiskEncryptionSet 的资源组。  如果使用 Azure Cloud Shell，则可使用 vi 或 nano 来创建此文件，就像在虚拟或物理系统上工作一样：

```
kind: StorageClass
apiVersion: storage.k8s.io/v1  
metadata:
  name: hdd
provisioner: kubernetes.io/azure-disk
parameters:
  skuname: Standard_LRS
  kind: managed
  diskEncryptionSetID: "/subscriptions/{myAzureSubscriptionId}/resourceGroups/{myResourceGroup}/providers/Microsoft.Compute/diskEncryptionSets/{myDiskEncryptionSetName}"
```
接下来，在 AKS 群集中运行此部署：
```azurecli-interactive
# Get credentials
az aks get-credentials --name myAksCluster --resource-group myResourceGroup --output table

# Update cluster
kubectl apply -f byok-azure-disk.yaml
```

## <a name="limitations"></a>限制

* BYOK 目前仅在特定[Azure 区域][supported-regions]的 GA 和预览版中提供
* Kubernetes 版本 1.17 及更高版本支持 OS 磁盘加密   
* 仅适用于支持 BYOK 的区域
* 当前仅针对新的 AKS 群集进行加密，当前无法升级现有群集
* 使用虚拟机规模集的 AKS 群集是必需的，不支持虚拟机可用性集


## <a name="next-steps"></a>后续步骤

查看 [AKS 群集安全性最佳做法][best-practices-security]

<!-- LINKS - external -->

<!-- LINKS - internal -->
[az-extension-add]: /cli/azure/extension#az-extension-add
[az-extension-update]: /cli/azure/extension#az-extension-update
[best-practices-security]: /azure/aks/operator-best-practices-cluster-security
[byok-azure-portal]: /azure/storage/common/storage-encryption-keys-portal
[customer-managed-keys]: /azure/virtual-machines/windows/disk-encryption#customer-managed-keys
[key-vault-generate]: /azure/key-vault/key-vault-manage-with-cli2
[supported-regions]: /azure/virtual-machines/windows/disk-encryption#supported-regions
