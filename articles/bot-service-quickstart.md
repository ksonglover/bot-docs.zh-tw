---
title: 使用 Bot 服務建立 Bot | Microsoft Docs
description: 了解如何使用 Bot 服務建立 Bot，Bot 服務是專用於 Bot 的整合式開發環境。
keywords: Quickstart, create bot, bot service, web app bot, 快速入門, 建立 Bot, Bot 服務, Web 應用程式 Bot
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: abs
ms.date: 02/07/2019
ms.openlocfilehash: 882dc15bd900667d76eac41049ee88b51ec5c120
ms.sourcegitcommit: f84b56beecd41debe6baf056e98332f20b646bda
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/03/2019
ms.locfileid: "65033127"
---
# <a name="create-a-bot-with-azure-bot-service"></a>建立具有 Azure Bot Service 的 Bot

::: moniker range="azure-bot-service-3.0"

[!INCLUDE [pre-release-label](includes/pre-release-label-v3.md)]

Bot 服務會提供建立 Bot 的核心元件，包括用於開發 Bot 的 Bot Framework SDK 和將 Bot 連線至通道的 Bot Framework。 建立支援 .NET 和 Node.js 的 Bot 時，Bot 服務提供五種範本供您選擇。 在本主題中，您將了解如何使用 Bot 服務來建立使用 Bot Framework SDK 的新 Bot。

