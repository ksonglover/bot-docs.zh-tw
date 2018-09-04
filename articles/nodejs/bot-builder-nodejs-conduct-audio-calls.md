---
title: 進行語音通話 | Microsoft Docs
description: 了解如何使用 Node.js 在 Bot 中與 Skype 進行語音通話
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/17
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 35af3a339a20fe0e7e70d001db035aec2647aa35
ms.sourcegitcommit: 2dc75701b169d822c9499e393439161bc87639d2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/24/2018
ms.locfileid: "42905317"
---
# <a name="support-audio-calls-with-skype"></a>使用 Skype 支援語音通話

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

Skype 支援稱為「呼叫 Bot」的豐富功能。  當啟用時，使用者可以撥打語音電話給 Bot，並使用互動語音回應系統 (IVR) 與其進行互動。  適用於 Node.js SDK 的 Bot 建立器包含特殊的[呼叫 SDK][calling_sdk]，開發人員可用來將撥打功能新增至其聊天 Bot。   

呼叫 SDK 與[聊天 SDK][chat_sdk] 非常類似。 它們具有類似的類別、會共用通用的建構，且您甚至可以使用聊天 SDK 將訊息傳送給您正在呼叫的使用者。  兩個 SDK 是設計為並排執行，雖然它們很類似，但有一些顯著的差異，而您通常應該避免將兩個程式庫中的類別混合。  

## <a name="create-a-calling-bot"></a>建立呼叫 Bot
下列範例程式碼顯示呼叫 Bot 的 "Hello World" 外觀，與一般聊天 Bot 類似的程度。 

```javascript
var restify = require('restify');
var calling = require('botbuilder-calling');

// Setup Restify Server
var server = restify.createServer();
server.listen(process.env.port || process.env.PORT || 3978, function () {
   console.log(`${server.name} listening to ${server.url}`); 
});

// Create calling bot
var connector = new calling.CallConnector({
    callbackUrl: 'https://<your host>/api/calls',
    appId: '<your bots app id>',
    appPassword: '<your bots app password>'
});
var bot = new calling.UniversalCallBot(connector);
server.post('/api/calls', connector.listen());

// Add root dialog
bot.dialog('/', function (session) {
    session.send('Watson... come here!');
});
```

