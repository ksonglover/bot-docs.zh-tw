---
title: 對使用 Bot Service 建置的 Bot 進行偵錯 | Microsoft Docs
description: 了解如何對使用 Bot Service 建置的 Bot 進行偵錯。
author: v-ducvo
ms.author: v-ducvo
keywords: Bot 建立器 SDK, 持續部署, 應用程式服務, 模擬器
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 07/13/2018
ms.openlocfilehash: 73cf6cedecd9acaef828bd41f4fccdaad2ae5731
ms.sourcegitcommit: 3bf3dbb1a440b3d83e58499c6a2ac116fe04b2f6
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/23/2018
ms.locfileid: "46707164"
---
# <a name="debug-a-bot-service-bot"></a>對 Bot Service Bot 進行偵錯

本文說明如何使用整合式開發環境 (IDE)，例如 Visual Studio 或 Visual Studio Code 和 Bot Framework 模擬器等，對 Bot 進行偵錯。 儘管您可以使用這些方法對任何 Bot 進行本機偵錯，本文會使用在[使用 Bot Service 建立 Bot](bot-service-quickstart.md) 一文中所建立的 **EchoBot**。

## <a name="debug-a-javascript-bot"></a>對 JavaScript Bot 進行偵錯

請遵循此節步驟，對以 JavaScript 撰寫的 Bot 進行偵錯。

### <a name="prerequisites"></a>必要條件

在對 JavaScript Bot 進行偵錯前，您必須先完成這些工作。

