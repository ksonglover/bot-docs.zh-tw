---
title: 從訊息內容辨識意圖 | Microsoft Docs
description: 了解如何使用規則運算式，或藉由檢查訊息內容來辨識使用者的意圖。
author: DeniseMak
ms.author: v-demak
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: e308445a43507db94fe54735432790dabdb88731
ms.sourcegitcommit: 23a1808e18176f1704f2f6f2763ace872b1388ae
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/25/2019
ms.locfileid: "67404855"
---
# <a name="recognize-user-intent-from-message-content"></a>從訊息內容辨識使用者意圖

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

當 Bot 收到來自使用者的訊息時，Bot 可以使用**辨識器**來檢查訊息並判斷意圖。 此意圖提供一個可叫用的對應 (從訊息到對話方塊)。 本文說明如何使用規則運算式或藉由檢查訊息內容來辨識意圖。 例如，Bot 可以使用規則運算式來檢查訊息是否包含 "help" 這個字，並叫用 [說明] 對話方塊。 Bot 也可以檢查使用者訊息的屬性，查看使用者已傳送映像而非文字，然後叫用映像處理對話方塊。 

> [!NOTE]
> 如需使用 LUIS 辨識意圖的相關資訊，請參閱[使用 LUIS 辨識意圖和實體](bot-builder-nodejs-recognize-intent-luis.md) 


## <a name="use-the-built-in-regular-expression-recognizer"></a>使用內建規則運算式辨識器
使用規則運算式 [RegExpRecognizer][RegExpRecognizer] 來偵測使用者的目的。 您可以將多個運算式傳遞至辨識器，以支援多種語言。 

> [!TIP]
> 請參閱[支援當地語系化](bot-builder-nodejs-localization.md)，以取得將 Bot 當地語系化的詳細資訊。

下列程式碼會建立名為 `CancelIntent` 的規則運算式辨識器並將它新增至 Bot。 此範例中的辨識器提供 `en_us` 和 `ja_jp` 地區設定的規則運算式。 

[!code-js[Add a regular expression recognizer (JavaScript)](../includes/code/node-regex-recognizer.js#addRegexRecognizer)]

將辨識器新增至 Bot 後，請將 [triggerAction][triggerAction] 附加到您希望 Bot 在辨識器偵測到意圖時叫用的對話方塊。 使用 [matches][matches] 選項來指定意圖名稱，如下列程式碼所示：

[!code-js[Map the CancelIntent recognizer to a cancel dialog (JavaScript)](../includes/code/node-regex-recognizer.js#bindCancelDialogToRegexRecognizer)]

意圖辨識器是全域的，這表示辨識器會針對從使用者接收的每則訊息執行。 如果辨識器使用 `triggerAction` 偵測到繫結至對話方塊的意圖，則可以觸發中斷目前使用中的對話方塊。 允許並處理中斷是一項有彈性的設計，該設計會考量使用者真正執行的動作。

> [!TIP] 
> 若要了解 `triggerAction` 如何搭配對話方塊運作，請參閱[管理交談流程](bot-builder-nodejs-manage-conversation-flow.md)。 若要深入了解您可與已辨識意圖相關聯的各種動作，請參閱[處理使用者動作](bot-builder-nodejs-dialog-actions.md)。

## <a name="register-a-custom-intent-recognizer"></a>註冊自訂意圖辨識器
您也可以實作自訂辨識器。 這個範例會新增一個簡單辨識器，以尋找會說出 'help' 或 'goodbye' 的使用者，但您可以輕易地新增可進行更複雜處理的辨識器，例如辨識使用者何時傳送映像的辨識器。 


[!code-js[Add a custom recognizer (JavaScript)](../includes/code/node-howto-recognize-intent.js#addCustomRecognizer)]

註冊辨識器之後，您即可使用 `matches` 子句來建立的辨識器與動作關聯。

[!code-js[Bind intents to actions (JavaScript)](../includes/code/node-howto-recognize-intent.js#bindIntentsToActions)]

## <a name="disambiguate-between-multiple-intents"></a>釐清多個意圖

Bot 可以註冊一個以上的辨識器。 請注意，自訂辨識器範例涉及將數值分數指派給每個意圖。 這是因為 Bot 可能有一個以上的辨識器，而且 Bot Framework SDK 會提供內建邏輯來釐清由多個辨識器所傳回的意圖。 指派給意圖的分數通常介於 0.0 到 1.0，但自訂辨識器可能會定義大於 1.1 的意圖，以確保 Bot Framework SDK 的去除混淆邏輯一律會選擇該意圖。 

根據預設，辨識器則會以平行方式執行，但您可以在 [IIntentRecognizerSetOptions][IntentRecognizerSetOptions] 中設定 recognizeOrder，所以只要 Bot 找到可提供分數 1.0 的辨識器時，程序就會立即結束。

Bot Framework SDK 包含一個[範例][DisambiguationSample]，該範例示範如何藉由實作 [IDisambiguateRouteHandler][IDisambiguateRouteHandler] 來提供自訂去除混淆邏輯。

## <a name="next-steps"></a>後續步驟
使用規則運算式和檢查訊息內容的邏輯可能會變複雜，尤其是在 Bot 的對話流程為開放式時。 為了協助 Bot 處理來自使用者的各種文字和語音輸入，您可以使用 [LUIS][LUIS] 等意圖辨識服務，將自然語言理解新增到 Bot。

> [!div class="nextstepaction"]
> [使用 LUIS 辨識意圖和實體](bot-builder-nodejs-recognize-intent-luis.md)


[LUIS]: https://www.luis.ai/

[triggerAction]: https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.dialog.html#triggeraction

[matches]: https://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.itriggeractionoptions.html#matches

[node-js-bot-how-to]: bot-builder-nodejs-recognize-intent-luis.md

[LUISAzureDocs]: /azure/cognitive-services/LUIS/Home

[IMessage]: http://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.imessage

[IntentRecognizerSetOptions]: https://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.iintentrecognizersetoptions.html

[LuisRecognizer]: https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.luisrecognizer

[LUISSample]: https://aka.ms/v3-js-luisSample

[LUISConcepts]: https://docs.botframework.com/node/builder/guides/understanding-natural-language/

[DisambiguationSample]: https://aka.ms/v3-js-onDisambiguateRoute

[IDisambiguateRouteHandler]: https://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.idisambiguateroutehandler.html

[RegExpRecognizer]: https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.regexprecognizer.html

[AlarmBot]: https://aka.ms/v3-js-luisSample
