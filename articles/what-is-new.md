---
title: 新功能 | Microsoft Docs
description: 了解 Bot Framework 中的新功能。
keywords: bot framework, azure bot service
author: kamrani
ms.author: kamrani
manager: kamrani
ms.topic: conceptual
ms.service: bot-service
ms.date: 11/04/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 8cb03814a9fbb7fdbbfb457400eca3cf6a67ec17
ms.sourcegitcommit: 490810d278d1c8207330b132f28a5eaf2b37bd07
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 11/05/2019
ms.locfileid: "73592244"
---
# <a name="whats-new-november-2019-ignite"></a>2019 年 11 月的新功能 (Ignite)

[!INCLUDE[applies-to](includes/applies-to.md)]

Bot Framework SDK v4 是[開放原始碼 SDK](https://github.com/microsoft/botframework-sdk/#readme)，可讓開發人員使用其慣用的程式設計語言塑造與建置複雜的對話。

本文摘要說明 Bot Framework 和 Azure Bot Service 中的重要新功能和增強功能。


|   | C#  | JS  | Python |  Java | 
|---|:---:|:---:|:------:|:-----:|
|版本 |[4.6 GA][1] | [4.6 GA][2] | [Beta 4][3] | [Preview 3][3a]|
|Docs | [docs][5] |[docs][5] |  | |
|範例 |[.NET Core][6]、[WebAPI][10] |[Node.js][7]、[TypeScript][8]、[es6][9]  | | | 


[1]:https://github.com/Microsoft/botbuilder-dotnet/#packages
[2]:https://github.com/Microsoft/botbuilder-js#packages
[3]:https://github.com/Microsoft/botbuilder-python#packages
[3a]:https://github.com/Microsoft/botbuilder-java#packages
[5]:https://docs.microsoft.com/azure/bot-service/?view=azure-bot-service-4.0
[6]:https://github.com/Microsoft/BotBuilder-Samples/tree/master/samples/csharp_dotnetcore
[7]:https://github.com/Microsoft/BotBuilder-Samples/tree/master/samples/javascript_nodejs
[8]:https://github.com/Microsoft/BotBuilder-Samples/tree/master/samples/typescript_nodejs
[9]:https://github.com/Microsoft/BotBuilder-Samples/tree/master/samples/javascript_es6
[10]:https://github.com/Microsoft/BotBuilder-Samples/tree/master/samples/csharp_webapi

#### <a name="bot-framework-sdk-for-microsoft-teams-ga"></a>適用於 Microsoft Teams 的 Bot Framework SDK (GA)
Bot Framework SDK 4.6 版已完全整合 Teams Bot 的建置支援，可讓使用者在通道或群組聊天對話中使用這些 Bot。 藉由將 Bot 新增至小組或聊天中，讓對話的所有使用者都可以在對話中直接使用 Bot 功能。  [[Docs](https://docs.microsoft.com/azure/bot-service/bot-builder-basics-teams)]

#### <a name="bot-framework-for-power-virtual-agent-preview"></a>適用於 Power Virtual Agent 的 Bot Framework (預覽)

Power Virtual Agent 的設計訴求是讓商務使用者可以在建置 SaaS 體驗的 UI 型 Bot 中建立 Bot，而不需要撰寫程式碼或管理特定 AI 服務。 Power Virtual Agents 可透過 Microsoft Bot Framework 進行擴充，讓開發人員和商務使用者可一起為其組織建置 Bot。 [[Docs](https://docs.microsoft.com/dynamics365/ai/customer-service-virtual-agent/overview)]


#### <a name="bot-framework-sdk-for-skills-preview"></a>用於建立技能的 Bot Framework SDK (預覽)

- **Bot 的技能**：建立可重複使用的對話技能，以將功能新增至 Bot。 利用預先建立的技能，例如行事曆、電子郵件、工作、景點、汽車、天氣和新聞技能。 技能包括因需求而進行自訂和延伸時所傳遞的語言模型、對話、問與答及整合程式碼。 [[Docs](https://microsoft.github.io/botframework-solutions/overview/skills/)]

- **Power Virtual Agent 的技能 - 即將推出！** ：針對使用 Power Virtual Agents 建立的 Bot，您可以使用 Bot Framework 和 Azure 認知服務來為這些 Bot 建置新的技能，而不需要從頭建立新的 Bot。 

#### <a name="adaptive-dialogs-preview"></a>自適性對話方塊 (預覽)
自適性對話方塊可讓開發人員根據內容和事件來動態更新對話流程。 在處理對話過程中的對話環境切換和中斷情形時，這特別有用。 [[Docs][48] | [C# 範例][49]] 

#### <a name="language-generation-preview"></a>語言產生 (預覽)
語言產生可讓開發人員區隔用來產生 Bot 回應的邏輯，包括定義片語的多個變化、根據內容執行簡單運算式，以及參考對話記憶。 [[Docs][44] | [C# 範例][45]]

#### <a name="common-expression-language-preview"></a>通用運算式語言 (預覽)
通用運算式語言可讓您在執行階段上評估條件式邏輯的結果。 您可以跨 Bot Framework SDK 和對話式 AI 元件使用通用語言，例如自適型對話方塊和語言產生。 [[Docs][40] | [API][41]]


[40]:https://github.com/Microsoft/BotBuilder-Samples/tree/master/experimental/common-expression-language#readme
[41]:https://github.com/Microsoft/BotBuilder-Samples/blob/master/experimental/common-expression-language/api-reference.md
[43]:https://github.com/Microsoft/BotBuilder-Samples/tree/master/experimental/language-generation#readme
[44]:https://github.com/Microsoft/BotBuilder-Samples/tree/master/experimental/language-generation/docs
[45]:https://github.com/Microsoft/BotBuilder-Samples/tree/master/experimental/language-generation/csharp_dotnetcore
[46]:https://github.com/Microsoft/BotBuilder-Samples/tree/master/experimental/language-generation/javascript_nodejs/13.core-bot
[47]:https://github.com/Microsoft/BotBuilder-Samples/tree/master/experimental/adaptive-dialog#readme
[48]:https://github.com/Microsoft/BotBuilder-Samples/tree/master/experimental/adaptive-dialog/docs
[49]:https://github.com/Microsoft/BotBuilder-Samples/tree/master/experimental/adaptive-dialog/csharp_dotnetcore
[50]:https://github.com/Microsoft/BotBuilder-Samples/tree/master/experimental/adaptive-dialog/declarative

## <a name="additional-information"></a>其他資訊
- 您可以在[這裡](what-is-new-archive.md)查看先前的公告。
