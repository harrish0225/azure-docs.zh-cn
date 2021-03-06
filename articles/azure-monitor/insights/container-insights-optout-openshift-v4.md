---
title: 如何停止监视 Azure 和 Red Hat OpenShift v4 群集 |Microsoft Docs
description: 本文介绍如何通过 Azure Monitor 容器停止监视 Azure Red Hat OpenShift 和 Red Hat OpenShift 版本4群集。
ms.topic: conceptual
ms.date: 04/24/2020
ms.openlocfilehash: 768c4db8d72778b555a4f343cf2e23b8fa861991
ms.sourcegitcommit: 849bb1729b89d075eed579aa36395bf4d29f3bd9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/28/2020
ms.locfileid: "82196434"
---
# <a name="how-to-stop-monitoring-your-azure-and-red-hat-openshift-v4-cluster"></a>如何停止监视 Azure 和 Red Hat OpenShift v4 群集

启用对 Azure Red Hat OpenShift 和 Red Hat OpenShift 版本4.x 群集的监视后，如果你决定不再想要对其进行监视，则可以使用容器 Azure Monitor 停止监视群集。 本文介绍如何实现此目的。  

## <a name="how-to-stop-monitoring-using-helm"></a>如何使用 Helm 停止监视

1. 若要首先确定群集上安装的 helm 图表版本的 Azure Monitor，请运行以下 helm 命令。

    ```
    helm list
    ```

    输出如下所示：

    ```
    NAME                            NAMESPACE       REVISION        UPDATED                                 STATUS          CHART                           APP VERSION
    azmon-containers-release-1      default         3               2020-04-21 15:27:24.1201959 -0700 PDT   deployed        azuremonitor-containers-2.7.0   7.0.0-1
    ```

    *azmon-版本-1*表示容器 Azure Monitor 的 helm 图表版本。

2. 若要删除图表版本，请运行以下 helm 命令。

    `helm delete <releaseName>`

    示例：

    `helm delete azmon-containers-release-1`

    这将从群集中删除发布。 可以通过运行以下`helm list`命令进行验证：

    ```
    NAME                            NAMESPACE       REVISION        UPDATED                                 STATUS          CHART                           APP VERSION
    ```

配置更改可能需要几分钟才能完成。 因为 Helm 会在你删除发布后对其进行跟踪，因此你可以审核群集的历史记录，甚至还可以使用`helm rollback`撤消删除发布。

## <a name="next-steps"></a>后续步骤

如果 Log Analytics 工作区仅用于支持监视群集，并且不再需要它，则必须手动将其删除。 如果你不熟悉如何删除工作区，请参阅[删除 Azure Log Analytics 工作区](../../log-analytics/log-analytics-manage-del-workspace.md)。
