---
title: 驗證 | Microsoft Docs
description: 了解如何在 Direct Line API v3.0 中驗證 API 要求。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 08/22/2019
ms.openlocfilehash: d79cea421e6743c504e3fa68056de71974194923
ms.sourcegitcommit: c200cc2db62dbb46c2a089fb76017cc55bdf26b0
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2019
ms.locfileid: "70037439"
---
# <a name="authentication"></a>Authentication

用戶端可以藉由在 Bot Framework 入口網站使用您[從 Direct Line 頻道設定頁面取得](../bot-service-channel-connect-directline.md)的**祕密**來驗證 Direct Line API 3.0 的要求，或使用您在執行階段取得的**權杖**來驗證。 祕密或權杖應該使用下列格式，指定於每個要求的 `Authorization` 標頭中： 

```http
Authorization: Bearer SECRET_OR_TOKEN
```

## <a name="secrets-and-tokens"></a>祕密和權杖

Direct Line **祕密**是一個主要金鑰，可用來存取屬於相關聯 Bot 的任何對話。 **祕密**也可用來取得**權杖**。 祕密不會過期。 

Direct Line **權杖**是一個金鑰，可用來存取單一對話。 權杖會過期，但可以重新整理。 

如果您要建立服務對服務的應用程式，則在 Direct Line API 要求的 `Authorization` 標頭中指定**祕密**可能是最簡單的方法。 如果您要撰寫用戶端在網頁瀏覽器或行動裝置應用程式中執行的應用程式，您可以將交換祕密以取得權杖 (這只適用於單一對話，而且除非重新整理，否則將會過期)，並在 Direct Line API 要求的 `Authorization` 標頭中指定**權杖**。 選擇最適合您的安全性模型。

> [!NOTE]
> 您的 Direct Line 用戶端認證與您的 Bot 認證不同。 這可讓您獨立修訂金鑰，並讓您共用用戶端權杖，而不會洩漏 Bot 的密碼。 

## <a name="get-a-direct-line-secret"></a>取得 Direct Line 祕密

您可以在 <a href="https://dev.botframework.com/" target="_blank">Bot Framework 入口網站</a>中，透過 Bot 的 Direct Line 頻道設定頁面來[取得 Direct Line 祕密](../bot-service-channel-connect-directline.md)：

![Direct Line 設定](../media/direct-line-configure.png)

## <a id="generate-token"></a> 產生 Direct Line 權杖

若要產生可用來存取單一對話的 Direct Line 權杖，請先從 <a href="https://dev.botframework.com/" target="_blank">Bot Framework 入口網站</a>的 Direct Line 頻道設定頁面取得 Direct Line 祕密。 然後發出此要求，將您的 Direct Line 祕密交換以取得 Direct Line 權杖：

```http
POST https://directline.botframework.com/v3/directline/tokens/generate
Authorization: Bearer SECRET
```

在此要求的 `Authorization` 標頭中，使用您的 Direct Line 祕密值來取代 **SECRET**。

下列程式碼片段提供產生權杖要求和回應的範例。

### <a name="request"></a>要求

```http
POST https://directline.botframework.com/v3/directline/tokens/generate
Authorization: Bearer RCurR_XV9ZA.cwA.BKA.iaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0
```

要求承載 (其中包含權杖參數) 是選用，但建議使用的項目。 在產生可送回 Direct Line 服務的權杖時，請提供下列承載，讓連線更加安全。 藉由包含這些值，Direct Line 可對使用者識別碼和名稱執行額外的安全性驗證，以抑制惡意用戶端竄改這些值。 包含這些值也會改善 Direct Line 傳送_交談更新_活動的能力，可讓其在使用者加入交談時立即產生交談更新。 若未提供此資訊，使用者必須先傳送內容，Direct Line 才能傳送交談更新。

```json
{
  "user": {
    "id": "string",
    "name": "string"
  },
  "trustedOrigins": [
    "string"
  ]
}
```

| 參數 | 類型 | 說明 |
| :--- | :--- | :--- |
| `user.id` | 字串 | 選用。 權杖內要編碼的使用者通道專屬識別碼。 若為 Direct Line 使用者，這必須以 `dl_` 開頭。 您可以針對每次交談建立唯一的使用者識別碼，且基於安全考量，您應該讓此識別碼無法猜測。 |
| `user.name` | 字串 | 選用。 權杖內要編碼的使用者易記顯示名稱。 |
| `trustedOrigins` | 字串陣列 | 選用。 權杖內要內嵌的受信任網域清單。 這些是可以裝載 Bot 網路聊天用戶端的網域。 這應該符合 Direct Line 設定頁面中您 Bot 的清單。 |

