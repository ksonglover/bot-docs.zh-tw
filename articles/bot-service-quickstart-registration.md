---
title: 使用 Bot 服務建立 Bot 通道註冊 | Microsoft Docs
description: 了解如何使用 Bot 服務註冊現有的 Bot。
ms.author: kamrani
manager: kamrani
ms.topic: conceptual
ms.service: bot-service
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 014f5c998fcb9d322439ca8b0e0bf2ba5f9f0679
ms.sourcegitcommit: 0b647dc6716b0c06f04ee22ebdd7b53039c2784a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/28/2019
ms.locfileid: "70076518"
---
# <a name="register-a-bot-with-azure-bot-service"></a>使用 Azure Bot 服務註冊 Bot

本主題說明如何建立 **Azure Bot 服務**資源以註冊您的 Bot。 如果 Bot 裝載於其他位置，而您想要在 Azure 中加以使用，並將其連線至 Azure Bot 服務通道，則必須執行此作業。

這可讓您建置、連接及管理您的 Bot，以透過 Cortana、Skype、Messenger 和許多其他服務與任何一處的使用者互動。

> [!IMPORTANT] 
> 如果您的 Bot 未裝載於 Azure 中，您只需要註冊 Bot 即可。 如果您已透過 Azure 入口網站[建立 Bot](v4sdk/abs-quickstart.md)，則您的 Bot 即已註冊至該服務。

## <a name="create-a-registration-resource"></a>建立註冊資源

