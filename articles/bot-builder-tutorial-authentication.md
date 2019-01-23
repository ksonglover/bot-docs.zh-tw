---
title: 透過 Azure Bot 服務將驗證新增至您的 Bot | Microsoft Docs
description: 了解如何使用 Azure Bot 服務驗證功能以便將 SSO 新增至您的 Bot。
author: JonathanFingold
ms.author: JonathanFingold
manager: kamrani
ms.topic: article
ms.service: bot-service
ROBOTS: NOINDEX
ms.date: 10/04/2018
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 33c6f22696038ed5e9d2ae09ad2ec99d401f6a60
ms.sourcegitcommit: b94361234816e6b95459f142add936732fc40344
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/15/2019
ms.locfileid: "54317708"
---
[!INCLUDE [pre-release-label](includes/pre-release-label-v3.md)]

# <a name="add-authentication-to-your-bot-via-azure-bot-service"></a>透過 Azure Bot 服務將驗證新增至您的 Bot
此教學課程將使用 Azure Bot 服務中的全新 Bot 驗證功能，並提供相關功能，讓您輕鬆開發可針對 Azure AD (Azure Active Directory)、GitHub、Uber 等不同身分識別提供者驗證使用者的 Bot。 此外，這些更新也排除了部分用戶端採用的_神奇代碼驗證 (Magic code verification)_，可進一步改善使用者體驗。

在此之前，您的 Bot 必須加入 OAuth 控制器和登入連結、儲存目標用戶端識別碼及密碼，並執行使用者權杖管理作業。
<!--
These capabilities were bundled in the BotAuth and AuthBot samples that are on GitHub.
-->

現在，Bot 開發人員無須際遇主控 OAuth 控制器或管理權杖的生命週期，這些工作現在全都可由 Azure Bot 服務執行。

這些功能包括：

- 改善通道以支援全新驗證功能 (例如新的 WebChat 和 DirectLineJS 程式庫)，無須再使用 6 位數神奇代碼驗證方式。
- 改善 Azure 入口網站，以針對不同的 OAuth 身分識別提供者新增、刪除和配置連線設定。
- 支援各種現成的身分識別提供者服務，例如：Azure AD (包括 v1 和 v2 端點)、GitHub 等等。
- 更新 C# 和 Node.js Bot Framework SDK，以支援擷取權杖、建立 OAuthCard 及處理 TokenResponse 事件。
- 如何建立能針對 Azure AD (包括 v1 和 v2 端點) 和 GitHub 進行驗證之 Bot 的範例。

您可從本文中的步驟推測如何將這類功能新增至現有的 Bot。 以下是展示全新驗證功能的範例 Bot

