---
title: 驗證 | Microsoft Docs
description: 了解如何在 Direct Line API v3.0 中驗證 API 要求。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ms.openlocfilehash: 18a5815a96e2052a54c48f6af211d8b28e20d983
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/18/2018
ms.locfileid: "39299366"
---
# <a name="authentication"></a>驗證

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

## <a name="additional-resources"></a>其他資源

- [重要概念](bot-framework-rest-direct-line-3-0-concepts.md)
- [將 Bot 連線至 Direct Line](../bot-service-channel-connect-directline.md)