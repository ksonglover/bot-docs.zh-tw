---
title: 使用 Bot 服務建立 Bot | Microsoft Docs
description: 了解如何使用 Bot 服務建立 Bot，Bot 服務是專用於 Bot 的整合式開發環境。
keywords: Quickstart, create bot, bot service, web app bot, 快速入門, 建立 Bot, Bot 服務, Web 應用程式 Bot
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 08/15/2019
ms.openlocfilehash: b8307877bf08db170173486ec9df88fae4d27c29
ms.sourcegitcommit: 9e1034a86ffdf2289b0d13cba2bd9bdf1958e7bc
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/21/2019
ms.locfileid: "69890621"
---
# <a name="create-a-bot-with-azure-bot-service"></a>建立具有 Azure Bot Service 的 Bot

[!INCLUDE [applies-to-v4](../includes/applies-to.md)]

Azure Bot 服務提供建立 Bot 的核心元件，包括用於開發 Bot 的 Bot Framework SDK 和連接 Bot 與通道的 Bot 服務。 在此主題中，您可以使用 Bot Framework SDK v4，選擇 .NET 或 Node.js 範本來建立 Bot。

>[!NOTE] 
> 您所建立的 Bot 會自動向 Azure Bot 服務註冊。 如果您已有裝載於其他位置的 Bot，而您想要加以註冊，請參閱下列文章：[使用 Azure Bot 服務註冊 Bot](../bot-service-quickstart-registration.md)。

[!INCLUDE [Azure vs local development](~/includes/snippet-quickstart-paths.md)]

## <a name="prerequisites"></a>必要條件

- [Azure](http://portal.azure.com) 帳戶

### <a name="create-a-new-bot-service"></a>建立新的 Bot 服務

1. 登入 [Azure 入口網站](http://portal.azure.com/)。
1. 按一下 Azure 入口網站左上角的 [建立新資源]  連結，然後選取 [AI + 機器學習服務]   > [Web 應用程式 Bot]  。 

![建立 Bot](../media/azure-bot-quickstarts/abs-create-blade.png)

2. 隨即開啟含有 *Web 應用程式 Bot* 相關資訊的「新刀鋒視窗」  。  

3. 在 [Bot 服務]  刀鋒視窗中，依照下圖下方表格中說明的內容，提供有關您 Bot 的必要資訊。  <br/>
 ![建立 Web 應用程式 Bot 刀鋒視窗](../media/azure-bot-quickstarts/sdk-create-bot-service-blade.png)

 | 設定 | 建議的值 | 說明 |
 | ---- | ---- | ---- |
 | **Bot 名稱** | 您的 Bot 顯示名稱 | Bot 在通道和目錄中的顯示名稱。 您可以隨時變更此名稱。 |
 | **訂用帳戶** | 您的訂用帳戶 | 選取您要使用的 Azure 訂用帳戶。 |
 | **資源群組** | myResourceGroup | 您可以建立新的[資源群組](/azure/azure-resource-manager/resource-group-overview#resource-groups)，或選擇現有的資源群組。 |
 | **位置** | 預設的位置 | 選取資源群組的地理位置。 您可以選擇任何列出的位置，但通常最佳的選擇是最靠近您客戶的位置。 一旦建立 Bot，就無法變更位置。 |
 | **定價層** | F0 | 選取定價層。 您可以隨時更新定價層。 如需詳細資訊，請參閱 [Bot 服務價格](https://azure.microsoft.com/pricing/details/bot-service/)。 |
 | **應用程式名稱** | 唯一的名稱 | Bot 的唯一 URL 名稱。 例如，如果您將 Bot 命名為 *myawesomebot*，則 Bot 的 URL 將會是 `http://myawesomebot.azurewebsites.net`。 名稱只能使用英數字元和底線字元。 此欄位有 35 個字元的長度限制。 一旦建立 Bot，就無法變更應用程式名稱。 |
 | **Bot 範本** | 回應 Bot | 選擇 [SDK v4]  。 選取 [C#] 或 [Node.js] 以供本快速入門使用，然後按一下 [選取]  。  
 | **App Service 方案/位置** | 您的 App Service 方案  | 選取 [App Service 方案](https://azure.microsoft.com/pricing/details/app-service/plans/)位置。 您可以選擇任何列出的位置，但通常最好是選擇與 Bot 服務相同的位置。 |
 | **Application Insights** | 另一 | 決定您要**開啟**或**關閉** [Application Insights](/bot-framework/bot-service-manage-analytics)。 如果您選取 [開啟]  ，您也必須指定區域位置。 您可以選擇任何列出的位置，但通常最好是選擇與 Bot 服務相同的位置。 |
 | **Microsoft 應用程式識別碼和密碼** | 自動建立應用程式識別碼和密碼 | 如果您需要手動輸入 Microsoft 應用程式識別碼和密碼，請使用此選項。 否則，在 Bot 建立程序期間，便會為您建立新的 Microsoft 應用程式識別碼和密碼。 在為 Bot Service 手動建立應用程式註冊時，請確定您已將支援的帳戶類型設為「任何組織目錄中的帳戶」或「任何組織目錄中的帳戶及個人的 Microsoft 帳戶 (例如 Skype、Outlook.com、Xbox 等等)」。 |

4. 按一下 [建立]  以建立服務，並將 Bot 部署到雲端。 此程序可能需要幾分鐘的時間。

勾選 [通知]  來確認已部署 Bot。 通知會從 [部署進行中]  變更為 [部署成功]  。 按一下 [前往資源]  按鈕以開啟 Bot 的資源刀鋒視窗。

現在，既然您已建立 Bot，請在 Web 聊天中測試。

## <a name="test-the-bot"></a>測試 Bot
在 [Bot 管理]  區段中，按一下 [在網路聊天中測試]  。 Azure Bot Service 會將網路聊天控制項載入，並連線至 Bot。 

![Azure 網路聊天測試](../media/azure-bot-quickstarts/azure-webchat-test.png)

輸入訊息，您的 Bot 應會回應。

## <a name="manual-app-registration"></a>手動註冊應用程式

在下列情況下必須以手動方式註冊：

- 您無法在組織中進行註冊，而需經由其他合作對象為您建置的 Bot 建立應用程式識別碼。
- 您需要手動建立自己的應用程式識別碼 (和密碼)。

請參閱[常見問題集 - 應用程式註冊](../bot-service-resources-bot-framework-faq.md#app-registration)。


## <a name="download-code"></a>下載程式碼
您可以下載程式碼，以在本機上處理。 
1. 在 [Bot 管理]  區段中，按一下 [組建]  。 
1. 在右窗格中按一下**下載 Bot 原始程式碼**連結。 
1. 遵循提示以下載程式碼，然後將資料夾解壓縮。
    1. [!INCLUDE [download keys snippet](../includes/snippet-abs-key-download.md)]

## <a name="next-steps"></a>後續步驟
下載程式碼之後，您可以繼續在電腦本機開發 Bot。 當您測試 Bot 並準備將 Bot 程式碼上傳至 Azure 入口網站時，請遵循[設定持續部署](../bot-service-build-continuous-deployment.md)底下所列的指示，以在進行變更後自動更新程式碼。
