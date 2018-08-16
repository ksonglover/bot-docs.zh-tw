---
title: 如何使用主動式傳訊 | Microsoft Docs
description: 了解如何搭配 Bot 進行主動式傳訊。
keywords: 主動式訊息
author: jonathanfingold
ms.author: jonathanfingold
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 05/01/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: fd53a897d9847432fd337402d40edfcd6f4ff061
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/18/2018
ms.locfileid: "39299830"
---
# <a name="how-to-use-proactive-messaging"></a>如何使用主動式傳訊

Bot 通常是傳送_被動式訊息_，但我們有時也必須傳送[主動式訊息](bot-builder-proactive-messages.md)。 

常見的主動式傳訊案例為 Bot 正在執行無法確定所需時間的工作。 在此情況下，您可以儲存有關該工作的資訊，告訴使用者 Bot 將會在工作完成時通知他們，並讓該對話繼續進行。 當工作完成時，Bot 便可以主動地傳送確認訊息來繼續對話。

# <a name="ctabcs"></a>[C#](#tab/cs)

## <a name="notes-about-this-sample"></a>關於此範例的注意事項

我們會修改基本的 EchoBot 範例。
- 我們會使用 `Microsoft.Samples.Proactive` 作為命名空間。
- 我們會以 `JobData.cs` 檔案來取代狀態檔案。
- 我們會以 `ProactiveBot.cs` 檔案來取代 Bot 檔案。

> [!NOTE]
> 目前，主動式傳訊需要 Bot 具備有效的應用程式識別碼和密碼。


## <a name="define-task-data"></a>定義工作資料

在此案例中，我們會追蹤可由各種使用者在不同的對話中建立的任意工作。 因此我們會使用一般的 Bot 狀態中介軟體，而非使用者或對話狀態中介軟體。

下列類別會定義我們將用於個別作業的資料結構。


```csharp
using Microsoft.Bot.Schema;
using System.Collections.Generic;

namespace Microsoft.Samples.Proactive
{
    /// <summary>
    /// Class for storing job state. 
    /// </summary>
    public class JobData
    {
        /// <summary>
        /// The name to use to read and write this bot state object to storage.
        /// </summary>
        public readonly static string PropertyName = $"BotState:{typeof(Dictionary<int, JobData>).FullName}";

        public int JobNumber { get; set; } = 0;
        public bool Completed { get; set; } = false;

        /// <summary>
        /// The conversation reference to which to send status updates.
        /// </summary>
        public ConversationReference Conversation { get; set; }
    }
}
```


我們也需要將狀態中介軟體加入至啟始程式碼。


在 `StartUp.cs` 檔案中，更新 `ConfigureServices` 方法以將作業的字典加入至 Bot 狀態。 在下列程式碼中，它是針對 `options.Middleware.Add` 的最後一個呼叫。
```csharp
// This method gets called by the runtime. Use this method to add services to the container.
public void ConfigureServices(IServiceCollection services)
{
    services.AddBot<ProactiveBot>(options =>
    {
        options.CredentialProvider = new ConfigurationCredentialProvider(Configuration);

        // The CatchExceptionMiddleware provides a top-level exception handler for your bot. 
        // Any exceptions thrown by other Middleware, or by your OnTurn method, will be 
        // caught here. To facillitate debugging, the exception is sent out, via Trace, 
        // to the emulator. Trace activities are NOT displayed to users, so in addition
        // an "Ooops" message is sent. 
        options.Middleware.Add(new CatchExceptionMiddleware<Exception>(async (context, exception) =>
        {
            await context.TraceActivity($"{nameof(ProactiveBot)} Exception", exception);
            await context.SendActivity("Sorry, it looks like something went wrong!");
        }));

        // The Memory Storage used here is for local bot debugging only. When the bot
        // is restarted, anything stored in memory will be gone. 
        IStorage dataStore = new MemoryStorage();

        // Using the base BotState here, since the job log is not necessarily tied to a
        // specific user or conversation.
        options.Middleware.Add(
            new BotState<Dictionary<int, JobData>>(
                dataStore, JobData.PropertyName, (context) => $"jobs/{typeof(Dictionary<int, JobData>)}"));
    });
}
```


## <a name="update-your-bot-to-create-and-run-jobs"></a>更新 Bot 以建立並執行作業

在每一回合中，我們將會讓某個使用者輸入 `run` 或 `run job` 來建立作業。

