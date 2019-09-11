---
title: 使用對話方塊管理交談流程 | Microsoft Docs
description: 了解如何使用對話和適用於 .NET 的 Bot Framework SDK 來建立交談模型及管理交談流程。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 9e183c375e16a951ed77819ab982790944ac0c13
ms.sourcegitcommit: a6d02ec4738e7fc90b7108934740e9077667f3c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/04/2019
ms.locfileid: "70298818"
---
# <a name="manage-conversation-flow-with-dialogs"></a>使用對話方塊管理交談流程

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-manage-conversation-flow.md)
> - [Node.js](../nodejs/bot-builder-nodejs-dialog-manage-conversation-flow.md)

[!INCLUDE [Dialog flow example](../includes/snippet-dotnet-manage-conversation-flow-intro.md)]

本文說明如何使用[對話](bot-builder-dotnet-dialogs.md)和適用於 .NET 的 Bot Framework SDK 建立此交談流程的模型。 

## <a name="invoke-the-root-dialog"></a>叫用根對話方塊

首先，Bot 控制器會叫用「根對話方塊」。 下列範例示範如何將基本 HTTP POST 呼叫連線至控制器，然後叫用根對話方塊。 

```cs
public class MessagesController : ApiController
{
    public async Task<HttpResponseMessage> Post([FromBody]Activity activity)
    {
            // Redirect to the root dialog.
            await Conversation.SendAsync(activity, () => new RootDialog()); 
            ...
    }
}
```

## <a name="invoke-the-new-order-dialog"></a>叫用「新訂單」對話方塊

然後，根對話會叫用「新訂單」對話方塊。 

```cs
[Serializable]
public class RootDialog : IDialog<object>
{
    public async Task StartAsync(IDialogContext context)
    {
        // Root dialog initiates and waits for the next message from the user. 
        // When a message arrives, call MessageReceivedAsync.
        context.Wait(this.MessageReceivedAsync); 
    }

    public virtual async Task MessageReceivedAsync(IDialogContext context, IAwaitable<IMessageActivity> result)
    {
        var message = await result; // We've got a message!
        if (message.Text.ToLower().Contains("order"))
        {
            // User said 'order', so invoke the New Order Dialog and wait for it to finish.
            // Then, call ResumeAfterNewOrderDialog.
            await context.Forward(new NewOrderDialog(), this.ResumeAfterNewOrderDialog, message, CancellationToken.None);
        }
        // User typed something else; for simplicity, ignore this input and wait for the next message.
        context.Wait(this.MessageReceivedAsync);
    }

    private async Task ResumeAfterNewOrderDialog(IDialogContext context, IAwaitable<string> result)
    {
        // Store the value that NewOrderDialog returned. 
        // (At this point, new order dialog has finished and returned some value to use within the root dialog.)
        var resultFromNewOrder = await result;

        await context.PostAsync($"New order dialog just told me this: {resultFromNewOrder}");

        // Again, wait for the next message from the user.
        context.Wait(this.MessageReceivedAsync);
    }
}
```

## <a id="dialog-lifecycle"></a> 對話方塊生命週期

當叫用對話方塊時，它會控制交談流程。 每則新訊息都必須由該對話方塊處理，直到關閉或重新導向至另一個對話方塊為止。 

在 C# 中，您可以使用 `context.Wait()` 指定要在使用者下次傳送訊息時叫用的回撥。 若要關閉對話方塊，並從堆疊移除 (藉此將使用者傳回給堆疊中之前的對話方塊)，請使用 `context.Done()`。 您必須使用 `context.Wait()`、`context.Fail()`、`context.Done()` 或某些重新導項指示詞 (如 `context.Forward()` 或 `context.Call()`) 來結束每個對話方塊方法。 未使用上述其中一種方式結束的對話方塊方法會導致發生錯誤 (原因是架構不知道要在使用者下次傳送訊息時採取什麼動作)。

## <a name="passing-state-between-dialogs"></a>在對話方塊之間傳遞狀態

雖然您可以將狀態儲存在 Bot 狀態中，但也可透過多載對話方塊類別建構函式，在不同的對話方塊間傳遞資料。

```cs
[Serializable]
public class AgeDialog : IDialog<int>
{
    private string name;

    public AgeDialog(string name)
    {
        this.name = name;
    }
}
 ```

呼叫對話方塊程式碼，傳入使用者的名稱值。

```cs
private async Task NameDialogResumeAfter(IDialogContext context, IAwaitable<string> result)
{
    try
    {
        this.name = await result;
        context.Call(new AgeDialog(this.name), this.AgeDialogResumeAfter);
    }
    catch (TooManyAttemptsException)
    {
        await context.PostAsync("I'm sorry, I'm having issues understanding you. Let's try again.");
        await this.SendWelcomeMessageAsync(context);
    }
}
```

## <a name="sample-code"></a>範例程式碼 

如需示範如何透過在適用於 .NET 的 Bot Framework SDK 中使用對話來管理交談的完整範例，請參閱 GitHub 中的 [Basic Multi-Dialog sample](https://aka.ms/v3cs-MultiDialog-Sample) (基本多對話範例)。

## <a name="additional-resources"></a>其他資源

- [對話方塊](bot-builder-dotnet-dialogs.md)
- [設計和控制交談流程](../bot-service-design-conversation-flow.md)
- [Basic Multi-Dialog sample (GitHub)](https://aka.ms/v3cs-MultiDialog-Sample) (基本多對話方塊範例 (GitHub))
- <a href="/dotnet/api/?view=botbuilder-3.11.0" target="_blank">適用於 .NET 的 Bot Framework SDK 參考</a>
