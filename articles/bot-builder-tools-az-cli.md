---
title: 使用 Azure CLI 建立 Bot
description: Botbuilder 工具可讓您直接從命令列管理 Bot 資源
author: matthewshim-ms
ms.author: v-shimma
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 04/25/2018
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 20258949cd8ea403e5cc9bf774d6a3b7c1e86e7e
ms.sourcegitcommit: dcbc8ad992a3e242a11ebcdf0ee99714d919a877
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/30/2018
ms.locfileid: "39352897"
---
# <a name="create-bots-with-azure-cli"></a>使用 Azure CLI 建立 Bot

[Bot 建立器工具](https://github.com/microsoft/botbuilder-tools)是新的工具組，可讓您直接從命令列管理 Bot 資源並與其進行互動。 

本教學課程將為您示範如何：

- 啟用 Azure CLI Bot 擴充功能
- 使用 Azure CLI 建立新的 Bot 
- 下載適用於開發的本機複本
- 使用新 MSBot 工具儲存您所有的 Bot 資源資訊
- 使用 LUDown 來管理、建立或更新 LUIS 和 QnA 模型
- 從 CLI 連線至 LUIS 和 QnA Maker 服務
- 使用 CLI 將 Bot 部署至 Azure

## <a name="prerequisites"></a>必要條件

若要從命令列啟用這些工具，您必須在電腦上安裝 Node.js： 

- [Node.js (v8.5 或更新版本)](https://nodejs.org/en/)

## <a name="1-enable-azure-cli"></a>1.啟用 Azure CLI

您現在已可使用 [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/?view=azure-cli-latest) 來管理 Bot，就像任何其他 Azure 資源一樣。 若要啟用 Azure CLI，請完成下列步驟：

1. [下載](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest) Azure CLI (如果您還沒有)。 

2. 輸入下列命令以下載 Azure Bot 擴充功能散發套件。

```azurecli
az extension add -n botservice
```

>[!TIP]
> Azure Bot 擴充功能目前僅支援 v3 Bot。
  
3. 執行下列命令以[登入](https://docs.microsoft.com/en-us/cli/azure/authenticate-azure-cli?view=azure-cli-latest) Azure CLI。

```azurecli
az login
```
系統會提示您提供唯一的暫時性驗證碼。 若要登入，請使用網頁瀏覽器瀏覽 Microsoft [裝置登入](https://microsoft.com/devicelogin)，並貼上 CLI 所提供的程式碼，以繼續操作。 

![MS 裝置登入](media/bot-builder-tools/ms-device-login.png)

成功登入後，您會看到 Azure CLI 歡迎畫面，以及可供您管理帳戶和資源的選項清單。

![Azure Bot CLI](media/bot-builder-tools/az-cli-bot.png)


 如需 Azure CLI 命令的完整清單，請[按一下這裡](https://docs.microsoft.com/en-us/cli/azure/reference-index?view=azure-cli-latest)。


## <a name="2-create-a-new-bot-from-azure-cli"></a>2.從 Azure CLI 建立新的 Bot

使用 Azure CLI 和新的 Bot 擴充功能，可讓您完全從命令列建立新的 Bot。 

```azurecli
az bot [command]
```
|命令|  |
|----|----|
| create      |新增資源|
| delete     |複製資源|
| 下載   | 下載 Bot 原始程式碼|
| publish   |發佈至現有的 Bot Service|
| 顯示 |顯示現有的 Bot 資源。|
| update| 更新現有的 Bot Service|

若要從 CLI 建立新的 Bot，您必須選取現有的[資源群組](https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-overview)，或新的資源群組。 

```azurecli
az bot create --resource-group "my-resource-group" --name "my-bot-name" --kind "my-resource-type" --description "description-of-my-bot"
```
要求成功後，您會看到下列確認訊息。
```
obtained msa app id and password. Provisioning bot now.
```

> [!TIP]
> 如果出現**找不到資源群組**的錯誤訊息，您可能需要在 Azure CLI 中設定您的[訂用帳戶](https://docs.microsoft.com/en-us/azure/architecture/cloud-adoption-guide/adoption-intro/subscription-explainer)。 Azure 訂用帳戶必須符合您在建立資源群組時輸入的訂用帳戶。 若要加以設定，請輸入以下內容。
> ```azurecli
> az account set --subscription "your-subscription-name"
> ```
> 若要檢視您帳戶的訂用帳戶清單，請輸入下列命令。
> ```azurecli
> az account list
> ```

依預設會建立新的.NET Bot。 您可以使用 **-- lang** 引數指定語言，藉以指定確切的平台 SDK。 目前，Bot 擴充功能套件可支援 C# 和 Node.js Bot SDK。 例如，若要**建立 Node.js Bot**：

```azurecli
az bot create --resource-group "my-resource-group" --name "my-bot-name" --kind "my-resource-type" --description "description-of-my-bot" --lang Node 
```
新的回應 Bot 將會佈建到您在 Azure 上的資源群組，若要加以測試，只需在 Web 應用程式 Bot 檢視的 Bot 管理標頭下方選取 [在網路聊天中測試] 即可。 

![Azure 回應 Bot](media/bot-builder-tools/az-echo-bot.png) 

## <a name="3-download-the-bot-locally"></a>3.在本機下載 Bot

有兩種方式可用來下載新 Bot 的原始程式碼。
- 從 Azure 入口網站下載。
- 使用新的 Azure CLI 下載。

若要從入口網站下載 Bot 的原始程式碼，只要選取您的 Bot 資源，再選取 Bot 管理下方的 [組建] 即可。 有數個不同的選項可讓您在本機管理或擷取 Bot 的原始程式碼。 

![Azure 入口網站 Bot 下載](media/bot-builder-tools/az-portal-manage-code.png)

若要使用 CLI 來下載 Bot 原始碼，請輸入下列命令。 您的 Bot 會下載至子目錄。 如果子目錄尚不存在，命令會自動加以建立。

```azurecli
az bot download --name "my-bot-name" --resource-group "my-resource-group"
```
不過，您也可以指定要用來下載 Bot 的目錄。
例如︰

![CLI 下載命令](media/bot-builder-tools/cli-bot-download-command.png)

![CLI Bot 下載](media/bot-builder-tools/cli-bot-download.png)

上述命令可讓您直接將 Bot 的原始程式碼下載至指定的位置，進而在本機開發您的 Bot。


## <a name="4-store-your-bot-information-with-msbot"></a>4.使用 MSBot 儲存您的 Bot 資訊

新的 [MSBot](https://github.com/Microsoft/botbuilder-tools/tree/master/MSBot) 工具可讓您建立 **.bot** 檔案，其中儲存與 Bot 取用不同服務有關的中繼資料，全都放在同一個位置。 此檔案也可讓您的 Bot 從 CLI 連線至這些服務。 此工具可作為 npm 模組，若要加以安裝，請執行：

```shell
npm install -g msbot 
```

若要建立 Bot 檔案，請從您的 CLI 輸入 **msbot init**，其後再加上 Bot 的名稱和目標 URL 端點，例如：

```shell
msbot init --name name-of-my-bot --endpoint http://localhost:bot-port-number/api/messages
```
若要將您的 Bot 連線至服務，請在 CLI 中輸入 **msbot connect**，後接適當的服務：

```shell
msbot connect service-type
```

| 服務類型 | 說明 |
| ------ | ----------- |
| azure  |將您的 Bot 連線至 Azure Bot Service 註冊|
|endpoint| 將您的 Bot 連線至 localhost 之類的端點|
|luis     | 將您的 Bot 連線至 LUIS 應用程式 |
| qna     |將您的 Bot 連線至 QnA 知識庫|
|help [cmd]  |顯示 [cmd] 的說明|

### <a name="connect-your-bot-to-abs-with-the-bot-file"></a>使用 .bot 檔案將您的 Bot 連線至 ABS

安裝 MSBot 工具後，您可以執行 az bot **show** 命令，輕鬆地將 Bot 連線至 Azure Bot Service 中現有的資源群組。 

```azurecli
az bot show -n my-bot-name -g my-resource-group --msbot | msbot connect azure --stdin
```

這會從目標資源群組中取得目前的端點、MSA appID 和密碼，並據此更新 .bot 檔案中的資訊。 


## <a name="5-manage-update-or-create-luis-and-qna-services-with--new-botbuilder-tools"></a>5.使用新的 Botbuilder 工具來管理、更新或建立 LUIS 和 QnA 服務

[Bot 建立器工具](https://github.com/microsoft/botbuilder-tools)是新的工具組，可讓您直接從命令列管理 Bot 資源並與其進行互動。 

>[!TIP]
> 每個 Bot 建立器工具都包含全域的 help 命令，可輸入 **-h** 或 **--help**，從命令列中加以存取。 您可隨時從任何動作中使用此命令，這會為您提供可用選項及其描述的實用顯示。

### <a name="ludown"></a>LUDown
[LUDown](https://github.com/Microsoft/botbuilder-tools/tree/master/Ludown) 可讓您使用 **.lu** 檔案來說明及建立功能強大的語言元件。 新的 .lu 檔案是一種 Markdown 格式，LUDown 工具會加以取用，並輸出目標服務特定的 .json 檔案。 目前，針對每個使用不同的格式，您可以使用 .lu 檔案來建立新的 [LUIS](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/luis-get-started-create-app) 應用程式或 [QnA](https://qnamaker.ai/Documentation/CreateKb) 知識庫。 LUDown 可用來作為 npm 模組，並可全域安裝到您的電腦來使用：

```shell
npm install -g ludown
```
LUDown 工具可用來建立適用於 LUIS 和 QnA 的新 .json 模型。  


### <a name="creating-a-luis-application-with-ludown"></a>使用 LUDown 建立 LUIS 應用程式

您可以定義 LUIS 應用程式的[意圖](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/add-intents)和[實體](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/add-entities)，就如同從 LUIS 入口網站進行。 

`# \<intent-name\>` 會說明新的意圖定義區段。 後續幾行包含說明該意圖的[語句](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/add-example-utterances)。

例如，您可以在單一 .lu 檔案中建立多個 LUIS 意圖，如下所示： 

```ludown
# Greeting
Hi
Hello
Good morning
Good evening

# Help
help
I need help
please help
```

### <a name="qna-pairs-with-ludown"></a>QnA 與 LUDown 的組合

.lu 檔案格式也會使用下列標記法支援 QnA 組合： 

  ```ludown
  > This is a comment. QnA definitions have the general format:
  ### ? this-is-the-question-string
  - this-is-an-alternate-form-of-the-same-question
  - this-is-another-one
    ```markdown
    this-is-the-answer
    ```
  ```
LUDown 工具會自動將問題和解答分開到 qnamaker JSON 檔案，以便您用來建立新的 [QnaMaker.ai](http://qnamaker.ai) 知識庫。

  ```ludown
  ### ? How do I change the default message for QnA Maker?
    ```markdown
    You can change the default message if you use the QnAMakerDialog. 
    See [this link](https://docs.botframework.com/en-us/azure-bot-service/templates/qnamaker/#navtitle) for details. 
    ```
  ```

您也可以將多個問題新增到相同的答案，只需針對單一解答新增問題變化的新行。 

  ```ludown
  ### ? What is your name?
  - What should I call you?
    ```markdown
    I'm the echoBot! Nice to meet you.
    ```
  ```

### <a name="generating-json-models-with-ludown"></a>使用 LUDown 產生 .json 模型

您以 .lu 格式定義 LUIS 或 QnA 語言元件之後，可以發佈至 LUIS.json、QnA.json 或 QnA.tsv 檔案。 執行時，LUDown 工具會在要剖析的相同工作目錄內尋找任何 .lu 檔案。 因為 LUDown 工具可以使用 .lu 檔案將目標鎖定為 LUIS 或 QnA，所以我們只需使用 <luFile> 中的一般命令 ludown parse <Service> --，指定所要產生的語言服務。 

在我們的範例工作目錄中，有兩個要剖析的 .lu 檔案，用來建立 LUIS 模型的 'luis-sample.lu'，以及用來建立 QnA 知識庫的 'qna-sample.lu'。


#### <a name="generate-luis-json-models"></a>產生 LUIS.json 模型

**luis-sample.lu** 
```ludown
# Greeting
- Hi
- Hello
- Good morning
- Good evening
```

若要使用 LUDown 產生 LUIS 模型，只需在您目前的工作目錄中輸入下列內容即可：

```shell
ludown parse ToLuis --in ludown-file-name.lu
```

#### <a name="generate-qna-knowledge-base-json"></a>產生 QnA 知識庫 .json

**qna-sample.lu**
  ```ludown
  > This is a sample ludown file for QnA Maker.

  ### ? How do I change the default message
    ```markdown
    You can change the default message if you use the QnAMakerDialog. 
    See [this link](https://docs.botframework.com/en-us/azure-bot-service/templates/qnamaker/#navtitle) for details. 
    ```
  ```

同樣地，若要建立 QnA 知識庫，只需變更剖析目標即可。 

```shell
ludown parse ToQna --in ludown-file-name.lu
```

LUIS 和 QnA 可透過其個別的入口網站，或透過新的 CLI 工具來取用所產生的 JSON 檔案。 

## <a name="6-connect-to-luis-an-qna-maker-services-from-the-cli"></a>6.從 CLI 連線至 LUIS 和 QnA Maker 服務

### <a name="connect-to-luis-from-the-cli"></a>從 CLI 連線至 LUIS 

新工具組中所包含的是 [LUIS 擴充功能](https://github.com/Microsoft/botbuilder-tools/tree/master/LUIS)，可讓您獨立管理您的 LUIS 資源。 它可用來作為 npm 模組，以供您下載：

```shell
npm install -g luis-apis
```
從 CLI 使用 LUIS 工具基本命令的方式為：

```shell
luis action-name resource-name arguments-list
```
若要將您的 bot 連線至 LUIS，您必須建立 **.luisrc** 檔案。 當您的應用程式進行輸出呼叫時，這個組態檔可將您的 LUIS appID 和密碼佈建到服務端點。 您可以執行 **luis init** 來建立這個檔案，如下所示：

```shell
luis init
```
此工具在產生檔案之前，系統會提示您在終端機中輸入 LUIS 製作金鑰、區域和 appID。  

![LUIS init](media/bot-builder-tools/luis-init.png) 


產生這個檔案之後，您的應用程式將能夠從 CLI 使用下列命令，來取用 LUIS .json 檔案 (從 LUDown 產生)： 

```shell
luis import application --in luis-app.json | msbot connect luis --stdin
```

### <a name="connect-to-qna-from-the-cli"></a>從 CLI 連線至 QnA

新工具組中所包含的是 [QnA 擴充功能](https://github.com/Microsoft/botbuilder-tools/tree/master/QnAMaker)，可讓您獨立管理 LUIS 資源。 它可用來作為 npm 模組，以供您下載：

```shell
npm install -g qnamaker
```
透過 QnA Maker 工具，您可以建立、更新、發佈、刪除及訓練知識庫。 若要開始使用，您必須建立 **.qnamakerrc** 檔案，以啟用服務端點。 您可以執行 **qnamaker init**，並遵循提示和使用您的 QnA Maker 知識庫識別碼，輕鬆地建立這個檔案。 

```shell
qnamaker init 
```
![QnaMaker init](media/bot-builder-tools/qnamaker-init.png)

在產生 .qnamakerrc 檔案後，您現在可以連線至 QnA 知識庫，使用下列命令來取用知識庫 .json/.tsv 檔案 (從 LUDown 產生)：

```shell
qnamaker create --in qnaKB.json --msbot | msbot connect qna --stdin
```

## <a name="7-publish-to-azure-from-the-cli"></a>7.從 CLI 發佈至 Azure

對 Bot 的原始程式碼進行變更後，您可以執行下列命令順暢地發佈變更：

```azurecli
az bot publish --name "my-bot-name" --resource-group "my-resource-group"
```

## <a name="references"></a>參考
- [BotBuilder 工具原始程式碼](https://github.com/Microsoft/botbuilder-tools)
- [MSBot](https://github.com/Microsoft/botbuilder-tools/tree/master/MSBot)
- [ChatDown](https://github.com/Microsoft/botbuilder-tools/tree/master/Chatdown)
- [LUDown](https://github.com/Microsoft/botbuilder-tools/tree/master/ludown)
- [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest)

