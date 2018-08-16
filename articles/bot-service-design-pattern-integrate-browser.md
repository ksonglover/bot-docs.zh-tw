---
title: 整合 Bot 和網頁瀏覽器 |Microsoft Docs
description: 了解如何為使用者在 Bot 和網頁瀏覽器之間設計出流暢的切換體驗。
author: matvelloso
ms.author: mateusv
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ms.openlocfilehash: 7f3a6ace5e3ef8122cca32baf8ec4c9a1f250dfa
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/18/2018
ms.locfileid: "39299183"
---
# <a name="integrate-your-bot-with-a-web-browser"></a>整合 Bot 和網頁瀏覽器

某些情況下，並非單靠 Bot 就能滿足需求。 Bot 可能需要將使用者傳送到網頁瀏覽器來完成作業，然後在作業完成後，再恢復與使用者的交談。 

## <a name="authentication-and-authorization"></a>驗證和授權
如果 Bot 要讀取 Office 365 中使用者的行事曆，或甚至代表使用者建立約會，使用者必須先使用 Microsoft Azure Active Directory 進行驗證，並授權 Bot 存取使用者的行事曆資料。 Bot 會將使用者重新導向到網頁瀏覽器以完成驗證和授權工作，然後再恢復與使用者的交談。 

## <a name="security-and-compliance"></a>安全性與合規性
安全性與合規性需求通常會限制與 Bot 與使用者可交換的資訊類型。 某些情況下，使用者可能必須在目前的交談外傳送或接收資料。 例如，若使用者想要使用第三方付款供應商來執行付款，就不應在交談中指定信用卡號碼。 反之，Bot 會將使用者導向網頁瀏覽器來完成付款程序，然後再恢復與使用者的交談。

本文章將探索可促進使用者在 Bot 與網頁瀏覽器之間來回進行切換的程序。 

> [!NOTE]
> 從聊天轉換至網頁瀏覽器，然後再回到聊天中，並非理想的流程，因為不同應用程式之間的切換容易混淆使用者。 為了提供更理想的體驗，許多通道會提供 Bot 能用來呈現應用程式的內建 HTML 視窗，而非讓應用程式顯示在網頁瀏覽器中。 這項技術可讓使用者持續交談，同時存取外部資源。 這個方法與使用內嵌網頁檢視中的 OAuth 來管理授權流程的行動應用程式具有類似的概念。

## <a name="bot-to-web-browser-and-back-again"></a>從 Bot 切換到網頁瀏覽器，再切換回 Bot

本圖表說明 Bot 與網頁瀏覽器之間的高階流程整合作業。 

![Bot 與網路整合](~/media/bot-service-design-pattern-integrate-browser/bot-to-web1.png)

請考量以下流程中的每個步驟：

1. <a id="generate-hyperlink"></a>Bot 產生並顯示可將使用者重新導向至網站的超連結。 
   在指定了目前交談內容相關資訊 (例如交談識別碼、通道識別碼和通道中的使用者識別碼) 的目標 URL 上，超連結通常會加入透過查詢字串參數取得的資料。 

2. 使用者只要按一下超連結，就會重新導向至網頁瀏覽器內的目標 URL。 

3. Bot 會進入等待來自網站之通訊的狀態，表示網站流程已經完成。  
   > [!TIP]
   > 設計此流程的目的，是為了避免讓 Bot 在使用者未完成網站流程的狀態下，持續維持在「等待」狀態。 換句話說，如果使用者離開網頁瀏覽器，並再次開啟與 Bot 的通訊，則 Bot 應認可該輸入內容，而非加以[忽略](~/bot-service-design-navigation.md#the-mysterious-bot)。

4. 使用者則透過網頁瀏覽器完成必要的工作。 
   這個程序基本上就是 OAuth 流程，或任何手邊發生的情況中必須完成的事件序列。 

5. <a id="generate-magic-number"></a>當使用者完成網站流程，網站會產生「[魔術數字](#verify-identity)」並指示使用者複製該值，然後貼回與 Bot 進行的交談中。 

6. 使用者完成網站流程後，<a id="signal-to-bot"></a>網站[會傳送訊號給 Bot](#website-signal-to-bot)。 
   如此就向 Bot 傳播「魔術數字」，並提供所有其他相關資料。
   舉例來說，在 OAuth 流程的例子中，網站會將 存取權杖提供給 Bot。

7. 使用者返回 Bot 後，再將「魔術數字」貼到聊天中。 
   Bot 會驗證使用者所提供的「魔術數字」是否與預期值相符，藉此確認目前的使用者與之前按下超連結開始網站流程的使用者為同一人。 

### <a id="verify-identity"></a>透過「魔術數字」確認使用者身分

從 Bot 轉換至網站的流程 (上述的[步驟 5](#generate-magic-number)) 期間，「魔術數字」的產生可讓 Bot 隨之確認開啟網站流程的使用者確實和預期開啟該流程的使用者為同一人。 例如，如果 Bot 和多位使用者進行群組聊天，而其中一人按下超連結開始網站流程。 如果沒有「魔術數字」驗證程序，Bot 就無法得知哪位使用者已完成流程。 任何使用者都能驗證並在其他使用者的工作階段中插入存取權杖。 

> [!WARNING] 
> 這樣的風險並不只存在群組聊天內。 如果沒有「魔術數字」驗證程序，任何取得此超連結的使用者，都能啟動網站流程並冒用其他使用者的身分。 

魔術數字必須是使用強式密碼編譯程式庫所產生的隨機數字。 如需 C# 中的產生程序範例，請參閱 <a href="https://www.nuget.org/packages/BotAuth" target="_blank">BotAuth</a> 程式庫中的<a href="https://github.com/MicrosoftDX/botauth/tree/master/CSharp" target="_blank">這段程式碼</a>。 BotAuth 可讓 Microsoft Bot Framework 的內建 Bot 執行從 Bot 轉換到網站的流程，藉此驗證網站中的使用者並繼而使用驗證程序中產生的存取權杖。 由於 BotAuth 不會假設通道功能，因此這類流程在大部分通道中都能運作良好。 

> [!NOTE]
> 如果通道建立了自己的內嵌網站檢視，就不需要「魔術數字」驗證程序了。

### <a id="website-signal-to-bot"></a> 網站如何向 Bot 傳送「訊號」？

Bot [產生使用者按下後會開始網站流程的超連結](#generate-hyperlink)後， 超連結會加入透過目標 URL 中的查詢字串參數取得之目前交談環境的相關資訊 (例如交談識別碼、通道識別碼和通道中的使用者識別碼)。 網站隨後會針對該使用 Bot Builder SDK 或 REST API 的使用者或交談，使用這類資訊來讀取並寫入狀態變數。 請參閱上述[步驟 6](#signal-to-bot)，了解網站如何向已完成網站流程的 Bot 傳送「訊號」。

## <a name="sample-code"></a>範例程式碼

如本文所述，<a href="https://github.com/MicrosoftDX/botauth" target="_blank">BotAuth</a> 程式庫可讓 OAuth 流程繫結至使用 .NET 和 Microsoft Bot Framework 的節點建立的 Bot。

## <a name="additional-resources"></a>其他資源

- [對話方塊](~/dotnet/bot-builder-dotnet-dialogs.md)
- [使用對話方塊管理交談流程 (.NET)](~/dotnet/bot-builder-dotnet-manage-conversation-flow.md)
- [使用對話方塊管理交談流程 (Node.js)](~/nodejs/bot-builder-nodejs-manage-conversation-flow.md)
