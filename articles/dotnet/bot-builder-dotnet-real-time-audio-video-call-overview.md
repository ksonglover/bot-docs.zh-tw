---
title: 建置適用於 Skype 的即時媒體 Bot | Microsoft Docs
description: 了解如何建置 Bot，以便透過 Skype 進行即時音訊/視訊通話、使用適用於 .NET 的 Bot 建立器 SDK 和 Bot Builder-RealTimeMediaCalling SDK for .NET。
author: MalarGit
ms.author: malarch
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/17
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 3e990da2abcb63c695cc79d5d8d9af40d8966cfa
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/18/2018
ms.locfileid: "39299571"
---
# <a name="build-a-real-time-media-bot-for-skype"></a>建置適用於 Skype 的即時媒體 Bot

適用於 Bot 的即時媒體平台是一項進階功能，可以讓 Bot「逐框架」傳送及接收語音和視訊內容。 Bot 具有語音、視訊和畫面共用即時形式的「原始」存取權。 本文提供建置音訊/視訊通話 Bot 和存取即時形式的概觀。

在本文中，Bot 會以自我裝載 ASP.NET Web API 架構的 Web 角色或背景工作角色，在 Azure 雲端服務中執行。

> [!IMPORTANT]
> 此文內容都是初步內容；如需範例即時媒體 Bot 的完整程式碼，請參閱 <a href="https://github.com/Microsoft/BotBuilder-RealTimeMediaCalling">BotBuilder-RealTimeMediaCalling</a> GitHub 存放庫中的 Samples 資料夾。

## <a name="configure-the-service-hosting-the-real-time-media-bot"></a>設定裝載即時媒體 Bot 的服務

若要使用即時媒體平台，則必須具備下列服務組態。

* Bot 必須知道其服務的完整網域名稱 (FQDN)。 這不是由 Azure RoleEnvironment API 所提供；反而，FQDN 必須儲存在 Bot 的雲端服務組態中，並且在服務啟動期間讀取。

* Bot Service 必須具有由公認的憑證授權單位簽發。 憑證的指紋必須儲存在 Bot 的雲端服務組態中，並且在服務啟動期間讀取。

* 必須佈建公用<a href="/azure/cloud-services/cloud-services-enable-communication-role-instances#instance-input-endpoint">執行個體輸入點</a>。 這會將唯一的公用連接埠，指派給 Bot Service 中的每個虛擬機器 (VM) 執行個體。 即時媒體平台會使用此連接埠來與 Skype Calling Cloud 通訊。
  ```xml
  <InstanceInputEndpoint name="InstanceMediaControlEndpoint" protocol="tcp" localPort="20100">
    <AllocatePublicPortFrom>
    <FixedPortRange max="20200" min="20101" />
    </AllocatePublicPortFrom>
  </InstanceInputEndpoint>
  ```

  它也適合用來為通話相關回撥和通知，建立另一個執行個體輸入端點。 使用執行個體輸入端點，可確保回呼和通知會傳遞給服務部署中裝載通話即時媒體工作階段的相同 VM 執行個體。
  ```xml
  <InstanceInputEndpoint name="InstanceCallControlEndpoint" protocol="tcp" localPort="10100">
    <AllocatePublicPortFrom>
    <FixedPortRange max="10200" min="10101" />
    </AllocatePublicPortFrom>
  </InstanceInputEndpoint>
  ```

* 每個 VM 執行個體必須具有執行個體層級的公用 IP 位址 (ILPIP)。 在啟動期間，Bot 必須探索已指派給每個服務執行個體的 ILPIP 位址。 如需取得及設定 ILPIP 的詳細資訊，請參閱 <a href="/azure/virtual-network/virtual-networks-instance-level-public-ip">ILPIP</a>。
  ```xml
  <NetworkConfiguration>
  <AddressAssignments>
    <InstanceAddress roleName="WorkerRole">
    <PublicIPs>
        <PublicIP name="InstancePublicIP" domainNameLabel="InstancePublicIP" />
    </PublicIPs>
    </InstanceAddress>
  </AddressAssignments>
  </NetworkConfiguration>
  ```

* 在服務執行個體啟動期間，指令碼 `MediaPlatformStartupScript.bat` (當作 Nuget 套件的一部分提供) 必須以提升的權限作為啟動工作執行。 呼叫平台的初始化方法之前，必須完成指令碼執行。 

