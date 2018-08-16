---
title: 撰寫您自己的中介軟體 | Microsoft Docs
description: 了解如何撰寫您自己的中介軟體。
keywords: middleware, custom middleware, short circuit, fallback, activity handlers, 中介軟體, 自訂中介軟體, 最少運算, 後援, 活動處理常式
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 03/21/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 6bc73b2886374fbb50d8257c387df54f21a12ed7
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/18/2018
ms.locfileid: "39299863"
---
# <a name="write-your-own-middleware"></a>撰寫您自己的中介軟體

中介軟體可讓您為自己的 Bot 撰寫功能豐富的外掛程式，且後續也可以供其他人使用。 我們將會示範如何新增並實作基本的中介軟體，並示範其運作方式。 v4 SDK 能為您提供一些中介軟體，可用於狀態管理、LUIS、QnAMaker 及轉譯等項目。 請參考適用於 [.NET](https://github.com/Microsoft/botbuilder-dotnet) \(英文\) 或 [JavaScript](https://github.com/Microsoft/botbuilder-js) \(英文\) 的 Bot 建立器 SDK 以取得詳細資訊。

## <a name="adding-middleware"></a>新增中介軟體

在下列範例中，以我們的基本 HelloBot 範例為基礎，我們會將兩件不同的中介軟體新增至服務中，且那些類別都會有個別的新執行個體。

> [!IMPORTANT]
> 請記住，將它們新增至選項的順序將會決定其執行順序。 在使用超過一件中介軟體的情況下，請務必將此納入考量。

**Startup.cs**

# <a name="ctabcsaddmiddleware"></a>[C#](#tab/csaddmiddleware)

針對您想要新增的每件中介軟體，將 `options.Middleware.Add(new MyMiddleware());` 方法呼叫新增至您的 Bot 服務選項。

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddSingleton(_ => Configuration);
    services.AddBot<HelloBot>(options =>
    {
        options.CredentialProvider = new ConfigurationCredentialProvider(Configuration);
        options.Middleware.Add(new MyMiddleware());
        options.Middleware.Add(new MyOtherMiddleware());
    });
}
```
# <a name="javascripttabjsaddmiddleware"></a>[JavaScript](#tab/jsaddmiddleware)

針對您想要新增的每件中介軟體，將 `adapter.use(MyMiddleware());` 新增至您的配接器。

```javascript
// Create adapter
const adapter = new botbuilder.BotFrameworkAdapter({
    appId: process.env.MICROSOFT_APP_ID, 
    appPassword: process.env.MICROSOFT_APP_PASSWORD 
});

adapter.use(MyMiddleware());
adapter.use(MyOtherMiddleware());
```

---


## <a name="implementing-your-middleware"></a>實作中介軟體

每件中介軟體都會繼承自中介軟體介面，且一律會實作其處理處理常式 (該處理常式會針對每一個被傳送至您 Bot 的活動執行)。 針對每件新增的中介軟體，隨著內容物件沿著管線前進，處理處理常式將會有機會在允許其他中介軟體或 Bot 邏輯與該內容物件互動之前，對內容物件或執行工作 (例如記錄) 做出修改。

# <a name="ctabcsetagoverwrite"></a>[C#](#tab/csetagoverwrite)

每件中介軟體都會繼承自 `IMiddleware` 並一律會實作 `OnTurn()`。

**ExampleMiddleware.cs**
```csharp
public class MyMiddleware : IMiddleware
{
    public async Task OnTurn(ITurnContext context, MiddlewareSet.NextDelegate next)
    {            
        // This simple middleware reports the request type and if we responded
        await context.SendActivity($"Request type: {context.Activity.Type}");
        
        await next();            

        // Report if any responses were recorded
        string response = context.Responded ? "yes" : "no";
        await context.SendActivity($"Responded?  {response}");
    }
}

public class MyOtherMiddleware : IMiddleware
{
    public async Task OnTurn(ITurnContext context, MiddlewareSet.NextDelegate next)
    {
        // simple middleware to add an additional send activity
        await context.SendActivity($"My other middleware just saying Hi before the bot logic");

        await next();
    }
}

