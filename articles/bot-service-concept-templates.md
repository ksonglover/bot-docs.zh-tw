---
title: Bot 服務範本 | Microsoft Docs
description: 了解可在使用 Bot 服務建立 Bot 時使用的不同範本。
author: robstand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: f1cfa6de6dbc0b1dc6ed0408f405a91493741a93
ms.sourcegitcommit: a6d02ec4738e7fc90b7108934740e9077667f3c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/04/2019
ms.locfileid: "70297630"
---
# <a name="bot-service-templates"></a>Bot 服務範本

[!INCLUDE [pre-release-label](includes/pre-release-label-v3.md)]

Bot 服務包含五個範本以協助您開始建置 Bot。 這些範本會提供現成的完整功能 Bot 以協助您快速入門。 當您[建立 Bot](bot-service-quickstart.md) 時，您會選擇適用於該 Bot 的範本和 SDK 語言。

每個範本都會提供以 Bot 的其中一項核心功能為基礎的起始點。 

## <a name="basic-bot"></a>基本 Bot
若要建立使用對話方塊來回應使用者輸入的 Bot，請選擇 [基本] 範本。 [基本]  範本會建立一開始具有最少檔案集合和程式碼的 Bot。 Bot 會在使用者輸入時做出回應。 您可以使用此範本來開始在 Bot 中建置對話流程。

## <a name="form-bot"></a>表單 Bot
若要建立會透過引導式對話收集使用者輸入的 Bot，請選擇 [表單]  範本。 [表單] 範本是設計來從使用者收集特定的資訊集合。 例如，設計來取得使用者三明治訂單的 Bot，將會需要收集如麵包類型、配料選擇、三明治大小等的資訊。

以 C# 語言搭配 [表單] 範本所建立的 Bot 會使用 [FormFlow](dotnet/bot-builder-dotnet-formflow.md) 來管理表單，而以 Node.js 語言搭配 [表單] 範本所建立的 Bot 則會使用 [waterfalls](nodejs/bot-builder-nodejs-dialog-waterfall.md) 來管理表單。

## <a name="language-understanding-bot"></a>語言理解 Bot
若要建立使用自然語言模型來了解使用者意圖的 Bot，請選擇 [語言理解]  範本。 此範本會利用 <a href="https://www.luis.ai" target="_blank">Language Understanding (LUIS)</a> \(英文\) 來提供了解自然語言的功能。

若使用者傳送如 "get news about virtual reality companies" (取得虛擬實境公司的相關新聞) 的訊息，您的 Bot 便可以使用 LUIS 來解譯該訊息的意思。 透過使用 LUIS，您可以迅速地部署 HTTP 端點，其將會根據使用者輸入所傳遞的意圖 (尋找新聞) 和存在的主要實體 (虛擬實境公司) 來解譯該輸入。 LUIS 可讓您指定與應用程式相關的意圖和實體集合，並引導您完成語言理解應用程式的建置程序。

當您使用 [語言理解] 範本建立 Bot 時，Bot 服務會建立相對應的空白 LUIS 應用程式 (也就是會一律傳回 `None`)。 若要更新 LUIS 應用程式模型以使它能解譯使用者輸入，您必須登入 <a href="https://www.luis.ai" target="_blank">LUIS</a> \(英文\)，按一下 [My applications]  \(我的應用程式\)，選取服務為您建立的應用程式，然後建立意圖，指定實體，並將應用程式定型。

## <a name="question-and-answer-bot"></a>問題與答案 Bot
若要建立能將如問題與答案配對的半結構化資料擷取為具區別性且有幫助之答案的 Bot，請選擇 [問題與答案]  範本。 此範本會利用 <a href="https://qnamaker.ai">QnA Maker</a> \(英文\) 服務來剖析問題並提供答案。 

當您使用 [問題與答案] 範本建立 Bot 時，您必須登入 <a href="https://qnamaker.ai">QnA Maker</a> \(英文\) 並建立新的 QnA 服務。 此 QnA 服務將會提供知識庫識別碼和訂用帳戶金鑰，可讓您用來更新 **QnAKnowldegebasedId** 和 **QnASubscriptionKey** 的[應用程式設定](bot-service-manage-settings.md)值。 設定好那些值之後，您的 Bot 便能回答 QnA 服務在其知識庫中具有答案的任何問題。

## <a name="proactive-bot"></a>主動式 Bot
若要建立可以傳送主動式訊息給使用者的 Bot，請選擇 [主動式]  範本。 一般而言，Bot 直接傳送給使用者的每個訊息都與使用者先前的輸入相關。 不過，在某些情況下，Bot 可能需要將與使用者的最新訊息沒有直接關聯的資訊傳送給該使用者。 這些類型的訊息稱為**主動式訊息**。 主動式訊息在許多案例中皆很有用。 例如，如果 Bot 設定了計時器或提醒，它便可能需要在到達該時間時通知使用者。 或者，如果 Bot 接收到有關外部事件的通知，它可能需要將該資訊傳達給使用者。 

當您使用 [主動式] 範本建立 Bot 時，系統會自動建立數個 Azure 資源並將它們加入至您的資源群組。 根據預設，這些 Azure 資源都會設定為啟用非常簡單的主動式傳訊案例。 

| 資源 | 說明 |
|----|----|
| Azure 儲存體 | 用來建立佇列。 |
| Azure 函數應用程式 | `queueTrigger` Azure 函式，會在佇列中有訊息時觸發。 它會使用 [Direct Line](https://docs.microsoft.com/bot-framework/rest-api/bot-framework-rest-direct-line-3-0-concepts) \(英文\) 與 Bot 服務通訊。 此函式會使用 Bot 繫結來以觸發程序承載之一部分的形式傳送訊息。 我們的範例函式會從佇列以現況的方式轉送使用者的訊息。
| Bot 服務 | 您的 Bot。 包含下列邏輯：接收來自使用者的訊息、將訊息加入至 Azure 佇列、接收來自 Azure 函式的觸發程序，以及透過觸發程序的承載將其所接收到的訊息傳回。 |

下圖顯示在您使用 [主動式] 範本建立 Bot 時，觸發事件的運作方式。

![主動式 Bot 範例概觀](~/media/bot-proactive-diagram.png)

程序會從使用者透過 Bot Framework 伺服器傳送訊息至您的 Bot 開始 (步驟 1)。

## <a name="next-steps"></a>後續步驟
您已了解各種不同的範本，現在請繼續了解用於 Bot 內的認知服務。

> [!div class="nextstepaction"]
> [Bot 的認知服務](bot-service-concept-intelligence.md)
