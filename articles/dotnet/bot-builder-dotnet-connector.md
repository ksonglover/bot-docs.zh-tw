---
title: 傳送及接收活動 | Microsoft Docs
description: 了解如何透過適用於 .NET 的 Bot Builder SDK 使用 Connector 服務，跨各種通道來與使用者交換資訊。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 2a5c9dcad0d9fd70caaf28ff7ac95830bd47e2d6
ms.sourcegitcommit: 67445b42796d90661afc643c6bb6533e9a662cbc
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/06/2018
ms.locfileid: "39574594"
---
# <a name="send-and-receive-activities"></a>傳送及接收活動

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

Bot Framework Connector 提供單一 REST API，讓 Bot 能夠跨多個通道 (例如 Skype、電子郵件、Slack 等) 進行通訊。 它可藉由將訊息從 Bot 轉送至通道以及從通道轉送至 Bot，讓 Bot 和使用者之間的通訊更為順暢。 

本文說明如何透過適用於 .NET 的 Bot Builder SDK 使用 Connector，在通道上的 Bot 和使用者之間交換資訊。 

> [!NOTE]
> 雖然您可以獨佔方式使用本文所述的技術來建構 Bot，但 Bot Builder SDK 會提供額外功能 (例如[對話](bot-builder-dotnet-dialogs.md)和 [FormFlow](bot-builder-dotnet-formflow.md))，可以簡化管理對話流程和狀態的程序，並且能夠以更簡單的方式來合併 Language Understanding 之類的認知服務。

## <a name="create-a-connector-client"></a>建立連接器用戶端

[ConnectorClient][ConnectorClient] 類別包含 Bot 用來與通道上的使用者進行通訊的方法。 當您的 Bot 接收到來自 Connector 的 <a href="https://docs.botframework.com/en-us/csharp/builder/sdkreference/dc/d2f/class_microsoft_1_1_bot_1_1_connector_1_1_activity.html" target="_blank">Activity</a> 物件時，它應該使用針對該活動指定的 `ServiceUrl` 來建立連接器用戶端，後續將使用它來產生回應。 

[!code-csharp[Create connector client](../includes/code/dotnet-send-and-receive.cs#createConnectorClient)]

> [!TIP]
> 由於通道的端點可能不穩定，因此，您的 Bot 應該盡可能直接與 Connector 在 `Activity` 物件中指定的端點進行通訊 (而不依賴快取的端點)。 
>
> 如果您的 Bot 需要起始對話，它可以針對指定的通道使用快取的端點 (因為在該案例中，將不會有任何傳入的 `Activity` 物件)，但它通常應該會重新整理快取的端點。 

## <a id="create-reply"></a> 建立回覆

Connector 會使用 [Activity](bot-builder-dotnet-activities.md) 物件，在 Bot 和通道 (使用者) 之間來回傳遞資訊。 每個活動都包含可用於將訊息路由傳送至適當目的地的資訊，以及有關訊息建立者 (`From` 屬性)、訊息的內容和訊息收件者 (`Recipient` 屬性) 的資訊。

當您的 Bot 接收來自 Connector 的活動時，傳入活動的 `Recipient` 屬性會在該對話中指定 Bot 的身分識別。 由於某些通道 (例如 Slack) 會在將 Bot 新增至對話時為其指派新的身分識別，因此，該 Bot 應一律使用傳入活動的 `Recipient` 屬性值作為其回應中 `From` 屬性的值。

雖然您可以自行從頭開始建立並初始化傳出的 `Activity` 物件，但 Bot Builder SDK 還是會提供更簡單的方法來建立回覆。 藉由使用傳入活動的 `CreateReply` 方法，您只需指定回應的訊息文字，而傳出的活動會使用自動填入的 `Recipient`、`From` 和 `Conversation` 屬性來建立。

[!code-csharp[Create reply](../includes/code/dotnet-send-and-receive.cs#createReply)]

## <a name="send-a-reply"></a>傳送回覆

一旦建立回覆之後，您就可以藉由呼叫連接器用戶端的 `ReplyToActivity` 方法來傳送它。 Connector 將使用適當的通道語意來傳遞回覆。 

[!code-csharp[Send reply](../includes/code/dotnet-send-and-receive.cs#sendReply)]

> [!TIP]
> 如果您的 Bot 正在回覆使用者的訊息，請一律使用 `ReplyToActivity` 方法。

## <a name="send-a-non-reply-message"></a>傳送 (非回覆) 訊息 

如果您的 Bot 是對話的一部分，它可以藉由呼叫 `SendToConversation` 方法，將不屬於直接回覆的訊息傳送給來自使用者的任何訊息。 

[!code-csharp[Send non-reply message](../includes/code/dotnet-send-and-receive.cs#sendNonReplyMessage)]

您可以使用 `CreateReply` 方法來初始化新訊息 (這會自動設定訊息的 `Recipient`、`From` 和 `Conversation` 屬性)。 或者，您可以使用 `CreateMessageActivity` 方法來建立新訊息，並自行設定所有的屬性值。

> [!NOTE]
> Bot Framework 對於 Bot 可傳送的訊息數目並無任何限制。 不過，大部分的通道都會強制執行節流限制，來限制 Bot 在短時間內傳送大量訊息。 此外，如果 Bot 連續快速地傳送多則訊息，通道不一定會依照正確順序呈現訊息。

## <a name="start-a-conversation"></a>開始對話

您的 Bot 有時可能需要起始與一或多位使用者的對話。 您可以呼叫 `CreateDirectConversation` 方法 (適用於與單一使用者的私人對話) 或 `CreateConversation` 方法 (適用於與多位使用者的群組對話) 來擷取 `ConversationAccount` 物件，藉以開始對話。 然後，建立訊息，並呼叫 `SendToConversation` 方法來傳送它。 若要使用 `CreateDirectConversation` 方法或 `CreateConversation` 方法，您必須先使用目標通道的服務 URL (如果您已從上一個訊息中保存它，則可從快取中擷取)，來[建立連接器用戶端](#create-a-connector-client)。 

> [!NOTE]
> 並非所有通道都支援群組對話。 若要判斷通道是否支援群組對話，請參閱通道的文件。

此程式碼範例使用 `CreateDirectConversation` 方法來建立與單一使用者的私人對話。

[!code-csharp[Start private conversation](../includes/code/dotnet-send-and-receive.cs#startPrivateConversation)]

此程式碼範例使用 `CreateConversation` 方法來建立與多位使用者的群組對話。

[!code-csharp[Start group conversation](../includes/code/dotnet-send-and-receive.cs#startGroupConversation)]

## <a name="additional-resources"></a>其他資源

- [活動概觀](bot-builder-dotnet-activities.md)
- [建立訊息](bot-builder-dotnet-create-messages.md)
- <a href="/dotnet/api/?view=botbuilder-3.11.0" target="_blank">適用於 .NET 的 Bot Builder SDK 參考</a>
- <a href="https://docs.botframework.com/en-us/csharp/builder/sdkreference/dc/d2f/class_microsoft_1_1_bot_1_1_connector_1_1_activity.html" target="_blank">Activity 類別</a> \(英文\)
- <a href="/dotnet/api/microsoft.bot.connector.connectorclient" target="_blank">ConnectorClient 類別</a>

[ConnectorClient]: /dotnet/api/microsoft.bot.connector.connectorclient
