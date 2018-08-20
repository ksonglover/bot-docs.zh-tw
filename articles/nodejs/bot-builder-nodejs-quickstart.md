---
title: 使用適用於 Node.js 的 Bot 建立器 SDK 建立 Bot | Microsoft Docs
description: 使用適用於 Node.js 的 Bot 建立器 SDK 建立 Bot，Bot 建立器 SDK 是功能強大的 Bot 建構架構。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: get-started-article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 6159997ec5ea3dbd3188ba2ea4b6207b5d9db08f
ms.sourcegitcommit: 97bb24f15041caccef4ca5736aa14f144881e0c6
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/06/2018
ms.locfileid: "39567547"
---
# <a name="create-a-bot-with-the-bot-builder-sdk-for-nodejs"></a>使用適用於 Node.js 的 Bot 建立器 SDK 建立 Bot

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-quickstart.md)
> - [Node.js](../nodejs/bot-builder-nodejs-quickstart.md)
> - [Bot Service](../bot-service-quickstart.md)
> - [REST](../rest-api/bot-framework-rest-connector-quickstart.md)

適用於 Node.js 的 Bot 建立器 SDK 是用來開發 Bot 架構。 除了容易使用外，此架構以 Express 和 Restify 等架構作為範本，讓 JavaScript 開發人員可以使用熟悉的方式來撰寫 Bot。

本教學課程會引導您使用適用於 Node.js 的 Bot 建立器 SDK 來建置 Bot。 您可以在主控台視窗中使用 Bot Framework 模擬器來測試 Bot。

## <a name="prerequisites"></a>必要條件
一開始請先完成下列必要工作：

1. 安裝 [Node.js](https://nodejs.org)。
2. 建立 Bot 的資料夾。
3. 從命令提示字元或終端機，巡覽至您剛剛建立的資料夾。
4. 執行下列 **npm** 命令：

```nodejs
npm init
```

依照畫面上的提示來輸入 Bot 相關資訊，然後 npm 會建立 **package.json** 檔案，其中包含您所提供的資訊。 

## <a name="install-the-sdk"></a>安裝 SDK
接著，執行下列 **npm** 命令來安裝適用於 Node.js 的 Bot 建立器 SDK：

```nodejs
npm install --save botbuilder
```

安裝好 SDK 後，您就可以開始撰寫第一個 Bot。

針對第一個 Bot，您將建立會直接回應任何使用者輸入的 Bot。 若要建立 Bot，請遵循下列步驟：

1. 在您稍早建立的 Bot 資料夾中，建立名為 **app.js** 的新檔案。
2. 在您偏好的文字編輯器或 IDE 中開啟 **app.js**。 將下列程式碼新增至該檔案： 

   [!code-javascript[consolebot code sample Node.js](../includes/code/node-getstarted.js#consolebot)]

3. 儲存檔案。 現在您已準備好執行並測試您的 Bot。

### <a name="start-your-bot"></a>啟動 Bot

在主控台視窗中瀏覽至您的 Bot 目錄，並啟動 Bot：

```nodejs
node app.js
```

Bot 此時將在本機執行。 在主控台視窗中輸入一些訊息以測試您的 Bot。
您應該會看到 Bot 透過在您訊息前方加上「您說了：」文字，來回應您送出的每個訊息。

## <a name="install-restify"></a>安裝 Restify

主控台 Bot 是很好的文字型用戶端，但若要可使用所有 Bot Framework 通道 (或在模擬器中執行 Bot)，Bot 將需要在 API 端點上執行。 執行下列 **npm** 命令來安裝 <a href="http://restify.com/" target="_blank">restify</a>：

```nodejs
npm install --save restify
```

安裝好 Restify 後，即可對 Bot 進行一些變更。

## <a name="edit-your-bot"></a>編輯 Bot

您必須對 **app.js** 檔案進行一些變更。 

1. 新增一行命令來要求 `restify` 模組。
2. 將 `ConsoleConnector` 變更為 `ChatConnector`。
3. 包含您的 Microsoft 應用程式識別碼和應用程式密碼。
4. 讓連接器在 API 端點上接聽。

   [!code-javascript[echobot code sample Node.js](../includes/code/node-getstarted.js#echobot)]

5. 儲存檔案。 現在您已準備好在模擬器中執行並測試您的 Bot。

> [!NOTE] 
> 在 Bot Framework 模擬器中執行 Bot 時，不需要用到 **Microsoft 應用程式識別碼**或 **Microsoft 應用程式密碼**。

## <a name="test-your-bot"></a>測試 Bot
接下來，使用 [Bot Framework 模擬器](../bot-service-debug-emulator.md)來測試 Bot，以觀察 Bot 的運作。 模擬器是桌面應用程式，可讓您在 localhost 上測試和偵錯您的 Bot，或透過通道從遠端執行 Bot。

首先，必須[下載](https://emulator.botframework.com)並安裝模擬器。 下載完成後，啟動可執行檔並完成安裝程序。

### <a name="start-your-bot"></a>啟動 Bot

安裝模擬器之後，在主控台視窗中瀏覽至您的 Bot 目錄，並啟動 Bot：

```nodejs
node app.js
```
   
Bot 此時將在本機執行。

### <a name="start-the-emulator-and-connect-your-bot"></a>啟動模擬器並且連線到您的 Bot
啟動 Bot 之後，啟動模擬器並連線到您的 Bot：

1. 按一下模擬器視窗中的 [建立新的 Bot 組態] 連結。 

2. 將 `http://localhost:port-number/api/messages` 輸入網址列，其中「連接埠號碼」必須符合應用程式執行所在瀏覽器中顯示的連接埠號碼。

3. 按一下儲存並連線。 您不需要指定 Microsoft 應用程式識別碼和 Microsoft 應用程式密碼。 目前可以將這些欄位保留空白。 稍後註冊 Bot 時，您會取得這項資訊。

### <a name="try-out-your-bot"></a>試用 Bot

現在，您的 Bot 會在本機執行，並且已連線到模擬器，請在模擬器中輸入一些訊息來試用 Bot。
您應該會看到 Bot 透過在您訊息前方加上「您說了：」文字，來回應您送出的每個訊息。

您已使用適用於 Node.js 的 Bot 建立器 SDK 成功建立第一個 Bot！

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [適用於 Node.js 的 Bot 建立器 SDK](bot-builder-nodejs-overview.md)
