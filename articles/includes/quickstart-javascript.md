---
ms.openlocfilehash: 04b015963b8ea991b87f085dd5d6aa0110c50a18
ms.sourcegitcommit: bdb981c0b11ee99d128e30ae0462705b2dae8572
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/17/2019
ms.locfileid: "54360808"
---
## <a name="prerequisites"></a>必要條件

- [Visual Studio Code](https://www.visualstudio.com/downloads)
- [Node.js](https://nodejs.org/)
- [Yeoman](http://yeoman.io/)，可使用產生器為您建立 Bot
- [git](https://git-scm.com/)
- [Bot Framework 模擬器](https://github.com/Microsoft/BotFramework-Emulator) (英文)
- 了解 [restify](http://restify.com/) 和 JavaScript 中的非同步程式設計

> [!NOTE]
> 在某些安裝中，restify 的安裝步驟會產生與 node-gyp 相關的錯誤。
> 如果情況如此，請嘗試以提升的權限執行此命令：
> ```bash
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

- 輸入 Bot 的名稱。 (myChatBot)
- 輸入描述。 (示範 Microsoft Bot Framework 的核心功能)
- 選擇 Bot 的語言。 (JavaScript)
- 選擇要使用的範本。 (Echo)

由於有範本，專案中會包含要在本快速入門建立 Bot 所需的所有程式碼。 您實際上不需要撰寫任何額外的程式碼。

> [!NOTE]
> 如果選擇建立 `Basic` Bot，您需要 LUIS 語言模型。 您可以在 [luis.ai](https://www.luis.ai) 上建立一個模型。 建立模型之後，請更新 .bot 檔案。 您的 Bot 檔案應該看起來類似這[一個](../v4sdk/bot-builder-service-file.md)。

## <a name="start-your-bot"></a>啟動 Bot

在終端機或命令提示字元中，將目錄變更為針對您的 Bot 所建立的目錄，並以 `npm start` 開頭。 此時，Bot 正在本機執行。

## <a name="start-the-emulator-and-connect-your-bot"></a>啟動模擬器並連線至您的 Bot

1. 啟動 Bot Framework 模擬器。
2. 按一下模擬器 [歡迎] 索引標籤中的 [開啟 Bot] 連結。
3. 選取位於您建立專案的目錄中的 .bot 檔案。

傳送訊息給 Bot，Bot 就會以訊息回應。
![模擬器執行中](../media/emulator-v4/js-quickstart.png)
