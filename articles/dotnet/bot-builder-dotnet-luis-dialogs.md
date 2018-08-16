---
title: 使用 LUIS 辨識意圖和實體 | Microsoft Docs
description: 了解如何在適用於 .NET 的 Bot Builder SDK 中使用 LUIS 對話，讓您的 Bot 了解自然語言。
author: DeniseMak
ms.author: v-demak
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: b98eea6bdae097beec85e93301e5380a1de991c3
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/18/2018
ms.locfileid: "39299886"
---
# <a name="recognize-intents-and-entities-with-luis"></a>使用 LUIS 辨識意圖和實體 

本文使用要用來記筆記的 Bot 範例，來示範 Language Understanding ([LUIS][LUIS]) 如何協助您的 Bot 適當地回應自然語言輸入。 Bot 會藉由識別使用者的**意圖**，來偵測他們想要做什麼。 此意圖是從語音或文字輸入，或是**語句**來判斷的。 此意圖會將語句對應到 Bot 所採取的動作。 例如，記筆記的 Bot 會辨識 `Notes.Create` 意圖，以叫用可用來建立筆記的功能。 Bot 可能也需要擷取**實體**，其為語句中的重要單字。 在記筆記的 Bot 範例中，`Notes.Title` 實體會識別每個筆記的標題。

## <a name="create-a-language-understanding-bot-with-bot-service"></a>使用 Bot 服務來建立 Language Understanding Bot

