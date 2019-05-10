---
title: 使用 Skype 進行音訊通話 | Microsoft Docs
description: 了解如何使用適用於 .NET 的 Bot Framework SDK，透過 Skype 進行音訊通話。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 72b2c42acb4743c67c63f158fa37c2bdd0e09ab9
ms.sourcegitcommit: 980612a922b8290b2faadaca193496c4117e415a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/26/2019
ms.locfileid: "64563937"
---
# <a name="conduct-audio-calls-with-skype"></a>使用 Skype 進行音訊通話

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

[!INCLUDE [Introduction to conducting audio calls](../includes/snippet-audio-call-intro.md)]

支援音訊通話的 Bot 架構與一般 Bot 的架構非常類似。 下列程式碼範例說明如何使用適用於 .NET 的 Bot Framework SDK 啟用，透過 Skype 進行音訊通話的支援。 

## <a name="enable-support-for-audio-calls"></a>啟用音訊通話的支援

若要讓 Bot 支援音訊通話，請定義 `CallingController`。

```cs
[BotAuthentication]
[RoutePrefix("api/calling")]
public class CallingController : ApiController
{
    public CallingController() : base()
    {
        CallingConversation.RegisterCallingBot(callingBotService => new IVRBot(callingBotService));
    }

    [Route("callback")]
    public async Task<HttpResponseMessage> ProcessCallingEventAsync()
    {
        return await CallingConversation.SendAsync(this.Request, CallRequestType.CallingEvent);
    }

    [Route("call")]
    public async Task<HttpResponseMessage> ProcessIncomingCallAsync()
    {
        return await CallingConversation.SendAsync(this.Request, CallRequestType.IncomingCall);
    }
}
```

> [!NOTE]
> 除了支援音訊通話的 `CallingController` 以外，Bot 也可包含 `MessagesController` 以支援訊息。 同時使用這兩個選項，可讓使用者選擇其偏好的方式與 Bot 互動。 <!-- docs on MessagesController are where? -->

##  <a name="answer-the-call"></a>接聽來電

每當使用者用 Skype 起始與此 Bot 的通話時，就會執行 `ProcessIncomingCallAsync` 工作。
建構函式會註冊 `IVRBot` 類別，其中包含為 `incomingCallEvent` 預先定義的處理常式。

工作流程中的第一個動作應先決定 Bot 要接聽還是拒絕來電。 此工作流程會指示 Bot 接聽來電，然後播放歡迎訊息。 

```cs
private Task OnIncomingCallReceived(IncomingCallEvent incomingCallEvent)
{
    this.callStateMap[incomingCallEvent.IncomingCall.Id] = new CallState(incomingCallEvent.IncomingCall.Participants);

    incomingCallEvent.ResultingWorkflow.Actions = new List<ActionBase>
    {
        new Answer { OperationId = Guid.NewGuid().ToString() },
        GetPromptForText(WelcomeMessage)
    };

    return Task.FromResult(true);
}
```

## <a name="after-the-bot-answers"></a>在 Bot 接聽後

如果 Bot 接聽電話，工作流程中指定的後續動作會指示 **Skype Bot 通話平台**播放提示，錄製音訊、辨識語音，或從撥號鍵台收集數字。 工作流程的最後一個動作可能是結束通話。 

下列程式碼範例會定義，將在歡迎訊息完成後設定功能表的處理常式。

```cs
private Task OnPlayPromptCompleted(PlayPromptOutcomeEvent playPromptOutcomeEvent)
{
    var callState = this.callStateMap[playPromptOutcomeEvent.ConversationResult.Id];
    SetupInitialMenu(playPromptOutcomeEvent.ResultingWorkflow);
    return Task.FromResult(true);
}
```

`CreateIvrOptions` 方法定義會要向使用者顯示的功能表。

```cs
private static Recognize CreateIvrOptions(string textToBeRead, int numberOfOptions, bool includeBack)
{
    if (numberOfOptions > 9)
    {
        throw new Exception("too many options specified");
    }

    var choices = new List<RecognitionOption>();

    for (int i = 1; i <= numberOfOptions; i++)
    {
        choices.Add(new RecognitionOption { Name = Convert.ToString(i), DtmfVariation = (char)('0' + i) });
    }

    if (includeBack)
    {
        choices.Add(new RecognitionOption { Name = "#", DtmfVariation = '#' });
    }

    var recognize = new Recognize
    {
        OperationId = Guid.NewGuid().ToString(),
        PlayPrompt = GetPromptForText(textToBeRead),
        BargeInAllowed = true,
        Choices = choices
    };

    return recognize;
}
```

`RecognitionOption` 類別會定義回話和相對應的雙音多頻訊號 (DTMF) 變化。 DTMF 可讓使用者以撥號鍵台輸入對應的數字來應答，而無須實際發話。

`OnRecognizeCompleted` 方法會處理使用者的選取項目，而輸入參數 `recognizeOutcomeEvent` 會包含使用者選取項目的值。

```cs
private Task OnRecognizeCompleted(RecognizeOutcomeEvent recognizeOutcomeEvent)
{
    var callState = this.callStateMap[recognizeOutcomeEvent.ConversationResult.Id];
    ProcessMainMenuSelection(recognizeOutcomeEvent, callState);
    return Task.FromResult(true);
}
```

## <a name="support-natural-language"></a>支援自然語言
Bot 也可以設計成支援自然語言回應。 **Bing 語音 API** 可讓 Bot 辨識使用者實際回答的文字。

```cs
private async Task OnRecordCompleted(RecordOutcomeEvent recordOutcomeEvent)
{
    recordOutcomeEvent.ResultingWorkflow.Actions = new List<ActionBase>
    {
        GetPromptForText(EndingMessage),
        new Hangup { OperationId = Guid.NewGuid().ToString() }
    };

    // Convert the audio to text
    if (recordOutcomeEvent.RecordOutcome.Outcome == Outcome.Success)
    {
        var record = await recordOutcomeEvent.RecordedContent;
        string text = await this.GetTextFromAudioAsync(record);

        var callState = this.callStateMap[recordOutcomeEvent.ConversationResult.Id];

        await this.SendSTTResultToUser("We detected the following audio: " + text, callState.Participants);
    }

    recordOutcomeEvent.ResultingWorkflow.Links = null;
    this.callStateMap.Remove(recordOutcomeEvent.ConversationResult.Id);
}
```

## <a name="sample-code"></a>範例程式碼

如需完整範例以了解如何使用適用於 .NET 的 Bot Framework SDK 支援，透過 Skype 的音訊通話，請參閱 GitHub 中的 <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/skype-CallingBot" target="_blank">Skype 通話 Bot 範例</a>。

## <a name="additional-resources"></a>其他資源

- <a href="/dotnet/api/?view=botbuilder-3.11.0" target="_blank">適用於 .NET 的 Bot Framework SDK 參考</a>
- <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/skype-CallingBot" target="_blank">Skype 通話 Bot 範例 (GitHub)</a>