```

# <a name="javascripttabjsimplementmiddleware"></a>[JavaScript](#tab/jsimplementmiddleware)

每件中介軟體都會繼承自 `MiddlewareSet` 並一律會實作 `onTurn()`。

**ExampleMiddleware.js**
```js
adapter.use({onTurn: async (context, next) =>{

    // This simple middleware reports the activity type and if we responded
    await context.sendActivity(`Activity type: ${context.activity.type}`); 
    await next();            

    // Report if any responses were recorded
    const response = context.activity.text ? "yes" : "no";
    await context.sendActivity(`Responded?  ${response}`);

}}, {onTurn: async (context, next) => {

     // simple middleware to add an additional send activity
     await context.sendActivity("My other middleware just saying Hi before the bot logic");

     await next();
}})
```

---

呼叫 `next()` 會導致執行繼續至下一件中介軟體。 由於能夠選擇執行傳遞的時機，您將可以撰寫在其餘中介軟體皆已執行**之後**才會執行的程式碼。 我們可以在 Bot 邏輯和其他中介軟體皆已執行之後，針對處理處理常式的「尾端邊緣」採取行動。  在上述範例中所實作的第一個中介軟體就會那麼做，它會在我們針對此內容物件做出回應的情況下，在於管線中往回傳遞執行之前做出報告。

## <a name="short-circuit-routing"></a>最少運算路由

在某些情況下，您可能會想要停止進一步處理所接收到的活動，我們將此稱為最少運算。 這在中介軟體會完全處理要求、會針對特定命令提供簡單回應，或是可以在 Bot 邏輯不需查看連入要求便處理該要求的情況下相當有用。

讓我們建立一件會在使用者說出 "ping" (偵測) 時便傳送回應，並防止對要求進行任何進一步路由的中介軟體：

# <a name="ctabcsmiddlewareshortcircuit"></a>[C#](#tab/csmiddlewareshortcircuit)
```cs
public class ExampleMiddleware : IMiddleware
{
    public async Task OnTurn(ITurnContext context, MiddlewareSet.NextDelegate next)
    {
        var utterance = context.Activity?.AsMessageActivity()?.Text.Trim().ToLower();

        if (utterance == "ping") 
        {
            context.SendActivity("pong");
            return;
        } 
        else 
        {
            await next();
        }
    }
}
```
# <a name="javascripttabjsmiddlewareshortcircuit"></a>[JavaScript](#tab/jsmiddlewareshortcircuit)
```JavaScript
adapter.use({onTurn: async (context, next) =>{
    const utterance = (context.activity.text || '').trim().toLowerCase();
        if (utterance == "ping") 
        {
            await context.sendActivity("pong");
            return;
        } 
        else 
        {
            await next();
        }

}})

```

---

## <a name="fallback-processing"></a>後援處理

您可能需要做的另一件事，便是回應尚未被回應的要求。 這可以使用處理處理常式的尾端邊緣並透過檢查 `context.Responded` 屬性來輕鬆達成。 讓我們建立一件簡單的中介軟體，讓它會在 Bot 無法處理要求時自動說出 "I didn't understand" (我不了解)：

# <a name="ctabcsfallback"></a>[C#](#tab/csfallback)
```cs
public async Task OnTurn(ITurnContext context, MiddlewareSet.NextDelegate next)
{
    await next();

    if (!context.Responded) 
    {
        context.SendActivity("I didn't understand.");
    }
}
```
# <a name="javascripttabjsfallback"></a>[JavaScript](#tab/jsfallback)
```JavaScript
adapter.use({onTurn: async (context, next) =>{
    await next();

    if (!context.responded) 
    {
       await context.sendActivity("I didn't understand.");
    }

}})
```

---

> [!NOTE] 
> 這可能無法適用於所有情況，例如當有另一個中介軟體可能可以回應使用者，或是當 Bot 正確接收到訊息但沒有回應時。 在這些情況下，回應 "I don't understand" (我不了解) 可能會誤導使用者。


