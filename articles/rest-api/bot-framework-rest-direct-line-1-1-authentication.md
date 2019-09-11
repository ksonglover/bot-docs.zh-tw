---
title: 驗證 | Microsoft Docs
description: 了解如何在 Direct Line API v1.1 中驗證 API 要求。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
ms.openlocfilehash: 555cb3298114c3eb8ba8a4e1c41b5515e929fd91
ms.sourcegitcommit: a6d02ec4738e7fc90b7108934740e9077667f3c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/04/2019
ms.locfileid: "70299638"
---
# <a name="authentication"></a>Authentication

> [!IMPORTANT]
> 本文描述 Direct Line API 1.1 中的驗證。 如果要在用戶端應用程式與 Bot 之間建立新連線，請改為使用 [Direct Line API 3.0](bot-framework-rest-direct-line-3-0-authentication.md)。

用戶端可以藉由在 Bot Framework 入口網站使用您從 [Direct Line 頻道設定頁面](../bot-service-channel-connect-directline.md)取得的**祕密**來驗證 Direct Line API 1.1 的要求，或使用您在執行階段取得的**權杖**來驗證。

祕密或權杖應該指定在每個要求的 `Authorization` 標頭，使用持有人配置或 "BotConnector" 配置。 

**持有人配置**：
```http
Authorization: Bearer SECRET_OR_TOKEN
```

**BotConnector 配置**：
```http
Authorization: BotConnector SECRET_OR_TOKEN
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
POST https://directline.botframework.com/api/tokens/conversation
Authorization: Bearer SECRET
```

在此要求的 `Authorization` 標頭中，使用您的 Direct Line 祕密值來取代 **SECRET**。

下列程式碼片段提供產生權杖要求和回應的範例。

### <a name="request"></a>要求

```http
POST https://directline.botframework.com/api/tokens/conversation
Authorization: Bearer RCurR_XV9ZA.cwA.BKA.iaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0
```

### <a name="response"></a>Response

如果要求成功，回應會包含適用於一個對話的權杖。 權杖將於 30 分鐘過期。 若要讓權杖仍然可用，您必須在它過期之前[重新整理權杖](#refresh-token)。

```http
HTTP/1.1 200 OK
[other headers]

"RCurR_XV9ZA.cwA.BKA.iaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0y8qbOF5xPGfiCpg4Fv0y8qqbOF5x8qbOF5xn"
```

### <a name="generate-token-versus-start-conversation"></a>產生權杖與開始對話

產生權杖作業 (`POST /api/tokens/conversation`) 和[開始對話](bot-framework-rest-direct-line-1-1-start-conversation.md)作業 (`POST /api/conversations`) 類似，兩個作業都會傳回可用於存取單一對話的 `token`。 不過，不同於「開始對話」作業，產生權杖作業不會開始對話或連絡 Bot。 

如果您計畫要將權杖散發給用戶端，並且要它們起始對話，請使用「產生權杖」作業。 如果您想要立即開始對話，請改用[開始對話](bot-framework-rest-direct-line-1-1-start-conversation.md)作業。

## <a id="refresh-token"></a> 重新整理 Direct Line 權杖

Direct Line 權杖的有效期為從產生起算 30 分鐘的時間，而且可以重新整理，時間量不限，只要未過期即可。 無法重新整理過期的權杖。 若要重新整理 Direct Line 權杖，請發出此要求：

```http
POST https://directline.botframework.com/api/tokens/{conversationId}/renew
Authorization: Bearer TOKEN_TO_BE_REFRESHED
```

在此要求的 URI，將 **{conversationId}** 取代為權杖有效的對話識別碼，並在此要求的 `Authorization` 標頭中，將 **TOKEN_TO_BE_REFRESHED** 取代為您要重新整理的 Direct Line 權杖。

下列程式碼片段提供重新整理權杖要求和回應的範例。

### <a name="request"></a>要求

```http
POST https://directline.botframework.com/api/tokens/abc123/renew
Authorization: Bearer CurR_XV9ZA.cwA.BKA.iaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0y8qbOF5xPGfiCpg4Fv0y8qqbOF5x8qbOF5xn
```

### <a name="response"></a>Response

如果要求成功，回應會包含適用於與先前權杖相同之對話的新權杖。 新權杖將於 30 分鐘過期。 若要讓新權杖仍然可用，您必須在它過期之前[重新整理權杖](#refresh-token)。

```http
HTTP/1.1 200 OK
[other headers]

"RCurR_XV9ZA.cwA.BKA.y8qbOF5xPGfiCpg4Fv0y8qqbOF5x8qbOF5xniaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0"
```

## <a name="additional-resources"></a>其他資源

- [重要概念](bot-framework-rest-direct-line-1-1-concepts.md)
- [將 Bot 連線至 Direct Line](../bot-service-channel-connect-directline.md)