## <a name="log-in-to-azure"></a>登入 Azure
登入 [Azure 入口網站](http://portal.azure.com)。

> [!TIP]
> 若您還沒有訂用帳戶，則可以註冊<a href="https://azure.microsoft.com/en-us/free/" target="_blank">免費帳戶</a>。

## <a name="create-a-new-bot-service"></a>建立新的 Bot 服務

1. 按一下 Azure 入口網站左上角的 [建立新資源] 連結，然後選取 [AI + 機器學習服務] > [Web 應用程式 Bot]。 

2. 隨即開啟含有 **Web 應用程式 Bot** 相關資訊的新刀鋒視窗。  

3. 在 [Bot 服務] 刀鋒視窗中，依照下圖下方表格中說明的內容，提供有關您 Bot 的必要資訊。  <br/>
   ![建立 Web 應用程式 Bot 刀鋒視窗](./media/azure-bot-quickstarts/sdk-create-bot-service-blade.png)

   | 設定 | 建議的值 | 說明 |
   | ---- | ---- | ---- |
   | **Bot 名稱** | 您的 Bot 顯示名稱 | Bot 在通道和目錄中的顯示名稱。 您可以隨時變更此名稱。 |
   | **訂用帳戶** | 您的訂用帳戶 | 選取您要使用的 Azure 訂用帳戶。 |
   | **資源群組** | myResourceGroup | 您可以建立新的[資源群組](/azure/azure-resource-manager/resource-group-overview#resource-groups)，或選擇現有的資源群組。 |
   | **位置** | 預設的位置 | 選取資源群組的地理位置。 您可以選擇任何列出的位置，但通常最佳的選擇是最靠近您客戶的位置。 一旦建立 Bot，就無法變更位置。 |
   | **定價層** | F0 | 選取定價層。 您可以隨時更新定價層。 如需詳細資訊，請參閱 [Bot 服務價格](https://azure.microsoft.com/en-us/pricing/details/bot-service/)。 |
   | **應用程式名稱** | 唯一的名稱 | Bot 的唯一 URL 名稱。 例如，如果您將 Bot 命名為 *myawesomebot*，則 Bot 的 URL 將會是 `http://myawesomebot.azurewebsites.net`。 名稱只能使用英數字元和底線字元。 此欄位有 35 個字元的長度限制。 一旦建立 Bot，就無法變更應用程式名稱。 |
   | **Bot 範本** | 基本 | 選擇 [C#] 或 [Node.js]，並選取 [基本] 範本以供此快速入門使用，然後按一下 [選取]。 基本範本會建立回應 Bot。 [深入了解](bot-service-concept-templates.md)範本。 |
   | **App Service 方案/位置** | 您的 App Service 方案  | 選取 [App Service 方案](https://azure.microsoft.com/en-us/pricing/details/app-service/plans/)位置。 您可以選擇任何列出的位置，但通常最佳的選擇是最靠近您客戶的位置。 (不適用於 Functions Bot)。 |
   | **Application Insights** | 另一 | 決定您要**開啟**或**關閉** [Application Insights](/bot-framework/bot-service-manage-analytics)。 如果您選取 [開啟]，您也必須指定區域位置。 您可以選擇任何列出的位置，但通常最好是選擇與 Bot 服務位置相同的位置。 |
   | **Microsoft 應用程式識別碼和密碼** | 自動建立應用程式識別碼和密碼 | 如果您需要手動輸入 Microsoft 應用程式識別碼和密碼，請使用此選項。 否則，在 Bot 建立程序期間，便會為您建立新的 Microsoft 應用程式識別碼和密碼。 |

   > [!NOTE]
   > 
   > 如果您建立的是 **Functions Bot**，則不會看到 [App Service 方案/位置] 欄位。 相反地，您會看到 [主控方案] 欄位。 在該案例中，請選擇[主控方案](bot-service-overview-readme.md#hosting-plans)。

4. 按一下 [建立] 以建立服務，並將 Bot 部署到雲端。 此程序可能需要幾分鐘的時間。

勾選 [通知] 來確認已部署 Bot。 通知會從 [部署進行中] 變更為 [部署成功]。 按一下 [前往資源] 按鈕以開啟 Bot 的資源刀鋒視窗。

 > [!TIP] 
 > 基於效能考量，執行 Node.js 範本的 **Functions Bot** 已經使用 *Azure Functions Pack* 工具封裝。 *Azure Functions Pack* 工具會擷取所有 Node.js 模組，並將它們結合成一個 *.js 檔案。
 > 如需詳細資訊，請參閱 [Azure Functions Pack](https://github.com/Azure/azure-functions-pack) \(英文\)。
 
## <a name="test-the-bot"></a>測試 Bot
現在，既然您已建立 Bot，請在 [Web 聊天](bot-service-manage-test-webchat.md)中測試它。 輸入訊息，您的 Bot 應會回應。

## <a name="next-steps"></a>後續步驟

在此主題中，您已了解如何使用 Bot 服務建立**基本** Web 應用程式 Bot/Functions Bot，並在 Azure 中使用內建的 Web 聊天控制項驗證 Bot 功能。 現在，請了解如何管理 Bot 並開始處理其原始程式碼。

> [!div class="nextstepaction"]
> [管理 Bot](bot-service-manage-overview.md)

::: moniker-end

::: moniker range="azure-bot-service-4.0"

[!INCLUDE [applies-to-v4](includes/applies-to.md)]


Azure Bot 服務提供建立 Bot 的核心元件，包括用於開發 Bot 的 Bot Framework SDK 和連接 Bot 與通道的 Bot 服務。 在此主題中，您可以使用 Bot Framework SDK v4，選擇 .NET 或 Node.js 範本來建立 Bot。

[!INCLUDE [Azure vs local development](~/includes/snippet-quickstart-paths.md)]

## <a name="prerequisites"></a>必要條件

- [Azure](http://portal.azure.com) 帳戶

### <a name="create-a-new-bot-service"></a>建立新的 Bot 服務

1. 登入 [Azure 入口網站](http://portal.azure.com/)。
1. 按一下 Azure 入口網站左上角的 [建立新資源] 連結，然後選取 [AI + 機器學習服務] > [Web 應用程式 Bot]。 

![建立 Bot](~/media/azure-bot-quickstarts/abs-create-blade.png)

2. 隨即開啟含有 *Web 應用程式 Bot* 相關資訊的「新刀鋒視窗」。  

3. 在 [Bot 服務] 刀鋒視窗中，依照下圖下方表格中說明的內容，提供有關您 Bot 的必要資訊。  <br/>
 ![建立 Web 應用程式 Bot 刀鋒視窗](~/media/azure-bot-quickstarts/sdk-create-bot-service-blade.png)

 | 設定 | 建議的值 | 說明 |
 | ---- | ---- | ---- |
 | **Bot 名稱** | 您的 Bot 顯示名稱 | Bot 在通道和目錄中的顯示名稱。 您可以隨時變更此名稱。 |
 | **訂用帳戶** | 您的訂用帳戶 | 選取您要使用的 Azure 訂用帳戶。 |
 | **資源群組** | myResourceGroup | 您可以建立新的[資源群組](/azure/azure-resource-manager/resource-group-overview#resource-groups)，或選擇現有的資源群組。 |
 | **位置** | 預設的位置 | 選取資源群組的地理位置。 您可以選擇任何列出的位置，但通常最佳的選擇是最靠近您客戶的位置。 一旦建立 Bot，就無法變更位置。 |
 | **定價層** | F0 | 選取定價層。 您可以隨時更新定價層。 如需詳細資訊，請參閱 [Bot 服務價格](https://azure.microsoft.com/en-us/pricing/details/bot-service/)。 |
 | **應用程式名稱** | 唯一的名稱 | Bot 的唯一 URL 名稱。 例如，如果您將 Bot 命名為 *myawesomebot*，則 Bot 的 URL 將會是 `http://myawesomebot.azurewebsites.net`。 名稱只能使用英數字元和底線字元。 此欄位有 35 個字元的長度限制。 一旦建立 Bot，就無法變更應用程式名稱。 |
 | **Bot 範本** | 回應 Bot | 選擇 [SDK v4]。 選取 [C#] 或 [Node.js] 以供本快速入門使用，然後按一下 [選取]。  
 | **App Service 方案/位置** | 您的 App Service 方案  | 選取 [App Service 方案](https://azure.microsoft.com/en-us/pricing/details/app-service/plans/)位置。 您可以選擇任何列出的位置，但通常最好是選擇與 Bot 服務相同的位置。 |
 | **Application Insights** | 另一 | 決定您要**開啟**或**關閉** [Application Insights](/bot-framework/bot-service-manage-analytics)。 如果您選取 [開啟]，您也必須指定區域位置。 您可以選擇任何列出的位置，但通常最好是選擇與 Bot 服務相同的位置。 |
 | **Microsoft 應用程式識別碼和密碼** | 自動建立應用程式識別碼和密碼 | 如果您需要手動輸入 Microsoft 應用程式識別碼和密碼，請使用此選項。 否則，在 Bot 建立程序期間，便會為您建立新的 Microsoft 應用程式識別碼和密碼。 |

4. 按一下 [建立] 以建立服務，並將 Bot 部署到雲端。 此程序可能需要幾分鐘的時間。

勾選 [通知] 來確認已部署 Bot。 通知會從 [部署進行中] 變更為 [部署成功]。 按一下 [前往資源] 按鈕以開啟 Bot 的資源刀鋒視窗。

現在，既然您已建立 Bot，請在 Web 聊天中測試。 

## <a name="test-the-bot"></a>測試 Bot
在 [Bot 管理] 區段中，按一下 [在網路聊天中測試]。 Azure Bot Service 會將網路聊天控制項載入，並連線至 Bot。 

![Azure 網路聊天測試](./media/azure-bot-quickstarts/azure-webchat-test.png)

輸入訊息，您的 Bot 應會回應。

## <a name="download-code"></a>下載程式碼
您可以下載程式碼，以在本機上處理。 
1. 在 [Bot 管理] 區段中，按一下 [組建]。 
1. 在右窗格中按一下**下載 Bot 原始程式碼**連結。 
1. 遵循提示以下載程式碼，然後將資料夾解壓縮。
    1. [!INCLUDE [download keys snippet](~/includes/snippet-abs-key-download.md)]

## <a name="next-steps"></a>後續步驟
下載程式碼之後，您可以繼續在電腦本機開發 Bot。 當您測試 Bot 並準備將 Bot 程式碼上傳至 Azure 入口網站時，請遵循[設定持續部署](./bot-service-build-continuous-deployment.md)底下所列的指示，以在進行變更後自動更新程式碼。

::: moniker-end
