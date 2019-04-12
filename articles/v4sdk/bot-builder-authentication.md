---
title: 透過 Azure Bot 服務將驗證新增至您的 Bot | Microsoft Docs
description: 了解如何使用 Azure Bot 服務驗證功能以便將 SSO 新增至您的 Bot。
author: JonathanFingold
ms.author: v-jofing
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: abs
ms.date: 04/09/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 27c97d257261a6f3b9d867503aee40382b685e20
ms.sourcegitcommit: 562dd44e38abacaa31427da5675da556a970cf11
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/10/2019
ms.locfileid: "59477101"
---
# <a name="add-authentication-to-your-bot-via-azure-bot-service"></a>透過 Azure Bot 服務將驗證新增至您的 Bot

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

Azure Bot 服務和 v4 SDK 包含全新的 Bot 驗證功能，並提供相關功能，讓您輕鬆開發可對 Azure AD (Azure Active Directory)、GitHub、Uber 等不同識別提供者驗證使用者的 Bot。 這些功能也讓部分用戶端無須進行_神奇代碼驗證 (Magic code verification)_，而改善使用者體驗。

在此之前，您的 Bot 必須加入 OAuth 控制器和登入連結、儲存目標用戶端識別碼及密碼，並執行使用者權杖管理作業。 Bot 會要求使用者登入網站，繼而產生使用者可用來驗證其身分識別的_神奇代碼_。

現在，Bot 開發人員無須際遇主控 OAuth 控制器或管理權杖的生命週期，這些工作現在全都可由 Azure Bot 服務執行。

這些功能包括：

- 改善通道以支援全新驗證功能 (例如新的 WebChat 和 DirectLineJS 程式庫)，無須再使用 6 位數神奇代碼驗證方式。
- 改善 Azure 入口網站，以針對不同的 OAuth 身分識別提供者新增、刪除和配置連線設定。
- 支援各種現成的身分識別提供者服務，例如：Azure AD (包括 v1 和 v2 端點)、GitHub 等等。
- 更新 C# 和 Node.js Bot Framework SDK，以支援擷取權杖、建立 OAuthCard 及處理 TokenResponse 事件。
- 如何建立能對 Azure AD 進行驗證的 Bot 範例。

您可從本文中的步驟推測如何將這類功能新增至現有的 Bot。 下列範例 Bot 將展示新的驗證功能。

> [!NOTE]
> 驗證功能亦可搭配 BotBuilder v3 運作。 不過，本文內容僅涵蓋 v4 範例程式碼。

### <a name="about-this-sample"></a>關於此範例

您必須建立 Azure Bot 資源，且必須建立新的 Azure AD (v1 或 v2) 應用程式，讓 Bot 能夠存取 Office 365。 Bot 資源會註冊您的 Bot 認證；您需要這些認證來測試驗證功能，即使您在本機執行 Bot 程式碼亦然。

> [!IMPORTANT]
> 每個在 Azure 中註冊的 Bot，都會被指派一個 Azure AD 應用程式。 不過，此應用程式會保護通道對 Bot 的存取。
每個應用程式都還額外需要一個您要讓 Bot 能夠代表使用者，進行驗證的 AAD 應用程式。

