---
title: 將複合式資訊卡 (Rich Card) 附件新增至訊息 | Microsoft Docs
description: 了解如何使用適用於 .NET 的 Bot 建立器 SDK 將複合式資訊卡 (Rich Card) 新增至訊息。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 9eb07a4ac63816b84830956bca0c3a3910669e0d
ms.sourcegitcommit: 67445b42796d90661afc643c6bb6533e9a662cbc
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/06/2018
ms.locfileid: "39574534"
---
# <a name="add-rich-card-attachments-to-messages"></a>將複合式資訊卡 (Rich Card) 附件新增至訊息

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-add-rich-card-attachments.md)
> - [Node.js](../nodejs/bot-builder-nodejs-send-rich-cards.md)
> - [REST](../rest-api/bot-framework-rest-connector-add-rich-cards.md)

使用者與 Bot 之間的訊息交換，可以包含一或多個轉譯為清單或浮動切換的複合式資訊卡。 <a href="https://docs.botframework.com/en-us/csharp/builder/sdkreference/dc/d2f/class_microsoft_1_1_bot_1_1_connector_1_1_activity.html" target="_blank">活動</a>物件的 `Attachments` 屬性包含<a href="https://docs.microsoft.com/en-us/dotnet/api/microsoft.bot.connector.attachments?view=botconnector-3.12.2.4" target="_blank">附件</a>物件的陣列，代表訊息內的複合式資訊卡 (Rich Card) 和媒體附件。 

> [!NOTE]
> 如需如何將媒體附件新增至訊息的相關資訊，請參閱[將媒體附件新增至訊息](bot-builder-dotnet-add-media-attachments.md)。

## <a name="types-of-rich-cards"></a>複合式資訊卡 (Rich Card) 的類型

Bot Framework 目前支援八種類型的複合式資訊卡 (Rich Card)： 

