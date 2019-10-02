---
title: 使用檢查中介軟體來對聊天機器人進行偵錯 | Microsoft Docs
description: 了解如何使用檢查中介軟體來對聊天機器人進行偵錯
author: zxyanliu
ms.author: v-liyan
keywords: Bot Framework SDK, 對聊天機器人進行偵錯, 檢查中介軟體, 聊天機器人模擬器, Azure Bot 通道註冊
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 7/9/2019
ms.openlocfilehash: 4a3ff1ef255b914a30c10f6ebd070b7ca98d2f86
ms.sourcegitcommit: e9cd857ee11945ef0b98a1ffb4792494dfaeb126
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/01/2019
ms.locfileid: "71693125"
---
# <a name="debug-a-bot-with-inspection-middleware"></a>使用檢查中介軟體來對聊天機器人進行偵錯
本文說明如何使用檢查中介軟體對 Bot 進行偵錯。 除了查看 Bot 目前的狀態以外，這項功能也可讓 Bot Framework Emulator 偵測 Bot 的輸入和輸出流量。 您可以使用追蹤訊息將資料傳送給模擬器，然後在任何指定的對話回合中檢查聊天機器人的狀態。 

我們會使用透過 Bot Framework v4 ([C#](https://docs.microsoft.com/azure/bot-service/dotnet/bot-builder-dotnet-sdk-quickstart?view=azure-bot-service-4.0) | [JavaScript](https://docs.microsoft.com/azure/bot-service/javascript/bot-builder-javascript-quickstart?view=azure-bot-service-4.0)) 在本機建立的 EchoBot，來說明如何偵測和檢查 Bot 的訊息狀態。 您也可以[使用 IDE 對 Bot 進行偵錯](./bot-service-debug-bot.md)或[使用 Bot Framework Emulator 進行偵錯](./bot-service-debug-emulator.md)，但若要對狀態進行偵錯，則必須在 Bot 內新增檢查中介軟體。 這裡有提供檢查聊天機器人的範例：[C#](https://github.com/microsoft/BotBuilder-Samples/tree/master/samples/csharp_dotnetcore/47.inspection) 和 [JavaScript](https://github.com/microsoft/BotBuilder-Samples/tree/master/samples/javascript_nodejs/47.inspection)。 

## <a name="prerequisites"></a>必要條件
- 下載並安裝 [Bot Framework Emulator](https://aka.ms/Emulator-wiki-getting-started)
- 了解聊天機器人的[中介軟體](https://docs.microsoft.com/en-us/azure/bot-service/bot-builder-concept-middleware?view=azure-bot-service-4.0)
- 了解聊天機器人的[管理狀態](https://docs.microsoft.com/en-us/azure/bot-service/bot-builder-concept-state?view=azure-bot-service-4.0)
- 下載並安裝 [ngrok](https://ngrok.com/) (如果您想要對 Azure 中設定為使用其他通道的聊天機器人進行偵錯)

## <a name="update-your-emulator-to-the-latest-version"></a>將模擬器更新為最新版本 
在使用聊天機器人檢查中介軟體來對聊天機器人進行偵錯之前，您必須先將模擬器更新為 4.5 版或更新版本。 檢查[最新版本](https://github.com/Microsoft/BotFramework-Emulator/releases)以獲得更新。 

若要檢查模擬器的版本，請選取功能表中的 [說明]   -> [關於]  。 您會看到模擬器目前的版本。 

![目前版本](./media/bot-debug-inspection-middleware/bot-debug-check-emulator-version.png) 

## <a name="update-your-bots-code"></a>更新聊天機器人的程式碼

# <a name="ctabcsharp"></a>[C#](#tab/csharp)
在 **Startup** 檔案中設定檢查狀態。 將檢查中介軟體新增至配接器。 檢查狀態會透過插入相依性來提供。 請參閱下面的程式碼更新，或參閱此處的檢查範例：[C#](https://github.com/microsoft/BotBuilder-Samples/tree/master/samples/csharp_dotnetcore/47.inspection)。 

**Startup.cs**  
[!code-csharp [inspection bot sample](~/../botbuilder-samples/samples/csharp_dotnetcore/47.inspection/Startup.cs?range=17-37)]

**AdapterWithInspection.cs**  
[!code-csharp [inspection bot sample](~/../botbuilder-samples/samples/csharp_dotnetcore/47.inspection/AdapterwithInspection.cs?range=11-21)]

**EchoBot.cs**  
[!code-csharp [inspection bot sample](~/../botbuilder-samples/samples/csharp_dotnetcore/47.inspection/Bots/EchoBot.cs?range=14-43)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)
在更新聊天機器人的程式碼之前，請先在終端機執行下列命令，以將聊天機器人的套件更新為最新版本： 
```cmd
npm install --save botbuilder@latest 
```  
接著，您必須更新 JavaScript 聊天機器人的程式碼，如下所示。 您可以在這裡取得詳細資訊：[更新聊天機器人的程式碼](https://github.com/Microsoft/BotFramework-Emulator/blob/master/content/CHANNELS.md#1-update-your-bots-code)，或參閱此處的檢查範例：[JavaScript](https://github.com/microsoft/BotBuilder-Samples/tree/master/samples/javascript_nodejs/47.inspection)。 

**index.js**

設定檢查狀態，並將檢查中介軟體新增至 **index.js** 檔案中的配接器。 

[!code-javascript [inspection bot sample](~/../botbuilder-samples/samples/javascript_nodejs/47.inspection/index.js?range=10-43)]

**bot.js**

更新 **bot.js** 檔案中的聊天機器人類別。 

[!code-javascript [inspection bot sample](~/../botbuilder-samples/samples/javascript_nodejs/47.inspection/bot.js?range=6-50)]

---

## <a name="test-your-bot-locally"></a>在本機測試 Bot 
更新程式碼之後，您可以在本機執行聊天機器人，並使用兩個模擬器來測試偵錯功能：一個模擬器用來傳送和接收訊息，另一個則用來以偵測模式檢查訊息狀態。 若要在本機測試聊天機器人，請採取下列步驟： 

1. 在終端機流覽至聊天機器人的目錄，然後執行下列命令以在本機執行聊天機器人： 

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```cmd
dotnet run
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```cmd
npm start 
```

---

2. 開啟模擬器。 按一下 [開啟聊天機器人]  。 使用 http://localhost:3978/api/messages 和 **MicrosoftAppId** 與 **MicrosoftAppPassword** 值填入 [Bot URL]。 如果您有 JavaScript 聊天機器人，則可以在聊天機器人的 **.env** 檔案中找到這些值。 如果您有 C# 聊天機器人，則可以在 **appsettings.json** 檔案中找到這些值。 按一下 [ **連接**]。 

3. 現在開啟另一個模擬器。 第二個模擬器會作為偵錯工具。 遵循上一個步驟中所述的指示。 核取 [以偵錯模式開啟]  ，然後按一下 [連線]  。 

4. 至此，您會在偵錯工具模擬器中看到 UUID (`/INSPECT attach <identifier>`)。 複製 UUID，並將其貼至第一個模擬器的聊天方塊。 

> [!NOTE]
> 通用唯一識別碼 (UUID) 是用來識別資訊的唯一識別碼。 在您於聊天機器人的程式碼中新增檢查中介軟體之後，每次以偵錯模式啟動模擬器時，都會產生一個 UUID。 

5. 現在，您可以在第一個模擬器的聊天方塊中傳送訊息，然後在偵錯模擬器中檢查訊息。 若要檢查訊息的狀態，請在偵錯模擬器中按一下 [Bot 狀態]  ，然後在右邊的 [JSON]  視窗上展開 [值]  。 您將能夠看到聊天機器人的狀態，如下所示：![聊天機器人狀態](./media/bot-debug-inspection-middleware/bot-debug-bot-state.png)

## <a name="inspect-the-state-of-a-bot-configured-in-azure"></a>檢查在 Azure 中設定的 Bot 所處的狀態 
如果您想要檢查 Azure 中所設定且已連線至通道 (如 Teams) 的聊天機器人狀態，您必須安裝並執行 [ngrok](https://ngrok.com/)。

### <a name="run-ngrok"></a>執行 ngrok
至此，您已將模擬器更新為最新版本，並已在聊天機器人的程式碼中新增檢查中介軟體。 下一個步驟是執行 ngrok，並對 Azure Bot 通道註冊設定本機聊天機器人。 在執行 ngrok 之前，您必須在本機執行聊天機器人。 

若要在本機執行聊天機器人，請執行下列動作： 
1. 在終端機流覽至聊天機器人的資料夾，並將 npm 註冊設定為使用[最新組建](https://botbuilder.myget.org/feed/botbuilder-v4-js-daily/package/npm/botbuilder-azure)

2. 在本機執行您的 Bot。 您會看到聊天機器人公開一個連接埠號碼，例如 3978。 

3. 開啟另一個命令提示字元，並瀏覽至聊天機器人的專案資料夾。 執行以下命令：
```
ngrok http 3978
```
4. ngrok 現已連線至在本機執行的聊天機器人。 複製公用 IP 位址。 
![ngrok-success](./media/bot-debug-inspection-middleware/bot-debug-ngrok.png)

### <a name="update-channel-registrations-for-your-bot"></a>更新聊天機器人的通道註冊
現在，本機聊天機器人已連線至 ngrok，因此接下來您可以在 Azure 中對 Bot 通道註冊設定本機聊天機器人。

1. 移至 Azure 中的 Bot 通道註冊。 按一下左側功能表上的 [設定]  ，並使用 ngrok IP 來設定 [訊息端點]  。 如有必要，請在 IP 位址後面新增 **/api/messages**。 (例如： https://e58549b6.ngrok.io/api/messages) 。 核取 [啟用串流端點]  和 [儲存]  。
![endpoint](./media/bot-debug-inspection-middleware/bot-debug-channels-setting-ngrok.png)
> [!TIP]
> 如果 [儲存]  未啟用，則可以取消核取 [啟用串流端點]  ，按一下 [儲存]  ，然後再核取 [啟用串流端點]  並再次按一下 [儲存]  。 您必須確定 [啟用串流端點]  已核取，且端點的設定也已儲存。 

2. 移至聊天機器人的資源群組，按一下 [部署]  ，然後選取先前成功部署的 Bot 通道註冊。 按一下左側的 [輸入]  以取得 **appId** 和 **appSecret**。 使用 **appId** 和 **appSecret** 更新聊天機器人的 **.env** 檔案 (如果使用 C# 聊天機器人，則請更新 **appsettings.json** 檔案)。 
![get-inputs](./media/bot-debug-inspection-middleware/bot-debug-get-inputs-id-secret.png)

3. 啟動模擬器，按一下 [開啟聊天機器人]  ，然後將 http://localhost:3978/api/messages 放入 [Bot URL]  。 使用您新增至聊天機器人 **.env** (**appsettings.json**) 檔案的相同 **appId** 和 **appSecret** 來填入 [Microsoft 應用程式識別碼]  和 [Microsoft 應用程式密碼]  。 然後按一下 [ **連接**]。 

4. 執行中的聊天機器人現已連線至 Azure 中的 Bot 通道註冊。 您可以按一下 [在網路聊天中測試]  並在聊天方塊中傳送訊息，以測試網路聊天。 
![test-web-chat](./media/bot-debug-inspection-middleware/bot-debug-test-webchat.png)

5. 現在，讓我們在模擬器中啟用偵測模式。 在模擬器中選取 [偵錯]   -> [開始偵錯]  。 在 [Bot URL]  中填入 ngrok IP 位址 (例如 https://e58549b6.ngrok.io/api/messages) ) (別忘了新增 **/api/messages**)。 使用 **appId** 填入 [Microsoft 應用程式識別碼]  ，並使用 **appSecret** 填入 [Microsoft 應用程式密碼]  。 確定也已核取 [以偵錯模式開啟]  。 按一下 [ **連接**]。 

6. 偵錯模式啟用時，模擬器會產生 UUID。 UUID 是您每次在模擬器中啟動偵錯模式時，所會產生的唯一識別碼。 將 UUID 複製並貼到 [在網路聊天中測試]  聊天方塊或通道的聊天方塊。 您會在聊天方塊中看到「已連結至工作階段，正在複寫所有流量以進行檢查」的訊息。 

 您可以藉由在所設定通道的聊天方塊中傳送訊息，來開始對聊天機器人進行偵錯。 本機模擬器會自動以所有偵錯詳細資料來更新訊息。 若要檢查聊天機器人的訊息狀態，請按一下 [Bot 狀態]  ，然後在右邊的 [JSON] 視窗中展開 [值]  。 

 ![debug-inspection-middleware](./media/bot-debug-inspection-middleware/debug-state-inspection-channel-chat.gif)

## <a name="additional-resources"></a>其他資源
- 在此嘗試新的檢查聊天機器人範例：[C#](https://github.com/microsoft/BotBuilder-Samples/tree/master/samples/csharp_dotnetcore/47.inspection) 和 [JavaScript](https://github.com/microsoft/BotBuilder-Samples/tree/master/samples/javascript_nodejs/47.inspection)。 
- 請閱讀[針對一般問題進行疑難排解](bot-service-troubleshoot-bot-configuration.md)以及該區段中的其他疑難排解文章。
- 閱讀如何[使用模擬器進行偵錯](bot-service-debug-emulator.md)一文。
