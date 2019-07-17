---
ms.openlocfilehash: 4af367b04f84d935936b5752cf9dbc863430105c
ms.sourcegitcommit: fa6e775dcf95a4253ad854796f5906f33af05a42
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/16/2019
ms.locfileid: "68230807"
---
## <a name="prerequisites"></a>必要條件

- [Visual Studio Code](https://www.visualstudio.com/downloads)
- [Node.js](https://nodejs.org/)
- [Yeoman](http://yeoman.io/)，可使用產生器為您建立 Bot
- [git](https://git-scm.com/)
- [Bot Framework 模擬器](https://aka.ms/bot-framework-emulator-readme) (英文)
- 了解 [restify](http://restify.com/) 和 JavaScript 中的非同步程式設計

> [!NOTE]
> 只有在您使用 Windows 作為開發作業系統時，才需要安裝以下所列的 Windows 建置工具。
> 在某些安裝中，restify 的安裝步驟會產生與 node-gyp 相關的錯誤。
> 如果情況如此，您可嘗試以提升的權限執行此命令。
> 如果您的系統上已安裝 python，此呼叫可能也會停止回應，但不會結束：

> ```bash
> # only run this command if you are on Windows. Read the above note. 
> npm install -g windows-build-tools
> ```

## <a name="create-a-bot"></a>建立 Bot

建立 Bot 並初始化其套件

1. 開啟終端機或提升權限的命令提示字元。
1. 如果您的 JavaScript Bot 還沒有目錄，請加以建立並切換到該目錄。 (即使我們在本教學課程中只建立一個 Bot，但是在一般情況下，我們會為 JavaScript Bot 建立目錄。)

   ```bash
   mkdir myJsBots
   cd myJsBots
   ```

1. 確保您的 npm 版本是最新版本。

   ```bash
   npm install -g npm
   ```

1. 接下來，安裝適用於 JavaScript 的 Yeoman 和產生器。

   ```bash
   npm install -g yo generator-botbuilder
   ```

1. 然後，使用產生器來建立 echo Bot。

   ```bash
   yo botbuilder
   ```

Yeoman 會提示您輸入一些用來建立 Bot 的資訊。 本教學課程使用預設值。

- 輸入 Bot 的名稱。 (my-chat-bot)
- 輸入描述。 (示範 Microsoft Bot Framework 的核心功能)
- 選擇 Bot 的語言。 (JavaScript)
- 選擇要使用的範本。 (回應聊天機器人 - https://aka.ms/bot-template-echo)

由於有範本，專案中會包含要在本快速入門建立聊天機器人所需的所有程式碼。 您實際上不需要撰寫任何額外的程式碼。

> [!NOTE]
> 如果選擇建立 `Core` Bot，您需要 LUIS 語言模型。 您可以在 [luis.ai](https://www.luis.ai) 上建立一個模型。 建立模型之後，請更新設定檔。

## <a name="start-your-bot"></a>啟動 Bot

在終端機或命令提示字元中，將目錄變更為針對您的 Bot 所建立的目錄，並以 `npm start` 開頭。 此時，Bot 正在本機執行。

## <a name="start-the-emulator-and-connect-your-bot"></a>啟動模擬器並連線至您的 Bot

1. 啟動 Bot Framework 模擬器。
2. 按一下模擬器 [歡迎使用] 索引標籤中的 [建立新的 Bot 設定]  連結。 
3. 填寫聊天機器人的欄位。 使用聊天機器人的歡迎頁面網址 (通常為 http://localhost:3978) )，並在此網址附加路由資訊 '/api/messages'。
4. 然後按一下 [儲存並連線]  。

傳送訊息給 Bot，Bot 就會以訊息回應。
![模擬器執行中](../media/emulator-v4/js-quickstart.png)