本文說明使用 Azure AD v1 或 v2 權杖連線至 Microsoft Graph 的範例 Bot。 此外也說明如何建立及註冊相關聯的 Azure AD 應用程式。 而在此程序中，您將使用 [Microsoft/BotBuilder-Samples](https://github.com/Microsoft/BotBuilder-Samples) GitHub 存放庫中的程式碼。 本文將說明這些程序。

- **建立 Bot 資源**
- **建立 Azure AD 應用程式**
- **向 Bot 註冊 Azure AD 應用程式**
- **準備 Bot 範例程式碼**

完成後，您將有一個在本機執行的 Bot 能夠對 Azure AD 應用程式回應幾項簡單工作，例如，檢查及傳送電子郵件，或者顯示您或管理員的身分。 您的 Bot 必須針對 Microsoft.Graph 程式庫使用來自 Azure AD 應用程式的權杖，才能達到上述目的。 您不需要發佈 Bot 以測試 OAuth 登入功能；不過，您的 Bot 將需要有效的 Azure 應用程式識別碼和密碼。

這些驗證功能也可與其他類型的 Bot 搭配使用。 不過，本文將使用僅具有註冊功能的 Bot。

### <a name="web-chat-and-direct-line-considerations"></a>網路聊天和 Direct Line 的考量

<!-- Summarized from: https://blog.botframework.com/2018/09/25/enhanced-direct-line-authentication-features/ -->

在網路聊天使用 Azure Bot 服務驗證時，必須考量幾個重要的安全性問題。

1. 防止攻擊者進行模擬而使 Bot 將其誤認為其他人。 在網路聊天中，攻擊者可藉由變更他人網路聊天執行個體的使用者識別碼，來模擬其他人。

    若要防止此問題發生，請使用難以猜測的使用者識別碼。 當您在 Direct Line 通道中啟用增強式驗證選項時，Azure Bot 服務可偵測並拒絕任何使用者識別碼變更。 從 Direct Line 到 Bot 的訊息，一律會使用您用來初始化網路聊天的相同使用者識別碼。 請注意，此功能需要以 `dl_` 開頭的使用者識別碼。

1. 請確實登入正確的使用者。 使用者有兩個身分識別：通道中的身分識別，和用於識別提供者的身分識別。 在網路聊天中，Azure Bot 服務可確保登入程序會在與網路聊天本身相同的瀏覽器工作階段中完成。

    若要啟用這項保護，請以 Direct Line 權杖啟動網路聊天，且該權杖必須包含可裝載 Bot 網路聊天用戶端的信任網域清單。 然後，在 Direct Line 組態頁面中以靜態方式指定信任網域 (原始) 清單。

使用 Direct Line 的 `/v3/directline/tokens/generate` REST 端點產生對話的權杖，並指定要求承載中的使用者識別碼。 如需程式碼範例，請參閱[增強型 Direct Line 驗證功能](https://blog.botframework.com/2018/09/25/enhanced-direct-line-authentication-features/)部落格文章。

<!-- The eventual article about this should talk about the tokens/generate endpoint and its parameters: user, trustedOrigins, and [maybe] eTag.
Sample payload
{
  "user": {
    "id": "string",
    "name": "string",
    "aadObjectId": "string",
    "role": "string"
  },
  "trustedOrigins": [
    "string"
  ],
  "eTag": "string"
}
 -->

## <a name="prerequisites"></a>必要條件

- [Bot 基本概念][concept-basics]和[管理狀態][concept-state]的知識。
- Azure 和 OAuth 2.0 開發的知識。
- Visual Studio 2017 或更新版本、Node.js、npm 和 git。
- 以下其中一個範例。

| 範例 | BotBuilder 版本 | 示範 |
|:---|:---:|:---|
| [**CSharp**][cs-auth-sample] 或 [**JavaScript**][js-auth-sample] 中的 **Bot 驗證** | v4 | OAuthCard 支援 |
| [**CSharp**][cs-msgraph-sample] 或 [**JavaScript**][js-msgraph-sample] 中的 **Bot 驗證 MSGraph** | v4 |  OAuth 2 的 Microsoft Graph API 支援 |

## <a name="create-your-bot-resource-on-azure"></a>在 Azure 上建立 Bot 資源

使用 [Azure 入口網站](https://portal.azure.com/)建立 **Bot 通道註冊**。

## <a name="create-and-register-an-azure-ad-application"></a>建立和註冊 Azure AD 應用程式

您需要一個可讓 Bot 用來連線至 Microsoft Graph API 的 Azure AD 應用程式。

針對此 Bot，您可使用 Azure AD v1 或 v2 端點。
如需 v1 和 v2 端點之間差異的相關資訊，請參閱 [v1 與 v2 比較](https://docs.microsoft.com/azure/active-directory/develop/active-directory-v2-compare)，以及 [Azure AD v2.0 端點概觀](https://docs.microsoft.com/azure/active-directory/develop/active-directory-appmodel-v2-overview)。

### <a name="create-your-azure-ad-application"></a>建立 Azure AD 應用程式

建立下列步驟建立新的 Azure AD 應用程式。 您可以將 v1 或 v2 端點與所建立的應用程式一起使用。

> [!TIP]
> 您必須在具有管理員權限的租用戶中，建立並登錄 Azure AD 應用程式。

1. 在 Azure 入口網站中開啟 [Azure Active Directory][azure-aad-blade] 面板。
    如果您不在正確的租用戶中，請按一下 [切換目錄] 以切換至正確的租用戶。 (如需建立租用戶的指示，請參閱[存取入口網站並建立租用戶](https://docs.microsoft.com/en-us/azure/active-directory/fundamentals/active-directory-access-create-new-tenant)。)
1. 開啟 [應用程式註冊] 面板。
1. 在 [應用程式註冊] 面板中，按一下 [新增應用程式註冊]。
1. 填寫必要欄位並建立應用程式註冊。

   1. 為您的應用程式命名。
   1. 將 [應用程式類型] 設為 [Web 應用程式/API]。
   1. 將 [登入 URL] 設為 `https://token.botframework.com/.auth/web/redirect`。
   1. 按一下頁面底部的 [新增] 。

      - 建立後，即會顯示在 [註冊的應用程式] 窗格中。
      - 記錄 [應用程式識別碼] 值。 稍後當您向 Bot 註冊 Azure AD 應用程式時，您將使用此值作為_用戶端識別碼_。

1. 按一下 [設定] 以設定應用程式。
1. 按一下 [金鑰] 以開啟 [金鑰] 面板。

   1. 在 [密碼] 下方建立 `BotLogin` 金鑰。
   1. 將其 [持續時間] 設為 [永不過期]。
   1. 按一下 [儲存] 並記錄金鑰值。 稍後當您向 Bot 註冊 Azure AD 應用程式時，您將使用此值作為_用戶端密碼_。
   1. 關閉 [金鑰] 面板。

1. 按一下 [必要權限] 以開啟 [必要權限] 面板。

   1. 按一下 [新增] 。
   1. 按一下 [選取 API]，然後選取 [Microsoft Graph]，再按一下 [選取]。
   1. 按一下 [選取權限]。 選擇您的應用程式將使用的委派權限。

      > [!NOTE]
      > 只要是標記為 [需要系統管理員] 的權限，則使用者和租用戶管理員皆必須登入，才能將 Bot 隔離在這些權限之外。

      選取下列 Microsoft Graph 委派的權限：
      - 讀取所有使用者的基本個人資料
      - 讀取使用者的郵件
      - 以使用者身分傳送郵件
      - 登入及讀取使用者設定檔
      - 檢視所有使用者的基本個人資料
      - 檢視使用者的電子郵件地址

   1. 按一下 [選取]，然後按一下 [完成]。
   1. 關閉 [必要權限] 面板。

您現在已完成 Azure AD v1 應用程式設定。

### <a name="register-your-azure-ad-application-with-your-bot"></a>向 Bot 註冊 Azure AD 應用程式

下一步是向 Bot 註冊您已建立的 Azure AD 應用程式。

# [<a name="azure-ad-v1"></a>Azure AD v1](#tab/aadv1)

1. 在 [Azure 入口網站](http://portal.azure.com/)瀏覽至 Bot 的資源頁面。
1. 按一下 [設定] 。
1. 在靠近頁面底部的 [OAuth 連線設定] 下方，按一下 [新增設定]。
1. 填寫表單，如下所示：

    1. 若為 [名稱] 欄位，請輸入連線的名稱。 您將在 Bot 程式碼中使用此名稱。
    1. 若為 [服務提供者]，請選取 [Azure Active Directory]。 選取此項目後，Azure AD 專用的欄位隨即顯示。
    1. 若為 [用戶端識別碼] 欄位，請輸入您為 Azure AD v1 應用程式記錄的應用程式識別碼。
    1. 若為 [用戶端密碼] 欄位，請輸入您為應用程式 `BotLogin` 金鑰記錄的金鑰。
    1. 若為 [授與類型] 欄位，請輸入「`authorization_code`」。
    1. 若為 [登入 URL] 欄位，請輸入「`https://login.microsoftonline.com`」。
    1. 若為 [租用戶識別碼] 欄位，請輸入 Azure Active Directory 的租用戶識別碼，例如：`microsoft.com` 或 `common`。

       這將會是與可驗證之使用者相關聯的租用戶。 若要允許任何人透過 Bot 驗證身分，請使用 `common` 租用戶。

    1. 針對 [資源 URL]，輸入 `https://graph.microsoft.com/`。
    1. 請將 [範圍] 欄位保留空白。

1. 按一下 [檔案] 。

> [!NOTE]
> 這些值可讓應用程式透過 Microsoft Graph API 存取 Office 365 資料。

# [<a name="azure-ad-v2"></a>Azure AD v2](#tab/aadv2)

1. 在 [Azure 入口網站](http://portal.azure.com/)上瀏覽至 Bot 的 [Bot 通道註冊] 頁面。
1. 按一下 [設定] 。
1. 在靠近頁面底部的 [OAuth 連線設定] 下方，按一下 [新增設定]。
1. 填寫表單，如下所示：

    1. 若為 [名稱] 欄位，請輸入連線的名稱。 Bot 程式碼中會用到此名稱。
    1. 若為 [服務提供者] ，請選取 [Azure Active Directory v2]。 選取此項目後，Azure AD 專用的欄位隨即顯示。
    1. 若為 [用戶端識別碼] 欄位，請輸入應用程式註冊時的 Azure AD v2 應用程式識別碼。
    1. 若為 [用戶端密碼] 欄位，請輸入應用程式註冊時的 Azure AD v2 應用程式密碼。
    1. 若為 [租用戶識別碼] 欄位，請輸入 Azure Active Directory 的租用戶識別碼，例如：`microsoft.com` 或 `common`。

        這將會是與可驗證之使用者相關聯的租用戶。 若要允許任何人透過 Bot 驗證身分，請使用 `common` 租用戶。

    1. 若為 [範圍] 欄位，請輸入應用程式註冊時您選擇之權限的名稱：`Mail.Read Mail.Send openid profile User.Read User.ReadBasic.All`。

        > [!NOTE]
        > 若為 Azure AD v2，[範圍] 欄位接受區分大小寫並以空格區隔值的清單。

1. 按一下 [檔案] 。

> [!NOTE]
> 這些值可讓應用程式透過 Microsoft Graph API 存取 Office 365 資料。

---

### <a name="test-your-connection"></a>測試連線

1. 按一下連線項目以開啟您剛剛建立的連線。
1. 按一下 [服務提供者連線設定] 窗格頂端的 [測試連線]。
1. 第一次將開啟新的瀏覽器索引標籤，其中列出應用程式要求的權限，並提示您接受要求。
1. 按一下 [接受]。
1. 這應會將您重新導向至 [測試對 \<your-connection-name> 的連線已成功] 頁面。

您現可在 Bot 程式碼中使用此連線名稱，以擷取使用者權杖。

## <a name="prepare-the-bot-sample-code"></a>準備 Bot 範例程式碼

視您所選的範例而定，您會使用 C# 或 Node。

| 範例 | BotBuilder 版本 | 示範 |
|:---|:---:|:---|
| [**CSharp**][cs-auth-sample] 或 [**JavaScript**][js-auth-sample] 中的 **Bot 驗證** | v4 | OAuthCard 支援 |
| [**CSharp**][cs-msgraph-sample] 或 [**JavaScript**][js-msgraph-sample] 中的 **Bot 驗證 MSGraph** | v4 |  OAuth 2 的 Microsoft Graph API 支援 |

1. 按一下以上其中一個範例連結並複製 github 存放庫。
1. 遵循 GitHub 讀我檔案頁面上的指示，了解如何執行該特定 Bot (C# 或 Node)。
1. 如果您使用 C# Bot-Authentication 範例：

    1. 將 `AuthenticationBot.cs` 檔案中的 `ConnectionName` 參數設為您在配置 Bot 的 OAuth 2.0 連線設定時所用的值。
    1. 將 `BotConfiguration.bot` 檔案中的 `appId` 值設為 Bot 的應用程式識別碼。
    1. 將 `BotConfiguration.bot` 檔案中的 `appPassword` 值設為 Bot 的祕密。

1. 如果您使用 Node/JS Bot-Authentication 範例：

    1. 將 `bot.js` 檔案中的 `CONNECTION_NAME` 參數設為您在配置 Bot 的 OAuth 2.0 連線設定時所用的值。
    1. 將 `bot-authentication.bot` 檔案中的 `appId` 值設為 Bot 的應用程式識別碼。
    1. 將 `bot-authentication.bot` 檔案中的 `appPassword` 值設為 Bot 的祕密。

    > [!IMPORTANT]
    > 視密碼中的字元而定，您可能需要讓 XML 逸出該密碼。 例如，任何 & 符號都必須編碼為 `&amp;`。

    ```json
    {
        "name": "BotAuthentication",
        "secretKey": "",
        "services": [
            {
            "appId": "",
            "id": "http://localhost:3978/api/messages",
            "type": "endpoint",
            "appPassword": "",
            "endpoint": "http://localhost:3978/api/messages",
            "name": "BotAuthentication"
            }
        ]
    }
    ```

    如果您不知道如何取得 **Microsoft 應用程式識別碼**和 **Microsoft 應用程式密碼**值，請找到您在 Azure 入口網站上為 Bot 佈建的 Azure 應用程式服務，然後查看其中的 **ApplicationSettings**。

    > [!NOTE]
    > 您現在可將此 Bot 程式碼重新發佈至 Azure 訂用帳戶 (以滑鼠右鍵按一下專案，然後選擇 [發佈])，但此動作在本教學課程的範例中並非必要。 在 Azure 入口網站中配置 Bot 時，您必須進行發佈設定，其應使用您所用的應用程式和主控方案。

## <a name="use-the-emulator-to-test-your-bot"></a>使用模擬器測試您的 Bot

您必須安裝 [Bot 模擬器](https://github.com/Microsoft/BotFramework-Emulator)以便在本機測試 Bot。 您可使用 v3 或 v4 模擬器。

1. 啟動 Bot (無論是否啟用偵錯)。
1. 請記下該頁面的 localhost 連接埠號碼。 稍後與 Bot 互動時，您會需要使用此資訊。
1. 啟動模擬器。
1. 連線至 Bot。 請確定 Bot 組態會在使用驗證時使用 **Microsoft 應用程式識別碼**和 **Microsoft 應用程式密碼**
1. 在模擬器設定中，確定已核取 [使用 OAuthCard 的登入驗證程式碼] 並已啟用 **ngrok**，所以 Azure Bot Service 可以將可用的權杖傳回模擬器。

   如果您尚未設定連線，請提供位址及 Bot 的 Microsoft 應用程式識別碼和密碼。 將 `/api/messages` 新增至 Bot 的 URL。 URL 外觀應該會類似於 `http://localhost:portNumber/api/messages`。

1. 輸入「`help`」以檢視適用於 Bot 的可用命令清單，以及測試驗證功能。
1. 登入後一直到登出前，您都不需要再次提供認證。
1. 若要登出並取消驗證，請輸入「`signout`」。

<!--To restart completely from scratch you also need to:
1. Navigate to the **AppData** folder for your account.
1. Go to the **Roaming/botframework-emulator** subfolder.
1. Delete the **Cookies** and **Coolies-journal** files.
-->

> [!NOTE]
> Bot 驗證需要使用 Bot 連接器服務。 該服務將針對您的 Bot 存取 Bot 通道註冊資訊，因此您必須在入口網站上設定 Bot 的傳訊端點。 此外，驗證也會需要使用 HTTPS，因此您必須建立 HTTPS 轉送位址，以供 Bot 在本機上執行。

<!--The following is necessary for WebChat:
1. Use the **ngrok** command-line tool to get a forwarding HTTPS address for your bot.
   - For information on how to do this, see [Debug any Channel locally using ngrok](https://blog.botframework.com/2017/10/19/debug-channel-locally-using-ngrok/).
   - Any time you exit **ngrok**, you will need to redo this and the following step before starting the Emulator.
1. On the Azure Portal, go to the **Settings** blade for your bot.
   1. In the **Configuration** section, change the **Messaging endpoint** to the HTTPS forwarding address generated by **ngrok**.
   1. Click **Save** to save your change.
-->

## <a name="notes-on-the-token-retrieval-flow"></a>權杖擷取流程注意事項

使用者要求 Bot 執行需要使用者登入的工作時，Bot 可使用 `OAuthPrompt` 起始擷取特定連線的權杖。 `OAuthPrompt` 會建立權杖擷取流程，其中包含：

1. 查看 Azure Bot Service 是否已經具有可供目前使用者和連線使用的權杖。 如果有權杖，則會傳回權杖。
1. 如果 Azure Bot Service 沒有快取的權杖，則會建立 `OAuthCard`，這是使用者可以按一下的登入按鈕。
1. 在使用者按一下 `OAuthCard` 登入按鈕之後，Azure Bot Service 會將使用者的權杖傳直接送給 Bot，或者會向使用者顯示要在聊天視窗中輸入的 6 位數驗證碼。
1. 如果使用者看到驗證碼，則 Bot 會將此驗證碼換成使用者的權杖。

接下來的幾個程式碼片段皆取自 `OAuthPrompt`，以顯示這些步驟在提示字元中的運作方式。

### <a name="check-for-a-cached-token"></a>檢查已快取的權杖

在此程式碼中，首先 Bot 會快速檢查以判斷 Azure Bot 服務是否已有該使用者的權杖 (藉由目前的活動傳送者識別) 和指定的 ConnectionName (即設定中使用的連線名稱)。 Azure Bot 服務可能會快取權杖，也可能不會。 針對 GetUserTokenAsync 的呼叫會執行此快速檢查。 如果 Azure Bot 服務具有權杖並將其傳回，則該權杖可立即使用。 如果 Azure Bot 服務沒有權杖，此方法將傳回 Null。 在此情況下，Bot 可傳送自訂的 OAuthCard 供使用者登入。

# [<a name="c"></a>C#](#tab/csharp)

```csharp
// First ask Bot Service if it already has a token for this user
var token = await adapter.GetUserTokenAsync(turnContext, connectionName, null, cancellationToken).ConfigureAwait(false);
if (token != null)
{
    // use the token to do exciting things!
}
else
{
    // If Bot Service does not have a token, send an OAuth card to sign in
}
```

# [<a name="javascript"></a>JavaScript](#tab/javascript)

```javascript
public async getUserToken(context: TurnContext, code?: string): Promise<TokenResponse|undefined> {
    // Get the token and call validator
    const adapter: any = context.adapter as any; // cast to BotFrameworkAdapter
    return await adapter.getUserToken(context, this.settings.connectionName, code);
}
```

---

### <a name="send-an-oauthcard-to-the-user"></a>傳送 OAuthCard 給使用者

您可將 OAuthCard 自訂為任何您想用的文字和按鈕文字。 需要注意的重點如下：

- 將 `ContentType` 設為 `OAuthCard.ContentType`。
- 將 `ConnectionName` 屬性設為您要使用之連線的名稱。
- 包含一個具有 `Type` `ActionTypes.Signin` 之 `CardAction` 的按鈕；請注意，您不需要為登入連結指定任何值。

在此呼叫結束時，Bot 必須「等候權杖」回傳。 由於可能有許多使用者需要登入，因此等候過程會在主要的活動流中發生。

# [<a name="c"></a>C#](#tab/csharp)

```csharp
private async Task SendOAuthCardAsync(ITurnContext turnContext, IMessageActivity message, CancellationToken cancellationToken = default(CancellationToken))
{
    if (message.Attachments == null)
    {
        message.Attachments = new List<Attachment>();
    }

    message.Attachments.Add(new Attachment
    {
        ContentType = OAuthCard.ContentType,
        Content = new OAuthCard
        {
            Text = "Please sign in",
            ConnectionName = connectionName,
            Buttons = new[]
            {
                new CardAction
                {
                    Title = "Sign In",
                    Text = "Sign In",
                    Type = ActionTypes.Signin,
                },
            },
        },
    });

    await turnContext.SendActivityAsync(message, cancellationToken).ConfigureAwait(false);
}
```

# [<a name="javascript"></a>JavaScript](#tab/javascript)

```javascript
private async sendOAuthCardAsync(context: TurnContext, prompt?: string|Partial<Activity>): Promise<void> {
    // Initialize outgoing message
    const msg: Partial<Activity> =
        typeof prompt === 'object' ? {...prompt} : MessageFactory.text(prompt, undefined, InputHints.ExpectingInput);
    if (!Array.isArray(msg.attachments)) { msg.attachments = []; }

    const cards: Attachment[] = msg.attachments.filter((a: Attachment) => a.contentType === CardFactory.contentTypes.oauthCard);
    if (cards.length === 0) {
        // Append oauth card
        msg.attachments.push(CardFactory.oauthCard(
            this.settings.connectionName,
            this.settings.title,
            this.settings.text
        ));
    }

    // Send prompt
    await context.sendActivity(msg);
}
```

---

### <a name="wait-for-a-tokenresponseevent"></a>等候 TokenResponseEvent

在此程式碼中，Bot 正在等候 `TokenResponseEvent` (下文將詳述其如何路由至對話方塊堆疊)。 `WaitForToken` 方法會先判斷此事件是否已傳送。 如果其已傳送，則可供 Bot 使用。 如果未傳送，則 `RecognizeTokenAsync` 方法將擷取任何傳送給 Bot 的文字，再將其傳遞至 `GetUserTokenAsync`。 此作法的原因是部分用戶端 (例如：WebChat) 不需要進行神奇代碼驗證碼，而可直接在 `TokenResponseEvent` 中傳送權杖。 其他用戶端 (例如：Facebook 或 Slack) 仍須使用神奇代碼。 Azure Bot 服務會向用戶端顯示一組六位數的神奇代碼，並要求使用者在聊天視窗中輸入該代碼。 此方式並非最理想的「回復」行為，因此，假如 `RecognizeTokenAsync` 收到代碼，則 Bot 可將代碼傳送至 Azure Bot Service 並獲取權杖。 若此呼叫也失敗，則您可決定是否要回報錯誤或採取其他行動。 不過大部分情況下，Bot 現在都會有使用者權杖。

若您檢視每個範例的 Bot 程式碼，即可發現 `Event` 和 `Invoke` 活動也會路由傳送至對話方塊堆疊。

# [<a name="c"></a>C#](#tab/csharp)

```csharp
// This can be called when the bot receives an Activity after sending an OAuthCard
private async Task<TokenResponse> RecognizeTokenAsync(ITurnContext turnContext, CancellationToken cancellationToken = default(CancellationToken))
{
    if (IsTokenResponseEvent(turnContext))
    {
        // The bot received the token directly
        var tokenResponseObject = turnContext.Activity.Value as JObject;
        var token = tokenResponseObject?.ToObject<TokenResponse>();
        return token;
    }
    else if (IsTeamsVerificationInvoke(turnContext))
    {
        var magicCodeObject = turnContext.Activity.Value as JObject;
        var magicCode = magicCodeObject.GetValue("state")?.ToString();

        var token = await adapter.GetUserTokenAsync(turnContext, _settings.ConnectionName, magicCode, cancellationToken).ConfigureAwait(false);
        return token;
    }
    else if (turnContext.Activity.Type == ActivityTypes.Message)
    {
        // make sure it's a 6-digit code
        var matched = _magicCodeRegex.Match(turnContext.Activity.Text);
        if (matched.Success)
        {
            var token = await adapter.GetUserTokenAsync(turnContext, _settings.ConnectionName, matched.Value, cancellationToken).ConfigureAwait(false);
            return token;
        }
    }

    return null;
}

private bool IsTokenResponseEvent(ITurnContext turnContext)
{
    var activity = turnContext.Activity;
    return activity.Type == ActivityTypes.Event && activity.Name == "tokens/response";
}

private bool IsTeamsVerificationInvoke(ITurnContext turnContext)
{
    var activity = turnContext.Activity;
    return activity.Type == ActivityTypes.Invoke && activity.Name == "signin/verifyState";
}
```

# [<a name="javascript"></a>JavaScript](#tab/javascript)

```javascript
private async recognizeToken(context: TurnContext): Promise<PromptRecognizerResult<TokenResponse>> {
    let token: TokenResponse|undefined;
    if (this.isTokenResponseEvent(context)) {
        token = context.activity.value as TokenResponse;
    } else if (this.isTeamsVerificationInvoke(context)) {
        const code: any = context.activity.value.state;
        await context.sendActivity({ type: 'invokeResponse', value: { status: 200 }});
        token = await this.getUserToken(context, code);
    } else if (context.activity.type === ActivityTypes.Message) {
        const matched: RegExpExecArray = /(\d{6})/.exec(context.activity.text);
        if (matched && matched.length > 1) {
            token = await this.getUserToken(context, matched[1]);
        }
    }

    return token !== undefined ? { succeeded: true, value: token } : { succeeded: false };
}

private isTokenResponseEvent(context: TurnContext): boolean {
    const activity: Activity = context.activity;
    return activity.type === ActivityTypes.Event && activity.name === 'tokens/response';
}

private isTeamsVerificationInvoke(context: TurnContext): boolean {
    const activity: Activity = context.activity;
    return activity.type === ActivityTypes.Invoke && activity.name === 'signin/verifyState';
}
```

---

### <a name="message-controller"></a>訊息控制器

在對 Bot 的後續呼叫中，請注意此範例 Bot 永遠不會快取權杖。 這是因為 Bot 可一律向 Azure Bot 服務要求權杖。 當 Azure Bot 服務為您執行一切工作時，如此可避免 Bot 必須管理權杖生命週期、更新權杖等繁瑣作業。

### <a name="further-reading"></a>進階閱讀

- [Bot Framework 其他資源](https://docs.microsoft.com/azure/bot-service/bot-service-resources-links-help)包含其他支援的連結。
- [Bot Framework SDK](https://github.com/microsoft/botbuilder) 存放庫提供更多與 Bot Builder SDK 相關聯的存放庫、範例、工具和規格的詳細資訊。

<!-- Footnote-style links -->

[Azure portal]: https://ms.portal.azure.com
[azure-aad-blade]: https://ms.portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/Overview
[aad-registration-blade]: https://ms.portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/RegisteredApps

[concept-basics]: bot-builder-basics.md
[concept-state]: bot-builder-concept-state.md
[concept-dialogs]: bot-builder-concept-dialog.md

[simple-dialog]: bot-builder-dialog-manage-conversation-flow.md
[dialog-prompts]: bot-builder-prompts.md
[component-dialogs]: bot-builder-compositcontrol.md

[cs-auth-sample]: https://aka.ms/v4cs-bot-auth-sample
[js-auth-sample]: https://aka.ms/v4js-bot-auth-sample
[cs-msgraph-sample]: https://aka.ms/v4cs-auth-msgraph-sample
[js-msgraph-sample]: https://aka.ms/v4js-auth-msgraph-sample
