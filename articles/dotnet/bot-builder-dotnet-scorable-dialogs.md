---
title: 使用可評分對話的全域訊息處理常式
description: 在適用於 .NET 的 Bot 建立器 SDK  內，使用可評分對話方塊來建立更有彈性的對話方塊。
author: matthewshim-ms
ms.author: v-shimma
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 8ba63ad99c772c7cf5884180a62244e0dfe11db2
ms.sourcegitcommit: 67445b42796d90661afc643c6bb6533e9a662cbc
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/06/2018
ms.locfileid: "39574904"
---
# <a name="global-message-handlers-using-scorables"></a>使用可評分對話的全域訊息處理常式

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

當使用者嘗試在對話中使用「說明」、「取消」或「重新開始」等字，來存取 Bot 內的某些功能，而 Bot 預期不同的回應時。 您可以將 Bot 設計為使用可評分對話方塊正常處理這類要求。

可評分對話方塊會監視所有傳入訊息，並判斷訊息是否可用某種方式採取動作。 可評分的訊息會依每個可評分對話方塊，獲指派 [0 – 1] 之間的分數。 決定分數最高的可評分對話方塊會新增至對話方塊堆疊的頂端，然後將回應傳遞給使用者。 可評分對話方塊完成執行之後，對話會從停止的地方繼續進行。

可評分對話方塊能藉由讓您的使用者「中斷」在一般對話方塊中可見的一般對話流程，讓您建立更有彈性的對話。

## <a name="create-a-scorable-dialog"></a>建立可評分對話方塊

首先，定義新的[對話方塊](bot-builder-dotnet-dialogs.md)。 下列程式碼會使用衍生自 `IDialog` 介面的對話方塊。

```cs
public class SampleDialog : IDialog<object>
{
    public async Task StartAsync(IDialogContext context)
    {
        await context.PostAsync("This is a Sample Dialog which is Scorable. Reply with anything to return to the prior prior dialog.");

        context.Wait(this.MessageReceived);
    }

    private async Task MessageReceived(IDialogContext context, IAwaitable<IMessageActivity> result)
    {
        var message = await result;

        if ((message.Text != null) && (message.Text.Trim().Length > 0))
        {
            context.Done<object>(null);
        }
        else
        {
            context.Fail(new Exception("Message was not a string or was an empty string."));
        }
    }
}
```
若要製作可評分對話方塊，請建立繼承自 `ScorableBase` 抽象類別的類別。 下列程式碼會示範 `SampleScorable` 類別。

```cs
using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Bot.Builder.Dialogs.Internals;
using Microsoft.Bot.Builder.Internals.Fibers;
using Microsoft.Bot.Builder.Scorables.Internals;

public class SampleScorable : ScorableBase<IActivity, string, double>
{
    private readonly IDialogTask task;

    public SampleScorable(IDialogTask task)
    {
        SetField.NotNull(out this.task, nameof(task), task);
    }
}
```
`ScorableBase` 抽象類別繼承自 `IScorable` 介面。 您必須在類別中實作下列 `IScorable` 方法：

- `PrepareAsync` 是可評分執行個體中呼叫的第一個方法。 它會接受傳入訊息活動、分析並設定對話方塊的狀態，接著會傳遞至 `IScorable` 介面的所有其他方法。

```cs
protected override async Task<string> PrepareAsync(IActivity item, CancellationToken token)
{
        // TODO: insert your code here
}
```

- `HasScore` 方法會檢查 state 屬性，來判斷可評分對話方塊是否應該提供訊息的分數。 如果它傳回 false，則可評分對話方塊會忽略該訊息。

```cs
protected override bool HasScore(IActivity item, string state)
{
        // TODO: insert your code here
}
```

- 只有在 `HasScore`傳回 true 時才會觸發 `GetScore`。 您將佈建這個方法中的邏輯，以判斷分數介於 0 - 1 之間的訊息。

```cs
protected override double GetScore(IActivity item, string state)
{
        // TODO: insert your code here
}
```
- 在 `PostAsync` 方法中，定義要針對可評分類別執行的核心動作。 所有的可評分對話方塊都會監視傳入訊息，並根據可評分對話方塊的 GetScore 方法，將分數指派給有效的訊息。 接著，決定最高分數 (介於 0 - 1.0 之間) 的可評分類別將會觸發可評分對話方塊的 `PostAsync` 方法。

```cs
protected override Task PostAsync(IActivity item, string state, CancellationToken token)
{
        //TODO: insert your code here
}
```

- 在評分流程完成之後，會呼叫 `DoneAsync`。 使用此方法來處置任何已設定範圍的資源。

```cs
protected override Task DoneAsync(IActivity item, string state, CancellationToken token)
{
        //TODO: insert your code here
}
```

## <a name="create-a-module-to-register-the-iscorable-service"></a>建立模組來註冊 IScorable 服務

接下來，定義會將 `SampleScorable` 類別註冊為元件的 `Module`。 這會佈建 `IScorable` 服務。

```cs
public class GlobalMessageHandlersBotModule : Module
{
    protected override void Load(ContainerBuilder builder)
    {
        base.Load(builder);

        builder
            .Register(c => new SampleScorable(c.Resolve<IDialogTask>()))
            .As<IScorable<IActivity, double>>()
            .InstancePerLifetimeScope();
    }
}
```
## <a name="register-the-module"></a>註冊模組  

此程序的最後一個步驟是要將 `SampleScorable` 套用至 Bot 的對話容器。 這會註冊 Bot Framework 訊息處理管線內的可評分服務。 下列程式碼隨即顯示，以在 **Global.asax.cs** 中更新 Bot 應用程式初始化內的 `Conversation.Container`：

```cs
public class WebApiApplication : System.Web.HttpApplication
{
    protected void Application_Start()
    {
        this.RegisterBotModules();
        GlobalConfiguration.Configure(WebApiConfig.Register);
    }

    private void RegisterBotModules()
    {
        var builder = new ContainerBuilder();
        builder.RegisterModule(new ReflectionSurrogateModule());

        //Register the module within the Conversation container
        builder.RegisterModule<GlobalMessageHandlersBotModule>();

        builder.Update(Conversation.Container);
    }
}
```

## <a name="additional-resources"></a>其他資源
* [全域訊息處理常式範例](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-GlobalMessageHandlers) (英文)
* [簡單可評分 Bot 範例](https://github.com/Microsoft/BotFramework-Samples/tree/master/blog-samples/CSharp/ScorableBotSample) (英文)
* [對話方塊概觀](bot-builder-dotnet-dialogs.md)
* [AutoFac](https://autofac.org/) (英文)
