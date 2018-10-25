---
title: 驗證要求 | Microsoft Docs
description: 了解如何在 Bot 連接器 API 和 Bot 狀態 API 中驗證 API 要求。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
ms.openlocfilehash: 41cc36b7e4abc12bf57df7bf4272dd35031cf251
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/24/2018
ms.locfileid: "49997995"
---
# <a name="authentication"></a>驗證

您的 Bot 會透過安全通道 (SSL/TLS) 使用 HTTP 和 Bot 連接器服務通訊。 當 Bot 傳送要求至連接器服務時，必須包含連接器可用來驗證該 Bot 之身分識別的資訊。 同樣地，當連接器服務傳送要求至 Bot 時，也必須包含 Bot 可用來驗證該服務之身分識別的資訊。 本文描述 Bot 和 Bot 連接器服務之間所進行之服務層級驗證的驗證技術和需求。 若您準備自行撰寫驗證程式碼，您必須實作本文所述的安全性程序，以使 Bot 能與 Bot 連接器服務交換訊息。

> [!IMPORTANT]
> 若您準備自行撰寫驗證程式碼，請務必正確實作所有安全性程序。 透過實作本文中的所有步驟，您將能降低攻擊者讀取傳送至 Bot 的訊息、模擬 Bot 傳送訊息，以及竊取祕密金鑰的風險。 

若您準備使用[適用於 .NET 的 Bot Builder SDK](../dotnet/bot-builder-dotnet-overview.md) 或[適用於 Node.js 的 Bot Builder SDK](../nodejs/index.md)，便不需要實作本文所述的安全性程序，因為 SDK 會自動實作那些程序。 您只需以在[註冊](../bot-service-quickstart-registration.md)期間為 Bot 取得的應用程式識別碼和密碼設定專案，SDK 便會處理剩餘的部分。

