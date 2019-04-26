---
title: 使用 CLI 工具管理 Bot
description: Bot Framework 工具可讓您直接從命令列管理 Bot 資源
keywords: botbuilder 範本, ludown, qna, luis, msbot, 管理, cli, .bot, bot
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: tools
ms.date: 11/13/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 07df43111f3b2e57dcf0140f291a771e749de563
ms.sourcegitcommit: c6ce4c42fc56ce1e12b45358d2c747fb77eb74e2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/22/2019
ms.locfileid: "54453852"
---
# <a name="manage-bots-using-cli-tools"></a>使用 CLI 工具管理 Bot

Bot Framework 工具涵蓋端對端 Bot 開發工作流程，其中包括規劃、建置、測試、發佈、連線及評估階段。 我們來看看這些工具如何協助您進行開發週期中的每個階段。

## <a name="plan"></a>規劃

### <a name="create-mock-conversations-using-chatdown"></a>使用 Chatdown 建立模擬對話

Chatdown 是一個文字記錄產生器，會使用 .chat 檔案來產生模擬文字記錄。 產生的模擬文字記錄檔會輸出至 stdout。

如同任何成功的應用程式或網站，良好的 Bot 在支援案例上也會有清楚明確的開端。 在 Bot 和使用者之間建立對話原型適用於：

