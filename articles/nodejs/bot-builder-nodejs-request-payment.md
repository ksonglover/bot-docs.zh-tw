---
title: 要求付款 | Microsoft Docs
description: 了解如何使用適用於 Node.js 的 Bot Framework SDK 傳送付款要求。
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 296004c654cfd59de6c245bf9702a80024526140
ms.sourcegitcommit: b15cf37afc4f57d13ca6636d4227433809562f8b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/11/2019
ms.locfileid: "54225743"
---
# <a name="request-payment"></a>要求付款

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-request-payment.md)
> - [Node.js](../nodejs/bot-builder-nodejs-request-payment.md)

如果 Bot 要讓使用者能購買商品，則可在[複合式資訊卡 (Rich Card)](bot-builder-nodejs-send-rich-cards.md) 中包含一個特別的按鈕，藉此要求付款。 本文說明如何使用適用於 Node.js 的 Bot Framework SDK 傳送付款要求。

## <a name="prerequisites"></a>必要條件

您必須先完成這些必要工作，才可以使用適用於 Node.js 的 Bot Framework SDK 傳送付款要求。

### <a name="register-and-configure-your-bot"></a>註冊並設定 Bot

將 `MicrosoftAppId` 和 `MicrosoftAppPassword` 的 Bot 環境變數，更新為在[註冊](~/bot-service-quickstart-registration.md)過程中針對 Bot 所產生的應用程式識別碼和密碼值。 

