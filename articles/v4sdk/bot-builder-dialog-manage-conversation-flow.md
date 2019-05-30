---
title: 實作循序對話流程 | Microsoft Docs
description: 了解如何在 Bot Framework SDK 中利用交談管理簡單的交談流程。
keywords: 簡單對話流程, 循序對話流程, 對話, 提示, 瀑布, 對話方塊集
author: JonathanFingold
ms.author: v-jofing
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 05/23/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 680d9148b463bbb5d10f4a6a06cc7b32b824b66e
ms.sourcegitcommit: ea64a56acfabc6a9c1576ebf9f17ac81e7e2a6b7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/24/2019
ms.locfileid: "66215425"
---
# <a name="implement-sequential-conversation-flow"></a>實作循序對話流程

[!INCLUDE[applies-to](../includes/applies-to.md)]

張貼問題來收集資訊是 Bot 與使用者互動時的其中一種主要方式。 dialogs 程式庫可讓您輕鬆地詢問問題，以及驗證回應以確保回應符合特定資料類型或符合自訂驗證規則。

您可以使用對話方塊程式庫來管理簡單和複雜的對話流程。 在簡單互動中，Bot 透過一連串固定的步驟執行，然後完成對話。 一般而言，當 Bot 需要向使用者蒐集資訊時，對話就很實用。 本主題詳細說明如何建立提示並從瀑布對話進行呼叫，以實作簡單的交談流程。 

## <a name="prerequisites"></a>必要條件

- [Bot 基本概念][concept-basics]、[管理狀態][concept-state]和 [dialogs 程式庫][concept-dialogs]的知識。
- [**CSharp**][cs-sample] 或 [**JavaScript**][js-sample] 中的一份**多回合提示**範例。

## <a name="about-this-sample"></a>關於此範例

在多回合提示範例中，我們會使用瀑布式對話、一些提示和元件對話來建立簡單互動，以詢問使用者一系列的問題。 程式碼會使用對話方塊來循環下列步驟：

| 步驟        | 提示類型  |
|:-------------|:-------------| 
| 要求使用者的運輸模式 | 選擇提示 |
| 要求使用者的名稱 | 文字提示 |
| 詢問使用者是否要提供年齡 | 確認提示 |
| 如果他們回答 [是]，則要求年齡  | 包含驗證的數字提示，只接受大於 0 且小於 150 的年齡。 |
| 詢問是否可收集資訊 | 重複使用確認提示 |

最後，如果他們回答 [是]，則顯示所收集的資訊；否則，告訴使用者將不會保留其資訊。

## <a name="create-the-main-dialog"></a>建立主要對話

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

若要使用對話，請安裝 **Microsoft.Bot.Builder.Dialogs** NuGet 套件。

Bot 會透過 `UserProfileDialog` 與使用者互動。 當我們建立 Bot 的 `DialogBot` 類別時，我們會設定 `UserProfileDialog` 作為其主要對話。 Bot 接著會使用 `Run` 協助程式方法來存取此對話。

![使用者設定檔對話](media/user-profile-dialog.png)

**Dialogs\UserProfileDialog.cs**

我們首先會建立 `UserProfileDialog`，其衍生自 `ComponentDialog` 類別並具有 6 個步驟。

在 `UserProfileDialog` 建構函式中，建立瀑布式步驟、提示和瀑布式對話，然後將其新增至對話集。 提示必須位於其使用所在的相同對話集中。

[!code-csharp[Constructor snippet](~/../botbuilder-samples/samples/csharp_dotnetcore/05.multi-turn-prompt/Dialogs/UserProfileDialog.cs?range=22-41)]

接下來，我們會實作對話使用的步驟。 若要使用提示，請從您對話中的步驟中呼叫，並在下列步驟中使用 `stepContext.Result` 擷取提示結果。 在幕後，提示為兩個步驟的對話方塊。 第一步，提示會要求輸入；第二步，其會傳回有效值，或利用重新提示從頭開始，直到其收到有效輸入為止。

您應該一律從瀑布式步驟傳回非 Null 的 `DialogTurnResult`。 如果不這麼做，您的對話可能無法依照設計方式運作。 我們在此示範如何在瀑布式對話中實作 `NameStepAsync`。

