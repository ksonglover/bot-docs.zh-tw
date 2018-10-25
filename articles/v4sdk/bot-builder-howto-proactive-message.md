---
title: 如何使用主動式傳訊 | Microsoft Docs
description: 了解如何使用 Bot 傳送主動傳訊。
keywords: 主動式訊息
author: jonathanfingold
ms.author: jonathanfingold
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 09/27/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: ffc733eb42842becf955b912fc6c795d7f3a50d2
ms.sourcegitcommit: bd4f9669c0d26ac2a4be1ab8e508f163a1f465f3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/28/2018
ms.locfileid: "47430367"
---
# <a name="how-to-use-proactive-messaging"></a>如何使用主動式傳訊

[!INCLUDE [pre-release-label](~/includes/pre-release-label.md)]

一般而言，Bot 直接傳送給使用者的每個訊息都與使用者先前的輸入相關。
在某些情況下，Bot 可能需要將與目前對話主題或使用者最後傳送的訊息不直接相關的訊息傳送給使用者。 這些類型的訊息稱為_主動訊息_。

## <a name="uses"></a>使用

主動訊息可用於各種情況。 如果 Bot 設定了計時器或提醒，它必須在到達該時間時通知使用者。 或者，如果 Bot 從外部系統收到通知，它可能需要將該資訊立即傳達給使用者。 比方說，如果使用者先前已要求 Bot 監視產品的價格，則 Bot 可以在產品的價格下降了 20% 時對使用者發出警示。 或者，如果 Bot 需要一些時間來編譯對使用者問題的回應，它可能會通知使用者已延遲，並且在此同時允許交談繼續進行。 當 Bot 完成問題回應的編譯時，它會與使用者分享該資訊。

在 Bot 中實作主動訊息時：

- 請勿在短時間內傳送數個主動訊息。 某些通道會強制限制 Bot 將訊息傳送給使用者的頻率，如果違反了這些限制，將會停用 Bot。
- 請勿將主動訊息傳送給先前未與 Bot 互動或透過其他方式　(例如電子郵件或簡訊) 與 Bot 連繫的使用者。

**臨機操作主動式訊息**是最簡單的主動式訊息類型。
Bot 只會在每次觸發時將訊息插入對話，不會顧及使用者目前是否與 Bot 在其他對話主題中，也不會嘗試以任何方式變更對話。

若要更順利地處理通知，請考慮使用其他方式將通知整合到對話流程中，例如在對話狀態中設定旗標，或將通知新增至佇列。

## <a name="prerequisites"></a>必要條件

若要傳送主動式訊息，Bot 必須擁有有效的應用程式識別碼和密碼。 不過，如需在模擬器中進行本機測試，您可以使用預留位置應用程式識別碼。