作為回應，我們的 Bot 將會在該回合內執行下列步驟：
- 建立作業。
- 記錄有關目前對話的資訊，讓我們可以於稍後傳送主動式訊息。
- 讓使用者知道我們將啟動他們的作業，並會在稍後於作業完成時通知他們。
- 啟動非同步作業。
- 讓該回合結束。

我們要啟動的作業是簡單的 5 秒計時器，並會透過傳送主動式訊息來完成該作業。
- 呼叫配接器的持續對話方法會建立由 Bot 所起始的新回合。
- 此回合會有屬於自己的回合內容，我們會從其中擷取狀態資訊。
- 我們會使用此內容來將主動式訊息傳送給使用者。



> [!NOTE]
> `GetAppId` 方法是在 .NET SDK 中啟用主動式傳訊的因應措施。

```csharp
using Microsoft.Bot;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Connector.Authentication;
using Microsoft.Bot.Schema;
using System;
using System.Collections.Generic;
using System.Linq;
using System.Security.Claims;
using System.Security.Principal;
using System.Threading;
using System.Threading.Tasks;

namespace Microsoft.Samples.Proactive
{
    public class ProactiveBot : IBot
    {
        /// <summary>
        /// Random number generator for job numbers.
        /// </summary>
        private static Random NumberGenerator = new Random();

        /// <summary>
        /// Gets the job log from the bot state.
        /// </summary>
        /// <param name="context">The current turn context.</param>
        /// <returns>The job log.</returns>
        private static Dictionary<int, JobData> GetJobLog(ITurnContext context)
        {
            return context.Services.Get<Dictionary<int, JobData>>(JobData.PropertyName);
        }

        /// <summary>
        /// Workaround to get the bot's app ID.
        /// </summary>
        /// <param name="context">The current turn context.</param>
        /// <returns>The application ID for the bot.</returns>
        private static string GetAppId(ITurnContext context)
        {
            // The BotFrameworkAdapter sets the identity provider on the context object.
            var claimsIdentity = context.Services.Get<IIdentity>("BotIdentity") as ClaimsIdentity;

            // For requests from a channel, the app ID is in the Audience claim of the JWT token.
            // For requests from the emulator, it is in the AppId claim.
            // For unauthenticated requests, we have anonymouse identity provided auth is disabled.
            // For Activities coming from Emulator AppId claim contains the Bot's AAD AppId.
            var botAppIdClaim =
                (claimsIdentity.Claims?.SingleOrDefault(claim => claim.Type == AuthenticationConstants.AudienceClaim)
                ?? claimsIdentity.Claims?.SingleOrDefault(claim => claim.Type == AuthenticationConstants.AppIdClaim));

            return botAppIdClaim?.Value;
        }

        /// <summary>
        /// Every Conversation turn calls this method.
        /// When the user types "run" or "run job", the bot starts a "job".
        /// When the job finishes, the bot proactively notifies the user.
        /// </summary>
        /// <param name="context">The turn context.</param>
        /// <remarks>When our virtual job finishes, it sends a proactive message
        /// to notify the user that the job completed.</remarks>
        public async Task OnTurn(ITurnContext context)
        {
            // This bot is only handling Messages
            if (context.Activity.Type is ActivityTypes.Message)
            {
                var text = context.Activity.AsMessageActivity()?.Text?.Trim().ToLower();
                switch (text)
                {
                    case "run":
                    case "run job":

                        var jobLog = GetJobLog(context);
                        var job = CreateJob(context, jobLog);
                        var appId = GetAppId(context);
                        var conversation = TurnContext.GetConversationReference(context.Activity);

                        await context.SendActivity($"We're starting job {job.JobNumber} for you. We'll notify you when it's complete.");

                        // Since the context is disposed at the end of the turn, extract and send the
                        // information we need to send the proactive message later.
                        var adapter = context.Adapter;
                        Task.Run(() =>
                        {
                            // Simulate a separate process to complete the user's job.
                            Thread.Sleep(5000);

                            // Perform bookkeeping and send the proactive message.
                            CompleteJob(adapter, appId, conversation, job.JobNumber);
                        });

                        break;

                    default:

                        await context.SendActivity("Type 'run' or 'run job' to start a new job.");

                        break;
                }
            }
        }

        /// <summary>
        /// Creates a simulated job and updates the job log.
        /// </summary>
        /// <param name="context">The current turn context.</param>
        /// <param name="jobLog">The job log.</param>
        /// <returns>A new job.</returns>
        private JobData CreateJob(ITurnContext context, Dictionary<int, JobData> jobLog)
        {
            // Generate a non-duplicate job number;
            int number;
            while (jobLog.ContainsKey(number = NumberGenerator.Next())) { }

            // Simulate creaing the job and logging it.
            var job = new JobData
            {
                JobNumber = number,
                Conversation = TurnContext.GetConversationReference(context.Activity)
            };
            jobLog.Add(job.JobNumber, job);

            // Return the created job.
            return job;
        }

        /// <summary>
        /// Performs bookkeeping and proactively notifies the user that their job completed.
        /// </summary>
        /// <param name="adapter">The bot adapter with which to send the message.</param>
        /// <param name="appId">The app ID of the bot to send the message from.</param>
        /// <param name="conversation">The conversation in which to put the message.</param>
        /// <param name="jobNumber">The number of the job that completed.</param>
        private async void CompleteJob(BotAdapter adapter, string appId, ConversationReference conversation, int jobNumber)
        {
            await adapter.ContinueConversation(appId, conversation, async context =>
            {
                // Get the job log from state, and retrieve the job.
                var jobLog = GetJobLog(context);
                var job = jobLog[jobNumber];

                // Perform bookkeeping.
                job.Completed = true;

                // Send the user a proactive confirmation message.
                await context.SendActivity($"Job {job.JobNumber} is complete.");
            });
        }
    }
}
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

在您可以將主動式訊息傳送給使用者之前，該使用者必須先至少傳送一則被動式訊息給您的 Bot。 

之所以需要先傳送一則訊息給 Bot 的原因，是因為它需要取得針對活動物件的參考，並將它儲存於某處以供未來使用。 您可以將活動物件想成使用者的地址，因為它會包含該使用者連入時所使用的通道、其使用者識別碼、對話識別碼，甚至是應接收所有未來訊息之伺服器的相關資訊。 此物件為簡單 JSON，並應該在不被竄改的情況下完整儲存。

作為開始，以下是一個示範如何在使用者說出 "subscribe" (訂閱) 時便會儲存對話參考的簡短程式碼片段：
```javascript
const { MemoryStorage } = require('botbuilder');

