---
title: 傳送主動訊息 | Microsoft Docs
description: 了解如何使用適用於 .NET 的 Bot Framework SDK 來傳送主動式訊息。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: c601db171f253b83bfa2d354f79018f03287bcf6
ms.sourcegitcommit: a6d02ec4738e7fc90b7108934740e9077667f3c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/04/2019
ms.locfileid: "70297360"
---
# <a name="send-proactive-messages"></a>傳送主動訊息

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-proactive-messages.md)
> - [Node.js](../nodejs/bot-builder-nodejs-proactive-messages.md)

[!INCLUDE [Introduction to proactive messages - part 1](../includes/snippet-proactive-messages-intro-1.md)]

## <a name="types-of-proactive-messages"></a>主動式訊息的類型 

[!INCLUDE [Introduction to proactive messages - part 2](../includes/snippet-proactive-messages-intro-2.md)]

## <a name="send-an-ad-hoc-proactive-message"></a>傳送臨機操作的主動訊息

下列程式碼範例示範如何使用適用於 .NET 的 Bot Framework SDK 傳送臨機操作的主動式訊息。

為了能夠將臨機操作的訊息傳送給使用者，Bot 必須先收集並儲存一些來自目前對話的使用者相關資訊。 

```cs
public virtual async Task MessageReceivedAsync(IDialogContext context, IAwaitable<IMessageActivity> result)
{
    var message = await result;
    
    // Extract data from the user's message that the bot will need later to send an ad hoc message to the user. 
    // Store the extracted data in a custom class "ConversationStarter" (not shown here).

    ConversationStarter.toId = message.From.Id;
    ConversationStarter.toName = message.From.Name;
    ConversationStarter.fromId = message.Recipient.Id;
    ConversationStarter.fromName = message.Recipient.Name;
    ConversationStarter.serviceUrl = message.ServiceUrl;
    ConversationStarter.channelId = message.ChannelId;
    ConversationStarter.conversationId = message.Conversation.Id;

    // (Save this information somewhere that it can be accessed later, such as in a database.)

    await context.PostAsync("Hello user, good to meet you! I now know your address and can send you notifications in the future.");
    context.Wait(MessageReceivedAsync);
}
```
> [!NOTE]
> 為了簡單起見，此範例並未指定如何儲存使用者資料。 如何儲存資料並不重要，只要 Bot 稍後可擷取資料即可。

既然資料已儲存，Bot 只需擷取資料、建構臨機操作的主動式訊息，然後加以傳送。 

```cs
// Use the data stored previously to create the required objects.
var userAccount = new ChannelAccount(toId,toName);
var botAccount = new ChannelAccount(fromId, fromName);
var connector = new ConnectorClient(new Uri(serviceUrl));

// Create a new message.
IMessageActivity message = Activity.CreateMessageActivity();
if (!string.IsNullOrEmpty(conversationId) && !string.IsNullOrEmpty(channelId))  
{
    // If conversation ID and channel ID was stored previously, use it.
    message.ChannelId = channelId;
}
else
{
    // Conversation ID was not stored previously, so create a conversation. 
    // Note: If the user has an existing conversation in a channel, this will likely create a new conversation window.
    conversationId = (await connector.Conversations.CreateDirectConversationAsync( botAccount, userAccount)).Id;
}

// Set the address-related properties in the message and send the message.
message.From = botAccount;
message.Recipient = userAccount;
message.Conversation = new ConversationAccount(id: conversationId);
message.Text = "Hello, this is a notification";
message.Locale = "en-us";
await connector.Conversations.SendToConversationAsync((Activity)message);
```

> [!NOTE]
> 如果 Bot 指定先前儲存的對話識別碼，很可能會將訊息傳遞給用戶端上現有對話視窗中的使用者。 如果 Bot 產生新的對話識別碼，即會將訊息傳遞給用戶端上新對話視窗中的使用者，但前提是用戶端支援多個對話視窗。 

## <a name="send-a-dialog-based-proactive-message"></a>傳送對話方塊式的主動訊息

下列程式碼範例示範如何使用適用於 .NET 的 Bot Framework SDK 傳送對話式主動訊息。

