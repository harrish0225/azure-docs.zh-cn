---
title: 数据加密疑难解答-Azure Database for MySQL
description: 了解如何排查 Azure Database for MySQL 中的数据加密
author: kummanish
ms.author: manishku
ms.service: mysql
ms.topic: conceptual
ms.date: 02/13/2020
ms.openlocfilehash: 42956d115590fd322d2851fd546c505a76a851fa
ms.sourcegitcommit: 849bb1729b89d075eed579aa36395bf4d29f3bd9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/28/2020
ms.locfileid: "79297034"
---
# <a name="troubleshoot-data-encryption-in-azure-database-for-mysql"></a>Azure Database for MySQL 中的数据加密的疑难解答

本文介绍如何使用客户管理的密钥来识别和解决 Azure Database for MySQL 中使用数据加密配置时可能出现的常见问题。

## <a name="introduction"></a>简介

在 Azure Key Vault 中将数据加密配置为使用客户托管的密钥时，服务器需要持续访问密钥。 如果服务器在 Azure Key Vault 中失去了对客户管理的密钥的访问权限，它将拒绝所有连接，返回相应的错误消息，并将其状态更改为在 Azure 门户中***不可访问***。

如果不再需要无法访问的 Azure Database for MySQL 服务器，可以将其删除以停止产生成本。 在对密钥保管库的访问权限恢复并且服务器可用之前，不允许服务器上的其他操作。 当使用客户管理的密钥加密时，也不能`Yes`将 "数据加密" 选项`No`从（客户管理）更改为（服务管理）。 您必须手动重新验证密钥，然后才能再次访问服务器。 此操作是保护数据免遭未经授权的访问所必需的，而对客户管理的密钥的权限被吊销。

## <a name="common-errors-that-cause-the-server-to-become-inaccessible"></a>导致服务器无法访问的常见错误

以下错误配置会导致使用 Azure Key Vault 密钥的数据加密的大多数问题：

- Key vault 不可用或不存在：
  - 密钥保管库遭意外删除。
  - 间歇性网络错误导致密钥保管库不可用。

- 你没有访问密钥保管库的权限，或者该密钥不存在：
  - 密钥已过期或意外删除或禁用。
  - 意外删除了 Azure Database for MySQL 实例的托管标识。
  - Azure Database for MySQL 实例的托管标识没有足够的密钥权限。 例如，权限不包括 Get、Wrap 和解包。
  - 对 Azure Database for MySQL 实例的托管标识权限已被吊销或删除。

## <a name="identify-and-resolve-common-errors"></a>发现和解决常见错误

### <a name="errors-on-the-key-vault"></a>密钥保管库上的错误

#### <a name="disabled-key-vault"></a>禁用的密钥保管库

- `AzureKeyVaultKeyDisabledMessage`
- **说明**：无法在服务器上完成操作，因为已禁用 Azure Key Vault 项。

#### <a name="missing-key-vault-permissions"></a>缺少密钥保管库权限

- `AzureKeyVaultMissingPermissionsMessage`
- **说明**：服务器没有必需的 Get、Wrap 和解包权限，无法 Azure Key Vault。 向 ID 为的服务主体授予任何缺少的权限。

### <a name="mitigation"></a>缓解

- 确认密钥保管库中存在客户管理的密钥。
- 标识密钥保管库，然后在 Azure 门户中转到密钥保管库。
- 确保密钥 URI 标识存在的键。

## <a name="next-steps"></a>后续步骤

[使用 Azure 门户在 Azure Database for MySQL 上使用客户管理的密钥设置数据加密](howto-data-encryption-portal.md)
