---
title: 定義對話作為執行動作 | Microsoft Docs
description: 了解如何設定對話作為執行動作。
author: vkannan
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ROBOTS: NoIndex, NoFollow
ms.openlocfilehash: d59df20821a7f63eb9ee5dea365597b1af839f75
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/18/2018
ms.locfileid: "39299050"
---
# <a name="define-a-dialogue-as-a-do-action"></a>定義對話作為執行動作
> [!IMPORTANT]
> 對話設計工具尚未提供給所有客戶。 有關對話設計工具可用性的詳細資訊會於本年度稍後提供。

對話涵蓋適用於特定工作的對話式模型。 例如，可協助使用者進行預訂的咖啡廳 Bot 可以定義名為「預訂」的工作。 **執行**動作會繫結至要在 Bot 和使用者之間將對話式流程模型化的對話。 

當 Bot 與使用者進行來回對話 (conversation) 以協助完成工作時，對話 (dialogue) 特別有用。  使用者很少在單一語句中表達完成工作所需的所有值。 

訂位需要地點、宴會規模、日期及時間的喜好設定。 進行訂位的 Bot 必須了解並處理使用者可能說出的各種可能片語。 

- *訂*：在此案例中，不會擷取任何必要的實體。 Bot 現在必須與使用者進行對話。
- *訂這個星期六下午 7 點*：在此案例中，會指定日期和時間的喜好設定，但 Bot 仍然必須收集預期的地點與宴會規模。

在對話期間，某些實體可能會隨著使用者輸入而變更。 例如，如果使用者說「訂這個星期六晚上 7 點在 Redmond 的位子」。 但是，Redmond 店會在星期六晚上六點結束營業，則 Bot 的對話應該要處理這類無效的要求。 

對話 (conversation) 設計工具提供一個拖放對話 (dialogue) 設計工具，可協助您將對話式流程視覺化。 對話 (dialogue) 設計工具提供了七個「對話 (dialogue) 狀態」，讓您可用來建立對話 (conversation) 流程的模型。

## <a name="dialogue-states"></a>對話狀態

對話 (dialogue) 是由對話式狀態所組成的。 對話 (dialogue) 本身會將已結構化的導向式流程模型化，以便為如何執行對話式流程的結構提供對話 (conversation) 執行階段。

對話隨附許多您可以使用的內建狀態。 支援的內建狀態包含下列各項：

