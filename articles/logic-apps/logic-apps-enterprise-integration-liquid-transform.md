---
title: 使用 Liquid 转换转换 JSON 数据
description: 使用逻辑应用和 Liquid 模板创建用于高级 JSON 转换的转换或映射
services: logic-apps
ms.suite: integration
author: divyaswarnkar
ms.author: divswa
ms.reviewer: estfan, logicappspm
ms.topic: article
ms.date: 04/01/2020
ms.openlocfilehash: d2598dfe9d7972dcb764abf4a1239613a1e8417a
ms.sourcegitcommit: 849bb1729b89d075eed579aa36395bf4d29f3bd9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/28/2020
ms.locfileid: "80879167"
---
# <a name="perform-advanced-json-transformations-with-liquid-templates-in-azure-logic-apps"></a>在 Azure 逻辑应用中使用 Liquid 模板执行高级 JSON 转换

可以在逻辑应用中通过**撰写**或**分析 JSON** 等本机数据操作完成基本 JSON 转换。 若要执行高级 JSON 转换，可以使用 [Liquid](https://shopify.github.io/liquid/)（一种用于灵活 Web 应用的开源模板语言）创建模板或映射。 Liquid 模板定义如何转换 JSON 输出并支持更复杂的 JSON 转换，例如迭代、控制流、变量，等等。

在逻辑应用中执行 Liquid 转换之前，必须先使用 Liquid 模板定义 JSON 到 JSON 的映射，并将该映射存储在集成帐户中。 本文展示了如何创建并使用此 Liquid 模板或映射。

## <a name="prerequisites"></a>必备条件

* Azure 订阅。 如果没有订阅，可以[注册免费的 Azure 帐户](https://azure.microsoft.com/free/)。

* 有关[如何创建逻辑应用的](../logic-apps/quickstart-create-first-logic-app-workflow.md)基本知识

* 基本[集成帐户](../logic-apps/logic-apps-enterprise-integration-create-integration-account.md)

* 关于[液体模板语言](https://shopify.github.io/liquid/)的基本知识

## <a name="create-liquid-template-or-map-for-your-integration-account"></a>为集成帐户创建 Liquid 模板或映射

1. 对于此示例，请创建此步骤中所述的示例 Liquid 模板。 在液体模板中，可以使用[液体过滤器](https://shopify.github.io/liquid/basics/introduction/#filters)，使用[DotLiquid](https://github.com/dotliquid/dotliquid)和 c # 命名约定。

   > [!NOTE]
   > 请确保筛选器名称在模板中使用*句子大小写*。 否则，筛选器将不起作用。 此外，映射具有[文件大小限制](../logic-apps/logic-apps-limits-and-config.md#artifact-capacity-limits)。

   ```json
   {%- assign deviceList = content.devices | Split: ', ' -%}
   
   {
      "fullName": "{{content.firstName | Append: ' ' | Append: content.lastName}}",
      "firstNameUpperCase": "{{content.firstName | Upcase}}",
      "phoneAreaCode": "{{content.phone | Slice: 1, 3}}",
      "devices" : [
         {%- for device in deviceList -%}
            {%- if forloop.Last == true -%}
            "{{device}}"
            {%- else -%}
            "{{device}}",
            {%- endif -%}
        {%- endfor -%}
        ]
   }
   ```

1. 在[Azure 门户](https://portal.azure.com)的 "Azure 搜索" 框中，输入`integration accounts`，然后选择 "**集成帐户**"。

   ![查找 "集成帐户"](./media/logic-apps-enterprise-integration-liquid-transform/find-integration-accounts.png)

1. 找到并选择你的集成帐户。

   ![选择“集成帐户”](./media/logic-apps-enterprise-integration-liquid-transform/select-integration-account.png)

1. 在**概述**窗格中的 "**组件**" 下，选择 "**映射**"。

    ![选择 "地图" 磁贴](./media/logic-apps-enterprise-integration-liquid-transform/select-maps-tile.png)

1. 在 "**地图**" 窗格中，选择 "**添加**"，并为地图提供以下详细信息：

   | 属性 | 值 | 说明 | 
   |----------|-------|-------------|
   | **名称** | `JsonToJsonTemplate` | 映射的名称，在此示例中为“JsonToJsonTemplate” | 
   | **映射类型** | **liquid** | 你的映射的类型。 对于 JSON 到 JSON 转换，必须选择“liquid”****。 | 
   | **将** | `SimpleJsonToJsonTemplate.liquid` | 用于转换的现有 Liquid 模板或映射文件，在此示例中为“SimpleJsonToJsonTemplate.liquid”。 若要查找此文件，可使用文件选取器。 有关地图大小限制，请参阅[限制和配置](../logic-apps/logic-apps-limits-and-config.md#artifact-capacity-limits)。 |
   ||| 

   ![添加液体模板](./media/logic-apps-enterprise-integration-liquid-transform/add-liquid-template.png)
    
## <a name="add-the-liquid-action-for-json-transformation"></a>为 JSON 转换添加 Liquid 操作

1. 在 Azure 门户中，执行以下步骤[创建空的逻辑应用](../logic-apps/quickstart-create-first-logic-app-workflow.md)。

1. 在逻辑应用设计器中，向逻辑应用中添加[请求触发器](../connectors/connectors-native-reqres.md#add-request)。

1. 在触发器下，选择“新建步骤”。**** 在搜索框中，输入`liquid`作为筛选器，然后选择此操作：**将 JSON 转换为 json-液体**

   ![查找并选择 Liquid 操作](./media/logic-apps-enterprise-integration-liquid-transform/search-action-liquid.png)

1. 打开**地图**列表，并选择你的液体模板，此示例中为 "JsonToJsonTemplate"。

   ![选择映射](./media/logic-apps-enterprise-integration-liquid-transform/select-map.png)

   如果映射列表为空，则逻辑应用很可能未链接到集成帐户。 
   要将逻辑应用链接到具有 Liquid 模板或映射的集成帐户，请执行以下步骤：

   1. 在逻辑应用菜单中选择“工作流设置”****。

   1. 从 "**选择集成帐户**" 列表中，选择你的集成帐户，然后选择 "**保存**"。

      ![将逻辑应用链接到集成帐户](./media/logic-apps-enterprise-integration-liquid-transform/link-integration-account.png)

1. 现在，向此操作添加**Content**属性。 打开 "**添加新参数**" 列表，然后选择 "**内容**"。

   ![向操作添加 "Content" 属性](./media/logic-apps-enterprise-integration-liquid-transform/add-content-property-to-action.png)

1. 若要设置**content**属性值，请在 "**内容**" 框中单击，以便显示 "动态内容" 列表。 选择 "**正文**" 标记，它表示从触发器输出的正文内容。

   ![选择 "Content" 属性值的 "正文" 标记](./media/logic-apps-enterprise-integration-liquid-transform/select-body.png)

   完成后，此操作如以下示例所示：

   ![已完成 "将 JSON 转换为 JSON" 操作](./media/logic-apps-enterprise-integration-liquid-transform/finished-transform-action.png)

## <a name="test-your-logic-app"></a>测试逻辑应用

通过 [Postman](https://www.getpostman.com/postman) 或类似工具，将 JSON 输入发布到逻辑应用。 逻辑应用转换后的 JSON 输出如此示例中所示：
  
![示例输出](./media/logic-apps-enterprise-integration-liquid-transform/example-output-jsontojson.png)

## <a name="more-liquid-action-examples"></a>更多 Liquid 操作示例
Liquid 并非仅可用于 JSON 转换。 下面列出了使用 Liquid 的其他可用转换操作。

* 将 JSON 转换为文本
  
  下面是用于此示例的 Liquid 模板：
   
   ``` json
   {{content.firstName | Append: ' ' | Append: content.lastName}}
   ```
   下面是示例输入和输出：
  
   ![将 JSON 输出为文本的示例](./media/logic-apps-enterprise-integration-liquid-transform/example-output-jsontotext.png)

* 将 XML 转换为 JSON
  
  下面是用于此示例的 Liquid 模板：
   
   ``` json
   [{% JSONArrayFor item in content -%}
        {{item}}
    {% endJSONArrayFor -%}]
   ```
   下面是示例输入和输出：

   ![将 XML 输出为 JSON 的示例](./media/logic-apps-enterprise-integration-liquid-transform/example-output-xmltojson.png)

* 将 XML 转换为文本
  
  下面是用于此示例的 Liquid 模板：

   ``` json
   {{content.firstName | Append: ' ' | Append: content.lastName}}
   ```

   下面是示例输入和输出：

   ![将 XML 输出为文本的示例](./media/logic-apps-enterprise-integration-liquid-transform/example-output-xmltotext.png)

## <a name="next-steps"></a>后续步骤

* [了解有关 Enterprise Integration Pack 的详细信息](../logic-apps/logic-apps-enterprise-integration-overview.md "了解 Enterprise Integration Pack")  
* [详细了解映射](../logic-apps/logic-apps-enterprise-integration-maps.md "了解企业集成地图")  