> [!WARNING]
> 在 2016 年 12 月，Bot Framework 安全性通訊協定 v3.1 已針對數個用於權杖產生及驗證的值導入變更。 在 2017 年秋末，已推出了 Bot Framework 安全性通訊協定 v3.2，其針對用於權杖產生及驗證的值納入變更。
> 如需詳細資訊，請參閱[安全性通訊協定變更](#security-protocol-changes)。

## <a name="authentication-technologies"></a>驗證技術

一共會使用四個驗證技術來在 Bot 和 Bot 連接器之間建立信任：

| Technology | 說明 |
|----|----|
| **SSL/TLS** | 會針對所有服務對服務連線使用 SSL/TLS。 會使用 `X.509v3` 憑證來建立所有 HTTPS 服務的身分識別。 **用戶端應一律檢查服務憑證以確保其為受信任且有效的憑證。** (此配置「不會」使用用戶端憑證。) |
| **OAuth 2.0** | 會使用 OAuth 2.0 登入至 Microsoft 帳戶 (MSA)/AAD v2 登入服務來產生 Bot 可用來傳送訊息的安全權杖。 此權杖為服務對服務權杖，且不需要使用者登入。 |
| **JSON Web 權杖 (JWT)** | 會使用 JSON Web 權杖來對在 Bot 來回傳送的權杖進行加密。 根據此文章所概述的需求，**用戶端應完整驗證其所接收到的所有 JWT 權杖**。 |
| **OpenID 中繼資料** | Bot 連接器服務會發行其用來在已知的靜態端點將其 JWT 權杖簽署至 OpenID 中繼資料的有效權杖清單。 |

此文章說明如何透過標準 HTTPS 和 JSON 使用這些技術。 不需要任何特殊的 SDK，不過適用於 OpenID 等的協助程式可能對您會有幫助。

## <a id="bot-to-connector"></a> 驗證從 Bot 傳送至 Bot 連接器服務的要求

若要與 Bot 連接器服務通訊，您必須使用下列格式在每個 API 要求的 `Authorization` 標頭中指定存取權杖： 

```http
Authorization: Bearer ACCESS_TOKEN
```

此圖表示範 Bot 至連接器的驗證步驟：

![在驗證 MSA 登入服務之後再驗證 Bot](../media/connector/auth_bot_to_bot_connector.png)

> [!IMPORTANT]
> 若您尚未這麼做，請務必向 Bot Framework [註冊您的 Bot](../bot-service-quickstart-registration.md)，以取得其應用程式識別碼和密碼。 您將需要 Bot 的應用程式識別碼和密碼以要求存取權杖。

### <a name="step-1-request-an-access-token-from-the-msaaad-v2-login-service"></a>步驟 1：從 MSA/AAD v2 登入服務要求存取權杖

若要從 MSA/AAD v2 登入服務要求存取權杖，請發出下列要求，並將 **MICROSOFT-APP-ID** 和 **MICROSOFT-APP-PASSWORD** 取代為您向 Bot Framework [註冊](../bot-service-quickstart-registration.md)您的 Bot 時所取得的應用程式識別碼和密碼。

```http
POST https://login.microsoftonline.com/botframework.com/oauth2/v2.0/token
Host: login.microsoftonline.com
Content-Type: application/x-www-form-urlencoded

grant_type=client_credentials&client_id=MICROSOFT-APP-ID&client_secret=MICROSOFT-APP-PASSWORD&scope=https%3A%2F%2Fapi.botframework.com%2F.default
```

### <a name="step-2-obtain-the-jwt-token-from-the-msaaad-v2-login-service-response"></a>步驟 2：從 MSA/AAD v2 登入服務回應取得 JWT 權杖

若您的應用程式已由 MSA/AAD v2 登入服務授權，JSON 回應主體將會指定您的存取權杖、其類型，以及其到期時間 (以秒為單位)。 

將權杖加入至要求的 `Authorization` 標頭時，您必須使用於此回應中所指定的確切值 (亦即不要對權杖值進行逸出或編碼)。 存取權杖在到期之後便會失效。 若要避免權杖到期影響 Bot 的效能，您可以選擇對權杖進行快取並主動重新整理它。

此範例示範來自 MSA/AAD v2 登入服務的回應：

```http
HTTP/1.1 200 OK
... (other headers) 
```

```json
{
    "token_type":"Bearer",
    "expires_in":3600,
    "ext_expires_in":3600,
    "access_token":"eyJhbGciOiJIUzI1Ni..."
}
```

### <a name="step-3-specify-the-jwt-token-in-the-authorization-header-of-requests"></a>步驟 3：在要求的 Authorization 標頭中指定 JWT 權杖

當您將 API 要求傳送至 Bot 連接器服務時，請使用下列格式在要求的 `Authorization` 標頭中指定存取權杖：

```http
Authorization: Bearer ACCESS_TOKEN
```

您傳送至 Bot 連接器服務的所有要求，都必須在 `Authorization` 標頭中包含存取權杖。 若權杖格式正確、尚未到期且是由 MSA/AAD v2 登入服務所產生，Bot 連接器服務將會授權該要求。 系統會執行額外檢查，以確保權杖是屬於傳送要求的 Bot。

下列範例示範如何在要求的 `Authorization` 標頭中指定存取權杖。 

```http
POST https://smba.trafficmanager.net/apis/v3/conversations/12345/activities 
Authorization: Bearer eyJhbGciOiJIUzI1Ni...
    
(JSON-serialized Activity message goes here)
```

> [!IMPORTANT]
> 請僅在傳送至 Bot 連接器服務之要求的 `Authorization` 標頭中指定 JWT 權杖。 請「不要」透過不安全的通道傳送權杖，且「不要」將它包含在您傳送至其他服務的 HTTP 要求中。 您從 MSA/AAD v2 登入服務取得的 JWT 權杖就像密碼一樣，因此應該謹慎處理。 任何擁有該權杖的人都可以使用它來代表您的 Bot 執行作業。 

#### <a name="bot-to-connector-example-jwt-components"></a>Bot 至連接器：範例 JWT 元件

```json
header:
{
  typ: "JWT",
  alg: "RS256",
  x5t: "<SIGNING KEY ID>",
  kid: "<SIGNING KEY ID>"
},
payload:
{
  aud: "https://api.botframework.com",
  iss: "https://sts.windows.net/d6d49420-f39b-4df7-a1dc-d59a935871db/",
  nbf: 1481049243,
  exp: 1481053143,
  appid: "<YOUR MICROSOFT APP ID>",
  ... other fields follow
}
```

> [!NOTE]
> 實際的欄位可能會有所不同。 請以上述方式建立並驗證所有 JWT 權杖。

## <a id="connector-to-bot"></a> 驗證從 Bot 連接器服務傳送至 Bot 的要求

當 Bot 連接器服務傳送要求至您的 Bot 時，它會在要求的 `Authorization` 標頭中指定已簽署的 JWT 權杖。 您的 Bot 可透過確認該已簽署 JWT 權杖的真確性，以驗證來自 Bot 連接器服務的呼叫。 

此圖表示範連接器至 Bot 的驗證步驟：

![驗證從 Bot 連接器傳送至您 Bot 的呼叫](../media/connector/auth_bot_connector_to_bot.png)

### <a id="openid-metadata-document"></a> 步驟 2：取得 OpenID 中繼資料文件

OpenID 中繼資料文件會指定第二份文件的位置，其中會列出 Bot 連接器服務的有效簽署金鑰。 若要取得 OpenID 中繼資料文件，請透過 HTTPS 發出此要求：

```http
GET https://login.botframework.com/v1/.well-known/openidconfiguration
```

> [!TIP]
> 這是一個您可以硬式編碼至應用程式的靜態 URL。 

下列範例示範回應 `GET` 要求所傳回的 OpenID 中繼資料文件。 `jwks_uri` 屬性指定包含 Bot 連接器服務之有效簽署金鑰的文件位置。

```json
{
    "issuer": "https://api.botframework.com",
    "authorization_endpoint": "https://invalid.botframework.com",
    "jwks_uri": "https://login.botframework.com/v1/.well-known/keys",
    "id_token_signing_alg_values_supported": [
      "RS256"
    ],
    "token_endpoint_auth_methods_supported": [
      "private_key_jwt"
    ]
}
```

### <a id="connector-to-bot-step-3"></a> 步驟 3：取得有效簽署金鑰的清單

若要取得有效簽署金鑰的清單，請針對由 OpenID 中繼資料文件中的 `jwks_uri` 屬性所指定的 URL，透過 HTTPS 發出 `GET` 要求。 例如︰

```http
GET https://login.botframework.com/v1/.well-known/keys
```

回應主體會以 [JWK 格式](https://tools.ietf.org/html/rfc7517) \(英文\) 指定文件，但也會針對每個金鑰包含額外屬性：`endorsements`。 金鑰清單本身相對穩定，並可以長時間進行快取 (在 Bot 建立器 SDK 中的預設值為 5 天)。

每個金鑰內的 `endorsements` 屬性都包含一或多個簽署字串，您可以使用它們來驗證連入要求之 [Activity][Activity] 物件內的 `channelId` 屬性所指定之通道識別碼的真確性。 需要進行簽署之通道識別碼的清單，可在每個 Bot 內設定。 根據預設，它將會是所有已發佈之通道識別碼的清單，雖然 Bot 開發人員可能會覆寫特定通道識別碼的值。 若通道識別碼需要進行簽署：

- 您應該要求搭配該通道識別碼傳送至您 Bot 的任何 [Activity][Activity] 物件，都應具有針對該頻道進行簽署的 JWT 權杖。 
- 若簽署不存在，您的 Bot 應傳回 **HTTP 403 (禁止)** 狀態碼來拒絕該要求。

### <a name="step-4-verify-the-jwt-token"></a>步驟 4：驗證 JWT 權杖

若要驗證由 Bot 連接器服務所傳送之權杖的真確性，您必須從要求的 `Authorization` 標頭擷取該權杖，對它進行剖析並驗證其內容，然後驗證其簽章。 

JWT 剖析程式庫可供許多平台使用，且大多數都會針對 JWT 權杖實作安全且可靠的剖析，不過您通常必須設定這些程式庫，以要求權杖的某些特性 (其簽發者、對象等) 需包含正確的值。 剖析權杖時，您必須設定剖析程式庫或自行撰寫驗證，以確保權杖能符合這些需求：

1. 權杖是在 HTTP `Authorization` 標頭中傳送，並具有「持有人」配置。
2. 權杖為符合 [JWT 標準](http://openid.net/specs/draft-jones-json-web-token-07.html) \(英文\) 的有效 JSON。
3. 權杖包含具有 `https://api.botframework.com` 值的「簽發者」宣告。
4. 權杖包含具有與 Bot 的 Microsoft 應用程式識別碼相等之值的「對象」宣告。
5. 權杖仍處於其有效期間之內。 業界標準的時鐘誤差為 5 分鐘。
6. 透過使用於在[步驟 2](#openid-metadata-document) 中所擷取之 Open ID 中繼資料文件的 `id_token_signing_alg_values_supported` 屬性中所指定的簽署演算法，搭配列於在[步驟 3](#connector-to-bot-step-3) 中所擷取之 OpenID 金鑰文件中的金鑰，使權杖具有有效的密碼編譯簽章。
7. 權杖包含 "serviceUrl" 宣告，其值符合連入要求之 [Activity][Activity] 物件根目錄中的 `servieUrl` 屬性。 

若權杖不符合上述所有需求，您的 Bot 應傳回 **HTTP 403 (禁止)** 狀態碼來拒絕該要求。

> [!IMPORTANT]
> 這些需求都非常重要，特別是需求 4 和 6。 若無法實作上述所有驗證需求，將會使 Bot 處於被攻擊的風險中，並可能導致 Bot 洩露其 JWT 權杖。

實作者不應該公開停用傳送至 Bot 之 JWT 權杖的驗證方法。

#### <a name="connector-to-bot-example-jwt-components"></a>連接器至 Bot：範例 JWT 元件

```json
header:
{
  typ: "JWT",
  alg: "RS256",
  x5t: "<SIGNING KEY ID>",
  kid: "<SIGNING KEY ID>"
},
payload:
{
  aud: "<YOU MICROSOFT APP ID>",
  iss: "https://api.botframework.com",
  nbf: 1481049243,
  exp: 1481053143,
  ... other fields follow
}
```

> [!NOTE]
> 實際的欄位可能會有所不同。 請以上述方式建立並驗證所有 JWT 權杖。

## <a id="emulator-to-bot"></a> 驗證從 Bot Framework 模擬器傳送至 Bot 的要求

> [!WARNING]
> 在 2017 年秋末，已推出了 Bot Framework 安全性通訊協定 v3.2。 這個新的版本將會在 Bot Framework Eumaltor 和您的 Bot 之間交換的權杖內包含新的「簽發者」值。 為了針對此變更做準備，下列步驟概述檢查 v3.1 和 v3.2 簽發者值的方法。 

[Bot Framework Emulator](../bot-service-debug-emulator.md) 是可用來測試 Bot 功能的傳統型工具。 雖然 Bot Framework Emulator 是使用和上述相同的[驗證技術](#authentication-technologies)，它並無法模擬真實的 Bot 連接器服務。 相反地，它會使用您將模擬器連線至 Bot 時所指定的 Microsoft 應用程式識別碼和 Microsoft 應用程式密碼，來建立與 Bot 所建立的權杖完全相同的權杖。 當模擬器傳送要求至您的 Bot 時，它會在要求的 `Authorization` 標頭中指定 JWT 權杖，基本上便是使用 Bot 自己的認證來驗證要求。 

若您要實作驗證程式庫，並想要接受來自 Bot Framework Emulator 的要求，您必須加入這個額外的驗證路徑。 路徑的結構類似[連接器 -> Bot](#connector-to-bot) 的驗證路徑，但它會使用 MSA 的 OpenID 文件，而非 Bot 連接器的 OpenID 文件。

此圖表示範模擬器至 Bot 的驗證步驟：

![驗證從 Bot Framework 模擬器傳送至您 Bot 的呼叫](../media/connector/auth_bot_framework_emulator_to_bot.png)

---
### <a name="step-2-get-the-msa-openid-metadata-document"></a>步驟 2：取得 MSA OpenID 中繼資料文件

OpenID 中繼資料文件會指定第二份文件的位置，其中會列出有效簽署金鑰。 若要取得 MSA OpenID 中繼資料文件，請透過 HTTPS 發出此要求：

```http
GET https://login.microsoftonline.com/botframework.com/v2.0/.well-known/openid-configuration
```

下列範例示範回應 `GET` 要求所傳回的 OpenID 中繼資料文件。 `jwks_uri` 屬性會指定包含有效簽署金鑰之文件的位置。

```json
{
    "authorization_endpoint":"https://login.microsoftonline.com/common/oauth2/v2.0/authorize",
    "token_endpoint":"https://login.microsoftonline.com/common/oauth2/v2.0/token",
    "token_endpoint_auth_methods_supported":["client_secret_post","private_key_jwt"],
    "jwks_uri":"https://login.microsoftonline.com/common/discovery/v2.0/keys",
    ...
}
```

### <a id="emulator-to-bot-step-3"></a> 步驟 3：取得有效簽署金鑰的清單

若要取得有效簽署金鑰的清單，請針對由 OpenID 中繼資料文件中的 `jwks_uri` 屬性所指定的 URL，透過 HTTPS 發出 `GET` 要求。 例如︰

```http
GET https://login.microsoftonline.com/common/discovery/v2.0/keys 
Host: login.microsoftonline.com
```

回應主體會以 [JWK 格式](https://tools.ietf.org/html/rfc7517) \(英文\) 指定文件。 

### <a name="step-4-verify-the-jwt-token"></a>步驟 4：驗證 JWT 權杖

若要驗證由模擬器所傳送之權杖的真確性，您必須從要求的 `Authorization` 標頭擷取該權杖，對它進行剖析並驗證其內容，然後驗證其簽章。 

JWT 剖析程式庫可供許多平台使用，且大多數都會針對 JWT 權杖實作安全且可靠的剖析，不過您通常必須設定這些程式庫，以要求權杖的某些特性 (其簽發者、對象等) 需包含正確的值。 剖析權杖時，您必須設定剖析程式庫或自行撰寫驗證，以確保權杖能符合這些需求：

1. 權杖是在 HTTP `Authorization` 標頭中傳送，並具有「持有人」配置。
2. 權杖為符合 [JWT 標準](http://openid.net/specs/draft-jones-json-web-token-07.html) \(英文\) 的有效 JSON。
3. 權杖包含具有 `https://sts.windows.net/d6d49420-f39b-4df7-a1dc-d59a935871db/` 或 `https://sts.windows.net/f8cdef31-a31e-4b4a-93e4-5f571e91255a/` 值的「簽發者」宣告 (檢查這兩個簽發者值將能確保您會同時檢查安全性通訊協定 v3.1 和 v3.2 的簽發者值)。
4. 權杖包含具有與 Bot 的 Microsoft 應用程式識別碼相等之值的「對象」宣告。
5. 權杖包含具有與 Bot 的 Microsoft 應用程式識別碼相等之值的「應用程式識別碼」宣告。
6. 權杖仍處於其有效期間之內。 業界標準的時鐘誤差為 5 分鐘。
7. 權杖具有具備於[步驟 3](#emulator-to-bot-step-3) 中所擷取的 OpenID 金鑰文件中所列之金鑰的有效密碼編譯簽章。

> [!NOTE]
> 需求 5 為針對模擬器驗證路徑的特定需求。 

若權杖不符合上述所有需求，您的 Bot 應傳回 **HTTP 403 (禁止)** 狀態碼來終止該要求。

> [!IMPORTANT]
> 這些需求都非常重要，特別是需求 4 和 7。 若無法實作上述所有驗證需求，將會使 Bot 處於被攻擊的風險中，並可能導致 Bot 洩露其 JWT 權杖。

#### <a name="emulator-to-bot-example-jwt-components"></a>模擬器至 Bot：範例 JWT 元件

```json
header:
{
  typ: "JWT",
  alg: "RS256",
  x5t: "<SIGNING KEY ID>",
  kid: "<SIGNING KEY ID>"
},
payload:
{
  aud: "<YOUR MICROSOFT APP ID>",
  iss: "https://sts.windows.net/d6d49420-f39b-4df7-a1dc-d59a935871db/",
  nbf: 1481049243,
  exp: 1481053143,
  ... other fields follow
}
```

> [!NOTE]
> 實際的欄位可能會有所不同。 請以上述方式建立並驗證所有 JWT 權杖。

## <a name="security-protocol-changes"></a>安全性通訊協定變更

> [!WARNING]
> 針對安全性通訊協定 v3.0 的支援，已在 **2017 年 7 月 31 日**中止。 若您有自行撰寫驗證程式碼 (也就是沒有使用 Bot Builder SDK 來建立 Bot)，您必須更新應用程式以使用下列的 v3.1 值，來升級至安全性通訊協定 v3.1。 

### <a name="bot-to-connector-authenticationbot-to-connector"></a>[Bot 至連接器驗證](#bot-to-connector)

#### <a name="oauth-login-url"></a>OAuth 登入 URL

| 通訊協定版本 | 有效值 |
|----|----|
| v3.1 與 v3.2 | `https://login.microsoftonline.com/botframework.com/oauth2/v2.0/token` |

#### <a name="oauth-scope"></a>OAuth 範圍

| 通訊協定版本 | 有效值 |
|----|----|
| v3.1 與 v3.2 |  `https://api.botframework.com/.default` |

### <a name="connector-to-bot-authenticationconnector-to-bot"></a>[連接器至 Bot 驗證](#connector-to-bot)

#### <a name="openid-metadata-document"></a>OpenID 中繼資料文件

| 通訊協定版本 | 有效值 |
|----|----|
| v3.1 與 v3.2 | `https://login.botframework.com/v1/.well-known/openidconfiguration` |

#### <a name="jwt-issuer"></a>JWT 簽發者

| 通訊協定版本 | 有效值 |
|----|----|
| v3.1 與 v3.2 | `https://api.botframework.com` |

### <a name="emulator-to-bot-authenticationemulator-to-bot"></a>[模擬器至 Bot 驗證](#emulator-to-bot)

#### <a name="oauth-login-url"></a>OAuth 登入 URL

| 通訊協定版本 | 有效值 |
|----|----|
| v3.1 與 v3.2 | `https://login.microsoftonline.com/botframework.com/oauth2/v2.0/token` |

#### <a name="oauth-scope"></a>OAuth 範圍

| 通訊協定版本 | 有效值 |
|----|----|
| v3.1 與 v3.2 |  您 Bot 的 Microsoft 應用程式識別碼 + `/.default` |

#### <a name="jwt-audience"></a>JWT 對象

| 通訊協定版本 | 有效值 |
|----|----|
| v3.1 與 v3.2 | 您 Bot 的 Microsoft 應用程式識別碼 |

#### <a name="jwt-issuer"></a>JWT 簽發者

| 通訊協定版本 | 有效值 |
|----|----|
| v3.1 | `https://sts.windows.net/d6d49420-f39b-4df7-a1dc-d59a935871db/` |
| v3.2 | `https://sts.windows.net/f8cdef31-a31e-4b4a-93e4-5f571e91255a/` |

#### <a name="openid-metadata-document"></a>OpenID 中繼資料文件

| 通訊協定版本 | 有效值 |
|----|----|
| v3.1 與 v3.2 | `https://login.microsoftonline.com/botframework.com/v2.0/.well-known/openid-configuration` |

## <a name="additional-resources"></a>其他資源

- [對 Bot Framework 驗證進行疑難排解](../bot-service-troubleshoot-authentication-problems.md)
- [JSON Web 權杖 (JWT) draft-jones-json-web-token-07](http://openid.net/specs/draft-jones-json-web-token-07.html) \(英文\)
- [JSON Web 簽章 (JWS) draft-jones-json-web-signature-04](https://tools.ietf.org/html/draft-jones-json-web-signature-04) \(英文\)
- [JSON Web 金鑰 (JWK) RFC 7517](https://tools.ietf.org/html/rfc7517) \(英文\)

[Activity]: bot-framework-rest-connector-api-reference.md#activity-object
