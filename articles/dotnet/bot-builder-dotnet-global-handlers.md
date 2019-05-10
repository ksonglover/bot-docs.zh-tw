---
title: 實作全域訊息處理常式 | Microsoft Docs
description: 了解如何使用適用於 .NET 的 Bot Framework SDK，讓 Bot 接聽和處理包含某些關鍵字的使用者輸入。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 86964bc39a95a23f397af649cfac6e2784dd588a
ms.sourcegitcommit: 980612a922b8290b2faadaca193496c4117e415a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/26/2019
ms.locfileid: "64563633"
---
# <a name="implement-global-message-handlers"></a>實作全域訊息處理常式

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

[!INCLUDE [Introduction to global message handlers](../includes/snippet-global-handlers-intro.md)]

## <a name="listen-for-keywords-in-user-input"></a>接聽使用者輸入中的關鍵字

下列逐步解說示範如何使用適用於 .NET 的 Bot Framework SDK 來實作全域訊息處理常式。

首先，`Global.asax.cs` 會註冊 `GlobalMessageHandlersBotModule`，實作如下所示。 在此範例中，模組會註冊兩個可評分的設定：一個用於管理變更設定的要求 (`SettingsScorable`)，另一個用於管理要取消的要求 (`CancelScoreable`)。

```cs
public class GlobalMessageHandlersBotModule : Module
{
    protected override void Load(ContainerBuilder builder)
    {
        base.Load(builder);

        builder
            .Register(c => new SettingsScorable(c.Resolve<IDialogTask>()))
            .As<IScorable<IActivity, double>>()
            .InstancePerLifetimeScope();

        builder
            .Register(c => new CancelScorable(c.Resolve<IDialogTask>()))
            .As<IScorable<IActivity, double>>()
            .InstancePerLifetimeScope();
    }
}
```

`CancelScorable` 包含定義觸發程序的 `PrepareAsync` 方法：如果訊息文字是「取消」，將會觸發這個可評分的設定。

```cs
protected override async Task<string> PrepareAsync(IActivity activity, CancellationToken token)
{
    var message = activity as IMessageActivity;
    if (message != null && !string.IsNullOrWhiteSpace(message.Text))
    {
        if (message.Text.Equals("cancel", StringComparison.InvariantCultureIgnoreCase))
        {
            return message.Text;
        }
    }
    return null;
}
```

收到「取消」要求時，`CancelScoreable` 內的 `PostAsync` 方法會重設對話堆疊。 

```cs
protected override async Task PostAsync(IActivity item, string state, CancellationToken token)
{
    this.task.Reset();
}
```

收到「變更設定」要求時，`SettingsScorable` 內的 `PostAsync` 方法會叫用 `SettingsDialog` (將要求傳遞至該對話)，從而將 `SettingsDialog` 新增至對話堆疊的最上方，使其可控制對話。

```cs
protected override async Task PostAsync(IActivity item, string state, CancellationToken token)
{
    var message = item as IMessageActivity;
    if (message != null)
    {
        var settingsDialog = new SettingsDialog();
        var interruption = settingsDialog.Void<object, IMessageActivity>();
        this.task.Call(interruption, null);
        await this.task.PollAsync(token);
    }
}
```

## <a name="sample-code"></a>範例程式碼

如需示範如何使用適用於 .NET 的 Bot Framework SDK 來實作全域訊息處理常式的完整範例，請參閱 GitHub 中的<a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-GlobalMessageHandlers" target="_blank">全域訊息處理常式範例</a> \(英文\)。

## <a name="additional-resources"></a>其他資源

- [設計和控制交談流程](../bot-service-design-conversation-flow.md)
- <a href="/dotnet/api/?view=botbuilder-3.12.2.4" target="_blank">適用於 .NET 的 Bot Framework SDK 參考</a>
- <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-GlobalMessageHandlers" target="_blank">全域訊息處理常式範例 (GitHub)</a> \(英文\)
