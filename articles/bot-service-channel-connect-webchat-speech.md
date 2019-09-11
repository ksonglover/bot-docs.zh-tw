---
title: 啟用網路聊天中的語音 | Microsoft Docs
description: 了解如何針對連線到 Web 聊天頻道的 Bot，啟用 Web 聊天控制項中的語音。
keywords: speech, web chat, voice, microphone, audio, 語音, Web 聊天, 麥克風, 音訊
author: DeniseMak
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: b83dff7969c58451e5752938f74b682b2163c49d
ms.sourcegitcommit: a6d02ec4738e7fc90b7108934740e9077667f3c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/04/2019
ms.locfileid: "70298198"
---
# <a name="enable-speech-in-web-chat"></a>啟用網路聊天中的語音
您可以啟用 Web 聊天控制項中的語音介面。 使用者可透過使用 Web 聊天控制項中的麥克風，與語音介面進行互動。

![Web 聊天語音範例](~/media/bot-service-channel-webchat/webchat-sample-speech.png)

如果使用者輸入而不是說出回應，則 Web 聊天將關閉語音功能，並且 Bot 僅提供文字回應而不是大聲朗讀。 若要重新啟用語音回應，使用者可以在下一次使用麥克風回應 Bot。 如果麥克風正在接受輸入，則它會顯示暗色或填滿。 如果它顯示為灰色，則使用者可按一下它加以啟用。

## <a name="prerequisites"></a>必要條件

  在執行範例之前，您需要針對要使用 Web 聊天控制項執行之 Bot 提供直接線路祕密或驗證權杖。 
  * 如需取得與 Bot 相關聯之直接線路祕密的相關資訊，請參閱[將機器人連線至直接線路](bot-service-channel-connect-directline.md)。
  * 如需交換權杖祕密的詳細資訊，請參閱[產生直接線路權杖](rest-api/bot-framework-rest-direct-line-3-0-authentication.md)。

## <a name="customizing-web-chat-for-speech"></a>自訂 Web 聊天的語音
若要在 Web 聊天中啟用語音功能，您需要自訂叫用 Web 聊天控制項的 JavaScript 程式碼。 您可以使用下列步驟，在本機嘗試啟用語音的 Web 聊天。