1. 在 [Azure 入口網站](https://portal.azure.com)中，選取功能表刀鋒視窗中的 [建立新資源]，然後按一下 [查看全部]。<!-- Start with the steps in [Create a bot with Bot Service](../bot-service-quickstart.md) to start creating a new bot service.  -->

    ![建立新資源](../media/bot-builder-dotnet-use-luis/bot-service-creation.png)

2. 在搜尋方塊中，搜尋 **Web 應用程式 Bot**。 

    ![建立新資源](../media/bot-builder-dotnet-use-luis/bot-service-selection.png)

3. 在 [Bot 服務] 刀鋒視窗中提供必要資訊，然後按一下 [建立]。 這會建立 Bot 服務和 LUIS 應用程式，並將其部署到 Azure。 
   * 將 [應用程式名稱] 設定為您 Bot 的名稱。 將 Bot 部署到雲端時，此名稱會用來作為子網域 (例如 mynotesbot.azurewebsites.net)。 此名稱也會用來作為與您 Bot 相關聯的 LUIS 應用程式名稱。 複製它以在稍後用來尋找與 Bot 相關聯的 LUIS 應用程式。
   * 選取訂用帳戶、[資源群組](/azure/azure-resource-manager/resource-group-overview)、App Service 方案，以及[位置](https://azure.microsoft.com/en-us/regions/)。
   * 針對 [Bot 範本] 欄位，選取 [Language Understanding (C#)] 範本。

     ![Bot 服務刀鋒視窗](../media/bot-builder-dotnet-use-luis/bot-service-setting-callout-template.png)

   * 選取方塊以確認服務條款。

4. 確認已部署 Bot 服務。
    * 按一下 [通知] (位於 Azure 入口網站頂端邊緣的鈴鐺圖示)。 通知會從 [部署已開始] 變更為 [部署成功]。
    * 在通知變更為 [部署成功] 之後，按一下該通知上的 [前往資源]。

## <a name="try-the-bot"></a>測試聊天機器人

勾選 [通知] 來確認已部署 Bot。 通知會從 [部署進行中] 變更為 [部署成功]。 按一下 [前往資源] 按鈕以開啟 Bot 的資源刀鋒視窗。

註冊 Bot 後，按一下 [在網路聊天中測試] 以開啟 [網路聊天] 窗格。 在網路聊天中輸入 "hello"。

  ![在網路聊天中測試 Bot](../media/bot-builder-dotnet-use-luis/bot-service-web-chat.png)

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

     ![LUIS 應用程式中所顯示的意圖](../media/bot-builder-dotnet-use-luis/luis-intent-list.png)

3. 按一下右上角的 [定型] 按鈕，來將您的應用程式定型。
4. 按一下上方導覽列中的 [發佈]，以開啟 [發佈] 頁面。 按一下 [發佈到生產位置] 按鈕。 成功發佈之後，複製 [發佈應用程式] 頁面上 [端點] 欄中所顯示的 URL，位於以資源名稱 Starter_Key 開頭的列中。 儲存此 URL，以便稍後在您的 Bot 程式碼中使用。 URL 的格式類似此範例：`https://westus.api.cognitive.microsoft.com/luis/v2.0/apps/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx?subscription-key=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx&timezoneOffset=0&verbose=true&q=`

## <a name="modify-the-bot-code"></a>修改 Bot 程式碼

按一下 [建置]，然後按一下 [開啟線上程式碼編輯器]。
    ![開啟線上程式碼編輯器](../media/bot-builder-dotnet-use-luis/bot-service-build.png)

在程式碼編輯器中，開啟 `BasicLuisDialog.cs`。 它包含下列程式碼，以處理來自 LUIS 應用程式的意圖。
```cs
using System;
using System.Configuration;
using System.Threading.Tasks;

using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Bot.Builder.Luis;
using Microsoft.Bot.Builder.Luis.Models;

namespace Microsoft.Bot.Sample.LuisBot
{
    // For more information about this template visit http://aka.ms/azurebots-csharp-luis
    [Serializable]
    public class BasicLuisDialog : LuisDialog<object>
    {
        public BasicLuisDialog() : base(new LuisService(new LuisModelAttribute(
            ConfigurationManager.AppSettings["LuisAppId"], 
            ConfigurationManager.AppSettings["LuisAPIKey"], 
            domain: ConfigurationManager.AppSettings["LuisAPIHostName"])))
        {
        }

        [LuisIntent("None")]
        public async Task NoneIntent(IDialogContext context, LuisResult result)
        {
            await this.ShowLuisResult(context, result);
        }

        // Go to https://luis.ai and create a new intent, then train/publish your luis app.
        // Finally replace "Greeting" with the name of your newly created intent in the following handler
        [LuisIntent("Greeting")]
        public async Task GreetingIntent(IDialogContext context, LuisResult result)
        {
            await this.ShowLuisResult(context, result);
        }

        [LuisIntent("Cancel")]
        public async Task CancelIntent(IDialogContext context, LuisResult result)
        {
            await this.ShowLuisResult(context, result);
        }

        [LuisIntent("Help")]
        public async Task HelpIntent(IDialogContext context, LuisResult result)
        {
            await this.ShowLuisResult(context, result);
        }

        private async Task ShowLuisResult(IDialogContext context, LuisResult result) 
        {
            await context.PostAsync($"You have reached {result.Intents[0].Intent}. You said: {result.Query}");
            context.Wait(MessageReceived);
        }
    }
}
```
### <a name="create-a-class-for-storing-notes"></a>建立用來儲存筆記的類別

在 BasicLuisDialog.cs 中新增下列 `using` 陳述式。

```cs
using System.Collections.Generic;
```

在建構函式定義之後，於 `BasicLuisDialog` 類別內新增下列程式碼。

```cs
        // Store notes in a dictionary that uses the title as a key
        private readonly Dictionary<string, Note> noteByTitle = new Dictionary<string, Note>();
        
        [Serializable]
        public sealed class Note : IEquatable<Note>
        {

            public string Title { get; set; }
            public string Text { get; set; }

            public override string ToString()
            {
                return $"[{this.Title} : {this.Text}]";
            }

            public bool Equals(Note other)
            {
                return other != null
                    && this.Text == other.Text
                    && this.Title == other.Title;
            }

            public override bool Equals(object other)
            {
                return Equals(other as Note);
            }

            public override int GetHashCode()
            {
                return this.Title.GetHashCode();
            }
        }

        // CONSTANTS        
        // Name of note title entity
        public const string Entity_Note_Title = "Note.Title";
        // Default note title
        public const string DefaultNoteTitle = "default";
```

### <a name="handle-the-notecreate-intent"></a>處理 Note.Create 意圖
若要處理 Note.Create 意圖，請將下列程式碼新增至 `BasicLuisDialog` 類別。

```cs
        private Note noteToCreate;
        private string currentTitle;
        [LuisIntent("Note.Create")]
        public Task NoteCreateIntent(IDialogContext context, LuisResult result)
        {
            EntityRecommendation title;
            if (!result.TryFindEntity(Entity_Note_Title, out title))
            {
                // Prompt the user for a note title
                PromptDialog.Text(context, After_TitlePrompt, "What is the title of the note you want to create?");
            }
            else
            {
                var note = new Note() { Title = title.Entity };
                noteToCreate = this.noteByTitle[note.Title] = note;

                // Prompt the user for what they want to say in the note           
                PromptDialog.Text(context, After_TextPrompt, "What do you want to say in your note?");
            }
            
            return Task.CompletedTask;
        }
        
        
        private async Task After_TitlePrompt(IDialogContext context, IAwaitable<string> result)
        {
            EntityRecommendation title;
            // Set the title (used for creation, deletion, and reading)
            currentTitle = await result;
            if (currentTitle != null)
            {
                title = new EntityRecommendation(type: Entity_Note_Title) { Entity = currentTitle };
            }
            else
            {
                // Use the default note title
                title = new EntityRecommendation(type: Entity_Note_Title) { Entity = DefaultNoteTitle };
            }

            // Create a new note object 
            var note = new Note() { Title = title.Entity };
            // Add the new note to the list of notes and also save it in order to add text to it later
            noteToCreate = this.noteByTitle[note.Title] = note;

            // Prompt the user for what they want to say in the note           
            PromptDialog.Text(context, After_TextPrompt, "What do you want to say in your note?");

        }

        private async Task After_TextPrompt(IDialogContext context, IAwaitable<string> result)
        {
            // Set the text of the note
            noteToCreate.Text = await result;
            
            await context.PostAsync($"Created note **{this.noteToCreate.Title}** that says \"{this.noteToCreate.Text}\".");
            
            context.Wait(MessageReceived);
        }
```

### <a name="handle-the-notereadaloud-intent"></a>處理 Note.ReadAloud 意圖
Bot 可以使用 `Note.ReadAloud` 意圖來顯示筆記的內容，如果未偵測到筆記標題，則會顯示所有筆記的內容。

將下列程式碼貼到 `BasicLuisDialog` 類別中。
```cs
        [LuisIntent("Note.ReadAloud")]
        public async Task NoteReadAloudIntent(IDialogContext context, LuisResult result)
        {
            Note note;
            if (TryFindNote(result, out note))
            {
                await context.PostAsync($"**{note.Title}**: {note.Text}.");
            }
            else
            {
                // Print out all the notes if no specific note name was detected
                string NoteList = "Here's the list of all notes: \n\n";
                foreach (KeyValuePair<string, Note> entry in noteByTitle)
                {
                    Note noteInList = entry.Value;
                    NoteList += $"**{noteInList.Title}**: {noteInList.Text}.\n\n";
                }
                await context.PostAsync(NoteList);
            }

            context.Wait(MessageReceived);
        }
        
        public bool TryFindNote(string noteTitle, out Note note)
        {
            bool foundNote = this.noteByTitle.TryGetValue(noteTitle, out note); // TryGetValue returns false if no match is found.
            return foundNote;
        }
        
        public bool TryFindNote(LuisResult result, out Note note)
        {
            note = null;

            string titleToFind;

            EntityRecommendation title;
            if (result.TryFindEntity(Entity_Note_Title, out title))
            {
                titleToFind = title.Entity;
            }
            else
            {
                titleToFind = DefaultNoteTitle;
            }

            return this.noteByTitle.TryGetValue(titleToFind, out note); // TryGetValue returns false if no match is found.
        }
```

### <a name="handle-the-notedelete-intent"></a>處理 Note.Delete 意圖
將下列程式碼貼到 `BasicLuisDialog` 類別中。

```cs
        [LuisIntent("Note.Delete")]
        public async Task NoteDeleteIntent(IDialogContext context, LuisResult result)
        {
            Note note;
            if (TryFindNote(result, out note))
            {
                this.noteByTitle.Remove(note.Title);
                await context.PostAsync($"Note {note.Title} deleted");
            }
            else
            {                             
                // Prompt the user for a note title
                PromptDialog.Text(context, After_DeleteTitlePrompt, "What is the title of the note you want to delete?");                         
            }           
        }

        private async Task After_DeleteTitlePrompt(IDialogContext context, IAwaitable<string> result)
        {
            Note note;
            string titleToDelete = await result;
            bool foundNote = this.noteByTitle.TryGetValue(titleToDelete, out note);

            if (foundNote)
            {
                this.noteByTitle.Remove(note.Title);
                await context.PostAsync($"Note {note.Title} deleted");
            }
            else
            {
                await context.PostAsync($"Did not find note named {titleToDelete}.");
            }

            context.Wait(MessageReceived);
        }
```

## <a name="build-the-bot"></a>建置 Bot
在程式碼編輯器中，以滑鼠右鍵按一下 **build.cmd**，然後選擇 [從主控台執行]。

   ![執行 build.cmd](../media/bot-builder-dotnet-use-luis/bot-service-run-console.png)

## <a name="test-the-bot"></a>測試 Bot

在 Azure 入口網站中，按一下 [在網路聊天中測試] 來測試 Bot。 請嘗試輸入像是「建立筆記」、「讀取我的筆記」和「刪除筆記」等訊息。
   ![在網路聊天中測試筆記 Bot](../media/bot-builder-dotnet-use-luis/bot-service-test-notebot.png)

> [!TIP]
> 如果您發現 Bot 並未總是辨識出正確的意圖或實體，請提供更多範例語句來進行 LUIS 應用程式定型，以改善應用程式的效能。 您無須對 Bot 程式碼進行任何修改，即可將 LUIS 應用程式重新定型。 請參閱[新增範例語句](/azure/cognitive-services/LUIS/add-example-utterances) \(英文\) 和[對您的 LUIS 應用程式進行定型和測試](/azure/cognitive-services/LUIS/train-test) \(英文\)。

> [!TIP]
> 如果您的 Bot 程式碼執行發生問題，請檢查下列各項：
> * 您必須[建置 Bot](./bot-builder-dotnet-luis-dialogs.md#build-the-bot)。
> * 您的 Bot 程式碼會在 LUIS 應用程式中，定義適用於每個意圖的處理常式。

## <a name="next-steps"></a>後續步驟

透過試用此 Bot，您可以查看 LUIS 意圖叫用工作的方式。 不過，這個簡單範例不允許中斷目前作用中的對話。 允許並處理中斷 (例如「說明」或「取消」) 是一項有彈性的設計，該設計會考量使用者真正執行的動作。 深入了解如何使用可評分的對話，如此就可讓您的對話處理中斷。

> [!div class="nextstepaction"]
> [使用可評分對話的全域訊息處理常式](bot-builder-dotnet-scorable-dialogs.md)

## <a name="additional-resources"></a>其他資源

- [對話](bot-builder-dotnet-dialogs.md)
- [使用對話 (dialogue) 管理對話 (conversation) 流程](bot-builder-dotnet-manage-conversation-flow.md)
- <a href="https://www.luis.ai" target="_blank">LUIS</a>
- <a href="/dotnet/api/?view=botbuilder-3.11.0" target="_blank">適用於 .NET 的 Bot Builder SDK 參考</a>

[LUIS]: https://www.luis.ai/
[NotesSample]: https://github.com/Microsoft/BotFramework-Samples/tree/master/docs-samples/CSharp/Simple-LUIS-Notes-Sample
[NotesSampleJSON]: https://github.com/Microsoft/BotFramework-Samples/blob/master/docs-samples/CSharp/Simple-LUIS-Notes-Sample/Notes.json