[!code-csharp[Name step](~/../botbuilder-samples/samples/csharp_dotnetcore/05.multi-turn-prompt/Dialogs/UserProfileDialog.cs?range=56-61)]

在 `AgeStepAsync` 中，我們會在使用者的輸入驗證失敗時，指定重試提示，而驗證失敗是因為提示無法剖析其格式，或輸入不符合驗證準則。 在此情況下，如果未提供任何重試提示，則提示會使用初始提示文字來重新提示使用者提供輸入。

[!code-csharp[Age step](~/../botbuilder-samples/samples/csharp_dotnetcore/05.multi-turn-prompt/Dialogs/UserProfileDialog.cs?range=74-93&highlight=10)]

**UserProfile.cs**

使用者的運輸方式、名稱和年齡都會儲存在 `UserProfile` 類別的執行個體中。

[!code-csharp[UserProfile class](~/../botbuilder-samples/samples/csharp_dotnetcore/05.multi-turn-prompt/UserProfile.cs?range=9-16)]

**Dialogs\UserProfileDialog.cs**

在最後一個步驟中，我們會檢查由前一個瀑布式步驟中呼叫的對話所傳回的 `stepContext.Result`。 如果傳回的值為 true，我們會使用使用者設定檔存取子來取得及更新使用者設定檔。 若要取得使用者設定檔，我們會呼叫 `GetAsync` 方法，然後設定 `userProfile.Transport`、`userProfile.Name` 和 `userProfile.Age` 屬性的值。 最後，我們會先摘要說明使用者資訊，再呼叫 `EndDialogAsync` 來結束對話。 結束對話就會將其從對話堆疊中取出，並將選擇性結果傳回給對話的父代。 父代是開始剛結束之對話的對話或方法。