- 塑造 Bot 所支援的案例。
- 讓商務決策者檢視，並提供意見。
- 透過使用者與 Bot 之間的對話流程定義「理想路徑 (happy path)」(以及其他路徑)。chat 檔案格式可協助您建立使用者與 Bot 之間的對話原型。 Chatdown CLI 工具會將 .chat 檔案轉換成對話文字記錄 (.transc ript 檔案)，而您可以在 [Bot Framework 模擬器 V4](https://github.com/microsoft/botframework-emulator) 中加以檢視。

以下是 `.chat` 檔案的範例：

```markdown
user=Joe
bot=LulaBot

bot: Hi!
user: yo!
bot: [Typing][Delay=3000]
Greetings!
What would you like to do?
* update - You can update your account
* List - You can list your data
* help - you can get help

user: I need the bot framework logo.

bot:
Here you go.
[Attachment=bot-framework.png]
[Attachment=http://yahoo.com/bot-framework.png]
[AttachmentLayout=carousel]

user: thanks
bot:
Here's a form for you
[Attachment=card.json adaptivecard]
```

### <a name="create-a-transcript-file-from-chat-file"></a>從 .chat 檔案建立文字記錄檔

Chatdown 命令看起來如下所示︰

```bash
chatdown sample.chat > sample.transcript
```

這會使用 `sample.chat` 並輸出 `sample.transcript`。 請參閱 [Chatdown CLI][chatdown] 以取得詳細資訊。

## <a name="build"></a>建置

### <a name="create-a-luis-application-with-ludown"></a>使用 LUDown 建立 LUIS 應用程式

LUDown 工具可用來建立適用於 LUIS 和 QnA 的新 .json 模型。  
您可以定義 LUIS 應用程式的[意圖](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/add-intents)和[實體](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/add-entities)，就如同從 LUIS 入口網站進行。

'#\<intent-name\>' 可描述新的意圖定義區段。 之後的每一行都會列出描述該意圖的[語句](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/add-example-utterances)。

例如，您可以在單一 .lu 檔案中建立多個 LUIS 意圖，如下所示：

```LUDown
# Greeting
- Hi
- Hello
- Good morning
- Good evening

# Help
- help
- I need help
- please help
```

### <a name="create-qna-pairs-with-ludown"></a>使用 LUDown 建立 QnA 組合

.lu 檔案格式也會使用下列標記法支援 QnA 組合：

~~~LUDown
> comment
### ? question ?
  ```
    answer
  ```
~~~

LUDown 工具會自動將問題和解答分開到 qnamaker JSON 檔案，以便您可用來建立您的新 [QnaMaker.ai](http://qnamaker.ai) (英文) 知識庫。

~~~LUDown
### ? How do I change the default message for QnA Maker?
  ```
  You can change the default message if you use the QnAMakerDialog. 
  See [this link](https://docs.botframework.com/en-us/azure-bot-service/templates/qnamaker/#navtitle) for details.
  ```
~~~

您也可以將多個問題新增到相同的答案，只需針對單一解答新增問題變化的新行即可。

~~~LUDown
### ? What is your name?
- What should I call you?
  ```
    I'm the echoBot! Nice to meet you.
  ```
~~~

### <a name="generate-json-models-with-ludown"></a>使用 LUDown 產生 .json 模型

您以 .lu 格式定義 LUIS 或 QnA 語言元件之後，可以發佈至 LUIS.json、QnA.json 或 QnA.tsv 檔案。 執行時，LUDown 工具會在要剖析的相同工作目錄內尋找任何 .lu 檔案。 因為 LUDown 工具可以使用 .lu 檔案將目標鎖定為 LUIS 或 QnA，所以我們只需使用一般命令 **ludown parse \<to-service-type> -- in \<lu-file-path>**，指定所要產生的語言服務。

在我們的範例工作目錄中，有兩個要剖析的 .lu 檔案，用來建立 LUIS 模型的 '1.lu'，以及用來建立 QnA 知識庫的 'qna1.lu'。

#### <a name="generate-luis-json-models"></a>產生 LUIS.json 模型

若要使用 LUDown 產生 LUIS 模型，只需在您目前的工作目錄中輸入下列內容即可：

```shell
ludown parse ToLuis --in <luFile>
```

#### <a name="generate-qna-knowledge-base"></a>產生 QnA 知識庫

同樣地，若要建立 QnA 知識庫，只需變更剖析目標即可。

```shell
ludown parse ToQna --in <luFile>
```

LUIS 和 QnA 可透過其個別的入口網站，或透過新的 CLI 工具來取用所產生的 JSON 檔案。 若要深入了解，請參閱 [LUdown CLI][ludown] GitHub 存放庫。

### <a name="track-service-references-using-bot-file"></a>使用.bot 檔案追蹤服務參考

新的 [MSBot][msbotCli] (英文) 工具可讓您建立 **.bot** 檔案，這會儲存您 Bot 所取用不同服務的相關中繼資料，全都放在一個位置。 此檔案也可讓您的 Bot 從 CLI 連線至這些服務。 此工具會以 npm 模組的形式提供，若要安裝請執行：

```shell
npm install -g msbot
```

若要建立 Bot 檔案，請從您的 CLI 輸入 **msbot init**，後接您 Bot 的名稱和目標 URL 端點，例如：

```shell
msbot init --name TestBot --endpoint http://localhost:9499/api/messages
```
若要將您的 Bot 連線到服務，請在 CLI 中輸入 **msbot connect**，後接適當的服務：

```shell
msbot connect [Service]
```

若要取得支援的服務清單，請參閱[讀我檔案][msbotCli]。

### <a name="create-and-manage-luis-applications-using-luis-cli"></a>使用 LUIS CLI 建立和管理 LUIS 應用程式

新工具組中所包含的是 [LUIS 擴充功能][luisCli]，可讓您獨立管理您的 LUIS 資源。 它會以 npm 模組的形式提供，可供您下載：

```shell
npm install -g luis-apis
```
從 CLI 使用 LUIS 工具基本命令的方式為：

```shell
luis <action> <resource> <args...>
```
若要將您的 Bot 連線至 LUIS，您必須建立 **.luisrc** 檔案。 當您的應用程式進行輸出呼叫時，這個組態檔可將您的 LUIS appID 和密碼佈建到服務端點。 您可以執行 **luis init** 來建立這個檔案，如下所示：

```shell
luis init
```
此工具在產生檔案之前，系統會提示您在終端機中輸入 LUIS 製作金鑰、區域和 appID。  

產生這個檔案之後，您的應用程式將能夠從 CLI 使用下列命令，來取用 LUIS .json 檔案 (從 LUDown 產生)。

```shell
luis import application --in luis-app.json | msbot connect luis --stdin
```
若要深入了解，請參閱 [LUIS CLI][luisCli] GitHub 存放庫。

### <a name="create-qna-maker-kb-using-qna-maker-cli"></a>使用 QnA Maker CLI 建立 QnA Maker KB

新工具組中所包含的是 [QnA 擴充功能][qnaCli] (英文)，可讓您獨立管理您的 LUIS 資源。 它會以 npm 模組的形式提供，可供您下載。

```shell
npm install -g qnamaker
```
透過 QnA Maker 工具，您可以建立、更新、發佈、刪除及訓練知識庫。 您可以使用透過 [ludown parse toqna](#generate-qna-knowledge-base) 命令產生的檔案來建立/取代知識庫。

```shell
qnamaker create --in qnaKB.json --msbot | msbot connect qna --stdin
```

若要深入了解，請參閱 [QnA Maker CLI][qnaCli] GitHub 存放庫。

### <a name="create-dispatch-model-using-dispatch-cli"></a>使用分派 CLI 建立分派模型

分派是建立和評估 LUIS 模型的工具，用來將意圖分派到多個 Bot 模組，例如 LUIS 模型、QnA 知識庫和其他項目 (已新增並分派為檔案類型)。

適用分派模型的案例：

- 您的 Bot 包含多個模組，而且需要工具來協助您將使用者的語句路由到這些模組，並評估 Bot 整合。
- 評估單一 LUIS 模型的意圖分類品質。
- 從文字檔建立文字分類模型。

一旦您使用 Bot 依賴的 [LUIS 應用程式][msbotCli-luis]和 [QnA Maker 知識庫][msbotCli-qna] 來彙總 Bot 檔案，您就可以直接使用下列命令建置分派模型： 

```shell
dispatch create -b <YOUR-BOT-FILE> | msbot connect dispatch --stdin
```
若要深入了解，請參閱 [Disptach CLI][dispatchCli]。

## <a name="test"></a>測試

[Bot Framework 模擬器](bot-service-debug-emulator.md)是一項桌面應用程式，可讓 Bot 開發人員在本機或透過通道從遠端對其 Bot 進行測試和偵錯。

## <a name="publish"></a>發佈

您可以使用 Azure CLI 來建立和下載 Bot，並將其發佈至 Azure Bot Service。

使用 msbot 4.3.2 和更新版本，您需要 Azure CLI 2.0.53 版或更新版本。 如果您安裝了 botservice 擴充功能，則使用此命令加以移除。

```shell
az extension remove --name botservice
```

### <a name="create-azure-bot-service-bot"></a>建立 Azure Bot Service Bot

注意：您必須使用最新版的 `az cli`。 請升級，以便 az cli 搭配 MSBot 工具運作。

透過下列命令登入您的 Azure 帳戶

```shell
az login
```

如果您還沒有可供發佈 Bot 的資源群組，請建立一個：

```shell
az group create --name <resource-group-name> --location <geographic-location> --verbose
```

| 選項 | 說明 |
|:---|:---|
| --name | 資源群組的唯一名稱。 請勿在名稱中包含空格或底線。 |
| --location | 用來建立資源群組的地理位置。 例如，`eastus`、`westus` 及 `westus2` 等等。 使用 `az account list-locations` 來取得位置清單。 |

然後，建立您將在其中發佈 Bot 的 Bot 資源。

```shell
az bot create [options]
```

若要使用 Bot 設定來建立 Bot 並更新 .bot 檔案  
```shell
az bot create [options] --msbot | msbot connect bot --stdin
```

如果您已經有 Bot  
```shell
az bot show [options] --msbot | msbot connect bot --stdin
```

| 選項                            | 說明                                   |
|-----------------------------------|-----------------------------------------------|
| --kind -k [必要]              | Bot 的種類。  允許的值：function、registration、webapp。|
| --name -n [必要]              | Bot 的資源名稱。 |
| --appid                           | 要用於 Bot 的 msa 帳戶識別碼。   |
| --location -l                     | 位置。 您可以使用 `az configure --defaults location=<location>` 來設定預設位置。  預設：westus。|
| --msbot                           | 將輸出顯示為與 .bot 檔案相容的 json。  允許的值：false、true。|
| --password -p                     | 開發人員入口網站的 Bot msa 密碼。 |
| --resource-group -g               | 資源群組的名稱。 您可以使用 `az configure --defaults group=<name>` 來設定預設群組。  預設：build2018。 |
| --tags                            | 要新增至 Bot 的標籤組。 |

### <a name="configure-channels"></a>設定通道

您可以使用 Azure CLI 來管理您的 Bot 通道。

```shell
>az bot -h
Group
   az bot: Manage Bot Services.
    Subgroups:
        directline: Manage Directline Channel on a Bot.
        email     : Manage Email Channel on a Bot.
        facebook  : Manage Facebook Channel on a Bot.
        kik       : Manage Kik Channel on a Bot.
        msteams   : Manage Msteams Channel on a Bot.
        skype     : Manage Skype Channel on a Bot.
        slack     : Manage Slack Channel on a Bot.
        sms       : Manage Sms Channel on a Bot.
        telegram  : Manage Telegram Channel on a Bot.
        webchat   : Manage Webchat Channel on a Bot.

    Commands:
        create    : Create a new Bot Service.
        delete    : Delete an existing Bot Service.
        download  : Download an existing Bot Service.
        publish   : Publish to an existing Bot Service.
        show      : Get an existing Bot Service.
        update    : Update an existing Bot Service.

```

## <a name="additional-information"></a>其他資訊

- [GitHub 上的 Bot Framework 工具][cliTools]
- [.lu 檔案表單](https://aka.ms/ludown-file-format)

<!-- Footnote links -->

[cliTools]: https://aka.ms/botbuilder-tools-readme
[azureCli]: https://aka.ms/botbuilder-tools-azureCli
[msbotCli]: https://aka.ms/botbuilder-tools-msbot-readme
[msbotCli-luis]: https://aka.ms/botbuilder-tools-msbot-readme#connecting-to-luis-application
[msbotCli-qna]: https://aka.ms/botbuilder-tools-msbot-readme#connecting-to-qna-maker-knowledge-base
[chatdown]: https://aka.ms/botbuilder-tools-chatdown
[ludown]: https://aka.ms/botbuilder-ludown
[luisCli]: https://aka.ms/botbuilder-luis-cli
[qnaCli]: https://aka.ms/botbuilder-tools-qnaMaker
[dispatchCli]: https://aka.ms/botbuilder-tools-dispatch
