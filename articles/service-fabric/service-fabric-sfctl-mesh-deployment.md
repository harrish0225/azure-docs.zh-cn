---
title: Azure Service Fabric sfctl 网格部署
description: 了解 sfctl，Azure Service Fabric 命令行界面。 包含用于创建 Service Fabric 网格资源的命令的列表。
author: jeffj6123
ms.topic: reference
ms.date: 1/16/2020
ms.author: jejarry
ms.openlocfilehash: 108389407221779ed20e81310f084b7b5c23b8c7
ms.sourcegitcommit: 849bb1729b89d075eed579aa36395bf4d29f3bd9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/28/2020
ms.locfileid: "76906023"
---
# <a name="sfctl-mesh-deployment"></a>sfctl mesh deployment
创建 Service Fabric 网格资源。

## <a name="commands"></a>命令

|Command|说明|
| --- | --- |
| create | 创建 Service Fabric 网格资源的部署。 |

## <a name="sfctl-mesh-deployment-create"></a>sfctl mesh deployment create
创建 Service Fabric 网格资源的部署。

### <a name="arguments"></a>参数

|参数|说明|
| --- | --- |
| --input-yaml-files [必需] | 逗号分隔的所有 yaml 文件的相对或绝对文件路径，或包含 yaml 文件的目录的相对或绝对路径（递归）。 |
| --parameters | 指向 yaml 文件的相对路径或绝对路径，或包含需要重写的参数的 json 对象。 |

### <a name="global-arguments"></a>全局参数

|参数|说明|
| --- | --- |
| --debug | 提高日志记录详细程度以显示所有调试日志。 |
| --help -h | 显示此帮助消息并退出。 |
| --output -o | 输出格式。  允许的值\: json、jsonc、table、tsv。  默认值\: json。 |
| --query | JMESPath 查询字符串。 有关详细信息和示例，请参阅 http\://jmespath.org/。 |
| --verbose | 提高日志记录详细程度。 使用 --debug 获取完整的调试日志。 |

### <a name="examples"></a>示例

通过覆盖 yaml 文件中提到的参数将所有资源整合并部署到群集中
``` 
sfctl mesh deployment create --input-yaml-files ./app.yaml,./network.yaml --parameters  
./param.yaml    
```

通过覆盖 yaml 文件中提到的参数将某个目录中的所有资源整合并部署到群集中

``` 
sfctl mesh deployment create --input-yaml-files ./resources --parameters ./param.yaml
```

通过重写作为 json 对象直接传递的参数，将目录中的所有资源合并并部署到群集中
``` 
sfctl mesh deployment create --input-yaml-files ./resources --parameters "{ 'my_param' :    
{'value' : 'my_value'} }"   
```

## <a name="next-steps"></a>后续步骤
- [设置](service-fabric-cli.md)Service Fabric CLI。
- 了解如何通过[示例脚本](/azure/service-fabric/scripts/sfctl-upgrade-application)使用 Service Fabric CLI。
