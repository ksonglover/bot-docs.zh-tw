---
title: 攔截訊息 | Microsoft Docs
description: 了解如何使用適用於 .NET 的 Bot Framework SDK 攔截使用者與 Bot 之間的訊息。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 03/01/2019
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 0e8873d2914d42b928004c31c14c8d60cb35b2a3
ms.sourcegitcommit: 4139ef7ebd8bb0648b8af2406f348b147817d4c7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/19/2019
ms.locfileid: "58073764"
---
# <a name="intercept-messages"></a>攔截訊息

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-middleware.md)
> - [Node.js](../nodejs/bot-builder-nodejs-intercept-messages.md)

[!INCLUDE [Introducton to message logging](../includes/snippet-message-logging-intro.md)]

## <a name="intercept-and-log-messages"></a>攔截和記錄訊息

下列程式碼範例顯示如何使用適用於 .NET 的 Bot Framework SDK 的**中介軟體**概念，攔截使用者與 Bot 之間交換的訊息。 

首先，建立 `DebugActivityLogger` 類別並定義 `LogAsync` 方法，以指定攔截到的每個訊息會執行哪些動作。 此範例只會列印每個訊息的部分資訊。

```cs
public class DebugActivityLogger : IActivityLogger
{
    public async Task LogAsync(IActivity activity)
    {
        Debug.WriteLine($"From:{activity.From.Id} - To:{activity.Recipient.Id} - Message:{activity.AsMessageActivity()?.Text}");
    }
}
```

然後，將下列程式碼新增至 `Global.asax.cs`。  每個使用者與 Bot (任一方向) 之間交換的訊息，將會立即觸發 `DebugActivityLogger` 類別中的 `LogAsync`方法。 

```cs
    public class WebApiApplication : System.Web.HttpApplication
    {
        protected void Application_Start()
        {
            var builder = new ContainerBuilder();
            builder.RegisterType<DebugActivityLogger>().AsImplementedInterfaces().InstancePerDependency();
            builder.Update(Conversation.Container);

            GlobalConfiguration.Configure(WebApiConfig.Register);
        }
    }
```

雖然此範例只會列印每個訊息的部分資訊，您可以更新 `LogAsync` 方法，以指定您想要針對每個訊息採取的動作。 

## <a name="sample-code"></a>範例程式碼 

如需顯示如何使用適用於 .NET 的 Bot Framework SDK 攔截和記錄訊息的完整範例，請參閱 GitHub 中的<a href="https://github.com/Microsoft/BotBuilder-Samples/tree/v3-sdk-samples/CSharp/core-Middleware" target="_blank">中介軟體範例</a>。 

## <a name="additional-resources"></a>其他資源

- <a href="/dotnet/api/?view=botbuilder-3.11.0" target="_blank">適用於 .NET 的 Bot Framework SDK 參考</a>
- <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/v3-sdk-samples/CSharp/core-Middleware" target="_blank">中介軟體範例 (GitHub)</a>
