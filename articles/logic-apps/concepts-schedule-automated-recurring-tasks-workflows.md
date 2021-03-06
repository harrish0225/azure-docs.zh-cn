---
title: 在 Azure 逻辑应用中计划重复性任务和工作流
description: 有关如何使用 Azure 逻辑应用计划重复性自动任务、过程和工作流的概述
services: logic-apps
ms.suite: integration
ms.reviewer: deli, jonfan, logicappspm
ms.topic: conceptual
ms.date: 03/25/2020
ms.openlocfilehash: 6d00c7d7cc88427a3500b28891ec70bb8a4bbb43
ms.sourcegitcommit: ac4a365a6c6ffa6b6a5fbca1b8f17fde87b4c05e
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/10/2020
ms.locfileid: "83005201"
---
# <a name="schedule-and-run-recurring-automated-tasks-processes-and-workflows-with-azure-logic-apps"></a>使用 Azure 逻辑应用计划和运行重复性自动任务、过程与工作流

逻辑应用可帮助你创建自动化的重复性任务与过程并按计划运行。 创建使用内置重复触发器或滑动窗口触发器（计划类型的触发器）启动的逻辑应用工作流，以后可以立即运行或者按重复间隔运行任务。 可以调用 Azure 内部和外部的服务（例如 HTTP 或 HTTPS 终结点）、将消息发送到 Azure 存储和 Azure 服务总线等 Azure 服务，或者将文件上传到文件共享。 使用重复触发器，还可以设置复杂的计划，以及运行任务的详细重复周期。 若要详细了解内置计划触发器和操作，请参阅[计划触发器](#schedule-triggers)和[计划操作](#schedule-actions)。 

> [!TIP]
> 无需为每个计划的作业单独创建逻辑应用，也无需占用[每个区域和订阅的工作流限制配额](../logic-apps/logic-apps-limits-and-config.md#definition-limits)，即可计划和运行重复性工作负荷。 可以使用 [Azure 快速入门模板：逻辑应用作业计划程序](https://github.com/Azure/azure-quickstart-templates/tree/master/301-logicapps-jobscheduler/)创建的逻辑应用模式。
>
> 逻辑应用作业计划程序模板会创建一个调用 TimerJob 逻辑应用的 CreateTimerJob 逻辑应用。 然后，你可以通过发出 HTTP 请求并传递一个计划作为请求的输入，以 API 的形式调用 CreateTimerJob 逻辑应用。 每次调用 CreateTimerJob 逻辑应用时，都会调用 TimerJob 逻辑应用，这会创建一个根据指定计划持续运行的、或者一直运行到达到指定限制的新 TimerJob 实例。 这样，便可以运行任意数目的 TimerJob 实例，而无需担心工作流限制，因为实例不是单独的逻辑应用工作流定义或资源。

此列表显示了可以使用计划内置触发器运行的一些示例任务：

* 获取内部数据，例如每天运行某个 SQL 存储过程。

* 获取外部数据，例如每隔 15 分钟从 NOAA 提取天气报告。

* 发送报告数据，例如通过电子邮件发送过去一周内超过特定金额的所有订单的摘要。

* 处理数据，例如在每个工作日的非高峰时间压缩当日的已上传图像。

* 清除数据，例如删除超过三个月的所有推文。

* 存档数据，例如在未来 9 个月的每天凌晨 1:00 将发票推送到备份服务。

在下一操作运行之前，还可以使用计划内置操作来暂停工作流，例如：

* 等到工作日通过电子邮件发送状态更新。

* 在恢复和检索结果前，延迟工作流直到 HTTP 调用有时间完成。

本文介绍计划内置触发器和操作的功能。

<a name="schedule-triggers"></a>

## <a name="schedule-triggers"></a>计划触发器

你可以使用 "定期触发器" 或 "滑动窗口" 触发器（不与任何特定服务或系统关联）启动逻辑应用工作流。 这些触发器根据你指定的重复周期启动并运行工作流，你可以在其中选择时间间隔和频率，如秒数、分钟数、小时数、天数、周数或月数。 还可以设置开始日期和时间以及时区。 每当某个触发器激发时，逻辑应用服务将为你的逻辑应用创建并运行一个新的工作流实例。

下面是这些触发器之间的差异：

* **重复周期**：根据指定的计划按固定的时间间隔运行工作流。 如果错过了重复周期，则重复触发器不会处理错过的重复周期，而是根据下一个计划的时间间隔重启重复周期。 可以指定开始日期和时间以及时区。 如果选择“天”，则可以指定该日期的小时，以及该小时的分钟，例如，每日 2:30。 如果选择“周”，则还可以选择星期，例如“星期三”和“星期六”。 有关详细信息，请参阅[使用重复触发器创建、计划和运行重复性任务与工作流](../connectors/connectors-native-recurrence.md)。

* **滑动窗口**：按定期时间间隔运行工作流来处理连续区块中的数据。 如果错过了重复周期，则滑动窗口触发器将会返回并处理错过的重复周期。 可以指定开始日期和时间以及时区，并可以指定一个持续时间来延迟工作流中的每个重复周期。 此触发器不支持高级计划，例如，一天中的特定小时数、小时数以及一周中的几天。 有关详细信息，请参阅[使用滑动窗口触发器创建、计划和运行重复性任务与工作流](../connectors/connectors-native-sliding-window.md)。

<a name="schedule-actions"></a>

## <a name="schedule-actions"></a>计划操作

在逻辑应用工作流中指定任何操作后，可以使用“延迟”和“延迟到”操作来使工作流等到下一个操作运行为止。

* **延迟**：按指定的时间单位数（例如秒数、分钟数、小时数、天数、周数或月数）等待运行下一操作。 有关详细信息，请参阅[延迟工作流中的下一操作](../connectors/connectors-native-delay.md)。

* **延迟截止时间**：在指定的日期和时间之前等待运行下一操作。 有关详细信息，请参阅[延迟工作流中的下一操作](../connectors/connectors-native-delay.md)。

## <a name="patterns-for-start-date-and-time"></a>开始日期和时间的模式

<a name="start-time"></a>

下面这些模式将演示如何使用开始日期和时间控制重复计划，以及逻辑应用服务如何运行这些重复计划：

| 开始时间 | 不按计划循环 | 按计划重复（仅适用于重复触发器） |
|------------|-----------------------------|----------------------------------------------------|
| {无} | 即时运行第一个工作负荷。 <p>基于上次运行时间运行将来的工作负荷。 | 即时运行第一个工作负荷。 <p>基于指定的计划运行将来的工作负荷。 |
| 开始时间在过去 | **重复**触发器：基于指定的开始时间计算运行时间，并丢弃过去的运行时间。 在下一个将来运行时间运行第一个工作负荷。 <p>基于上次运行时间的计算结果运行将来的工作负荷。 <p><p>**滑动窗口**触发器：基于指定的开始时间计算运行时间，并遵循过去的运行时间。 <p>基于指定的开始时间的计算结果运行将来的工作负荷。 <p><p>有关详细说明，请参阅此表格后面的示例。 | 基于从开始时间计算的计划，在不早于开始时间的时间运行第一个工作负荷。  <p>基于指定的计划运行将来的工作负荷。 <p>**注意：** 如果针对计划指定了定期模式，但没有为该计划指定小时或分钟，则将来的运行时间会分别使用小时或分钟来计算（从首次运行时间开始算起）。 |
| 开始时间在当前或将来 | 在指定的开始时间运行第一个工作负荷。 <p>基于上次运行时间的计算结果运行将来的工作负荷。 | 基于从开始时间计算的计划，在不早于开始时间的时间运行第一个工作负荷。  <p>基于指定的计划运行将来的工作负荷。 <p>**注意：** 如果针对计划指定了定期模式，但没有为该计划指定小时或分钟，则将来的运行时间会分别使用小时或分钟来计算（从首次运行时间开始算起）。 |
||||

> [!IMPORTANT]
> 当重复执行不指定高级计划选项时，将来的重复将基于上次运行时间。
> 这些重复周期的开始时间可能会由于诸如存储调用期间的延迟等因素而发生偏差。 若要确保逻辑应用不会错过循环，尤其是频率为天或更长时，请使用以下选项之一：
> 
> * 提供定期的开始时间。
> 
> * 使用 "**此时** **" 和 "时间"** 属性指定运行定期的时间和时间。
> 
> * 使用[滑动窗口触发器](../connectors/connectors-native-sliding-window.md)，而不是重复触发器。

*指定了过去开始时间和重复周期但未指定计划的示例*

假设当前日期和时间是 2017 年 9 月 8 日下午 1:00。 将开始日期和时间指定为已过去的 2017 年 9 月 7 日下午 2:00，重复周期为每隔两天运行一次。

| 开始时间 | 当前时间 | 定期 | 计划 |
|------------|--------------|------------|----------|
| 2017-09-**07**T14:00:00Z <br>（2017-09-**07** ，晚上2:00） | 2017-09-**08**T13:00:00Z <br>（2017-09-**08** 下午 1:00） | 每隔两天 | {无} |
|||||

对于重复触发器，逻辑应用引擎会基于开始时间计算运行时间，丢弃过去的运行时间，为首次运行使用下一个将来开始时间，并基于上次运行时间计算将来的运行。

下面是此重复周期的大致形式：

| 开始时间 | 首次运行时间 | 将来的运行时间 |
|------------|----------------|------------------|
| 2017-09-**07** 的 2:00 PM | 2017-09-**09** 的 2:00 PM | 2017-09-**11** 的 2:00 PM </br>2017-09-**13** 的 2:00 PM </br>2017-09-**15** 的 2:00 PM </br>依此类推... |
||||

因此，不管在过去的多长期限内指定了开始时间（例如，2017-09-**05** 下午 2:00，或 2017-09-**01** 下午 2:00），首次运行始终使用下一个将来的开始时间。

对于滑动窗口触发器，逻辑应用引擎会基于开始时间计算运行时间，遵循过去的运行时间，为首次运行使用该开始时间，并基于该开始时间计算将来的运行。

下面是此重复周期的大致形式：

| 开始时间 | 首次运行时间 | 将来的运行时间 |
|------------|----------------|------------------|
| 2017-09-**07** 的 2:00 PM | 2017-09-**07** 的 2:00 PM | 2017-09-**09** 的 2:00 PM </br>2017-09-**11** 的 2:00 PM </br>2017-09-**13** 的 2:00 PM </br>2017-09-**15** 的 2:00 PM </br>依此类推... |
||||

因此，不管在过去的多长期限内指定了开始时间（例如，2017-09-**05** 下午 2:00，或 2017-09-**01** 下午 2:00），首次运行始终使用指定的开始时间。

<a name="example-recurrences"></a>

## <a name="example-recurrences"></a>示例重复周期

下面是可为支持上述选项的触发器设置的各种示例重复周期：

| 触发器 | 定期 | 时间间隔 | 频率 | 开始时间 | 在这些日期 | 在这些小时 | 在这些分钟 | 注意 |
|---------|------------|----------|-----------|------------|---------------|----------------|------------------|------|
| 重复 <br>滑动窗口 | 每隔 15 分钟运行（没有开始日期和时间） | 15 | Minute | {无} | {不可用} | {无} | {无} | 此计划立即启动，然后根据上次运行时间计算将来的重复周期。 |
| 重复 <br>滑动窗口 | 每隔 15 分钟运行（有开始日期和时间） | 15 | Minute | *startDate*T*startTime*Z | {不可用} | {无} | {无} | 此计划不会在指定的开始日期和时间之前启动，将根据上次运行时间计算将来的重复周期。** |
| 重复 <br>滑动窗口 | 每隔一小时整点运行（有开始日期和时间） | 1 | Hour | *startDate*Thh:00:00Z | {不可用} | {无} | {无} | 此计划不会在指定的开始日期和时间之前启动。** 将来的重复计划每隔一小时在第“00”分钟（基于开始时间计算）运行。 <p>如果频率为“周”或“月”，则此计划只会分别在每周的一个星期日期或每月的一个日期运行。 |
| 重复 <br>滑动窗口 | 每天每隔一小时运行（没有开始日期和时间） | 1 | Hour | {无} | {不可用} | {无} | {无} | 此计划立即启动，并根据上次运行时间计算将来的重复周期。 <p>如果频率为“周”或“月”，则此计划只会分别在每周的一个星期日期或每月的一个日期运行。 |
| 重复 <br>滑动窗口 | 每天每隔一小时运行（有开始日期和时间） | 1 | Hour | *startDate*T*startTime*Z | {不可用} | {无} | {无} | 此计划不会在指定的开始日期和时间之前启动，将根据上次运行时间计算将来的重复周期。** <p>如果频率为“周”或“月”，则此计划只会分别在每周的一个星期日期或每月的一个日期运行。 |
| 重复 <br>滑动窗口 | 每隔一小时在整点过后的第 15 分钟运行（有开始日期和时间） | 1 | Hour | *startDate*T00:15:00Z | {不可用} | {无} | {无} | 此计划不会在指定的开始日期和时间之前启动。** 将来的重复计划将在第“15”分钟（基于开始时间计算）运行，即凌晨 00:15、1:15、2:15 等。 |
| 定期 | 每隔一小时在整点过后的第 15 分钟运行（没有开始日期和时间） | 1 | 日期 | {无} | {不可用} | 0、1、2、3、4、5、6、7、8、9、10、11、12、13、14、15、16、17、18、19、20、21、22、23 | 15 | 此计划在 00:15 AM、1:15 AM、2:15 AM 等有规律的时间运行。 此外，此计划相当于频率为“小时”且开始时间为“15”分钟。 |
| 定期 | 每隔 15 分钟在指定的分钟数（无开始日期和时间）运行。 | 1 | 日期 | {无} | {不可用} | 0、1、2、3、4、5、6、7、8、9、10、11、12、13、14、15、16、17、18、19、20、21、22、23 | 0、15、30、45 | 此计划在下一个指定的 15 分钟标记之前不会启动。 |
| 定期 | 每天上午 8 点加上** 保存逻辑应用时的分钟标记运行 | 1 | 日期 | {无} | {不可用} | 8 | {无} | 如果没有开始日期和时间，此计划将基于你保存逻辑应用（PUT 操作）的时间运行。 |
| 定期 | 每天上午 8:00 运行（有开始日期和时间） | 1 | 日期 | *startDate*T08:00:00Z | {不可用} | {无} | {无} | 此计划不会在指定的开始日期和时间之前启动。** 将来每天上午 8:00 运行。 | 
| 定期 | 每日上午8:00 运行（无开始日期和时间） | 1 | 日期 | {无} | {不可用} | 8 | 00 | 此计划每天上午8:00 运行。 |
| 定期 | 每日上午8:00 到 4:00 PM 运行 | 1 | 日期 | {无} | {不可用} | 8, 16 | 0 | |
| 定期 | 每天上午 8:30、上午 8:45、下午 4:30 和下午 4:45 运行 | 1 | 日期 | {无} | {不可用} | 8, 16 | 30, 45 | |
| 定期 | 在每个星期六的下午5:00 运行（没有开始日期和时间） | 1 | Week | {无} | “星期六” | 17 | 0 | 此计划在每个星期六的 5:00 PM 运行。 |
| 定期 | 在每个星期六的下午5:00 运行（有开始日期和时间） | 1 | Week | *startDate*T17:00:00Z | “星期六” | {无} | {无} | 此计划不会在指定的开始日期和时间（在本例中为 2017 年 9 月 9 日 5:00 PM）之前启动。** 将来的定期计划在每个星期六的 5:00 PM 运行。 |
| 定期 | 每周二、周四下午 5 点加上** 保存逻辑应用时的分钟标记运行| 1 | Week | {无} | “星期二”、“星期四” | 17 | {无} | |
| 定期 | 在工作时间内每小时运行一次。 | 1 | Week | {无} | 选择除星期六和星期日以外的所有星期日期。 | 选择所需的日期小时。 | 选择所需的任何小时分钟。 | 例如，如果工作时间为 8:00 AM 到 5:00 PM，则选择 "8、9、10、11、12、13、14、15、16、17" 作为当天的时间*加上*"0" 作为小时的分钟。 |
| 定期 | 在周末每隔一个星期日期运行一次 | 1 | Week | {无} | “星期六”、“星期日” | 选择所需的日期小时。 | 选择适当的任何小时分钟。 | 此计划根据指定的计划在每个星期六和星期日运行。 |
| 定期 | 仅每两周在星期一每隔 15 分钟运行 | 2 | Week | {无} | “星期一” | 0、1、2、3、4、5、6、7、8、9、10、11、12、13、14、15、16、17、18、19、20、21、22、23 | 0、15、30、45 | 此计划按每个 15 分钟标记每隔两周在星期一运行。 |
| 定期 | 每月运行 | 1 | 月份 | *startDate*T*startTime*Z | {不可用} | {不可用} | {不可用} | 此计划不会在指定的开始日期和时间之前启动，将基于开始日期和时间计算将来的重复周期。** 如果未指定开始日期和时间，则此计划使用创建日期和时间。 |
| 定期 | 在每个月每隔一小时运行一天 | 1 | 月份 | {参阅备注} | {不可用} | 0、1、2、3、4、5、6、7、8、9、10、11、12、13、14、15、16、17、18、19、20、21、22、23 | {参阅备注} | 如果未指定开始日期和时间，则此计划使用创建日期和时间。 若要控制定期计划的分钟，请指定小时分钟和开始时间，或使用创建时间。 例如，如果开始时间或创建时间为 8:25 AM，则此计划在 8:25 AM、9:25 AM、10:25 AM 等有规律的时间运行。 |
|||||||||

<a name="run-once"></a>

## <a name="run-one-time-only"></a>仅运行一次

如果只想在将来一次运行逻辑应用，则可以使用**计划程序： "运行一次作业**" 模板。 创建新的逻辑应用之后、打开逻辑应用设计器之前，请从“模板”部分下的“类别”列表中选择“计划”，然后选择以下模板：************

![选择 "计划程序：运行一次作业" 模板](./media/concepts-schedule-automated-recurring-tasks-workflows/choose-run-once-template.png)

或者，可以使用“收到 HTTP 请求时 - 请求”触发器启动逻辑应用，并将开始时间作为触发器的参数传递。**** 对于第一个操作，请使用“延迟截止时间 - 计划”操作，并提供开始运行下一操作的时间。****

## <a name="next-steps"></a>后续步骤

* [使用“定期”触发器创建、计划和运行重复任务和工作流](../connectors/connectors-native-recurrence.md)
* [使用滑动窗口触发器创建、计划和运行重复性任务与工作流](../connectors/connectors-native-sliding-window.md)
* [使用延迟操作暂停工作流](../connectors/connectors-native-delay.md)
