---
title: 回應範本 | Microsoft Docs
description: 了解如何設定對話設計工具 Bot 的回應範本。
author: vkannan
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ROBOTS: NoIndex, NoFollow
ms.openlocfilehash: b09b7fa5f5672c121711deb2edcdc9718a79983f
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/18/2018
ms.locfileid: "39300358"
---
# <a name="response-template-for-conversation-designer-bots"></a>對話設計工具 Bot 的回應範本
> [!IMPORTANT]
> 對話設計工具尚未提供給所有客戶使用。 有關對話設計工具可用性的詳細資訊會於本年度稍後提供。

語言產生可讓 Bot 使用變數的回應訊息，以豐富且自然的方式與使用者通訊。 這些訊息會透過對話設計工具中的回應範本來管理。

回應範本能夠重複使用回應，並協助在 Bot 回應中維持一致的語氣和語言。 

## <a name="create-response-templates"></a>建立回應範本

在對話設計工具中，您可以建立回應範本，以便在任何您需要將回應傳送給使用者的情況下重複使用。 

簡單回應範本可定義一次性的可能口語和顯示語句集合。 您可以接著在 Bot 的回應、提示狀態或通用卡 (Adaptive Card) 中使用這些範本，重新建構完整解析的字串。

若要新增簡單回應範本，請執行下列作業︰
1. 按一下左面板中的 [新增]。 操作功能表隨即出現。
2. 按一下 [簡單回應]。 編輯視窗會出現在主要內容面板中。
3. 在 [簡單回應名稱] 欄位中，輸入此簡單回應的名稱。
4. 在 [Bot 給使用者的回應] 欄位中，輸入回應 (一次一個片語)。
5. 當您完成簡單回應的設定時，按一下右上角的 [儲存]。 

例如，您可以使用下列項目建立通知片語範本 (稱之為 "AckPhrase")：

- 確定
- Sure
- You bet
- I'd be happy to help
- Glad to help

使用此範本時，對話執行階段會隨機解析為上述其中一個字串，讓 Bot 的回應聽起來更自然，而不會既無趣又單調。

## <a name="conditional-response-templates"></a>條件式回應範本

條件式回應範本使用條件來指定多個回應。 您撰寫的回呼函式可協助語言產生引擎，根據指定條件來決定所要使用的回應字串。 

例如，當天時間範本應根據當天的實際時間解析為 "good morning" 或 "good evening"。 

若要新增條件式回應範本，請執行下列作業︰
1. 按一下左面板中的 [新增]。 操作功能表隨即出現。
2. 按一下 [條件式回應]。 編輯視窗會出現在主要內容面板中。
3. 在 [條件式回應名稱] 欄位中，輸入此範本的名稱。
4. 在 [條件式回應] 欄位中，輸入您的片語 (一次一個)。 例如，針對 "timeOfDayTemplate"，輸入 "Good morning" 和 "Good evening" 作為範本中兩個可能的回應。
5. 對於 "Good morning" 回應，將其 [條件名稱] 指定為 "morning"。 同樣地，對於 "Good evening" 回應，將其 [條件名稱] 指定為 "evening"。 您將撰寫的自訂指令碼會傳回上述其中一個條件。
6. 在 [要執行的程式碼] 欄位中，輸入回呼函式名稱 (例如：`fnResolveTimeOfDayTemplate`)。 然後，按一下 [檢視程式碼] 以載入 [指令碼] 編輯器。 您可以在此定義此回呼函式的實作。
7. 按一下右上角的 [儲存]，以儲存您的變更並建立此範本。

fnResolveTimeOfDayTemplate 回呼函式的程式碼範例。 此函式會傳回符合回應所指定 "morning" 或 "evening" [條件名稱] 的字串。

```javascript
module.exports.fnTimeOfDayTemplate = function(context) {
    var currentTime = new Date().getHours();
    if(currentTime >= 12) {
        return "evening!";
    } else {
        return "morning!";
    }
}
```

## <a name="send-a-response-to-user"></a>將回應傳送給使用者

若要使用回應範本將回應傳送給使用者，請以方括號括住範本名稱。 例如，若要在意見反應狀態中使用 "AckPhrase" 簡單回應範本，請在 [Bot 給使用者的回應]  片語中輸入 `[AckPhrase], I will get that done for you right away`

如果您想要在回應中使用方括號，請使用 "\" 作為逸出字元，以指示對話執行階段略過方括號的解析。

如果您已定義實體，則可在您給使用者的回應中使用它們。 以大括號括住實體名稱，即可在語言產生中參照您的實體。 例如，如果 Bot 有 `location` 實體，您可以在回應中參考，如下所示：`[AckPhrase], I can help you find a table at {location}.`

## <a name="nesting-response-templates"></a>巢狀回應範本

回應範本可以是「巢狀」；一個回應範本可以有另一個回應範本的參考。 例如，您可以使用 `AckPhrase` 簡單回應範本和 `timeOfDayTemplate` 條件式回應範本，建構新的 `timeBasedGreeting` 回應範本，以使用這兩個範本來解析最後回應。 

> [!TIP]
> 雖然範本巢狀處理是一項強大的功能，請務必檢查您的巢狀範本不會造成無限迴圈。 也就是，避免讓 `AckPhrase` 呼叫 `timeOfDayTemplate` 以及 `timeOfDayTemplate` 回呼 `AckPhrase` 的情況。

例如，建立新的 [簡單回應] 名稱 `timeBasedGreeting`，並輸入下列文字作為此範本可能的 [Bot 給使用者的回應]：`[timeOfDayTemplate] [AckPhrase], ... `

## <a name="define-user-utterance-as-help-tips"></a>定義使用者語句作為說明提示

定義使用者**語句**時，您也可將語句設定為 [說明提示]。 若要將語句設定為 [說明提示]，請按一下語句右邊的 `...`，然後選取 [作為說明提示]。 

說明提示會使用於內建語言產生範本 `[builtin-tasktips]`，而您會在其中定義 [Bot 給使用者的回應] 欄位。 例如，您可以撰寫如下回應：`Sorry I did not understand that. Here are some things you can try - [builtin.tasktips]`

如果工作未定義任何 [協助提示]，則 `[builtin-tasktips]` 會解析為空字串。 如果工作已定義多個 [協助提示]，則 `[builtin-tasktips]` 會在每次得到回應時隨機選取一個協助提示。

## <a name="next-step"></a>後續步驟
> [!div class="nextstepaction"]
> [調適型卡片](conversation-designer-adaptive-cards.md)
