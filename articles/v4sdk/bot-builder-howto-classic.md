---
title: 如何在 SDK V4 中執行 .NET SDK V3 Bot | Microsoft Docs
description: 了解如何使用傳統的 NuGet 套件將 Bot 從 3.x 轉換到 4.0。
keywords: 移轉, 傳統 Bot, 轉換 v3, v3 到 v4
author: v-royhar
ms.author: v-royhar
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 4/25/18
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 3f7e88192d989e79f2124e03571ddba3b5291955
ms.sourcegitcommit: 2dc75701b169d822c9499e393439161bc87639d2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/24/2018
ms.locfileid: "42904632"
---
# <a name="how-to-run-net-sdk-v3-bots-in-sdk-40"></a>如何在 SDK 4.0 中執行.NET SDK V3 Bot

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

**Microsoft.Bot.Builder.Classic** NuGet 套件可讓您輕鬆將 Bot 從 Microsoft Bot Framework 3.x 版本移轉至 4.0 版本。

**注意︰** 此程序僅適用於 **ASP.NET Web 應用程式 (.NET Framework)** Bot。 這不適用於 **ASP.NET Core Web 應用程式** Bot。

## <a name="the-process"></a>程序

程序相當簡單：

- 將 **Microsoft.Bot.Builder.Classic** NuGet 套件新增至專案。
    - 您可能也需要更新 **Autofac** NuGet 套件。
- 更新 **Microsoft.Bot.Builder.Classic** 命名空間。
- 使用 4.0 **ITurnContext** 型 **Conversation.SendAsync()** 叫用對話方塊。

### <a name="add-the-microsoftbotbuilderclassic-nuget-package"></a>新增 Microsoft.Bot.Builder.Classic NuGet 套件

若要新增 **Microsoft.Bot.Builder.Classic** NuGet 套件，請使用 [管理 NuGet 套件] 新增套件。

### <a name="update-the-namespaces"></a>更新命名空間

若要更新命名空間，請刪除您找到的下列任一個 `using` 陳述式：

```csharp
using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Bot.Builder.Dialogs.Internals;
using Microsoft.Bot.Builder.FormFlow;
using Microsoft.Bot.Builder.Scorables;
```

接著，在其位置新增下列 `using` 陳述式：

```csharp
using Microsoft.Bot.Builder.Adapters;
using Microsoft.Bot.Builder.Classic.Dialogs;
using Microsoft.Bot.Builder.Classic.Dialogs.Internals;
using Microsoft.Bot.Builder.Classic.FormFlow;
using Microsoft.Bot.Builder.Classic.Scorables;
using Microsoft.Bot.Connector;
using Microsoft.Bot.Schema;
```

如果您有 `[BotAuthentication]` 行，請刪除該行或使它成為註解。

### <a name="invoke-your-3x-dialog"></a>叫用 3.x 對話方塊

若要叫用 3.x 對話方塊，您仍然應該使用 `Conversation.SendAsync`，只是現在使用 4.0 **ITurnContext** 而非 **Activity**。

```csharp
// invoke a Classic V3 IDialog 
await Conversation.SendAsync(turnContext, () => new EchoDialog());
```

如果您沒有 **ITurnContext** 物件但是有 **Activity** 物件，則能以下列方式取得 **ITurnContext** 物件：

```csharp
BotFrameworkAdapter adapter = new BotFrameworkAdapter("", "");

await adapter.ProcessActivity(this.Request.Headers.Authorization?.Parameter,
        activity,
        async (context) =>
        {
            // Do something with context here. For example, the body of your Post() method may go here.
        });
```

## <a name="fix-assembly-conflicts"></a>修正組件衝突

以傳統 NuGet 套件建立的 Bot 將會與它們的組件產生衝突。 建置之後，這將會出現在 Visual Studio 的 [錯誤清單] 視窗中。

### <a name="if-you-see-warning-found-conflicts-between-different-versions-of-the-same-dependent-assembly"></a>如果您看到「警告: 在同一個相依組件的不同版本之間發現衝突」

如果您發現開頭為下列文字「在同一個相依組件的不同版本之間發現衝突」的警告：

- 按兩下警告訊息。 一個對話方塊隨即出現，詢問「您要在應用程式組態檔中加入繫結重新導向記錄以修正這些衝突嗎?」
- 按一下 [是]。
- 再次建置專案。

### <a name="if-you-see-error-missing-method-exception-on-startup"></a>如果您看到「錯誤：啟動時缺少方法例外狀況」

使用 .NET Standard 時有一個錯誤，通常是使用較舊 .NET 4.6 專案並升級到 4.6.1，然後嘗試使用 .NET Standard 程式庫時會發生這種情況。 基本上，有 2 個不同的 System.Net.Http 組件會嘗試進行動態交換。因應措施是新增會將您重新導向至 System.Net.Http 之 Web.config 的繫結。 

如果您收到這個錯誤，請將下列內容新增至 Web.config 檔案：

```xml
<dependentAssembly>
    <assemblyIdentity name="System.Net.Http" publicKeyToken="B03F5F7F11D50A3A" culture="neutral" />
    <bindingRedirect oldVersion="0.0.0.0-4.2.0.0" newVersion="4.2.0.0" />
</dependentAssembly>
```

如需此問題的詳細資訊，請參閱[從 MSBuild 工具複製/載入 System.Net.Http v4.2.0.0 #25773](https://github.com/dotnet/corefx/issues/25773) \(英文\)。

## <a name="sample-of-a-converted-bot"></a>已轉換 Bot 的範例

針對已經轉換的 Bot，您可以參閱 [EchoBot-Classic](https://github.com/Microsoft/botbuilder-dotnet/tree/master/samples/Microsoft.Bot.Samples.EchoBot-Classic) 範例，它示範 3.x Bot 經過轉換以搭配 4.0 使用。

## <a name="limitations"></a>限制
Microsoft.Bot.Builder.Classic 只是 .NET 4.61 程式庫，不適用於 .NET Core。
