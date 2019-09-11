---
title: 使用適用於 .NET 的 Bot Framework SDK 建立訊息 | Microsoft Docs
description: 了解適用於 .NET 的 Bot Framework SDK 內的常用訊息屬性。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: bb75e49e50a479e0141000ef49d75559148fe43a
ms.sourcegitcommit: a6d02ec4738e7fc90b7108934740e9077667f3c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/04/2019
ms.locfileid: "70297306"
---
# <a name="create-messages"></a>建立訊息

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

您的 Bot 會傳送**訊息**[活動](bot-builder-dotnet-activities.md) 以傳達資訊給使用者，同樣也會接收來自使用者的**訊息**活動。 某些訊息可能只包含純文字，而其他訊息可能包含更豐富的內容，例如[要讀出的文字](bot-builder-dotnet-text-to-speech.md)、[建議的動作](bot-builder-dotnet-add-suggested-actions.md)、[媒體附件](bot-builder-dotnet-add-media-attachments.md)、[複合式資訊卡 (Rich Card)](bot-builder-dotnet-add-rich-card-attachments.md)，和[通道特定資料](bot-builder-dotnet-channeldata.md)。 

本文說明部分的常用訊息屬性。

## <a name="customizing-a-message"></a>自訂訊息

若要更充分地掌控訊息的文字格式設定，您可以先使用 [Activity](https://docs.botframework.com/csharp/builder/sdkreference/dc/d2f/class_microsoft_1_1_bot_1_1_connector_1_1_activity.html) 物件建立自訂訊息並設定所需屬性，再將此物件傳送給使用者。

下列範例說明如何建立自訂 `message` 物件，並設定 `Text`、`TextFormat` 和 `Local` 屬性。

[!code-csharp[Set message properties](../includes/code/dotnet-create-messages.cs#setBasicProperties)]

訊息的 `TextFormat` 屬性可用來指定文字格式。 `TextFormat` 屬性可以設定為 **plain**、**markdown** 或 **xml**。 `TextFormat` 的預設值為 **markdown**。 

## <a name="attachments"></a>附件

訊息物件的 `Attachments` 屬性可用來傳送及接收簡單媒體附件 (影像、音訊、視訊、檔案) 和複合式資訊卡 (Rich Card)。 如需詳細資訊，請參閱[將媒體附件新增至訊息](bot-builder-dotnet-add-media-attachments.md)和[將複合式資訊卡 (Rich Card) 新增至訊息](bot-builder-dotnet-add-rich-card-attachments.md)。

## <a name="entities"></a>實體

訊息的 `Entities` 屬性是開放式 <a href="http://schema.org/" target="_blank">schema.org</a> 物件的陣列，其允許交換通道與 Bot 之間的通用內容中繼資料。

### <a name="mention-entities"></a>提及實體

許多通道都支援 Bot 或使用者在交談內容中「提及」某人的功能。 若要在訊息中提及使用者，請以 `Mention` 物件填入訊息的 `Entities` 屬性。 `Mention` 物件包含下列屬性： 

| 屬性 | 說明 | 
|----|----|
| 類型 | 實體的類型 ("mention") | 
| 提及我 | `ChannelAccount` 物件，指出所提及的使用者 | 
| 文字 | `Activity.Text` 屬性中的文字，表示提及本身 (可以是空白或 null) |

此程式碼範例示範如何將 `Mention` 實體新增至 `Entities` 集合。

[!code-csharp[set Mention](../includes/code/dotnet-create-messages.cs#setMention)]

> [!TIP]
> 嘗試判斷使用者意圖時，Bot 可以忽略提到它的訊息部分。 呼叫 `GetMentions` 方法，並評估 `Mention` 回應中傳回的物件。

### <a name="place-objects"></a>放置物件

以 `Place` 物件或 `GeoCoordinates` 物件填入訊息的 `Entities` 屬性，即可在訊息中傳達<a href="https://schema.org/Place" target="_blank">位置相關資訊</a>。 

`Place` 物件包含下列屬性：

| 屬性 | 說明 | 
|----|----|
| 類型 | 實體的類型 ("Place") |
| 位址 | 說明或 `PostalAddress` 物件 (未來) | 
| 地理區域 | GeoCoordinates | 
| HasMap | 地圖或 `Map` 物件的 URL (未來) |
| Name | 位置的名稱 |

`GeoCoordinates` 物件包含下列屬性：

| 屬性 | 說明 | 
|----|----|
| 類型 | 實體的類型 ("GeoCoordinates") |
| Name | 位置的名稱 |
| 經度 | 位置的經度 (<a href="https://en.wikipedia.org/wiki/World_Geodetic_System" target="_blank">WGS 84</a>) | 
| 緯度 | 位置的緯度 (<a href="https://en.wikipedia.org/wiki/World_Geodetic_System" target="_blank">WGS 84</a>) | 
| Elevation | 位置的提升 (<a href="https://en.wikipedia.org/wiki/World_Geodetic_System" target="_blank">WGS 84</a>) | 

此程式碼範例示範如何將 `Place` 實體新增至 `Entities` 集合：

[!code-csharp[set GeoCoordinates](../includes/code/dotnet-create-messages.cs#setGeoCoord)]

### <a name="consume-entities"></a>取用實體

若要取用實體，請使用 `dynamic` 關鍵字或強型別類別。

此程式碼範例示範如何使用 `dynamic` 關鍵字來處理訊息 `Entities` 屬性內的實體：

[!code-csharp[examine entity using dynamic keyword](../includes/code/dotnet-create-messages.cs#examineEntity1)]

此程式碼範例示範如何使用強型別類別來處理訊息 `Entities` 屬性內的實體：

[!code-csharp[examine entity using typed class](../includes/code/dotnet-create-messages.cs#examineEntity2)]

## <a name="channel-data"></a>通道資料

訊息活動的 `ChannelData` 屬性可以用來實作通道特有功能。 如需詳細資訊，請參閱[實作通道特有功能](bot-builder-dotnet-channeldata.md)。

## <a name="text-to-speech"></a>將文字轉換成語音

訊息活動的 `Speak` 屬性可用來指定將由 Bot 在啟用語音的通道上讀出的文字。 訊息活動的 `InputHint` 屬性可用來控制用戶端的麥克風和輸入方塊 (如果有的話) 狀態。 如需詳細資訊，請參閱[將語音新增至訊息](bot-builder-dotnet-text-to-speech.md)。

## <a name="suggested-actions"></a>建議的動作

訊息活動的 `SuggestedActions` 屬性可用來呈現使用者可點選以提供輸入的按鈕。 不同於複合式資訊卡 (Rich Card) 中出現的按鈕 (即使在點選之後，使用者仍可看見並可存取)，出現在建議動作窗格內的按鈕會在使用者進行選取後消失。 如需詳細資訊，請參閱[將建議的動作新增至訊息](bot-builder-dotnet-add-suggested-actions.md)。

## <a name="next-steps"></a>後續步驟

Bot 和使用者可以互相傳送訊息。 如果訊息較為複雜，Bot 可在訊息中將複合式資訊卡 (Rich Card) 傳送給使用者。 複合式資訊卡 (Rich Card) 涵蓋大部分 Bot 常需要的許多簡報與互動案例。

> [!div class="nextstepaction"]
> [在訊息中傳送複合式資訊卡 (Rich Card)](bot-builder-dotnet-add-rich-card-attachments.md)

## <a name="additional-resources"></a>其他資源

- [活動概觀](bot-builder-dotnet-activities.md)
- [傳送及接收活動](bot-builder-dotnet-connector.md)
- [將媒體附件新增至訊息](bot-builder-dotnet-add-media-attachments.md)
- [將複合式資訊卡 (Rich Card) 新增至訊息](bot-builder-dotnet-add-rich-card-attachments.md)
- [將語音新增至訊息](bot-builder-dotnet-text-to-speech.md)
- [將建議的動作新增至訊息](bot-builder-dotnet-add-suggested-actions.md)
- [實作通道特有功能](bot-builder-dotnet-channeldata.md)
- <a href="https://docs.botframework.com/csharp/builder/sdkreference/dc/d2f/class_microsoft_1_1_bot_1_1_connector_1_1_activity.html" target="_blank">Activity 類別</a>
- <a href="/dotnet/api/microsoft.bot.connector.imessageactivity" target="_blank">IMessageActivity 介面</a>