[!code-csharp[SummaryStepAsync](~/../botbuilder-samples/samples/csharp_dotnetcore/05.multi-turn-prompt/Dialogs/UserProfileDialog.cs?range=108-134&highlight=5-10,25-26)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

若要使用對話，您的專案需要安裝 **botbuilder-dialogs** npm 套件。

Bot 會透過 `UserProfileDialog` 與使用者互動。 當我們建立 Bot 的 `DialogBot` 時，我們會設定 `UserProfileDialog` 作為其主要對話。 Bot 接著會使用 `run` 協助程式方法來存取此對話。

![使用者設定檔對話](media/user-profile-dialog-js.png)

**dialogs\userProfileDialog.js**

我們首先會建立 `UserProfileDialog`，其衍生自 `ComponentDialog` 類別並具有 6 個步驟。

在 `UserProfileDialog` 建構函式中，建立瀑布式步驟、提示和瀑布式對話，然後將其新增至對話集。 提示必須位於其使用所在的相同對話集中。

[!code-javascript[Constructor snippet](~/../botbuilder-samples/samples/javascript_nodejs/05.multi-turn-prompt/dialogs/userProfileDialog.js?range=25-47)]

接下來，我們會實作對話使用的步驟。 若要使用提示，請從您對話中的步驟中呼叫，並從步驟內容的下列步驟中擷取提示結果，在此例中請使用 `step.result`。 在幕後，提示為兩個步驟的對話方塊。 第一步，提示會要求輸入；第二步，其會傳回有效值，或利用重新提示從頭開始，直到其收到有效輸入為止。

您應該一律從瀑布式步驟傳回非 Null 的 `DialogTurnResult`。 如果不這麼做，您的對話可能無法依照設計方式運作。 我們在此示範如何在瀑布式對話中實作 `nameStep`。

[!code-javascript[name step](~/../botbuilder-samples/samples/javascript_nodejs/05.multi-turn-prompt/dialogs/userProfileDialog.js?range=75-78)]

在 `ageStep` 中，我們會在使用者的輸入驗證失敗時指定重試提示，而驗證失敗是因為提示無法剖析其格式，或輸入不符合驗證準則 (在前述建構函式中指定)。 在此情況下，如果未提供任何重試提示，則提示會使用初始提示文字來重新提示使用者提供輸入。

[!code-javascript[age step](~/../botbuilder-samples/samples/javascript_nodejs/05.multi-turn-prompt/dialogs/userProfileDialog.js?range=90-101&highlight=5)]

**userProfile.js**

使用者的運輸方式、名稱和年齡都會儲存在 `UserProfile` 類別的執行個體中。

[!code-javascript[user profile](~/../botbuilder-samples/samples/javascript_nodejs/05.multi-turn-prompt/userProfile.js?range=4-10)]

**Dialogs\UserProfileDialog.cs**

在最後一個步驟中，我們會檢查由前一個瀑布式步驟中呼叫的對話所傳回的 `step.result`。 如果傳回的值為 true，我們會使用使用者設定檔存取子來取得及更新使用者設定檔。 若要取得使用者設定檔，我們會呼叫 `get` 方法，然後設定 `userProfile.transport`、`userProfile.name` 和 `userProfile.age` 屬性的值。 最後，我們會先摘要說明使用者資訊，再呼叫 `endDialog` 來結束對話。 結束對話就會將其從對話堆疊中取出，並將選擇性結果傳回給對話的父代。 父代是開始剛剛結束對話的對話或方法。

[!code-javascript[summary step](~/../botbuilder-samples/samples/javascript_nodejs/05.multi-turn-prompt/dialogs/userProfileDialog.js?range=115-136&highlight=4-8,20-21)]

---

## <a name="create-the-extension-method-to-run-the-waterfall-dialog"></a>建立擴充方法來執行瀑布式對話

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

我們已定義 `Run` 擴充方法，其將用於建立及存取對話內容。 在此，`accessor` 是對話狀態屬性的狀態屬性存取子，而 `dialog` 是使用者設定檔元件對話。 由於元件對話會定義內部對話集，因此我們必須建立可讓訊息處理常式程式碼看見的外部對話集，並使用其建立對話內容。

對話內容是經由呼叫 `CreateContext` 方法來建立，且用於在 Bot 的回合處理常式內與對話集互動。 對話內容包含目前的回合內容、父代對話，以及對話狀態，可供保留對話中的資訊。

對話內容可讓您開始一個具有字串識別碼的對話，或繼續目前的對話 (例如具有多個步驟的瀑布式對話)。 對話內容會傳遞至 Bot 的對話和瀑布式步驟。

**DialogExtensions.cs**

[!code-csharp[Run method](~/../botbuilder-samples/samples/csharp_dotnetcore/05.multi-turn-prompt/DialogExtensions.cs?range=13-24)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

我們已在 `userProfileDialog` 內定義 `run` 協助程式方法，其將用於建立及存取對話內容。 在此，`accessor` 是對話狀態屬性的狀態屬性存取子，而 `this` 是使用者設定檔元件對話。 由於元件對話會定義內部對話集，因此我們必須建立可讓訊息處理常式程式碼看見的外部對話集，並使用其建立對話內容。

對話內容是經由呼叫 `createContext` 方法來建立，且用於在 Bot 的回合處理常式內與對話集互動。 對話內容包含目前的回合內容、父代對話，以及對話狀態，可供保留對話中的資訊。

對話內容可讓您開始一個具有字串識別碼的對話，或繼續目前的對話 (例如具有多個步驟的瀑布式對話)。 對話內容會傳遞至 Bot 的對話和瀑布式步驟。

[!code-javascript[run method](~/../botbuilder-samples/samples/javascript_nodejs/05.multi-turn-prompt/dialogs/userProfileDialog.js?range=55-64)]

---

## <a name="run-the-dialog"></a>執行對話

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

**Bots\DialogBot.cs**

`OnMessageActivityAsync` 處理常式會使用擴充方法來開始或繼續對話。 在 `OnTurnAsync` 中，我們會使用 Bot 的狀態管理物件將任何狀態變更保存到儲存體。 (`ActivityHandler.OnTurnAsync` 方法會呼叫各種活動處理常式方法，例如 `OnMessageActivityAsync`。 如此一來，我們會在訊息處理常式完成後，但在回合本身完成之前儲存狀態。)

[!code-csharp[overrides](~/../botbuilder-samples/samples/csharp_dotnetcore/05.multi-turn-prompt/Bots/DialogBot.cs?range=33-48&highlight=5-7)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

`onMessage` 處理常式會使用協助程式方法來開始或繼續對話。 在 `onDialog` 中，我們會使用 Bot 的狀態管理物件將任何狀態變更保存到儲存體。 (在其他已定義的處理常式 (例如 `onMessage`) 執行之後，最後會呼叫 `onDialog` 方法。 如此一來，我們會在訊息處理常式完成後，但在回合本身完成之前儲存狀態。)

[!code-javascript[overrides](~/../botbuilder-samples/samples/javascript_nodejs/05.multi-turn-prompt/bots/dialogBot.js?range=30-44)]

---

## <a name="register-services-for-the-bot"></a>為 Bot 註冊服務

此 Bot 會使用下列_服務_。

- Bot 的基本服務：認證提供者、配接器及 Bot 實作。
- 用於管理狀態的服務：儲存體、使用者狀態及交談狀態。
- Bot 會使用的對話。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

**Startup.cs**

我們會在 `Startup` 中為 Bot 註冊服務。 這些服務可透過相依性插入來提供給程式碼的其他部分。

[!code-csharp[ConfigureServices](~/../botbuilder-samples/samples/csharp_dotnetcore/05.multi-turn-prompt/Startup.cs?range=17-41)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**index.js**

我們會在 `index.js` 中為 Bot 註冊服務。 

[!code-javascript[overrides](~/../botbuilder-samples/samples/javascript_nodejs/05.multi-turn-prompt/index.js?range=18-49)]

---

> [!NOTE]
> 記憶體儲存體僅供測試用途，而不適用於生產環境。
> 針對生產 Bot，請務必使用永續性的儲存體類型。

## <a name="to-test-the-bot"></a>測試 Bot

1. 如果您尚未安裝 [Bot Framework Emulator](https://aka.ms/bot-framework-emulator-readme)，請進行安裝。
1. 在您的電腦本機執行範例。
1. 啟動模擬器、連線到您的 Bot 並傳送如下所示的訊息。

![多回合提示對話的執行範例](../media/emulator-v4/multi-turn-prompt.png)

## <a name="additional-information"></a>其他資訊

### <a name="about-dialog-and-bot-state"></a>關於對話方塊和 Bot 狀態

在此 Bot 中，我們已定義兩個狀態屬性存取子：

- 一個建立於對話狀態屬性的對話方塊狀態內。 對話方塊狀態會追蹤使用者在對話方塊集中對話方塊的位置，並由對話方塊內容更新，例如當我們呼叫開始對話方塊或繼續對話方塊方法時。
- 一個建立於使用者設定檔屬性的使用者狀態內。 Bot 會使用此項來追蹤其使用者的相關資訊，而我們會在對話程式碼中明確地管理此狀態。

狀態屬性存取子的 _get_ 和 _set_ 方法，會取得及設定狀態管理物件的快取中的屬性值。 快取會第一次填入回合中要求的狀態屬性值，但必須明確地保存。 為了保存這兩個狀態屬性的變更，我們會呼叫對應狀態管理物件的「儲存變更」  方法。

此範例會在對話方塊內更新使用者設定檔狀態。 這種做法適用於簡單的 Bot，但如果您想要在所有 Bot 中重複使用一個對話方塊，則不可行。

有各種選項可將對話方塊步驟和 Bot 狀態分開。 例如，一旦對話方塊蒐集完整資訊，您即可：

- 使用結束對話方法，提供所收集的資料作為父代內容的傳回值。 這可以是 Bot 的回合處理常式，或對話方塊堆疊上先前作用中的對話方塊。 這就是提示類別的設計方式。
- 產生適當服務的要求。 如果 Bot 作為大型服務的前端，這可能效果良好。

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [將自然語言理解新增至您的 Bot](bot-builder-howto-v4-luis.md)

<!-- Footnote-style links -->

[concept-basics]: bot-builder-basics.md
[concept-state]: bot-builder-concept-state.md
[concept-dialogs]: bot-builder-concept-dialog.md

[prompting]: bot-builder-prompts.md
[component-dialogs]: bot-builder-compositcontrol.md

[cs-sample]: https://aka.ms/cs-multi-prompts-sample
[js-sample]: https://aka.ms/js-multi-prompts-sample
