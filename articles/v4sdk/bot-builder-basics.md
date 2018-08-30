---
title: Bot Builder 的基本概念 |Microsoft Docs
description: Bot Builder SDK 基本結構。
keywords: 轉向內容，Bot 結構，接收輸入
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 04/18/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 34564b411f911ae82197d5a34cb954a103abe70b
ms.sourcegitcommit: 2dc75701b169d822c9499e393439161bc87639d2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/24/2018
ms.locfileid: "42905412"
---
# <a name="basic-bot-structure"></a>基本 Bot 結構

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

Azure Bot Service 和 Bot Builder SDK 提供資料庫、範本和工具來協助您建置和偵錯 Bot。 不過，在我們深入進入細節前，請務必瞭解 Bot 和搭配使用的基本結構。 這些概念套用您所選的任一程式語言中。 全文有提供更深入內容的連結，此文章將提供 Bot 如何運作的初始心理架構。

從最基礎開始，讓我們來探索 Bot 的基本結構。

## <a name="creation-of-your-bot"></a>您 Bot 的建立

Bot 內可以建立各種不同的方式，例如在 [Visual Studio](~/dotnet/bot-builder-dotnet-sdk-quickstart.md) 中的 [Azure 入口網站](~/bot-service-quickstart.md)，或透過 [JavaScript](~/javascript/bot-builder-javascript-quickstart.md)，[Java](~/java/bot-builder-java-quickstart.md)，或 [Python](~/python/bot-builder-python-quickstart.md) 的命令列工具。 建立之後，就可以本機電腦，Azure，或另一個雲端服務上執行 Bot。 無論是在執行的位置，或建置方式，這些函數都非常類似。

## <a name="interaction-with-your-bot"></a>與您的 Bot 互動

Bot 在任何網站或應用程式中本來就沒有任何可見的 UI，而是使用者透過[對話](~/v4sdk/bot-concepts.md#activities-and-conversations)與其進行交流。 透過應用程式來連接到 Bot (我們稱之為[通道](~/v4sdk/bot-concepts.md)，但我們先不會提到)，會在使用者與您的 Bot 之間來回傳送特定資訊。

資訊的每個單位在我們的 Bot 內稱為**活動**，且活動可以有各種形式。 活動包括來自使用者的通訊，也就是**訊息**，或其他資訊包含在少數幾個其他[活動類型](~/bot-service-activities-entities.md)。 其他資訊可以包含像是新的對象加入或離開的對話，交談結束等。使用者連線的通訊類型由底層系統發送，使用者不需要採取任何動作。

Bot 接收通訊並將其包裝在具有正確類型的活動物件中，以將其提供給您的 Bot 程式碼。 所有其他活動類型提供有用的資訊，不過最有趣且最常見的活動為使用者的**訊息**活動。

Bot 接收每個活動都會開始一個回合，我們稍後將更詳細地說明。

## <a name="receiving-user-input"></a>接收使用者輸入

當我們收到來自使用者的訊息活動時，我們需要了解他們要傳送什麼給我們。 最直接的方法是單純比對輸入訊息文字與字串。 根據這個字串，我們可以選擇按照您的 Bot 預完成的來執行動作。 這可能包括回覆使用者，更新變數或資源，將它儲存在[記憶體](~/v4sdk/bot-builder-storage-concept.md)中或相似的處理。

也有更複雜的方式來辨識使用者輸入，例如使用 [LUIS](~/v4sdk/bot-builder-concept-luis.md) 或是 [QnA Maker](~/v4sdk/bot-builder-howto-qna.md)，但字串比對是最簡單。

## <a name="defining-a-turn"></a>定義一個回合

[!INCLUDE [Define a turn](~/includes/snippet-definition-turn.md)]

活動處理是由**配接器**所管理，[活動處理](~/v4sdk/bot-builder-concept-activity-processing.md)中有記述更多詳細資料。 當配接器接收活動時，它會建立**回合內容**來提供活動的相關資訊，並將內容提供給我們正在處理的回合 回合內容在回合期間使用，回合結束後丟棄並標記回合結束。

[回合內容](~/v4sdk/bot-builder-concept-activity-processing.md#turn-context)包含少數幾個的詳細資訊，而且相同的物件在您的 Bot 的所有層面中使用。 這是很有幫助的，因為這個回合內容的物件可以使用，而且應該是用來儲存稍後回合中會需要用到的資訊。

> [!IMPORTANT]
> 所有**回合**都是獨立的，可以自行運作並有可能會重複內容。 Bot 可能同時從不同通道上的不同使用者處理多個回合。 每個回合會有自己的回合內容，但在某些情況下值得考慮所引進的複雜問題。

## <a name="where-to-go-from-here"></a>現在該如何開始

這篇文章避免許多太詳細的資料，例如，[活動如何處理](~/v4sdk/bot-builder-concept-activity-processing.md)、不同的[對話類型](~/v4sdk/bot-builder-conversations.md)，追蹤[您的對話狀態](~/v4sdk/bot-builder-storage-concept.md)以得到更聰明的對話等等。 其餘概念主題建立在基礎知識的基礎上，並涵蓋瞭解 Bot 和 Azure Bot 服務所需的其他概念。 您可以跟著以下的步驟區塊來建構您的知識，或者您可以跳到親自試用[快速入門](~/bot-service-quickstart.md)來建置您的第一個 Bot，或深入了解如何[開發](~/v4sdk/bot-builder-howto-send-messages.md) Bot。

## <a name="next-steps"></a>後續步驟

接下來，Bot 連接器服務允許您的 Bot 與不同平台上的使用者進行通訊。

> [!div class="nextstepaction"]
> [通道和 Bot 連接器服務](~/v4sdk/bot-concepts.md)
