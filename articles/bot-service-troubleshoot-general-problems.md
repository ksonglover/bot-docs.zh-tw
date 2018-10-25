---
title: 對 Bot 進行疑難排解 |Microsoft Docs
description: 使用技術性常見問題集來對 Bot 開發中的一般問題進行疑難排解。
author: DeniseMak
ms.author: v-demak
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 09/26/2018
ms.openlocfilehash: 3d9c41d0c0c51d00dc5ce86dfb774228e53ff3a3
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/24/2018
ms.locfileid: "49999065"
---
# <a name="troubleshooting-general-problems"></a>對一般問題進行疑難排解
這些常見問答集可協助您對常見的 Bot 開發或作業問題進行疑難排解。

## <a name="how-can-i-troubleshoot-issues-with-my-bot"></a>如何對 Bot 的問題進行疑難排解？

1. 使用 [Visual Studio Code](debug-bots-locally-vscode.md) 或 [Visual Studio](https://docs.microsoft.com/en-us/visualstudio/debugger/navigating-through-code-with-the-debugger?view=vs-2017) 針對 Bot 的原始程式碼進行偵錯。
2. 在將 Bot 部署至雲端之前，使用[模擬器](bot-service-debug-emulator.md)來測試它。
3. 將 Bot 部署至如 Azure 的雲端主機平台，然後在 <a href="https://dev.botframework.com" target="_blank">Bot Framework 入口網站</a> \(英文\) 中使用 Bot 儀表板上的內建網路聊天控制項來測試 Bot 的連線能力。 如果您在將 Bot 部署至 Azure 後遇到問題，則可以考慮使用這篇部落格文章：[了解 Azure 疑難排解和支援](https://azure.microsoft.com/en-us/blog/understanding-azure-troubleshooting-and-support/)。
4. 排除問題是由[驗證][TroubleshootingAuth]所導致的可能性。
5. 在 Skype 上測試您的 Bot。 這將能協助您驗證端對端的使用者體驗。
6. 請考慮在具有額外驗證需求的通道 (例如直接線路或網路聊天) 上測試 Bot。

## <a name="how-can-i-troubleshoot-authentication-issues"></a>如何對驗證問題進行疑難排解？

如需有關對 Bot 驗證問題進行疑難排解的詳細資料，請參閱[針對 Bot Framework 驗證進行疑難排解][TroubleshootingAuth]。

## <a name="im-using-the-bot-builder-sdk-for-net-how-can-i-troubleshoot-issues-with-my-bot"></a>我是使用適用於 .NET 的 Bot 建立器 SDK。 如何對 Bot 的問題進行疑難排解？

**尋找例外狀況。**  
在 Visual Studio 2017 中，移至 [偵錯] > [Windows] > [例外狀況設定]。 在 [例外狀況設定] 視窗中，選取位於 [Common Language Runtime 例外狀況] 旁邊的 [當擲回時中斷] 核取方塊。 當有擲回或未處理的例外狀況時，您也可以查看 [輸出] 視窗中的診斷輸出。

**查看呼叫堆疊。**  
在 Visual Studio 中，您可以選擇是否要對 [Just My Code](https://msdn.microsoft.com/en-us/library/dn457346.aspx) 進行偵錯。 檢查完整的呼叫堆疊可能可以針對問題提供額外的見解。

**確保所有對話方法都會以能處理下一個訊息的方案作為結尾。**  
所有對話方塊步驟都必須饋送到瀑布的下一個步驟，或結束目前的對話方塊才能脫離堆疊。 如果未正確處理步驟，如您所預期，將不會繼續對話。 查看[對話方塊](v4sdk/bot-builder-concept-dialog.md)概念文章，更進一步了解對話方塊。

## <a name="why-doesnt-the-typing-activity-do-anything"></a>為何輸入活動不會執行任何動作？
某些通道在其用戶端中並不支援暫時性輸入更新。

## <a name="what-is-the-difference-between-the-connector-library-and-builder-library-in-the-sdk"></a>SDK 中的連接器程式庫和建立器程式庫有何差異？

連接器程式庫為 REST API 的說明。 建立器程式庫能加入交談式對話程式設計模型及其他功能，例如通知、瀑布圖、鏈結，以及引導式表單完成功能。 建立器程式庫也能提供如 LUIS 等認知服務的存取。

## <a name="what-causes-an-error-with-http-status-code-429-too-many-requests"></a>造成具有 HTTP 狀態碼 429「太多要求」之錯誤的原因為何？

具有 HTTP 狀態碼 429 的錯誤回應，表示在特定期間內發出太多要求。 回應的主體應該會包含問題的解釋，也可能會指定要求之間所需的最低間隔。 此錯誤其中一個可能的來源為 [ngrok](https://ngrok.com/) \(英文\)。 如果您是使用免費方案並達到 ngrok 的限制，請移至其網站的定價與限制頁面以取得更多[選項](https://ngrok.com/product#pricing) \(英文\)。 

## <a name="why-arent-my-bot-messages-getting-received-by-the-user"></a>我的 Bot 訊息為何不是由使用者接收？

回應中產生的訊息活動必須正確地定址，否則無法送達其目的地。 在大部分情況下，您不需要明確處理此事，SDK 會負責為您將訊息活動定址。 

將活動正確地定址，表示包含適當的「對話參考」詳細資料，以及傳送者和接收者的相關詳細資料。 在大部分情況下，訊息活動會以回應形式傳送至已送達的其中一個活動。 因此，可以從輸入活動取得定址詳細資料。 

如果您檢查追蹤或稽核記錄，您可以檢查以確保您的訊息正確地定址。 若非如此，請在 Bot 中設定中斷點，並查看在何處針對您的訊息設定識別碼。

## <a name="how-can-i-run-background-tasks-in-aspnet"></a>如何在 ASP.NET 中執行背景工作？ 

在某些情況下，您可能會想要起始非同步工作以在幾秒後執行某些程式碼，來清除使用者設定檔或重設交談/對話狀態。 如需達成此目的的詳細資料，請參閱[如何在 ASP.NET 中執行背景工作](https://www.hanselman.com/blog/HowToRunBackgroundTasksInASPNET.aspx) \(英文\)。 請特別考慮使用 [HostingEnvironment.QueueBackgroundWorkItem](https://msdn.microsoft.com/en-us/library/dn636893(v=vs.110).aspx) \(機器翻譯\)。 


## <a name="how-do-user-messages-relate-to-https-method-calls"></a>使用者訊息和 HTTPS 方法呼叫有何關聯？

當使用者透過通道傳送訊息時，Bot Framework Web 服務將會對 Bot 的 Web 服務端點發出 HTTPS POST。 該 Bot 會透過針對其所傳送的每個訊息向 Bot Framework 發出個別的 HTTPS POST，於該通道上回傳零個、一個或多個訊息給使用者。

## <a name="my-bot-is-slow-to-respond-to-the-first-message-it-receives-how-can-i-make-it-faster"></a>Bot 對其所接收到的第一則訊息的回應速度很慢。 要如何讓它快一點？

Bot 為 Web 服務，而某些主機平台 (包括 Azure) 在該服務於一段時間內沒有接收到流量時，便會自動使該服務進入睡眠狀態。 如果您的 Bot 發生此情況，它於下次接收到訊息時便必須從頭重新啟動，這會讓它的回應速度比起已在執行情況下的速度還要慢上許多。

某些主機平台會允許您設定服務，使它不會進入睡眠狀態。 如果要在 Azure 中執行此步驟，請在 [Azure 入口網站](https://portal.azure.com)中瀏覽至 Bot 的服務，選取 [應用程式設定]，然後選取 [一律開啟]。 此選項可供大部分 (但非全部) 的服務方案使用。

## <a name="how-can-i-guarantee-message-delivery-order"></a>如何保證訊息傳遞順序？

Bot Framework 將會盡可能地保留訊息排序。 例如，如果您先傳送訊息 A 並等候該 HTTP 作業完成，然後才起始另一個 HTTP 作業以傳送訊息 B，則 Bot Framework 將會自動了解訊息 A 應該排在訊息 B 之前。不過，一般而言 Bot Framework 並無法保證訊息傳遞順序，因為通道是最終負責傳遞訊息的角色，並可能會對訊息進行重新排序。 如果要降低訊息以錯誤順序傳遞的風險，您可以考慮在訊息之間實作時間延遲。

## <a name="how-can-i-intercept-all-messages-between-the-user-and-my-bot"></a>如何攔截使用者與我的 Bot 之間的所有訊息？

使用適用於 .NET 的 Bot 建立器 SDK 時，您可以對 `Autofac` 相依性插入容器提供 `IPostToBot` 和 `IBotToUser` 介面的實作。 使用適用於任何語言的 Bot Builder SDK 時，您可以針對大致相同的目的使用中介軟體。 [BotBuilder-Azure](https://github.com/Microsoft/BotBuilder-Azure) \(英文\) 存放庫包含 C# 和 Node.js 程式庫，並能將此資料記錄至 Azure 資料表。

## <a name="why-are-parts-of-my-message-text-being-dropped"></a>我的訊息文字為何有一部分被卸除？

Bot Framework 和許多通道都會把文字當成是以 [Markdown](https://en.wikipedia.org/wiki/Markdown) 進行格式化。 請檢查您的文字是否包含可能會被解譯為 Markdown 語法的字元。

## <a name="how-can-i-support-multiple-bots-at-the-same-bot-service-endpoint"></a>如何在同一個 Bot 服務端點支援多個 Bot？ 

此[範例](https://github.com/Microsoft/BotBuilder/issues/2258#issuecomment-280506334) \(英文\) 示範如何以正確的 `MicrosoftAppCredentials` 設定 `Conversation.Container`，並使用簡單的 `MultiCredentialProvider` 來驗證多個應用程式識別碼與密碼。

## <a name="identifiers"></a>識別項

## <a name="how-do-identifiers-work-in-the-bot-framework"></a>識別碼在 Bot Framework 中的運作方式為何？

如需有關 Bot Framework 中識別碼的詳細資料，請參閱[針對識別碼的 Bot Framework 指南][BotFrameworkIDGuide]。

## <a name="how-can-i-get-access-to-the-user-id"></a>如何存取使用者識別碼？

SMS 和電子郵件訊息將會在 `from.Id` 屬性中提供未經處理的使用者識別碼。 在 Skype 訊息中，`from.Id` 屬性將會包含適用於該使用者的唯一識別碼，其有別於該使用者的 Skype 識別碼。 如果您需要連線至現有帳戶，您可以使用登入卡並實作自己的 OAuth 流程，以將使用者識別碼連線至您自己服務的使用者識別碼。

## <a name="why-are-my-facebook-user-names-not-showing-anymore"></a>為何已不再顯示我的 Facebook 使用者名稱？

您是否有變更您的 Facebook 密碼？ 這麼做將會使存取權杖無效，且您必須在 <a href="https://dev.botframework.com" target="_blank">Bot Framework 入口網站</a> \(英文\) 中針對 Facebook Messenger 通道更新您 Bot 的組態設定。

## <a name="why-is-my-kik-bot-replying-im-sorry-i-cant-talk-right-now"></a>為何我的 Kik Bot 會回覆 "I'm sorry, I can't talk right now" (抱歉，我現在無法交談)？

在 Kik 上開發的 Bot 僅允許有 50 個訂閱者。 在有 50 個唯一使用者與您的 Bot 互動之後，任何嘗試與 Bot 聊天的新使用者，都會接收到訊息 "I'm sorry, I can't talk right now" (抱歉，我現在無法交談)。 如需詳細資訊，請參閱 [Kik 文件](https://botsupport.kik.com/hc/en-us/articles/225764648-How-can-I-share-my-bot-with-Kik-users-while-in-development-) \(英文\)。

## <a name="how-can-i-use-authenticated-services-from-my-bot"></a>如何從 Bot 使用已驗證的服務？

如需 Azure Active Directory 驗證，請參閱新增驗證 [V3](https://docs.microsoft.com/en-us/azure/bot-service/bot-builder-tutorial-authentication?view=azure-bot-service-3.0&tabs=csharp) | [V4](https://docs.microsoft.com/en-us/azure/bot-service/bot-builder-tutorial-authentication?view=azure-bot-service-4.0&tabs=csharp)。 

> [!NOTE] 
> 如果您有將驗證和安全性功能加入您的 Bot，您應該確保自己在程式碼中所實作的模式會符合適用於您應用程式的安全性標準。

## <a name="how-can-i-limit-access-to-my-bot-to-a-pre-determined-list-of-users"></a>如何限制只有預先決定清單中的使用者才能存取我的 Bot？

某些通道 (例如 SMS 和電子郵件) 會提供不限範圍的位址。 在這些情況下，來自使用者的訊息將會在 `from.Id` 屬性中包含未經處理的使用者識別碼。

其他通道 (例如 Skype、Facebook 和 Slack) 會提供	有範圍或承租的位址，這會使 Bot 無法預估使用者的識別碼。 在這些情況下，您將會需要透過登入連結或共用祕密來驗證使用者，以判斷該使用者是否有使用 Bot 的授權。

## <a name="why-does-my-direct-line-11-conversation-start-over-after-every-message"></a>為何我的直接線路 1.1 交談會在每則訊息之後重新開始？

如果您的直接線路交談會在每則訊息之後重新開始，便代表直接線路傳送至 Bot 之訊息中的 `from` 屬性很可能已經遺失或為 `null`。 當直接線路用戶端所傳送之訊息中的 `from` 屬性遺失或為 `null` 時，直接線路服務會自動配置識別碼，這會使用戶端所傳送的每則訊息都會像是來自全新且不同的使用者。

如果要修正此問題，請將直接線路用戶端所傳送之每則訊息中的 `from` 屬性，設定為能唯一代表傳送該訊息之使用者的穩定值。 例如，如果使用者已經登入網頁或應用程式，您便可以將該使用者的現有識別碼，作為其所傳送之訊息中 `from` 屬性的值。 或者，您可以選擇在頁面載入或應用程式載入時產生隨機的使用者識別碼，將該識別碼儲存於 Cookie 或裝置狀態中，然後將該識別碼作為使用者所傳送之訊息中 `from` 屬性的值。

## <a name="what-causes-the-direct-line-30-service-to-respond-with-http-status-code-502-bad-gateway"></a>造成直接線路 3.0 服務回應 HTTP 狀態碼 502「不正確的閘道」的原因為何？
直接線路 3.0 會在嘗試連絡 Bot 但無法成功完成要求的情況下，傳回 HTTP 狀態碼 502。 此錯誤表示 Bot 傳回錯誤或要求逾時。如需 Bot 會產生之錯誤的詳細資訊，請在 <a href="https://dev.botframework.com" target="_blank">Bot Framework 入口網站</a> \(英文\) 內移至 Bot 的儀表板，然後按一下 [問題] 連結以了解受影響的通道。 如果您已經針對 Bot 設定 Application Insights，便也可以在那裡找到詳細的錯誤資訊。 

::: moniker range="azure-bot-service-3.0"

## <a name="what-is-the-idialogstackforward-method-in-the-bot-builder-sdk-for-net"></a>適用於 .NET 的 Bot 建立器 SDK 中的 IDialogStack.Forward 方法是什麼？

`IDialogStack.Forward` 的主要目的是重複使用經常為「被動」的現有子對話；在此情況下，(`IDialog.StartAsync` 中的) 子對話會針對某些 `ResumeAfter` 處理常式等候物件 `T`。 特別是在具有會等候 `IMessageActivity` `T` 之子對話的情況下，您可以使用 `IDialogStack.Forward` 方法來轉送傳入的 `IMessageActivity` (已由某些父對話接收)。 例如，若要將傳入的 `IMessageActivity` 轉送至 `LuisDialog`，請呼叫 `IDialogStack.Forward` 以將 `LuisDialog` 推送至對話堆疊上，在 `LuisDialog.StartAsync` 中執行程式碼直到其針對下則訊息做出等候的排程為止，然後立即透過轉送的 `IMessageActivity` 來滿足該等候。

`T` 通常是 `IMessageActivity`，因為 `IDialog.StartAsync` 通常會建構為等候該類型的活動。 您可以針對 `LuisDialog` 使用 `IDialogStack.Forward` 作為攔截來自使用者之訊息的機制，以在將該訊息轉送至現有 `LuisDialog` 之前進行某些處理。 或者，您也可以針對該目的搭配 `ContinueToNextGroup` 使用 `DispatchDialog`。

您可以預期在由 `StartAsync` 所排程的第一個 `ResumeAfter` 處理常式 (例如 `LuisDialog.MessageReceived`) 中找到已轉送的項目。

::: moniker-end

## <a name="what-is-the-difference-between-proactive-and-reactive"></a>「主動」和「被動」之間的差異為何？

從您 Bot 的觀點來看，「被動」表示交談是由使用者傳送訊息給 Bot 來起始，而 Bot 則是透過回應該訊息來反應。 相反地，「主動」表示交談是由 Bot 傳送第一則訊息給使用者來起始。 例如，Bot 可能會傳送主動式訊息以通知使用者計時器已到期，或是已發生某個事件。

## <a name="how-can-i-send-proactive-messages-to-the-user"></a>如何傳送主動式訊息給使用者？

如需如何傳送主動式訊息的範例，請參閱 GitHub 上 BotBuilder-Samples 存放庫內的 [C# V4 範例](https://github.com/Microsoft/BotBuilder-Samples/tree/master/samples/csharp_dotnetcore/16.proactive-messages)和 [Node.js V4 範例](https://github.com/Microsoft/BotBuilder-Samples/tree/master/samples/javascript_nodejs/16.proactive-messages)。

## <a name="how-can-i-reference-non-serializable-services-from-my-c-dialogs"></a>如何參考 C# 對話中不可序列化的服務？

有數個選項︰

* 透過 `Autofac` 和 `FiberModule.Key_DoNotSerialize`解析相依性。 這是最乾淨的解決方案。
* 使用 [NonSerialized](https://msdn.microsoft.com/en-us/library/system.nonserializedattribute(v=vs.110).aspx) \(機器翻譯\) 和 [OnDeserialized](https://msdn.microsoft.com/en-us/library/system.runtime.serialization.ondeserializedattribute(v=vs.110).aspx) \(機器翻譯\) 屬性來還原針對還原序列化的相依性。 這是最簡單的解決方案。
* 不儲存相依性，以使它不會被序列化。 此解決方案就技術上而言雖然可行，但並不建議。
* 使用反映序列化代理。 此解決方案在某些情況下可能不可行，並會有過度序列化的風險。

::: moniker range="azure-bot-service-3.0"

## <a name="where-is-conversation-state-stored"></a>交談狀態的儲存位置為何？

使用者、交談及私人交談屬性包中的資料，會使用連接器的 `IBotState` 介面進行儲存。 每個屬性包都會依 Bot 的識別碼設定範圍。 使用者屬性包是依使用者識別碼進行索引、交談屬性包是由交談識別碼進行索引，而私人交談屬性包則是由使用者識別碼和交談識別碼進行索引。 

如果您是使用適用於 .NET 的 Bot 建立器 SDK 或適用於 Node.js 的 Bot 建立器 SDK 來建置 Bot，對話堆疊和對話資料都將會自動儲存為私人交談屬性包中的項目。 C# 實作會使用二進位序列化，而 Node.js 實作則會使用 JSON 序列化。

`IBotState` REST 介面是由兩個服務所實作。

* Bot Framework 連接器會提供能實作此介面並將資料儲存於 Azure 中的雲端服務。  此資料會進行待用加密，且不會刻意到期。
* Bot Framework 模擬器會提供此介面的記憶體內部實作，以對 Bot 進行偵錯。 此資料會在模擬器處理序結束時到期。

如果您想要將此資料儲存在資料中心內，則可以提供狀態服務的自訂實作。 這可以透過至少兩種方式來完成：

* 使用 REST 層來提供自訂 `IBotState` 服務。
* 在語言 (Node.js 或 C#) 層中使用建立器介面。

> [!IMPORTANT]
> 建議您不要將 Bot Framework 狀態服務 API 用於生產環境，未來版本可能會將它淘汰。 建議更新您的 Bot 程式碼以使用記憶體內部儲存體進行測試，或將其中一個 **Azure 擴充功能**用於生產環境 Bot。 如需詳細資訊，請參閱適用於 [.NET](~/dotnet/bot-builder-dotnet-state.md) 或 [Node](~/nodejs/bot-builder-nodejs-state.md) 實作的**管理狀態資料**主題。

::: moniker-end

## <a name="what-is-an-etag--how-does-it-relate-to-bot-data-bag-storage"></a>什麼是 ETag？  它與 Bot 資料包儲存體之間的關聯為何？

[ETag](https://en.wikipedia.org/wiki/HTTP_ETag) 為適用於[開放式並行存取控制](https://en.wikipedia.org/wiki/Optimistic_concurrency_control)的機制。 Bot 資料包儲存體會使用 ETag 來防止資料更新衝突。 具有 HTTP 狀態碼 412「前置條件失敗」的 ETag 錯誤，表示針對該 Bot 資料包有多個並行執行的「讀取-修改-寫入」序列。

對話堆疊和狀態會儲存到 Bot 資料包中。 例如，在 Bot 仍在處理前一則訊息時便接收到新交談訊息的情況下，您可能就會看見「前置條件失敗」ETag 錯誤。

## <a name="what-causes-an-error-with-http-status-code-412-precondition-failed-or-http-status-code-409-conflict"></a>造成具有 HTTP 狀態碼 412「前置條件失敗」或 HTTP 狀態碼 409「衝突」之錯誤的原因為何？

連接器的 `IBotState` 服務是用來儲存 Bot 資料包 (也就是使用者、交談及私人 Bot 資料包，其中私人 Bot 資料包會包含「控制流程」狀態)。 `IBotState` 服務中的並行控制是透過 ETag 由開放式並行存取進行管理。 如果在「讀取-修改-寫入」序列期間發生更新衝突 (基於針對單一 Bot 資料包的並行更新)，則：

* 如果有保留 ETag，`IBotState` 服務會擲回具有 HTTP 狀態碼 412「前置條件失敗」的錯誤。 這是適用於 .NET 的 Bot 建立器 SDK 中的預設行為。
* 如果未保留 ETag (也就是已將 ETag 設定為 `\*`)，則「最後寫入者優先」的原則將會生效，並會在具有資料遺失風險的情況下防止發生「前置條件失敗」錯誤。 這是適用於 Node.js 的 Bot 建立器 SDK 中的預設行為。

## <a name="how-can-i-fix-precondition-failed-412-or-conflict-409-errors"></a>如何修正「前置條件失敗」(412) 或「衝突」(409) 錯誤？

這些錯誤表示您的 Bot 針對相同的交談同時處理多則訊息。 如果您的 Bot 已連線至要求精準排序之訊息的服務，您應該考慮鎖定交談狀態，以確保訊息不會以平行方式處理。 

::: moniker range="azure-bot-service-3.0"

適用於 .NET 的 Bot 建立器 SDK 會提供機制 (會實作 `IScope` 的類別 `LocalMutualExclusion`) 以搭配記憶體內部旗號，來以保守模式序列化單一交談的處理。 您可以延伸此實作，以使用由交談位址設定範圍的 Redis 租用。

如果您的 Bot 未連線至外部服務，或服務可接受以平行方式處理來自相同交談的訊息，您便可以加入此程式碼以忽略發生在 Bot 狀態 API 中的任何衝突。 這將會允許最後的回覆設定交談狀態。

```cs
var builder = new ContainerBuilder();
builder
    .Register(c => new CachingBotDataStore(c.Resolve<ConnectorStore>(), CachingBotDataStoreConsistencyPolicy.LastWriteWins))
    .As<IBotDataStore<BotData>>()
    .AsSelf()
    .InstancePerLifetimeScope();
builder.Update(Conversation.Container);
```
::: moniker-end

## <a name="is-there-a-limit-on-the-amount-of-data-i-can-store-using-the-state-api"></a>使用狀態 API 儲存的資料量是否有限制？

是，每個狀態存放區 (也就是使用者、交談及私人 Bot 資料包) 可以包含最多 64kb 的資料。 如需詳細資訊，請參閱[管理狀態資料][StateAPI]。

::: moniker range="azure-bot-service-3.0"

## <a name="how-do-i-version-the-bot-data-stored-through-the-state-api"></a>如何設定透過狀態 API 所儲存之 Bot 資料的版本？

> [!IMPORTANT]
> 建議您不要將 Bot Framework 狀態服務 API 用於生產環境或 v4 Bot，因為未來版本可能會完全淘汰。 建議更新您的 Bot 程式碼以使用記憶體內部儲存體進行測試，或將其中一個 **Azure 擴充功能**用於生產環境 Bot。 如需詳細資訊，請參閱[管理狀態資料](v4sdk/bot-builder-howto-v4-state.md)主題。

狀態服務可讓您保存交談中的對話進度，讓使用者於稍後繼續與 Bot 交談時不會失去原先的位置。 若要保留此進度，透過狀態 API 所儲存的 Bot 資料屬性包將不會於您修改 Bot 的程式碼時自動清除。 您應該根據已修改的程式碼是否能與舊版資料相容，來決定是否要清除 Bot 資料。 

* 如果您想要在開發 Bot 的期間手動重設交談的對話堆疊及狀態，則可以使用 ` /deleteprofile` 命令來刪除狀態資料。 請務必包含此命令的前置空格，以防止通道解譯它。
* 在您的 Bot 已部署至生產環境之後，您便可以對 Bot 資料進行版本設定，使系統會在您提升版本時清除相關聯的資料。 使用適用於 Node.js 的 Bot 建立器 SDK 時，可以使用中介軟體來達成此目的；使用適用於 .NET 的 Bot 建立器 SDK 時，則可以使用 `IPostToBot` 實作來達成此目的。

> [!NOTE]
> 如果對話堆疊因序列化格式變更或程式碼大幅變更而無法正確還原序列化，交談狀態將會重設。

::: moniker-end

## <a name="what-are-the-possible-machine-readable-resolutions-of-the-luis-built-in-date-time-duration-and-set-entities"></a>LUIS 內建日期、時間、持續時間及設定實體有哪些可能的電腦可讀取解決方法？

如需範例清單，請參閱 LUIS 文件的[預先建置實體][LUISPreBuiltEntities]小節。

## <a name="how-can-i-use-more-than-the-maximum-number-of-luis-intents"></a>如何使用超過最大值的 LUIS 意圖？

您可以考慮分割您的模型，並以序列或平行方式呼叫 LUIS 服務。

## <a name="how-can-i-use-more-than-one-luis-model"></a>如何使用多個 LUIS 模型？

適用於 Node.js 的 Bot 建立器 SDK 和適用於 .NET 的 Bot 建立器 SDK 皆支援從單一 LUIS 意圖對話呼叫多個 LUIS 模型。 請記住下列注意事項：

* 使用多個 LUIS 模型時，系統會假設 LUIS 模型具有未重疊的意圖集合。
* 使用多個 LUIS 模型時，系統會假設不同模型的分數是可以互相比較的，以從多個模型中選出「最相符的意圖」。
* 使用多個 LUIS 模型代表當某個意圖符合其中一個模型時，它也會強烈符合其他模型的「無」意圖。 在此情況下，您可以避免選取「無」意圖；適用於 Node.js 的 Bot 建立器 SDK 將會自動相應減少「無」意圖的分數以避免此問題。

## <a name="where-can-i-get-more-help-on-luis"></a>我可以在哪裡取得更多有關 LUIS 的協助？

* [Language Understanding (LUIS) 簡介 - Microsoft 認知服務](https://www.youtube.com/watch?v=jWeLajon9M8) \(英文\) (影片)
* [適用於 Language Understanding (LUIS) 的進階學習課程](https://www.youtube.com/watch?v=39L0Gv2EcSk) \(英文\) (影片)
* [LUIS 文件](/azure/cognitive-services/LUIS/Home)
* [Language Understanding 論壇](https://social.msdn.microsoft.com/forums/azure/en-US/home?forum=LUIS) 


## <a name="what-are-some-community-authored-dialogs"></a>有哪些由社群撰寫的對話？

* [BotAuth](https://www.nuget.org/packages/BotAuth) \(英文\) - Azure Active Directory 驗證
* [BestMatchDialog](http://www.garypretty.co.uk/2016/08/01/bestmatchdialog-for-microsoft-bot-framework-now-available-via-nuget/) \(英文\) - 以規則運算式為基礎的使用者文字至對話分派方法

## <a name="what-are-some-community-authored-templates"></a>有哪些由社群撰寫的範本？

* [ES6 BotBuilder](https://github.com/brene/botbuilder-es6-template) \(英文\) - ES6 Bot 建立器範本

## <a name="why-do-i-get-an-authorizationrequestdenied-exception-when-creating-a-bot"></a>為何在建立 Bot 時會收到 Authorization_RequestDenied 例外狀況？

建立 Azure Bot 服務 Bot 的權限是透過 Azure Active Directory (AAD) 入口網站來管理。 如果未在 [AAD 入口網站](http://aad.portal.azure.com)中正確設定權限，使用者在嘗試建立 Bot 服務時將會收到 **Authorization_RequestDenied** 例外狀況。

請先檢查您是否為該目錄的「來賓」：

1. 登入 [Azure 入口網站](http://portal.azure.com)。
2. 選取 [所有服務]，並搜尋 *active*。
3. 選取 **Azure Active Directory**。
4. 按一下 [使用者] 。
5. 從清單上找到該使用者，並確定 [使用者類型] 不是 [來賓]。

![Azure Active Directory 使用者類型](~/media/azure-active-directory/user_type.png)

在您確定自己不是**來賓**之後，若要確保有效目錄內的使用者可以建立 Bot 服務，目錄管理員必須設定下列設定：

1. 登入 [AAD 入口網站](http://aad.portal.azure.com)。 移至 [使用者和群組] 並選取 [使用者設定]。
2. 在 [應用程式註冊] 區段底下，將 [使用者可以註冊應用程式] 設定為 [是]。 這可讓您目錄中的使用者建立 Bot 服務。
3. 在 [外部使用者] 區段底下，將 [來賓使用者權限受限] 設定為 [否]。 這可讓您目錄中的來賓使用者建立 Bot 服務。

![Azure Active Directory 管理中心](~/media/azure-active-directory/admin_center.png)

## <a name="why-cant-i-migrate-my-bot"></a>為何我無法移轉 Bot？

如果您無法移轉 Bot，這可能是因為該 Bot 所屬的目錄不是您的預設目錄。 請嘗試這些步驟：

1. 在目標目錄中，新增不是預設目錄成員的新使用者 (透過電子郵件地址)，將要移轉之目標訂用帳戶上的參與者角色授與該使用者。

2. 在 [Dev Portal](https://dev.botframework.com) \(英文\) 中，將該使用者的電子郵件地址新增為要移轉之 Bot 的共同擁有者。 然後登出。

3. 以新使用者的身分登入 [Dev Portal](https://dev.botframework.com) \(英文\)，然後繼續移轉該 Bot。

## <a name="where-can-i-get-more-help"></a>我可以在哪裡取得更多協助？

* 在 [Stack Overflow](https://stackoverflow.com/questions/tagged/botframework) \(英文\) 上利用先前已解答之問題中的資訊，或使用 `botframework` 標記來張貼您自己的問題。 請注意，Stack Overflow 有一些使用上的指導方針，例如要求必須使用描述性的標題、完整且簡潔的問題陳述，以及能夠重現問題的足夠詳細資料。 功能要求或過於廣泛的問題都會被視為偏離主題。新的使用者應瀏覽 [Stack Overflow 說明中心](https://stackoverflow.com/help/how-to-ask) \(英文\) 以取得詳細資料。
* 請參閱 GitHub 中的 [BotBuilder 問題](https://github.com/Microsoft/BotBuilder/issues) \(英文\) 以取得 Bot 建立器 SDK 的相關已知問題，或回報新的問題。
* 利用 BotBuilder 社群在 [Gitter](https://gitter.im/Microsoft/BotBuilder) \(英文\) 上的討論內容中的資訊。




[LUISPreBuiltEntities]: /azure/cognitive-services/luis/pre-builtentities
[BotFrameworkIDGuide]: bot-service-resources-identifiers-guide.md
[StateAPI]: ~/rest-api/bot-framework-rest-state.md
[TroubleshootingAuth]: bot-service-troubleshoot-authentication-problems.md

