---
title: 使用 Visual Studio 部署 C# Bot
description: 將您的 Bot 部署至 Azure 雲端。
keywords: 部署 bot, azure 部署, 發佈 bot, az 部署 bot, visual studio 部署 bot, msbot 發佈, msbot 複製
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: get-started-article
ms.service: bot-service
ms.subservice: abs
ms.date: 12/08/2018
ms.openlocfilehash: ac4e5f2ea385cb8318ad59e04c8ca8787480f5c8
ms.sourcegitcommit: 77664484e1b0780a15f686ef08bd23716b049b4a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 12/09/2018
ms.locfileid: "53121784"
---
# <a name="deploy-your-c-bot-using-visual-studio"></a>使用 Visual Studio 部署 C# Bot

[!INCLUDE [pre-release-label](./includes/pre-release-label.md)]

建立 Bot 並在本機測試後，您可以將其部署至 Azure，以便從任何地方進行存取。 將 Bot 部署至 Azure 時，需支付您所使用的服務。 [計費與成本管理](https://docs.microsoft.com/en-us/azure/billing/)一文協助您了解 Azure 計費方式、監視使用量和成本，以及管理帳戶和訂用帳戶。

在本文中，我們會示範如何使用 Visual Studio 和 Azure 入口網站來部署 C# Bot。 在依照步驟執行前閱讀本文很有用，您可完全了解部署 Bot 的相關事項。

## <a name="prerequisites"></a>必要條件
- 安裝 [Bot Framework 模擬器](https://aka.ms/Emulator-wiki-getting-started)。
- 安裝及設定 [ngrok](https://github.com/Microsoft/BotFramework-Emulator/wiki/Tunneling-%28ngrok%29)。
- [.bot](v4sdk/bot-file-basics.md) 檔案的知識。

## <a name="deploy-your-bot-in-app-service"></a>在 App Service 中部署 Bot
您會先在 App Service 中將 Bot 從 Visual Studio部署至 Azure。 然後，您將使用 Bot 通道註冊，透過 Azure Bot Service 設定您的 Bot。

**注意：如果 Visual Studio 專案名稱包含空格，如下所述的部署步驟將無法運作。**

在 [方案總管] 視窗中，以滑鼠右鍵按一下專案的節點，然後選取 [發佈]。

![發佈設定](media/azure-bot-quickstarts/getting-started-publish-setting.png)

2. 在 [挑選發佈目標] 對話方塊中，確保選取左側的 [App Service]，以及右側的 [新建]。

3. 按一下 [發佈] 按鈕。

4. 在對話方塊的右上角，確保對話方塊顯示 Azure 訂用帳戶的正確使用者識別碼。

![發佈主要區段](media/azure-bot-quickstarts/getting-started-publish-main.png)

5. 輸入應用程式名稱、訂用帳戶、資源群組和主控方案的資訊。

6. 準備好後，按一下 [建立]。 完成此程序可能需要幾分鐘時間。

7. 完成之後，將開啟一個網頁瀏覽器，顯示 Bot 的公用 URL。

8. 複製此 URL (它將類似於 https://<yourbotname>.azurewebsites.net/)。

> [!NOTE] 
> 註冊 Bot 時，您需要使用 HTTPS 版本的 URL。 Azure 透過 Azure 應用程式服務提供 SSL 支援。

## <a name="create-your-bot-channels-registration"></a>建立您的 Bot 通道註冊
在 Azure 中部署 Bot 時，您需要向 Azure Bot 服務註冊它。

1. 存取 Azure 入口網站： https://portal.azure.com 。

2. 使用您之前從 Visual Studio 用來發佈 Bot 的相同身分識別登入。

3. 按一下 [建立資源]。

4. 在 [搜尋市集] 欄位中，輸入 [Bot 通道註冊]，然後按 Enter。

5. 在傳回的清單中，選擇 [Bot 通道註冊]：

![publish](media/azure-bot-quickstarts/getting-started-bot-registration.png)

6. 在開啟的刀鋒視窗中按一下 [建立]。

7. 為您的 Bot 提供名稱。

8. 選擇您在其中部署 Bot 的程式碼的相同訂用帳戶。

9. 挑選將會設定位置的現有資源群組。

10. 您可以選擇 F0 定價層進行開發和測試。

11. 輸入您 Bot 的 URL。 請確定您以 HTTPS 開頭，並新增 /api/messages，例如 https://yourbotname.azurewebsites.net/api/messages

12. 暫時關閉 Application Insights。

13. 按一下 [Microsoft 應用程式識別碼和密碼]

14. 在新的刀鋒視窗上，按一下 [新建]。

15. 在右側開啟的新刀鋒視窗中，按一下 [在應用程式註冊入口網站中建立應用程式識別碼]，它將會以新的瀏覽器索引標籤開啟。

![bot msa](media/azure-bot-quickstarts/getting-started-msa.png)

16. 在新的索引標籤中，建立應用程式識別碼複本，並將它儲存到其他位置。 

17. 按一下 [產生應用程式密碼以繼續] 按鈕。

18. 瀏覽器對話方塊隨即開啟，並提供您應用程式的密碼，這是您會取得密碼的唯一時間。 複製密碼，並儲存在稍後可以取得此密碼的位置。

19. 儲存密碼後，按一下 [確定]。

20. 只需關閉瀏覽器索引標籤，並返回 [Azure 入口網站] 索引標籤。

21. 在正確的欄位中貼上您的應用程式識別碼和密碼，然後按一下 [確定]。

22. 現在按一下 [建立] 以設定您的通道註冊。 這可能需要幾分鐘。

## <a name="update-your-bots-application-settings"></a>更新 Bot 的應用程式設定
為了讓您的 Bot 向 Azure Bot 服務進行身分驗證，您需要在 Azure 應用程式服務中將兩個設定新增至您 Bot 的應用程式設定。 

1. 按一下 [應用程式服務]。 在 [訂用帳戶] 文字方塊中輸入您的 Bot 名稱。 然後按一下清單中您的 Bot 名稱。

![應用程式服務](media/azure-bot-quickstarts/getting-started-app-service.png)

2. 在 Bot 選項內左側的選項清單中，在 [設定] 區段中找出 [應用程式設定]，然後按一下它。

![bot 識別碼](media/azure-bot-quickstarts/getting-started-app-settings-1.png)

3. 捲動直到您找到 [應用程式設定] 區段。

![bot msa](media/azure-bot-quickstarts/getting-started-app-settings-2.png)

4. 按一下 [新增設定]。

5. 輸入 **MicrosoftAppId** 作為名稱，並輸入您的應用程式識別碼作為值。

6. 按一下 [新增設定]

7. 輸入 **MicrosoftAppPassword** 作為名稱，並輸入您的密碼作為值。

8. 按一下最上面的 [儲存] 按鈕。

## <a name="test-your-bot-in-production"></a>在生產環境中測試您的 Bot
此時，您可以使用內建的 Web 聊天用戶端，從 Azure 測試您的 Bot。

1. 返回您在 Azure 入口網站中的資源群組

2. 開啟 Bot。

3. 在 [Bot 管理] 下，選取 [在網路聊天中測試]。

![在 Web 聊天中測試](media/azure-bot-quickstarts/getting-started-test-webchat.png)

4. 輸入訊息 (例如 `Hi`)，然後按 Enter。 Bot 將回應 `Turn 1: You sent Hi`。

---

## <a name="additional-resources"></a>其他資源

當您部署 Bot 時，這些資源通常會建立於 Azure 入口網站中：

| 資源      | 說明 |
|----------------|-------------|
| Web 應用程式 Bot | 已部署至 Azure App Service 的 Azure Bot Service Bot。|
| [App Service](https://docs.microsoft.com/en-us/azure/app-service/)| 可讓您建置及裝載 Web 應用程式。|
| [App Service 計劃](https://docs.microsoft.com/en-us/azure/app-service/azure-web-sites-web-hosting-plans-in-depth-overview)| 針對要執行的 Web 應用程式定義一組計算資源。|
| [Application Insights](https://docs.microsoft.com/en-us/azure/application-insights/app-insights-overview)| 提供用來收集和分析遙測的工具。|
| [儲存體帳戶](https://docs.microsoft.com/en-us/azure/storage/common/storage-introduction)| 提供高可用性、安全、耐用、可擴充和備援性雲端儲存體。|

如果您不熟悉 Azure 資源群組，請參閱這個[術語](https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-overview#terminology)主題。

## <a name="next-steps"></a>後續步驟
> [!div class="nextstepaction"]
> [設定持續部署](bot-service-build-continuous-deployment.md)