1. 下載[範例 index.html](https://aka.ms/web-chat-speech-sample)。 <!-- this aka.ms link needs to be updated if the sample location changes -->
2. 根據要新增的語音支援類型，編輯 `index.html` 中的程式碼。 [啟用語音服務](#enable-speech-services)中描述了語音實作的類型。 
3. 啟動 Web 伺服器。 一種方法是在 Node.js 命令提示字元中使用 `npm http-server`。

   * 若要全域安裝 `http-server` 以便可以從命令列執行它，請執行下列命令：

     ```
     npm install http-server -g
     ```

   * 若要啟動使用連接埠 8000 的 Web 伺服器，請從包含 `index.html` 的目錄執行下列命令：

     ```
     http-server -p 8000
     ```
4. 將您的瀏覽器目標設為 `http://localhost:8000/samples?parameters`。 例如，`http://localhost:8000/samples?s=YOURDIRECTLINESECRET` 使用直接線路祕密叫用 Bot。 下表說明可以在查詢字串中設定的參數：

   | 參數 | 說明 |
   |-----------|-------------|
   | s | 直接線路祕密。 如需取得直接線路祕密的相關資訊，請參閱[將機器人連線至直接線路](bot-service-channel-connect-directline.md)。 |
   | t | 直接線路權杖。 如需如何產生此權杖的相關資訊，請參閱[產生直接線路權杖](rest-api/bot-framework-rest-direct-line-3-0-authentication.md)。 |
   | 網域 | 選用。 替代直接線路端點的 URL。  |
   | WebSocket | 選用。 設定為 'true' 以使用 WebSocket 來接收訊息。 預設值為 `false`。 |
   | userid | 選用。 Bot 使用者的識別碼。  |
   | username | 選用。 Bot 使用者的使用者名稱。  |
   | botid | 選用。 Bot 的識別碼。 |
   | botname | 選用。 Bot 的名稱。 |


## <a name="enable-speech-services"></a>啟用語音服務
自訂可讓您以下列任何方式新增語音功能：

* **瀏覽器提供的語音** - 使用網頁瀏覽器內建的語音功能。 目前，此功能僅提供 Chrome 瀏覽器使用。
<!--* **Use Bing Speech service** - You can use the Bing Speech service to provide speech recognition and synthesis. This way of access speech functionality is supported by a variety of browsers. In this case, the processing is done on a server instead of on the browser.-->
* **建立自訂語音服務** - 您可以建立自己的自訂語音識別和語音合成元件。

### <a name="browser-provided-speech"></a>瀏覽器提供的語音

下列程式碼具現化瀏覽器隨附的語音辨識器和語音合成元件。 並非所有瀏覽器都支援這種新增語音的方法。 

> [!NOTE] 
> Google Chrome 支援瀏覽器語音辨識器。 但是，在下列情況中，Chrome 可能會封鎖麥克風：
> * 如果包含 Web 聊天之頁面的 URL 以 `http://` 開頭而不是 `https://`。
> * 如果 URL 是使用 `file://` 通訊協定而不是 `http://localhost:8000` 的本機檔案。

[!code-js[Specify speech options to use in-browser speech (JavaScript)](./includes/code/bot-service-channel-connect-webchat-speech.js#BrowserSpeech)]

<!--### Bing Speech service

The following code instantiates speech recognizer and speech synthesis components that use the Bing Speech service. The recognition and generation of speech is performed on the server. This mechanism is supported in multiple browsers. 

> [!TIP]
> You can use speech recognition priming to improve your bot's speech recognition accuracy if you use the Bing Speech service. For more information, check out the [Speech Support in Bot Framework](https://blog.botframework.com/2017/06/26/Speech-To-Text) blog post.

[!code-js[Specify speech options to use the Bing Speech API (JavaScript)](./includes/code/bot-service-channel-connect-webchat-speech.js#BingSpeech)]

#### Use the Bing Speech service with a token

You also have the option to enable Cognitive Services speech recognition using a token. The token is generated in a secure back end using your API key.

The following example code shows how the token fetch is done from a secure back end to avoid exposing the API key.

[!code-js[Fetch a token to use with the Bing Speech API (JavaScript)](./includes/code/bot-service-channel-connect-webchat-speech.js#FetchToken)]
-->
### <a name="custom-speech-service"></a>自訂語音服務

您也可以提供您自己的自訂語音辨識 (實作 ISpeechRecognizer) 或語音合成 (實作 ISpeechSynthesis)。 

[!code-js[Fetch a token to use with a custom speech service (JavaScript)](./includes/code/bot-service-channel-connect-webchat-speech.js#CustomSpeechService)]

### <a name="pass-the-speech-options-to-web-chat"></a>將語音選項傳遞至 Web 聊天

下列程式碼會將語音選項傳遞至 Web 聊天控制項：

[!code-js[Pass speech options to Web Chat (JavaScript)](./includes/code/bot-service-channel-connect-webchat-speech.js#PassSpeechOptionsToWebChat)]

## <a name="next-steps"></a>後續步驟
現在您可以透過 Web 聊天啟用語音互動，了解您的 Bot 如何建構語音訊息並調整麥克風的狀態：
* [將語音新增至訊息 (C#)](dotnet/bot-builder-dotnet-text-to-speech.md)
* [將語音新增至訊息 (Node.js)](nodejs/bot-builder-nodejs-text-to-speech.md)

## <a name="additional-resources"></a>其他資源

* 您可以在 GitHub 上[下載 Web 聊天控制項的原始程式碼](https://github.com/Microsoft/BotFramework-WebChat)。
* [Bing 語音 API 文件](https://docs.microsoft.com/azure/cognitive-services/speech/home)提供有關 Bing 語音 API 的詳細資訊。

