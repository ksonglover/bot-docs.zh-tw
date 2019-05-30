---
title: 透過 Azure Bot 服務將驗證新增至您的 Bot | Microsoft Docs
description: 了解如何使用 Azure Bot 服務驗證功能以便將 SSO 新增至您的 Bot。
author: JonathanFingold
ms.author: v-jofing
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: abs
ms.date: 05/23/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: f41409b534b997d486a94dc206ecc85bf6f0ca9e
ms.sourcegitcommit: ea64a56acfabc6a9c1576ebf9f17ac81e7e2a6b7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/24/2019
ms.locfileid: "66215557"
---
<!-- Related TODO:
- Check code in [Web Chat channel](https://docs.microsoft.com/en-us/azure/bot-service/bot-service-channel-connect-webchat?view=azure-bot-service-4.0)
- Check guidance in [DirectLine authentication](https://docs.microsoft.com/en-us/azure/bot-service/rest-api/bot-framework-rest-direct-line-3-0-authentication?view=azure-bot-service-4.0)
-->

<!-- General TODO: (Feedback from CSE (Nafis))
- Add note that: token management is based on user ID
- Explain why/how to share existing website authentication with a bot.
- Risk: Even people who use a DirectLine token can be vulnerable to user ID impersonation.
    Docs/samples that show exchange of secret for token don't specify a user ID, so an attacker can impersonate a different user by modifying the ID client side. There's a [blog post](https://nam06.safelinks.protection.outlook.com/?url=https%3A%2F%2Fblog.botframework.com%2F2018%2F09%2F01%2Fusing-webchat-with-azure-bot-services-authentication%2F&data=02%7C01%7Cv-jofing%40microsoft.com%7Cee005e1c9d2c4f4e7ea508d6b231b422%7C72f988bf86f141af91ab2d7cd011db47%7C1%7C0%7C636892323874079713&sdata=b0DWMxHzmwQvg5EJtlqKFDzR7fYKmg10fXma%2B8zGqEI%3D&reserved=0) that shows how to do this properly.
"Major issues":
- This doc is a sample walkthrough, but there's no deeper documentation explaining how the Azure Bot Service is handling tokens. How does the OAuth flow work? Where is it storing my users' access tokens? What's the security and best practices around using it?

"Minor issues":
- AAD v2 steps tell you to add delegated permission scopes during registration, but this shouldn't be necessary in AAD v2 due to dynamic scopes. (Ming, "This is currently necessary because scopes are not exposed through our runtime API. We don’t currently have a way for the developer to specify which scope he wants at runtime.")

- "The scope of the connection setting needs to have both openid and a resource in the Azure AD graph, such as Mail.Read." Unclear if I need to take some action at this point to make happen. Kind of out of context. I'm registering an AAD application in the portal, there's no connection setting
- Does the bot need all of these scopes for the samples? (e.g. "Read all users' basic profiles")
-->

# <a name="add-authentication-to-your-bot-via-azure-bot-service"></a>透過 Azure Bot 服務將驗證新增至您的 Bot

[!INCLUDE [applies-to-v4](../includes/applies-to.md)]

Azure Bot 服務和 v4 SDK 包含全新的 Bot 驗證功能，並提供相關功能，讓您輕鬆開發可對 Azure AD (Azure Active Directory)、GitHub、Uber 等不同識別提供者驗證使用者的 Bot。 這些功能也讓部分用戶端無須進行_神奇代碼驗證 (Magic code verification)_ ，而改善使用者體驗。

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

## <a name="prerequisites"></a>必要條件

- [Bot 基本概念][concept-basics]、[管理狀態][concept-state]、[dialogs 程式庫][concept-dialogs]、如何[實作循序交談流程][simple-dialog]、如何[使用對話提示蒐集使用者輸入][dialog-prompts]，以及如何[重複使用對話][component-dialogs]的知識。
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

# <a name="azure-ad-v1tabaadv1"></a>[Azure AD v1](#tab/aadv1)

1. 在 Azure 入口網站中開啟 [Azure Active Directory][azure-aad-blade] 面板。
    如果您不在正確的租用戶中，請按一下 [切換目錄]  以切換至正確的租用戶。 (如需建立租用戶的指示，請參閱[存取入口網站並建立租用戶](https://docs.microsoft.com/en-us/azure/active-directory/fundamentals/active-directory-access-create-new-tenant)。)
1. 開啟 [應用程式註冊]  面板。
1. 在 [應用程式註冊]  面板中，按一下 [新增應用程式註冊]  。
1. 填寫必要欄位並建立應用程式註冊。

   1. 為您的應用程式命名。
   1. 將 [應用程式類型]  設為 [Web 應用程式/API]  。
   1. 將 [登入 URL]  設為 `https://token.botframework.com/.auth/web/redirect`。
   1. 按一下頁面底部的 [新增]  。

      - 建立後，即會顯示在 [註冊的應用程式]  窗格中。
      - 記錄 [應用程式識別碼]  值。 稍後當您向 Bot 註冊 Azure AD 應用程式時，您將使用此值作為_用戶端識別碼_。

1. 按一下 [設定]  以設定應用程式。
1. 按一下 [金鑰]  以開啟 [金鑰]  面板。

   1. 在 [密碼]  下方建立 `BotLogin` 金鑰。
   1. 將其 [持續時間]  設為 [永不過期]  。
   1. 按一下 [儲存]  並記錄金鑰值。 稍後當您向 Bot 註冊 Azure AD 應用程式時，您將使用此值作為_用戶端密碼_。
   1. 關閉 [金鑰]  面板。

1. 按一下 [必要權限]  以開啟 [必要權限]  面板。

   1. 按一下 [新增]  。
   1. 按一下 [選取 API]  ，然後選取 [Microsoft Graph]  ，再按一下 [選取]  。
   1. 按一下 [選取權限]  。 選擇您的應用程式將使用的委派權限。

      > [!NOTE]
      > 只要是標記為 [需要系統管理員]  的權限，則使用者和租用戶管理員皆必須登入，才能將 Bot 隔離在這些權限之外。

      選取下列 Microsoft Graph 委派的權限：
      - 讀取所有使用者的基本個人資料
      - 讀取使用者的郵件
      - 以使用者身分傳送郵件
      - 登入及讀取使用者設定檔
      - 檢視所有使用者的基本個人資料
      - 檢視使用者的電子郵件地址

   1. 按一下 [選取]  ，然後按一下 [完成]  。
   1. 關閉 [必要權限]  面板。

您現在已完成 Azure AD v1 應用程式設定。

# <a name="azure-ad-v2tabaadv2"></a>[Azure AD v2](#tab/aadv2)

1. 移至 [Microsoft 應用程式註冊入口網站](https://portal.azure.com/#blade/Microsoft_AAD_RegisteredApps/ApplicationsListBlade)。
1. 按一下 [新增應用程式] 
1. 為您的 Azure AD 應用程式輸入名稱，然後按一下 [建立]  。

    記錄**應用程式識別碼**的 GUID。 您稍後需要提供此 GUID 做為連線設定的用戶端識別碼。

1. 在 [應用程式密碼]  下，按一下 [產生新密碼]  。

    記錄快顯視窗中的密碼。 您稍後需要提供此密碼做為連線設定的用戶端密碼。

1. 按一下 [平台]  下方的 [新增平台]  。
1. 在 [新增平台]  快顯視窗中，按一下 [Web]  。

    1. 將 [允許隱含流程]  保持勾選。
    1. 在 [重新導向 URL]  中輸入 `https://token.botframework.com/.auth/web/redirect`。
    1. 將 [登出 URL]  欄位保留空白。

1. 在 [Microsoft Graph 權限]  下方，您可新增其他委派的權限。

    - 在本教學課程中，新增 **Mail.Read**、**Mail.Send**、**openid**、**profile**、**User.Read** 和 **User.ReadBasic.All** 權限。
      連線設定的範圍需要使用 **openid** 和 Azure AD Graph 中的資源，例如：**Mail.Read**。
    - 記錄您選擇的權限。 您稍後需要提供此權限做為連線設定的範圍。

1. 按一下頁面底部的 [儲存]  。

您現在已完成 Azure AD v2 應用程式設定。

---

### <a name="register-your-azure-ad-application-with-your-bot"></a>向 Bot 註冊 Azure AD 應用程式

下一步是向 Bot 註冊您已建立的 Azure AD 應用程式。

# <a name="azure-ad-v1tabaadv1"></a>[Azure AD v1](#tab/aadv1)

1. 在 [Azure 入口網站](http://portal.azure.com/)瀏覽至 Bot 的資源頁面。
1. 按一下 [設定]  。
1. 在靠近頁面底部的 [OAuth 連線設定]  下方，按一下 [新增設定]  。
1. 填寫表單，如下所示：

    1. 若為 [名稱]  欄位，請輸入連線的名稱。 您將在 Bot 程式碼中使用此名稱。
    1. 若為 [服務提供者]  ，請選取 [Azure Active Directory]  。 選取此項目後，Azure AD 專用的欄位隨即顯示。
    1. 若為 [用戶端識別碼]  欄位，請輸入您為 Azure AD v1 應用程式記錄的應用程式識別碼。
    1. 若為 [用戶端密碼]  欄位，請輸入您為應用程式 `BotLogin` 金鑰記錄的金鑰。
    1. 若為 [授與類型]  欄位，請輸入「`authorization_code`」。
    1. 若為 [登入 URL]  欄位，請輸入「`https://login.microsoftonline.com`」。
    1. 若為 [租用戶識別碼]  欄位，請輸入 Azure Active Directory 的租用戶識別碼，例如：`microsoft.com` 或 `common`。

       這將會是與可驗證之使用者相關聯的租用戶。 若要允許任何人透過 Bot 驗證身分，請使用 `common` 租用戶。

    1. 針對 [資源 URL]  ，輸入 `https://graph.microsoft.com/`。
    1. 請將 [範圍]  欄位保留空白。

1. 按一下 [檔案]  。

> [!NOTE]
> 這些值可讓應用程式透過 Microsoft Graph API 存取 Office 365 資料。

# <a name="azure-ad-v2tabaadv2"></a>[Azure AD v2](#tab/aadv2)

1. 在 [Azure 入口網站](http://portal.azure.com/)上瀏覽至 Bot 的 [Bot 通道註冊] 頁面。
1. 按一下 [設定]  。
1. 在靠近頁面底部的 [OAuth 連線設定]  下方，按一下 [新增設定]  。
1. 填寫表單，如下所示：

    1. 若為 [名稱]  欄位，請輸入連線的名稱。 Bot 程式碼中會用到此名稱。
    1. 若為 [服務提供者]  ，請選取 [Azure Active Directory v2]  。 選取此項目後，Azure AD 專用的欄位隨即顯示。
    1. 若為 [用戶端識別碼]  欄位，請輸入應用程式註冊時的 Azure AD v2 應用程式識別碼。
    1. 若為 [用戶端密碼]  欄位，請輸入應用程式註冊時的 Azure AD v2 應用程式密碼。
    1. 若為 [租用戶識別碼]  欄位，請輸入 Azure Active Directory 的租用戶識別碼，例如：`microsoft.com` 或 `common`。

        這將會是與可驗證之使用者相關聯的租用戶。 若要允許任何人透過 Bot 驗證身分，請使用 `common` 租用戶。

    1. 若為 [範圍]  欄位，請輸入應用程式註冊時您選擇之權限的名稱：`Mail.Read Mail.Send openid profile User.Read User.ReadBasic.All`。

        > [!NOTE]
        > 若為 Azure AD v2，[範圍]  欄位接受區分大小寫並以空格區隔值的清單。

1. 按一下 [檔案]  。

> [!NOTE]
> 這些值可讓應用程式透過 Microsoft Graph API 存取 Office 365 資料。

---

### <a name="test-your-connection"></a>測試連線

1. 按一下連線項目以開啟您剛剛建立的連線。
1. 按一下 [服務提供者連線設定]  窗格頂端的 [測試連線]  。
1. 第一次將開啟新的瀏覽器索引標籤，其中列出應用程式要求的權限，並提示您接受要求。
1. 按一下 [接受]  。
1. 這應會將您重新導向至 [測試對 \<your-connection-name> 的連線已成功]  頁面。

您現可在 Bot 程式碼中使用此連線名稱，以擷取使用者權杖。

## <a name="prepare-the-bot-code"></a>準備 Bot 程式碼

您需要 Bot 的應用程式識別碼和密碼來完成此程序。 若要擷取您的 Bot 應用程式識別碼和密碼：

1. 在 [Azure 入口網站][]中，巡覽至您在其中建立通道註冊 Bot 的資源群組。
1. 開啟 [部署]  窗格，然後開啟 Bot 的部署。
1. 開啟 [輸入]  窗格，並複製 Bot 的 **appId** 和 **appSecret** 值。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

<!-- TODO: Add guidance (once we have it) on how not to hard-code IDs and ABS auth. -->

1. 從您想要使用的 github 存放庫複製：[**Bot 驗證**][cs-auth-sample]或 [**Bot 驗證 MSGraph**][cs-msgraph-sample]。
1. 遵循 GitHub 讀我檔案頁面上的指示，了解如何執行該特定 Bot。 <!--TODO: Can we remove this step and still have the instructions make sense? What is the minimum we need to say in its place? -->
1. 更新 **appsettings.json**：

    - 將 `ConnectionName` 設定為您新增至 Bot 的 OAuth 連線名稱。
    - 將 `MicrosoftAppId` 和 `MicrosoftAppPassword` 設定為您 Bot 的應用程式識別碼和應用程式祕密。

      視 Bot 祕密中的字元而定，您可能需要讓 XML 逸出該密碼。 例如，任何 & 符號都必須編碼為 `&amp;`。

[!code-json[appsettings](~/../botbuilder-samples/samples/csharp_dotnetcore/18.bot-authentication/appsettings.json)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

1. 從您想要使用的 github 存放庫複製：[**Bot 驗證**][js-auth-sample]或 [**Bot 驗證 MSGraph**][js-msgraph-sample]。
1. 遵循 GitHub 讀我檔案頁面上的指示，了解如何執行該特定 Bot。 <!--TODO: Can we remove this step and still have the instructions make sense? What is the minimum we need to say in its place? -->
1. 更新 **.env**：

    - 將 `connectionName` 設定為您新增至 Bot 的 OAuth 連線名稱。
    - 將 `MicrosoftAppId` 和 `MicrosoftAppPassword` 值設定為您 Bot 的應用程式識別碼和應用程式祕密。

      視 Bot 祕密中的字元而定，您可能需要讓 XML 逸出該密碼。 例如，任何 & 符號都必須編碼為 `&amp;`。

    [!code-txt[.env](~/../botbuilder-samples/samples/javascript_nodejs/18.bot-authentication/.env)]

---

如果您不知道如何取得 **Microsoft 應用程式識別碼**和 **Microsoft 應用程式密碼**值，您可以：

- [如下所述](../bot-service-quickstart-registration.md#bot-channels-registration-password)建立新密碼
- 從[這裡所述](https://blog.botframework.com/2018/07/03/find-your-azure-bots-appid-and-appsecret)的部署，擷取透過 [Bot 通道註冊]  佈建的 [Microsoft 應用程式識別碼]  和 [Microsoft 應用程式密碼] 

> [!NOTE]
> 您現在可將此 Bot 程式碼發佈至 Azure 訂用帳戶 (以滑鼠右鍵按一下專案，然後選擇 [發佈]  )，但此動作在本文的範例中並非必要。 在 Azure 入口網站中配置 Bot 時，您必須進行發佈設定，其應使用您所用的應用程式和主控方案。

## <a name="test-the-bot"></a>測試 Bot

1. 如果您尚未安裝 [Bot Framework Emulator](https://aka.ms/bot-framework-emulator-readme)，請進行安裝。
1. 在您的電腦本機執行範例。
1. 啟動模擬器、連線到您的 Bot，然後傳送訊息。

    - 當您連線到 Bot 時，您必須提供 Bot 的應用程式識別碼和密碼。
    - 輸入「`help`」以檢視適用於 Bot 的可用命令清單，以及測試驗證功能。
    - 登入後一直到登出前，您都不需要再次提供認證。
    - 若要登出並取消驗證，請輸入「`logout`」。

> [!NOTE]
> Bot 驗證需要使用 Bot 連接器服務。 該服務將針對您的 Bot 存取 Bot 通道註冊資訊。

# <a name="bot-authenticationtabbot-oauth"></a>[Bot 驗證](#tab/bot-oauth)

在 **Bot 驗證**範例中，對話的設計訴求是要在使用者登入後擷取使用者權杖。

![範例輸出](media/how-to-auth/auth-bot-test.png)

# <a name="bot-authentication-msgraphtabbot-msgraph-auth"></a>[Bot 驗證 MSGraph](#tab/bot-msgraph-auth)

在 **Bot 驗證 MSGraph**範例中，對話的設計訴求是要接受使用者登入後的一組有限命令。

![範例輸出](media/how-to-auth/msgraph-bot-test.png)

---

## <a name="additional-information"></a>其他資訊

使用者要求 Bot 執行需要使用者登入的工作時，Bot 可使用 `OAuthPrompt` 起始擷取特定連線的權杖。 `OAuthPrompt` 會建立權杖擷取流程，其中包含：

1. 查看 Azure Bot Service 是否已經具有可供目前使用者和連線使用的權杖。 如果有權杖，則會傳回權杖。
1. 如果 Azure Bot Service 沒有快取的權杖，則會建立 `OAuthCard`，這是使用者可以按一下的登入按鈕。
1. 在使用者按一下 `OAuthCard` 登入按鈕之後，Azure Bot Service 會將使用者的權杖傳直接送給 Bot，或者會向使用者顯示要在聊天視窗中輸入的 6 位數驗證碼。
1. 如果使用者看到驗證碼，則 Bot 會將此驗證碼換成使用者的權杖。

下列各節說明範例如何實作一些常見的驗證工作。

### <a name="use-an-oauth-prompt-to-sign-the-user-in-and-get-a-token"></a>使用 OAuth 提示來登入使用者和取得權杖

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

![Bot 架構](media/how-to-auth/architecture.png)

<!-- The two authentication samples have nearly identical architecture. Using 18.bot-authentication for the sample code. -->

**Dialogs\MainDialog.cs**

將 OAuth 提示新增至其建構函式中的 **MainDialog**。 在此，已從 **appsettings.json** 檔案擷取連線名稱值。

[!code-csharp[Add OAuthPrompt](~/../botbuilder-samples/samples/csharp_dotnetcore/18.bot-authentication/Dialogs/MainDialog.cs?range=23-31)]

在對話步驟中，使用 `BeginDialogAsync` 來啟動 OAuth 提示，其會要求使用者登入。

- 如果使用者已經登入，這會產生一個權杖回應事件，但不會提示使用者。
- 否則會提示使用者登入。 Azure Bot Service 會在使用者嘗試登入後傳送權杖回應事件。

[!code-csharp[Use the OAuthPrompt](~/../botbuilder-samples/samples/csharp_dotnetcore/18.bot-authentication/Dialogs/MainDialog.cs?range=49)]

在下列對話步驟中，檢查上一個步驟的結果中是否有權杖存在。 如果不是 Null，則表示使用者登入成功。

[!code-csharp[Get the OAuthPrompt result](~/../botbuilder-samples/samples/csharp_dotnetcore/18.bot-authentication/Dialogs/MainDialog.cs?range=54-58)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

![Bot 架構](media/how-to-auth/architecture-js.png)

**dialogs/mainDialog.js**

將 OAuth 提示新增至其建構函式中的 **MainDialog**。 在此，已從 **.env** 檔案擷取連線名稱值。

[!code-javascript[Add OAuthPrompt](~/../botbuilder-samples/samples/javascript_nodejs/18.bot-authentication/dialogs/mainDialog.js?range=23-28)]

在對話步驟中，使用 `beginDialog` 來啟動 OAuth 提示，其會要求使用者登入。

- 如果使用者已經登入，這會產生一個權杖回應事件，但不會提示使用者。
- 否則會提示使用者登入。 Azure Bot Service 會在使用者嘗試登入後傳送權杖回應事件。

[!code-javascript[Use OAuthPrompt](~/../botbuilder-samples/samples/javascript_nodejs/18.bot-authentication/dialogs/mainDialog.js?range=57)]

在下列對話步驟中，檢查上一個步驟的結果中是否有權杖存在。 如果不是 Null，則表示使用者登入成功。

[!code-javascript[Get OAuthPrompt result](~/../botbuilder-samples/samples/javascript_nodejs/18.bot-authentication/dialogs/mainDialog.js?range=61-64)]

---

### <a name="wait-for-a-tokenresponseevent"></a>等候 TokenResponseEvent

當您啟動 OAuth 提示時，其會等候權杖回應事件，而且會從中擷取使用者的權杖。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

**Bots\AuthBot.cs**

**AuthBot** 衍生自 `ActivityHandler` 並明確處理權杖回應事件活動。 在此，我們會持續進行作用中的對話，讓 OAuth 提示得以處理事件和擷取權杖。

[!code-csharp[OnTokenResponseEventAsync](~/../botbuilder-samples/samples/csharp_dotnetcore/18.bot-authentication/Bots/AuthBot.cs?range=32-38)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**bots/authBot.js**

**AuthBot** 衍生自 `ActivityHandler` 並明確處理權杖回應事件活動。 在此，我們會持續進行作用中的對話，讓 OAuth 提示得以處理事件和擷取權杖。

[!code-javascript[onTokenResponseEvent](~/../botbuilder-samples/samples/javascript_nodejs/18.bot-authentication/bots/authBot.js?range=28-33)]

---

### <a name="log-the-user-out"></a>登出使用者

最佳做法是讓使用者明確登出，而不是依賴連接逾時。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

**Dialogs\LogoutDialog.cs**

[!code-csharp[Allow logout](~/../botbuilder-samples/samples/csharp_dotnetcore/18.bot-authentication/Dialogs/LogoutDialog.cs?range=20-61&highlight=35)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**dialogs/logoutDialog.js**

[!code-javascript[Allow logout](~/../botbuilder-samples/samples/javascript_nodejs/18.bot-authentication/dialogs/logoutDialog.js?range=13-42&highlight=25)]

---

### <a name="further-reading"></a>進階閱讀

- [Bot Framework 其他資源](https://docs.microsoft.com/azure/bot-service/bot-service-resources-links-help)包含其他支援的連結。
- [Bot Framework SDK](https://github.com/microsoft/botbuilder) 存放庫提供更多與 Bot Builder SDK 相關聯的存放庫、範例、工具和規格的詳細資訊。

<!-- Footnote-style links -->

[Azure 入口網站]: https://ms.portal.azure.com
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