```xml
<Startup>
<Task commandLine="MediaPlatformStartupScript.bat" executionContext="elevated" taskType="simple" />      
</Startup> 
```

## <a name="initialize-the-media-platform-on-service-startup"></a>在服務啟動時初始化媒體平台

在服務執行個體啟動期間，必須初始化即時媒體平台。 Bot 在該執行個體上接受任何音訊/視訊 Skype 通話之前，這只能進行一次。 初始化媒體平台時，必須提供各種服務組態設定，包括服務的 FQDN、公用 ILPIP 位址、執行個體輸入端點連接埠值，以及 Bot 的 **Microsoft 應用程式識別碼**。

> [!NOTE]
> 若要尋找您 Bot 的**應用程式識別碼**和**應用程式密碼**，請參閱 [MicrosoftAppID 和 MicrosoftAppPassword](~/bot-service-manage-overview.md#microsoftappid-and-microsoftapppassword)。

```cs
var mediaPlatformSettings = new MediaPlatformSettings()
{
    MediaPlatformInstanceSettings = new MediaPlatformInstanceSettings()
    {
        CertificateThumbprint = certificateThumbprint,
        InstanceInternalPort = instanceMediaControlEndpointInternalPort,
        InstancePublicIPAddress = instancePublicIPAddress,
        InstancePublicPort = instanceMediaControlEndpointPublicPort,
        ServiceFqdn = serviceFqdn
    },

    ApplicationId = MicrosoftAppId
};

MediaPlatform.Initialize(mediaPlatformSettings);            
```

## <a name="register-to-receive-incoming-call-requests"></a>註冊即可收到傳入的通話要求

定義 `CallController` 類別。 這可讓 Bot Service 註冊傳入的 Skype 通話，並確保回呼和通知要求都會轉送到適當的 `RealTimeMediaCall` 物件。

```cs
[BotAuthentication]
[RoutePrefix("api/calling")]
public class CallController : ApiController
{
    static CallController()
    {
        RealTimeMediaCalling.RegisterRealTimeMediaCallingBot(
            c => { return new RealTimeMediaCall(c); },
            new RealTimeMediaCallingBotServiceSettings()
        );
    }

    [Route("call")]
    public async Task<HttpResponseMessage> OnIncomingCallAsync()
    {
        // forwards the incoming call to the associated RealTimeMediaCall object
        return await RealTimeMediaCalling.SendAsync(this.Request, RealTimeMediaCallRequestType.IncomingCall);
    }

    [Route("callback")]
    public async Task<HttpResponseMessage> OnCallbackAsync()
    {
        // forwards the incoming callback to the associated RealTimeMediaCall object
        return await RealTimeMediaCalling.SendAsync(this.Request, RealTimeMediaCallRequestType.CallingEvent);
    }

    [Route("notification")]
    public async Task<HttpResponseMessage> OnNotificationAsync()
    {
        // forwards the incoming notification to the associated RealTimeMediaCall object
        return await RealTimeMediaCalling.SendAsync(this.Request, RealTimeMediaCallRequestType.NotificationEvent);
    }
}
```

`RealTimeMediaCallingBotServiceSettings` 可實作 `IRealTimeMediaCallServiceSettings` 並提供回呼與通知事件的 Webhook 連結。

## <a name="register-for-incoming-events-for-the-call"></a>註冊通話的傳入事件

定義可實作 `IRealTimeMediaCall` 的 `RealTimeMediaCall` 類別。 對於 Bot 所收到的每個呼叫，Bot Framework 會建立 `RealTimeMediaCall` 執行個體。 傳遞至建構函式的 `IRealTimeMediaCallService` 物件可讓 Bot 註冊事件，以處理與即時媒體呼叫相關聯的事件。

```cs
internal class RealTimeMediaCall : IRealTimeMediaCall
{
     public RealTimeMediaCall(IRealTimeMediaCallService callService)
     {
         if (callService == null)
             throw new ArgumentNullException(nameof(callService));

         CallService = callService;
         CorrelationId = callService.CorrelationId;
         CallId = CorrelationId + ":" + Guid.NewGuid().ToString();

         // Register for the call events
         CallService.OnIncomingCallReceived += OnIncomingCallReceived;
         CallService.OnAnswerAppHostedMediaCompleted += OnAnswerAppHostedMediaCompleted;
         CallService.OnCallStateChangeNotification += OnCallStateChangeNotification;
         CallService.OnRosterUpdateNotification += OnRosterUpdateNotification;
         CallService.OnCallCleanup += OnCallCleanup;
     }
}
```

## <a name="create-audio-and-video-sockets"></a>建立音訊和視訊通訊端
Bot 必須先建立 `AudioSocket` 和 `VideoSocket` 物件以便支援傳送和接收即時媒體，才可接受傳入的音訊/視訊 Skype 通話。 (如果 Bot 不想支援視訊，則應該只建立 AudioSocket。)

Bot 必須預先決定它要支援的形式，並建立適當的 AudioSocket 和 VideoSocket 物件。 在接受傳入的通話之後，Bot 就無法變更它所支援的通話形式。

針對每個 AudioSocket 和 VideoSocket，Bot 可指定媒體通訊端同時支援傳送和接收媒體，還是僅支援傳送或接收。 例如，Bot 可能想要傳送和接收音訊 ("Sendrecv")，但只能傳送視訊 ("Sendonly")。 Bot 也必須指定每個媒體通訊端支援哪些媒體格式。 針對 AudioSocket，目前支援的格式為 "Pcm16K"：(帶正負號) 16 位元 PCM 編碼、16KHz 取樣率。 針對 VideoSocket，會分別指定用於傳送和接收的媒體格式。 只有 "NV12" 格式支援接收視訊，而有數種不同的格式支援傳送。

```cs
_audioSocket = new AudioSocket(new AudioSocketSettings
{
    StreamDirections = StreamDirection.Sendrecv,
    SupportedAudioFormat = AudioFormat.Pcm16K,
    CallId = correlationId
});

_videoSocket = new VideoSocket(new VideoSocketSettings
{
    StreamDirections = StreamDirection.Sendrecv,
    ReceiveColorFormat = VideoColorFormat.NV12,
    SupportedSendVideoFormats = new VideoFormat[]
    {
        VideoFormat.Yuy2_1280x720_30Fps,
        VideoFormat.Yuy2_720x1280_30Fps,
    },
    CallId = correlationId
});

_audioSocket.AudioMediaReceived += OnAudioMediaReceived;
_audioSocket.AudioSendStatusChanged += OnAudioSendStatusChanged;
_audioSocket.DominantSpeakerChanged += OnDominantSpeakerChanged;
_videoSocket.VideoMediaReceived += OnVideoMediaReceived;
_videoSocket.VideoSendStatusChanged += OnVideoSendStatusChanged;
```                             

## <a name="create-a-mediaconfiguration"></a>建立 MediaConfiguration
建立媒體通訊端後，Bot 必須接著建立 `MediaConfiguration` 物件，這是要讓媒體通訊端與傳入的音訊/視訊 Skype 通話產生關聯所需的物件。

```cs
var mediaConfiguration = MediaPlatform.CreateMediaConfiguration(_audioSocket, _videoSocket);
```

##  <a name="answer-an-incoming-audiovideo-call"></a>接聽傳入的音訊/視訊通話
叫用 `OnIncomingCallReceived` 事件，可允許 Bot 接受傳入的 Skype 音訊/視訊通話。 若要這麼做，Bot 會使用 `MediaConfiguration` 物件建立 `AnswerAppHostedMedia` 物件。 Bot 會註冊與此 Skype 通話相關聯的通知。

```cs
private Task OnIncomingCallReceived(RealTimeMediaIncomingCallEvent incomingCallEvent)
{
    // ... create Audio/VideoSocket objects and MediaConfiguration ...

    incomingCallEvent.RealTimeMediaWorkflow.Actions = new ActionBase[]
    {
        new AnswerAppHostedMedia
        {
            MediaConfiguration = mediaConfiguration,
            OperationId = Guid.NewGuid().ToString()
        }
    };

    // subscribe for roster and call state changes
    incomingCallEvent.RealTimeMediaWorkflow.NotificationSubscriptions = new NotificationType[]
    {
        NotificationType.CallStateChange,
        NotificationType.RosterUpdate
    };
}
```

## <a name="outcome-of-the-call"></a>通話的結果
`AnswerAppHostedMedia` 動作完成時會引發 `OnAnswerAppHostedMediaCompleted`。 `AnswerAppHostedMediaOutcomeEvent` 中的 `Outcome` 屬性可指出成功或失敗。 如果無法建立通話，則 Bot 應該處置它為通話所建立的 AudioSocket 和 VideoSocket 物件。

## <a name="receive-audio-media"></a>接收音訊媒體
如果透過接收音訊功能建立 `AudioSocket`，則會在每次收到音訊框架時叫用 `AudioMediaReceived` 事件。 不論可能成為音訊內容來源的對等裝置為何，Bot 都應該要每秒處理大約 50 次這個事件 (因為若未從對等裝置收到任何音訊，就會在本機產生舒適雜訊緩衝區)。 音訊內容的每個封包都是在 `AudioMediaBuffer` 物件中傳遞。 此物件包含一個指標，可指出包含已解碼音訊內容的原生堆積配置記憶體緩衝區。 

```cs
void OnAudioMediaReceived(
            object sender,
            AudioMediaReceivedEventArgs args)
{
   var buffer = args.Buffer;

   // native heap-allocated memory containing decoded content
   IntPtr rawData = buffer.Data;            
}
```

事件處理常式必須快速傳回。 建議應用程式將 `AudioMediaBuffer` 排成以非同步方式處理。 `OnAudioMediaReceived` 事件會由即時媒體平台序列化 (也就是，直到目前的事件傳回，才會引發下一個事件)。 取用 `AudioMediaBuffer` 之後，應用程式就必須呼叫緩衝區的 Dispose 方法，以便媒體平台回收基礎非受控記憶體。 

```cs
   // release/dispose buffer when done 
   buffer.Dispose();
```

> [!IMPORTANT]
> Bot 在完成緩衝區存取之前，不得呼叫緩衝區的 Dispose 方法。

## <a name="receive-video-media"></a>接收視訊媒體
接收視訊與接收音訊類似，不同之處在於每秒的緩衝區數目將取決於畫面播放速率的值。 `VideoMediaBuffer` 具有 `VideoFormat` 和 `OriginalVideoFormat` 屬性。 `OriginalVideoFormat` 代表緩衝區在作為來源時的原始格式。 它只適用於透過 `IVideoSocket.VideoMediaReceived` 事件處理常式接收視訊緩衝區時。 如果緩衝區已在傳輸前調整大小，則 `OriginalVideoFormat` 屬性會有原始的寬度和高度，而 `VideoFormat` 會有調整大小之後的現行寬度和高度。 如果 `OriginalVideoFormat` 的寬度和高度屬性與 `VideoFormat` 屬性不同，則在 `VideoMediaReceived` 事件中引發的 `VideoMediaBuffer` 取用者應調整緩衝區大小以符合 `OriginalVideoFormat` 大小。 目前只有 NV12 格式支援接收。

## <a name="send-audio-media"></a>傳送音媒體
如果 `AudioSocket` 已設為傳送媒體，則 Bot 應該在 `AudioSocket` 上註冊 `AudioSendStatusChanged` 事件處理常式，以取得有關傳送狀態變更的通知。 Bot 只有在傳送狀態變更為 [作用中] 之後，才應該開始傳送音訊。

```cs
void AudioSocket_OnSendStatusChanged(
             object sender,
             AudioSendStatusChangedEventArgs args)
{
    switch (args.MediaSendStatus)
    {
    case MediaSendStatus.Active:
        // notify bot to begin sending audio 
        break;
     
    case MediaSendStatus.Inactive:
        // notify bot to stop sending audio
        break;
    }
}
```

為了傳送音訊媒體，假設 PCM 音訊內容包含在原生堆積配置的緩衝區內。 Bot 必須衍生自 `AudioMediaBuffer` 抽象類別。 AudioSocket 的 `Send` 方法會以非同步方式傳送媒體，而平台會在傳送完成時，在 `AudioMediaBuffer` 上呼叫 `Dispose`。 當 `Send` 傳回時，Bot 不得釋放 (或傳回給集區配置器) 非受控資源。 它必須等待 `Dispose` 被呼叫。

## <a name="send-video-media"></a>傳送視訊媒體
傳送視訊媒體類似於傳送音訊媒體。 Bot 應該註冊 `VideoSendStatusChanged` 事件，並等候 `MediaSendStatus` 變成 `Active`。 在呼叫 `Dispose` 方法之前，Bot 不得釋放或回收緩衝區的非受控資源。 支援 RGB24、NV12 和 Yuy2 色彩格式。

雖有多個 `VideoFormat` 支援傳送視訊，但目前慣用的 `VideoFormat` 會透過 `VideoSendStatusChanged` 事件傳達給 Bot。 慣用於傳送視訊的 `VideoFormat` 可能會因為網路頻寬條件，或對等用戶端調整其視訊視窗大小而變更。

```cs
void VideoSocket_OnSendStatusChanged(
            object sender,
            VideoSendStatusChangedEventArgs args)
{
    VideoFormat preferredVideoFormat;

    switch (args.MediaSendStatus)
    {
    case MediaSendStatus.Active:
        // notify bot to begin sending audio 
        // bot is recommended to use this format for sourcing video content.
        preferredVideoFormat = args.PreferredVideoSourceFormat;
        break;
     
    case MediaSendStatus.Inactive:
        // notify bot to stop sending audio
        break;
    }
}
```

每個 `VideoMediaBuffer` 都有 VideoFormat 屬性，可指出該特定緩衝區的視訊內容格式。 雖然 `VideoFormat` 屬性不必符合 `VideoSendStatusChanged` 事件中所指出的 `PreferredVideoSourceFormat` 屬性，但強烈建議使用所指出的 `PreferredVideoSourceFormat`，以避免將 CPU 週期浪費在調整視訊框架大小。

## <a name="call-notifications"></a>呼叫通知
### <a name="call-state-changes"></a>呼叫狀態變更
Bot 可藉由在 `RealTimeMediaIncomingCallEvent.RealTimeMediaWorkflow` 的 `NotificationSubscriptions` 中訂閱 `NotificationType.CallStateChange`，以取得呼叫狀態變更通知。

```cs
private Task OnCallStateChangeNotification(CallStateChangeNotification callStateChangeNotification)
{
    if (callStateChangeNotification.CurrentState == CallState.Terminated)
    {   
        // stop sending media and dispose the media sockets
        _audioSocket.Dispose();
        _videoSocket.Dispose();
    }

    return Task.CompletedTask;
}
 ```

### <a name="roster-update"></a>名冊更新
如果將 Bot 新增至會議，它可藉由在 `RealTimeMediaIncomingCallEvent.RealTimeMediaWorkflow` 的 `NotificationSubscriptions` 中訂閱 `NotificationType.RosterUpdate`，以聽取會議的名冊。 `RosterUpdateNotification` 具有會議中所有參與者的詳細資料。 Bot 可以選擇等候參與者傳送視訊的通知，然後在其中一個 `VideoSocket` 物件上 `Subscribe` 參與者的視訊。

```cs
private async Task OnRosterUpdateNotification(RosterUpdateNotification rosterUpdateNotification)
{
    // Find a video source in the conference to subscribe
    foreach (RosterParticipant participant in rosterUpdateNotification.Participants)
    {
        if (participant.MediaType == ModalityType.Video
            && (participant.MediaStreamDirection == "sendonly" || participant.MediaStreamDirection == "sendrecv")
            )
        {
            var videoSubscription = new VideoSubscription
            {
                ParticipantIdentity = participant.Identity,
                OperationId = Guid.NewGuid().ToString(),
                SocketId = _videoSocket.SocketId,
                VideoModality = ModalityType.Video,
                VideoResolution = ResolutionFormat.Hd720p
            };

            await CallService.Subscribe(videoSubscription).ConfigureAwait(false);
            break;
        }
    }  
}
```

## <a name="end-the-call"></a>結束呼叫

### <a name="handle-call-termination-from-the-caller"></a>由呼叫者處理呼叫終止
當使用者在對等對話中中斷對 Bot 的呼叫時，或是當 Bot 從會議中移除時，Bot 就會取得 CallState 為 Terminated 的呼叫狀態變更通知。 Bot 應該處置它所建立的任何媒體通訊端。

### <a name="terminate-the-call-from-the-bot"></a>終止來自 Bot 的呼叫
Bot 可以選擇藉由在 `IRealTimeMediaCallService` 上呼叫 `EndCall` 來結束呼叫。

### <a name="handle-call-clean-up-by-the-bot-framework"></a>由 Bot Framework 處理呼叫清除
在錯誤狀況 (例如，若未在合理的時間內接收 `AnswerAppHostedMediaOutcomeEvent`) 下，Bot Framework 可能會終止呼叫。 Bot 應該註冊 `OnCallCleanup` 事件並處置媒體通訊端。

