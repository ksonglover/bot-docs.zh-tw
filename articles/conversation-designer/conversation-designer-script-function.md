---
title: 定義指令碼函式做為執行動作 | Microsoft Docs
description: 了解如何設定指令碼動作函式做為執行動作。
author: vkannan
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ROBOTS: NoIndex, NoFollow
ms.openlocfilehash: 674781c2cb5a8700941feb59b2ce7ab902979d4f
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/18/2018
ms.locfileid: "39300310"
---
# <a name="define-a-script-function-as-a-do-action"></a>定義指令碼函式做為執行動作

[指令碼動作函式](conversation-designer-context-object.md#script-callback-functions)執行您定義的自訂程式碼以協助您完成工作。 例如，呼叫服務，以便在使用者說「將我的控溫器設定為 74 度」時，實際將控溫器設定為 74 度。 

若要使用自訂指令碼做為動作，請選取**指令碼動作**做為工作編輯器中的**執行**動作類型。 然後輸入實作動作的函式名稱。 按一下 [**編輯**] 即可啟動**指令碼**編輯器並寫入此函式的實作。 

## <a name="script-action-function-parameter"></a>指令碼動作函式參數

指定的動作回呼函式將一律使用 [`context`](conversation-designer-context-object.md) 物件來呼叫。

`context` 物件同時包含 `taskEntities` 和 `contextEntities`。 工作實體是已針對此工作定義或產生的實體，而內容實體是屬性包，其中的實體可在與使用者交談期間加以保留。

## <a name="return-value"></a>傳回值
**指令碼動作**函式預期會傳回布林值。

## <a name="sample-script-action-function"></a>範例指令碼動作函式
下列範例函數可讓 HTTP 呼叫在傳回相符的觸發程序之前取得回應。

```javascript
module.exports.NewTask_do_onRun = function(context) {
    var options =  {
        host: 'HOST',
        path: 'PATH',
        method: 'post',
        headers: {
            "HEADER1" : "VALUE"
        }, 
        body: {
            "BODY": VALUE
        }
    };

    return http.request(options).then(function(response) {
      // parse response payload
      // Send a message to user
      context.responses.push({text: "Done", type: "message"});
    });
} 
```

## <a name="next-step"></a>後續步驟
> [!div class="nextstepaction"]
> [對話方塊動作](conversation-designer-dialogues.md)
