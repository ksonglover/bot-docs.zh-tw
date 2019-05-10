---
redirect_url: https://aka.ms/realTimeMediaCalling-repo
ms.openlocfilehash: f6568ed5d4f98addb19f452142a0a5d6d37e00c8
ms.sourcegitcommit: f84b56beecd41debe6baf056e98332f20b646bda
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/03/2019
ms.locfileid: "65032972"
---
<a name="--"></a><!--
---
標題：使用 Skype 進行即時媒體通話 | Microsoft Docs 描述：了解建置 Bot 的主要概念，可以使用適用於 .NET 的 Bot Framework SDK，透過 Skype 進行即時音訊和視訊通話。
author: ssulzer ms.author: ssulzer manager: kamrani ms.topic: article ms.service: bot-service ms.subservice: sdk ms.date:12/13/2017 monikerRange: 'azure-bot-service-3.0'
---

# <a name="real-time-media-calling-with-skype"></a>使用 Skype 進行即時媒體通話

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

Bot 的即時媒體平台可實現即時語音、視訊和畫面共用，讓 Bot 能以更豐富的形式與使用者互動。 這可提供建置引人注目和互動式環境、教育及協助 Bot 的功能。 使用者以 Skype 與即時媒體 Bot 通訊。

這是進階功能，可以讓 Bot「逐格」傳送及接收語音和視訊內容。 Bot 具有語音、視訊和畫面共用即時形式的「原始」存取權。 例如，當使用者說話時，Bot 每秒會收到 50 個音訊框架，每個框架包含 20 毫秒 (ms) 的音訊。 當 Bot 收到音訊框架時，就可以執行即時語音辨識，不需要等待使用者停止說話之後的錄音。 Bot 也可以伴隨著音訊，將每秒 30 格畫面格數的高畫質解析視訊傳送給使用者。

Bot 的即時媒體平台可以與 Skype 通話雲端搭配運作，負責通話設定和媒體工作階段建立，讓 Bot 參與和 Skype 來電者的語音與 (選擇性) 視訊對話。 平台提供簡單的類「通訊端」API，可讓 Bot 傳送及接收媒體，並且使用轉碼器 (例如針對音訊使用 SILK 而針對視訊使用 H.264) 處理媒體的即時編碼和解碼。 即時媒體 Bot 也可以參與有多個參與者的 Skype 群組視訊通話。

這篇文章介紹了重要概念，與建置可以透過 Skype 進行即時音訊和視訊通話的 Bot 相關，並且提供相關開發人員資源的連結。

## <a name="media-session"></a>媒體工作階段
當即時媒體 Bot 回應連入的 Skype 通話時，會決定要同時支援音訊和視訊形式，還是只支援音訊。 針對每種支援的形式，Bot 可以同時傳送及接收媒體、僅接收或僅傳送。 例如，Bot 可能想要同時傳送及接收音訊，但是僅傳送視訊 (因為不想要接收 Skype 來電者的視訊)。 在 Skype 來電者與 Bot 之間所建立的一組音訊和視訊形式，稱為**媒體工作階段**。

可支援兩種視訊形式：「主視訊」和「畫面共用」。 主視訊會將 Bot 產生的視訊傳輸給來電者，或者對來電者播放，以及將來電者的視訊 (通常來自使用者的網路攝影機) 傳輸給 Bot。 畫面共用形式可以讓來電者將自己的畫面 (視訊形式) 與 Bot 共用。 Bot 無法將畫面共用視訊傳送給使用者。

當加入多方 Skype **群組視訊通話**時，即時媒體 Bot 可以同時支援接收多個主視訊串流。 這樣可以讓 Bot「看見」群組視訊通話中的多位參與者。

## <a name="frames-and-frame-rate"></a>框架和畫面播放速率
即時媒體 Bot 會與 Skype 通話的音訊和視訊形式直接互動。 這表示 Bot 是以序列**框架**的形式來傳送和/或接收媒體，其中每個框架代表一個單位的內容。 可以用 50 個框架的序列來傳輸一秒的音訊，其中每個框架包含 20 毫秒 (ms) (每秒的 1/50) 的內容。 可以將一秒的視訊配量為 30 個靜止影像的序列，在下一個視訊框架顯示之前，每個框架只會顯示 33ms (每秒的 1/30)。 每秒傳輸或轉譯的框架數稱為**畫面播放速率**。 "30fps" 表示每秒 30 格畫面格數。

## <a name="audio-format"></a>音訊格式
每秒的音訊是以 16,000 個**樣本**表示，每個樣本儲存 16 位元的資料。 20ms 音訊框架包含 320 個樣本 (640 個位元組的資料)。

## <a name="video-format"></a>視訊格式
有數種支援的視訊格式。 視訊格式的兩個主要屬性是其**框架大小**和**色彩格式**。 支援的框架大小包括 640x360 ("360p")、1280x720 ("720p") 和 1920x1080 ("1080p")。 支援的色彩格式包括 NV12 (12 位元/像素) 和 RGB24 (24 位元/像素)。

"720p" 視訊框架包含 921,600 個像素 (1280 乘以 720)。 在 RGB24 色彩格式中，每個像素表示為 3 個位元組 (24 位元)，由紅色、綠色及藍色色彩元件各一個位元組所組成。 因此，單一 720p RGB24 視訊框架需要 2,764,800 個位元組的資料 (921,600 像素乘以 3 位元組/像素)。 在 30fps 的畫面播放速率中，傳送 720p RGB24 視訊框架表示處理了大約 80 MB/秒的內容 (在網路傳輸之前，會以 H.264 視訊轉碼器進行大幅壓縮)。

## <a name="active-and-dominant-speakers"></a>作用中和主控說話者
當加入由多位參與者組成的 Skype 群組視訊通話時，Bot 可以識別正在說話的是哪位參與者。 **作用中說話者**會識別在每個接收的音訊框架中，聽到哪位參與者說話。 **主控說話者**會識別群組對話中哪位參與者目前最活躍 (也就是「主控」)，即使並未在每個音訊框架中聽到他們的聲音。 主控說話者的集合可能會隨著不同參與者輪流說話而變更。

## <a name="video-subscription"></a>視訊訂閱
在與單一 Skype 來電者的通話中，Bot 會自動接收來電者的視訊 (如果 Bot 已啟用接收主視訊)。 在多方 Skype 群組視訊通話中，Bot 必須向即時媒體平台指出想要看到哪位參與者。 視訊訂閱是由 Bot 要求，以便接收參與者的視訊。 當群組視訊通話中的參與者進行他們的對話時，Bot 可以根據主控說話者集合的更新，修改想要的視訊訂閱。

## <a name="developer-resources"></a>開發人員資源 

### <a name="sdks"></a>SDK

若要開發即時媒體 Bot，您必須在 Visual Studio 專案內安裝這些 NuGet 套件：

- [適用於 .NET 的 Bot Framework SDK ](bot-builder-dotnet-overview.md)
- [適用於 .NET 的 Bot 建立器即時媒體呼叫](https://www.nuget.org/packages?q=Bot.Builder.RealTimeMediaCalling)
- [Microsoft.Skype.Bots.Media .NET 程式庫](https://www.nuget.org/packages?q=Microsoft.Skype.Bots.Media)

### <a name="code-samples"></a>程式碼範例

[BotBuilder-RealTimeMediaCalling](https://github.com/Microsoft/BotBuilder-RealTimeMediaCalling) GitHub 存放庫包含樣本，示範如何建置適用於 Skype 的即時媒體 Bot。
-->