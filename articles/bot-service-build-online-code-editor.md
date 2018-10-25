---
title: 使用 Azure 線上程式碼編輯器建置 Bot | Microsoft Docs
description: 了解如何使用 Bot 服務中的線上程式碼編輯器建置您的 Bot。
keywords: online code editor, azure portal, functions bot, 線上程式碼編輯器, Azure 入口網站, Functions Bot
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: tools
ms.date: 09/21/2018
ms.openlocfilehash: ba8e87073f888f21afb806221108bd6b5391744c
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/24/2018
ms.locfileid: "49997235"
---
# <a name="edit-a-bot-with-online-code-editor"></a>使用線上程式碼編輯器來編輯 Bot

您可以使用線上程式碼編輯器來建置 Bot，而不需要使用 IDE。 此主題將說明如何在線上程式碼編輯器開啟您的 Bot 程式碼。 

## <a name="edit-bot-source-code-in-online-code-editor"></a>在線上程式碼編輯器中編輯 Bot 原始程式碼

若要在線上程式碼編輯器中編輯 Bot 的原始程式碼，請依照您擁有的特定 Bot 類型來執行下列動作。

### <a name="web-app-bot"></a>Web 應用程式 Bot
1. 登入 [Azure 入口網站](http://portal.azure.com)，並開啟 Bot 的刀鋒視窗。
2. 在 [Bot 管理] 區段下，按一下 [建置]。
3. 按一下 [開啟線上程式碼編輯器]。 這將會在新的瀏覽器視窗中開啟 Bot 的程式碼。 

   ![開啟線上程式碼編輯器](~/media/azure-bot-build/open-online-code-editor.png)

   視 Bot 的程式設計語言而定，[WWWRoot] 目錄下的檔案結構會有所不同。 例如，如果您有 C# Bot，則您的 **WWWRoot** 可能看起來像這樣：

   ![C# 檔案結構](~/media/azure-bot-build/cs-wwwroot-structure.png)

   如果您有 Node.js Bot，則 **WWWRoot** 可能看起來像這樣：

   ![Node.js 檔案結構](~/media/azure-bot-build/node-wwwroot-structure.png)

4. 變更程式碼。 例如，若為 C# Bot，您可以使用 .cs 檔案。 針對 Node.js Bot，您可以從 App.js 檔案開始。

   > [!NOTE]
   > 雖然您可以對專案中目前的原始程式檔進行程式碼變更，但無法使用線上程式碼編輯器來建立新的原始程式檔。 若要將新的原始程式檔新增到 Bot，您需要[下載來源](bot-service-build-download-source-code.md)專案、新增您的檔案，然後將變更發佈回 Azure。

5. 針對在**使用情況方案**上的 C# Bot 和所有的 Node.js Bot，系統會自動儲存 Bot。 

6. 針對在 **App Service** 方案上的 C# Bot，開啟 [主控台] 刀鋒視窗，並傳送 **build.cmd** 命令。 

   ![在主控台刀鋒視窗中建置專案](~/media/azure-bot-build/cs-console-build-cmd.png)
 
   > [!NOTE]
   > 如果此命令無法建置，請嘗試重新啟動 Bot 的應用程式服務，然後再次嘗試建置。 若要重新啟動應用程式服務，請按一下 Bot 刀鋒視窗中的 [所有應用程式服務設定]，然後按一下 [重新啟動] 按鈕。
   > ![重新啟動 Web 應用程式](~/media/azure-bot-build/open-online-code-editor-restart-appservice.png)

7. 切換回 Azure 入口網站，按一下 [在 Web 聊天中測試] 以測試您的變更。 如果您已經開啟此 Bot 的 [Web 聊天]，請按一下 [重新開始] 來查看新的變更。

### <a name="functions-bot"></a>Functions Bot

1. 登入 [Azure 入口網站](http://portal.azure.com)，並開啟 Bot 的刀鋒視窗。
2. 在 [Bot 管理] 區段下，按一下 [建置]。
3. 按一下 [在 Azure Functions 中開啟此 Bot]。 這會開啟 Bot 並顯示 <a href="http://go.microsoft.com/fwlink/?linkID=747839" target="_blank">Azure Functions</a> UI。 
4. 變更程式碼。 例如，更新函式的 messages (訊息) 程式碼。 以下的螢幕擷取畫面顯示 Node.js Functions Bot 的 Messages (訊息) 程式碼。

   ![Functions Bot Messages (訊息) 程式碼編輯器](~/media/azure-bot-build/functions-messages-code.png)

5. 儲存您的程式碼變更。
6. 切換回 Azure 入口網站，按一下 [在 Web 聊天中測試] 以測試您的變更。 如果您已經開啟此 Bot 的 [Web 聊天]，請按一下 [重新開始] 來查看新的變更。

## <a name="next-steps"></a>後續步驟
既然您已經知道如何使用線上程式碼編輯器來編輯您的 Bot 程式碼，您也可以使用您最愛的 IDE 在本機上建置自己的 Bot。

> [!div class="nextstepaction"]
> [下載 Bot 原始程式碼](bot-service-build-download-source-code.md)
