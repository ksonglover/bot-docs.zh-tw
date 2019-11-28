---
ms.openlocfilehash: 9c86001a51914f359163e7d755aa57e1c54127f8
ms.sourcegitcommit: dcacda776c927bcc7c76d00ff3cc6b00b062bd6b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 11/23/2019
ms.locfileid: "74414581"
---
## <a name="prerequisites"></a>必要條件
- [Visual Studio 2017 或更新版本](https://www.visualstudio.com/downloads)
- [適用於 C# 的 Bot Framework SDK v4 範本](https://aka.ms/bot-vsix)
- [Bot Framework 模擬器](https://aka.ms/bot-framework-emulator-readme) (英文)
- 了解 [ASP.Net Core](https://docs.microsoft.com/aspnet/core/) 和 [ C# 的非同步程式設計](https://docs.microsoft.com/dotnet/csharp/programming-guide/concepts/async/index)

## <a name="create-a-bot"></a>建立 Bot
安裝您在必要條件區段下載的 [BotBuilderVSIX.vsix](https://aka.ms/bot-vsix) 範本。

在 Visual Studio 中，使用 **Echo Bot (Bot Framework v4)** 範本建立新的 Bot 專案。 在搜尋方塊中輸入 bot framework v4  ，即可只顯示 Bot 範本。

![Visual Studio 會建立新的專案對話方塊](../media/azure-bot-quickstarts/bot-builder-dotnet-project-vs2019.png)

> [!TIP] 
> 如果使用 Visual Studio 2017，請確定專案組建類型為 ``.Net Core 2.1`` 或更新版本。 此外，視需要更新 `Microsoft.Bot.Builder` [NuGet 套件](https://docs.microsoft.com/nuget/quickstart/install-and-use-a-package-in-visual-studio)。

由於有範本，專案中會包含要在本快速入門建立聊天機器人所需的所有程式碼。 您實際上不需要撰寫任何額外的程式碼。

## <a name="start-your-bot-in-visual-studio"></a>在 Visual Studio 中啟動 Bot

當您按一下執行按鈕時，Visual Studio 將會建置應用程式、將其部署到 localhost，並啟動 Web 瀏覽器來顯示應用程式的 `default.htm` 頁面。 此時，Bot 正在本機執行。

## <a name="start-the-emulator-and-connect-your-bot"></a>啟動模擬器並且連線至您的 Bot

接下來，請啟動模擬器，然後在模擬器中連線至您的 Bot：

1. 按一下模擬器 [歡迎使用] 索引標籤中的 [建立新的 Bot 設定]  連結。 
2. 填寫聊天機器人的欄位。 使用聊天機器人的歡迎頁面網址 (通常為 http://localhost:3978) )，並在此網址附加路由資訊 '/api/messages'。
3. 然後按一下 [儲存並連線]  。

## <a name="interact-with-your-bot"></a>與您的 Bot 互動

傳送訊息給 Bot，Bot 就會以訊息回應。

![執行中模擬器](~/media/emulator-v4/emulator-running.png)

> [!NOTE]
> 如果您發現訊息無法傳送，則可能需要重新啟動電腦，因為 ngrok 尚未在您的系統上取得所需的權限 (只需要進行一次)。
