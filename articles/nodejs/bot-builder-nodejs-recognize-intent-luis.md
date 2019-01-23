---
title: 使用 LUIS 辨識意圖和實體 | Microsoft Docs
description: 將 Bot 與 LUIS 整合來偵測使用者的意圖，並使用適用於 Node.js 的 Bot Framework SDK 來觸發對話，藉以適當地回應。
author: DeniseMak
ms.author: v-demak
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 03/28/2018
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: acdc6053f7d666c2f086dca554efafc93c8af769
ms.sourcegitcommit: b15cf37afc4f57d13ca6636d4227433809562f8b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/11/2019
ms.locfileid: "54225283"
---
# <a name="recognize-intents-and-entities-with-luis"></a>使用 LUIS 辨識意圖和實體 

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

本文使用要用來記筆記的 Bot 範例，來示範 Language Understanding ([LUIS][LUIS]) 如何協助您的 Bot 適當地回應自然語言輸入。 Bot 會藉由識別使用者的**意圖**，來偵測他們想要做什麼。 此意圖是從語音或文字輸入，或是**語句**來判斷的。 此意圖會將語句對應到 Bot 所採取的動作，例如叫用對話。 Bot 可能也需要擷取**實體**，其為語句中的重要字組。 有時，實體必須滿足意圖。 在記筆記的 Bot 範例中，`Notes.Title` 實體會識別每個筆記的標題。

## <a name="create-a-language-understanding-bot-with-bot-service"></a>使用 Bot 服務來建立 Language Understanding Bot

