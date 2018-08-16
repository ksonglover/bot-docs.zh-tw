---
title: 教學課程 - 儲存使用者狀態資料 | Microsoft Docs
description: 了解如何在 Bot Builder SDK 中儲存使用者狀態資料。
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 4/23/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 10c1cb240a22c1c16dd0d946ee55531d514f332e
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/18/2018
ms.locfileid: "39299859"
---
# <a name="save-user-state-data"></a>儲存使用者狀態資料

當 Bot 要求使用者輸入時，您可能會想要以某些形式的儲存體保存某些資訊。 Bot Builder SDK 可讓您使用*記憶體內部儲存體*、*檔案儲存體*、資料庫儲存體 (*CosmosDB* 或 *SQL*) 來儲存使用者輸入。 

此教學課程說明如何在中介軟體層中定義您的儲存體物件，以及如何將使用者輸入儲存到該儲存體物件中。

## <a name="prequisite"></a>先決條件 

此教學課程是以[使用瀑布圖管理交談流程](bot-builder-tutorial-waterfall.md)教學課程為基礎而建立。

## <a name="add-storage-to-middleware-layer"></a>將儲存體新增至中介軟體層


## <a name="save-user-input-to-storage"></a>將使用者輸入儲存至儲存體

若要管理訂位交談內容，您將需要使用四個步驟來定義**瀑布**對話。 在此交談中，除了 `TextPrompt` 之外，您也將會使用 `DatetimePrompt` 和 `NumberPrompt`。

`reserveTable` 對話看起來像這樣：

```javascript
// Reserve a table:
// Help the user to reserve a table

var reservationInfo = {
    dateTime: '',
    partySize: '',
    reserveName: ''
}

dialogs.add('reserveTable', [
    async function(dc, args, next){
        await dc.context.sendActivity("Welcome to the reservation service.");
        reservationInfo = {}; // Clears any previous data
        await dc.prompt('dateTimePrompt', "Please provide a reservation date and time.");
    },
    async function(dc, result){
        reservationInfo.dateTime = result[0].value;

        // Ask for next info
        await dc.prompt('partySizePrompt', "How many people are in your party?");
    },
    async function(dc, result){
        reservationInfo.partySize = result;

        // Ask for next info
        await dc.prompt('textPrompt', "Who's name will this be under?");
    },
    async function(dc, result){
        reservationInfo.reserveName = result;

        // Confirm reservation
        var msg = `Reservation confirmed. Reservation details: 
            <br/>Date/Time: ${reservationInfo.dateTime} 
            <br/>Party size: ${reservationInfo.partySize} 
            <br/>Reservation name: ${reservationInfo.reserveName}`;
        await dc.context.sendActivity(msg);
        await dc.end();
    }
]);

```

現在，您已準備好將此連結到 Bot 邏輯。

## <a name="start-the-dialog"></a>開始對話

將您的 Bot 的 `processActivity()` 方法修改為下面這樣：

```javascript
// Listen for incoming activity 
server.post('/api/messages', (req, res) => {
    adapter.processActivity(req, res, async (context) => {
        if (context.activity.type === 'message') {
            // State will store all of your information 
            const state = conversationState.get(context);
            const dc = dialogs.createContext(context, state);

            if(context.activity.text.match(/hi/ig)){
                await dc.begin('greeting');
            }
            if(context.activity.text.match(/reserve table/ig)){
                await dc.begin('reserveTable');
            }
            else{
                // Continue executing the "current" dialog, if any.
                await dc.continue();
            }
        }
    });
});
```

在執行時，每當使用者傳送包含字串 `reserve table` 的訊息時，Bot 就會開始 `reserveTable` 交談。

## <a name="next-steps"></a>後續步驟

??? 

> [!div class="nextstepaction"]
> [儲存使用者狀態資料](bot-builder-tutorial-save-data.md)