| 範例 | BotBuilder 版本 | 說明 |
|:---|:---:|:---|
| [AadV1Bot](https://aka.ms/AadV1Bot) | v3 | 展使用 Azure AD v1 端點示 v3 C# SDK 中的 OAuthCard 支援 |
| [AadV2Bot](https://aka.ms/AadV2Bot) | v3 |  使用 Azure AD v2 端點展示 v3 C# SDK 中的 OAuthCard 支援 |
| [GitHubBot](https://aka.ms/GitHubBot) | v3 |  使用 GitHub 展示 v3 C# SDK 中的 OAuthCard 支援， |
| [BasicOAuth](https://aka.ms/BasicOAuth) | v3 |  展示 v3 C# SDK 中的 OAuth 2.0 支援 |

> [!NOTE]
> 驗證功能亦可搭配 Node.js 和 BotBuilder v3 使用。 不過，本文內容僅涵蓋範例 C# 程式碼。

如需其他資訊和支援，請參閱 [Bot Framework 其他資源](https://docs.microsoft.com/azure/bot-service/bot-service-resources-links-help)。

## <a name="overview"></a>概觀

本教學課程使用 Azure AD v1 或 v2 權杖建立了一個連線至 Microsoft Graph 的範例 Bot。 <!--verify this info and fix wording--> 而在此程序中，您將使用來自 GitHub 存放庫的程式碼；本教學課程將說明如何進行設定，包括 Bot 應用程式。

- [建立 Bot 和驗證應用程式](#create-your-bot-and-an-authentication-application)
- [準備 Bot 範例程式碼](#prepare-the-bot-sample-code)
- [使用模擬器測試您的 Bot](#use-the-emulator-to-test-your-bot)

若要完成這些步驟，您必須先行安裝 Visual Studio 2017、npm、節點和 git。 此外，您也應熟悉 Azure、OAuth 2.0 及 Bot 開發流程。

完成後，您就有一個 Bot 能針對 Azure AD 應用程式回應幾項簡單工作，例如：檢查及傳送電子郵件，或者顯示您或管理員的身分。 您的 Bot 必須針對 Microsoft.Graph 程式庫使用來自 Azure AD 應用程式的權杖，才能達到上述目的。

最後章節會詳細說明部分 Bot 程式碼

- [權杖擷取流程注意事項](#notes-on-the-token-retrieval-flow)

## <a name="create-your-bot-and-an-authentication-application"></a>建立 Bot 和驗證應用程式

您必須建立註冊 Bot 以藉此發佈 Bot 程式碼，以及建立 Azure AD (v1 或 v2) 應用程式以允許 Bot 存取 Office 365。

> [!NOTE]
> 這些驗證功能皆可搭配其他類型的 Bot 使用。 不過，本教學課程所用的範例是僅具有註冊功能的 Bot。

### <a name="register-an-application-in-azure-ad"></a>在 Azure AD 中註冊應用程式

您需要一個可讓 Bot 用來連線至 Microsoft Graph API 的 Azure AD 應用程式，以及專屬的 Azure AD 保護資源等等。

針對此 Bot，您可使用 Azure AD v1 或 v2 端點。
如需 v1 和 v2 端點之間差異的相關資訊，請參閱 [v1 與 v2 比較](https://docs.microsoft.com/azure/active-directory/develop/active-directory-v2-compare)，以及 [Azure AD v2.0 端點概觀](https://docs.microsoft.com/azure/active-directory/develop/active-directory-appmodel-v2-overview)。

#### <a name="to-create-an-azure-ad-v1-application"></a>建立 Azure AD v1 應用程式

1. 移至 [Azure 入口網站的 Azure AD](https://portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/Overview)。
1. 按一下 [應用程式註冊]。
1. 在 [應用程式註冊] 面板中，按一下 [新增應用程式註冊]。
1. 填寫必要欄位並建立應用程式註冊。
   1. 為您的應用程式命名。
   1. 將 [應用程式類型] 設為 [Web 應用程式/API]。
   1. 將 [登入 URL] 設為 `https://token.botframework.com/.auth/web/redirect`。
   1. 按一下頁面底部的 [新增] 。
      - 建立後，即會顯示在 [註冊的應用程式] 窗格中。
      - 記錄 [應用程式識別碼] 值。 您稍後需要提供此值做為_用戶端識別碼_。
1. 按一下 [設定] 以設定應用程式。
1. 按一下 [金鑰] 以開啟 [金鑰] 面板。
   1. 在 [密碼] 下方建立 `BotLogin` 金鑰。
   1. 將其 [持續時間] 設為 [永不過期]。
   1. 按一下 [儲存] 並記錄金鑰值。 您稍後需要提供此值做為_應用程式密碼_。
   1. 關閉 [金鑰] 面板。
1. 按一下 [必要權限] 以開啟 [必要權限] 面板。
   1. 按一下 [新增] 。
   1. 按一下 [選取 API]，然後選取 [Microsoft Graph]，再按一下 [選取]。
   1. 按一下 [選取權限]。 選擇您要讓應用程式使用的應用程式權限。

      > [!NOTE]
      > 只要是標記為 [需要系統管理員] 的權限，則使用者和租用戶管理員皆必須登入，才能將 Bot 隔離在這些權限之外。

      選取下列 Microsoft Graph 委派的權限：
      - 讀取所有使用者的基本個人資料
      - 讀取使用者的郵件
      - 登入及讀取使用者設定檔
      - 以使用者身分傳送郵件
      - 檢視所有使用者的基本個人資料
      - 檢視使用者的電子郵件地址

   1. 按一下 [選取]，然後按一下 [完成]。
   1. 關閉 [必要權限] 面板。

您現在已完成 Azure AD v1 應用程式設定。

#### <a name="to-create-an-azure-ad-v2-application"></a>建立 Azure AD v2 應用程式

1. 移至 [Microsoft 應用程式註冊入口網站](https://apps.dev.microsoft.com)。
1. 按一下 [新增應用程式]
1. 為您的 Azure AD 應用程式輸入名稱，然後按一下 [建立]。

    記錄**應用程式識別碼**的 GUID。 您稍後需要提供此 GUID 做為連線設定的用戶端識別碼。

1. 在 [應用程式密碼] 下，按一下 [產生新密碼]。

    記錄快顯視窗中的密碼。 您稍後需要提供此密碼做為連線設定的用戶端密碼。

1. 按一下 [平台] 下方的 [新增平台]。
1. 在 [新增平台] 快顯視窗中，按一下 [Web]。
    1. 將 [允許隱含流程] 保持勾選。
    1. 在 [重新導向 URL] 中輸入 `https://token.botframework.com/.auth/web/redirect`。
    1. 將 [登出 URL] 欄位保留空白。
1. 在 [Microsoft Graph 權限] 下方，您可新增其他委派的權限。
    - 在本教學課程中，新增的是 **Mail.Read**、**Mail.Send**、**openid**、**profile**、**User.Read** 和 **User.ReadBasic.All** 權限。
      連線設定的範圍需要使用 **openid** 和 Azure AD Graph 中的資源，例如：**Mail.Read**。
    - 記錄您選擇的權限。 您稍後需要提供此權限做為連線設定的範圍。

1. 按一下頁面底部的 [儲存]  。

### <a name="create-your-bot-on-azure"></a>在 Azure 建立 Bot

使用 [Azure 入口網站](https://portal.azure.com/)建立 **Bot 通道註冊**。

### <a name="register-your-azure-ad-application-with-your-bot"></a>向 Bot 註冊 Azure AD 應用程式

下一步是向 Bot 註冊您剛剛建立的 Azure AD 應用程式。

#### <a name="to-register-an-azure-ad-v1-application"></a>註冊 Azure AD v1 應用程式

1. 在 [Azure 入口網站](http://portal.azure.com/)瀏覽至 Bot 的資源頁面。
1. 按一下 [設定] 。
1. 在靠近頁面底部的 [OAuth 連線設定] 下方，按一下 [新增設定]。
1. 填寫表單，如下所示：
    1. 若為 [名稱] 欄位，請輸入連線的名稱。 Bot 程式碼中會用到此名稱。
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

您現可在 Bot 程式碼中使用此連線名稱，以擷取使用者權杖。

#### <a name="to-register-an-azure-ad-v2-application"></a>註冊 Azure AD v2 應用程式

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

您現可在 Bot 程式碼中使用此連線名稱，以擷取使用者權杖。

#### <a name="to-test-your-connection"></a>測試連線

1. 開啟您剛建立的連線。
1. 按一下 [服務提供者連線設定] 窗格頂端的 [測試連線]。
1. 第一次將開啟新的瀏覽器索引標籤，其中列出應用程式要求的權限，並提示您接受要求。
1. 按一下 [接受]。
1. 此動作會將您重新導向 [測試對 `<your-connection-name>' 的連線已成功] 頁面。

## <a name="prepare-the-bot-sample-code"></a>準備 Bot 範例程式碼

1. 複製 https://github.com/Microsoft/BotBuilder 的 github 存放庫。
1. 開啟並建置解決方案：`BotBuilder\CSharp\Microsoft.Bot.Builder.sln`。
1. 關閉該解決方案，然後開啟：`BotBuilder\CSharp\Samples\Microsoft.Bot.Builder.Samples.sln`。
1. 設定啟動專案。
    - 若為採用 v1 Azure AD 應用程式的 Bot，請使用 `Microsoft.Bot.Sample.AadV1Bot` 專案。
    - 若為採用 v2 Azure AD 應用程式的 Bot，請使用 `Microsoft.Bot.Sample.AadV2Bot` 專案。
1. 開啟 `Web.config` 檔案，並按照以下指示修改應用程式設定：
    1. 將 `ConnectionName` 設為您在配置 Bot 的 OAuth 2.0 連線設定時所用的值。
    1. 將 `MicrosoftAppId` 值設為 Bot 的應用程式識別碼。
    1. 將 `MicosoftAppPassword` 值設為 Bot 的密碼。

    > [!IMPORTANT]
    > 視密碼中的字元而定，您可能需要讓 XML 逸出該密碼。 例如，任何 & 符號都必須編碼為 `&amp;`。

    ```xml
    <appSettings>
        <add key="ConnectionName" value="<your-AAD-connection-name>"/>
        <add key="MicrosoftAppId" value="<your-bot-appId>" />
        <add key="MicrosoftAppPassword" value="<your-bot-password>" />
    </appSettings>
    ```

    如果您不知道如何取得 **Microsoft 應用程式識別碼**和 **Microsoft 應用程式密碼**值，請找到您在 Azure 入口網站上為 Bot 佈建的 Azure 應用程式服務，然後查看其中的 **ApplicationSettings**。

    > [!NOTE]
    > 您現在可將此 Bot 程式碼重新發佈至 Azure 訂用帳戶 (以滑鼠右鍵按一下專案，然後選擇 [發佈])，但此動作在本教學課程的範例中並非必要。 在 Azure 入口網站中配置 Bot 時，您必須進行發佈設定，其應使用您所用的應用程式和主控方案。

## <a name="use-the-emulator-to-test-your-bot"></a>使用模擬器測試您的 Bot

您必須安裝 [Bot 模擬器](https://github.com/Microsoft/BotFramework-Emulator)以便在本機測試 Bot。 您可使用 v3 或 v4 模擬器。

1. 啟動 Bot (無論是否啟用偵錯)。
1. 請記下該頁面的 localhost 連接埠號碼。 稍後與 Bot 互動時，您會需要使用此資訊。
1. 啟動模擬器。
1. 連線至 Bot。

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

使用者要求 Bot 執行需要使用者登入的工作時，Bot 可使用 `Microsoft.Bot.Builder.Dialogs.GetTokenDialog` 起始擷取特定連線的權杖。 接下來幾個程式碼片段是取自 `GetTokenDialog` 類別。

### <a name="check-for-a-cached-token"></a>檢查已快取的權杖

在此程式碼中，首先 Bot 會快速檢查以判斷 Azure Bot 服務是否已有該使用者的權杖 (藉由目前的活動傳送者識別) 和指定的 ConnectionName (即設定中使用的連線名稱)。 Azure Bot 服務可能會快取權杖，也可能不會。 針對 GetUserTokenAsync 的呼叫會執行此「快速檢查」。 如果 Azure Bot 服務具有權杖並將其傳回，則該權杖可立即使用。 如果 Azure Bot 服務沒有權杖，此方法將傳回 Null。 在此情況下，Bot 可傳送自訂的 OAuthCard 供使用者登入。

```csharp
// First ask Bot Service if it already has a token for this user
var token = await context.GetUserTokenAsync(ConnectionName).ConfigureAwait(false);
if (token != null)
{
    // use the token to do exciting things!
}
else
{
    // If Bot Service does not have a token, send an OAuth card to sign in
    await SendOAuthCardAsync(context, (Activity)context.Activity);
}
```

### <a name="send-an-oauthcard-to-the-user"></a>傳送 OAuthCard 給使用者

您可將 OAuthCard 自訂為任何您想用的文字和按鈕文字。 需要注意的重點如下：

- 將 `ContentType` 設為 `OAuthCard.ContentType`。
- 將 `ConnectionName` 屬性設為您要使用之連線的名稱。
- 包含一個具有 `Type` `ActionTypes.Signin` 之 `CardAction` 的按鈕；請注意，您不需要為登入連結指定任何值。

在此呼叫結束時，Bot 必須「等候權杖」回傳。 由於可能有許多使用者需要登入，因此等候過程會在主要的活動流中發生。

```csharp
private async Task SendOAuthCardAsync(IDialogContext context, Activity activity)
{
    await context.PostAsync($"To do this, you'll first need to sign in.");

    var reply = await context.Activity.CreateOAuthReplyAsync(_connectionName, _signInMessage, _buttonLabel).ConfigureAwait(false);
    await context.PostAsync(reply);

    context.Wait(WaitForToken);
}
```

### <a name="wait-for-a-tokenresponseevent"></a>等候 TokenResponseEvent

在此程式碼中，Bot 的對話方塊類別正在等候 `TokenResponseEvent` (下文將詳述其如何路由至對話方塊堆疊)。 `WaitForToken` 方法會先判斷此事件是否已傳送。 如果其已傳送，則可供 Bot 使用。 如果未傳送，則 `WaitForToken` 方法將擷取任何傳送給 Bot 的文字，再將其傳遞至 `GetUserTokenAsync`。 此作法的原因是部分用戶端 (例如：WebChat) 不需要進行神奇代碼驗證碼，而可直接在 `TokenResponseEvent` 中傳送權杖。 其他用戶端 (例如：Facebook 或 Slack) 仍須使用神奇代碼。 Azure Bot 服務會向用戶端顯示一組六位數的神奇代碼，並要求使用者在聊天視窗中輸入該代碼。 此方式並非最理想的「回復」行為，因此，假如 `WaitForToken` 收到代碼，則 Bot 可將代碼傳送至 Azure Bot Service 並獲取權杖。 若此呼叫也失敗，則您可決定是否要回報錯誤或採取其他行動。 不過大部分情況下，Bot 現在都會有使用者權杖。

若您檢視 **MessageController.cs** 檔案，即可發現此類型的 `Event` 活動也會路由至對話方塊堆疊。

```csharp
private async Task WaitForToken(IDialogContext context, IAwaitable<object> result)
{
    var activity = await result as Activity;

    var tokenResponse = activity.ReadTokenResponseContent();
    if (tokenResponse != null)
    {
        // Use the token to do exciting things!
    }
    else
    {
        if (!string.IsNullOrEmpty(activity.Text))
        {
            tokenResponse = await context.GetUserTokenAsync(ConnectionName,
                                                               activity.Text);
            if (tokenResponse != null)
            {
                // Use the token to do exciting things!
                return;
            }
        }
        await context.PostAsync($"Hmm. Something went wrong. Let's try again.");
        await SendOAuthCardAsync(context, activity);
    }
}
```

### <a name="message-controller"></a>訊息控制器

在對 Bot 的後續呼叫中，請注意此範例 Bot 永遠不會快取權杖。 這是因為 Bot 可一律向 Azure Bot 服務要求權杖。 當 Azure Bot 服務為您執行一切工作時，如此可避免 Bot 必須管理權杖生命週期、更新權杖等繁瑣作業。

```csharp
else if(message.Type == ActivityTypes.Event)
{
    if(message.IsTokenResponseEvent())
    {
        await Conversation.SendAsync(message, () => new Dialogs.RootDialog());
    }
}
```
## <a name="additional-resources"></a>其他資源
[Bot Framework SDK](https://github.com/microsoft/botbuilder)
