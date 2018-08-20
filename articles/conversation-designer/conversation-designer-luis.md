---
title: 定義 LUIS 辨識器作為工作觸發程序 | Microsoft Docs
description: 了解如何使用 LUIS.ai 將語言理解辨識器設定為工作觸發程序
author: vkannan
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ROBOTS: NoIndex, NoFollow
ms.openlocfilehash: 39fe222fb1d54346b33617c425b1fdf2d56daa0d
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/18/2018
ms.locfileid: "39300351"
---
# <a name="define-a-luis-recognizer-as-task-trigger"></a>定義 LUIS 辨識器作為工作觸發程序
> [!IMPORTANT]
> 「對話設計工具」尚未提供所有客戶使用。 有關對話設計工具可用性的詳細資訊會於本年度稍後提供。

大多數情況下，使用者會以自然語言表達要執行某工作的意圖。 透過對話設計工具，您可以為運用 <a href="https://luis.ai" target="_blank">LUIS.ai</a> (英文) 的 Bot 輕鬆設定自然語言理解模型。

若要這樣做，請選取觸發程序類型 [使用者說出或輸入內容]。 這會提供您可指定**意圖**名稱的選項。 

您可以搜尋現有的意圖，或在「哪個語言的意圖？」欄位中建立新意圖。

## <a name="create-a-new-intent"></a>建立新意圖

若要建立新的意圖，輸入意圖名稱，然後按一下 [建立新意圖]。 請針對您預期使用者可能說出的內容，輸入會觸發此特定工作的範例語句。

例如，咖啡廳 Bot 應能執行尋找使用者附近咖啡廳位置的工作。 若要處理這個案例，請選取 [使用者說出或輸入內容] 並將 [意圖名稱] 設定為 "LocationNearMe"。 然後，為此意圖提供範例語句。 例如︰ 
- 尋找附近地點
- 尋找附近的咖啡廳位置
- Redmond 現在開了嗎？
- 西雅圖有店嗎？
- 哪些咖啡廳現在開了？
- 哪裡可以吃東西？
- 我想要吃東西
- 我餓了

盡可能輸入您所想得到的多個語句，以協助使用者表達意圖，觸發此特定工作。

## <a name="default-intents-provisioned-for-your-bot"></a>為 Bot 佈建的預設意圖

根據預設，您的 Bot 會佈建 4 種意圖。 
- **無**：這是 Bot 的後援 (預設) 意圖。 使用此意圖有助於界定 Bot 還不知道如何回應的項目。
- **協助**：設定範例語句，協助判斷使用者是否需要協助。 「求助、我需要協助、我能怎麼辦？、我卡住了」等等。
- **問候**：設定範例語句，協助符合問候意圖，例如「嗨、哈囉、早安、您好 Bot」等等。
- **取消**：設定取消意圖的範例語句。 「停止、取消、不要這樣、還原」等等。

## <a name="create-and-label-entities"></a>建立並為實體加上標籤

除了有助於判斷使用者意圖，語言理解也可協助您判斷與工作相關的特定實體。 例如，當使用者說「尋找 Redmond 附近的咖啡廳位置」，您需要擷取「Redmond」作為「位置」實體的可能值。 

為了設定工作的實體，請從**語句**字串中選取語句中應作為實體值範例的部分。 將它指派到現有實體或建立新實體。 若要建立新的實體，請將實體名稱輸入到 [搜尋或建立] 欄位，然後按一下 [建立新實體]。 

# <a name="supported-entity-types"></a>支援的實體類型

語言理解提供可建立不同類型實體的功能。 建立新的實體時，您必須指定 `type`。 

可用的類型為：

- **簡單**：這是「預設」類型。
- **清單**：如果您的實體具有一組有限的可能值，請使用這個類型。 範例：色彩、城市。
- **階層式**：使用此類型來建立具有父子關聯性的實體。 範例：fromCity 和 toCity 兩者都有 City 作為父代實體
- **複合**：使用此類型來建立可組成有意義單位的值群組。 範例：「城市」和「州」可共同組成「位置」實體。

<!-- # pre-built entity types TBD -->

# <a name="entities-in-use"></a>使用中的實體

隨著您建立並將實體新增至語言理解區段，「使用中的實體」資料表會以此特定工作所使用的實體清單進行更新。 您可以手動將這個工作中所使用的其他實體新增至清單內。 

可用選項包括：

- **程式碼**：這是在您自訂指令碼中所建立的實體。 您可以在這裡指定它以進行撰寫功能，例如 IntelliSense。

<!-- # Use as help tip TBD  -->

## <a name="next-step"></a>後續步驟
> [!div class="nextstepaction"]
> [程式碼辨識器](conversation-designer-code-recognizer.md)
