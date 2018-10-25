---
title: 使用適用於 JavaScript 的 Bot 建立器 SDK 建立 Bot | Microsoft Docs
description: 使用適用於 JavaScript 的 Bot 建立器 SDK 快速建立 Bot。
keywords: 快速入門, Bot 建立器 sdk, 開始使用
author: jonathanfingold
ms.author: jonathanfingold
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 09/23/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 3e721c9142ffb1511ef926f5b1caca782e0919ed
ms.sourcegitcommit: 54ed5000c67a5b59e23b667547565dd96c7302f9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/13/2018
ms.locfileid: "49315084"
---
# <a name="create-a-bot-with-the-bot-builder-sdk-for-javascript"></a>使用適用於 JavaScript 的 Bot Builder SDK 建立 Bot

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

本快速入門會逐步引導您使用 Yeoman Bot 建立器 產生器和適用於 JavaScript 的 Bot 建立器 SDK 建置 Bot，然後使用 Bot Framework 模擬器進行測試。 

## <a name="prerequisites"></a>必要條件

- [Visual Studio Code](https://www.visualstudio.com/downloads)
- [Node.js](https://nodejs.org/)
- [Yeoman](http://yeoman.io/)，可使用產生器為您建立 Bot
- [Bot Framework 模擬器](https://github.com/Microsoft/BotFramework-Emulator) (英文)
- 了解 [restify](http://restify.com/) 和 JavaScript 中的非同步程式設計

> [!NOTE]
> 在某些安裝中，restify 的安裝步驟會產生與 node-gyp 相關的錯誤。
> 如果您遇到這種情況，請嘗試執行 `npm install -g windows-build-tools`。

## <a name="create-a-bot"></a>建立 Bot

開啟提升權限的命令提示字元、建立目錄，然後為 Bot 初始化套件。

```bash
md myJsBots
cd myJsBots
```

確保您的 npm 版本是最新版本。
```bash
npm i npm
```

接下來，安裝適用於 JavaScript 的 Yeoman 和產生器。

```bash
npm install -g yo
npm install -g generator-botbuilder
```

然後，使用產生器來建立 echo Bot。

```bash
yo botbuilder
```

Yeoman 會提示您輸入一些用來建立 Bot 的資訊。

- 輸入 Bot 的名稱。
- 輸入描述。
- 選擇 Bot 的語言，可以是 `JavaScript` 或 `TypeScript`。
- 選擇 `Echo` 範本。

由於有範本，專案中會包含要在本快速入門建立 Bot 所需的所有程式碼。 您實際上不需要撰寫任何額外的程式碼。

> [!NOTE]
> 對於基本 Bot，您需要 LUIS 語言模型。 您可以在 [luis.ai](https://www.luis.ai) 上建立一個模型。 建立模型之後，請更新 .bot 檔案。 您的 Bot 檔案應該看起來類似這[一個](../v4sdk/bot-builder-service-file.md)。 

## <a name="start-your-bot"></a>啟動 Bot

在 Powershell/Bash 中，將目錄變更為針對您的 Bot 所建立的目錄，並使用 `npm start` 加以啟動。 此時，Bot 正在本機執行。

## <a name="start-the-emulator-and-connect-your-bot"></a>啟動模擬器並且連線至您的 Bot
1. 啟動模擬器。
2. 按一下模擬器 [歡迎] 索引標籤中的 [開啟 Bot] 連結。
3. 選取位於您建立專案的目錄中的 .bot 檔案。

傳送訊息給 Bot，Bot 就會以訊息回應。
![模擬器執行中](../media/emulator-v4/emulator-running.png)

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [Bot 的運作方式](../v4sdk/bot-builder-basics.md) 
