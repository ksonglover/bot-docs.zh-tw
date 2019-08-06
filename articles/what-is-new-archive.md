---
title: 新功能 | Microsoft Docs
description: 了解 Bot Framework 中的新功能。
keywords: bot framework, azure bot service
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: conceptual
ms.service: bot-service
ms.subservice: abs
ms.date: 07/17/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 4553335cbca5b5eb720c7cffd11c8e14c8aa19c1
ms.sourcegitcommit: f3fda6791f48ab178721b72d4f4a77c373573e38
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/31/2019
ms.locfileid: "68671524"
---
# <a name="whats-new-in-bot-framework-may-2019"></a>Bot Framework 中的新功能 (2019 年 5 月)

|   | C#  | JS  | Python |  Java | 
|---|:---:|:---:|:------:|:-----:|
|SDK |[4.4.3][1] | [4.4.0][2] | [4.4.0b1 (預覽)][3] | [4.0.0a6 (預覽)][3a]|
|Docs | [docs][5] |[docs][5] |  | |
|範例 |[.NET Core][6]、[WebAPI][10] |[Node.js][7]、[TypeScript][8]、[es6][9]  | [Python][111] | | 

[1a]:https://github.com/microsoft/botframework-sdk/#readme
[1]:https://github.com/Microsoft/botbuilder-dotnet/#packages
[2]:https://github.com/Microsoft/botbuilder-js#packages
[3]:https://github.com/Microsoft/botbuilder-python#packages
[3a]:https://github.com/Microsoft/botbuilder-java#packages
[4]:https://github.com/Microsoft/botbuilder-java#packages
[5]:https://docs.microsoft.com/azure/bot-service/?view=azure-bot-service-4.0
[6]:https://github.com/Microsoft/BotBuilder-Samples/tree/master/samples/csharp_dotnetcore
[7]:https://github.com/Microsoft/BotBuilder-Samples/tree/master/samples/javascript_nodejs
[8]:https://github.com/Microsoft/BotBuilder-Samples/tree/master/samples/javascript_typescript
[9]:https://github.com/Microsoft/BotBuilder-Samples/tree/master/samples/javascript_es6
[10]:https://github.com/Microsoft/BotBuilder-Samples/tree/master/samples/csharp_webapi
[111]:https://github.com/Microsoft/botbuilder-python/tree/master/samples

<a name="V4-whats-new"></a>
## <a name="bot-framework-sdk-new-in-preview"></a>Bot Framework SDK (新功能！ 處於預覽狀態)