若要取得要用於 Bot 的應用程式識別碼和密碼，您可以登入 [Azure 入口網站](https://portal.azure.com)並建立 **Bot 通道註冊**資源。 基於測試目的，您可以接著在本機對 Bot 使用此應用程式識別碼和密碼，而不需部署至 Azure。

> [!TIP]
> 若您還沒有訂用帳戶，則可以註冊<a href="https://azure.microsoft.com/en-us/free/" target="_blank">免費帳戶</a>。

### <a name="required-libraries"></a>必要的程式庫

如果您從其中一個 BotBuilder 範本著手，則會為您安裝必要的程式庫。 這些是主動式傳訊所需的特定 BotBuilder 程式庫：

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

**Microsoft.Bot.Builder.Integration.AspNet.Core** NuGet 套件。 (安裝此套件也會安裝 **Microsoft.Bot.Builder** 套件。)

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**Microsoft.Bot.Builder** npm 套件。

---

## <a name="notes-on-the-sample-code"></a>範例程式碼的注意事項

本文的程式碼取自主動式訊息範例 [[C#](https://aka.ms/proactive-sample-cs) | [JS](https://aka.ms/proactive-sample-js)]。

此範例展示可能需要不定量時間的使用者工作。 Bot 會儲存有關該工作的資訊，告訴使用者將會在工作完成時通知他們，並且讓對話繼續。 當工作完成時，Bot 會在原始對話上主動傳送確認訊息。

## <a name="define-job-data-and-state"></a>定義作業資料和狀態

在此案例中，我們會追蹤可由各種使用者在不同的對話中建立的任意作業。 我們需要儲存每項工作的相關資訊，包括對話參考和作業識別碼。

- 我們需要對話參考，才能將主動式訊息傳送至適當的對話。
- 我們需要可識別作業的方法。 在此範例中，我們使用簡單的時間戳記。
- 我們需要將作業狀態與對話或使用者狀態分開儲存。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

我們需要定義作業資料和工作狀態的類別。 我們也需要註冊我們的 Bot，並設定作業記錄的狀態屬性存取子。

### <a name="define-a-class-for-job-data"></a>定義作業資料的類別

**JobLog** 類別會追蹤作業資料 (依作業編號 (時間戳記) 編製索引)。 作業資料會定義為字典的內部類別。

```csharp
/// <summary>Contains a dictionary of job data, indexed by job number.</summary>
/// <remarks>The JobLog class tracks all the outstanding jobs.  Each job is
/// identified by a unique key.</remarks>
public class JobLog : Dictionary<long, JobLog.JobData>
{
    /// <summary>Describes the state of a job.</summary>
    public class JobData
    {
        /// <summary>Gets or sets the time-stamp for the job.</summary>
        /// <value>
        /// The time-stamp for the job when the job needs to fire.
        /// </value>
        public long TimeStamp { get; set; } = 0;

        /// <summary>Gets or sets a value indicating whether indicates whether the job has completed.</summary>
        /// <value>
        /// A value indicating whether indicates whether the job has completed.
        /// </value>
        public bool Completed { get; set; } = false;

        /// <summary>
        /// Gets or sets the conversation reference to which to send status updates.
        /// </summary>
        /// <value>
        /// The conversation reference to which to send status updates.
        /// </value>
        public ConversationReference Conversation { get; set; }
    }
}
```

### <a name="define-a-state-middleware-class"></a>定義狀態中介軟體類別

**JobState** 可將作業狀態與對話或使用者狀態分開管理。

```csharp
using Microsoft.Bot.Builder;

/// <summary>A <see cref="BotState"/> for managing bot state for "bot jobs".</summary>
/// <remarks>Independent from both <see cref="UserState"/> and <see cref="ConversationState"/> because
/// the process of running the jobs and notifying the user interacts with the
/// bot as a distinct user on a separate conversation.</remarks>
public class JobState : BotState
{
    /// <summary>The key used to cache the state information in the turn context.</summary>
    private const string StorageKey = "ProactiveBot.JobState";

    /// <summary>
    /// Initializes a new instance of the <see cref="JobState"/> class.</summary>
    /// <param name="storage">The storage provider to use.</param>
    public JobState(IStorage storage)
        : base(storage, StorageKey)
    {
    }

    /// <summary>Gets the storage key for caching state information.</summary>
    /// <param name="turnContext">A <see cref="ITurnContext"/> containing all the data needed
    /// for processing this conversation turn.</param>
    /// <returns>The storage key.</returns>
    protected override string GetStorageKey(ITurnContext turnContext) => StorageKey;
}
```

### <a name="register-the-bot-and-required-services"></a>註冊 Bot 和必要的服務

**Startup.cs** 檔案可註冊 Bot 和相關聯的服務。

1. 這組 using 陳述式會擴展為參考這些命名空間：

    ```csharp
    using System;
    using System.Linq;
    using Microsoft.AspNetCore.Builder;
    using Microsoft.AspNetCore.Hosting;
    using Microsoft.Bot.Builder;
    using Microsoft.Bot.Builder.Integration;
    using Microsoft.Bot.Builder.Integration.AspNet.Core;
    using Microsoft.Bot.Configuration;
    using Microsoft.Bot.Connector.Authentication;
    using Microsoft.Extensions.Configuration;
    using Microsoft.Extensions.DependencyInjection;
    using Microsoft.Extensions.Logging;
    using Microsoft.Extensions.Options;
    ```

1. `ConfigureServices` 方法可註冊 Bot，包括錯誤處理和狀態管理。 也可註冊 Bot 的端點服務和作業狀態存取子。

    ```csharp
    public void ConfigureServices(IServiceCollection services)
    {
        // The Memory Storage used here is for local bot debugging only. When the bot
        // is restarted, everything stored in memory will be gone.
        IStorage dataStore = new MemoryStorage();

        // ...

        // Create Job State object.
        // The Job State object is where we persist anything at the job-scope.
        // Note: It's independent of any user or conversation.
        var jobState = new JobState(dataStore);

        // Make it available to our bot
        services.AddSingleton(sp => jobState);

        // Register the proactive bot.
        services.AddBot<ProactiveBot>(options =>
        {
            var secretKey = Configuration.GetSection("botFileSecret")?.Value;
            var botFilePath = Configuration.GetSection("botFilePath")?.Value;

            // Loads .bot configuration file and adds a singleton that your Bot can access through dependency injection.
            var botConfig = BotConfiguration.Load(botFilePath ?? @".\BotConfiguration.bot", secretKey);
            services.AddSingleton(sp => botConfig ?? throw new InvalidOperationException($"The .bot config file could not be loaded. ({botConfig})"));

            // Retrieve current endpoint.
            var environment = _isProduction ? "production" : "development";
            var service = botConfig.Services.Where(s => s.Type == "endpoint" && s.Name == environment).FirstOrDefault();
            if (!(service is EndpointService endpointService))
            {
                throw new InvalidOperationException($"The .bot file does not contain an endpoint with name '{environment}'.");
            }

            options.CredentialProvider = new SimpleCredentialProvider(endpointService.AppId, endpointService.AppPassword);

            // Creates a logger for the application to use.
            ILogger logger = _loggerFactory.CreateLogger<ProactiveBot>();

            // Catches any errors that occur during a conversation turn and logs them.
            options.OnTurnError = async (context, exception) =>
            {
                logger.LogError($"Exception caught : {exception}");
                await context.SendActivityAsync("Sorry, it looks like something went wrong.");
            };

        });

        services.AddSingleton(sp =>
        {
            var config = BotConfiguration.Load(@".\BotConfiguration.bot");
            var endpointService = (EndpointService)config.Services.First(s => s.Type == "endpoint")
                                    ?? throw new InvalidOperationException(".bot file 'endpoint' must be configured prior to running.");

            return endpointService;
        });
    }
    ```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

### <a name="set-up-the-server-code"></a>設定伺服器程式碼

**Index.js** 檔案可進行下列作業：

- 包含必要的套件和服務
- 參考 Bot 類別和 **.bot** 檔案
- 建立 HTTP 伺服器
- 建立 Bot 配接器和儲存體物件
- 建立 Bot 並啟動伺服器，以將活動傳遞到 Bot

```javascript
// Copyright (c) Microsoft Corporation. All rights reserved.
// Licensed under the MIT License.

const restify = require('restify');
const path = require('path');

// Import required bot services. See https://aka.ms/bot-services to learn more about the different part of a bot.
const { BotFrameworkAdapter, BotState, MemoryStorage } = require('botbuilder');
const { BotConfiguration } = require('botframework-config');

const { ProactiveBot } = require('./bot');

// Read botFilePath and botFileSecret from .env file.
// Note: Ensure you have a .env file and include botFilePath and botFileSecret.
const ENV_FILE = path.join(__dirname, '.env');
require('dotenv').config({ path: ENV_FILE });

// Create HTTP server.
let server = restify.createServer();
server.listen(process.env.port || process.env.PORT || 3978, function() {
    console.log(`\n${ server.name } listening to ${ server.url }.`);
    console.log(`\nGet Bot Framework Emulator: https://aka.ms/botframework-emulator.`);
    console.log(`\nTo talk to your bot, open proactive-messages.bot file in the Emulator.`);
});

// .bot file path
const BOT_FILE = path.join(__dirname, (process.env.botFilePath || ''));

// Read the bot's configuration from a .bot file identified by BOT_FILE.
// This includes information about the bot's endpoints and configuration.
let botConfig;
try {
    botConfig = BotConfiguration.loadSync(BOT_FILE, process.env.botFileSecret);
} catch (err) {
    console.error(`\nError reading bot file. Please ensure you have valid botFilePath and botFileSecret set for your environment.`);
    console.error(`\n - The botFileSecret is available under appsettings for your Azure Bot Service bot.`);
    console.error(`\n - If you are running this bot locally, consider adding a .env file with botFilePath and botFileSecret.\n\n`);
    process.exit();
}

const DEV_ENVIRONMENT = 'development';

// Define the name of the bot, as specified in .bot file.
// See https://aka.ms/about-bot-file to learn more about .bot files.
const BOT_CONFIGURATION = (process.env.NODE_ENV || DEV_ENVIRONMENT);

// Load the configuration profile specific to this bot identity.
const endpointConfig = botConfig.findServiceByNameOrId(BOT_CONFIGURATION);

// Create the adapter. See https://aka.ms/about-bot-adapter to learn more about using information from
// the .bot file when configuring your adapter.
const adapter = new BotFrameworkAdapter({
    appId: endpointConfig.appId || process.env.MicrosoftAppId,
    appPassword: endpointConfig.appPassword || process.env.MicrosoftAppPassword
});

// Define the state store for your bot. See https://aka.ms/about-bot-state to learn more about using MemoryStorage.
// A bot requires a state storage system to persist the dialog and user state between messages.
const memoryStorage = new MemoryStorage();

// Create state manager with in-memory storage provider.
const botState = new BotState(memoryStorage, () => 'proactiveBot.botState');

// Create the main dialog, which serves as the bot's main handler.
const bot = new ProactiveBot(botState, adapter);

// Listen for incoming requests.
server.post('/api/messages', (req, res) => {
    adapter.processActivity(req, res, async (turnContext) => {
        // Route the message to the bot's main handler.
        await bot.onTurn(turnContext);
    });
});

// Catch-all for errors.
adapter.onTurnError = async (context, error) => {
    // This check writes out errors to console log .vs. app insights.
    console.error(`\n [onTurnError]: ${ error }`);
    // Send a message to the user
    context.sendActivity(`Oops. Something went wrong!`);
};
```

---

<!--TODO: (Post-Ignite) -- link to a second topic on how to write a job completion DirectLine client that will generate appropriate job completed event activities.-->

## <a name="define-the-bot"></a>定義 Bot

使用者可以要求 Bot 為他們建立及執行作業。 當作業完成後，個別的作業服務可以通知 Bot。

Bot 的設計訴求：

- 建立作業，以回應來自使用者的 `run` 或 `run job` 訊息。
- 顯示所有已註冊的作業，以回應來自使用者的 `show` 或 `show jobs` 訊息。
- 完成作業，以回應可識別已完成作業的「作業已完成」事件。
- 模擬「作業已完成」事件，以回應 `done <jobIdentifier>` 訊息。
- 當作業完成時，使用原始對話，將主動式訊息傳送給使用者。

我們不會示範如何實作可將事件活動傳送至 Bot 的系統。
<!--TODO: DirectLine--Add back in once the DirectLine topic is added back to the TOC.
See [how to create a Direct Line bot and client](bot-builder-howto-direct-line.md) for information on how to do so.
-->

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Bot 有幾個層面：

- 初始化程式碼
- 回合處理常式
- 用於建立和完成工作的方法

### <a name="declare-the-class"></a>宣告類別

```csharp
using System;
using System.Linq;
using System.Threading;
using System.Threading.Tasks;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Configuration;
using Microsoft.Bot.Schema;

namespace Microsoft.BotBuilderSamples
{
    /// <summary>
    /// For each interaction from the user, an instance of this class is called.
    /// This is a Transient lifetime service.  Transient lifetime services are created
    /// each time they're requested. For each Activity received, a new instance of this
    /// class is created. Objects that are expensive to construct, or have a lifetime
    /// beyond the single Turn, should be carefully managed.
    /// </summary>
    public class ProactiveBot : IBot
    {
        /// <summary>The name of events that signal that a job has completed.</summary>
        public const string JobCompleteEventName = "jobComplete";

        public const string WelcomeText = "Type 'run' or 'run job' to start a new job.\r\n" +
                                          "Type 'show' or 'show jobs' to display the job log.\r\n" +
                                          "Type 'done <jobNumber>' to complete a job.";
    }
}
```

### <a name="add-initialization-code"></a>新增初始化程式碼

```csharp
private readonly JobState _jobState;
private readonly IStatePropertyAccessor<JobLog> _jobLogPropertyAccessor;

public ProactiveBot(JobState jobState, EndpointService endpointService)
{
    _jobState = jobState ?? throw new ArgumentNullException(nameof(jobState));
    _jobLogPropertyAccessor = _jobState.CreateProperty<JobLog>(nameof(JobLog));

    // Validate AppId.
    // Note: For local testing, .bot AppId is empty for the Bot Framework Emulator.
    AppId = string.IsNullOrWhiteSpace(endpointService.AppId) ? "1" : endpointService.AppId;
}

private string AppId { get; }
```

### <a name="add-a-turn-handler"></a>新增回合處理常式

每個 Bot 都必須實作回合處理常式。 配接器會將活動轉送至這個方法。

```csharp
public async Task OnTurnAsync(ITurnContext turnContext, CancellationToken cancellationToken = default(CancellationToken))
{
    // See https://aka.ms/about-bot-activity-message to learn more about the message and other activity types.
    if (turnContext.Activity.Type != ActivityTypes.Message)
    {
        // Handle non-message activities.
        await OnSystemActivityAsync(turnContext);
    }
    else
    {
        // Get the job log.
        // The job log is a dictionary of all outstanding jobs in the system.
        JobLog jobLog = await _jobLogPropertyAccessor.GetAsync(turnContext, () => new JobLog());

        // Get the user's text input for the message.
        var text = turnContext.Activity.Text.Trim().ToLowerInvariant();
        switch (text)
        {
            case "run":
            case "run job":

                // Start a virtual job for the user.
                JobLog.JobData job = CreateJob(turnContext, jobLog);

                // Set the new property
                await _jobLogPropertyAccessor.SetAsync(turnContext, jobLog);

                // Now save it into the JobState
                await _jobState.SaveChangesAsync(turnContext);

                await turnContext.SendActivityAsync(
                    $"We're starting job {job.TimeStamp} for you. We'll notify you when it's complete.");

                break;

            case "show":
            case "show jobs":

                // Display information for all jobs in the log.
                if (jobLog.Count > 0)
                {
                    await turnContext.SendActivityAsync(
                        "| Job number &nbsp; | Conversation ID &nbsp; | Completed |<br>" +
                        "| :--- | :---: | :---: |<br>" +
                        string.Join("<br>", jobLog.Values.Select(j =>
                            $"| {j.TimeStamp} &nbsp; | {j.Conversation.Conversation.Id.Split('|')[0]} &nbsp; | {j.Completed} |")));
                }
                else
                {
                    await turnContext.SendActivityAsync("The job log is empty.");
                }

                break;

            default:
                // Check whether this is simulating a job completed event.
                string[] parts = text?.Split(' ', StringSplitOptions.RemoveEmptyEntries);
                if (parts != null && parts.Length == 2
                    && parts[0].Equals("done", StringComparison.InvariantCultureIgnoreCase)
                    && long.TryParse(parts[1], out long jobNumber))
                {
                    if (!jobLog.TryGetValue(jobNumber, out JobLog.JobData jobInfo))
                    {
                        await turnContext.SendActivityAsync($"The log does not contain a job {jobInfo.TimeStamp}.");
                    }
                    else if (jobInfo.Completed)
                    {
                        await turnContext.SendActivityAsync($"Job {jobInfo.TimeStamp} is already complete.");
                    }
                    else
                    {
                        await turnContext.SendActivityAsync($"Completing job {jobInfo.TimeStamp}.");

                        // Send the proactive message.
                        await CompleteJobAsync(turnContext.Adapter, AppId, jobInfo);
                    }
                }

                break;
        }

        if (!turnContext.Responded)
        {
            await turnContext.SendActivityAsync(WelcomeText);
        }
    }
}

private static async Task SendWelcomeMessageAsync(ITurnContext turnContext)
{
    foreach (var member in turnContext.Activity.MembersAdded)
    {
        if (member.Id != turnContext.Activity.Recipient.Id)
        {
            await turnContext.SendActivityAsync($"Welcome to SuggestedActionsBot {member.Name}.\r\n{WelcomeText}");
        }
    }
}

// Handles non-message activities.
private async Task OnSystemActivityAsync(ITurnContext turnContext)
{
    // On a job completed event, mark the job as complete and notify the user.
    if (turnContext.Activity.Type is ActivityTypes.Event)
    {
        var jobLog = await _jobLogPropertyAccessor.GetAsync(turnContext, () => new JobLog());
        var activity = turnContext.Activity.AsEventActivity();
        if (activity.Name == JobCompleteEventName
            && activity.Value is long timestamp
            && jobLog.ContainsKey(timestamp)
            && !jobLog[timestamp].Completed)
        {
            await CompleteJobAsync(turnContext.Adapter, AppId, jobLog[timestamp]);
        }
    }
    else if (turnContext.Activity.Type is ActivityTypes.ConversationUpdate)
    {
        if (turnContext.Activity.MembersAdded.Any())
        {
            await SendWelcomeMessageAsync(turnContext);
        }
    }
}
```

### <a name="add-job-creation-and-completion-methods"></a>新增作業建立和完成方法

若要開始作業，Bot 會建立作業，以及在作業記錄中記錄其相關資訊和目前的對話。 當 Bot 在任何對話中收到「作業已完成」事件時，會先驗證作業識別碼，再呼叫程式碼來完成作業。

用來完成作業的程式碼會從作業記錄中取得狀態，然後將作業標示為已完成，並使用配接器的「繼續對話」方法來傳送主動式訊息。

- 繼續對話呼叫會提示通道起始與使用者無關的回合。
- 配接器會執行相關聯的回呼，代替 Bot 的正常回合處理常式。 此回合有屬於自己的回合內容，我們可從中擷取狀態資訊並傳送主動式訊息給使用者。

```csharp
// Creates and "starts" a new job.
private JobLog.JobData CreateJob(ITurnContext turnContext, JobLog jobLog)
{
    JobLog.JobData jobInfo = new JobLog.JobData
    {
        TimeStamp = DateTime.Now.ToBinary(),
        Conversation = turnContext.Activity.GetConversationReference(),
    };

    jobLog[jobInfo.TimeStamp] = jobInfo;

    return jobInfo;
}

// Sends a proactive message to the user.
private async Task CompleteJobAsync(
    BotAdapter adapter,
    string botId,
    JobLog.JobData jobInfo,
    CancellationToken cancellationToken = default(CancellationToken))
{
    await adapter.ContinueConversationAsync(botId, jobInfo.Conversation, CreateCallback(jobInfo), cancellationToken);
}

// Creates the turn logic to use for the proactive message.
private BotCallbackHandler CreateCallback(JobLog.JobData jobInfo)
{
    return async (turnContext, token) =>
    {
        // Get the job log from state, and retrieve the job.
        JobLog jobLog = await _jobLogPropertyAccessor.GetAsync(turnContext, () => new JobLog());

        // Perform bookkeeping.
        jobLog[jobInfo.TimeStamp].Completed = true;

        // Set the new property
        await _jobLogPropertyAccessor.SetAsync(turnContext, jobLog);

        // Now save it into the JobState
        await _jobState.SaveChangesAsync(turnContext);

        // Send the user a proactive confirmation message.
        await turnContext.SendActivityAsync($"Job {jobInfo.TimeStamp} is complete.");
    };
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Bot 定義於 **bot.js**，而且有幾個層面：

- 初始化程式碼
- 回合處理常式
- 用於建立和完成工作的方法

### <a name="declare-the-class-and-add-initialization-code"></a>類別宣告及新增初始化程式碼

```javascript
const { ActivityTypes, TurnContext } = require('botbuilder');

const JOBS_LIST = 'jobs';

class ProactiveBot {
    constructor(botState, adapter) {
        this.botState = botState;
        this.adapter = adapter;

        this.jobsList = this.botState.createProperty(JOBS_LIST);
    }

    // ...
};

// Helper function to check if object is empty.
function isEmpty(obj) {
    for (var key in obj) {
        if (obj.hasOwnProperty(key)) {
            return false;
        }
    }
    return true;
};

module.exports.ProactiveBot = ProactiveBot;
```

### <a name="the-turn-handler"></a>回合處理常式

`onTurn` 和 `showJobs` 方法定義於 `ProactiveBot` 類別中。 `onTurn` 會處理來自使用者的輸入。 其也會接收來自假設作業履行系統的事件活動。 `showJobs` 會將作業記錄格式化並且傳送。

```javascript
/**
    *
    * @param {TurnContext} turnContext A TurnContext object representing an incoming message to be handled by the bot.
    */
async onTurn(turnContext) {
    // See https://aka.ms/about-bot-activity-message to learn more about the message and other activity types.
    if (turnContext.activity.type === ActivityTypes.Message) {
        const utterance = (turnContext.activity.text || '').trim().toLowerCase();
        var jobIdNumber;

        // If user types in run, create a new job.
        if (utterance === 'run') {
            await this.createJob(turnContext);
        } else if (utterance === 'show') {
            await this.showJobs(turnContext);
        } else {
            const words = utterance.split(' ');

            // If the user types done and a Job Id Number,
            // we check if the second word input is a number.
            if (words[0] === 'done' && !isNaN(parseInt(words[1]))) {
                jobIdNumber = words[1];
                await this.completeJob(turnContext, jobIdNumber);
            } else if (words[0] === 'done' && (words.length < 2 || isNaN(parseInt(words[1])))) {
                await turnContext.sendActivity('Enter the job ID number after "done".');
            }
        }

        if (!turnContext.responded) {
            await turnContext.sendActivity(`Say "run" to start a job, or "done <job>" to complete one.`);
        }
    } else if (turnContext.activity.type === ActivityTypes.Event && turnContext.activity.name === 'jobCompleted') {
        jobIdNumber = turnContext.activity.value;
        if (!isNaN(parseInt(jobIdNumber))) {
            await this.completeJob(turnContext, jobIdNumber);
        }
    }

    await this.botState.saveChanges(turnContext);
}

// Show a list of the pending jobs
async showJobs(turnContext) {
    const jobs = await this.jobsList.get(turnContext, {});
    if (Object.keys(jobs).length) {
        await turnContext.sendActivity(
            '| Job number &nbsp; | Conversation ID &nbsp; | Completed |<br>' +
            '| :--- | :---: | :---: |<br>' +
            Object.keys(jobs).map((key) => {
                return `${ key } &nbsp; | ${ jobs[key].reference.conversation.id.split('|')[0] } &nbsp; | ${ jobs[key].completed }`;
            }).join('<br>'));
    } else {
        await turnContext.sendActivity('The job log is empty.');
    }
}
```

### <a name="logic-to-start-a-job"></a>開始作業的邏輯

`createJob` 方法定義於 `ProactiveBot` 類別中。 其會為使用者建立並記錄新作業。 理論上，其也會將此資訊轉送到作業履行系統。

```javascript
// Save job ID and conversation reference.
async createJob(turnContext) {
    // Create a unique job ID.
    var date = new Date();
    var jobIdNumber = date.getTime();

    // Get the conversation reference.
    const reference = TurnContext.getConversationReference(turnContext.activity);

    // Get the list of jobs. Default it to {} if it is empty.
    const jobs = await this.jobsList.get(turnContext, {});

    // Try to find previous information about the saved job.
    const jobInfo = jobs[jobIdNumber];

    try {
        if (isEmpty(jobInfo)) {
            // Job object is empty so we have to create it
            await turnContext.sendActivity(`Need to create new job ID: ${ jobIdNumber }`);

            // Update jobInfo with new info
            jobs[jobIdNumber] = { completed: false, reference: reference };

            try {
                // Save to storage
                await this.jobsList.set(turnContext, jobs);
                // Notify the user that the job has been processed
                await turnContext.sendActivity('Successful write to log.');
            } catch (err) {
                await turnContext.sendActivity(`Write failed: ${ err.message }`);
            }
        }
    } catch (err) {
        await turnContext.sendActivity(`Read rejected. ${ err.message }`);
    }
}
```

### <a name="logic-to-complete-a-job"></a>完成作業的邏輯

`completeJob` 方法定義於 `ProactiveBot` 類別中。 其會執行一些簿記工作，並將主動式訊息傳送給作業已完成的使用者 (在使用者的原始對話中)。

```javascript
async completeJob(turnContext, jobIdNumber) {
    // Get the list of jobs from the bot's state property accessor.
    const jobs = await this.jobsList.get(turnContext, {});

    // Find the appropriate job in the list of jobs.
    let jobInfo = jobs[jobIdNumber];

    // If no job was found, notify the user of this error state.
    if (isEmpty(jobInfo)) {
        await turnContext.sendActivity(`Sorry no job with ID ${ jobIdNumber }.`);
    } else {
        // Found a job with the ID passed in.
        const reference = jobInfo.reference;
        const completed = jobInfo.completed;

        // If the job is not yet completed and conversation reference exists,
        // use the adapter to continue the conversation with the job's original creator.
        if (reference && !completed) {
            // Since we are going to proactively send a message to the user who started the job,
            // we need to create the turnContext based on the stored reference value.
            await this.adapter.continueConversation(reference, async (proactiveTurnContext) => {
                // Complete the job.
                jobInfo.completed = true;
                // Save the updated job.
                await this.jobsList.set(turnContext, jobs);
                // Notify the user that the job is complete.
                await proactiveTurnContext.sendActivity(`Your queued job ${ jobIdNumber } just completed.`);
            });

            // Send a message to the person who completed the job.
            await turnContext.sendActivity('Job completed. Notification sent.');
        } else if (completed) { // The job has already been completed.
            await turnContext.sendActivity('This job is already completed, please start a new job.');
        };
    };
};
```

---

## <a name="test-your-bot"></a>測試 Bot

在本機建置及執行 Bot，然後開啟兩個模擬器視窗。

1. 請注意，兩個視窗內的對話識別碼不同。
1. 在第一個視窗中，輸入 `run` 數次以開始一些作業。
1. 在第二個視窗中，輸入 `show` 以查看記錄中的作業清單。
1. 在第二個視窗中，輸入 `done <jobNumber>`，其中 `<jobNumber>` 是記錄中的其中一個作業編號 (不含角括弧)。 (Bot 程式碼的設計訴求是要將此解譯為 jobComplete 事件。)
1. 請注意，Bot 會將主動式訊息傳送給第一個視窗中的使用者。

<!--TODO: Recreate the screen shots once we're happy with both the C# and JS versions of the code.-->

從使用者的觀點來看，您的對話可能看起來像這樣：

![使用者的模擬器工作階段](~/v4sdk/media/how-to-proactive/user.png)

而從模擬作業系統的觀點來看，則看起來像這樣：

![作業系統的模擬器工作階段](~/v4sdk/media/how-to-proactive/job-system.png)

<!-- Add a next steps section. -->
