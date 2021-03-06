---
title: 配置混合 Kubernetes 群集与 Azure Monitor 容器 |Microsoft Docs
description: 本文介绍如何配置容器 Azure Monitor，以监视托管在 Azure Stack 或其他环境中的 Kubernetes 群集。
ms.topic: conceptual
ms.date: 04/22/2020
ms.openlocfilehash: a0008f7a2d6b808a8ff55d85330801305361d7c8
ms.sourcegitcommit: 849bb1729b89d075eed579aa36395bf4d29f3bd9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/28/2020
ms.locfileid: "82185959"
---
# <a name="configure-hybrid-kubernetes-clusters-with-azure-monitor-for-containers"></a>为容器配置混合 Kubernetes 群集 Azure Monitor

容器 Azure Monitor 提供丰富的监视体验，适用于 azure 上的 Azure Kubernetes 服务（AKS）和[AKS 引擎](https://github.com/Azure/aks-engine)，这是托管在 azure 上的自托管 Kubernetes 群集。 本文介绍如何启用对 Azure 外部托管的 Kubernetes 群集的监视并实现类似的监视体验。

## <a name="supported-configurations"></a>支持的配置

用于容器的 Azure Monitor 正式支持以下内容。

* 环境： 

    * 本地 Kubernetes
    
    * Azure 上的 AKS 引擎和 Azure Stack。 有关详细信息，请参阅[Azure Stack 上的 AKS Engine](https://docs.microsoft.com/azure-stack/user/azure-stack-kubernetes-aks-engine-overview?view=azs-1908)
    
    * [OpenShift](https://docs.openshift.com/container-platform/4.3/welcome/index.html)版本4及更高版本，本地或其他云环境。

* Kubernetes 和支持策略的版本与 [AKS 支持](../../aks/supported-kubernetes-versions.md)的版本相同。

* 容器运行时： Docker、小鲸鱼和 CRI 兼容的运行时，例如 CRI 和 ContainerD。

* 适用于主节点和工作节点的 Linux OS 版本： Ubuntu （18.04 LTS 和 16.04 LTS）和 Red Hat Enterprise Linux CoreOS 43.81。

* 支持的访问控制： Kubernetes RBAC 和非 RBAC

## <a name="prerequisites"></a>先决条件

在开始之前，请确保做好以下准备：

* Log Analytics 工作区。

    用于容器的 Azure Monitor 支持在 Azure [产品(按区域)](https://azure.microsoft.com/global-infrastructure/services/?regions=all&products=monitor) 中列出的区域中的 Log Analytics 工作区。 若要创建自己的工作区，可以通过[Azure 资源管理器](../platform/template-workspace-configuration.md)、 [PowerShell](../scripts/powershell-sample-create-workspace.md?toc=%2fpowershell%2fmodule%2ftoc.json)或[Azure 门户](../learn/quick-create-workspace.md)创建。

    >[!NOTE]
    >不支持对同一 Log Analytics 工作区使用同一群集名称监视多个群集。 群集名称必须是唯一的。
    >

* 你是**Log Analytics 参与者角色**的成员，用于启用容器监视。 有关如何控制对 Log Analytics 工作区的访问的详细信息，请参阅[管理对工作区和日志数据的访问](../platform/manage-access.md)

* [HELM client](https://helm.sh/docs/using_helm/) ，为指定的 Kubernetes 群集载入容器的 Azure Monitor 图表。

* 适用于 Linux 的容器化版本的 Log Analytics agent 需要以下代理和防火墙配置信息才能与 Azure Monitor 通信：

    |代理资源|端口 |
    |------|---------|
    |*.ods.opinsights.azure.com |端口 443 |
    |*.oms.opinsights.azure.com |端口 443 |
    |*. dc.services.visualstudio.com |端口 443 |

* 容器化代理需要在群集`cAdvisor secure port: 10250`中`unsecure port :10255`的所有节点上打开 Kubelet 或以收集性能指标。 建议你在 Kubelet `secure port: 10250`的 cAdvisor 上进行配置（如果尚未配置）。

* 容器化代理要求在容器上指定以下环境变量，以便与群集中的 Kubernetes API 服务通信，以收集清单数据`KUBERNETES_SERVICE_HOST`和。 `KUBERNETES_PORT_443_TCP_PORT`

>[!IMPORTANT]
>监视混合 Kubernetes 群集所支持的最低代理版本为 ciprod10182019 或更高版本。

## <a name="enable-monitoring"></a>启用监视

启用混合 Kubernetes 群集的容器 Azure Monitor 包含按顺序执行以下步骤。

1. 配置具有容器 Insights 解决方案的 Log Analytics 工作区。

2. 使用 Log Analytics 工作区为容器 HELM 图启用 Azure Monitor。

### <a name="how-to-add-the-azure-monitor-containers-solution"></a>如何添加 Azure Monitor 容器解决方案

可以通过使用 Azure PowerShell cmdlet `New-AzResourceGroupDeployment`或 Azure CLI 使用提供的 Azure 资源管理器模板来部署解决方案。

如果不熟悉使用模板部署资源的概念，请参阅：

* [使用 Resource Manager 模板和 Azure PowerShell 部署资源](../../azure-resource-manager/templates/deploy-powershell.md)

* [使用资源管理器模板和 Azure CLI 部署资源](../../azure-resource-manager/templates/deploy-cli.md)

如果选择使用 Azure CLI，首先需要在本地安装和使用 CLI。 必须运行 Azure CLI 2.0.59 或更高版本。 若要确定版本，请运行 `az --version`。 如果需要安装或升级 Azure CLI，请参阅[安装 Azure CLI](https://docs.microsoft.com/cli/azure/install-azure-cli)。

此方法包含两个 JSON 模板。 一个模板指定用于启用监视的配置，另一个模板包含参数值，通过配置这些参数值可指定：

- **workspaceResourceId** -Log Analytics 工作区的完整资源 ID。
- **workspaceRegion** -创建工作区的区域，在 Azure 门户中查看时也称为工作区属性中的**位置**。

若要首先标识`workspaceResourceId` **containerSolutionParams**文件中参数值所需的 Log Analytics 工作区的完整资源 ID，请执行以下步骤，然后运行 PowerShell cmdlet 或 Azure CLI 命令添加解决方案。

1. 使用以下命令列出你有权访问的所有订阅：

    ```azurecli
    az account list --all -o table
    ```

    输出如下所示：

    ```azurecli
    Name                                  CloudName    SubscriptionId                        State    IsDefault
    ------------------------------------  -----------  ------------------------------------  -------  -----------
    Microsoft Azure                       AzureCloud   68627f8c-91fO-4905-z48q-b032a81f8vy0  Enabled  True
    ```

    复制 **SubscriptionId** 的值。

2. 使用以下命令切换到托管 Log Analytics 工作区的订阅：

    ```azurecli
    az account set -s <subscriptionId of the workspace>
    ```

3. 以下示例以默认 JSON 格式显示订阅中的工作区列表。

    ```azurecli
    az resource list --resource-type Microsoft.OperationalInsights/workspaces -o json
    ```

    在输出中找到工作区名称，然后将该 Log Analytics 工作区的完整资源 ID 复制到字段**id**下。

4. 将以下 JSON 语法复制并粘贴到文件中：

    ```json
    {
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "workspaceResourceId": {
            "type": "string",
            "metadata": {
                "description": "Azure Monitor Log Analytics Workspace Resource ID"
            }
        },
        "workspaceRegion": {
            "type": "string",
            "metadata": {
                "description": "Azure Monitor Log Analytics Workspace region"
            }
        }
    },
    "resources": [
        {
            "type": "Microsoft.Resources/deployments",
            "name": "[Concat('ContainerInsights', '-',  uniqueString(parameters('workspaceResourceId')))]",
            "apiVersion": "2017-05-10",
            "subscriptionId": "[split(parameters('workspaceResourceId'),'/')[2]]",
            "resourceGroup": "[split(parameters('workspaceResourceId'),'/')[4]]",
            "properties": {
                "mode": "Incremental",
                "template": {
                    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
                    "contentVersion": "1.0.0.0",
                    "parameters": {},
                    "variables": {},
                    "resources": [
                        {
                            "apiVersion": "2015-11-01-preview",
                            "type": "Microsoft.OperationsManagement/solutions",
                            "location": "[parameters('workspaceRegion')]",
                            "name": "[Concat('ContainerInsights', '(', split(parameters('workspaceResourceId'),'/')[8], ')')]",
                            "properties": {
                                "workspaceResourceId": "[parameters('workspaceResourceId')]"
                            },
                            "plan": {
                                "name": "[Concat('ContainerInsights', '(', split(parameters('workspaceResourceId'),'/')[8], ')')]",
                                "product": "[Concat('OMSGallery/', 'ContainerInsights')]",
                                "promotionCode": "",
                                "publisher": "Microsoft"
                            }
                        }
                    ]
                },
                "parameters": {}
            }
         }
      ]
   }
    ```

5. 将此文件作为 containerSolution 保存到本地文件夹。

6. 将以下 JSON 语法粘贴到文件中：

    ```json
    {
      "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
      "contentVersion": "1.0.0.0",
      "parameters": {
        "workspaceResourceId": {
          "value": "<workspaceResourceId>"
      },
      "workspaceRegion": {
        "value": "<workspaceRegion>"
      }
     }
    }
    ```

7. 使用在步骤3中复制的值编辑**workspaceResourceId**的值; 对于**workspaceRegion** ，请在运行 Azure CLI 命令[az monitor log analytics 工作区显示](https://docs.microsoft.com/cli/azure/monitor/log-analytics/workspace?view=azure-cli-latest#az-monitor-log-analytics-workspace-list)后复制**区域**值。

8. 将此文件作为 containerSolutionParams 保存到本地文件夹。

9. 已做好部署此模板的准备。

   * 若要使用 Azure PowerShell 进行部署，请在包含模板的文件夹中使用以下命令：

       ```powershell
       # configure and login to the cloud of log analytics workspace.Specify the corresponding cloud environment of your workspace to below command.
       Connect-AzureRmAccount -Environment <AzureCloud | AzureChinaCloud | AzureUSGovernment>
       ```

       ```powershell
       # set the context of the subscription of Log Analytics workspace
       Set-AzureRmContext -SubscriptionId <subscription Id of log analytics workspace>
       ```

       ```powershell
       # execute deployment command to add container insights solution to the specified Log Analytics workspace
       New-AzureRmResourceGroupDeployment -Name OnboardCluster -ResourceGroupName <resource group of log analytics workspace> -TemplateFile .\containerSolution.json -TemplateParameterFile .\containerSolutionParams.json
       ```

       配置更改可能需要几分钟才能完成。 完成后，系统会显示包含结果的消息，如下所示：

       ```powershell
       provisioningState       : Succeeded
       ```

   * 若要使用 Azure CLI 进行部署，请运行下列命令：

       ```azurecli
       az login
       az account set --name <AzureCloud | AzureChinaCloud | AzureUSGovernment>
       az login
       az account set --subscription "Subscription Name"
       # execute deployment command to add container insights solution to the specified Log Analytics workspace
       az deployment group create --resource-group <resource group of log analytics workspace> --name <deployment name> --template-file  ./containerSolution.json --parameters @./containerSolutionParams.json
       ```

       配置更改可能需要几分钟才能完成。 完成后，系统会显示包含结果的消息，如下所示：

       ```azurecli
       provisioningState       : Succeeded
       ```

       启用监视后，可能需要约 15 分钟才能查看群集的运行状况指标。

## <a name="install-the-chart"></a>安装图表

>[!NOTE]
>以下命令仅适用于 Helm 版本2。 使用`--name`参数不适用于 Helm 版本3。

若要启用 HELM 图表，请执行以下操作：

1. 通过运行以下命令，将 Azure 图表存储库添加到你的本地列表：

    ```
    helm repo add incubator https://kubernetes-charts-incubator.storage.googleapis.com/
    ````

2. 通过运行以下命令来安装图表：

    ```
    $ helm install --name myrelease-1 \
    --set omsagent.secret.wsid=<your_workspace_id>,omsagent.secret.key=<your_workspace_key>,omsagent.env.clusterName=<my_prod_cluster> incubator/azuremonitor-containers
    ```

    如果 Log Analytics 工作区位于 Azure 中国区，请运行以下命令：

    ```
    $ helm install --name myrelease-1 \
     --set omsagent.domain=opinsights.azure.cn,omsagent.secret.wsid=<your_workspace_id>,omsagent.secret.key=<your_workspace_key>,omsagent.env.clusterName=<your_cluster_name> incubator/azuremonitor-containers
    ```

    如果 Log Analytics 工作区位于 Azure 美国政府版中，请运行以下命令：

    ```
    $ helm install --name myrelease-1 \
    --set omsagent.domain=opinsights.azure.us,omsagent.secret.wsid=<your_workspace_id>,omsagent.secret.key=<your_workspace_key>,omsagent.env.clusterName=<your_cluster_name> incubator/azuremonitor-containers
    ```

### <a name="enable-the-helm-chart-using-the-api-model"></a>使用 API 模型启用 Helm 图表

你可以在 AKS 引擎群集规范 json 文件中指定加载项，也称为 API 模型。 在此加载项中，提供在其中存储`WorkspaceGUID`所`WorkspaceKey`收集的监视数据的 Log Analytics 工作区的 base64 编码版本。

此示例[monitoring_existing_workspace_id_and_key kubernetes](https://github.com/Azure/aks-engine/blob/master/examples/addons/container-monitoring/kubernetes-container-monitoring_existing_workspace_id_and_key.json)中提供了 Azure Stack 中心群集支持的 API 定义。 具体而言，请在**kubernetesConfig**中查找**加载项**属性：

```json
"orchestratorType": "Kubernetes",
       "kubernetesConfig": {
         "addons": [
           {
             "name": "container-monitoring",
             "enabled": true,
             "config": {
               "workspaceGuid": "<Azure Log Analytics Workspace Guid in Base-64 encoded>",
               "workspaceKey": "<Azure Log Analytics Workspace Key in Base-64 encoded>"
             }
           }
         ]
       }
```

## <a name="configure-agent-data-collection"></a>配置代理数据收集

起始具有图表版本1.0.0，则从 ConfigMap 控制代理数据收集设置。 请参阅[此处](container-insights-agent-config.md)的有关代理数据收集设置的文档。

成功部署图表后，可以在 Azure 门户的容器 Azure Monitor 中查看混合 Kubernetes 群集的数据。  

>[!NOTE]
>从代理到 Azure Log Analytics 工作区中的提交，引入延迟时间大约为5到10分钟。 群集的状态显示值 "**无数据**" 或 "**未知**"，直到所有必需的监视数据在 Azure Monitor 中可用。

## <a name="troubleshooting"></a>故障排除

如果尝试为混合 Kubernetes 群集启用监视时遇到错误，请将 PowerShell 脚本复制[TroubleshootError_nonAzureK8s ps1](https://raw.githubusercontent.com/microsoft/OMS-docker/ci_feature/Troubleshoot/TroubleshootError_nonAzureK8s.ps1) ，并将其保存到计算机上的文件夹中。 提供此脚本是为了帮助检测和解决遇到的问题。 用于检测和尝试更正的问题如下所示：

* 指定的 Log Analytics 工作区有效
* Log Analytics 工作区配置有容器解决方案的 Azure Monitor。 如果不是，请配置工作区。
* OmsAgent replicaset pod 正在运行
* OmsAgent daemonset pod 正在运行
* OmsAgent 运行状况服务正在运行
* 容器化代理上配置的 Log Analytics 工作区 ID 和密钥与用于配置见解的工作区匹配。
* 验证所有 Linux 辅助角色节点都`kubernetes.io/role=agent`具有用于计划 rs pod 的标签。 如果它不存在，请添加它。
* 验证`cAdvisor secure port:10250`或`unsecure port: 10255`在群集中的所有节点上打开。

若要使用 Azure PowerShell 执行，请在包含脚本的文件夹中使用以下命令：

```powershell
.\TroubleshootError_nonAzureK8s.ps1 - azureLogAnalyticsWorkspaceResourceId </subscriptions/<subscriptionId>/resourceGroups/<resourcegroupName>/providers/Microsoft.OperationalInsights/workspaces/<workspaceName> -kubeConfig <kubeConfigFile> -clusterContextInKubeconfig <clusterContext>
```

## <a name="next-steps"></a>后续步骤

启用监视功能以收集混合 Kubernetes 群集的运行状况和资源利用率，并了解[如何使用](container-insights-analyze.md)容器 Azure Monitor。