- [自適性對話方塊][47] | [docs][48] | [C# 範例][49]：自適性對話方塊可讓開發人員建置對話，並隨著對話的進展而動態變化。  傳統上，開發人員已事先安排好整個對話流程，但這會限制對話彈性。  自適性對話方塊則可讓開發人員更有彈性地回應內容變化，並隨著對話的進展而插入新步驟或整個子對話方塊。 

- [語言產生][43] | [docs][44] | [C# 範例][45]：語言產生可讓開發人員從程式碼和資源檔案中擷取出內嵌字串，並透過語言產生執行階段和檔案格式來進行管理。  語言產生可讓客戶對片語定義多個變化、執行以內容為基礎的簡單運算式、參考對話式記憶體，久而久之，我們將能夠讓所有額外功能變成更自然的對話式體驗。

- [常見的運算式語言][40] | [api][41]：自適性對話方塊和語言產生均依賴並使用常見的運算式語言來支援 Bot 對話。

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

## <a name="botkit"></a>Botkit
[Botkit][100] 是一種開發人員工具和 SDK，可用於為主要的傳訊平台建置聊天 Bot、應用程式和自訂整合。 Botkit Bot 會 `hear()` 觸發程序、`ask()` 問題和 `say()` 回覆。 開發人員可以使用此語法來建置對話方塊 - 現在可與 Bot Framework SDK 的最新版本交叉相容。 

此外，Botkit 還帶來 6 個平台配接器，可讓 Javascript Bot 應用程式直接與傳訊平台通訊：[Slack][102]、[Webex Teams][103]、[Google Hangouts][104]、[Facebook Messenger][105]、[Twilio][106] 和[網路聊天][107]。

Botkit 是 Microsoft Bot Framework 的一部分，其依據 [MIT 開放原始碼授權][101]來發行

[100]:https://github.com/howdyai/botkit#readme
[101]:https://github.com/howdyai/botkit/blob/master/LICENSE.md
[102]:https://github.com/howdyai/botkit/tree/master/packages/botbuilder-adapter-slack#readme
[103]:https://github.com/howdyai/botkit/tree/master/packages/botbuilder-adapter-webex#readme
[104]:https://github.com/howdyai/botkit/tree/master/packages/botbuilder-adapter-hangouts#readme
[105]:https://github.com/howdyai/botkit/tree/master/packages/botbuilder-adapter-facebook#readme
[106]:https://github.com/howdyai/botkit/tree/master/packages/botbuilder-adapter-twilio-sms#readme
[107]:https://github.com/howdyai/botkit/tree/master/packages/botbuilder-adapter-web#readme

## <a name="bot-framework-solutions-new-in-preview"></a>Bot Framework 解決方案 (新功能！ 處於預覽狀態)

[Bot Framework 解決方案存放庫](https://github.com/Microsoft/AI#readme)是一組範本、解決方案加速器和技能的所在地，可用來協助建置類似小幫手的進階對話式體驗。

| Name | 說明 |  
|:------------|:------------| 
|[**虛擬小幫手**](https://github.com/Microsoft/AI/tree/master/docs#virtual-assistant) | 客戶極需要提供專屬於其品牌的對話式小幫手，並且還需要為他們的使用者量身打造，以及在各種畫布和裝置上提供使用。 <br/><br/> 企業範本可大幅簡化新 Bot 專案的建立程序，包括：基本的對話式意圖、分派整合、QnA Maker、Application Insights 和自動化部署。|
|[**技能**](https://github.com/Microsoft/AI/blob/master/docs/overview/skills.md)| 開發人員可以藉由將可重複使用的對話式功能 (稱為技能) 拼接在一起，來撰寫對話式體驗。 技能本身就是可從遠端叫用的 Bot，而且有技能開發人員範本 (.NET、TS) 可供加速新技能的建立。 
|[**分析**](https://github.com/Microsoft/AI/blob/master/docs/readme.md#analytics)| 可使用對話式 AI 分析解決方案來重點了解 Bot 的健康情況和行為。 檢閱可用的遙測資料、Application Insights 查詢範例和 Power BI 儀表板，來了解 Bot 與使用者的完整對話範圍。 |

## <a name="azure-bot-service"></a>Azure Bot 服務
Azure Bot Service 可讓您裝載有智慧的企業級 Bot，且您可以完整擁有和控制您的資料。 開發人員可以註冊其 Bot 並將 Bot 連線至 Skype、Microsoft Teams、Cortana、網路聊天等通道上的使用者。 [Azure][27]  |  [docs][28] | [連線至通道][29] 

* **Direct Line JS 用戶端**：如果您想要在 Azure Bot Service 中使用 Direct Line 通道，而不要使用網路聊天用戶端，則可以在自訂應用程式中使用 Direct Line JS 用戶端。 如需詳細資訊，請移至 [GitHub][30]。

<a name="ABS-whats-new"></a>

* **新功能！Direct Line Speech 通道**：我們會結合 Bot Framework 和 Microsoft 的語音服務來提供通道，以便能在用戶端與 Bot 應用程式之間雙向串流語音和文字。  如需詳細資訊，請參閱如何新增[語音通道至 Bot](https://docs.microsoft.com/azure/bot-service/directline-speech-bot?view=azure-bot-service-4.0)。

[27]:https://azure.microsoft.com/services/bot-service/
[28]:https://docs.microsoft.com/azure/bot-service/bot-service-overview-introduction?view=azure-bot-service-4.0
[29]:https://docs.microsoft.com/azure/bot-service/bot-service-manage-channels?view=azure-bot-service-4.0
[30]:https://github.com/Microsoft/BotFramework-DirectLineJS/blob/master/README.md


## <a name="bot-framework-emulator"></a>Bot Framework 模擬器
[Bot Framework Emulator][60] 是跨平台的桌面應用程式，可讓 Bot 開發人員針對使用 Bot Framework SDK 所建置的 Bot 進行測試和偵錯。 您可以使用 Bot Framework Emulator 來測試在電腦本機上執行的 Bot，或用來連線至遠端執行的 Bot。

- [下載最新版本][61] | [Docs][62]

<a name="Emulator-whats-new"></a>
### <a name="bot-inspector-new-in-preview"></a>Bot Inspector (新功能！ 處於預覽狀態)

Bot Framework Emulator 已發行新 Bot Inspector 功能的搶鮮版 (Beta)。 其可讓您在 Microsoft Teams、Slack、Cortana、Facebook Messenger、Skype 等通道上對 Bot Framework SDK v4 Bot 進行偵錯和測試。當您有對話時，訊息便會鏡像傳送到 Bot Framework Emulator，您可以在其中檢查 Bot 所收到的訊息資料。 此外，也會呈現通道和 Bot 之間任何給定回合的 Bot 狀態快照集。 深入了解 [Bot Inspector](https://github.com/Microsoft/BotFramework-Emulator/blob/master/content/CHANNELS.md)。

[60]:https://github.com/Microsoft/BotFramework-Emulator#readme
[61]:https://github.com/Microsoft/BotFramework-Emulator/releases/latest
[62]:https://docs.microsoft.com/azure/bot-service/bot-service-debug-emulator?view=azure-bot-service-4.0


## <a name="related-services"></a>相關服務

### <a name="language-understanding"></a>Language Understanding 
該服務以機器學習為基礎，可用來建置自然語言體驗。 快速建立持續改進的企業級自訂模型。 [Language Understanding 服務 (LUIS)][30] 可讓應用程式了解人在文字中所表達的意思。

<a name="LUIS-whats-new"></a>

- **新功能！角色、外部實體和動態實體**：LUIS 新增了數個功能，可讓開發人員從文字擷取更詳細的資訊，讓使用者現在可以用最少的力氣建置更有智慧的解決方案。 LUIS 也將角色延伸到所有實體類型，讓相同的實體可根據內容使用不同的子類型來分類。 開發人員現在可以更細微的控制其可以使用 LUIS 執行的動作，包括能夠在執行階段透過動態清單和外部實體識別和更新模型。 動態清單可用來在預測階段附加至清單實體，而讓使用者專屬資訊可以完全相符。 個別的增補實體擷取器會使用外部實體來執行，且該資訊可以附加至 LUIS 作為其他模型的強烈訊號。

- **新功能！分析儀表板**：LUIS 會釋出更詳細、有豐富視覺效果的完善分析儀表板。 其方便使用的設計會醒目指出大部分使用者在設計應用程式時所面臨的常見問題，其方法是提供簡單的解決方式說明，以協助使用者深入了解其模型的品質、潛在的資料問題和採用最佳做法的指導方針。

[Docs][31] | [將語言理解新增至 Bot][32] 

[18]:https://github.com/Microsoft/botbuilder-tools/tree/master/packages/LUIS#readme
[19]:https://github.com/Microsoft/botbuilder-tools/tree/master/packages/QnAMaker#readme
[30]:https://www.luis.ai
[31]:https://docs.microsoft.com/azure/cognitive-services/LUIS/Home
[32]:https://docs.microsoft.com/azure/bot-service/bot-builder-howto-v4-luis?view=azure-bot-service-4.0&branch=pr-en-us-1325&tabs=csharp

### <a name="qna-maker"></a>QnA Maker
[QnA Maker][33] 是一項雲端式 API 服務，可對您的資料建立交談式的問答層。 透過 QnA Maker，您可以根據常見問題集 URL、結構化文件、產品手冊或期刊內容，在短短幾分鐘內建置、定型及發佈簡單的問答 Bot。

<a name="QnA-whats-new"></a>

- **新功能！擷取管線**：現在，您可以從 URL、檔案和 SharePoint 擷取階層式資訊
- **新功能！智慧**：內容相關的排名模型、主動式學習建議
- **新功能！對話**：QnA Maker 中的多回合對話。

[Docs][34]  | [將 QnAMaker 新增至 Bot][35] 

[33]:https://www.qnamaker.ai/
[34]:https://aka.ms/qnamaker-docs-home
[35]:https://docs.microsoft.com/azure/bot-service/bot-builder-howto-qna?view=azure-bot-service-4.0&branch=pr-en-us-1325&tabs=cs

### <a name="speech-services"></a>語音服務
[語音服務](https://docs.microsoft.com/azure/cognitive-services/speech-service/)會使用整合的語音服務將音訊轉換成文字，並執行語音翻譯和文字轉換語音。 透過語音服務，您可以將語音整合至 Bot、建立自訂喚醒字組，並以多種語言撰寫。

### <a name="adaptive-cards"></a>調適型卡片
[自適性卡片](https://adaptivecards.io)是一套開放式標準，可讓開發人員以通用且一致的方式交換卡片內容，並可讓 Bot Framework 的開發人員用來建立絕佳的跨通道對話式體驗。

## <a name="additional-information"></a>其他資訊
- 如需詳細資訊，請瀏覽 [GitHub 頁面](https://github.com/Microsoft/botframework/blob/master/whats-new.md#whats-new)。
