---
title: 活動處理 |Microsoft Docs
description: 了解 Bot SDK 中的活動處理程序。
keywords: bot 介面卡, 自訂中介軟體, 最少運算, 後援, 事件處理常式
author: jonathanfingold
ms.author: jonathanfingold
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 05/23/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 0e0fe51e33e99bbb35ef1d731b611ff9ca12a138
ms.sourcegitcommit: a295a90eac461f8b96770dd902ba44919acf33fc
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/26/2019
ms.locfileid: "67404308"
---
# <a name="activity-processing"></a>活動處理

[!INCLUDE[applies-to](../includes/applies-to.md)]

Bot 和使用者會透過活動來互動並交換資訊。 Bot 應用程式收到的每個活動都會傳遞至 Bot 介面卡，而其可將活動資訊傳遞給 Bot 邏輯，最終再傳送回應給使用者。 接收活動與透過 Bot 的後續處理，稱為一回合；這代表 Bot 的一個完整的週期。 完成所有的執行、活動已完全處理，並且 Bot 的所有層級均已完成時，即為一輪結束。

系統將以非同步方式處理活動，特別是在 Bot 回合期間[從 Bot 傳送](#generating-responses)的活動。 其為建置 Bot 的必要部分；如果您想要溫習所有作業的運作方式，請選擇您適用的語言，並參閱 [.NET 非同步](https://docs.microsoft.com/dotnet/csharp/async)或 [JavaScript 非同步](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Statements/async_function)。

## <a name="the-bot-adapter"></a>Bot 介面卡

Bot 介面卡可封裝驗證處理程序，並可與 Bot 連接器服務相互傳送和接收活動。 Bot 接收活動時，介面卡會包裝與該活動有關的所有項目、為該回合建立[內容物件](#turn-context)、將物件傳送至 Bot 應用程式邏輯，以及將 Bot 產生的回應傳回至使用者通道。

## <a name="authentication"></a>Authentication

介面卡可使用活動的資訊和 REST 要求的 `Authentication` 標頭，驗證應用程式接收的每項內送活動。 介面卡會使用連接器物件及您的應用程式認證，驗證連出給使用者的活動。

Bot 連接器服務驗證採用 JWT (JSON Web 權杖) `Bearer` 權杖和 **Microsoft 應用程式識別碼**，以及 Azure 再您建立 Bot 服務或註冊 Bot 時為您建立的 **Microsoft 應用程式密碼**。 應用程式必須在初始化期間使用這些認證，介面卡才能驗證流量。

> [!NOTE]
> 例如，如果您目前已在本機執行或測試 Bot，只要使用 Bot Framework 模擬器即可達到此目標，您無須再設定介面卡驗證來回 Bot 的流量。

## <a name="turn-context"></a>回合內容

介面卡收到活動時會整合**回合內容**物件，其中提供內送活動、傳送者及收件者、通道、交談相關資訊，以及其他處理活動所需的資料。 接著，介面卡會將此內容物件傳遞至 Bot。 在整個回合期間，內容物件都會持續存在，並提供下列資訊：

* 交談 - 識別交談並加入參加交談之 Bot 和使用者的資訊。
* 活動 - 交談中的要求和回覆即所有類型的活動。 此內容將提供內送活動的相關資訊，包括路由資訊、通道資訊、交談、傳送者及收件者資訊。
* 自訂資訊 – 如果您實作中介軟體或在 Bot 邏輯中擴充 Bot 服務，則在每個回合中還可取得其他額外資訊。

內容物件也可用於傳送回應至使用者，並取得介面卡參考資料<!-- to create a new conversation or continue an existing one-->上也提供本文中使用的原始碼。

> [!NOTE]
> 應用程式和介面卡將以非同步方式處理要求，不過您的業務邏輯不需要是「要求-回應」導向。

## <a name="middleware"></a>中介軟體

您可將中介軟體新增至介面卡。 中介軟體和 Bot 邏輯可使用內容物件擷取活動相關資訊，並據此採取行動。 中介軟體和 Bot 亦可更新或新增資訊至內容物件，例如：追蹤回合、交談或其他範圍的狀態。 如需有關中介軟體的深入詳細資訊，請參閱[中介軟體文章](~/v4sdk/bot-builder-concept-middleware.md)。

## <a name="generating-responses"></a>產生回應

內容物件提供活動回應方法，讓程式碼回應活動：

* _傳送活動_和_傳送活動_方法可傳送一或多個活動至對話。
* 如果通道支援，_更新活動_方法也可更新交談內的活動。
* 如果通道支援，_刪除活動_方法也可移除交談內的活動。

每個回應方法都會以非同步程序執行。 系統呼叫活動回應方法時，該方法開始呼叫處理常式之前，會先複製相關聯的[事件處理常式](#response-event-handlers)清單，這代表其中包含每個要在此點上新增的每個處理常式，但不包含處理程序開始後新增的項目。

此外，這也表示無法保證回應順序，特別是其中一個工作比另一個更複雜的時候。 如果 Bot 可對內送活動產生多個回應，請確保使用者收到時無論什麼順序都符合常理。

> [!IMPORTANT]
> 處理主要 Bot 回合的執行緒可在其完成後處置內容物件。 如果回應 (包含其處理常式) 佔用大量時間，並嘗試在內容物件上動作，則可能會取得 `Context was disposed` 錯誤。 **請務必`await`任何活動呼叫**，這樣主要執行緒才能先等候已產生的活動，再結束其正在處理的工作並處置回合內容。

## <a name="response-event-handlers"></a>回應事件處理常式

除了 Bot 和中介軟體邏輯，回應處理常式 (有時也指事件處理常式，或活動事件處理常式) 亦可新增至內容物件。 目前的內容物件發生相關聯[回應](#generating-responses)時，系統在執行實際回應之前會先呼叫處理常式。 如果您已經知道想要針對其餘目前回應中，該類型的每個活動要執行哪些動作 (無論是在實際事件之前或之後執行)，這些處理常式非常實用。

> [!WARNING]
> 請特別小心，不要在其本身的個別回應事件處理常式內呼叫活動回應方法，例如：從_傳送活動_處理常式內呼叫傳送活動方法。 這樣可能會產生無限迴圈。

每個新活動都取得要在其中執行的新執行緒。 建立執行緒處理活動時，該活動的處理常式清單會複製到該執行緒。 系統不會針對該特定活動事件，執行該時間點後新增的任何處理常式。

內容物件上所註冊的處理常式，其處理方式非常類似於介面卡管理[中介軟體管線](~/v4sdk/bot-builder-concept-middleware.md#the-bot-middleware-pipeline)的方式。 也就是說，系統將依照處理常式新增的順序進行呼叫，然後再呼叫 _next_ 委派，將控制項傳遞至下個註冊的事件處理常式。 如果處理常式未呼叫下個委派，則不會呼叫任何後續的事件處理常式，事件會產生[最少運算路由](~/v4sdk/bot-builder-concept-middleware.md#short-circuiting)，而且介面卡不會傳送回應給通道。

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [中介軟體](~/v4sdk/bot-builder-concept-middleware.md)
