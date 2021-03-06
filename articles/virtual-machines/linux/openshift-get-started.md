---
title: Azure 中的 OpenShift 概述
description: Azure 中的 OpenShift 概述。
author: haroldwongms
manager: mdotson
ms.service: virtual-machines-linux
ms.subservice: workloads
ms.topic: article
ms.workload: infrastructure
ms.date: 05/7/2019
ms.author: haroldw
ms.openlocfilehash: c338e9a6a793d1c2d0557d70242996175d5a85ed
ms.sourcegitcommit: 849bb1729b89d075eed579aa36395bf4d29f3bd9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/28/2020
ms.locfileid: "81759449"
---
# <a name="openshift-in-azure"></a>Azure 中的 OpenShift

OpenShift 是一个开放可扩展的容器应用程序平台，面向企业提供 Docker 和 Kubernetes。  

OpenShift 包括用于容器业务流程和管理的 Kubernetes 容器。 它增加了以开发人员为中心的工具和以操作为中心的工具，这些工具可实现：

- 快速应用程序开发。
- 轻松部署和缩放。
- 针对团队和应用程序的长期生命期维护。

有多个 OpenShift 版本可用。  在这些版本中，目前只有两个版本可供客户在 Azure 中部署： OpenShift 容器平台和 OKD （以前称为 OpenShift 源）。

## <a name="azure-red-hat-openshift"></a>Azure Red Hat OpenShift

Microsoft Azure Red Hat OpenShift 是在 Azure 中运行的 OpenShift 的完全托管的服务。 此服务由 Microsoft 和 Red Hat 共同管理并提供支持。 有关更多详细信息，请参阅[Azure Red Hat OpenShift 服务](https://docs.microsoft.com/azure/openshift/)文档。

## <a name="openshift-container-platform"></a>OpenShift 容器平台

容器平台 是 Red Hat 支持的企业就绪[商业版本](https://www.openshift.com)。 使用此版本时，客户需购买 OpenShift 容器平台的必要权利，并负责安装和管理整个基础结构。

由于客户“拥有”整个平台，他们可在本地数据中心或公有云（例如 Azure）位置中进行安装。

## <a name="okd"></a>OKD

OKD 是社区支持的 OpenShift 的[开源](https://www.okd.io/)上游项目。 可在 CentOS 或 Red Hat Enterprise Linux (RHEL) 上安装 OKD。

## <a name="next-steps"></a>后续步骤

- [在 Azure 中配置 OpenShift 的常见先决条件](./openshift-container-platform-3x-prerequisites.md)
- [在 Azure 中部署 OpenShift 容器平台](./openshift-container-platform-3x.md)
- [部署 OpenShift 容器平台自行管理的 Marketplace 产品/服务](./openshift-container-platform-3x-marketplace-self-managed.md)
- [在 Azure Stack 中部署 OpenShift](./openshift-azure-stack.md)
- [部署后任务](./openshift-container-platform-3x-post-deployment.md)
- [OpenShift 部署故障排除](./openshift-container-platform-3x-troubleshooting.md)