| 卡片類型 | 說明 |
|----|----|
| <a href="/adaptive-cards/get-started/bots">調適型卡片</a> | 可包含文字、語音、影像、按鈕和輸入欄位之任意組合的可自訂卡片。 請參閱[個別頻道支援](/adaptive-cards/get-started/bots#channel-status)。  |
| [動畫卡片][animationCard] | 可播放動畫 GIF 或短片的卡片。 |
| [音訊卡片][audioCard] | 可以播放音訊檔案的卡片。 |
| [主圖卡片][heroCard] | 通常包含單一大型影像、一或多個按鈕和文字的卡片。 |
| [縮圖卡片][thumbnailCard] | 通常包含單一縮圖影像、一或多個按鈕和文字的卡片。 |
| [收據卡片][receiptCard] | 讓 Bot 向使用者提供收據的卡片。 其中通常包含收據上的項目清單 (稅金和總金額資訊) 和其他文字。 |
| [登入卡][signinCard] | 可讓 Bot 要求使用者登入的卡片。 通常包含文字，以及一或多個使用者可以按一下以起始登入程序的按鈕。 |
| [視訊卡片][videoCard] | 可播放視訊的卡片。 |

> [!TIP]
> 若要以清單格式顯示多個複合式資訊卡 (Rich Card)，請將活動的 `AttachmentLayout` 的屬性設定為 [清單]。 若要以浮動切換格式顯示多個複合式資訊卡 (Rich Card)，請將活動的 `AttachmentLayout` 的屬性設定為 [浮動切換]。 如果通道不支援浮動切換格式，則會以清單格式顯示複合式資訊卡 (Rich Card)，即使 `AttachmentLayout` 屬性指定 [浮動切換] 亦然。

## <a name="process-events-within-rich-cards"></a>處理複合式資訊卡 (Rich Card) 內的事件

若要處理複合式資訊卡 (Rich Card) 內的事件，請定義 `CardAction` 物件，指定在使用者按一下按鈕或點選卡片的某區段時所應發生的情況。 每個 `CardAction` 物件都包含下列屬性：

| 屬性 | 類型 | 說明 | 
|----|----|----|
| 類型 | 字串 | 動作的類型 (下表中指定的其中一個值) |
| 標題 | 字串 | 按鈕的標題 |
| 映像 | 字串 | 按鈕的影像 URL |
| 值 | 字串 | 執行指定動作類型所需的值 |

> [!NOTE]
> 調適型卡片內的按鈕不會使用 `CardAction` 物件來建立，而會改用<a href="http://adaptivecards.io" target="_blank">調適型卡片</a>所定義的結構描述來建立。 如需透過範例了解如何將按鈕新增至調適型卡片，請參閱[將調適型卡片新增至訊息](#adaptive-card)。

下表列出 `CardAction.Type` 的有效值，並說明每個類型的 `CardAction.Value` 應有的預期內容：

| CardAction.Type | CardAction.Value | 
|----|----|
| openUrl | 要在內建瀏覽器中開啟的 URL |
| imBack | 要傳送至 Bot 的訊息文字 (來自於按一下按鈕或點選卡片的使用者)。 此訊息 (從使用者到 Bot) 會透過裝載對話的用戶端應用程式顯示給所有對話參與者。 |
| postBack | 要傳送至 Bot 的訊息文字 (來自於按一下按鈕或點選卡片的使用者)。 某些用戶端應用程式可能會在訊息摘要中顯示此文字，讓所有對話參與者都能看見。 |
| call | 以下列格式撥打電話的目的地：**tel:123123123123** |
| playAudio | 待播放音訊的 URL |
| playVideo | 待播放視訊的 URL |
| showImage | 待顯示影像的 URL |
| downloadFile | 待下載檔案的 URL |
| signin | 待起始 OAuth 流程的 URL |

## <a name="add-a-hero-card-to-a-message"></a>將主圖卡新增至訊息

主圖卡通常包含單一大型影像、一或多個按鈕和文字。 

下列程式碼範例說明如何建立回覆訊息，且其中須包含三個以浮動切換格式轉譯的主圖卡： 

[!code-csharp[Add HeroCard attachment](../includes/code/dotnet-add-attachments.cs#addHeroCardAttachment)]

## <a name="add-a-thumbnail-card-to-a-message"></a>將縮圖卡新增至訊息

縮圖卡通常包含單一縮圖影像、一或多個按鈕和文字。 

下列程式碼範例說明如何建立回覆訊息，且其中須包含兩個以清單格式轉譯的縮圖卡： 

[!code-csharp[Add ThumbnailCard attachment](../includes/code/dotnet-add-attachments.cs#addThumbnailCardAttachment)]

## <a name="add-a-receipt-card-to-a-message"></a>將收據卡新增至訊息

收據卡可讓 Bot 向使用者提供收據。 其中通常包含收據上的項目清單 (稅金和總金額資訊) 和其他文字。 

下列程式碼範例說明如何建立包含收據卡的回覆訊息： 

[!code-csharp[Add ReceiptCard attachment](../includes/code/dotnet-add-attachments.cs#addReceiptCardAttachment)]

## <a name="add-a-sign-in-card-to-a-message"></a>將登入卡新增至訊息

登入卡可讓 Bot 要求使用者登入。 其中通常包含文字，以及一或多個可讓使用者點選以起始登入程序的按鈕。 

下列程式碼範例說明如何建立包含登入卡的回覆訊息：

[!code-csharp[Add SignInCard attachment](../includes/code/dotnet-add-attachments.cs#addSignInCardAttachment)]

## <a id="adaptive-card"></a>將調適型卡片新增至訊息

調適型卡片可包含文字、語音、影像、按鈕和輸入欄位的任意組合。 調適型卡片會使用<a href="http://adaptivecards.io" target="_blank">調適型卡片</a>中指定的 JSON 格式來建立，讓您能夠完整控制卡片的內容和格式。 

若要使用 .NET 建立調適型卡片，請安裝 `Microsoft.AdaptiveCards` NuGet 套件。 然後，請利用<a href="http://adaptivecards.io" target="_blank">調適型卡片</a>網站中的資訊了解調適型卡片的結構描述、探索調適型卡片的元素，並查看 JSON 範例，用以建立具有不同組合和複雜度的卡片。 此外，您可以使用互動式視覺化檢視，以設計調適型卡片承載，以及預覽卡片輸出。

下列程式碼範例示範如何建立包含調適型卡片作為行事曆提醒的訊息： 

[!code-csharp[Add Adaptive Card attachment](../includes/code/dotnet-add-attachments.cs#addAdaptiveCardAttachment)]

產生的卡片會包含三個文字區塊、一個輸入欄位 (選擇清單) 和三個按鈕：

![調適型卡片行事曆提醒](../media/adaptive-card-reminder.png)

## <a name="additional-resources"></a>其他資源

- [使用頻道偵測器來預覽功能][inspector]
- <a href="http://adaptivecards.io" target="_blank">調適型卡片</a>
- [活動概觀](bot-builder-dotnet-activities.md)
- [建立訊息](bot-builder-dotnet-create-messages.md)
- [將媒體附件新增至訊息](bot-builder-dotnet-add-media-attachments.md)
- <a href="https://docs.botframework.com/en-us/csharp/builder/sdkreference/dc/d2f/class_microsoft_1_1_bot_1_1_connector_1_1_activity.html" target="_blank">活動類別</a>
- <a href="https://docs.microsoft.com/en-us/dotnet/api/microsoft.bot.connector.attachments?view=botconnector-3.12.2.4" target="_blank">附件類別</a>

[animationCard]: /dotnet/api/microsoft.bot.connector.animationcard

[audioCard]: /dotnet/api/microsoft.bot.connector.audiocard 

[heroCard]: /dotnet/api/microsoft.bot.connector.herocard 

[thumbnailCard]: /dotnet/api/microsoft.bot.connector.thumbnailcard 

[receiptCard]: /dotnet/api/microsoft.bot.connector.receiptcard 

[signinCard]: /dotnet/api/microsoft.bot.connector.signincard 

[videoCard]: /dotnet/api/microsoft.bot.connector.videocard

[inspector]: ../bot-service-channel-inspector.md