> [!NOTE]
> 若要尋找您 Bot 的 **AppID** 和 **AppPassword**，請參閱 [MicrosoftAppID 和 MicrosoftAppPassword](~/bot-service-manage-overview.md#microsoftappid-and-microsoftapppassword)。

模擬器目前不支援對呼叫 Bot 進行測試。 若要測試您的 Bot，您必須完成發佈您 bot 所需的大部分步驟。  您也必須使用 Skype 用戶端來與 Bot 互動。 

### <a name="enable-the-skype-channel"></a>啟用 Skype 通道
[註冊您的 Bot](../bot-service-quickstart-registration.md)並啟用 Skype 通道。 當您註冊 Bot 時，必須提供傳訊端點。 建議您將呼叫 Bot 與聊天 Bot 配對，如此聊天 Bot 的端點就是您一般會放入該欄位的端點。  如果您只註冊呼叫 Bot，則可以只將呼叫端點貼入該欄位。  

若要啟用的實際呼叫功能，您必須進入 Bot 的 Skype 通道，並開啟呼叫功能。 接著您會取得一個欄位，可將您的呼叫端點複製進去。 請確定您將 https ngrok 連結用於您呼叫端點的主機部分。

在您註冊 Bot 期間，系統將會指派應用程式識別碼和密碼給您，您應該將其貼入您 hello world Bot 的連接器設定。 您也需要取得完整的呼叫連結，並貼到 callbackUrl 設定中。

### <a name="add-bot-to-contacts"></a>將 Bot 新增至連絡人
在您開發人員入口網站中的 Bot 註冊頁面上，您會在 Bot Skype 通道旁看到 [新增至 Skype] 按鈕。 按一下按鈕即可將 Bot 新增至 Skype 中的連絡人清單。  一旦您 (及取得此加入連結的任何人) 這麼做，將能夠與 Bot 進行通訊。

### <a name="test-your-bot"></a>測試 Bot
您可以使用 Skype 用戶端來測試 Bot。 當您按一下 Bot 連絡人項目 (您可能必須搜尋 Bot 才能看到它) 時，應該會注意到呼叫圖示亮起。如果您已將呼叫新增至現有的 Bot，呼叫圖示可能需要幾分鐘的時間才會亮起。  

如果您按下呼叫按鈕，應該會撥給您的 Bot，且您應該會聽到它說「Watson... 來這裡！」 然後掛斷。

## <a name="calling-basics"></a>呼叫基本概念
[UniversalCallBot](http://docs.botframework.com/en-us/node/builder/calling-reference/classes/_botbuilder_d_.universalcallbot) 和 [CallConnector](http://docs.botframework.com/en-us/node/builder/calling-reference/classes/_botbuilder_d_.callconnector) 類別可讓您使用與聊天 Bot 大致相同的方式來撰寫呼叫 Bot。 您要將基本上與[聊天對話方塊](bot-builder-nodejs-manage-conversation-flow.md)相同的對話方塊新增至您的 Bot。 您可以將[瀑布圖](bot-builder-nodejs-prompts.md)新增到您的 Bot。 有一個工作階段物件 ([CallSession](http://docs.botframework.com/en-us/node/builder/calling-reference/classes/_botbuilder_d_.callsession) 類別)，其中包含已新增的 [answer()](http://docs.botframework.com/en-us/node/builder/calling-reference/classes/_botbuilder_d_.callsession#answer)、[hangup()](http://docs.botframework.com/en-us/node/builder/calling-reference/classes/_botbuilder_d_.callsession#hangup)，以及 [reject()](http://docs.botframework.com/en-us/node/builder/calling-reference/classes/_botbuilder_d_.callsession#reject) 方法，可管理目前的呼叫。 一般情況下，您不需要擔心這些呼叫控制方法，因為 CallSession 具有邏輯，可自動管理您的呼叫。 如果您採取例如傳送訊息或呼叫內建提示等動作，工作階段會自動接聽電話。 如果您呼叫 [endConversation()](http://docs.botframework.com/en-us/node/builder/calling-reference/classes/_botbuilder_d_.callsession#endconversation)，它也會自動掛斷/拒絕呼叫，或是會偵測到您已停止詢問呼叫者問題 (您未呼叫內建的提示)。

呼叫和聊天 Bot 的另一個差異是，聊天 Bot 通常會將訊息、卡片和鍵盤傳送給使用者，而呼叫 Bot 則會處理「動作」和「結果」。 建立組成一或多個[動作](http://docs.botframework.com/en-us/node/builder/calling-reference/interfaces/_botbuilder_d_.iaction)的[工作流程](http://docs.botframework.com/en-us/node/builder/calling-reference/interfaces/_botbuilder_d_.iworkflow)時，需要 Skype 呼叫 Bot。  這在實務上是您不需要太擔心的另一項事情，因為 Bot 建立器呼叫 SDK 幾乎都會為您管理。 [CallSession.send()](http://docs.botframework.com/en-us/node/builder/calling-reference/classes/_botbuilder_d_.callsession#send) 方法可讓您傳遞動作或字串，從而會變成 [PlayPromptActions](http://docs.botframework.com/en-us/node/builder/calling-reference/classes/_botbuilder_d_.playpromptaction)。  工作階段包含自動批次邏輯，可將多個動作合併成提交給呼叫服務的單一工作流程，以便您可以安全地多次呼叫 send()。  且您應仰賴 SDK 的內建[提示](bot-builder-nodejs-prompts.md)，當使用者在處理所有結果時，向他們收集輸入。  

[calling_sdk]: http://docs.botframework.com/en-us/node/builder/calling-reference/modules/_botbuilder_d_
[chat_sdk]: http://docs.botframework.com/en-us/node/builder/chat-reference/modules/_botbuilder_d_
