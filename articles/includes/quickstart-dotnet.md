## <a name="prerequisites"></a>必要條件
- Visual Studio [2017](https://www.visualstudio.com/downloads)
- 適用於 [C#](https://aka.ms/bot-vsix) 的 Bot Framework SDK v4 範本
- Bot Framework [模擬器](https://aka.ms/Emulator-wiki-getting-started) (英文)
- 了解 [ASP.Net Core](https://docs.microsoft.com/aspnet/core/) 和 [C#](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/async/index) 中的非同步程式設計

## <a name="create-a-bot"></a>建立 Bot
安裝您在必要條件區段下載的 BotBuilderVSIX.vsix 範本。

在 Visual Studio 中，使用 **Bot Framework Echo Bot** V4 範本建立新的 Bot 專案。

![Visual Studio 專案](~/media/azure-bot-quickstarts/bot-builder-dotnet-project.png)

> [!TIP] 
> 如有需要，請將專案組建類型變更為 ``.Net Core 2.1``。 此外，視需要更新 `Microsoft.Bot.Builder` [NuGet 套件](https://docs.microsoft.com/en-us/nuget/quickstart/install-and-use-a-package-in-visual-studio)。

由於有範本，專案中會包含要在本快速入門建立 Bot 所需的所有程式碼。 您實際上不需要撰寫任何額外的程式碼。

## <a name="start-your-bot-in-visual-studio"></a>在 Visual Studio 中啟動 Bot

當您按一下執行按鈕時，Visual Studio 將會建置應用程式、將其部署到 localhost，並啟動 Web 瀏覽器來顯示應用程式的 `default.htm` 頁面。 此時，Bot 正在本機執行。

## <a name="start-the-emulator-and-connect-your-bot"></a>啟動模擬器並連線至您的 Bot

接下來，請啟動模擬器，然後在模擬器中連線至您的 Bot：

1. 按一下模擬器 [歡迎] 索引標籤中的 [開啟 Bot] 連結。 
2. 選取位於您建立 Visual Studio 解決方案的目錄中的 .bot 檔案。

## <a name="interact-with-your-bot"></a>與您的 Bot 互動

傳送訊息給 Bot，Bot 就會以訊息回應。

![執行中模擬器](~/media/emulator-v4/emulator-running.png)

> [!NOTE]
> 如果您發現訊息無法傳送，則可能需要重新啟動電腦，因為 ngrok 尚未在您的系統上取得所需的權限 (只需要進行一次)。
