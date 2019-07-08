---
title: 適用於 Node.js 的 Bot Framework SDK 重要概念 | Microsoft Docs
description: 了解建置和部署適用於 Node.js 的 Bot Framework SDK 中可用交談式 Bot 所需的重要概念和工具。
author: DeniseMak
ms.author: v-demak
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: a3cff9a77de098ee524334183ba891068f176b6e
ms.sourcegitcommit: dbbfcf45a8d0ba66bd4fb5620d093abfa3b2f725
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/28/2019
ms.locfileid: "67464777"
---
# <a name="key-concepts-in-the-bot-framework-sdk-for-nodejs"></a>適用於 Node.js 的 Bot Framework SDK 重要概念

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-concepts.md)
> - [Node.js](../nodejs/bot-builder-nodejs-concepts.md)

本文介紹適用於 Node.js 的 Bot Framework SDK 重要概念。 如需 Bot Framework 的簡介，請參閱 [Bot Framework 概觀](../overview-introduction-bot-framework.md)。

## <a name="connector"></a>連接器
Bot Framework Connector 是一項可將 Bot 連線至多個*通道* (即 [Teams](https://docs.microsoft.com/microsoftteams/platform/concepts/bots/bots-create)、Skype、Facebook、Slack 和 SMS 等用戶端) 的服務。 

此連接器可藉由從 Bot 至通道和從通道至 Bot 的訊息轉送，讓 Bot 和使用者之間的通訊更為順暢。 Bot 的邏輯裝載為 Web 服務，並透過連接器服務接收使用者的訊息，而 Bot 的回覆會使用 HTTPS POST 傳送至連接器。 

適用於 Node.js 的 Bot Framework SDK 所提供的 [UniversalBot][UniversalBot] and [ChatConnector][ChatConnector] 類別可用來設定 Bot，使其透過 Bot Framework Connector 來傳送和接收訊息。 `UniversalBot` 類別會形成 Bot 的思考核心。 它負責管理 Bot 與使用者之間的所有對話。 `ChatConnector` 類別會將您的 Bot 連線至 Bot Framework Connector 服務。
如需使用這些類別進行示範的範例，請參閱[使用適用於 Node.js 的 Bot Framework SDK 建立 Bot](bot-builder-nodejs-quickstart.md)。

連接器也會將 Bot 傳送至通道的訊息正規化，讓您能夠以跨平台的方式開發 Bot。 要將訊息正規化，必須將訊息從 Bot Framework 的結構描述轉換為通道的結構描述。 如果通道不支援架構結構描述的所有層面，則連接器會嘗試將訊息轉換為通道支援的格式。 例如，如果 Bot 將訊息傳送至 SMS 通道，而訊息中包含具有動作按鈕的卡片，則連接器可能會將卡片轉譯為影像，並將動作包含為訊息文字中的連結。 [通道偵測器][ChannelInspector]是一項 Web 工具，會說明連接器在各種通道上轉譯訊息的方式。

要使用 `ChatConnector`，您必須在 Bot 內設定 API 端點。 使用 Node.js SDK 時，這項作業通常可藉由安裝 `restify` Node.js 模組來完成。 您也可以使用 [ConsoleConnector][ConsoleConnector] 為主控台建立 Bot，而不需要 API 端點。

## <a name="messages"></a>訊息

訊息中可包含要顯示的文字、要讀出的文字、附件、複合式資訊卡 (Rich Card) 和建議的動作。 您可以使用 [session.send][SessionSend] 方法來傳送訊息，以回應使用者的訊息。 您的 Bot 可視需要呼叫任意次數的 `send`，以回應使用者的訊息。 如需示範這項功能的範例，請參閱[回應使用者訊息][RespondMessages]。

如需相關範例，以了解如何傳送含有互動式按鈕，可讓使用者點選以起始動作的圖形化複合式資訊卡，請參閱[將複合式資訊卡 (Rich Card) 新增至訊息](bot-builder-nodejs-send-rich-cards.md)。 如需示範如何傳送和接收附件的範例，請參閱[傳送附件](bot-builder-nodejs-send-receive-attachments.md)。 如需相關範例，以了解如何在訊息中指定您的 Bot 在具備語音功能的通道上要說出的文字，並傳送訊息，請參閱[將語音新增至訊息](bot-builder-nodejs-text-to-speech.md)。 如需示範如何傳送建議動作的範例，請參閱[傳送建議的動作](bot-builder-nodejs-send-suggested-actions.md)。

## <a name="dialogs"></a>對話方塊
對話方塊可協助您組織 Bot 中的對話邏輯，而且可作為[設計對話流程](../bot-service-design-conversation-flow.md)的基礎。 如需對話的簡介，請參閱[使用對話方塊來管理對話](bot-builder-nodejs-dialog-manage-conversation.md)。

## <a name="actions"></a>動作
您必須將 Bot 設計成具有中斷處理能力，例如，處理對話流程中隨時提出的取消或協助要求。 適用於 Node.js 的 Bot Framework SDK 提供全域訊息處理常式，可觸發如取消或叫用其他對話之類的動作。 如需如何使用 [triggerAction][triggerAction] 處理常式的範例，請參閱[處理使用者動作](bot-builder-nodejs-dialog-actions.md)。
<!--[Handling cancel](bot-builder-nodejs-manage-conversation-flow.md#handling-cancel), [Confirming interruptions](bot-builder-nodejs-manage-conversation-flow.md#confirming-interruptions) and-->


## <a name="recognizers"></a>辨識器
當使用者對您的 Bot 提出某項要求時 (例如「協助」或「尋找新聞」)，Bot 必須了解使用者的要求，並採取適當動作。 您可以將 Bot 設計成根據使用者的輸入辨識意圖，並將該意圖與動作產生關聯。 

您可以使用 Bot Framework SDK 所提供的內建規則運算式辨識器、呼叫的外部服務 (例如 LUIS API)，或實作自訂辨識器，以判斷使用者的意圖。 如需示範如何將辨識器新增至 Bot 並用它來觸發動作的範例，請參閱[辨識使用者意圖](bot-builder-nodejs-recognize-intent-messages.md)。


## <a name="saving-state"></a>儲存狀態

Bot 的設計是否精良，要件之一是能夠追蹤對話的上下文，讓 Bot 記住使用者最後提問這類的事項。 使用 Bot Framework SDK 建置的 Bot 會設計為無狀態，以便輕易調整為跨多個計算節點執行。 Bot Framework 提供可儲存 Bot 資料的儲存體系統，讓 Bot Web 服務可進行調整。 因此，您在一般情況下應避免使用全域變數或函式閉包來儲存狀態。 如果這麼做，當您想要相應放大 Bot 時將會出現問題。 您應使用 Bot [工作階段][Session]物件的下列屬性，來儲存相對於使用者或對話的資料：

* **userData** 會全域儲存使用者在所有對話中的資訊。
* **conversationData** 會全域儲存單一對話的資訊。 這項資料會顯示給對話中的所有人，因此在將資料儲存至此屬性時應謹慎操作。 依預設會啟用此屬性，但您可以使用 Bot 的 [persistConversationData][PersistConversationData] 設定將其停用。
* **privateConversationData** 會全域儲存單一交談的資訊，但這會是目前使用者專屬的私人資料。 這項資料會跨越所有對話方塊，因此很適合用來儲存您在對話結束後將會清除的暫時性狀態。
* **dialogData** 會保存單一對話方塊執行個體的資訊。 這是儲存對話方塊中[瀑布圖](bot-builder-nodejs-dialog-waterfall.md)步驟之間，暫存資訊所不可或缺的屬性。

如需示範如何使用這些屬性來儲存和擷取資料的範例，請參閱[管理狀態資料](bot-builder-nodejs-state.md)。

## <a name="natural-language-understanding"></a>自然語言理解

Bot 建立器可讓您使用 LUIS，將自然語言理解功能新增至使用 [LuisRecognizer][LuisRecognizer] 類別的 Bot。 您可以新增對已發佈語言模型進行參考的 **LuisRecognizer** 執行個體，然後再新增會執行動作以回應使用者語句的處理常式。 若要了解運作中的 LUIS，請觀看以下長度約 10 分鐘的教學課程：

* [Microsoft LUIS 教學課程][LUISVideo] (影片)

## <a name="next-steps"></a>後續步驟
> [!div class="nextstepaction"]
> [對話概觀](bot-builder-nodejs-dialog-overview.md)



[PersistConversationData]: https://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.iuniversalbotsettings.html#persistconversationdata
[UniversalBot]: https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.universalbot.html
[ChatConnector]: https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.chatconnector.html
[ConsoleConnector]: https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.consoleconnector.html

[ChannelInspector]: ../bot-service-channel-inspector.md

[Session]: https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.session.html
[SessionSend]: https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.session#send

[triggerAction]: https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.dialog.html#triggeraction
[waterfall]: bot-builder-nodejs-prompts.md

[RespondMessages]:bot-builder-nodejs-use-default-message-handler.md

[LUISRecognizer]: https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.luisrecognizer
[LUISVideo]: https://vimeo.com/145499419