### <a name="response"></a>Response

如果要求成功，回應就會包含對某個對話有效的 `token`，以及表示直到權杖過期前秒數的 `expires_in` 值。 若要讓權杖仍然可用，您必須在它過期之前[重新整理權杖](#refresh-token)。

```http
HTTP/1.1 200 OK
[other headers]
```

```json
{
  "conversationId": "abc123",
  "token": "RCurR_XV9ZA.cwA.BKA.iaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0y8qbOF5xPGfiCpg4Fv0y8qqbOF5x8qbOF5xn",
  "expires_in": 1800
}
```

### <a name="generate-token-versus-start-conversation"></a>產生權杖與開始對話

產生權杖作業 (`POST /v3/directline/tokens/generate`) 類似[開始對話](bot-framework-rest-direct-line-3-0-start-conversation.md)作業 (`POST /v3/directline/conversations`)，這兩個作業都會傳回可用於存取單一對話的 `token`。 不過，不同於開始對話作業，產生權杖作業不會開始對話、不會連絡 Bot，也不會建立串流處理的 WebSocket URL。 

如果您計畫要將權杖散發給用戶端，並且想要讓它們起始對話，請使用產生權杖作業。 如果您想要立即開始對話，請改用[開始對話](bot-framework-rest-direct-line-3-0-start-conversation.md)作業。

## <a id="refresh-token"></a> 重新整理 Direct Line 權杖

Direct Line 權杖只要尚未過期，就可以重新整理且不限次數。 過期的權杖無法重新整理。 若要重新整理 Direct Line 權杖，請發出此要求： 

```http
POST https://directline.botframework.com/v3/directline/tokens/refresh
Authorization: Bearer TOKEN_TO_BE_REFRESHED
```

在此要求的 `Authorization` 標頭中，使用您想要重新整理的 Direct Line 權杖來取代 **TOKEN_TO_BE_REFRESHED**。

下列程式碼片段提供重新整理權杖要求和回應的範例。

### <a name="request"></a>要求

```http
POST https://directline.botframework.com/v3/directline/tokens/refresh
Authorization: Bearer CurR_XV9ZA.cwA.BKA.iaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0y8qbOF5xPGfiCpg4Fv0y8qqbOF5x8qbOF5xn
```

### <a name="response"></a>Response

如果要求成功，回應就會包含對與前一個權杖相同之對話有效的新 `token`，以及表示直到新權杖過期前秒數的 `expires_in` 值。 若要讓新權杖仍然可用，您必須在它過期之前[重新整理權杖](#refresh-token)。

```http
HTTP/1.1 200 OK
[other headers]
```

```json
{
  "conversationId": "abc123",
  "token": "RCurR_XV9ZA.cwA.BKA.y8qbOF5xPGfiCpg4Fv0y8qqbOF5x8qbOF5xniaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0",
  "expires_in": 1800
}
```

## <a name="azure-bot-service-authentication"></a>Azure Bot 服務驗證

本節中所提供的資訊以[透過 Azure Bot 服務將驗證新增至您的 Bot](../v4sdk/bot-builder-authentication.md) 一文為基礎。

**Azure Bot 服務驗證**可讓您驗證使用者，並取得各種識別提供者 (例如 *Azure Active Directory*、*GitHub*、*Uber* 等等) 所提供的**存取權杖**。 您也可以設定自訂 **OAuth2** 識別提供者的驗證。 這種功能，讓您能夠撰寫可在所有支援的識別提供者和通道上運作的**一個驗證程式碼**。 若要使用這些功能，您必須執行下列步驟：

1. 在您的 Bot 上以靜態方式設定 `settings`，其中包含您向識別提供者註冊應用程式的詳細資料。
2. 使用您在先前的步驟中提供的應用程式資訊所支援的 `OAuthCard` 登入使用者。
3. 透過 **Azure Bot 服務 API** 擷取存取權杖。

### <a name="security-considerations"></a>安全性考量

<!-- Summarized from: https://blog.botframework.com/2018/09/25/enhanced-direct-line-authentication-features/ -->

當您搭配使用 *Azure Bot 服務驗證*與[網路聊天](../bot-service-channel-connect-webchat.md)時，必須留意某些重要的安全性考量。

1. **模擬**。 此處的模擬是指攻擊者設法讓 Bot 將其誤認為他人。 在網路聊天中，攻擊者可藉由**變更他人網路聊天執行個體的使用者識別碼**，來模擬其他人。 為了防止這一點，建議 Bot 開發人員**將使用者識別碼設計得難以猜測**。 如果您啟用了**增強式驗證**選項，Azure Bot 服務將可進一步偵測及拒絕任何使用者識別碼變更。 這表示，從 Direct Line 到 Bot 的訊息，一律會使用您用來初始化網路聊天的相同使用者識別碼 (`Activity.From.Id`)。 請注意，此功能需要以 `dl_` 開頭的使用者識別碼
1. **使用者身分識別**。 您必須注意，您所處理的使用者身分識別有兩個：

    1. 通道中的使用者身分識別。
    1. Bot 在識別提供者中感興趣的使用者身分識別。
  
    當 Bot 要求通道中的使用者 A 登入識別提供者 P 時，登入程序必須確定使用者 A 是登入 P 的使用者。如果允許另一個使用者 B 登入，則使用者 A 將有權透過 Bot 存取使用者 B 的資源。 在網路聊天中，我們有 2 項機制可確保登入的是正確的使用者，說明如下。

    1. 過去，在登入結束時，使用者會看到一個隨機產生的 6 位數代碼 (也稱為神奇代碼)。 使用者必須在起始登入的對話中輸入此代碼，才能完成登入程序。 這項機制常會導致不順暢的使用者體驗。 此外，此機制也容易遭受網路釣魚攻擊。 惡意使用者可透過網路釣魚誘騙其他使用者登入，並取得神奇代碼。

    2. 由於前述方法有這些問題，Azure Bot 服務已不再使用神奇代碼。 Azure Bot 服務可確保登入程序只能在與網路聊天本身**相同的瀏覽器工作階段**中完成。 
    若要啟用這項保護，您必須以 Bot 開發人員的身分使用 **Direct Line 權杖**啟動網路聊天，且該權杖必須包含**可裝載 Bot 網路聊天用戶端的信任網域清單**。 過去，您只能藉由將未記載的選擇性參數傳至 Direct Line 權杖 API 來取得此權杖。 現在，透過增強型驗證選項，您可以在 Direct Line 組態頁面中以靜態方式指定信任網域 (原始) 清單。

### <a name="code-examples"></a>程式碼範例

下列 .NET 控制器可在啟用增強型驗證選項的情況下運作，並傳回 Direct Line 權杖和使用者識別碼。

```csharp
public class HomeController : Controller
{
    public async Task<ActionResult> Index()
    {
        var secret = GetSecret();

        HttpClient client = new HttpClient();

        HttpRequestMessage request = new HttpRequestMessage(
            HttpMethod.Post,
            $"https://directline.botframework.com/v3/directline/tokens/generate");

        request.Headers.Authorization = new AuthenticationHeaderValue("Bearer", secret);

        var userId = $"dl_{Guid.NewGuid()}";

        request.Content = new StringContent(
            JsonConvert.SerializeObject(
                new { User = new { Id = userId } }),
                Encoding.UTF8,
                "application/json");

        var response = await client.SendAsync(request);
        string token = String.Empty;

        if (response.IsSuccessStatusCode)
        {
            var body = await response.Content.ReadAsStringAsync();
            token = JsonConvert.DeserializeObject<DirectLineToken>(body).token;
        }

        var config = new ChatConfig()
        {
            Token = token,
            UserId = userId  
        };

        return View(config);
    }
}

public class DirectLineToken
{
    public string conversationId { get; set; }
    public string token { get; set; }
    public int expires_in { get; set; }
}
public class ChatConfig
{
    public string Token { get; set; }
    public string UserId { get; set; }
}

```

下列 JavaScript 控制器可在啟用增強型驗證選項的情況下運作，並傳回 Direct Line 權杖和使用者識別碼。

```javascript
var router = express.Router(); // get an instance of the express Router

// Get a directline configuration (accessed at GET /api/config)
const userId = "dl_" + createUniqueId();

router.get('/config', function(req, res) {
    const options = {
        method: 'POST',
        uri: 'https://directline.botframework.com/v3/directline/tokens/generate',
        headers: {
            'Authorization': 'Bearer ' + secret
        },
        json: {
            User: { Id: userId }
        }
    };

    request.post(options, (error, response, body) => {
        if (!error && response.statusCode < 300) {
            res.json({ 
                    token: body.token,
                    userId: userId
                });
        }
        else {
            res.status(500).send('Call to retrieve token from Direct Line failed');
        } 
    });
});

```

## <a name="additional-resources"></a>其他資源

- [重要概念](bot-framework-rest-direct-line-3-0-concepts.md)
- [將 Bot 連線至 Direct Line](../bot-service-channel-connect-directline.md)
- [透過 Azure Bot 服務將驗證新增至您的 Bot](../bot-builder-tutorial-authentication.md)