為了能夠將對話型的主動式訊息傳送給使用者，Bot 必須先收集並儲存來自目前對話的相關資訊。 

```cs
public virtual async Task MessageReceivedAsync(IDialogContext context, IAwaitable<IMessageActivity> result)
{
    var message = await result;
    
    // Store information about this specific point the conversation, so that the bot can resume this conversation later.
    var conversationReference = message.ToConversationReference();
    ConversationStarter.conversationReference = JsonConvert.SerializeObject(conversationReference);

    await context.PostAsync("Greetings, user! I now know how to start a proactive message to you."); 
    context.Wait(MessageReceivedAsync);
}
```

傳送訊息時，Bot 會建立新的對話，並將它新增至對話堆疊的最上層。 新的對話會取得對話的控制權、傳遞主動式訊息、關閉，然後將控制權傳回給堆疊中的上一個對話。 

```cs
// This will interrupt the conversation and send the user to SurveyDialog, then wait until that's done 
public static async Task Resume() 
{
    // Recreate the message from the conversation reference that was saved previously.
    var message = JsonConvert.DeserializeObject<ConversationReference>(conversationReference).GetPostToBotMessage(); 
    var client = new ConnectorClient(new Uri(message.ServiceUrl));

    // Create a scope that can be used to work with state from bot framework.
    using (var scope = DialogModule.BeginLifetimeScope(Conversation.Container, message))
    {
        var botData = scope.Resolve<IBotData>();
        await botData.LoadAsync(CancellationToken.None);

        // This is our dialog stack.
        var task = scope.Resolve<IDialogTask>();

        // Create the new dialog and add it to the stack.
        var dialog = new SurveyDialog();
        // interrupt the stack. This means that we're stopping whatever conversation that is currently happening with the user
        // Then adding this stack to run and once it's finished, we will be back to the original conversation
        task.Call(dialog.Void<object, IMessageActivity>(), null);
        
        await task.PollAsync(CancellationToken.None);

        // Flush the dialog stack back to its state store.
        await botData.FlushAsync(CancellationToken.None);        
    }
}
```
`SurveyDialog` 會控制對話，直到該對話完成為止。 完成其工作時，它會呼叫 `context.Done` 然後關閉，將控制權傳回給上一個對話。 

```cs
[Serializable]
public class SurveyDialog : IDialog<object>
{
    public async Task StartAsync(IDialogContext context)
    {
        await context.PostAsync("Hello, I'm the survey dialog. I'm interrupting your conversation to ask you a question. Type \"done\" to resume");

        context.Wait(this.MessageReceivedAsync);
    }
    public virtual async Task MessageReceivedAsync(IDialogContext context, IAwaitable<IMessageActivity> result)
    {
        if ((await result).Text == "done")
        {
            await context.PostAsync("Great, back to the original conversation!");
            context.Done(String.Empty); // Finish this dialog.
        }
        else
        {
            await context.PostAsync("I'm still on the survey until you type \"done\"");
            context.Wait(MessageReceivedAsync); // Not done yet.
        }
    }
}
```

## <a name="sample-code"></a>範例程式碼

如需示範如何使用適用於 .NET 的 Bot Framework SDK 傳送主動訊息的完整範例，請參閱 GitHub 中的<a href="https://aka.ms/proactive-messaging-cs-v3 " target="_blank">主動訊息範例</a>。 在主動訊息範例中，<a href="https://aka.ms/proactive-sendmessage-cs-v3 " target="_blank">simpleSendMessage</a> 示範如何傳送臨機操作的主動訊息，而 <a href="https://aka.ms/proactive-newdialog-cs-v3 " target="_blank">startNewDialog</a> 示範如何傳送對話方塊式的主動訊息。 

## <a name="additional-resources"></a>其他資源

- [設計和控制交談流程](../bot-service-design-conversation-flow.md)
- <a href="/dotnet/api/?view=botbuilder-3.11.0" target="_blank">適用於 .NET 的 Bot Framework SDK 參考</a>
- <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-proactiveMessages" target="_blank">主動式訊息範例 (GitHub)</a>

