---
title: 使用 Bot 服務建立 Bot 通道註冊 | Microsoft Docs
description: 了解如何使用 Bot 服務註冊現有的 Bot。
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: abs
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: dfc90f4c4c6e3ad00899569f667b5d3d88dcf042
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/24/2018
ms.locfileid: "49999955"
---
# <a name="register-a-bot-with-bot-service"></a>使用 Bot 服務建立 Bot



如果您的 Bot 已裝載於其他位置，而且您想要使用 Bot 服務與其他通道連線，則您必須使用 Bot 服務註冊您的 Bot。 在本主題中，了解如何藉由建立 **Bot 通道註冊** Bot 服務，使用 Bot 服務註冊您的 Bot。

> [!IMPORTANT] 
> 如果您的 Bot 未裝載於 Azure 中，您只需要註冊 Bot 即可。 如果您透過 Azure 入口網站[建立 Bot](bot-service-quickstart.md)，表示您的 Bot 已使用 Bot 服務註冊。

## <a name="log-in-to-azure"></a>登入 Azure
登入 [Azure 入口網站](http://portal.azure.com)。

> [!TIP]
> 若您還沒有訂用帳戶，則可以註冊<a href="https://azure.microsoft.com/en-us/free/" target="_blank">免費帳戶</a>。

## <a name="create-a-bot-channels-registration"></a>建立 Bot 通道註冊
您需要 **Bot 通道註冊** Bot 服務才能使用 Bot 服務功能。 註冊 Bot 可讓您將 Bot 連線至通道。

如要建立 **Bot 通道註冊**，請執行下列操作：

1. 按一下 Azure 入口網站左上角的 [新增] 按鈕，然後選取 [AI + 認知服務] > [Bot 通道註冊]。 

2. 隨即開啟含有 **Bot 通道註冊**相關資訊的新刀鋒視窗。 按一下 [建立] 按鈕來開始建立程序。 

3. 在 [Bot 服務] 刀鋒視窗中，依照下圖下方表格中說明的內容，提供有關您 Bot 的必要資訊。  <br/>
   ![建立註冊 Bot 刀鋒視窗](~/media/azure-bot-quickstarts/registration-create-bot-service-blade.png)


   |                    設定                     |         建議的值         |                                                                                                  說明                                                                                                  |
   |------------------------------------------------|---------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
   |           <strong>Bot 名稱</strong>            |     您的 Bot 顯示名稱     |                                                  Bot 在通道和目錄中的顯示名稱。 您可以隨時變更此名稱。                                                  |
   |         <strong>訂用帳戶</strong>          |        您的訂用帳戶        |                                                                                選取您要使用的 Azure 訂用帳戶。                                                                                 |
   |        <strong>資源群組</strong>         |         myResourceGroup         |                                 您可以建立新的[資源群組](/azure/azure-resource-manager/resource-group-overview#resource-groups)，或選擇現有的資源群組。                                  |
   |                    位置                    |             美國西部             |                                                        選擇接近您的 Bot 部署位置，或接近您的 Bot 存取其他服務的位置。                                                         |
   |         <strong>定價層</strong>          |               F0                |             選取定價層。 您可以隨時更新定價層。 如需詳細資訊，請參閱 [Bot 服務價格](https://azure.microsoft.com/en-us/pricing/details/bot-service/)。              |
   |      <strong>傳訊端點</strong>       |               URL               |                                                                               輸入您的 Bot 傳訊端點的 URL。                                                                                |
   |     <strong>Application Insights</strong>      |               另一                | 決定您要<strong>開啟</strong>或<strong>關閉</strong> [Application Insights](bot-service-manage-analytics.md)。 如果您選取 [開啟]，您還必須指定區域位置。 |
   | <strong>Microsoft 應用程式識別碼和密碼</strong> | 自動建立應用程式識別碼和密碼 |              如果您需要手動輸入 Microsoft 應用程式識別碼和密碼，請使用此選項。 否則，在 Bot 建立流程當中，便會為您建立新的 Microsoft 應用程式識別碼和密碼。               |


4. 按一下 [建立] 即可建立服務並註冊您的 Bot 的傳訊端點。

勾選 [通知] 來確認已建立的註冊。 通知會從 [部署進行中] 變更為 [部署成功]。 按一下 [前往資源] 按鈕以開啟 Bot 的資源刀鋒視窗。 

## <a name="bot-channels-registration-password"></a>Bot 通道密碼

**Bot 通道註冊** Bot 服務沒有相關聯的應用程式服務。 因此，此 Bot 服務只有 *MicrosoftAppID*。 您需要手動產生密碼，然後自行儲存。 您必須使用此密碼才能使用[模擬器](bot-service-debug-emulator.md)測試 Bot。

若要產生 Microsoft 應用程式密碼，請執行下列操作：

1. 在 [設定] 刀鋒視窗中，按一下 [管理]。 這是 **Microsoft 應用程式識別碼**顯示的連結。 此連結會開啟視窗，您可以在其中產生新的密碼。 <br/>
  ![在 [設定] 刀鋒視窗中管理連結](~/media/azure-bot-quickstarts/registration-settings-manage-link.png)

2. 按一下 [產生新密碼]。 這會產生 Bot 的新密碼。 複製此密碼，並儲存至檔案。 這是您唯一會看到此密碼的時機。 如果您未儲存完整的密碼，您必須重複此程序來建立新的密碼，以便稍後使用。 <br/>
  ![產生 Microsoft 應用程式密碼](~/media/azure-bot-quickstarts/registration-generate-app-password.png)

## <a name="update-the-bot"></a>更新 Bot

如果您使用適用於 Node.js 的 Bot Builder SDK，請設定下列環境變數：

* MICROSOFT_APP_ID
* MICROSOFT_APP_PASSWORD

如果您使用適用於 .NET 的 Bot Builder SDK，請在 web.config 檔案中設定下列索引鍵值：

* MicrosoftAppId
* MicrosoftAppPassword

## <a name="test-the-bot"></a>測試 Bot

現在，既然您已建立 Bot，請[在網路聊天中進行測試](bot-service-manage-test-webchat.md)。 輸入訊息，您的 Bot 應會回應。

## <a name="next-steps"></a>後續步驟

本主題中，您已了解如何使用 Bot 服務註冊裝載的 Bot。 下一個步驟是了解如何管理您的 Bot 服務。

> [!div class="nextstepaction"]
> [管理 Bot](bot-service-manage-overview.md)