1. 在 [Azure 入口網站](https://portal.azure.com)中，選取功能表刀鋒視窗中的 [建立新資源]，然後按一下 [查看全部]。

    ![建立新資源](../media/bot-builder-nodejs-use-luis/bot-service-creation.png)

2. 在搜尋方塊中，搜尋 **Web 應用程式 Bot**。 

    ![建立新資源](../media/bot-builder-nodejs-use-luis/bot-service-selection.png)

3. 在 [Bot 服務] 刀鋒視窗中提供必要資訊，然後按一下 [建立]。 這會建立 Bot 服務和 LUIS 應用程式，並將其部署到 Azure。 
   * 將 [應用程式名稱] 設定為您 Bot 的名稱。 將 Bot 部署到雲端時，此名稱會用來作為子網域 (例如 mynotesbot.azurewebsites.net)。 此名稱也會用來作為與您 Bot 相關聯的 LUIS 應用程式名稱。 複製它以在稍後用來尋找與 Bot 相關聯的 LUIS 應用程式。
   * 選取訂用帳戶、[資源群組](/azure/azure-resource-manager/resource-group-overview)、App Service 方案，以及[位置](https://azure.microsoft.com/en-us/regions/)。
   * 針對 [Bot 範本] 欄位，選取 [Language Understanding (Node.js)] 範本。

     ![Bot 服務刀鋒視窗](../media/bot-builder-nodejs-use-luis/bot-service-setting-callout-template.png)

   * 選取方塊以確認服務條款。

4. 確認已部署 Bot 服務。
    * 按一下 [通知] (位於 Azure 入口網站頂端邊緣的鈴鐺圖示)。 通知會從 [部署已開始] 變更為 [部署成功]。
    * 在通知變更為 [部署成功] 之後，按一下該通知上的 [前往資源]。

## <a name="try-the-bot"></a>測試聊天機器人

勾選 [通知] 來確認已部署 Bot。 通知會從 [部署進行中] 變更為 [部署成功]。 按一下 [前往資源] 按鈕以開啟 Bot 的資源刀鋒視窗。

註冊 Bot 後，按一下 [在網路聊天中測試] 以開啟 [網路聊天] 窗格。 在網路聊天中輸入 "hello"。

  ![在網路聊天中測試 Bot](../media/bot-builder-nodejs-use-luis/bot-service-web-chat.png)

Bot 會說出 "You have reached Greeting. You said: hello" 來作為回應。 這確認了 Bot 已收到您的訊息，並已將其傳遞給它所建立的預設 LUIS 應用程式。 此預設 LUIS 應用程式偵測到 Greeting 意圖。

## <a name="modify-the-luis-app"></a>修改 LUIS 應用程式

使用您用來登入 Azure 的相同帳戶登入 [https://www.luis.ai](https://www.luis.ai)。 按一下 [My apps] \(我的應用程式\)。 在應用程式清單中，尋找開頭為當您建立 Bot 服務時，於 [Bot 服務] 刀鋒視窗的 [應用程式名稱] 中指定之名稱的應用程式。 

LUIS 應用程式會以下列 4 個意圖作為開頭：Cancel、Greeting、Help 及 None。 <!-- picture -->

下列步驟會新增 Note.Create、Note.ReadAloud 和 Note.Delete 意圖： 

1. 在頁面左下角按一下 [預先建置的網域]。 尋找 **Note** 網域，然後按一下 [新增網域]。

2. 本教學課程不會使用 **Note** 預先建置的網域中包含的所有意圖。 在 [意圖] 頁面上，按一下以下每一個意圖名稱，然後按一下 [刪除意圖] 按鈕。
   * Note.ShowNext
   * Note.DeleteNoteItem
   * Note.Confirm
   * Note.Clear
   * Note.CheckOffItem
   * Note.AddToNote

   以下為只應保留在 LUIS 應用程式中的意圖： 
   * Note.ReadAloud
   * Note.Create
   * Note.Delete
   * None
   * 說明
   * Greeting
   * 取消

     ![LUIS 應用程式中所顯示的意圖](../media/bot-builder-nodejs-use-luis/luis-intent-list.png)


3.  按一下右上角的 [定型] 按鈕，來將您的應用程式定型。
4.  按一下上方導覽列中的 [發佈]，以開啟 [發佈] 頁面。 按一下 [發佈到生產位置] 按鈕。 成功發佈之後，即會將 LUIS 應用程式部署到 [發佈應用程式] 頁面上 [端點] 欄中所顯示的 URL，位於以資源名稱 Starter_Key 開頭的列中。 URL 的格式類似此範例：`https://westus.api.cognitive.microsoft.com/luis/v2.0/apps/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx?subscription-key=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx&timezoneOffset=0&verbose=true&q=`。 此 URL 中的應用程式識別碼和訂用帳戶金鑰會與 [App Service 設定] > [ApplicationSettings] > [應用程式設定]**** 中的 LuisAppId 和 LuisAPIKey 相同。


## <a name="modify-the-bot-code"></a>修改 Bot 程式碼

按一下 [建置]，然後按一下 [開啟線上程式碼編輯器]。

   ![開啟線上程式碼編輯器](../media/bot-builder-nodejs-use-luis/bot-service-build.png)

在程式碼編輯器中，開啟 `app.js`。 其中包含下列程式碼：

```javascript
var restify = require('restify');
var builder = require('botbuilder');
var botbuilder_azure = require("botbuilder-azure");

// Setup Restify Server
var server = restify.createServer();
server.listen(process.env.port || process.env.PORT || 3978, function () {
   console.log('%s listening to %s', server.name, server.url); 
});
  
// Create chat connector for communicating with the Bot Framework Service
var connector = new builder.ChatConnector({
    appId: process.env.MicrosoftAppId,
    appPassword: process.env.MicrosoftAppPassword,
    openIdMetadata: process.env.BotOpenIdMetadata 
});

// Listen for messages from users 
server.post('/api/messages', connector.listen());

/*----------------------------------------------------------------------------------------
* Bot Storage: This is a great spot to register the private state storage for your bot. 
* We provide adapters for Azure Table, CosmosDb, SQL Azure, or you can implement your own!
* For samples and documentation, see: https://github.com/Microsoft/BotBuilder-Azure
* ---------------------------------------------------------------------------------------- */

var tableName = 'botdata';
var azureTableClient = new botbuilder_azure.AzureTableClient(tableName, process.env['AzureWebJobsStorage']);
var tableStorage = new botbuilder_azure.AzureBotStorage({ gzipData: false }, azureTableClient);

// Create your bot with a function to receive messages from the user
// This default message handler is invoked if the user's utterance doesn't
// match any intents handled by other dialogs.
var bot = new builder.UniversalBot(connector, function (session, args) {
    session.send('You reached the default message handler. You said \'%s\'.', session.message.text);
});

bot.set('storage', tableStorage);

// Make sure you add code to validate these fields
var luisAppId = process.env.LuisAppId;
var luisAPIKey = process.env.LuisAPIKey;
var luisAPIHostName = process.env.LuisAPIHostName || 'westus.api.cognitive.microsoft.com';

const LuisModelUrl = 'https://' + luisAPIHostName + '/luis/v2.0/apps/' + luisAppId + '?subscription-key=' + luisAPIKey;

// Create a recognizer that gets intents from LUIS, and add it to the bot
var recognizer = new builder.LuisRecognizer(LuisModelUrl);
bot.recognizer(recognizer);

// Add a dialog for each intent that the LUIS app recognizes.
// See https://docs.microsoft.com/en-us/bot-framework/nodejs/bot-builder-nodejs-recognize-intent-luis 
bot.dialog('GreetingDialog',
    (session) => {
        session.send('You reached the Greeting intent. You said \'%s\'.', session.message.text);
        session.endDialog();
    }
).triggerAction({
    matches: 'Greeting'
})

bot.dialog('HelpDialog',
    (session) => {
        session.send('You reached the Help intent. You said \'%s\'.', session.message.text);
        session.endDialog();
    }
).triggerAction({
    matches: 'Help'
})

bot.dialog('CancelDialog',
    (session) => {
        session.send('You reached the Cancel intent. You said \'%s\'.', session.message.text);
        session.endDialog();
    }
).triggerAction({
    matches: 'Cancel'
})

```


> [!TIP] 
> 您也可以在 [Notes Bot 範例][NotesSample]中找到本文所述的範例程式碼。



## <a name="edit-the-default-message-handler"></a>編輯預設訊息處理常式
Bot 具有預設訊息處理常式。 編輯它以符合下列內容： 
```javascript
// Create your bot with a function to receive messages from the user.
// This default message handler is invoked if the user's utterance doesn't
// match any intents handled by other dialogs.
var bot = new builder.UniversalBot(connector, function (session, args) {
    session.send("Hi... I'm the note bot sample. I can create new notes, read saved notes to you and delete notes.");

   // If the object for storing notes in session.userData doesn't exist yet, initialize it
   if (!session.userData.notes) {
       session.userData.notes = {};
       console.log("initializing userData.notes in default message handler");
   }
});
```

## <a name="handle-the-notecreate-intent"></a>處理 Note.Create 意圖

複製下列程式碼，並將它貼在 app.js 結尾：

```javascript
// CreateNote dialog
bot.dialog('CreateNote', [
    function (session, args, next) {
        // Resolve and store any Note.Title entity passed from LUIS.
        var intent = args.intent;
        var title = builder.EntityRecognizer.findEntity(intent.entities, 'Note.Title');

        var note = session.dialogData.note = {
          title: title ? title.entity : null,
        };
        
        // Prompt for title
        if (!note.title) {
            builder.Prompts.text(session, 'What would you like to call your note?');
        } else {
            next();
        }
    },
    function (session, results, next) {
        var note = session.dialogData.note;
        if (results.response) {
            note.title = results.response;
        }

        // Prompt for the text of the note
        if (!note.text) {
            builder.Prompts.text(session, 'What would you like to say in your note?');
        } else {
            next();
        }
    },
    function (session, results) {
        var note = session.dialogData.note;
        if (results.response) {
            note.text = results.response;
        }
        
        // If the object for storing notes in session.userData doesn't exist yet, initialize it
        if (!session.userData.notes) {
            session.userData.notes = {};
            console.log("initializing session.userData.notes in CreateNote dialog");
        }
        // Save notes in the notes object
        session.userData.notes[note.title] = note;

        // Send confirmation to user
        session.endDialog('Creating note named "%s" with text "%s"',
            note.title, note.text);
    }
]).triggerAction({ 
    matches: 'Note.Create',
    confirmPrompt: "This will cancel the creation of the note you started. Are you sure?" 
}).cancelAction('cancelCreateNote', "Note canceled.", {
    matches: /^(cancel|nevermind)/i,
    confirmPrompt: "Are you sure?"
});
```

語句中的任何實體都會使用 `args` 參數來傳遞給對話。 [瀑布圖][ waterfall]的第一個步驟會呼叫 [EntityRecognizer.findEntity][EntityRecognizer_findEntity]，從 LUIS 回應中的任何 `Note.Title` 實體取得筆記的標題。 如果 LUIS 應用程式未偵測到 `Note.Title` 實體，Bot 會提示使用者輸入筆記的名稱。 瀑布圖的第二個步驟會提示您輸入要包含於筆記中的文字。 一旦 Bot 擁有筆記的文字之後，第三個步驟就會使用 [session.userData][session_userData]，利用標題作為索引鍵，將筆記儲存於 `notes` 物件中。 如需關於 `session.UserData` 的詳細資訊，請參閱[管理狀態資料](./bot-builder-nodejs-state.md)。 



## <a name="handle-the-notedelete-intent"></a>處理 Note.Delete 意圖
Bot 只會針對 `Note.Create` 意圖檢查標題的 `args` 參數。 如果未偵測到任何標題，Bot 就會提示使用者。 標題可用來查詢要從 `session.userData.notes` 刪除的筆記。 



複製下列程式碼，並將它貼在 app.js 結尾：
```javascript
// Delete note dialog
bot.dialog('DeleteNote', [
    function (session, args, next) {
        if (noteCount(session.userData.notes) > 0) {
            // Resolve and store any Note.Title entity passed from LUIS.
            var title;
            var intent = args.intent;
            var entity = builder.EntityRecognizer.findEntity(intent.entities, 'Note.Title');
            if (entity) {
                // Verify that the title is in our set of notes.
                title = builder.EntityRecognizer.findBestMatch(session.userData.notes, entity.entity);
            }
            
            // Prompt for note name
            if (!title) {
                builder.Prompts.choice(session, 'Which note would you like to delete?', session.userData.notes);
            } else {
                next({ response: title });
            }
        } else {
            session.endDialog("No notes to delete.");
        }
    },
    function (session, results) {
        delete session.userData.notes[results.response.entity];        
        session.endDialog("Deleted the '%s' note.", results.response.entity);
    }
]).triggerAction({
    matches: 'Note.Delete'
}).cancelAction('cancelDeleteNote', "Ok - canceled note deletion.", {
    matches: /^(cancel|nevermind)/i
});
```

處理 `Note.Delete` 的程式碼會使用 `noteCount` 函式，來判斷 `notes` 物件是否包含筆記。 

將 `noteCount` 協助程式函式貼到 `app.js` 結尾。

[!code-js[Add a helper function that returns the number of notes (JavaScript)](../includes/code/node-basicNote.js#CountNotesHelper)]

## <a name="handle-the-notereadaloud-intent"></a>處理 Note.ReadAloud 意圖

複製下列程式碼並貼到 `app.js` 中的 `Note.Delete` 處理常式之後：

```javascript
// Read note dialog
bot.dialog('ReadNote', [
    function (session, args, next) {
        if (noteCount(session.userData.notes) > 0) {
           
            // Resolve and store any Note.Title entity passed from LUIS.
            var title;
            var intent = args.intent;
            var entity = builder.EntityRecognizer.findEntity(intent.entities, 'Note.Title');
            if (entity) {
                // Verify it's in our set of notes.
                title = builder.EntityRecognizer.findBestMatch(session.userData.notes, entity.entity);
            }
            
            // Prompt for note name
            if (!title) {
                builder.Prompts.choice(session, 'Which note would you like to read?', session.userData.notes);
            } else {
                next({ response: title });
            }
        } else {
            session.endDialog("No notes to read.");
        }
    },
    function (session, results) {        
        session.endDialog("Here's the '%s' note: '%s'.", results.response.entity, session.userData.notes[results.response.entity].text);
    }
]).triggerAction({
    matches: 'Note.ReadAloud'
}).cancelAction('cancelReadNote', "Ok.", {
    matches: /^(cancel|nevermind)/i
});

```

`session.userData.notes` 物件會當成第三個引數傳遞給 `builder.Prompts.choice`，如此一來，提示就會向使用者顯示筆記清單。

您現在已新增適用於新意圖的處理常式，`app.js` 的完整程式碼會包含下列內容：

```javascript
var restify = require('restify');
var builder = require('botbuilder');
var botbuilder_azure = require("botbuilder-azure");

// Setup Restify Server
var server = restify.createServer();
server.listen(process.env.port || process.env.PORT || 3978, function () {
   console.log('%s listening to %s', server.name, server.url); 
});
  
// Create chat connector for communicating with the Bot Framework Service
var connector = new builder.ChatConnector({
    appId: process.env.MicrosoftAppId,
    appPassword: process.env.MicrosoftAppPassword,
    openIdMetadata: process.env.BotOpenIdMetadata 
});

// Listen for messages from users 
server.post('/api/messages', connector.listen());

/*----------------------------------------------------------------------------------------
* Bot Storage: This is a great spot to register the private state storage for your bot. 
* We provide adapters for Azure Table, CosmosDb, SQL Azure, or you can implement your own!
* For samples and documentation, see: https://github.com/Microsoft/BotBuilder-Azure
* ---------------------------------------------------------------------------------------- */

var tableName = 'botdata';
var azureTableClient = new botbuilder_azure.AzureTableClient(tableName, process.env['AzureWebJobsStorage']);
var tableStorage = new botbuilder_azure.AzureBotStorage({ gzipData: false }, azureTableClient);

// Create your bot with a function to receive messages from the user.
// This default message handler is invoked if the user's utterance doesn't
// match any intents handled by other dialogs.
var bot = new builder.UniversalBot(connector, function (session, args) {
    session.send("Hi... I'm the note bot sample. I can create new notes, read saved notes to you and delete notes.");

   // If the object for storing notes in session.userData doesn't exist yet, initialize it
   if (!session.userData.notes) {
       session.userData.notes = {};
       console.log("initializing userData.notes in default message handler");
   }
});

bot.set('storage', tableStorage);

// Make sure you add code to validate these fields
var luisAppId = process.env.LuisAppId;
var luisAPIKey = process.env.LuisAPIKey;
var luisAPIHostName = process.env.LuisAPIHostName || 'westus.api.cognitive.microsoft.com';

const LuisModelUrl = 'https://' + luisAPIHostName + '/luis/v2.0/apps/' + luisAppId + '?subscription-key=' + luisAPIKey;

// Create a recognizer that gets intents from LUIS, and add it to the bot
var recognizer = new builder.LuisRecognizer(LuisModelUrl);
bot.recognizer(recognizer);

// CreateNote dialog
bot.dialog('CreateNote', [
    function (session, args, next) {
        // Resolve and store any Note.Title entity passed from LUIS.
        var intent = args.intent;
        var title = builder.EntityRecognizer.findEntity(intent.entities, 'Note.Title');

        var note = session.dialogData.note = {
          title: title ? title.entity : null,
        };
        
        // Prompt for title
        if (!note.title) {
            builder.Prompts.text(session, 'What would you like to call your note?');
        } else {
            next();
        }
    },
    function (session, results, next) {
        var note = session.dialogData.note;
        if (results.response) {
            note.title = results.response;
        }

        // Prompt for the text of the note
        if (!note.text) {
            builder.Prompts.text(session, 'What would you like to say in your note?');
        } else {
            next();
        }
    },
    function (session, results) {
        var note = session.dialogData.note;
        if (results.response) {
            note.text = results.response;
        }
        
        // If the object for storing notes in session.userData doesn't exist yet, initialize it
        if (!session.userData.notes) {
            session.userData.notes = {};
            console.log("initializing session.userData.notes in CreateNote dialog");
        }
        // Save notes in the notes object
        session.userData.notes[note.title] = note;

        // Send confirmation to user
        session.endDialog('Creating note named "%s" with text "%s"',
            note.title, note.text);
    }
]).triggerAction({ 
    matches: 'Note.Create',
    confirmPrompt: "This will cancel the creation of the note you started. Are you sure?" 
}).cancelAction('cancelCreateNote', "Note canceled.", {
    matches: /^(cancel|nevermind)/i,
    confirmPrompt: "Are you sure?"
});

// Delete note dialog
bot.dialog('DeleteNote', [
    function (session, args, next) {
        if (noteCount(session.userData.notes) > 0) {
            // Resolve and store any Note.Title entity passed from LUIS.
            var title;
            var intent = args.intent;
            var entity = builder.EntityRecognizer.findEntity(intent.entities, 'Note.Title');
            if (entity) {
                // Verify that the title is in our set of notes.
                title = builder.EntityRecognizer.findBestMatch(session.userData.notes, entity.entity);
            }
            
            // Prompt for note name
            if (!title) {
                builder.Prompts.choice(session, 'Which note would you like to delete?', session.userData.notes);
            } else {
                next({ response: title });
            }
        } else {
            session.endDialog("No notes to delete.");
        }
    },
    function (session, results) {
        delete session.userData.notes[results.response.entity];        
        session.endDialog("Deleted the '%s' note.", results.response.entity);
    }
]).triggerAction({
    matches: 'Note.Delete'
}).cancelAction('cancelDeleteNote', "Ok - canceled note deletion.", {
    matches: /^(cancel|nevermind)/i
});


// Read note dialog
bot.dialog('ReadNote', [
    function (session, args, next) {
        if (noteCount(session.userData.notes) > 0) {
           
            // Resolve and store any Note.Title entity passed from LUIS.
            var title;
            var intent = args.intent;
            var entity = builder.EntityRecognizer.findEntity(intent.entities, 'Note.Title');
            if (entity) {
                // Verify it's in our set of notes.
                title = builder.EntityRecognizer.findBestMatch(session.userData.notes, entity.entity);
            }
            
            // Prompt for note name
            if (!title) {
                builder.Prompts.choice(session, 'Which note would you like to read?', session.userData.notes);
            } else {
                next({ response: title });
            }
        } else {
            session.endDialog("No notes to read.");
        }
    },
    function (session, results) {        
        session.endDialog("Here's the '%s' note: '%s'.", results.response.entity, session.userData.notes[results.response.entity].text);
    }
]).triggerAction({
    matches: 'Note.ReadAloud'
}).cancelAction('cancelReadNote', "Ok.", {
    matches: /^(cancel|nevermind)/i
});


// Helper function to count the number of notes stored in session.userData.notes
function noteCount(notes) {

    var i = 0;
    for (var name in notes) {
        i++;
    }
    return i;
}
```

## <a name="test-the-bot"></a>測試 Bot

在 Azure 入口網站中，按一下 [在網路聊天中測試] 來測試 Bot。 試著輸入「建立筆記」、「讀取我的筆記」和「刪除筆記」之類的訊息，來叫用您想要新增到其中的意圖。
   ![在網路聊天中測試筆記 Bot](../media/bot-builder-nodejs-use-luis/bot-service-test-notebot.png)

> [!TIP]
> 如果您發現 Bot 並未總是辨識出正確的意圖或實體，請提供更多範例語句來進行 LUIS 應用程式定型，以改善應用程式的效能。 您無須對 Bot 程式碼進行任何修改，即可將 LUIS 應用程式重新定型。 請參閱[新增範例語句](/azure/cognitive-services/LUIS/add-example-utterances) \(英文\) 和[對您的 LUIS 應用程式進行定型和測試](/azure/cognitive-services/LUIS/train-test) \(英文\)。


## <a name="next-steps"></a>後續步驟

試用 Bot 之後，您會看到辨識器可以觸發中斷目前作用中的對話。 允許並處理中斷是一項有彈性的設計，該設計會考量使用者真正執行的動作。 深入了解您可與已辨識意圖相關聯的各種動作。

> [!div class="nextstepaction"]
> [處理使用者動作](bot-builder-nodejs-dialog-actions.md)


[LUIS]: https://www.luis.ai/

[intentDialog]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.intentdialog.html

[intentDialog_matches]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.intentdialog.html#matches 

[NotesSample]: https://github.com/Microsoft/BotFramework-Samples/tree/master/docs-samples/Node/basics-naturalLanguage

[triggerAction]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.dialog.html#triggeraction

[confirmPrompt]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.itriggeractionoptions.html#confirmprompt

[waterfall]: bot-builder-nodejs-dialog-manage-conversation-flow.md#manage-conversation-flow-with-a-waterfall

[session_userData]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session.html#userdata

[EntityRecognizer_findEntity]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.entityrecognizer.html#findentity

[matches]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.itriggeractionoptions.html#matches

[LUISAzureDocs]: /azure/cognitive-services/LUIS/Home

[Dialog]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.dialog.html

[IntentRecognizerSetOptions]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.iintentrecognizersetoptions.html

[LuisRecognizer]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.luisrecognizer

[LUISConcepts]: https://docs.botframework.com/en-us/node/builder/guides/understanding-natural-language/

[DisambiguationSample]: https://aka.ms/v3-js-onDisambiguateRoute

[IDisambiguateRouteHandler]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.idisambiguateroutehandler.html

[RegExpRecognizer]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.regexprecognizer.html

[AlarmBot]: https://aka.ms/v3-js-luisSample

[UniversalBot]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.universalbot.html
