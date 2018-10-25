---
title: 將媒體附件新增至訊息 | Microsoft Docs
description: 了解如何使用適用於 .NET 的 Bot 建立器 SDK 將媒體附件新增至訊息。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 8fd9b181676eee24b1e9c64c79663d0d0ac8abfa
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/24/2018
ms.locfileid: "49997315"
---
# <a name="add-media-attachments-to-messages"></a>將媒體附件新增至訊息

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-add-media-attachments.md)
> - [Node.js](../nodejs/bot-builder-nodejs-send-receive-attachments.md)
> - [REST](../rest-api/bot-framework-rest-connector-add-media-attachments.md)

使用者與 Bot 之間的訊息交換可以包含媒體附件 (例如影像、視訊、音訊、檔案等)。 <a href="https://docs.botframework.com/en-us/csharp/builder/sdkreference/dc/d2f/class_microsoft_1_1_bot_1_1_connector_1_1_activity.html" target="_blank">活動</a>物件的 `Attachments` 屬性包含<a href="https://docs.microsoft.com/en-us/dotnet/api/microsoft.bot.connector.attachments?view=botconnector-3.12.2.4" target="_blank">附件</a>物件的陣列，代表訊息的媒體附件和內含的複合式資訊卡 (Rich Card)。 

> [!NOTE]
> [將複合式資訊卡 (Rich Card) 新增至訊息](bot-builder-dotnet-add-rich-card-attachments.md)。

## <a name="add-a-media-attachment"></a>新增媒體附件  

若要將媒體附件新增至訊息，請建立 `message` 活動的 `Attachment` 物件，並設定 `ContentType`、`ContentUrl` 和 `Name` 屬性。 

[!code-csharp[Add media attachment](../includes/code/dotnet-add-attachments.cs#addMediaAttachment)]

如果附件是影像、音訊或視訊，則連接器服務會以可讓[通道](bot-builder-dotnet-channeldata.md)在對話內轉譯該附件的方式，將附件資料傳輸至通道。 如果附件是檔案，檔案 URL 將會轉譯為對話內的超連結。

## <a name="additional-resources"></a>其他資源

- [使用頻道偵測器來預覽功能][inspector]
- [活動概觀](bot-builder-dotnet-activities.md)
- [建立訊息](bot-builder-dotnet-create-messages.md)
- [將複合式資訊卡 (Rich Card) 新增至訊息](bot-builder-dotnet-add-rich-card-attachments.md)
- <a href="https://docs.botframework.com/en-us/csharp/builder/sdkreference/dc/d2f/class_microsoft_1_1_bot_1_1_connector_1_1_activity.html" target="_blank">活動類別</a>
- <a href="https://docs.microsoft.com/en-us/dotnet/api/microsoft.bot.connector.attachments?view=botconnector-3.12.2.4" target="_blank">附件類別</a>

[inspector]: ../bot-service-channel-inspector.md


