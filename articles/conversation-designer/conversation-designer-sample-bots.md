---
title: 從範例 Bot 建立對話設計工具 Bot | Microsoft Docs
description: 使用這其中一個範例 Bot 來啟動新的 Bot。
author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ROBOTS: NoIndex, NoFollow
ms.openlocfilehash: 0ebf0d1b90b03789d8a77710631c83f0914099c5
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/18/2018
ms.locfileid: "39299527"
---
# <a name="create-a-conversation-designer-bot-from-a-sample-bots"></a>從範例 Bot 建立對話設計工具 Bot

對話設計工具是一款強大的架構，可提供視覺效果 Bot 建立器、用於定義 Bot 的宣告式模型，以及讓建立 Bot 更簡單的執行階段。 讓 Bot 建立更簡單的其中一種方式是從現有的 Bot 啟動您的 Bot。

您可以從許多範例 Bot 之一，或先前匯出的對話設計工具 Bot，建立對話設計工具 Bot。 每個範例 Bot 都是功能完整的 Bot。 這可為您提供一個穩固的起點，讓您可以開始建置自己的 Bot，或自訂範例 Bot 以符合您的案例。

## <a name="the-sample-bots"></a>範例 Bot

對話設計工具隨附一組**範例 Bot**。 您可以根據這其中一個**範例 Bot**，開始建置您的 Bot。 這些**範例 Bot** 大部分都會涵蓋功能完整之 Bot 的小型案例。 如果您想要看到這所有的範例 Bot 都可一起運作，請建立**完整的 Contoso Cafe Bot**。 

下表列出您可以建立的 Bot 類型，以及每個 Bot 的相關簡短描述和它所涵蓋的案例。 

