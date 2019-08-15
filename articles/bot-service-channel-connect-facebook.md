---
title: 將 Bot 連線到 Facebook Messenger | Microsoft Docs
description: 了解如何設定 Bot 與 Facebook Messenger 的連線。
keywords: Facebook Messenger, Bot 通道, Facebook 應用程式, 應用程式識別碼, 應用程式祕密, Facebook Bot, 認證
manager: kamrani
ms.topic: article
ms.author: kamrani
ms.service: bot-service
ms.date: 08/03/2019
ms.openlocfilehash: 4e5dc332b463e9490c7aa265a08e8f126d59d3f9
ms.sourcegitcommit: 6a83b2c8ab2902121e8ee9531a7aa2d85b827396
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/09/2019
ms.locfileid: "68866586"
---
# <a name="connect-a-bot-to-facebook"></a>將 Bot 連線到 Facebook

您的 Bot 可以連線到 Facebook Messenger 和 Facebook Workplace，以便與這兩個平台上的使用者進行通訊。 下列教學課程示範如何將 Bot 連線到這兩個通道。

> [!NOTE]
> Facebook UI 可能會因為您所使用的版本而略有不同。

## <a name="connect-a-bot-to-facebook-messenger"></a>將 Bot 連線到 Facebook Messenger

若要深入了解如何針對 Facebook Messenger 進行開發，請參閱 [Messenger 平台文件](https://developers.facebook.com/docs/messenger-platform)。 您可以檢閱 Facebook 的[啟動前指導方針](https://developers.facebook.com/docs/messenger-platform/product-overview/launch#app_public)、[快速入門](https://developers.facebook.com/docs/messenger-platform/guides/quick-start)和[設定指南](https://developers.facebook.com/docs/messenger-platform/guides/setup)。

若要設定 Bot 使用 Facebook Messenger 進行通訊，請在 Facebook 頁面上啟用 Facebook Messenger，然後將 Bot 連線。

### <a name="copy-the-page-id"></a>複製頁面識別碼

已透過 Facebook 頁面存取 Bot。

1. 請[建立新的 Facebook 頁面](https://www.facebook.com/bookmarks/pages)或移至現有的頁面。

1. 請開啟 Facebook 頁面的 [關於]  頁面，然後複製並儲存 [頁面識別碼]  。

### <a name="create-a-facebook-app"></a>建立 Facebook 應用程式

1. 在您的瀏覽器中，瀏覽至[建立新的 Facebook 應用程式](https://developers.facebook.com/quickstarts/?platform=web)。
1. 輸入應用程式的名稱，然後按一下 [建立新的 Facebook 應用程式識別碼]  按鈕。

    ![建立應用程式](media/channels/fb-create-messenger-bot-app.png)

1. 在顯示的對話方塊中，輸入您的電子郵件地址，然後按一下 [建立應用程式識別碼]  按鈕。

    ![建立應用程式識別碼](media/channels/fb-create-messenger-bot-app-id.png)

1. 完成精靈步驟。

1. 輸入必要的檢查資訊，然後按一下右上方的 [略過快速入門]  按鈕。

1. 在下一個顯示視窗的左窗格中，展開 [設定]  ，然後按一下 [基本]  。

1. 在右窗格中，複製並儲存 [應用程式識別碼]  和 [應用程式秘密]  。

    ![複製應用程式識別碼和應用程式秘密](media/channels/fb-messenger-bot-get-appid-secret.png)

1. 在左窗格的 [設定]  下方，按一下 [進階]  。

1. 在右窗格中，將 [允許 API 存取應用程式設定]  滑桿設為 [是]  。

    ![複製應用程式識別碼和應用程式秘密](media/channels/fb-messenger-bot-api-settings.png)

1. 在右下方的頁面中，按一下 [儲存變更]  按鈕。

### <a name="enable-messenger"></a>啟用 Messenger

1. 在左窗格中，按一下 [儀表板]  。
1. 在右窗格中向下捲動，並在 [Messenger]  方塊中按一下 [設定]  按鈕。 Messenger 項目會顯示在左窗格的 [產品]  區段底下。  

    ![啟用 Messenger](media/channels/fb-messenger-bot-enable-messenger.png)

### <a name="generate-a-page-access-token"></a>產生頁面存取權杖

1. 在左窗格的 Messenger 項目底下，按一下 [設定]  。
1. 在右窗格中向下捲動，並在 [權杖產生]  區段中，選取目標頁面。

    ![啟用 Messenger](media/channels/fb-messenger-bot-select-messenger-page.png)

1. 按一下 [編輯權限]  按鈕以授與應用程式 pages_messaging，以便產生存取權杖。
1. 遵循精靈的步驟。 在最後一個步驟中，接受預設設定，然後按一下 [完成]  按鈕。 最後便會產生**頁面存取權杖**。

    ![Messenger 權限](media/channels/fb-messenger-bot-permissions.png)

1. 複製並儲存 [頁面存取權杖]  。

### <a name="enable-webhooks"></a>啟用 Webhook

為了從您的 Bot 傳送訊息和其他事件至 Facebook Messenger，您必須啟用 Webhook 整合。 此時，讓我們先將 Facebook 設定步驟擱置，之後再回來設定。

1. 在您的瀏覽器中開啟新視窗，並瀏覽至 [Azure 入口網站](https://portal.azure.com/)。 

1. 在資源清單中，按一下 Bot 資源註冊，並且在相關的刀鋒視窗中按一下 [通道]  。

1. 在右窗格中，按一下 **Facebook** 圖示。

1. 在精靈中，輸入先前步驟中儲存的 Facebook 資訊。 如果資訊正確，在精靈的底部，您應該會看到**回呼 URL** 和**驗證權杖**。 請複製並儲存該資訊。  

    ![fb messenger 通道設定](media/channels/fb-messenger-bot-config-channel.PNG)

1. 按一下 [儲存]  按鈕。


1. 讓我們回到 Facebook 設定。 在右窗格中向下捲動，並在 [Webhook]  區段中按一下 [訂閱事件]  按鈕。 這是要將訊息事件從 Facebook Messenger 轉送到 Bot。

    ![啟用 Webhook](media/channels/fb-messenger-bot-webhooks.PNG)

1. 在顯示的對話方塊中，輸入先前儲存的 **回呼 URL** 和**驗證權杖**值。 在 [訂用帳戶欄位]  底下，選取 [message\_deliveries]  、[messages]  、[messaging\_options]  及 [messaging\_postbacks]  。

    ![設定 Webhook](media/channels/fb-messenger-bot-config-webhooks.png)

1. 按一下 [驗證並儲存]  按鈕。
1. 選取要訂閱 Webhook 的 Facebook 頁面。 按一下 [訂閱]  按鈕。

    ![設定 Webhook 頁面](media/channels/fb-messenger-bot-config-webhooks-page.PNG)

### <a name="submit-for-review"></a>提交以供審查

Facebook 需要它基本應用程式設定頁面上的 [隱私權原則 URL] 和 [服務條款 URL]。 [管理辦法](https://investor.fb.com/corporate-governance/code-of-conduct/default.aspx)頁面包含第三方資源連結，可以協助建立隱私權原則。 [使用條款](https://www.facebook.com/terms.php)頁面包含範例條款，可以協助建立適當的「服務條款」文件。

Bot 完成之後，Facebook 對於發佈到 Messenger 的應用程式有自己的[審查程序](https://developers.facebook.com/docs/messenger-platform/app-review)。 系統會測試 Bot 以確保它符合 Facebook 的[平台原則](https://developers.facebook.com/docs/messenger-platform/policy-overview)規範。

### <a name="make-the-app-public-and-publish-the-page"></a>將應用程式設為公用並發佈至頁面

> [!NOTE]
> 在應用程式發佈之前，它都處於[開發模式](https://developers.facebook.com/docs/apps/managing-development-cycle)。 外掛程式和 API 功能僅適用於系統管理員、開發人員和測試人員。

審查成功之後，請在 [應用程式審查] 底下的 [應用程式儀表板] 中，將應用程式設為 [公用]。
確保與這個 Bot 相關聯的 Facebook 頁面已發佈。 狀態會顯示在頁面設定中。

## <a name="connect-a-bot-to-facebook-workplace"></a>將 Bot 連線到 Facebook Workplace

如需 Facebook Workplace 開發的指導方針，請參閱 [Workplace 說明中心](https://workplace.facebook.com/help/work/)以深入了解 Facebook Workplace 和 [Workplace 開發人員文件](https://developers.facebook.com/docs/workplace)。

若要將 Bot 設定為使用 Facebook Workplace 進行通訊，請建立自訂整合並將 Bot 連線到該整合。


1. 建立 Facebook Workplace Premium 帳戶。 遵循[這裡](https://www.facebook.com/workplace)的指示，建立 Facebook Workplace Premium 帳戶並將自己設定為系統管理員。 請記住，只有 Workplace 的系統管理員可以建立自訂整合。

1. 遵循下列所述的步驟，為您的 Workplace 建立[自訂整合](https://developers.facebook.com/docs/workplace/custom-integrations-new)。 當您建立自訂整合時，只會建立您的 Workplace 社群內可見且已定義權限的應用程式和 'Bot' 類型的頁面。

1. 在 [管理面板]  中，開啟 [整合]  索引標籤。
1. 按一下 [建立自己的自訂應用程式]  按鈕。

    ![Workplace 整合](media/channels/fb-integration.png)

1. 選擇應用程式的顯示名稱和設定檔圖片。 這類資訊會與 'Bot' 類型的頁面共用。
1. 將 [允許 API 存取應用程式設定]  設定為 [是]。
1. 複製並安全地儲存您所看見的應用程式識別碼、應用程式祕密和應用程式權杖。

    ![Workplace 金鑰](media/channels/fb-keys.png)

1. 您現在已經建立好自訂整合。 您可以在 Workplace 社群中尋找 'Bot' 類型的頁面，如下所示。

    ![Workplace 頁面](media/channels/fb-page.png)

### <a name="provide-facebook-credentials"></a>提供 Facebook 認證

在 Azure 入口網站中，貼上先前從 Facebook Workplace 複製的 [Facebook 應用程式識別碼]  、[Facebook 應用程式祕密]  和 [頁面存取權杖]  值。 請使用其 [關於]  頁面上整合名稱後面的數字，而不是傳統的 pageID。 類似於將 Bot 連線至 Facebook Messenger，Webhook 可透過 Azure 中顯示的認證來連線。

### <a name="submit-for-review"></a>提交以供審查
如需詳細資訊，請參閱**將 Bot 連線至 Facebook Messenger** 一節和 [Workplace 開發人員文件](https://developers.facebook.com/docs/workplace)。

### <a name="make-the-app-public-and-publish-the-page"></a>將應用程式設為公用並發佈至頁面
如需詳細資訊，請參閱**將 Bot 連線至 Facebook Messenger** 一節。

## <a name="setting-the-api-version"></a>設定 API 版本

如果您收到來自 Facebook 關於特定 Graph API 版本淘汰的通知，請移至 [Facebook 開發人員頁面](https://developers.facebook.com)。 瀏覽至 Bot 的 [應用程式設定]  並移至 [設定] > [進階] > [升級 API 版本]  ，然後將 [升級所有呼叫]  切換為 3.0。

![API 版本升級](media/channels/fb-version-upgrade.png)

## <a name="see-also"></a>另請參閱

- **範例程式碼**。 使用 <a href="https://aka.ms/facebook-events" target="_blank">Facebook-events</a> 範例 Bot 來探索 Bot 與 Facebook Messenger 的通訊。

- **可作為配接器提供**。 此通道也可[作為配接器提供](https://botkit.ai/docs/v4/platforms/facebook.html)。 為協助您在配接器和通道之間做選擇，請參閱[目前可用的配接器](bot-service-channel-additional-channels.md#currently-available-adapters)。
