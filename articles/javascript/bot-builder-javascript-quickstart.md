---
title: 使用適用於 JavaScript 的 Bot 建立器 SDK 建立 Bot | Microsoft Docs
description: 使用適用於 JavaScript 的 Bot 建立器 SDK 快速建立 Bot。
keywords: 快速入門, Bot 建立器 sdk, 開始使用
author: jonathanfingold
ms.author: jonathanfingold
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 07/12/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 337b6a0b8739b5e5de4d2d1b2b87dcad55f2c854
ms.sourcegitcommit: dcbc8ad992a3e242a11ebcdf0ee99714d919a877
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/30/2018
ms.locfileid: "39352927"
---
# <a name="create-a-bot-with-the-bot-builder-sdk-v4-preview-for-javascript"></a>使用適用於 JavaScript 的 Bot 建立器 SDK v4 (預覽) 建立 Bot
[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

本快速入門會逐步引導您使用 Yeoman Bot 建立器 產生器和適用於 JavaScript 的 Bot 建立器 SDK 建置 Bot，然後使用 Bot Framework 模擬器進行測試。 本文的根據是 [Microsoft Bot 建立器 SDK v4](https://github.com/Microsoft/botbuilder-js)。

## <a name="pre-requisites"></a>先決條件
- [Visual Studio Code](https://www.visualstudio.com/downloads)
- [Node.js](https://nodejs.org/en/)
- [Yeoman](http://yeoman.io/)，可使用產生器為您建立 Bot
- [Bot 模擬器](https://github.com/Microsoft/BotFramework-Emulator)
- 了解 [restify](http://restify.com/) 和 Java 中的非同步程式設計

> [!NOTE]
> 在某些安裝中，restify 的安裝步驟會產生與 node-gyp 相關的錯誤。
> 如果您遇到這種情況，請嘗試執行 `npm install -g windows-build-tools`。


適用於 JavaScript 的 Bot 建立器 SDK 包含一系列[套件](https://github.com/Microsoft/botbuilder-js/tree/master/libraries)，您可以使用特殊的 `@preview` 標記從 NPM 進行安裝。

# <a name="create-a-bot"></a>建立 Bot

開啟提升權限的命令提示字元、建立目錄，然後為 Bot 初始化套件。

```bash
md myJsBots
cd myJsBots
```

接下來，安裝適用於 JavaScript 的 Yeoman 和產生器。

```bash
npm install -g yo
npm install -g generator-botbuilder@preview
```

然後，使用產生器來建立 echo Bot。

```bash
yo botbuilder
```

Yeoman 會提示您輸入一些用來建立 Bot 的資訊。
-   輸入 Bot 的名稱。
-   輸入描述。
-   選擇 Bot 的語言，可以是 `JavaScript` 或 `TypeScript`。
-   選擇要使用的範本。 目前，`Echo` 是唯一的範本，但很快就會新增其他範本。

Yeoman 會在新資料夾中建立 Bot。

## <a name="explore-code"></a>探索程式碼

當您開啟新建立的 Bot 資料夾時，您會看到 `app.js` 檔案。 這個 `app.js` 檔案會包含執行 Bot 應用程式所需的所有程式碼。 此檔案內所包含的 echo Bot 會回傳您所輸入的任何內容，並將計數器遞增。 

在下列程式碼中，對話狀態中介軟體會使用記憶體內部儲存體。 它會在儲存體中讀取和寫入狀態物件。 計數變數會追蹤傳送至 Bot 的訊息數目。 您可以使用類似技巧，讓狀態維持到下個回合。 

**app.js**
```javascript
// Packages are installed for you
const { BotFrameworkAdapter, MemoryStorage, ConversationState } = require('botbuilder');
const restify = require('restify');

// Create server
let server = restify.createServer();
server.listen(process.env.port || process.env.PORT || 3978, function () {
    console.log(`${server.name} listening to ${server.url}`);
});

// Create adapter
const adapter = new BotFrameworkAdapter({ 
    appId: process.env.MICROSOFT_APP_ID, 
    appPassword: process.env.MICROSOFT_APP_PASSWORD 
});

// Add conversation state middleware
const conversationState = new ConversationState(new MemoryStorage());
adapter.use(conversationState);
```

下列程式碼會接聽傳入要求，並在傳送回覆給使用者之前，先檢查傳入的活動類型。

```javascript
// Listen for incoming requests 
server.post('/api/messages', (req, res) => {
    // Route received request to adapter for processing
    adapter.processActivity(req, res, (context) => {
        // This bot is only handling Messages
        if (context.activity.type === 'message') {
        
            // Get the conversation state
            const state = conversationState.get(context);
            
            // If state.count is undefined set it to 0, otherwise increment it by 1
            const count = state.count === undefined ? state.count = 0 : ++state.count;
            
            // Echo back to the user whatever they typed.
            return context.sendActivity(`${count}: You said "${context.activity.text}"`);
        } else {
            // Echo back the type of activity the bot detected if not of type message
            return context.sendActivity(`[${context.activity.type} event detected]`);
        }
    });
});
```

## <a name="start-your-bot"></a>啟動 Bot

變更為針對 Bot 所建立的目錄，並加以啟動。

```bash
cd <bot directory>
node app.js
```

## <a name="start-the-emulator-and-connect-your-bot"></a>啟動模擬器並連線到 Bot
此時，Bot 正在本機執行。 接下來，啟動模擬器，然後在模擬器中連線到 Bot：
1. 按一下模擬器 [歡迎使用] 索引標籤中的 [建立新的 Bot 組態] 連結。 

2. 輸入 **Bot 名稱**，然後輸入 Bot 程式碼的目錄路徑。 Bot 組態檔會儲存至這個路徑。

3. 將 `http://localhost:port-number/api/messages` 輸入 [端點 URL] 欄位，其中連接埠號碼必須符合應用程式執行所在瀏覽器中顯示的連接埠號碼。

4. 按一下 [連線] 來連線到 Bot。 您不需要指定 **Microsoft 應用程式識別碼**和 **Microsoft 應用程式密碼**。 目前可以將這些欄位保留空白。 稍後註冊 Bot 時，您會取得這項資訊。

傳送 "Hi" 給 Bot，然後 Bot 就會對該訊息回應 '0: You said "Hi"'。

## <a name="next-steps"></a>後續步驟

接下來，請跳至說明 Bot 及其運作方式的概念。

> [!div class="nextstepaction"]
> [基本 Bot 概念](../v4sdk/bot-builder-basics.md)
