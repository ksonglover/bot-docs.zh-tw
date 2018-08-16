---
title: 將 Bot 內嵌至應用程式 |Microsoft Docs
description: 了解如何設計內嵌在另一個應用程式中的 Bot。
author: matvelloso
ms.author: mateusv
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ms.openlocfilehash: 3264388cf253bb949eabe3be04fdaebabdc36b99
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/18/2018
ms.locfileid: "39299187"
---
# <a name="embed-a-bot-in-an-app"></a>將 Bot 內嵌至應用程式

雖然 Bot 最常存在於應用程式之外，但也可以與應用程式整合。 例如，您可以將[知識 Bot](~/bot-service-design-pattern-knowledge-base.md) 內嵌在應用程式內，以協助使用者在複雜的應用程式結構內尋找不易找到的資訊。 您可以將 Bot 內嵌在技術支援中心的應用程式內， 做為傳入使用者要求的第一個回應程式。 Bot 可以獨立解決簡單的問題，並將較為複雜的問題[遞交](~/bot-service-design-pattern-handoff-human.md)給人類專員。 

## <a name="integrating-bot-with-app"></a>整合應用程式與 Bot

整合 Bot 與應用程式的方式需視應用程式的類型而有所不同。 

### <a name="native-mobile-app"></a>原生行動應用程式

以原生程式碼建立的應用程式可以使用 [直接線路 API][directLineAPI]，透過 REST 或 websocket， 與 Bot Framework 通訊。

### <a name="web-based-mobile-app"></a>網頁型行動應用程式

使用 Web 語言和架構 (例如 <a href="https://cordova.apache.org/" target="_blank">Cordova</a>) 建立的行動應用程式，可能會藉由使用與[內嵌在網站內的 Bot ](~/bot-service-design-pattern-embed-web-site.md)所使用的相同的元件來與 Bot Framework 通訊，只要封裝在原生應用程式殼層中即可。

### <a name="iot-app"></a>IoT 應用程式

IoT 應用程式可以使用[直接線路 API][directLineAPI] 與 Bot Framework 進行通訊。 在某些案例中，它也可以使用 <a href="https://www.microsoft.com/cognitive-services/" target="_blank">Microsoft 認知服務</a>啟用影像辨識和語音等功能。

### <a name="other-types-of-apps-and-games"></a>其他類型的應用程式和遊戲

其他類型的應用程式和遊戲可以使用[直接線路 API][directLineAPI] 與 Bot Framework 進行通訊。 

## <a name="creating-a-cross-platform-mobile-app-that-runs-a-bot"></a>建立執行 Bot 的跨平台行動應用程式

在此範例中，是使用建立跨平台行動應用程式的熱門工具 <a href="https://www.xamarin.com/" target="_blank">Xamarin</a> 來建立執行 Bot 的行動應用程式。 

首先，建立簡單的 Web 檢視元件，並運用在架設<a href="https://github.com/Microsoft/BotFramework-WebChat" target="_blank">網路聊天控制項</a>。 然後，使用 Bot Framework 入口網站，[TODO](~/bot-service-manage-channels.md) 前往網路聊天頻道。 

![Bot 組態設定](~/media/bot-service-design-pattern-embed-app/webchat-channel.png)

接下來，指定已註冊的網路聊天 URL 做為 Xamarin 應用程式中 Web 檢視控制項的來源：

```cs
public class WebPage : ContentPage
{
    public WebPage()
    {
        var browser = new WebView();
        browser.Source = "https://webchat.botframework.com/embed/<YOUR SECRET KEY HERE>";
        this.Content = browser;
    }
}
```

您可以使用此程序來建立跨平台行動應用程式，以網站聊天控制項呈現內嵌的 Web 檢視。

![反向通道](~/media/bot-service-design-pattern-embed-app/xamarin-apps.png)

## <a name="sample-code"></a>範例程式碼

如需完整範例顯示如何建立執行 Bot 的跨平台行動應用程式 (如本文所述)，請參閱 GitHub 中的<a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/capability-BotInApps" target="_blank">應用程式範例中的 Bot</a>。

## <a name="additional-resources"></a>其他資源

- [直接線路 API][directLineAPI]
- <a href="https://www.microsoft.com/cognitive-services/" target="_blank">Microsoft 認知服務</a>

[directLineAPI]: https://docs.botframework.com/en-us/restapi/directline3/#navtitle