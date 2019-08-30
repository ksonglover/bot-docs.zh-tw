---
title: 使用 Bot Connector API 建立訊息 | Microsoft Docs
description: 了解 Bot Connector API 內的常用訊息屬性。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
ms.openlocfilehash: 3f8badd90c3be5191e24556d5f7fde66cc72fd6b
ms.sourcegitcommit: c200cc2db62dbb46c2a089fb76017cc55bdf26b0
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2019
ms.locfileid: "70037367"
---
# <a name="create-messages"></a>建立訊息

Bot 會傳送**訊息**類型的 [Activity][] 物件以傳達資訊給使用者，此外，也同樣會接收來自使用者的**訊息**活動。 某些訊息可能只包含純文字，而其他訊息可能包含更豐富的內容，例如[要讀出的文字](bot-framework-rest-connector-text-to-speech.md)、[建議的動作](bot-framework-rest-connector-add-suggested-actions.md)、[媒體附件](bot-framework-rest-connector-add-media-attachments.md)、[複合式資訊卡 (Rich Card)](bot-framework-rest-connector-add-rich-cards.md) 和[通道特有資料](bot-framework-rest-connector-channeldata.md)。 本文說明部分的常用訊息屬性。

## <a name="message-text-and-formatting"></a>簡訊和格式設定

使用 **plain**、**markdown** 或 **xml** 可以將訊息格式化。 `textFormat` 屬性的預設格式為 **markdown**，以及使用 Markdown 格式設定標準來解譯文字。 文字格式支援的層級因通道而異。 若要查看設為目標的通道是否支援您要使用的功能，請使用[通道偵測器][ChannelInspector]來預覽此功能。 

[Activity][] 物件的 `textFormat` 屬性可用來指定文字的格式。 例如，若要建立只包含純文字的基本訊息，請將 `Activity` 物件的 `textFormat` 屬性設定為 **plain**，將 `text` 屬性設定為訊息的內容，並將 `locale` 屬性設定為寄件者的地區設定。 

## <a name="attachments"></a>附件

[Activity][] 物件的 `attachments` 屬性可用來傳送簡單媒體附件 (影像、音訊、視訊、檔案) 和複合式資訊卡 (Rich Card)。 如需詳細資訊，請參閱[將媒體附件新增至訊息](bot-framework-rest-connector-add-media-attachments.md)和[將複合式資訊卡 (Rich Card) 新增至訊息](bot-framework-rest-connector-add-rich-cards.md)。

## <a name="entities"></a>實體

[Activity][] 物件的 `entities` 屬性是開放式 <a href="http://schema.org/" target="_blank">schema.org</a> 物件的陣列，允許交換通道與 Bot 之間的通用內容中繼資料。

### <a name="mention-entities"></a>提及實體

許多通道都支援 Bot 或使用者在交談內容中「提及」某人的功能。 若要在訊息中提及使用者，請以 [Mention][] 物件填入訊息的 `entities` 屬性。 

### <a name="place-entities"></a>放置實體

若要在訊息中傳達<a href="https://schema.org/Place" target="_blank">位置相關資訊</a>，請以 [Place][] 物件填入訊息的 `entities` 屬性。 

## <a name="channel-data"></a>通道資料

[Activity][] 物件的 `channelData` 屬性可用來實作通道特有功能。 如需詳細資訊，請參閱[實作通道特有功能](bot-framework-rest-connector-channeldata.md)。

## <a name="text-to-speech"></a>將文字轉換成語音

[Activity][] 物件的 `speak` 屬性可用來指定將由 Bot 在已啟用語音的通道上讀出的文字，而 `inputHint` 物件的 `Activity` 屬性可用來影響用戶端的麥克風狀態。 如需詳細資訊，請參閱[將語音新增至訊息](bot-framework-rest-connector-text-to-speech.md)和[將輸入提示新增至訊息](bot-framework-rest-connector-add-input-hints.md)。

## <a name="suggested-actions"></a>建議動作

[Activity][] 物件的 `suggestedActions` 屬性可用來呈現使用者可點選以提供輸入的按鈕。 不同於複合式資訊卡 (Rich Card) 中出現的按鈕 (即使在點選之後，使用者仍可看見並可存取)，出現在建議動作窗格內的按鈕會在使用者進行選取後消失。 如需詳細資訊，請參閱[將建議的動作新增至訊息](bot-framework-rest-connector-add-suggested-actions.md)。

## <a name="additional-resources"></a>其他資源

- [使用通道偵測器預覽功能][ChannelInspector]
- [活動概觀](bot-framework-rest-connector-activities.md)
- [傳送及接收訊息](bot-framework-rest-connector-send-and-receive-messages.md)
- [將媒體附件新增至訊息](bot-framework-rest-connector-add-media-attachments.md)
- [將複合式資訊卡 (Rich Card) 新增至訊息](bot-framework-rest-connector-add-rich-cards.md)
- [將語音新增至訊息](bot-framework-rest-connector-text-to-speech.md)
- [將輸入提示新增至訊息](bot-framework-rest-connector-add-input-hints.md)
- [將建議的動作新增至訊息](bot-framework-rest-connector-add-suggested-actions.md)
- [實作通道特有功能](bot-framework-rest-connector-channeldata.md)

[ChannelInspector]: ../bot-service-channel-inspector.md
[textFormating]: ../bot-service-channel-inspector.md#text-formatting

[Activity]: bot-framework-rest-connector-api-reference.md#activity-object
[Mention]: bot-framework-rest-connector-api-reference.md#mention-object
[Place]: bot-framework-rest-connector-api-reference.md#place-object
