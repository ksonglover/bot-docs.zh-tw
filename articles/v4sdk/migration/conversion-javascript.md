---
title: 將現有 v3 JavaScript Bot 遷移至新的 v4 專案 | Microsoft Docs
description: 我們採用現有 v3 JavaScript Bot 並將其遷移至 v4 SDK，而且使用新的專案。
keywords: JavaScript, bot 移轉, 對話方塊, v3 bot
author: JonathanFingold
ms.author: v-jofing
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 05/23/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 591f58e1cefca576e2e3e4a486ecc6fbe0a6b0e4
ms.sourcegitcommit: ea64a56acfabc6a9c1576ebf9f17ac81e7e2a6b7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/24/2019
ms.locfileid: "66215598"
---
# <a name="migrate-a-sdk-v3-javascript-bot-to-v4"></a>將 SDK v3 Javascript Bot 遷移至 v4

在本文中，我們會將 v3 SDK JavaScript [core-MultiDialogs-v3](https://aka.ms/v3-js-core-multidialog-migration-sample) Bot 移植到新的 v4 JavaScript Bot。
此轉換可細分成下列階段：

1. 建立新專案及新增相依性。
1. 更新進入點及定義常數。
1. 建立對話，並使用 SDK v4 加以重新實作。
1. 更新 Bot 程式碼以執行對話。
1. 移植 **store.js** 公用程式檔案。

在此程序結束時，我們會有運作中的 v4 Bot。 範例存放庫 [core-MultiDialogs-v4](https://aka.ms/v4-js-core-multidialog-migration-sample) 中也有一份已轉換的 Bot。

Bot Framework SDK v4 是以與 SDK v3 相同的基礎 REST API 作為基礎。 不過，SDK v4 是舊版 SDK 的重構，讓開發人員對於其 Bot 有更多的彈性和控制權。 SDK 中的主要變更包括：

- 透過狀態管理物件和屬性存取子管理狀態。
- 我們處理回合的方式已改變，也就是，Bot 接收及回應來自使用者通道的傳入活動的方式。
- v4 不會使用 `session` 物件，而是具有「回合內容」  物件，其中包含傳入活動的相關資訊，可用來送回回應活動。
- 與 v3 非常不同的新 Dialogs 程式庫。 您必須將舊的對話轉換至新的對話系統，並使用元件和瀑布式對話。

<!-- TODO
For more information about specific changes, see [differences between the v3 and v4 JavaScript SDK](???.md).
-->

> [!NOTE]
> 在移轉過程中，我們也清除了部分的程式碼，但我們只會強調我們在移轉過程中對 v3 邏輯所做的變更。

## <a name="prerequisites"></a>必要條件

- Node.js
- Visual Studio Code
- [Bot Framework 模擬器](https://aka.ms/bot-framework-emulator-readme) (英文)

## <a name="about-this-bot"></a>關於此 Bot

我們要移轉的 Bot 會示範如何使用多個對話來管理交談流程。 Bot 可以查閱航班或旅館資訊。

- 主要對話會詢問使用者他們要尋找何種資訊。
- 旅館對話會提示使用者輸入搜尋參數，然後執行模擬搜尋。
- 航班對話會產生 Bot 攔截的錯誤，並依正常程序處理。

## <a name="create-and-open-a-new-v4-bot-project"></a>建立並開啟新的 v4 Bot 專案

1. 您需要 Bot 程式碼要移植到其中的 v4 專案。 若要在本機建立專案，請參閱[使用適用於 JavaScript 的 Bot Framework SDK 建立 Bot](../../javascript/bot-builder-javascript-quickstart.md)。

    > [!TIP]
    > 您也可以在 Azure 上建立專案，請參閱[使用 Azure Bot 服務建立 Bot](../../bot-service-quickstart.md)。
    > 不過，這兩種方法會導致支援檔案稍有差異。 本文的 v4 專案已建立為本機專案。

1. 然後在 Visual Studio Code 中開啟專案。

## <a name="update-the-packagejson-file"></a>更新 package.json 檔案

1. 在 Visual Studio Code 的終端機視窗中輸入 `npm i botbuilder-dialogs`，以新增 **botbuilder-dialogs** 套件的相依性。

1. 編輯 **./package.json**，並更新 `name``version`、`description` 和其他所需的屬性。

## <a name="update-the-v4-app-entry-point"></a>更新 v4 應用程式進入點

V4 範本會針對應用程式進入點建立 **index.js** 檔案，以及針對 Bot 特有邏輯建立 **bot.js** 檔案。 在稍後步驟中，我們會將 **bot.js** 檔案重新命名為 **bots/reservationBot.js**，並且為每個對話新增類別。

編輯 **./index.js**，這是 Bot 應用程式的進入點。 這會包含 v3 **app.js** 檔案中設定 HTTP 伺服器的部份。

1. 除了 `BotFrameworkAdapter`，從 **botbuilder** 套件匯入 `MemoryStorage` 和 `ConversationState`。 還要匯入 Bot 和主要對話模組。 (我們很快就會建立這些模組，但我們需要在此參考它們。)

    ```javascript
    // Import required bot services.
    // See https://aka.ms/bot-services to learn more about the different parts of a bot.
    const { BotFrameworkAdapter, MemoryStorage, ConversationState } = require('botbuilder');

    // This bot's main dialog.
    const { MainDialog } = require('./dialogs/main')
    const { ReservationBot } = require('./bots/reservationBot');
    ```

1. 為配接器定義 `onTurnError` 處理常式。

    ```javascript
    // Catch-all for errors.
    adapter.onTurnError = async (context, error) => {
        const errorMsg = error.message ? error.message : `Oops. Something went wrong!`;
        // This check writes out errors to console log .vs. app insights.
        console.error(`\n [onTurnError]: ${ error }`);
        // Clear out state
        await conversationState.delete(context);
        // Send a message to the user
        await context.sendActivity(errorMsg);
    };
    ```

    在 v4 中，我們使用「Bot 配接器」  將傳入活動傳送至 Bot。 配接器可讓我們在回合完成前攔截及回應錯誤。 在此，如果應用程式發生錯誤，我們會清除交談狀態，這會重設所有對話並且讓 Bot 停留在損毀的交談狀態。

1. 以此取代用於建立 Bot 的範本程式碼。

    ```javascript
    // Define state store for your bot.
    const memoryStorage = new MemoryStorage();

    // Create conversation state with in-memory storage provider.
    const conversationState = new ConversationState(memoryStorage);

    // Create the base dialog and bot
    const dialog = new MainDialog();
    const reservationBot = new ReservationBot(conversationState, dialog);
    ```

    記憶體內儲存層現在由 `MemoryStorage` 類別提供，而我們需要明確地建立交談狀態管理物件。

    對話定義程式碼已移至我們馬上要定義的 `MainDialog` 類別。 我們也會將 Bot 定義程式碼遷移到 `ReservationBot` 類別中。

1. 最後，我們會更新伺服器的要求處理常式，以使用配接器將活動傳送到 Bot。

    ```javascript
    // Listen for incoming requests.
    server.post('/api/messages', (req, res) => {
        adapter.processActivity(req, res, async (context) => {
            // Route incoming activities to the bot.
            await reservationBot.run(context);
        });
    });
    ```

    在 v4 中，我們的 Bot 衍生自 `ActivityHandler`，其可定義 `run` 方法來接收回合的活動。

## <a name="add-a-constants-file"></a>新增常數檔案

建立 **./const.js** 檔案以保留 Bot 的識別碼。

```javascript
module.exports = {
    MAIN_DIALOG: 'mainDialog',
    INITIAL_PROMPT: 'initialPrompt',
    HOTELS_DIALOG: 'hotelsDialog',
    INITIAL_HOTEL_PROMPT: 'initialHotelPrompt',
    CHECKIN_DATETIME_PROMPT: 'checkinTimePrompt',
    HOW_MANY_NIGHTS_PROMPT: 'howManyNightsPrompt',
    FLIGHTS_DIALOG: 'flightsDialog',
};
```

在 v4 中，識別碼會指派給對話和提示物件，而對話和提示會由識別碼叫用。

## <a name="create-new-dialog-files"></a>建立新的對話檔案

建立下列檔案：

| 檔案名稱 | 說明 |
|:---|:---|
| **./dialogs/flights.js** | 這會包含 `hotels` 對話的遷移後邏輯。 |
| **./dialogs/hotels.js** | 這會包含 `flights` 對話的遷移後邏輯。 |
| **./dialogs/main.js** | 這會包含 Bot 的遷移後邏輯，而且會代替「根」  對話。 |

我們尚未遷移支援對話。 如需如何在 v4 中實作協助對話的範例，請參閱[處理使用者中斷](../bot-builder-howto-handle-user-interrupt.md?tabs=javascript)。

### <a name="implement-the-main-dialog"></a>實作主要對話

在 v3 中，所有 Bot 都是建置於對話系統之上。 在 v4 中，Bot 邏輯和對話邏輯目前不同。 我們已採用 v3 Bot 中的「根對話」  ，並且讓 `MainDialog` 類別取代它的位置。

編輯 **./dialogs/main.js**。

1. 匯入我們在對話時所需的類別和常數。

    ```javascript
    const { DialogSet, DialogTurnStatus, ComponentDialog, WaterfallDialog,
        ChoicePrompt } = require('botbuilder-dialogs');
    const { FlightDialog } = require('./flights');
    const { HotelsDialog } = require('./hotels');
    const { MAIN_DIALOG,
        INITIAL_PROMPT,
        HOTELS_DIALOG,
        FLIGHTS_DIALOG
    } = require('../const');
    ```

1. 定義和匯出 `MainDialog` 類別。

    ```javascript
    const initialId = 'mainWaterfallDialog';

    class MainDialog extends ComponentDialog {
        constructor() {
            super(MAIN_DIALOG);

            // Create a dialog set for the bot. It requires a DialogState accessor, with which
            // to retrieve the dialog state from the turn context.
            this.addDialog(new ChoicePrompt(INITIAL_PROMPT, this.validateNumberOfAttempts.bind(this)));
            this.addDialog(new FlightDialog(FLIGHTS_DIALOG));

            // Define the steps of the base waterfall dialog and add it to the set.
            this.addDialog(new WaterfallDialog(initialId, [
                this.promptForBaseChoice.bind(this),
                this.respondToBaseChoice.bind(this)
            ]));

            // Define the steps of the hotels waterfall dialog and add it to the set.
            this.addDialog(new HotelsDialog(HOTELS_DIALOG));

            this.initialDialogId = initialId;
        }
    }

    module.exports.MainDialog = MainDialog;
    ```

    這會宣告主要對話直接參考的其他對話和提示。

    - 主要瀑布式對話，其中包含此對話的步驟。 當元件對話開始時，它會開始其「初始對話」  。
    - 我們將用來詢問使用者想要執行哪項工作的選擇提示。 我們已使用驗證程式建立選擇提示。
    - 兩個子對話：航班和旅館。

1. 將 `run` 協助程式方法新增至此類別。

    ```javascript
    /**
     * The run method handles the incoming activity (in the form of a TurnContext) and passes it through the dialog system.
     * If no dialog is active, it will start the default dialog.
     * @param {*} turnContext
     * @param {*} accessor
     */
    async run(turnContext, accessor) {
        const dialogSet = new DialogSet(accessor);
        dialogSet.add(this);

        const dialogContext = await dialogSet.createContext(turnContext);
        const results = await dialogContext.continueDialog();
        if (results.status === DialogTurnStatus.empty) {
            await dialogContext.beginDialog(this.id);
        }
    }
    ```

    在 v4 中，Bot 會先建立對話內容，然後呼叫 `continueDialog`，藉此與對話系統互動。 如果有作用中的對話，控制權就會交給它；否則，只會傳回此呼叫。 `empty` 的結果表示沒有作用中的對話，所以我們會在此再次重新開始主要對話。

    `accessor` 參數會傳入對話狀態屬性的存取子。 「對話堆疊」  的狀態會儲存在這個屬性中。 如需有關狀態和對話在 v4 中運作方式的詳細資訊，請分別參閱[管理狀態](../bot-builder-concept-state.md)和 [Dialogs 程式庫](../bot-builder-concept-dialog.md)。

1. 在此類別中，為選擇提示新增主要對話的瀑布式步驟和驗證程式。

    ```javascript
    async promptForBaseChoice(stepContext) {
        return await stepContext.prompt(
            INITIAL_PROMPT, {
                prompt: 'Are you looking for a flight or a hotel?',
                choices: ['Hotel', 'Flight'],
                retryPrompt: 'Not a valid option'
            }
        );
    }

    async respondToBaseChoice(stepContext) {
        // Retrieve the user input.
        const answer = stepContext.result.value;
        if (!answer) {
            // exhausted attempts and no selection, start over
            await stepContext.context.sendActivity('Not a valid option. We\'ll restart the dialog ' +
                'so you can try again!');
            return await stepContext.endDialog();
        }
        if (answer === 'Hotel') {
            return await stepContext.beginDialog(HOTELS_DIALOG);
        }
        if (answer === 'Flight') {
            return await stepContext.beginDialog(FLIGHTS_DIALOG);
        }
        return await stepContext.endDialog();
    }

    async validateNumberOfAttempts(promptContext) {
        if (promptContext.attemptCount > 3) {
            // cancel everything
            await promptContext.context.sendActivity('Oops! Too many attempts :( But don\'t worry, I\'m ' +
                'handling that exception and you can try again!');
            return await promptContext.context.endDialog();
        }

        if (!promptContext.recognized.succeeded) {
            await promptContext.context.sendActivity(promptContext.options.retryPrompt);
            return false;
        }
        return true;
    }
    ```

    瀑布的第一個步驟會開始選擇提示 (其本身就是對話)，藉此要求使用者做出選擇。 瀑布的第二個步驟會取用選擇提示的結果。 這會開始子對話 (如果已做出選擇) 或結束主要對話 (如果使用者無法做出選擇)。

    選擇提示會傳回使用者的選擇 (如果他們做出有效的選擇)，或重新提示使用者再次做出選擇。 驗證程式會檢查已對使用者連續提示多少次，並可讓提示在嘗試失敗 3 次後失效，進而將控制權交回給主要瀑布式對話。

### <a name="implement-the-flights-dialog"></a>實作航班對話

在 v3 Bot 中，航班對話是一個虛設常式，其示範 Bot 如何處理交談錯誤。 在此，我們會執行相同的動作。

編輯 **./dialogs/flights.js**。

```javascript
const { ComponentDialog, WaterfallDialog } = require('botbuilder-dialogs');

const initialId = 'flightsWaterfallDialog';

class FlightDialog extends ComponentDialog {
    constructor(id) {
        super(id);

        // ID of the child dialog that should be started anytime the component is started.
        this.initialDialogId = initialId;

        // Define the conversation flow using a waterfall model.
        this.addDialog(new WaterfallDialog(initialId, [
            async () => {
                throw new Error('Flights Dialog is not implemented and is instead ' +
                    'being used to show Bot error handling');
            }
        ]));
    }
}

exports.FlightDialog = FlightDialog;
```

### <a name="implement-the-hotels-dialog"></a>實作旅館對話

我們會保留旅館對話的相同整體流程：詢問目的地、詢問日期、詢問過夜天數，然後向使用者顯示符合其搜尋的選項清單。

編輯 **./dialogs/hotels.js**。

1. 匯入我們在對話時所需的類別和常數。

    ```javascript
    const { ComponentDialog, WaterfallDialog, TextPrompt, DateTimePrompt } = require('botbuilder-dialogs');
    const { AttachmentLayoutTypes, CardFactory } = require('botbuilder');
    const store = require('../store');
    const {
        INITIAL_HOTEL_PROMPT,
        CHECKIN_DATETIME_PROMPT,
        HOW_MANY_NIGHTS_PROMPT
    } = require('../const');
    ```

1. 定義和匯出 `HotelsDialog` 類別。

    ```javascript
    const initialId = 'hotelsWaterfallDialog';

    class HotelsDialog extends ComponentDialog {
        constructor(id) {
            super(id);

            // ID of the child dialog that should be started anytime the component is started.
            this.initialDialogId = initialId;

            // Register dialogs
            this.addDialog(new TextPrompt(INITIAL_HOTEL_PROMPT));
            this.addDialog(new DateTimePrompt(CHECKIN_DATETIME_PROMPT));
            this.addDialog(new TextPrompt(HOW_MANY_NIGHTS_PROMPT));

            // Define the conversation flow using a waterfall model.
            this.addDialog(new WaterfallDialog(initialId, [
                this.destinationPromptStep.bind(this),
                this.destinationSearchStep.bind(this),
                this.checkinPromptStep.bind(this),
                this.checkinTimeSetStep.bind(this),
                this.stayDurationPromptStep.bind(this),
                this.stayDurationSetStep.bind(this),
                this.hotelSearchStep.bind(this)
            ]));
        }
    }

    exports.HotelsDialog = HotelsDialog;
    ```

1. 在此類別中，新增幾個我們將在對話步驟中使用的協助程式函式。

    ```javascript
    addDays(startDate, days) {
        const date = new Date(startDate);
        date.setDate(date.getDate() + days);
        return date;
    };

    createHotelHeroCard(hotel) {
        return CardFactory.heroCard(
            hotel.name,
            `${hotel.rating} stars. ${hotel.numberOfReviews} reviews. From ${hotel.priceStarting} per night.`,
            CardFactory.images([hotel.image]),
            CardFactory.actions([
                {
                    type: 'openUrl',
                    title: 'More details',
                    value: `https://www.bing.com/search?q=hotels+in+${encodeURIComponent(hotel.location)}`
                }
            ])
        );
    }
    ```

    `createHotelHeroCard` 會建立包含旅館相關資訊的主圖卡片。

1. 在此類別中，新增對話中使用的瀑布式步驟。

    ```javascript
    async destinationPromptStep(stepContext) {
        await stepContext.context.sendActivity('Welcome to the Hotels finder!');
        return await stepContext.prompt(
            INITIAL_HOTEL_PROMPT, {
                prompt: 'Please enter your destination'
            }
        );
    }

    async destinationSearchStep(stepContext) {
        const destination = stepContext.result;
        stepContext.values.destination = destination;
        await stepContext.context.sendActivity(`Looking for hotels in ${destination}`);
        return stepContext.next();
    }

    async checkinPromptStep(stepContext) {
        return await stepContext.prompt(
            CHECKIN_DATETIME_PROMPT, {
                prompt: 'When do you want to check in?'
            }
        );
    }

    async checkinTimeSetStep(stepContext) {
        const checkinTime = stepContext.result[0].value;
        stepContext.values.checkinTime = checkinTime;
        return stepContext.next();
    }

    async stayDurationPromptStep(stepContext) {
        return await stepContext.prompt(
            HOW_MANY_NIGHTS_PROMPT, {
                prompt: 'How many nights do you want to stay?'
            }
        );
    }

    async stayDurationSetStep(stepContext) {
        const numberOfNights = stepContext.result;
        stepContext.values.numberOfNights = parseInt(numberOfNights);
        return stepContext.next();
    }

    async hotelSearchStep(stepContext) {
        const destination = stepContext.values.destination;
        const checkIn = new Date(stepContext.values.checkinTime);
        const checkOut = this.addDays(checkIn, stepContext.values.numberOfNights);

        await stepContext.context.sendActivity(`Ok. Searching for Hotels in ${destination} from 
            ${checkIn.toDateString()} to ${checkOut.toDateString()}...`);
        const hotels = await store.searchHotels(destination, checkIn, checkOut);
        await stepContext.context.sendActivity(`I found in total ${hotels.length} hotels for your dates:`);

        const hotelHeroCards = hotels.map(this.createHotelHeroCard);

        await stepContext.context.sendActivity({
            attachments: hotelHeroCards,
            attachmentLayout: AttachmentLayoutTypes.Carousel
        });

        return await stepContext.endDialog();
    }
    ```

    我們已將 v3 旅館對話中的步驟遷移到 v4 旅館對話的瀑布式步驟。

## <a name="update-the-bot"></a>更新 Bot

在 v4 中，Bot 可因應對話系統外部的活動。 `ActivityHandler` 類別可定義常見活動類型的處理常式，讓您更輕鬆地管理程式碼。

將 **./bot.js** 重新命名為 **./bots/reservationBot.js**，並加以編輯。

1. 檔案已經匯入 **ActivityHandler**，其可提供 Bot 的基底實作。

    ```javascript
    const { ActivityHandler } = require('botbuilder');
    ```

1. 將類別重新命名為 `ReservationBot`。

    ```javascript
    class ReservationBot extends ActivityHandler {
        // ...
    }

    module.exports.ReservationBot = ReservationBot;
    ```

1. 更新建構函式的簽章，以接受我們所收到的物件。

    ```javascript
    /**
     *
     * @param {ConversationState} conversationState
     * @param {Dialog} dialog
     * @param {any} logger object for logging events, defaults to console if none is provided
    */
    constructor(conversationState, dialog, logger) {
        super();
        // ...
    }
    ```

1. 在建構函式中，新增 null 參數檢查並定義類別建構函式屬性。

    ```javascript
    if (!conversationState) throw new Error('[DialogBot]: Missing parameter. conversationState is required');
    if (!dialog) throw new Error('[DialogBot]: Missing parameter. dialog is required');
    if (!logger) {
        logger = console;
        logger.log('[DialogBot]: logger not passed in, defaulting to console');
    }

    this.conversationState = conversationState;
    this.dialog = dialog;
    this.logger = logger;
    this.dialogState = this.conversationState.createProperty('DialogState');
    ```

    這是我們建立對話狀態屬性存取子的地方，而該存取子將會儲存對話堆疊的狀態。

1. 在建構函式中，更新 `onMessage` 處理常式並新增 `onDialog` 處理常式。

    ```javascript
    this.onMessage(async (context, next) => {
        this.logger.log('Running dialog with Message Activity.');

        // Run the Dialog with the new message Activity.
        await this.dialog.run(context, this.dialogState);

        // By calling next() you ensure that the next BotHandler is run.
        await next();
    });

    this.onDialog(async (context, next) => {
        // Save any state changes. The load happened during the execution of the Dialog.
        await this.conversationState.saveChanges(context, false);

        // By calling next() you ensure that the next BotHandler is run.
        await next();
    });
    ```

    `ActivityHandler` 會將訊息活動傳送至 `onMessage`。 此 Bot 會處理所有透過對話的使用者輸入。

    `ActivityHandler` 會在回合結束時呼叫 `onDialog`，再將控制權還給配接器。 我們必須先明確地儲存狀態，再結束回合。 否則，將無法儲存狀態變更，而對話將無法正常執行。

1. 最後，更新處理常式中的 `onMembersAdded` 建構函式。

    ```javascript
    this.onMembersAdded(async (context, next) => {
        const membersAdded = context.activity.membersAdded;
        for (let cnt = 0; cnt < membersAdded.length; ++cnt) {
            if (membersAdded[cnt].id !== context.activity.recipient.id) {
                await context.sendActivity('Hello and welcome to Contoso help desk bot.');
            }
        }
        // By calling next() you ensure that the next BotHandler is run.
        await next();
    });
    ```

    `ActivityHandler` 會在收到交談更新活動時呼叫 `onMembersAdded`，這表示 Bot 以外的參與者已加入交談。 我們會更新這個方法，以在使用者加入交談時傳送問候訊息。

## <a name="create-the-store-file"></a>建立存放區檔案

建立旅館對話所使用的 **./store.js** 檔案。 如同在 v3 Bot 中，`searchHotels` 是模擬旅館搜尋函式。

```javascript
module.exports = {
    searchHotels: destination => {
        return new Promise(resolve => {

            // Filling the hotels results manually just for demo purposes
            const hotels = [];
            for (let i = 1; i <= 5; i++) {
                hotels.push({
                    name: `${destination} Hotel ${i}`,
                    location: destination,
                    rating: Math.ceil(Math.random() * 5),
                    numberOfReviews: Math.floor(Math.random() * 5000) + 1,
                    priceStarting: Math.floor(Math.random() * 450) + 80,
                    image: `https://placeholdit.imgix.net/~text?txtsize=35&txt=Hotel${i}&w=500&h=260`
                });
            }

            hotels.sort((a, b) => a.priceStarting - b.priceStarting);

            // complete promise with a timer to simulate async response
            setTimeout(() => { resolve(hotels); }, 1000);
        });
    }
};
```

## <a name="test-the-bot-in-the-emulator"></a>在模擬器中測試 Bot

此時，我們應該能夠在本機執行 Bot，然後將它與模擬器連結。

1. 在您的電腦本機執行範例。
    如果您在 Visual Studio Code 中開始偵錯工作階段，記錄資訊會在您測試 Bot 時傳送到偵錯主控台。
1. 啟動模擬器並連線到 Bot。
1. 傳送訊息以測試主要、航班和旅館對話。

## <a name="additional-resources"></a>其他資源

v4 概念性主題：

- [Bot 的運作方式](../bot-builder-basics.md)
- [管理狀態](../bot-builder-concept-state.md)
- [對話方塊程式庫](../bot-builder-concept-dialog.md)

v4 作法主題：

- [傳送及接收文字訊息](../bot-builder-howto-send-messages.md)
- [儲存使用者和對話資料](../bot-builder-howto-v4-state.md)
- [實作循序對話流程](../bot-builder-dialog-manage-conversation-flow.md)
