---
title: 設計 Bot 導覽 |Microsoft Docs
description: 了解如何為 Bot 設計優良的導覽架構，以及避免最常見的導覽設計錯誤。
keywords: 瀏覽, 概觀, 固執 bot, 毫無頭緒的 bot, 神秘的 bot, 講廢話的 bot, 念念不忘的 bot
author: matvelloso
ms.author: mateusv
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
ms.openlocfilehash: f2a97b35f7e83a825e533be528951e8c04c521a1
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/24/2018
ms.locfileid: "49998077"
---
# <a name="design-bot-navigation"></a>設計 Bot 導覽

使用者可以使用階層連結瀏覽網站，使用功能表瀏覽應用程式，以及使用**下一頁**和**上一頁**等按鈕瀏覽網頁。 不過，上述這些固有的導覽技術都無法完全解決 Bot 內的導覽需求。 如[先前](~/bot-service-design-conversation-flow.md#handle-interruptions)所述，使用者通常會以非線性的方式與 Bot 互動，因此要設計出可提供一致的優良使用者體驗之 Bot 導覽，變得非常困難。 

需要考量以下難題：

- 如何確保使用者不會在與 Bot 的交談中感到混淆？ 
- 使用者能否在與 Bot 的交談中進行「回溯」瀏覽？ 
- 使用者如何在與 Bot 交談期間瀏覽至「主功能表」？ 
- 使用者如何在與 Bot 交談期間「取消」作業？ 

Bot 導覽設計的具體細節絕大部分取決於 Bot 可支援的特色與功能。 不論要開發的 Bot 類型為何，您都要避免常見的錯誤，以免設計出不良的交談介面。 本文透過下列五種性格來說明這些錯誤：「固執的 Bot」、「毫無頭緒的 Bot」、「神祕的 Bot」、「講廢話的 Bot」和「念念不忘的 Bot」。 

> [!TIP]
> 只要正確地[處理使用者中斷](v4sdk/bot-builder-howto-handle-user-interrupt.md)，通常就能改善 Bot 的這些性格。

## <a name="the-stubborn-bot"></a>「固執的 Bot」

固執的 Bot 會堅持繼續目前的交談方向，即使使用者嘗試轉換話題，仍依然故我。 

請考慮下列狀況： 

![Bot](~/media/bot-service-design-navigation/stubborn-bot-new.png)

使用者會經常改變心意、決定取消或全部從頭開始。 

> [!TIP]
> <b>切記</b>：設計 Bot 時，請將使用者預設為可能會隨時改變交談方向納入考量。 
>
> <b>請勿</b>： 設計時，切勿使 Bot 忽略使用者輸入，並永無止境地持續重複相同問題。 

避免這項錯誤的方法有很多種，但要避免 Bot 無止盡地詢問相同問題，最簡單的方法可能是指定每個問題重試次數的上限。 若以這種方式來設計，Bot 雖不會「聰明」地理解使用者輸入內容並給予恰當回應，但至少能夠避免永無止境地反覆詢問相同問題。 

## <a name="the-clueless-bot"></a>「毫無頭緒的 Bot」

毫無頭緒的 Bot 會在無法理解使用者想要存取特定功能時，以無意義的方式回應。 使用者可能會嘗試輸入常見關鍵字命令，例如「幫助」或「取消」，並合理地期望 Bot 能妥善回應。

請考慮下列狀況： 

![Bot](~/media/bot-service-design-navigation/clueless-bot.png)

雖然您可能很想將 Bot 中的每個對話方塊設計成能夠聽取特定關鍵字，並給予恰當回應，但我們並不建議這麼做。 

> [!TIP]
> <b>切記</b>：請執行[中介軟體](v4sdk/bot-builder-create-middleware.md)，藉此檢查使用者輸入中是否有您指定的關鍵字 (例如「幫助」、「取消」、「從新開始」等)，並給予適當回應。 
> 
> <b>請勿</b>：將每個對話方塊設計成按照關鍵字清單檢查使用者輸入內容。 

只要定義**中介軟體**的邏輯，中介軟體就能存取每次與使用者進行的交流。 運用這個方法，個別對話方塊和提示就能在必要時安全地忽略關鍵字。

## <a name="the-mysterious-bot"></a>「神祕的 Bot」

神祕的 Bot 無論如何都無法立即認可使用者輸入。 

請考慮下列狀況： 

![Bot](~/media/bot-service-design-navigation/mysterious-bot.png)

某些情況下，這類狀況可能表示 Bot 發生中斷。 不過，也有可能 Bot 正忙於處理使用者輸入，而且回應的編譯作業尚未完成。 

> [!TIP]
> <b>切記</b>：請將 Bot 設計成可立即認可使用者輸入內容，即便 Bot 可能需要花一些時間來編譯回應亦然。 
> 
> <b>請勿</b>：將 Bot 設計成要先完成回應的編譯作業，才認可使用者輸入內容。

立即認可使用者輸入內容，您才能消除任何可能對 Bot 狀態產生的混淆。 如果您需要花很長的時間來編譯，不妨傳送「輸入中」訊息，表示 Bot 正在運作，接著再透過[主動訊息](v4sdk/bot-builder-howto-proactive-message.md)傳送後續內容

## <a name="the-captain-obvious-bot"></a>「講廢話的 Bot」

講廢話的 Bot 會提供未經要求且顯而易見的資訊，因此對使用者不具實用價值。 

請考慮下列狀況：

![Bot](~/media/bot-service-design-navigation/captainobvious-bot.png)

> [!TIP]
> <b>切記</b>：請妥善設計 Bot 以便為使用者提供實用資訊。 
> 
> <b>請勿</b>：將 Bot 設計成向使用者提供未經要求且可能不實用的資訊。

將 Bot 設計成可提供實用資訊，使用者與 Bot 的互動機率就會提高。

## <a name="the-bot-that-cant-forget"></a>「念念不忘的 Bot」

念念不忘的 Bot　會在不適當的時機，將過往交談中的資訊整合進目前的交談中。 

請考慮下列狀況：

![Bot](~/media/bot-service-design-navigation/rememberall-bot.png)

> [!TIP]
> <b>切記</b>：請將 Bot 設計成只要使用者並未重新提起之前的主題，就會維持目前的交談主題。 
> 
> <b>請勿</b>：將 Bot 設計成會將過往交談中的資訊插入目前並不相關的交談中。

只要能維持目前的交談主題，您就能減少造成的混淆和挫折感的可能性，同時提高使用者繼續和 Bot 互動的機率。

## <a name="next-steps"></a>後續步驟

如果能將 Bot 設計成可避免以上設計不良之交談介面常見的錯誤，您就能踏出重要的一步，從而確保絕佳的使用者體驗。 

接下來，請論了解通常 Bot 與使用者與交換資訊所仰賴的 [UX 元素](~/bot-service-design-user-experience.md)。 