| Bot 名稱 | 案例 | 引進的功能 |
| ---- | ---- | ---- |
| **基本的 Bot** | 已設定好一些基本功能的 Bot。 例如，這個 Bot 會在將新的使用者新增到對話時顯示歡迎使用訊息、回應問候訊息，然後在使用者詢問一些 Bot 不了解的事物時執行後援行為。 | - [工作](conversation-designer-tasks.md) <br/>- 工作觸發程序及動作 <br/>- [Language Understanding 觸發程序](conversation-designer-luis.md)<br/>- [指令碼觸發程序](conversation-designer-code-recognizer.md)<br/>- [回覆動作](conversation-designer-reply.md) |
| **回應 Bot** | 回應使用者訊息的**基本 Bot**。  不論您傳送給 Bot 何種訊息，它都將回應下列訊息：「以下是您所說的：...訊息...」。 | - [指令碼觸發程序](conversation-designer-code-recognizer.md)<br/>- 以程式碼解譯內容物件<br/>- 以程式碼建立實體<br>- 在 Bot 的回覆中使用實體 |
| **QnA Bot** | 這個 Bot 可以使用 [QnAMaker.ai](http://qnamaker.ai) \(英文\) 來執行單回合的問答體驗。 若要使此範例能夠運作，您必須已設定 [QnA Maker](http://qnamaker.ai) \(英文\) 帳戶，並已使用一些問題和答案來將它定型。 然後，開啟這個 **QnA Bot**、前往 [指令碼] 頁面，然後使用您的 QnA Maker 資訊來取代這兩個預留位置：`<INSERT YOUR KB ID>` 和 `<INSERT YOUR SUBSCRIPTION KEY>`。 當您輸入那兩項資訊之後，請[儲存](conversation-designer-save-bot.md) Bot，然後可[測試](conversation-designer-debug-bot.md)您的 Bot。 | - [指令碼觸發程序](conversation-designer-code-recognizer.md)<br/>- 從程式碼提出 HTTP 要求<br/>- 連接至自訂 NLP/LU 服務 (此範例中的 QnAMaker.ai) |
| **對話 Bot** | Bot 可以參與簡單的對話，並記住使用者名稱之類的內容。 | - [對話動作](conversation-designer-dialogues.md)<br/>- 基本的[對話狀態](conversation-designer-dialogues.md#dialogue-states)和屬性<br/>- 介紹[提示的對話](conversation-designer-dialogues.md#prompt-state) (定義提示和重新提示的行為)<br/>- [建立意圖](conversation-designer-luis.md#create-a-new-intent)並在提示的對話中標示實體<br/>- 在 Bot 的回覆中使用 Language Understanding 所產生的實體<br/>- 使用[卡片](conversation-designer-adaptive-cards.md)來改善使用者體驗<br/>- 將 Bot 實體繫結到[輸入表單](conversation-designer-adaptive-cards.md#input-form)<br/>- 建立和使用[回應範本](conversation-designer-response-templates.md) Bot 資產 |
| **記憶 Bot** | Bot 可以詢問使用者的喜好設定，並於稍後在對話中重新叫用該資訊。 當您說出像是「設定通訊喜好設定」之類的事項時。 這個 Bot 將詢問您要透過「來電」或「簡訊」進行通訊。 您也可以說出「尋找咖啡廳的位置」或「引導我前往位於 Redmond, WA 的 Contoso Cafe」之類的事項來尋找 Contoso Cafe 的位置資訊。 當 Bot 找到該資訊之後，它會透過電話或簡訊，將它找到的資訊傳送給您。 <br/><br/>如果已經設定通訊喜好設定，Bot 將直接使用該喜好設定，否則它將提示使用者輸入它。 | - [對話動作](conversation-designer-dialogues.md)<br/>- 使用 [`context.global`](conversation-designer-context-object.md#context-properties)，來將資料儲存到 Bot 的記憶體中<br/>- 從記憶體中重新叫用內容資訊以用於對話中 |
| **訂位 Bot** | Bot 可以與使用者進行多回合的對話，以協助使用者預訂 Contoso Cafe 的位子。 <br/><br/>若要訂位，這個 Bot 需要您提供三項資訊：(1) *宴會規模*、(2) *日期/時間*及 (3) *地點*。 <br/><br/>您可以說出「我想要訂星期二下午 2 點在 Redmond 店的 4 個人位子」之類的事項，藉以在同一個訊息中提供這三項資訊。 如果您省略這三項資訊中的任一項，Bot 將提示您輸入它。 | - [對話動作](conversation-designer-dialogues.md)<br/>- 含有對其他對話之參考的複雜對話動作<br/>- 一起撰寫對話<br/>- 使用 LUIS 的完整功能來設定您的 Bot 對話，以管理像是向前填滿位置和更正體驗之類的事項<br/>- 使用[卡片](conversation-designer-adaptive-cards.md)來改善使用者體驗<br/>- 將 Bot 實體繫結到[輸入表單](conversation-designer-adaptive-cards.md#input-form)<br/>- 建立和使用[條件式回應範本](conversation-designer-response-templates.md#conditional-response-templates) Bot 資產 |
| **精簡搜尋 Bot**  | Bot 可協助您使用自上一個對話保留下來的內容資訊來調整您的搜尋。 您可以說出「Seattle 的店這個週末有營業嗎？」 後面接著「那麼，Renton 的店呢」之類的事項，這個 Bot 會針對第二個問題記住您仍在討論這個週末的事項。 您也可以嘗試說出「這星期五下午 8 點哪些商店有營業呢」，後面接著「那下午 10 點呢？」 等事項 | - [對話動作](conversation-designer-dialogues.md)<br/>- 使用 [`context.global`](conversation-designer-context-object.md#context-properties) 來將資料儲存到 Bot 的記憶體中<br/>- 使用 Bot 的記憶體，透過將內容向前推進來與使用者進行對話<br/>- 從記憶體中重新叫用內容資訊以用於對話中 |
| **訂購三明治 Bot** | Bot 會示範在接收三明治訂單時提示使用者輸入的不同方式。 例如，這個 Bot 將協助您接收三明治訂單。 <br/><br/>若要訂購三明治，Bot 需要四項資訊：(1) 三明治的大小、(2) 麵包類型、(3) 蛋白質種類選項和 (4) 三明治配料。 <br/><br/>類似於**訂位 Bot**，您可以在一則訊息中提供這四項必要資訊，或者您可以說「您好，我想要訂購三明治。」 或者「我可以買個大三明治嗎？」，一次提供一項資訊， 然後 Bot 將要求您提供處理訂單所需的剩餘資訊。 | - [對話動作](conversation-designer-dialogues.md)<br/>- 設定[提示](conversation-designer-response-templates.md)，以不同方式收集資訊 |
| **Contoso Cafe Bot** | 針對 Contoso Cafe 建立的功能完整 Bot。 這個 Bot 可以執行其他範例 Bot 所做的一切。 它會提供一則歡迎使用訊息，以及您可互動的豐富卡片。 它支援說明、問候語和後援的訊息。 協助您預訂或尋找咖啡廳地點或訂購三明治。 | - 對話設計工具的所有功能<br/>- 使用 [`context.global`](conversation-designer-context-object.md#context-properties) 來將資料儲存到 Bot 的記憶體中<br/>- 使用[卡片](conversation-designer-adaptive-cards.md)，根據使用者輸入來觸發特定的 Bot 工作 (查看歡迎使用訊息中含有按鈕的卡片)<br/>- 建置複雜的 Bot，其可執行數個不同的[工作](conversation-designer-tasks.md)。 |

## <a name="explore-sample-bots"></a>探索範例 Bot

如果您目前使用的範例 Bot 不是您所需的，則可隨時切換到另一個範例 Bot。 您可以在 [探索範例 Bot] 頁面中，找到一份所有範例 Bot 的清單。

> [!WARNING] 
> 匯入範例 Bot，將使用範例 Bot 來取代目前的 Bot。 如果您不想在目前的 Bot 中遺失您的自訂，請將它[匯出](conversation-designer-export-import-bot.md#export-a-conversation-designer-bot)至 .zip 檔案。

若要切換至不同的範例 Bot，執行下列作業：
1. 從目前的 Bot，按一下 [瀏覽範例 Bot]。 您可以在左側導覽窗格底部找到此選項。 
2. 從 [範例 Bot] 的清單中挑選 Bot，然後按一下 [匯入]。
3. 如果您想要將工作儲存在目前的 Bot 中，請選擇 [備份並匯入] 選項。 這個選項會將目前的 Bot 儲存到您的本機電腦，接著匯入新的**範例 Bot**。 否則，請選擇 [匯入但不要備份]。

您的 Bot 現在已取代為您剛才選取的新範例 Bot。 

## <a name="next-step"></a>後續步驟
> [!div class="nextstepaction"]
> [儲存 Bot](conversation-designer-save-bot.md)