const storage = new MemoryStorage();

// Listen for incoming activity 
server.post('/api/messages', (req, res) => {
    // Route received activity to adapter for processing
    adapter.processActivity(req, res, async (context) => {
        if (context.activity.type === 'message') {
            const utterances = (context.activity.text || '').trim().toLowerCase()
            if (utterances === 'subscribe') {
                var userId = await saveReference(TurnContext.getConversationReference(context.activity));
                await subscribeUser(userId)
                await context.sendActivity(`Thank You! We will message you shortly.`);
               
            } else{
                await context.sendActivity("Say 'subscribe' to start proactive message");
            }
    
        }
    });
});
```
上述程式碼片段會呼叫 `saveReference()` 函式，該函式會搭配 `MemoryStorage` 儲存使用者的參考並傳回 `userId`。 在成功儲存參考之後，我們會接著呼叫 `subscribeUser()`，這將會通知使用者其已成功訂閱。 

`subscribeUser()` 函式是設定實際訂閱的函式。 讓我們來看看一個會啟動 2 秒計時器，並會在計時器結束時主動傳訊給使用者的簡單實作：

```javascript
// Persist info to storage
async function saveReference(reference){
    const userId = reference.activityId
    const changes = {};
    changes['reference/' + userId] = reference;
    await storage.write(changes); // Write reference info to persisted storage
    return userId;
}

// Subscribe user to a proactive call. In this case, we are using a setTimeOut() to trigger the proactive call
async function subscribeUser(userId) {
    setTimeout(async () => {
        const reference = await findReference(userId);
        if (reference) {
            await adapter.continueConversation(reference, async (context) => {
                await context.sendActivity("You have been notified");
            });
            
        }
    }, 2000); // Trigger after 2 secs
}

// Read the stored reference info from storage
async function findReference(userId){
    const referenceKey = 'reference/' + userId;
    var rows = await storage.read([referenceKey])
    var reference = await rows[referenceKey]

    return reference;
}
```

`subscribeUser()` 函式會設定計時器，該計時器會透過從儲存體讀取參考物件來找到它。 若有找到參考物件，我們便能繼續與該使用者對話。 `continueConversation` 方法會讓 Bot 主動傳送訊息至其曾經進行過通訊的對話或使用者。

---

## <a name="test-your-bot"></a>測試 Bot

若要測試您的 Bot，請將它以僅註冊之 Bot 的形式部署至 Azure，並在網路聊天中或是使用模擬器於本機測試它。