> [!NOTE]
> 若要尋找您 Bot 的 **AppID** 和 **AppPassword**，請參閱 [MicrosoftAppID 和 MicrosoftAppPassword](~/bot-service-manage-overview.md#microsoftappid-and-microsoftapppassword)。

### <a name="create-and-configure-merchant-account"></a>建立及設定商家帳戶

1. <a href="https://dashboard.stripe.com/register" target="_blank">如果您還沒有 Stripe 帳戶，請建立並啟用一個帳戶。</a>

2. <a href="https://seller.microsoft.com/en-us/dashboard/registration/seller/?accountprogram=botframework" target="_blank">使用您的 Microsoft 帳戶登入賣方中心。</a>

3. 在賣方中心內，連結您的帳戶與 Stripe。

4. 在賣方中心內，瀏覽至儀表板，並複製 **MerchantID** 的值。

5. 將 `PAYMENTS_MERCHANT_ID` 環境變數更新為您從賣方中心儀表板複製的值。 

[!INCLUDE [Payment process overview](../includes/snippet-payment-process-overview.md)]

## <a name="payment-bot-sample"></a>付款 Bot 範例

<a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/sample-payments" target="_blank">付款 Bot</a> 範例提供使用 Node.js 傳送付款要求的 Bot 範例。 若要查看作用中的這個範例 Bot，您可以<a href="https://webchat.botframework.com/embed/paymentsample?s=d39Bk7JOMzQ.cwA.Rig.dumLki9bs3uqfWFMjXPn5PFnQVmT2VAVR1Zl1iPi07k" target="_blank">在網路聊天中試用</a>、<a href="https://join.skype.com/bot/9fbc0f17-43eb-40fe-bf3b-af151e6ce45e" target="_blank">將它新增為 Skype 連絡人</a>，或下載付款 Bot 範例並使用 Bot Framework Emulator 在本機執行。 

> [!NOTE]
> 若要在網路聊天或 Skype 中使用**付款 Bot** 範例完成端對端付款程序，您必須在您的 Microsoft 帳戶中指定有效的信用卡或金融卡 (亦即，美國發卡機構簽發的有效卡片)。 我們不會向您的卡片收取費用，也不會驗證卡片的 CVV (信用卡檢查碼)，因為**付款 Bot** 範例是在測試模式中執行 (也就是，`PAYMENTS_LIVEMODE` 在 **.env** 中設定為 `false`)。

本文的後續幾節會在**付款 Bot** 範例的內容中，說明付款程序的三個部分。

## <a id="request-payment"></a> 要求付款

Bot 可藉由傳送一則包含[複合式資訊卡 (Rich Card)](bot-builder-nodejs-send-rich-cards.md) 的訊息來向使用者要求付款，該卡片中有一個指定 `type` 為「付款」的按鈕。 **付款 Bot**範例中的這個程式碼片段會建立一則包含 Hero 卡的訊息，該卡片中有一個 [購買] 按鈕，而使用者可以按一下 (或點選) 此按鈕來起始付款程序。 

[!code-javascript[Request payment](../includes/code/node-request-payment.js#requestPayment)]

在此範例中，按鈕的類型會指定為 `payments.PaymentActionType`，而應用程式將其定義為「付款」。 此按鈕的值會由 `createPaymentRequest` 函式填入，該函式會傳回 `PaymentRequest` 物件，其中包含支援的付款方式、詳細資料和選項相關資訊。 如需有關實作詳細資料的詳細資訊，請參閱<a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/sample-payments" target="_blank">付款 Bot</a> 範例中的 **app.js**。

此螢幕擷取畫面會顯示由上述程式碼片段所產生的 Hero 卡 (具有 [購買] 按鈕)。 
 
![付款範例 Bot](../media/payments-bot-buy.png) 

> [!IMPORTANT]
> 只要使用者可存取 [購買] 按鈕，就可以透過該按鈕起始付款程序。 在群組對話的內容中，不可能指定一個按鈕，僅供特定使用者使用。 

## <a id="user-experience"></a> 使用者體驗

當使用者按一下 [購買] 按鈕時，系統會將他或她導向至付款 Web 體驗，以透過其 Microsoft 帳戶提供所有必要的付款、交貨和連絡人資訊。 

![Microsoft 付款](../media/microsoft-payment.png)

### <a name="http-callbacks"></a>HTTP 回呼

HTTP 回呼將傳送至 Bot，指出它應該執行特定作業。 每個回呼都是包含下列屬性值的事件： 

| 屬性 | 值 |
|----|----|
| `type` | 叫用 | 
| `name` | 指出 Bot 應該執行的作業類型 (例如：交貨地址更新、交貨選項更新、付款完成)。 | 
| `value` | 採用 JSON 格式的要求承載。 | 
| `relatesTo` |  說明與付款要求相關聯的通道和使用者。 | 

> [!NOTE]
> `invoke` 是保留給 Microsoft Bot Framework 使用的特別事件類型。 `invoke` 事件的傳送者預期 Bot 會藉由傳送 HTTP 回應來認知回呼。

## <a id="process-callbacks"></a> 處理回呼

[!INCLUDE [Process callbacks overview](../includes/snippet-payment-process-callbacks-overview.md)]

### <a name="shipping-address-update-and-shipping-option-update-callbacks"></a>交貨地址更新和交貨選項更新回呼

接收「交貨地址更新」或「交貨選項更新」回呼，用戶端會在事件的 `value` 屬性中將目前付款狀態的詳細資料提供給 Bot。
如果您是商家，則應將這些回呼視為靜態，根據輸入付款詳細資料，將會計算某些輸出付款詳細資料，而如果用戶端所提供的輸入狀態因任何理由而無效，則回呼會失敗。 
如果 Bot 判斷指定的現況資訊有效，則只要傳送 HTTP 狀態碼 `200 OK` 以及未經修改的付款詳細資料。 或者，Bot 可以先傳送 HTTP 狀態碼 `200 OK` 以及應套用的更新後付款詳細資料，才能處理訂單。 在某些情況下，Bot 可能會判斷更新後的資訊無效，所以無法照現況處理訂單。 例如，使用者的交貨地址可能會指定產品供應商不出貨的國家/地區。 在此情況下，Bot 可以傳送 HTTP 狀態碼 `200 OK` 和一則訊息，其中填入付款詳細資料物件的錯誤屬性。 傳送任何 `400` 至 `500` 範圍內的 HTTP 狀態碼會導致客戶發生一般錯誤。

### <a name="payment-complete-callbacks"></a>付款完成回呼

接收「付款完成」回呼時，Bot 會在物件的 `value` 屬性中取得一份未經修改的最初付款要求，以及付款回應事件。 付款回應物件將包含客戶最終選取的項目以及付款代碼 (Token)。 Bot 應藉此機會根據最初付款要求和客戶最終選取的項目，重新計算最終付款要求。 假設系統將客戶的選取項目判定為有效，則 Bot 應確認付款權杖標頭中的金額和貨幣，確保其符合最終付款要求。  如果 Bot 決定向客戶收費，則只應收取付款權杖標頭中的金額，因為這是客戶所確認的價格。 如果 Bot 預期的值和在「付款完成」回呼中收到的值不相符，則可傳送 HTTP 狀態碼 `200 OK` 並將結果欄位設定為 `failure`，藉此讓付款要求失敗。   

除了驗證付款詳細資料以外，Bot 也應該先確認可以履行訂單，再起始付款處理。 例如，其可驗證所購買的項目是否仍可在庫存中取得。 如果值正確無誤，而且付款處理器已順利向付款權杖收取費用，則 Bot 應該以 HTTP 狀態碼 `200 OK` 回應並將結果欄位設為 `success`，以便付款 Web 體驗顯示付款確認。 Bot 收到的付款權杖只能由要求付款的商家使用一次，而且必須提交至 Stripe (Bot Framework 目前唯一支援的付款處理器)。 傳送任何 `400` 至 `500` 範圍內的 HTTP 狀態碼會導致客戶發生一般錯誤。

**付款 Bot** 範例中的這個程式碼片段會處理 Bot 所接收的回呼。 

[!code-javascript[Request payment](../includes/code/node-request-payment.js#processCallback)]

在此範例中，Bot 會檢查內送事件的 `name` 屬性，以識別它需要執行的作業類型，然後呼叫適當的方法來處理回呼。 如需有關實作詳細資料的詳細資訊，請參閱<a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/sample-payments" target="_blank">付款 Bot</a> 範例中的 **app.js**。

## <a name="testing-a-payment-bot"></a>測試付款 Bot

[!INCLUDE [Test a payment bot](../includes/snippet-payment-test-bot.md)]

在<a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/sample-payments" target="_blank">付款 Bot</a> 範例中，**.env** 中的 `PAYMENTS_LIVEMODE` 環境變數可決定「付款完成」回呼將包含模擬的付款代碼或實際的付款代碼。 如果 `PAYMENTS_LIVEMODE` 設為 `false`，則標頭會新增至 Bot 的輸出付款要求，以指出 Bot 處於測試模式，而且「付款完成」回呼將包含無法收費的模擬付款代碼。 如果 `PAYMENTS_LIVEMODE` 設為 `true`，則指出 Bot 處於測試模式的標頭會從 Bot 的輸出付款要求中省略，而且「付款完成」回呼會包含 Bot 將提交至 Stripe 進行付款處理的實際付款權杖。 這會是向指定的付款工具收費的真實交易。 

## <a name="additional-resources"></a>其他資源

- <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/sample-payments" target="_blank">付款 Bot 範例</a>
- [將複合式資訊卡 (Rich Card) 附件新增至訊息](bot-builder-nodejs-send-rich-cards.md)
- <a href="http://www.w3.org/Payments/" target="_blank">在 W3C 進行網路付款</a> 