- [**開始**](#start-state)：代表對話式流程的開始狀態。 所有對話都必須至少定義一個開始狀態。
- [**返回**](#return-state)：表示特定的對話式流程完成。 因為對話式流程是可組合的，則返回會指示對話 (conversation) 執行階段，使執行返回此對話 (dialogue) 的任何可能呼叫者。
- [**決策**](#decision-state)：代表對話式流程中分支的一個點。
- [**處理**](#process-state)：代表 Bot 正在執行商務邏輯的狀態。
- [**提示**](#prompt-state)：代表您可以提示使用者輸入的狀態。 
- [**意見反應**](#feedback-state)：代表您可以為使用者提供意見反應或確認的狀態。 例如，確認已進行預訂的對話。
- [**模組**](#module-state)：代表呼叫另一個對話。 由於對話流程預設都是可組合的，因此，此狀態可以呼叫共用的對話，或是如此工作底下所定義的一些其他對話。

每個對話式狀態均會使用對話設計工具中的對話連接器來連線到另一個狀態。

每個對話狀態都有一個相關聯的狀態編輯器，可用來指定該狀態的屬性，包括自訂指令碼的回呼函式名稱。 **狀態編輯器**已設置為對話設計工具主要檢視區底部的調整大小窗格。 若要顯示編輯器，按兩下對話設計工具中的狀態，**狀態編輯器**將顯示該狀態的屬性。
<!-- TODO: insert screenshot of the wrench in horizontal menu -->

下列各小節提供更多關於這其中每個內建對話狀態的詳細資料。

### <a name="start-state"></a>開始狀態

開始狀態表示對話的起點。 必要值為狀態**名稱**。 名稱預設為「開始」，您可以編輯它來將此狀態重新命名。

### <a name="return-state"></a>返回狀態

返回狀態表示已完成對話式流程的特定分支。 由於對話 (dialogue) 是可組合的，因此，狀態也會指示對話 (conversation) 執行階段，使執行返回呼叫者對話 (dialogue)。 必要值為狀態**名稱**。 名稱預設為「返回」，您可以編輯它來將此狀態重新命名。

### <a name="decision-state"></a>決策狀態

決策狀態代表對話式流程中的分支。 您可以撰寫自訂指令碼，以評估要遵循哪一個分支。 根據使用者輸入和商務邏輯而定，指令碼將傳回其中一個可能的轉換值。 每個轉換值都會提示對話 (conversation) 執行階段來執行對話 (dialogue) 的不同分支。

決策狀態的必要屬性：
- **名稱**：決策狀態的唯一名稱。
- **在執行上執行的程式碼**：回呼函式的名稱，此函式會實作您的商務邏輯來判斷要採用對話的哪些分支。 

#### <a name="example-code-for-decision-state"></a>決策狀態的範例程式碼

下列範例回呼函式會傳回一個決策，指示對話執行階段要執行哪一個分支。

```javascript
module.exports.fnDecisionState = function(context) {
    var a = context.taskEntities['a'];
    if (a[0].value === '0') {
        return 'yes';
    }
    else if (a[0].value === '1') {
        return 'no';
    }
}
```

### <a name="process-state"></a>處理狀態

處理狀態表示對話 (dialogue) 中的一個點，此時 Bot 正在對話 (conversation) 中向前推進，或正嘗試執行最後的工作完成動作。 

處理狀態的必要屬性：
- **名稱**：處理狀態的唯一名稱
- **在執行上執行的程式碼**：實作您商務邏輯的回呼函式名稱。

#### <a name="example-code-for-process-state"></a>處理狀態的範例程式碼

下列範例回呼函式會取得氣象資訊，並將氣象資訊傳回給使用者。

```javascript
module.exports.fnGetWeather = function(context) {
    var options =  {
        host: 'mock',
        path: '/get?a=b',
        method: 'get'
    };
    return http.request(options).then(function(response) {
        context.contextEntities['x'].value= response.statusCode;
        var jsonBody = JSON.parse(response.body);
          // understand response
        if (response.statusCode != "200") {
            // error
        }
    });
}
```

### <a name="prompt-state"></a>提示狀態

提示狀態會要求使用者提供特定資訊。 提示狀態會在它們內部收錄一個子對話系統，因此根據定義會是複雜的狀態。 

在提示狀態內，您可以定義要提供給使用者的實際回應，且選擇性地包含調適型卡片。 您接著可以指定觸發程序來剖析和了解使用者的回應。 這個觸發程序可以是 LUIS 或使用 Regex 的自訂程式碼辨識器。  

如果使用者提供無效的輸入，Bot 可以針對相同的資訊重新提示使用者。 此行為也可以定義在提示狀態編輯器中。 

#### <a name="prompting-the-user"></a>提示使用者

提示回應可讓您指定要在**提示使用者**輸入時使用的訊息。 例如，若要收集日期和時間，可能的回應或許是「您想要何時入座？」 或者「您想要何時和我們一起用餐？」

#### <a name="prompt-listening-for-user-input"></a>提示傾聽使用者輸入

當系統提示使用者回應之後，對話執行階段將自動傾聽使用者輸入，並試著了解使用者說了什麼。 根據 LUIS 或自訂程式碼辨識器設定觸發程序，以嘗試了解使用者的輸入與意圖。 這類似於工作觸發程序。

#### <a name="re-prompting-the-user"></a>重新提示使用者

使用重新提示區段來指定每次嘗試的回應。 重新提示區段中的每一列均會對應至針對該特定回合所使用的重新提示字串。 第一個回應將用於第一次重新提示，第二個用於第二次，依此類推。 例如︰

「很抱歉，我不懂，您想要何時入座？」
「是我的錯，我很難理解您的意思。讓我們再試一次：您想要何時入座？」

#### <a name="prompt-callback-functions"></a>提示回呼函式

您可以在提示狀態上指定兩個不同的回呼函式。 

1. **在每個提示和重新提示之前**：在每個提示或重新提示之前執行此函式。 此回呼函式預期布林的傳回值，其中 True 表示執行此提示或重新提示，False 表示不會執行此提示或重新提示。 您可以使用 `getTurnIndex()` 來取得該提示執行目前的回合索引。
2. **正在回應**：每次產生提示時，但在將它傳送給使用者之前 (包括重新提示回應)，執行此函式。 這可讓指令碼修改傳送給使用者的訊息。

#### <a name="sample-code"></a>範例程式碼

此程式碼片段顯示**執行之前**回呼的範例。

```javascript
module.exports.fnBeforeExecuting = function(context) {
    if(context.responses[0].text === "C") {
        return false;
    }
    return true;
}
```

此程式碼片段顯示**正在回應**回呼的範例。

```javascript
module.exports.fnOnPrompting = function(context) {
    // include a hint card
    var activity = context.responses.slice(-1).pop();
    activity.attachments.push({
        "contentType": "application/vnd.microsoft.card.hero",
        "content": {
            "buttons": [
                {
                    "type": "imBack",
                    "title": "1",
                    "value": "1"
                },
                {
                    "type": "imBack",
                    "title": "2",
                    "value": "2"
                }
            ]
        }
    });
}
```

此程式碼片段顯示**重新提示之前**回呼的範例。

```javascript
module.exports.fnBeforeReprompting = function(context) {
    if(context.responses[0].text === "C") {
        return false;
    }
    return true;
}
```

### <a name="feedback-state"></a>意見反應狀態

使用此狀態來將回應提供回給使用者。 對於此狀態的典型使用案例包括在工作完成嘗試之後的最終成果，或者在失敗狀況下將回應提供回給使用者等情況。 

每個意見反應狀態都需要唯一的名稱、一些可能的回應值，而且可以選擇性地包含調適型卡片定義。 深入了解[調適型卡片定義](conversation-designer-adaptive-cards.md)。

每個意見反應狀態也允許包含**正在回應**回呼函式，您可以在其中撰寫自訂指令碼，以在傳送給使用者之前，視需要修改活動承載。 


```javascript
module.exports.fnOnResponding = function(context) {
    // include a hint card
    var activity = context.responses.slice(-1).pop();
    activity.attachments.push({
        "contentType": "application/vnd.microsoft.card.hero",
        "content": {
            "buttons": [
                {
                    "type": "imBack",
                    "title": "1",
                    "value": "1"
                },
                {
                    "type": "imBack",
                    "title": "2",
                    "value": "2"
                }
            ]
        }
    });

}
```

### <a name="module-state"></a>模組狀態

模組狀態是用來新增參考以執行子對話。 使用此狀態來將對話串連在一起。 

每個模組狀態都需要唯一的名稱，以及指向要執行之特定對話的指標。 適用於要執行之對話的可能選項必須存在於**共用對話**底下或特定的**工作**底下。

## <a name="multiple-dialogues-under-a-task"></a>一個工作底下的多個對話

一個工作可以有多個對話。 若要將對話新增到工作，只需選取工作，然後按一下左側樹狀目錄窗格中的 [新增] 按鈕，然後按一下 [新增對話]。 這將會在所選取的工作底下新增對話。 

由於對話是可組合的，因此，您可以將根對話流程繫結至會呼叫該工作底下之其他對話的工作。 這可讓您封裝子工作並能夠重複使用。 這也可讓您使用「模組」狀態，來將這些對話鏈結成對話式流程。

## <a name="next-step"></a>後續步驟
> [!div class="nextstepaction"]
> [回應範本](conversation-designer-response-templates.md)
