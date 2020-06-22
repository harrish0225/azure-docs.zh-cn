---
title: 长音频 API （预览）-语音服务
titleSuffix: Azure Cognitive Services
description: 了解长音频 API 的设计方式是将长格式文本的异步合成转换为语音。
services: cognitive-services
author: trevorbye
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 01/30/2020
ms.author: trbye
ms.openlocfilehash: 0e18fd0c52fd4090477599f53cd0ef0bc05855f2
ms.sourcegitcommit: bb0afd0df5563cc53f76a642fd8fc709e366568b
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/19/2020
ms.locfileid: "83587334"
---
# <a name="long-audio-api-preview"></a>长音频 API （预览）

长音频 API 设计用于将长格式文本的异步合成到语音（例如：音频书籍、新闻文章和文档）。 此 API 不会实时返回合成音频，而是期望你会轮询响应并使用这些输出，因为服务中提供了这些响应。 与语音 SDK 使用的文本到语音 API 不同，长音频 API 可以创建超过10分钟的合成音频，使其成为发布者和音频内容平台的理想选择。

长音频 API 的其他好处：

* 服务返回的合成语音使用最佳的神经声音。
* 无需部署语音终结点，因为它在无实时批处理模式下合成语音。

> [!NOTE]
> 长音频 API 现在支持[公共神经声音](https://docs.microsoft.com/azure/cognitive-services/speech-service/language-support#neural-voices)和[自定义神经声音](https://docs.microsoft.com/azure/cognitive-services/speech-service/how-to-custom-voice#custom-neural-voices)。

## <a name="workflow"></a>工作流

通常，使用长音频 API 时，你将提交一个文本文件或文件以进行合成，轮询状态，如果状态为 "成功"，则可以下载音频输出。

此图提供工作流的高级概述。

![长音频 API 工作流关系图](media/long-audio-api/long-audio-api-workflow.png)

## <a name="prepare-content-for-synthesis"></a>为合成准备内容

准备文本文件时，请确保：

* 为纯文本（.txt）或 SSML 文本（.txt）
* 编码为[utf-8，具有字节顺序标记（BOM）](https://www.w3.org/International/questions/qa-utf8-bom.en#bom)
* 是单个文件，而不是 zip
* 对于 SSML 文本，对于 SSML 文本包含400个以上的字符[，对于 SSML 文本包含400个](https://docs.microsoft.com/azure/cognitive-services/speech-service/text-to-speech#pricing-note)，小于10000个段落
  * 对于纯文本，每个段落都按 "**输入/返回**-查看[纯文本输入" 示例](https://github.com/Azure-Samples/Cognitive-Speech-TTS/blob/master/CustomVoice-API-Samples/Java/en-US.txt)分隔
  * 对于 SSML 文本，每个 SSML 部分都被视为一个段落。 SSML 部分应由不同的段落分隔-查看[ssml 文本输入示例](https://github.com/Azure-Samples/Cognitive-Speech-TTS/blob/master/CustomVoice-API-Samples/Java/SSMLTextInputSample.txt)
> [!NOTE]
> 对于中文（大陆）、中文（香港特别行政区）、中文（台湾）、日语和韩语，一个字将被计为两个字符。 

## <a name="submit-synthesis-requests"></a>提交合成请求

准备好输入内容后，请遵循[长格式的音频合成快速](https://aka.ms/long-audio-python)启动来提交请求。 如果有多个输入文件，则需要提交多个请求。 

**HTTP 状态代码**指示常见错误。

| API | HTTP 状态代码 | 说明 | 提议 |
|-----|------------------|-------------|----------|
| 创建 | 400 | 此区域未启用语音合成。 | 更改受支持区域的语音订阅密钥。 |
|        | 400 | 只有此区域的**标准**语音订阅才有效。 | 将语音订阅密钥更改为 "标准" 定价层。 |
|        | 400 | 超过 Azure 帐户的20000请求限制。 请在提交新请求之前删除一些请求。 | 服务器将为每个 Azure 帐户最多保留20000请求。 请在提交新请求之前删除一些请求。 |
|        | 400 | 此模型不能用于语音合成： {modelID}。 | 请确保 {modelID} 的状态正确。 |
|        | 400 | 请求的区域与模型的区域不匹配： {modelID}。 | 请确保 {modelID} 的区域与请求的区域相匹配。 |
|        | 400 | 语音合成仅支持采用字节顺序标记的 UTF-8 编码格式的文本文件。 | 请确保输入文件采用带有字节顺序标记的 UTF-8 编码。 |
|        | 400 | 语音合成请求中仅允许有效的 SSML 输入。 | 请确保输入的 SSML 表达式是正确的。 |
|        | 400 | 在输入文件中找不到语音名称 {voiceName}。 | 输入的 SSML 语音名称与模型 ID 不一致。 |
|        | 400 | 输入文件中的段落量应小于10000。 | 请确保文件中的段落小于10000。 |
|        | 400 | 输入文件应超过400个字符。 | 请确保输入文件超过400个字符。 |
|        | 404 | 找不到语音合成定义中声明的模型： {modelID}。 | 请确保 {modelID} 是正确的。 |
|        | 429 | 超出活动的语音合成限制。 请等待一些请求完成。 | 允许服务器运行每个 Azure 帐户，并为多达120请求排队。 请等待并避免提交新请求，直到完成一些请求。 |
| 全部       | 429 | 请求太多。 | 对于每个 Azure 帐户，允许客户端每秒向服务器提交最多5个请求。 请减少每秒的请求数。 |
| 删除    | 400 | 语音合成任务仍在使用中。 | 只能删除**完成**或**失败**的请求。 |
| GetByID   | 404 | 找不到指定的实体。 | 请确保合成 ID 是正确的。 |

## <a name="regions-and-endpoints"></a>区域和终结点

长音频 API 在具有唯一终结点的多个区域中提供。

| 区域 | 终结点 |
|--------|----------|
| 澳大利亚东部 | `https://australiaeast.customvoice.api.speech.microsoft.com` |
| 加拿大中部 | `https://canadacentral.customvoice.api.speech.microsoft.com` |
| 美国东部 | `https://eastus.customvoice.api.speech.microsoft.com` |
| 印度中部 | `https://centralindia.customvoice.api.speech.microsoft.com` |
| 美国中南部 | `https://southcentralus.customvoice.api.speech.microsoft.com` |
| 东南亚 | `https://southeastasia.customvoice.api.speech.microsoft.com` |
| 英国南部 | `https://uksouth.customvoice.api.speech.microsoft.com` |
| 西欧 | `https://westeurope.customvoice.api.speech.microsoft.com` |
| 美国西部 2 | `https://westus2.customvoice.api.speech.microsoft.com` |

## <a name="audio-output-formats"></a>音频输出格式

支持灵活的音频输出格式。 可以通过设置 "concatenateResult" 参数，按段落生成音频输出或将音频输出连接到单个输出。 长音频 API 支持以下音频输出格式：

> [!NOTE]
> 默认音频格式为 riff-16khz-16 位。

* riff-8khz-16 位
* riff-16khz-16 位
* riff-24khz-16 位
* riff-48khz-16 位
* 16khz-32kbitrate-mp3
* 16khz-64kbitrate-mp3
* 16khz-128kbitrate-mp3
* 24khz-48kbitrate-mp3
* 24khz-96kbitrate-mp3
* 24khz-160kbitrate-mp3

## <a name="quickstarts"></a>快速入门

我们提供旨在帮助你成功运行长音频 API 的快速入门。 此表包含按语言组织的长音频 API 快速入门列表。

* [快速入门： Python](https://aka.ms/long-audio-python)

## <a name="sample-code"></a>示例代码
GitHub 上提供了长音频 API 的示例代码。

* [示例代码： Python](https://github.com/Azure-Samples/Cognitive-Speech-TTS/tree/master/CustomVoice-API-Samples/Python)
* [示例代码： C#](https://github.com/Azure-Samples/Cognitive-Speech-TTS/tree/master/CustomVoice-API-Samples/CSharp)
* [示例代码： Java](https://github.com/Azure-Samples/Cognitive-Speech-TTS/blob/master/CustomVoice-API-Samples/Java/)
