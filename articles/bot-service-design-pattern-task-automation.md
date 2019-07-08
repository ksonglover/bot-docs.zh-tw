---
title: 建立工作自動化 Bot | Microsoft Docs
description: 了解如何設計 Bot，使其在不需要其他人為介入的情況下執行工作。
author: matvelloso
ms.author: mateusv
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 2/13/2018
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: d81a7e55ab7ac5e3b430ae051d1abbb4ca94b44d
ms.sourcegitcommit: a295a90eac461f8b96770dd902ba44919acf33fc
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/26/2019
ms.locfileid: "67405837"
---
# <a name="create-task-automation-bots"></a>建立工作自動化 Bot

[!INCLUDE [pre-release-label](./includes/pre-release-label-v3.md)]

工作自動化 Bot 可讓使用者完成特定工作或一組工作，而不需要任何人類的協助。 這種類型的 Bot 通常十分類似一般應用程式或網站，主要是透過豐富的使用者控制項和文字，與使用者通訊。 可能會有了解自然語言的功能，可擴充與使用者的對話。 

## <a name="example-use-case-password-reset"></a>範例使用案例：密碼重設

為了進一步了解工作 Bot 的本質，請思考使用案例範例：密碼重設。 

每天都會有員工打好幾通電話來 Contoso 公司的技術支援中心，要求重設密碼。 Contoso 想要將重設員工密碼這個簡單、可重複執行的工作自動化，以便技術支援中心代表可以將時間花費在處理更複雜的問題。 

Contoso 的資深開發人員 John 決定建立 Bot，以將密碼重設工作自動化。 他開始撰寫 Bot 的設計規格，就像在建立新的應用程式或網站時一樣。 

### <a name="navigation-model"></a>巡覽模式

此規格會定義巡覽模式：

![對話方塊結構](~/media/bot-service-design-pattern-task-automation/simple-task1.png)

使用者會從 `RootDialog` 開始。 當他們要求密碼重設時，  
系統會將他們導向至 `ResetPasswordDialog`。 使用 `ResetPasswordDialog` 時，Bot 將會提示使用者提供兩項資訊：電話號碼和出生日期。 

> [!IMPORTANT]
> 本文中所述的 Bot 僅供範例用途。 在真實世界案例中，密碼重設 Bot 可能會實作更健全的身分識別驗證流程。

### <a name="dialogs"></a>對話方塊

接下來，規格會描述每個對話方塊的外觀和功能。 

#### <a name="root-dialog"></a>根對話方塊

根對話方塊會提供兩個選項給使用者： 

1. **變更密碼**的適用情況，是使用者知道他們目前的密碼，並只想要加以變更。
2. **重設密碼**的適用情況，是使用者已忘記或錯置其密碼，且必須產生一個新的密碼。

> [!NOTE]
> 為了簡單起見，本文只說明**重設密碼**流程。

此規格會描述根對話方塊，如下列螢幕擷取畫面所示。

![對話方塊結構](~/media/bot-service-design-pattern-task-automation/simple-task2.png)

#### <a name="resetpassword-dialog"></a>ResetPassword 對話方塊

當使用者從根對話方塊中選擇 [重設密碼]  時，會叫用 `ResetPassword` 對話方塊。 接著，`ResetPassword` 對話方塊會叫用另外兩個對話方塊。 首先，它會叫用 `PromptStringRegex` 對話方塊來收集使用者的電話號碼。 然後它會叫用 `PromptDate` 對話方塊來收集使用者的出生日期。 

> [!NOTE]
> 在此範例中，John 選擇實作的邏輯，會使用兩個不同的對話方塊來收集使用者的電話號碼與出生日期。 此方法不僅可簡化每個對話方塊所需的程式碼，也可增加未來在其他案例中使用這些對話方塊的機率。 

此規格描述 `ResetPassword` 對話方塊。

![對話方塊結構](~/media/bot-service-design-pattern-task-automation/simple-task3.png)

#### <a name="promptstringregex-dialog"></a>PromptStringRegex 對話方塊

`PromptStringRegex` 對話方塊會提示使用者輸入其電話號碼，並驗證使用者提供的電話號碼符合預期的格式。 它也會負責使用者重複提供無效輸入的案例。 此規格描述 `PromptStringRegex` 對話方塊。

![對話方塊結構](~/media/bot-service-design-pattern-task-automation/simple-task4.png)

### <a name="prototype"></a>原型

最後，此規格會提供一個範例，說明使用者與 Bot 進行通訊，以成功完成密碼重設工作。

![對話方塊結構](~/media/bot-service-design-pattern-task-automation/simple-task5.png)

## <a name="bot-app-or-website"></a>Bot、應用程式還是網站？

您可能會好奇，如果工作自動化 Bot 十分類似應用程式或網站，為什麼不改為建置應用程式或網站就好？ 根據您的特定案例，建置應用程式或網站而不建置 Bot 可能是完全合理的選擇。 您可能甚至會使用 [Bot Framework Direct Line API][directLineAPI] 或<a href="https://aka.ms/BotFramework-WebChat" target="_blank">網路聊天控制項</a>，選擇將您的 Bot 內嵌到應用程式中。 在應用程式的內容中實作 Bot 有兩個優點：在同一個位置提供豐富的應用程式與對話體驗。 

不過，在許多情況下，建置應用程式或網站可能遠比建置 Bot 還更複雜且更昂貴。 應用程式或網站通常必須支援多個用戶端和平台，因此封裝和部署可能會是既繁瑣又費時的流程，且必須下載及安裝應用程式的使用者經驗並不一定很理想。 基於這些理由，Bot 通常可提供較簡單的方式，來解決眼前的問題。 

此外，Bot 能夠自由地輕鬆延伸及擴充。 例如，開發人員可以選擇將自然語言和語音功能新增至密碼重設 Bot，使其可透過語音通話進行存取，或可新增文字簡訊的支援。 該公司可能會在整個建置中設定 kiosk，並在這個體驗中內嵌密碼重設 Bot。

<!-- TODO: SimpleTaskAutomation no longer exists
## Sample code

For a complete sample that shows how to implement simple task automation using the Bot Framework SDK for .NET, see the <a href="https://aka.ms/capability-SimpleTaskAutomation" target="_blank">Simple Task Automation sample</a> in GitHub.

For a complete sample that shows how to implement simple task automation using the Bot Framework SDK for Node.js, see the <a href="https://aka.ms/capability-SimpleTaskAutomation" target="_blank">Simple Task Automation sample</a> in GitHub.
-->

## <a name="additional-resources"></a>其他資源

- [對話方塊](~/dotnet/bot-builder-dotnet-dialogs.md)
- [使用對話方塊來管理對話流程 (.NET)](~/dotnet/bot-builder-dotnet-manage-conversation-flow.md)
- [使用對話方塊來管理對話流程 (Node.js)](~/nodejs/bot-builder-nodejs-manage-conversation-flow.md)


[directLineAPI]: https://docs.botframework.com/restapi/directline3/#navtitle
