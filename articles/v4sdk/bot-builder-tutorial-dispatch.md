---
title: 使用多個 LUIS 和 QnA 模型 | Microsoft Docs
description: 了解如何在 Bot 中使用 LUIS 和 QnA Maker。
keywords: Luis, QnA, 分派工具, 多種服務, 路由意圖
author: diberry
ms.author: diberry
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 05/23/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: c72f978e927f05f430ec94cf747016f4ebc15c5d
ms.sourcegitcommit: 0e6c49964b96c1ac8485ba7afe0daae04b671138
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/01/2019
ms.locfileid: "67492004"
---
# <a name="use-multiple-luis-and-qna-models"></a>使用多個 LUIS 和 QnA 模型

[!INCLUDE[applies-to](../includes/applies-to.md)]

如果聊天機器人使用多個 LUIS 模型和 QnA Maker 知識庫 (KB)，您可以使用分派工具來判斷哪一個 LUIS 模型或 QnA Maker 知識庫最符合使用者輸入。 分派工具執行此操作的方式是建立單一 LUIS 應用程式，以將使用者輸入路由至正確的模型。 如需有關分派工具的詳細資訊 (包括 CLI 命令)，請參閱[讀我檔案][dispatch-readme]。

## <a name="prerequisites"></a>必要條件
- [聊天機器人基本概念](bot-builder-basics.md)、[LUIS][howto-luis], and [QnA Maker][howto-qna] 的知識。 
- [分派工具](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/Dispatch)
- 從 [C# 範例][cs-sample] or [JS Sample][js-sample] 程式碼存放庫取得一份**採用分派的 NLP**。
- 用來發佈 LUIS 應用程式的 [luis.ai](https://www.luis.ai/) 帳戶。
- 用來發佈 QnA 知識庫的 [QnA Maker](https://www.qnamaker.ai/) 帳戶。

## <a name="about-this-sample"></a>關於此範例

此範例會以一組預先定義的 LUIS 和 QnA Maker 應用程式作為基礎。

## <a name="ctabcs"></a>[C#](#tab/cs)

![程式碼範例的邏輯流程](./media/tutorial-dispatch/dispatch-logic-flow.png)

系統會針對每個收到的使用者輸入呼叫 `OnMessageActivityAsync`。 此模組會尋找評分最高的使用者意圖，並將該結果傳遞至 `DispatchToTopIntentAsync`。 接著，DispatchToTopIntentAsync 會呼叫適當的應用程式處理常式

- `ProcessSampleQnAAsync` - 適用於 Bot 常見問題集的問題。
- `ProcessWeatherAsync` - 適用於天氣查詢。
- `ProcessHomeAutomationAsync` - 適用於家用光源命令。

## <a name="javascripttabjs"></a>[JavaScript](#tab/js)

![程式碼範例的邏輯流程](./media/tutorial-dispatch/dispatch-logic-flow-js.png)

系統會針對每個收到的使用者輸入呼叫 `onMessage`。 此模組會尋找評分最高的使用者意圖，並將該結果傳遞至 `dispatchToTopIntentAsync`。 接著，dispatchToTopIntentAsync 會呼叫適當的應用程式處理常式

- `processSampleQnA` - 適用於 Bot 常見問題集的問題。
- `processWeather` - 適用於天氣查詢。
- `processHomeAutomation` - 適用於家用光源命令。

---

處理常式會呼叫 LUIS 或 QnA Maker 服務，並將產生的結果傳回給使用者。

## <a name="create-luis-apps-and-qna-knowledge-base"></a>建立 LUIS 應用程式和 QnA 知識庫
建立分派模型之前，您必須先建立並發佈 LUIS 應用程式和 QnA 知識庫。 在本文中，我們將發佈下列模型，這些模型包含在_採用分派的 NLP_ 範例的 `\CognitiveModels` 資料夾中： 

| Name | 說明 |
|------|------|
| HomeAutomation | 使用相關聯的實體資料，辨識住家自動化意圖的 LUIS 應用程式。|
| Weather | 使用位置資料辨識天氣相關意圖的 LUIS 應用程式。|
| QnAMaker  | 提供聊天機器人相關簡單問題解答的 QnA Maker 知識庫。 |

### <a name="create-luis-apps"></a>建立 LUIS 應用程式
1. 登入 [LUIS Web 入口網站](https://www.luis.ai/)。 在 [我的應用程式]  區段之下，選取 [匯入新的應用程式]  索引標籤。 下列對話方塊隨即顯示：

    ![匯入 LUIS json 檔案](./media/tutorial-dispatch/import-new-luis-app.png)

2. 選取 [選擇應用程式檔案]  按鈕，瀏覽至您範例程式碼的 CognitiveModel 資料夾，然後選取 'HomeAutomation.json' 檔案。 將選用名稱欄位保留空白。 

3. 選取 [完成]  。

4. 在 LUIS 開啟您的住家自動化應用程式後，請選取 [訓練]  按鈕。 這會使用您剛使用 'home-automation.json' 檔案匯入的語句集合，訓練應用程式。

5. 訓練完成時，請選取 [發佈]  按鈕。 下列對話方塊隨即顯示：

    ![發佈 LUIS 應用程式](./media/tutorial-dispatch/publish-luis-app.png)

6. 選擇 [生產] 環境，然後選取 [發佈]  按鈕。

7. 開啟已發佈的新 LUIS 應用程式後，選取 [管理]  索引標籤。在 [應用程式資訊] 頁面中，將 `Application ID` 的值記錄為 "_app-id-for-app_"，並將 `Display name` 的值記錄為 "_name-of-app_"。 在 [金鑰和端點] 頁面中，將 `Authoring Key` 的值記錄為 "_your-luis-authoring-key_"，並將 `Region` 記錄為 "_your-region_"。 稍後在您的 'appsetting.json' 檔案中會用到這些值。

8. 完成後，針對 'Weather.json' 檔案重複上述步驟，以_訓練_和_發佈_您的 LUIS 氣象應用程式和 LUIS 分派應用程式。

### <a name="create-qna-maker-knowledge-base"></a>建立 QnA Maker 知識庫

設定 QnA Maker 知識庫的第一個步驟是在 Azure 中設定 QnA Maker 服務。 若要這麼做，請遵循[這裡](https://aka.ms/create-qna-maker)的逐步指示。

在 Azure 中建立 QnA Maker 服務之後，您需要記錄針對 QnA Maker 服務提供的認知服務「金鑰 1」  。 將 QnA Maker 應用程式新增至分派應用程式時，此金鑰會當作 \<azure-qna-service-key1> 使用。 下列步驟為您提供此金鑰：
    
![選取認知服務](./media/tutorial-dispatch/select-qna-cognitive-service.png)

1. 在 Azure 入口網站中選取您的 QnA Maker 認知服務。

    ![選取認知服務金鑰](./media/tutorial-dispatch/select-cognitive-service-keys.png)

1. 選取左側功能表上 [資源管理]  區段之下的 [金鑰] 圖示。

    ![選取認知服務金鑰 1](./media/tutorial-dispatch/select-cognitive-service-key1.png)

1. 將「金鑰 1」  的值複製到剪貼簿並儲存在本機。 稍後將 QnA Maker 應用程式新增至分派應用程式時，此金鑰會用於 (-k) 金鑰值\<azure-qna-service-key1>。

1. 現在登入 [QnAMaker 入口網站](https://qnamaker.ai)。 

1. 在步驟 2 中，選取下列項目：

    * 您的 Azure AD 帳戶。
    * 您的 Azure 訂用帳戶名稱。
    * 針對您的 QnA Maker 服務建立的名稱。 (如果您的 Azure QnA 服務並未一開始就出現在此下拉式清單中，請嘗試重新整理頁面。)

    ![建立 QnA 步驟 2](./media/tutorial-dispatch/create-qna-step-2.png) 
     

1. 在步驟 3 中，提供您的 QnA Maker 知識庫名稱。 在此範例中，請使用 'sample-qna' 這個名稱。

    ![建立 QnA 步驟 3](./media/tutorial-dispatch/create-qna-step-3.png)

1. 在步驟 4 中，選取 [+ 新增檔案]  選項，瀏覽至您範例程式碼的 CognitiveModel 資料夾，然後選取 'QnAMaker.tsv' 檔案。 有其他選取項目可將「閒聊」  個性新增至您的知識庫，但我們的範例不包含此選項。

    ![建立 QnA 步驟 4](./media/tutorial-dispatch/create-qna-step-4.png)

1. 在步驟 5 中，選取 [建立知識庫]  。

1. 從您上傳的檔案中建立知識庫後，選取 [儲存並訓練]  ，然後在完成時選取 [發佈]  索引標籤，並發佈您的應用程式。

1. 發佈 QnA Maker 應用程式後，選取 [設定]  索引標籤，然後捲動至 [部署詳細資料]。 記錄以下來自 _Postman_ 範例 HTTP 要求的值。

    ```text
    POST /knowledge bases/<knowledge-base-id>/generateAnswer
    Host: <your-hostname>  // NOTE - this is a URL.
    Authorization: EndpointKey <qna-maker-resource-key>
    ```
    
    您主機名稱的完整 URL 字串看起來會類似 "https://< >.azure.net/qnamaker"。 稍後在您的 `appsettings.json` 或 `.env` 檔案中會用到這些值。

## <a name="dispatch-app-needs-read-access-to-existing-apps"></a>分派應用程式需要現有應用程式的讀取權限

分派工具需要可讀取現有 LUIS 和 QnA Maker 應用程式的撰寫權限，才能建立新的父項 LUIS 應用程式來分派至 LUIS 和 QnA Maker 應用程式。 在提供此存取權時也會隨附應用程式識別碼和撰寫金鑰。 您需要這兩個 LUIS 應用程式和 QnA Maker 應用程式各自的識別碼和金鑰。

|應用程式|資訊的位置|
|--|--|
|LUIS|應用程式識別碼 - 可於 [LUIS 入口網站](https://www.luis.ai)中找到每個應用程式的識別碼，路徑為 [管理]-> [應用程式資訊]<br>撰寫金鑰 - 可於 LUIS 入口網站右上角，選取您自己的 [使用者] 和 [設定] 來找到。|
|QnA Maker| 應用程式識別碼 - 可於發佈應用程式後在 [QnA Maker 入口網站](https://http://qnamaker.ai)的 [設定] 頁面上找到。 這是在知識庫後面 POST 命令的第一個部分中所找到的識別碼。 要到哪裡尋找應用程式識別碼的範例是 `POST /knowledgebases/{APP-ID}/generateAnswer`。<br>撰寫金鑰 - 可在 Azure 入口網站中找到，若為 QnA Maker 資源，則位於 [金鑰]  底下。 您只需要其中一個金鑰即可。|

## <a name="create-the-dispatch-model"></a>建立分派模型

分派工具的 CLI 介面會建立分派給正確 LUIS 或 QnA Maker 應用程式的模型。

1. 開啟命令提示字元或終端機視窗，並將目錄變更為 **CognitiveModels** 目錄
1. 請確定您有最新版的 npm 和分派工具。

    ```cmd
    npm i -g npm
    npm i -g botdispatch
    ```

1. 使用 `dispatch init` 來初始化分派模型的 `.dispatch` 檔案建立。 使用您可辨識的檔案名稱來建立此檔案。

    ```cmd
    dispatch init -n <filename-to-create> --luisAuthoringKey "<your-luis-authoring-key>" --luisAuthoringRegion <your-region>
    ```

1. 使用 `dispatch add` 將您的 LUIS 應用程式和 QnA Maker 知識庫新增至 `.dispatch` 檔案。

    ```cmd
    dispatch add -t luis -i "<app-id-for-weather-app>" -n "<name-of-weather-app>" -v <app-version-number> -k "<your-luis-authoring-key>" --intentName l_Weather
    dispatch add -t luis -i "<app-id-for-home-automation-app>" -n "<name-of-home-automation-app>" -v <app-version-number> -k "<your-luis-authoring-key>" --intentName l_HomeAutomation
    dispatch add -t qna -i "<knowledge-base-id>" -n "<knowledge-base-name>" -k "<azure-qna-service-key1>" --intentName q_sample-qna
    ```

1. 使用 `dispatch create` 從 `.dispatch` 檔案中產生分派模型。

    ```cmd
    dispatch create
    ```

1. 發佈剛剛建立的分派 LUIS 應用程式。

## <a name="use-the-dispatch-luis-app"></a>使用分派 LUIS 應用程式

產生的 LUIS 應用程式會針對每個子應用程式和知識庫定義意圖，也會在語句沒有適當的相符項目時定義_無_意圖。

- `l_HomeAutomation`
- `l_Weather`
- `None`
- `q_sample-qna`

這些服務必須以正確的名稱發佈，才能讓聊天機器人正常運作。 Bot 需要已發佈服務的相關資訊，好讓其可以存取這些服務。

聊天機器人需要準備查詢預測端點供三個 LUIS 應用程式 (分派、天氣和家庭自動化) 和一個 QnA Maker 知識庫使用。 請使用下表來尋找端點金鑰：

|應用程式|查詢端點金鑰位置|
|--|--|
|LUIS|在 LUIS 入口網站中，請針對每個 LUIS 應用程式在 [管理] 區段中選取 [金鑰和端點設定]  ，以尋找與每個應用程式相關聯的金鑰。 如果您要遵循本教學課程，則端點金鑰就是和 `<your-luis-authoring-key>` 相同的金鑰。 撰寫金鑰允許 1000 次的端點叫用，然後就會過期。|
|QnA Maker|在 QnA Maker 入口網站中，請針對知識庫在 [管理] 設定中，使用顯示在**授權**標頭 Postman 設定中的金鑰值，但不要 `EndpointKey ` 文字。|

這些值會用於 **appsettings.json** (若為 C#) 和 **.env** 檔案 (若為 javascript)。

## <a name="ctabcs"></a>[C#](#tab/cs)

### <a name="installing-packages"></a>安裝套件

第一次執行此應用程式之前，請先確認您已安裝數個 NuGet 套件：

**Microsoft.Bot.Builder**

**Microsoft.Bot.Builder.AI.Luis**

**Microsoft.Bot.Builder.AI.QnA**

### <a name="manually-update-your-appsettingsjson-file"></a>手動更新 appsettings.json 檔案

建立所有服務應用程式後，每個服務應用程式的資訊都必須新增至 'appsettings.json' 檔案。 初始的 [C# 範例][cs-sample]程式碼會包含空白的 appsettings.json 檔案：

**appsettings.json**  
[!code-json[AppSettings](~/../botbuilder-samples/samples/csharp_dotnetcore/14.nlp-with-dispatch/AppSettings.json?range=8-17)]

針對下列所示的每個實體，在這些指示中新增您稍早所記錄的值：

**appsettings.json**
```json
"MicrosoftAppId": "",
"MicrosoftAppPassword": "",
  
"QnAKnowledgebaseId": "<knowledge-base-id>",
"QnAEndpointKey": "<qna-maker-resource-key>",
"QnAEndpointHostName": "<your-hostname>",

"LuisAppId": "<app-id-for-dispatch-app>",
"LuisAPIKey": "<your-luis-endpoint-key>",
"LuisAPIHostName": "<your-dispatch-app-region>",
```
完成所有變更時，儲存這個檔案。

## <a name="javascripttabjs"></a>[JavaScript](#tab/js)

### <a name="installing-packages"></a>安裝套件

在第一次執行此應用程式之前，您必須安裝數個 npm 套件。

```powershell
npm install --save botbuilder
npm install --save botbuilder-ai
```
若要使用 .env 組態檔，您的 Bot 必須包含額外的套件：

```powershell
npm install --save dotenv
```

### <a name="manually-update-your-env-file"></a>手動更新 .env 檔案

建立所有服務應用程式後，每個服務應用程式的資訊都必須新增至 '.env' 檔案。 初始 [JavaScript 範例][js-sample]程式碼會包含空白的 .env 檔案。 

**.env**  
[!code-file[EmptyEnv](~/../botbuilder-samples/samples/javascript_nodejs/14.nlp-with-dispatch/.env?range=1-10)]

新增您的服務連線值，如下所示：

**.env**
```file
MicrosoftAppId=""
MicrosoftAppPassword=""

QnAKnowledgebaseId="<knowledge-base-id>"
QnAEndpointKey="<qna-maker-resource-key>"
QnAEndpointHostName="<your-hostname>"

LuisAppId=<app-id-for-dispatch-app>
LuisAPIKey=<your-luis-endpoint-key>
LuisAPIHostName=<your-dispatch-app-region>

```
所有變更皆就緒時，儲存這個檔案。

---

### <a name="connect-to-the-services-from-your-bot"></a>從 Bot 連線到服務

若要連線到分派、LUIS 和 QnA Maker 服務，聊天機器人會從設定檔案 (`appsettings.json` 或 `.env` 檔案) 中提取資訊。

## <a name="ctabcs"></a>[C#](#tab/cs)

在 **BotServices.cs** 中，_appsettings.json_ 組態檔中包含的資訊會用於將分派 Bot 連線至 `Dispatch` 和 `SampleQnA` 服務。 建構函式會使用您提供的值來連線到這些服務。

**BotServices.cs**  
[!code-csharp[ReadConfigurationInfo](~/../botbuilder-samples/samples/csharp_dotnetcore/14.nlp-with-dispatch/BotServices.cs?range=14-30)]

## <a name="javascripttabjs"></a>[JavaScript](#tab/js)

在 **dispatchBot.js** 中， _.env_ 組態檔中包含的資訊會用來將分派 Bot 連線至 _LuisRecognizer(dispatch)_ 和 _QnAMaker_ 服務。 建構函式會使用您提供的值來連線到這些服務。

**dispatchBot.js**  
[!code-javascript[ReadConfigurationInfo](~/../botbuilder-samples/samples/javascript_nodejs/14.nlp-with-dispatch/bots/dispatchBot.js?range=18-31)]

---

### <a name="call-the-services-from-your-bot"></a>從 Bot 呼叫服務

對於每個來自您使用者的輸入，Bot 邏輯會針對結合的分派模型來檢查使用者輸入、尋找最高分的傳回意圖，並使用該資訊來為輸入呼叫適當的服務。

## <a name="ctabcs"></a>[C#](#tab/cs)

在 **DispatchBot.cs** 檔案中，每當呼叫 `OnMessageActivityAsync` 方法時，我們就會針對分派模型來檢查傳入的使用者訊息。 接著，我們會將分派模型的 `topIntent` 和 `recognizerResult` 傳遞至正確的方法，以呼叫服務並傳回結果。

**DispatchBot.cs**  
[!code-csharp[OnMessageActivity](~/../botbuilder-samples/samples/csharp_dotnetcore/14.nlp-with-dispatch/bots/DispatchBot.cs?range=26-36)]

## <a name="javascripttabjs"></a>[JavaScript](#tab/js)

在 **dispatchBot.js** `onMessage` 方法中，我們會針對分派模型檢查使用者輸入訊息、尋找 _topIntent_，然後藉由呼叫 _dispatchToTopIntentAsync_ 來傳遞此意圖。

**dispatchBot.js**  

[!code-javascript[OnMessageActivity](~/../botbuilder-samples/samples/javascript_nodejs/14.nlp-with-dispatch/bots/dispatchBot.js?range=37-50)]

---

### <a name="work-with-the-recognition-results"></a>使用辨識結果

## <a name="ctabcs"></a>[C#](#tab/cs)

當模型產生結果時，它會指出哪一項服務最適合處理語句。 此 Bot 中的程式碼會將要求路由傳送至對應的服務，然後概述來自所呼叫服務的回應。 根據分派傳回的_意圖_，此程式碼會使用傳回的意圖來路由至正確的 LUIS 模型或 QnA 服務。

**DispatchBot.cs**  
[!code-csharp[DispatchToTop](~/../botbuilder-samples/samples/csharp_dotnetcore/14.nlp-with-dispatch/bots/DispatchBot.cs?range=51-69)]

如果叫用 `ProcessHomeAutomationAsync` 或 `ProcessWeatherAsync` 方法，則會從 _luisResult.ConnectedServiceResult_ 內的分派模型中傳遞結果。 接著，指定方法會提供顯示分派模型最高意圖的使用者意見反應，並且以排名列出所有偵測到的意圖和實體。

如果叫用 `q_sample-qna` 方法，該方法會使用 turnContext 中包含的使用者輸入，從知識庫產生答案，並將該結果顯示給使用者。

## <a name="javascripttabjs"></a>[JavaScript](#tab/js)

當模型產生結果時，它會指出哪一項服務最適合處理語句。 此範例中的程式碼會使用可辨識的 _topIntent_，示範如何將要求路由至相對應的服務。

**DispatchBot.cs**  
[!code-javascript[DispatchToTop](~/../botbuilder-samples/samples/javascript_nodejs/14.nlp-with-dispatch/bots/dispatchBot.js?range=67-83)]

如果叫用 `processHomeAutomation` 或 `processWeather` 方法，則會從 _recognizerResult.luisResult_ 內的分派模型中傳遞結果。 接著，指定方法會提供顯示分派模型最高意圖的使用者意見反應，並且以排名列出所有偵測到的意圖和實體。

如果叫用 `q_sample-qna` 方法，該方法會使用 turnContext 中包含的使用者輸入，從知識庫產生答案，並將該結果顯示給使用者。

---

> [!NOTE]
> 如果這是生產應用程式，就會用此應用程式來將選取的 LUIS 方法連線到其指定服務、傳入使用者輸入及處理傳回的 LUIS 意圖和實體資料。

## <a name="test-your-bot"></a>測試 Bot

1. 使用您的開發環境啟動範例程式碼。 在由您應用程式開啟的瀏覽器視窗中，記下網址列顯示的 localhost  位址："https://localhost:<Port_Number>"。 
1. 開啟 Bot Framework Emulator，然後選取 `Create a new bot configuration`。 `.bot` 檔案可讓您在聊天機器人模擬器中使用_偵測器_來查看 LUIS 和 QnA Maker 所傳回的 JSON。
1. 在 [新增聊天機器人組態]  對話方塊中，輸入聊天機器人名稱和端點 URL，例如 `http://localhost:3978/api/messages`。 將檔案儲存在聊天機器人程式碼範例專案的根目錄。
1. 開啟聊天機器人檔案，並為 LUIS 和 QnA Maker 應用程式新增區段。 使用[這個範例檔](https://github.com/microsoft/botbuilder-tools/blob/master/packages/MSBot/docs/sample-bot-file.json)作為設定範本。 儲存變更。
1. 在 [我的聊天機器人]  清單中選取聊天機器人名稱，以存取執行中的聊天機器人。 專為 Bot 所建置的服務涵蓋以下一些問題和命令，供您參考：

    - QnA Maker
      - `hi`、`good morning`
      - `what are you`、`what do you do`
    - LUIS (住家自動化)
      - `turn on bedroom light`
      - `turn off bedroom light`
      - `make some coffee`
    - LUIS (天氣)
      - `whats the weather in redmond washington`
      - `what's the forecast for london`
      - `show me the forecast for nebraska`

## <a name="dispatch-for-user-utterance-to-qna-maker"></a>將使用者語句分派至 QnA Maker

1. 在聊天機器人模擬器中，輸入 `hi` 文字並提交語句。 聊天機器人會提交此查詢來分派 LUIS 應用程式並取得回應，其內容會指出哪個子應用程式應該取得此語句以便進一步處理。 

1. 藉由在記錄中選取 `LUIS Trace` 行，便可以在聊天機器人模擬器中看到 LUIS 回應。 來自分派 LUIS 應用程式的 LUIS 結果會顯示在偵測器中。 

    ```json
    {
      "luisResponse": {
        "entities": [],
        "intents": [
          {
            "intent": "q_sample-qna",
            "score": 0.9489713
          },
          {
            "intent": "l_HomeAutomation",
            "score": 0.0612499453
          },
          {
            "intent": "None",
            "score": 0.008567564
          },
          {
            "intent": "l_Weather",
            "score": 0.0025761195
          }
        ],
        "query": "Hi",
        "topScoringIntent": {
          "intent": "q_sample-qna",
          "score": 0.9489713
        }
      }
    }
    ```
    
    因為語句 `hi` 是分派 LUIS 應用程式的 **q_sample-qna** 意圖一部分，且您已將其選取為 `topScoringIntent`，所以聊天機器人會提出第二個要求，但這一次會對 QnA Maker 應用程式來提出，且使用相同的語句。 

1. 在聊天機器人模擬器記錄中選取 `QnAMaker Trace` 行。 QnA Maker 結果會顯示在偵測器中。 

```json
{
    "questions": [
        "hi",
        "greetings",
        "good morning",
        "good evening"
    ],
    "answer": "Hello!",
    "score": 1,
    "id": 96,
    "source": "QnAMaker.tsv",
    "metadata": [],
    "context": {
        "isContextOnly": false,
        "prompts": []
    }
}
```

## <a name="resolving-incorrect-top-intent-from-dispatch"></a>解決來自分派、不正確的最上層意圖

當聊天機器人正在執行時，移除已分派應用程式之間的類似或重疊語句有可能會改善聊天機器人的效能。 例如，假設在 `Home Automation` LUIS 應用程式中，「開燈」這類要求對應至「TurnOnLights」意圖，但「為什麼無法打開燈？」要求則對應至 「無」意圖，因此系統便會將其傳遞至 QnA Maker。 這兩個語句對分派 LUIS 應用程式來說太接近，因此無法判斷正確的子應用程式是 LUIS 應用程式還是 QnA Maker 應用程式。

使用分派工具結合 LUIS 應用程式和 QnA Maker 應用程式時，您必須執行下列_其中一個_動作：

* 從 `Home Automation` 子 LUIS 應用程式移除 "None" 意圖，然後將該意圖中的語句新增至發送器應用程式中的 "None" 意圖。
* 在聊天機器人中新增邏輯，以將符合分派 LUIS 應用程式 "None" 意圖的訊息傳遞至 QnA Maker 服務。 請比較分派 LUIS 應用程式的分數與 QnA Maker 應用程式的分數。 使用最高的分數。 這可有效地從分派循環中移除 QnA Maker。 

上述兩個動作都會減少 Bot 以「找不到答案」訊息回應使用者的次數。

### <a name="to-update-or-create-a-new-luis-model"></a>更新或建立新的 LUIS 模型

此範例是以預先設定的 LUIS 模型為基礎。 在[這裡](https://aka.ms/create-luis-model#updating-your-cognitive-models)可以找到其他資訊，協助您更新此模型，或建立新的 LUIS 模型。

### <a name="to-delete-resources"></a>刪除資源

此範例會建立一些應用程式和資源，您可以使用下面所列的步驟將其刪除，但不得刪除「任何其他應用程式或服務」  所依賴的資源。

刪除 LUIS 資源：

1. 登入 [luis.ai](https://www.luis.ai) 入口網站。
1. 移至 [我的應用程式]  頁面。
1. 選取此範例所建立的應用程式。
   - `Home Automation`
   - `Weather`
   - `NLP-With-Dispatch-BotDispatch`
1. 按一下 [刪除]  ，然後按一下 [確定]  進行確認。

刪除 QnA Maker 資源：

1. 登入 [qnamaker.ai](https://www.qnamaker.ai/) 入口網站。
1. 移至 [我的知識庫]  頁面。
1. 按一下 `Sample QnA` 知識庫的刪除按鈕，然後按一下 [刪除]  進行確認。

### <a name="best-practice"></a>最佳做法

若要改善此範例中使用的服務，請參閱 [LUIS](https://docs.microsoft.com/azure/cognitive-services/luis/luis-concept-best-practices) 及 [QnA Maker](https://docs.microsoft.com/azure/cognitive-services/qnamaker/concepts/best-practices) 的最佳做法。


[howto-luis]: bot-builder-howto-v4-luis.md
[howto-qna]: bot-builder-howto-qna.md

[cs-sample]: https://aka.ms/dispatch-sample-cs
[js-sample]: https://aka.ms/dispatch-sample-js

[dispatch-readme]: https://aka.ms/botbuilder-tools-dispatch