1. 在瀏覽器中導覽至 [Azure 入口網站](https://ms.portal.azure.com)。

    > [!TIP]
    > 若您沒有訂用帳戶，您可以註冊<a href="https://azure.microsoft.com/free/" target="_blank">免費帳戶</a>。

1. 在左窗格中，按一下 [建立資源]  。
1. 在右窗格的選取方塊中，輸入 *bot*。 在下拉式清單中，選取 [Bot 通道註冊]  或 [Web 應用程式 Bot]  ，視您的應用程式而定。
針對 **Web Bot 應用程式**，請依照下列文章中說明的步驟操作：[使用 Azure Bot 服務建立 Bot](v4sdk/abs-quickstart.md)。 您將在 Azure 中建立會自動向 Azure Bot 服務註冊的 Bot。
1. 按一下 [建立]  按鈕開始進行程序。
1. 在 [Bot 服務]  刀鋒視窗中，依照下圖下方表格中說明的內容，提供有關您 Bot 的必要資訊。  

   ![建立註冊 Bot 刀鋒視窗](media/azure-bot-quickstarts/registration-create-bot-service-blade.png)

   |設定 |建議的值|說明|
   |---|---|--|
   |**Bot 名稱** <img width="300px">|您的 Bot 顯示名稱|Bot 在通道和目錄中的顯示名稱。 您可以隨時變更此名稱。|
   |**訂用帳戶**|您的訂用帳戶|選取您要使用的 Azure 訂用帳戶。|
   |**資源群組**|myResourceGroup|您可以建立新的[資源群組](/azure/azure-resource-manager/resource-group-overview#resource-groups)，或選擇現有的資源群組。|
   |**位置**|美國西部|選擇接近您的 Bot 部署位置，或接近您的 Bot 存取其他服務的位置。|
   |**定價層**|F0|選取定價層。 您可以隨時更新定價層。 如需詳細資訊，請參閱 [Bot 服務價格](https://azure.microsoft.com/pricing/details/bot-service/)。|
   |**傳訊端點**|URL|輸入您的 Bot 傳訊端點的 URL。|
   |**Application Insights**|另一| 決定您要**開啟**或**關閉** [Application Insights](bot-service-manage-analytics.md)。 如果您選取 [開啟]  ，您還必須指定區域位置。 |
   |**Microsoft 應用程式識別碼和密碼**| 自動建立應用程式識別碼和密碼 |如果您需要手動輸入 Microsoft 應用程式識別碼和密碼，請使用此選項。 請參閱下一節：[手動註冊應用程式](#manual-app-registration)。 否則，在註冊程序中將會建立新的 Microsoft 應用程式識別碼和密碼。 |

    > [!IMPORTANT]
    > 別忘了輸入 Bot 傳訊端點的 URL。

1. 按一下 [ **建立** ] 按鈕。 等候資源建立。 其會顯示在您的資源清單中。

### <a name="get-registration-password"></a>取得註冊密碼

註冊完成後，Azure Active Directory 會為您的註冊指派唯一的應用程式識別碼，並將您導向至應用程式的 [概觀]  頁面。

若要取得密碼，請依照接下來說明的步驟操作。

1. 在資源清單中，按一下剛建立的 Azure App Service 資源。
1. 在右窗格的資源刀鋒視窗中，按一下 [設定]  。 資源的 [設定]  頁面隨即顯示。
1. 在 [設定] 頁面中，複製已產生的 **Microsoft 應用程式識別碼**，並將其儲存至檔案。
1. 按一下 *Microsoft 應用程式識別碼*的 [管理]  連結。

    ![建立註冊 Bot 刀鋒視窗](media/azure-bot-quickstarts/bot-channels-registration-app-settings.png)

1. 在顯示的 [憑證和祕密]  頁面中，按一下 [新增用戶端密碼]  按鈕。

    ![建立註冊 Bot 刀鋒視窗](media/azure-bot-quickstarts/bot-channels-registration-app-secrets.png)

1. 新增描述，選取到期時間，然後按一下 [新增]  按鈕。

    ![建立註冊 Bot 刀鋒視窗](media/azure-bot-quickstarts/bot-channels-registration-app-secrets-create.png)

    這會產生 Bot 的新密碼。 複製此密碼，並儲存至檔案。 這是您唯一會看到此密碼的時機。 如果您未儲存完整的密碼，您必須重複此程序來建立新的密碼，以便稍後使用。

## <a name="manual-app-registration"></a>手動註冊應用程式

在下列情況下必須以手動方式註冊：

- 您無法在組織中進行註冊，而需經由其他合作對象為您建置的 Bot 建立應用程式識別碼。
- 您需要手動建立自己的應用程式識別碼 (和密碼)。

請參閱[常見問題集 - 應用程式註冊](bot-service-resources-bot-framework-faq.md#app-registration)。

> [!IMPORTANT]
> 在*支援帳戶類型*一節中，您必須從兩種多租用戶類型中擇一使用：*任何組織目錄中的帳戶 (任何 Azure AD - 多租用戶)* 或*任何組織目錄中的帳戶 (任何 Azure AD - 多租用戶) 和個人 Microsoft 帳戶 (例如 Skype、Xbox、Outlook.com)* ，用以建立應用程式，否則 Bot 將無法運作。 如需詳細資訊，請參閱[使用 Azure 入口網站註冊新的應用程式](https://docs.microsoft.com/en-us/azure/active-directory/develop/quickstart-register-app#register-a-new-application-using-the-azure-portal)。

## <a name="update-the-bot"></a>更新 Bot

如果您使用適用於 .NET 的 Bot Framework SDK，請在 web.config 檔案中設定下列索引鍵值：

- `MicrosoftAppId = <appId>`
- `MicrosoftAppPassword = <appSecret>`

如果您使用適用於 Node.js 的 Bot Framework SDK，請設定下列環境變數：

- `MICROSOFT_APP_ID = <appId>`
- `MICROSOFT_APP_PASSWORD = <appSecret>`

## <a name="test-the-bot"></a>測試 Bot

現在，既然您已建立 Bot，請[在網路聊天中進行測試](bot-service-manage-test-webchat.md)。 輸入訊息，您的 Bot 應會回應。

## <a name="next-steps"></a>後續步驟

本主題中，您已了解如何使用 Bot 服務註冊裝載的 Bot。 下一個步驟是了解如何管理您的 Bot 服務。

> [!div class="nextstepaction"]
> [管理 Bot](bot-service-manage-overview.md)