- (從 Azure) 下載 Bot 的原始程式碼，如[下載 Bot 原始程式碼](bot-service-build-download-source-code.md)中所述。
- 下載並安裝 [Bot Framework 模擬器](https://github.com/Microsoft/BotFramework-Emulator/releases) (英文)。
- 下載並安裝程式碼編輯器，如 <a href="https://code.visualstudio.com" target="_blank">Visual Studio Code</a> (英文)。

### <a name="debug-a-javascript-bot-using-command-line-and-emulator"></a>使用命令列和模擬器對 JavaScript Bot 進行偵錯

若要使用命令列執行 JavaScript Bot 並使用模擬器測試 Bot，請執行下列操作：
1. 從命令列將目錄變更至 Bot 專案目錄。
2. 執行命令 **node app.js** 以啟動 Bot。
3. 啟動模擬器，然後連線到 Bot 的端點 (例如：**http://localhost:3978/api/messages**)。 如果這是您第一次執行 Bot，請按一下 [檔案] > [新的 Bot]，並遵循畫面上的指示。 否則，請按一下 [檔案] > [開啟 Bot] 以開啟現有的 Bot。 由於這個 Bot 在本機電腦上執行，您可以將 **MSA 應用程式識別碼**和 **MSA 應用程式密碼**欄位留白。 如需詳細資訊，請參閱[使用模擬器進行偵錯](bot-service-debug-emulator.md)。
4. 在模擬器中，向 Bot 傳送訊息 (例如：傳送「嗨」訊息)。 
5. 使用模擬器視窗右側的 [偵測器] 和 [記錄] 面板來對您的 Bot 進行偵錯。 比方說，按一下任何一個訊息泡泡 (例如：在下方螢幕擷取畫面中的「嗨」訊息泡泡)，在 [偵測器] 面板中會向您顯示該訊息的詳細資料。 您可以用它來檢視要求和回應，因訊息會在模擬器和 Bot 之間交換。 或者，您也可以按一下 [記錄] 面板中任何連結的文字，以在 [偵測器] 面板中檢視詳細資料。

   ![模擬器上的 [偵測器] 面板](~/media/bot-service-debug-bot/emulator_inspector.png)

### <a name="debug-a-javascript-bot-using-breakpoints-in-visual-studio-code"></a>使用 Visual Studio Code 的中斷點對 JavaScript Bot 進行偵錯

使用 Visual Studio Code 等整合式開發環境，您可以設定中斷點，並在偵錯模式中執行 Bot 以逐步執行程式碼。 若要在 VS Code 中設定中斷點，請執行下列操作：

1. 啟動 VS Code 並開啟 Bot 專案資料夾。
2. 在功能表列中，按一下 [偵錯] 然後按一下 [開始偵錯]。 如果系統提示您選取執行階段引擎來執行程式碼，請選取 **Node.js**。 此時，Bot 正在本機執行。 

   > [!NOTE]
   > 如果您收到「值不能為 Null」錯誤訊息，請檢查以確定**表格儲存體**設定有效。
   > 使用**表格儲存體**時的預設值為 **EchoBot**。 若要在 Bot 中使用表格儲存體，您需要「表格名稱」和「金鑰」。 若您尚未準備好表格儲存體執行個體，您可以建立一個，或者基於測試目的，可註解排除使用 **TableBotDataStore** 的程式碼，並取消註解使用 **InMemoryDataStore** 的程式碼。 **InMemoryDataStore** 僅適用於測試和原型設計。

3. 視需要設定中斷點。 在 VS Code 中，您可以將滑鼠停留在行號左側的資料行來設定中斷點。 會出現一個小紅點。 按一下小紅點，即可設定中斷點。 如果再按一下，則會移除中斷點。

   ![在 VS Code 中設定中斷點](~/media/bot-service-debug-bot/breakpoint-set.png)

4. 啟動 Bot Framework 模擬器並連接到您的 Bot，如上一節所述。 
5. 在模擬器中，向 Bot 傳送訊息 (例如：傳送「嗨」訊息)。 執行會停在您放置中斷點的那一行。

   ![在 VS Code 中進行偵錯](~/media/bot-service-debug-bot/breakpoint-caught.png)


## <a name="debug-a-c-bot"></a>對 C# Bot 進行偵錯

請遵循此節步驟，對以 C# 撰寫的 Bot 進行偵錯。


### <a name="prerequisites"></a>必要條件

在對 Web 應用程式 C# Bot 進行偵錯前，您必須先完成這些工作。

- (從 Azure) 下載 Bot 的原始程式碼，如[下載 Bot 原始程式碼](bot-service-build-download-source-code.md)中所述。
- 下載並安裝 [Bot Framework 模擬器](https://github.com/Microsoft/BotFramework-Emulator/releases) (英文)。
- 下載並安裝 <a href="https://www.visualstudio.com/downloads/" target="_blank">Visual Studio 2017</a> (Community 版本或更新版本)。

### <a name="debug-a-c-bot-using-breakpoints-in-visual-studio"></a>使用 Visual Studio 的中斷點對 C# Bot 進行偵錯

使用 Visual Studio (VS) 等整合式開發環境，您可以設定中斷點，並在偵錯模式中執行 Bot 以逐步執行程式碼。 若要在 VS 中設定中斷點，請執行下列操作：

1. 瀏覽至您的 Bot 資料夾，然後開啟 **.sln** 檔案。 這會在 VS 中開啟該解決方案。
2. 按一下功能表列中的 [建置]，然後按一下 [建置解決方案]。
3. 在 [方案總管] 中，展開 [對話方塊] 資料夾，然後按一下 [EchoDialog.cs]。 此檔案會定義主要的 Bot 邏輯。
4. 在功能表列中，按一下 [偵錯] 然後按一下 [開始偵錯]。 此時，Bot 正在本機執行。 

   > [!NOTE]
   > 如果您收到「值不能為 Null」錯誤訊息，請檢查以確定**表格儲存體**設定有效。
   > 使用**表格儲存體**時的預設值為 **EchoBot**。 若要在 Bot 中使用表格儲存體，您需要「表格名稱」和「金鑰」。 若您尚未準備好表格儲存體執行個體，您可以建立一個，或者基於測試目的，可註解排除使用 **TableBotDataStore** 的程式碼，並取消註解使用 **InMemoryDataStore** 的程式碼。 **InMemoryDataStore** 僅適用於測試和原型設計。

5. 視需要設定中斷點。 在 VS 中，您可以將滑鼠停留在行號左側的資料行來設定中斷點。 會出現一個小紅點。 按一下小紅點，即可設定中斷點。 如果再按一下，則會移除中斷點。

   ![在 VS 中設定中斷點](~/media/bot-service-debug-bot/breakpoint-set-vs.png)

6. 啟動 Bot Framework 模擬器並連接到您的 Bot，如上一節所述。 
7. 在模擬器中，向 Bot 傳送訊息 (例如：傳送「嗨」訊息)。 執行會停在您放置中斷點的那一行。

   ![在 VS 中進行偵錯](~/media/bot-service-debug-bot/breakpoint-caught-vs.png)

::: moniker range="azure-bot-service-3.0" 

## <a id="debug-csharp-serverless"></a> 針對取用方案 C 進行偵錯\# Functions Bot

比起一般的 C\# 應用程式，Bot Service 中的取用方案無伺服器 C\# 環境與 Node.js 更相似，因為它需要執行階段主機，正如同 Node 引擎。 在 Azure 中，執行階段是雲端裝載環境的一部分，但您必須在本機桌面上複寫該環境。 

### <a name="prerequisites"></a>必要條件

在對取用方案 C# Bot 進行偵錯前，您必須先完成這些工作。

- 下載 Bot 的原始程式碼 (從 Azure)，如[設定持續部署](bot-service-continuous-deployment.md)中所述。
- 下載並安裝 [Bot Framework 模擬器](https://github.com/Microsoft/BotFramework-Emulator/releases) (英文)。
- 安裝 <a href="https://www.npmjs.com/package/azure-functions-cli" target="_blank">Azure Functions CLI</a> (英文)。
- 安裝 <a href="https://github.com/dotnet/cli" target="_blank">DotNet CLI</a> (英文)。
  
如果您想要在 Visual Studio 2017 中使用中斷點對程式碼進行偵錯，也必須完成下列工作。
  
- 下載並安裝 <a href="https://www.visualstudio.com/downloads/" target="_blank">Visual Studio 2017</a> (Community 版本或更新版本)。
- 下載並安裝<a href="https://visualstudiogallery.msdn.microsoft.com/e6bf6a3d-7411-4494-8a1e-28c1a8c4ce99" target="_blank">命令工作執行器 Visual Studio 擴充功能</a> (英文)。

> [!NOTE]
> 目前不支援 Visual Studio Code。

### <a name="debug-a-consumption-plan-c-functions-bot-using-the-emulator"></a>使用模擬器對取用方案 C# Functions Bot 進行偵錯

在本機對 Bot 進行偵錯最簡單的方法是，啟動 Bot 然後從 Bot Framework 模擬器連線到 Bot。 
首先，開啟命令提示字元並瀏覽至存放庫中 **project.json** 檔案所在的資料夾。 然後，執行命令 `dotnet restore` 以還原 Bot 中所參考的各種套件。

> [!NOTE]
> Visual Studio 2017 變更了 Visual Studio 處理相依性的方式。 Visual Studio 2015 使用 **project.json** 來處理相依性，而 Visual Studio 2017 在 Visual Studio 中載入時則是使用 **.csproj** 模型。 如果您使用 Visual Studio 2017，<a href="https://aka.ms/bf-debug-project">請將此 **.csproj** 檔案下載</a>至存放庫中的 **/messages** 資料夾，然後再執行 `dotnet restore` 命令。

![命令提示字元](~/media/bot-service-debug-bot/csharp-azureservice-debug-envconfig.png)

接下來，執行 `debughost.cmd` 以載入並啟動您的 Bot。 

![命令提示字元會執行 debughost.cmd](~/media/bot-service-debug-bot/csharp-azureservice-debug-debughost.png)

此時，Bot 正在本機執行。 在主控台視窗中，複製 debughost 正在接聽的端點 (此範例中是 `http://localhost:3978`)。 然後，啟動 Bot Framework 模擬器，並將端點貼入模擬器的網址列。 針對此範例，您也必須將 `/api/messages` 附加至端點。 由於您不需要本機偵錯的安全性，您可以將 **Microsoft 應用程式識別碼**和 **Microsoft 應用程式密碼**欄位留白。 按一下 [連線]，使用指定端點建立到您 Bot 的連線。

![設定模擬器](~/media/bot-service-debug-bot/mac-azureservice-emulator-config.png)

將模擬器連接到 Bot 之後，在位於模擬器視窗底部的文字方塊 (亦即左下角會出現**輸入您的訊息...**) 中輸入一些文字，以傳送訊息至 Bot。 透過使用模擬器視窗右側的 [記錄] 和 [偵測器] 面板，您可以檢視要求和回應，因訊息會在模擬器和 Bot 之間交換。

![透過模擬器進行測試](~/media/bot-service-debug-bot/mac-azureservice-debug-emulator.png)

此外，您也可以在主控台視窗中檢視記錄詳細資料。

![主控台視窗](~/media/bot-service-debug-bot/csharp-azureservice-debug-debughostlogging.png)

::: moniker-end

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [使用模擬器進行偵錯](bot-service-debug-emulator.md)。
