---
title: 使用 Botbuilder 範本來建立 Bot
description: Botbuilder 工具可讓您直接從命令列管理您的 Bot 資源
keywords: botbuilder 範本, node.js, python, java, .net, ludown, yeoman
author: matthewshim-ms
ms.author: v-shimma
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 04/25/2018
ms.openlocfilehash: 7cf05b3396099f1c65fce7abbceb143a3ad43e9a
ms.sourcegitcommit: 67445b42796d90661afc643c6bb6533e9a662cbc
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/06/2018
ms.locfileid: "39574584"
---
# <a name="create-bots-with-botbuilder-templates"></a>使用 Botbuilder 範本來建立 Bot

現在可使用範本在每個 Botbuilder SDK 平台上建立 Bot： 

- Node.js
- Python
- Java
- .NET

## <a name="prerequisites"></a>必要條件

Node.js、Java 和 Python 範本全都會建立簡單的回應 Bot，且全都可透過 [Yeoman](http://yeoman.io/) 提供使用。

- [Node.js](https://nodejs.org/en/) 8.5 版或更新版本
- [Yeoman](http://yeoman.io/)

如果您尚未安裝 Yeoman，必須全域安裝。

```shell
npm install -g yo
```

## <a name="nodejs-python-java-templates"></a>Node.js, Python, Java 範本
從命令列中，瀏覽到您所選擇的新資料夾。 Botbuilder Node.js 範本有兩個可用的版本，分別適用於 **v3** 和 **v4** 版本的 SDK。 只有 v4 中才有提供 Python 和 Java 範本。 

若要安裝 **Node.js v3 Botbuilder** 範本產生器：

```shell
npm install generator-botbuilder
```

若要安裝 **Node.js v4 Botbuilder** 範本產生器：
```shell
npm install generator-botbuilder@preview
```

**只有 v4 中才有提供** Python 和 Java 範本。 

針對 **Python**：
```shell
npm install generator-botbuilder-python
```

針對 **Java**：
```shell
npm install generator-botbuilder-java
```

安裝產生器之後，您只要在 CLI 中執行 **yo** 命令，即可檢視 Yeoman 中可用的產生器清單。

![Yeoman 介面](media/botbuilder-templates/yeoman-generator-botbuilder.png)

切換至您選擇的工作目錄，並選取要使用的產生器。 系統會提示您建立 Bot 的各種選項，例如名稱和描述。 完成所有提示之後，您的回應 Bot 範本會建立在相同的工作資料夾中。

![Node.js Yeoman 範本](media/botbuilder-templates/new-template-node.png)


## <a name="net"></a>.NET

有兩個範本可供 .NET 使用，分別適用於 **v3** 和 **v4** 版本的 SDK。 這兩者都可用來作為 [VSIX](https://docs.microsoft.com/en-us/visualstudio/extensibility/anatomy-of-a-vsix-package) 套件，若要下載，請按下列 [Visual Studio Marketplace](https://marketplace.visualstudio.com/) 上的其中一個連結。

- [BotBuilder V3 範本](https://marketplace.visualstudio.com/items?itemName=BotBuilder.BotBuilderV3)
- [BotBuilder V4 範本](https://aka.ms/Ylcwxk)

#### <a name="prerequisites"></a>必要條件

- [Visual Studio 2015 或更新版本](https://www.visualstudio.com/downloads/)
- [Azure 帳戶](https://azure.microsoft.com/en-us/free/)

### <a name="install-the-templates"></a>安裝範本

從已儲存的目錄中，只要開啟 VSIX 套件，Bot 建立器範本就會安裝到 Visual Studio，並在您下次開啟時可供使用。

![VSIX 套件](media/botbuilder-templates/botbuilder-vsix-template.png)

若要使用範本建立新的 Bot 專案，只要開啟 Visual Studio，然後選取 [檔案] > [新增] > [專案]，並從 Visual C# 中，選取 [Bot Framework] > [簡單的回應 Bot 應用程式] 即可。 這會在本機新建回應 Bot 專案，您可以視需要加以編輯。 

![.NET Bot 範本](media/botbuilder-templates/new-template-dotnet.png)

## <a name="store-your-bot-information-with-msbot"></a>使用 MSBot 儲存您的 Bot 資訊

新的 [MSBot](https://github.com/Microsoft/botbuilder-tools/tree/master/MSBot) (英文) 工具可讓您建立 **.bot** 檔案，會儲存您 Bot 所取用不同服務的相關中繼資料，全都放在一個位置。 此檔案也可讓您的 Bot 從 CLI 連線到這些服務。 此工具會以 npm 模組的形式提供，若要安裝請執行：

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

| 服務 | 說明 |
| ------ | ----------- |
| azure  |將您的 Bot 連線到 Azure Bot Service 註冊|
|localhost| 將您的 Bot 連線到 localhost 端點|
|luis     | 將您的 Bot 連線到 LUIS 應用程式 |
| qna     |將您的 Bot 連線到 QnA 知識庫|
|說明 [cmd]  |顯示 [cmd] 的說明|

### <a name="connect-your-bot-to-abs-with-the-bot-file"></a>使用 .bot 檔案將您的 Bot 連線到 ABS

安裝 MSBot 工具後，您可以執行 az bot **show** 命令，輕鬆地將 Bot 連線到 Azure Bot Service 中現有的資源群組。 

```azurecli
az bot show -n <botname> -g <resourcegroup> --msbot | msbot connect azure --stdin
```

這會從目標資源群組中取得目前的端點、MSA appID 和密碼，並據此更新您 .bot 檔案中的資訊。 


## <a name="manage-update-or-create-luis-and-qna-services-with--new-botbuilder-tools"></a>使用新的 Botbuilder 工具來管理、更新或建立 LUIS 和 QnA 服務

[Bot 建立器工具](https://github.com/microsoft/botbuilder-tools)是新的工具組，可讓您直接從命令列管理 bot 資源並與其進行互動。 

>[!TIP]
> 每個 Bot 建立器工具都包含全域的 help 命令，可輸入 **-h** 或 **--help**，從命令列中加以存取。 您可隨時從任何動作中使用此命令，將提供可用選項與其描述的實用示範 

### <a name="ludown"></a>LUDown
[LUDown](https://github.com/Microsoft/botbuilder-tools/tree/master/Ludown) (英文) 可讓您使用 **.lu** 檔案，來描述並建立功能強大的語言元件。 新的 .lu 檔案是一種 Markdown 格式，LUDown 工具會加以取用，並輸出目標服務特定的 .json 檔案。 目前，您可以使用 .lu 檔案來建立新的 [LUIS](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/luis-get-started-create-app) (英文) 應用程式或 [QnA](https://qnamaker.ai/Documentation/CreateKb) (英文) 知識庫 (各自使用不同的格式)。 LUDown 會以 npm 模組的形式提供，並可全域安裝到您的電腦來使用：

```shell
npm install -g ludown
```
LUDown 工具可用來建立適用於 LUIS 和 QnA 的新 .json 模型。  


### <a name="creating-a-luis-application-with-ludown"></a>使用 LUDown 建立 LUIS 應用程式

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

### <a name="qna-pairs-with-ludown"></a>QnA 與 LUDown 組合

.lu 檔案格式也會使用下列標記法支援 QnA 組合： 

```LUDown
> comment
### ? question ?
  ```markdown
    answer
  ```

LUDown 工具會自動將問題和解答分開到 qnamaker JSON 檔案，以便您可用來建立您的新 [QnaMaker.ai](http://qnamaker.ai) (英文) 知識庫。

```LUDown
### ? How do I change the default message for QnA Maker?
  ```markdown
  You can change the default message if you use the QnAMakerDialog. 
  See [this link](https://docs.botframework.com/en-us/azure-bot-service/templates/qnamaker/#navtitle) for details. 
  ```

您也可以將多個問題新增到相同的答案，只需針對單一解答新增問題變化的新行即可。 

```LUDown
### ? What is your name?
- What should I call you?
  ```markdown
    I'm the echoBot! Nice to meet you.
  ```

### <a name="generating-json-models-with-ludown"></a>使用 LUDown 產生 .json 模型

您以 .lu 格式定義 LUIS 或 QnA 語言元件之後，可以發行至 LUIS.json、QnA.json 或 QnA.tsv 檔案。 當執行時，LUDown 工具會在要剖析的相同工作目錄內尋找任何 .lu 檔案。 因為 LUDown 工具可以使用 .lu 檔案將目標鎖定為 LUIS 或 QnA，所以我們只需使用 **<luFile> 中的一般命令 ludown parse <Service> --**，指定所要產生的語言服務即可。 

在我們的範例工作目錄中，有兩個要剖析的 .lu 檔案，用來建立 LUIS 模型的 '1.lu'，以及用來建立 QnA 知識庫的 'qna1.lu'。

#### <a name="generate-luis-json-models"></a>產生 LUIS.json 模型

若要使用 LUDown 產生 LUIS 模型，只需在您目前的工作目錄中輸入下列內容：

```shell
ludown parse ToLuis --in <luFile> 
```

#### <a name="generate-qna-knowledge-base-json"></a>產生 QnA 知識庫 .json

同樣地，若要建立 QnA 知識庫，您只需要變更剖析目標。 

```shell
ludown parse ToQna --in <luFile> 
```

LUIS 和 QnA 可透過其個別的入口網站，或透過新的 CLI 工具來取用所產生的 JSON 檔案。 

## <a name="connect-to-luis-an-qna-maker-services-from-the-cli"></a>從 CLI 連線到 LUIS QnA Maker 服務

### <a name="connect-to-luis-from-the-cli"></a>從 CLI 連線到 LUIS 

新工具組中所包含的是 [LUIS 擴充功能](https://github.com/Microsoft/botbuilder-tools/tree/master/LUIS)https://github.com/Microsoft/botbuilder-tools/tree/master/LUIS，可讓您獨立管理您的 LUIS 資源。 它會以 npm 模組的形式提供，可供您下載：

```shell
npm install -g luis-apis
```
LUIS 工具從 CLI 中的基本命令使用方式為：

```shell
luis <action> <resource> <args...>
```
若要將您的 Bot 連線至 LUIS，您必須建立 **.luisrc** 檔案。 當您的應用程式進行輸出呼叫時，這個組態檔可將您的 LUIS appID 和密碼佈建到服務端點。 您可以執行 **luis init** 來建立這個檔案，如下所示：

```shell
luis init
```
此工具在產生檔案之前，系統會提示您在終端機中輸入 LUIS 製作金鑰、區域和 appID。  

![LUIS init](media/bot-builder-tools/luis-init.png) 


產生這個檔案之後，您的應用程式將能夠從 CLI 使用下列命令，來取用 LUIS .json 檔案 (從 LUDown 產生)。

```shell
luis import application --in luis-app.json | msbot connect luis --stdin
```

### <a name="connect-to-qna-from-the-cli"></a>從 CLI 連線到 QnA

新工具組中所包含的是 [QnA 擴充功能](https://github.com/Microsoft/botbuilder-tools/tree/master/QnAMaker) (英文)，可讓您獨立管理您的 LUIS 資源。 它會以 npm 模組的形式提供，可供您下載。

```shell
npm install -g qnamaker
```
透過 QnA Maker 工具，您可以建立、更新、發佈、刪除及訓練您的知識庫。 若要開始，您必須建立 **.qnamakerrc** 檔案，才能啟用您服務的端點。 您可以執行 **qnamaker init** 並遵循提示，輕鬆地建立這個檔案，並佈建您的 QnA Maker 知識庫識別碼。 

```shell
qnamaker init 
```
![QnaMaker init](media/bot-builder-tools/qnamaker-init.png)

一旦產生 .qnamakerrc 檔案後，您現在可以連線到 QnA 知識庫，使用下列命令來取用知識庫 .json/.tsv 檔案 (從 LUDown 產生)。

```shell
qnamaker create --in qnaKB.json --msbot | msbot connect qna --stdin
```

## <a name="references"></a>參考
- [BotBuilder 工具原始程式碼](https://github.com/Microsoft/botbuilder-tools)
- [MSBot](https://github.com/Microsoft/botbuilder-tools/tree/master/MSBot)
- [ChatDown](https://github.com/Microsoft/botbuilder-tools/tree/master/Chatdown)
- [LUDown](https://github.com/Microsoft/botbuilder-tools/tree/master/Ludown)
- [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest)
