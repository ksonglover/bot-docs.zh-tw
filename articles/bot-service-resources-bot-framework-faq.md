---
title: Bot Service 常見問題集 | Microsoft Docs
description: Bot Framework 元素的常見問題集清單，以及新功能的上市時間。
author: DeniseMak
ms.author: v-demak
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 05/03/2018
ms.openlocfilehash: e59f9b10686b10ae821b8c4bf259a1fc301ac702
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/18/2018
ms.locfileid: "39298907"
---
# <a name="bot-framework-frequently-asked-questions"></a>Bot Framework 常見問題集

本文包含關於一些 Bot Framework 常見問題集的解答。

## <a name="background-and-availability"></a>背景與可用性
### <a name="why-did-microsoft-develop-the-bot-framework"></a>Microsoft 為何要開發 Bot Framework？

在我們使用對話使用者介面 (CUI) 時，目前只有少數開發人員具備必要的專業知識和工具，可以建立新的對話體驗；或是讓現有的應用程式和服務，具備使用者可以享受的對話介面。 我們建立了 Bot Framework，讓開發人員更容易建置良好的 Bot 並且與使用者連線，無論是在哪裡進行對話 (包括 Microsoft 的主要通道)。

### <a name="what-is-the-v4-sdk"></a>什麼是 v4 SDK？
Bot 建立器 v4 SDK 是根據意見反映而建置，並且參考了先前的 Bot 建立器 SDK。 它引入了正確層級的抽象概念，同時採用了豐富的 Bot 建置組塊元件化功能。 您可以從簡單的 Bot 開始，並且使用模組化和可延伸的架構，讓您的 Bot 更細緻。 您可以在 GitHub 上找到 SDK 的[常見問題集](https://github.com/Microsoft/botbuilder-dotnet/wiki/FAQ)。


## <a name="channels"></a>聲道
### <a name="when-will-you-add-more-conversation-experiences-to-the-bot-framework"></a>何時會將更多的對話體驗新增至 Bot Framework？

我們計劃持續改進 Bot Framework，包括額外的通道，但是目前尚無法提供排程。  
如果您希望將特定通道新增至架構，請[讓我們知道][Support]。

### <a name="i-have-a-communication-channel-id-like-to-be-configurable-with-bot-framework-can-i-work-with-microsoft-to-do-that"></a>我有想要能夠使用 Bot Framework 設定的通訊通道。 可以與 Microsoft 合作完成嗎？

我們沒有一般機制可以讓開發人員將新通道新增至 Bot Framework，但是您可以透過[直接線路 API][DirectLineAPI] 將您的 Bot 連線到應用程式。 如果您是通訊通道的開發人員，並且想要與我們配合，在 Bot Framework 中啟用您的通道，[歡迎與我們聯繫][Support]。

### <a name="if-i-want-to-create-a-bot-for-skype-what-tools-and-services-should-i-use"></a>如果我想要建立適用於 Skype 的 Bot，應該使用哪些工具和服務？

Bot Framework 的設計訴求，是為 Skype 和其他許多通道建置、連線及部署高品質、回應靈敏、高效能和可調整的 Bot。 SDK 可以用來建立文字/簡訊、影像、按鈕和支援卡片的 Bot (涵蓋了現今對話體驗的大多數 Bot 互動)，以及 Skype 特定的 Bot 互動，例如豐富的音訊和視訊體驗。

如果您已經有絕佳的 Bot，並且想要觸及 Skype 對象，您的 Bot 可以透過適用於 REST API 的 Bot 建立器 (前提是具有可存取網際網路的 REST 端點)，輕易地連線到 Skype (或任何受支援的通道)。

## <a name="security-and-privacy"></a>安全性和隱私權
### <a name="do-the-bots-registered-with-the-bot-framework-collect-personal-information-if-yes-how-can-i-be-sure-the-data-is-safe-and-secure-what-about-privacy"></a>向 Bot Framework 註冊的 Bot 是否會收集個人資訊？ 如果是的話，我要如何確定資料是安全的？ 隱私權呢？

每個 Bot 都是它自己的服務，這些服務的開發人員必須根據其開發人員管理辦法，提供服務條款及隱私權聲明。  您可以從 Bot 目錄中的 Bot 卡片，存取這項資訊。

為了提供 I/O 服務，Bot Framework 會從您使用的聊天服務，將您的訊息和訊息內容 (包括您的識別碼) 傳輸到 Bot。

### <a name="how-do-you-ban-or-remove-bots-from-the-service"></a>如何從服務中禁止或移除 Bot？

使用者可以透過目錄中 Bot 的連絡人卡片，回報行為異常的 Bot。 開發人員必須遵守 Microsoft 的服務條款，才能參與服務。

### <a name="which-specific-urls-do-i-need-to-whitelist-in-my-corporate-firewall-to-access-bot-services"></a>我需要將哪個特定 URL 列入公司防火牆的白名單中，才能存取 Bot Service？

您需要將下列 URL 列入公司防火牆的白名單中：
- login.botframework.com (Bot 驗證)
- login.microsoftonline.com (Bot 驗證)
- westus.api.cognitive.microsoft.com (適用於 Luis.ai NLP 整合)
- state.botframework.com (適用於原型設計的 Bot 狀態儲存體)
- cortanabfchanneleastus.azurewebsites.net (Cortana 通道)
- cortanabfchannelwestus.azurewebsites.net (Cortana 通道)
- *.botFramework.com (通道)

## <a name="rate-limiting"></a>速率限制
### <a name="what-is-rate-limiting"></a>什麼是速率限制？
Bot Framework 服務必須保護自己本身及其客戶免於不當呼叫模式 (例如拒絕服務的攻擊) 的影響，因此沒有單一 Bot 會對其他 Bot 的效能造成不良影響。 為了實現這種保護，我們已在所有端點上新增了速率限制 (也稱為節流)。 藉由強制執行速率限制，我們可以限制 Bot 進行特定呼叫的頻率。 例如：在啟用了速率限制的情況下，如果 Bot 想要張貼大量的活動，則必須在一段期間內將它們分散。 請注意，速率限制的目的不是要限制 Bot 的總量。 限制的設計目的是防止未遵循人類對話模式的對話基礎結構濫用。

### <a name="how-will-i-know-if-im-impacted"></a>如何得知我是否受到影響？
即使數量大，您也不太會遇到速率限制。 大部分速率限制只有在大量傳送活動 (來自 Bot 或來自用戶端)、極大的負載測試，或發生錯誤時才會發生。 當要求已節流時，會傳回 HTTP 429 (太多要求) 回應以及「稍候再試」標頭，指出重試要求成功之前要等待的時間量 (以秒為單位)。 您可以藉由透過 Azure Application Insights 啟用 Bot 的分析，來收集這項資訊。 或者，您可以在 Bot 中新增程式碼來記錄訊息。 

### <a name="how-does-rate-limiting-occur"></a>速率限制如何發生？
它會在符合下列情形時發生：
-   Bot 傳送訊息頻率過高
-   Bot 的用戶端傳送訊息頻率過高
-   直接線路用戶端要求新的 Web Socket 頻率過高

### <a name="what-are-the-rate-limits"></a>速率限制有哪些？
我們會持續微調速率限制，使其在保護我們的服務和使用者的同時，盡可能寬鬆。 因為閾值偶爾會變更，所以目前我們不會發佈數字。 如果您受到速率限制的影響，歡迎透過 [bf-reports@microsoft.com](mailto://bf-reports@microsoft.com) 與我們連絡。

## <a name="related-services"></a>相關服務
### <a name="how-does-the-bot-framework-relate-to-cognitive-services"></a>Bot Framework 與認知服務有何關聯？

Bot Framework 和[認知服務](http://www.microsoft.com/cognitive)都是在 [Microsoft Build 2016](http://build.microsoft.com) 中引進的新功能，將會在正式上市時整合至 Cortana Intelligence Suite。 這兩項服務都是根據多年的研究所建置，並且用於熱門的 Microsoft 產品中。 這些功能與 ‘Cortana Intelligence’ 合併，可以讓每個組織利用資料、雲端和智慧的能力，建置專屬的智慧系統，以便開啟新的機會、增加營運速度並且成為業界的領先者。

### <a name="what-is-cortana-intelligence"></a>什麼是 Cortana Intelligence？

Cortana Intelligence 是完全受控的巨量資料、進階分析及智慧套件，您可用來將資料轉換成可採取的智慧行動。  
它是一個全方位套件，將多年研究所得的技術和 Microsoft 的創新融會在一起 (範圍橫跨進階分析、機器學習、巨量資料儲存體和雲端中的處理) 以及：

* 可讓您收集、管理及儲存您的所有資料，這些資料會隨著時間，以可調整且安全的方式，順暢又符合成本效益地成長。
* 提供由雲端驅動的簡易且可操作的分析，讓您可以針對大部分嚴苛的問題進行預測、規定以及讓決策自動化。
* 透過認知服務和代理程式啟用智慧系統，讓您能以更關聯式及自然的方式看到、聽到、詮譯及了解世界周遭。

有了 Cortana Intelligence，我們希望可以協助我們的企業客戶開啟新的機會、增加營運速度並且成為業界的領先者。

### <a name="what-is-the-direct-line-channel"></a>什麼是直接線路通道？

直接線路是一種 REST API，可讓您將 Bot 新增至您的服務、行動裝置應用程式或網頁。

您可以使用任何語言針對直接線路 API 撰寫用戶端。 只要對[直接線路通訊協定][DirectLineAPI]編寫程式碼、在直接線路設定頁面中產生祕密，然後從程式碼所在的任何位置與您的 Bot 對話即可。

直接線路適用於：

* iOS、Android、Windows Phone 及其他平台上的行動裝置應用程式
* Windows、OSX 等等平台上的傳統型應用程式
* 在其中您需要比[可內嵌網路聊天通道][WebChat]所提供更多自訂的網頁
* 服務對服務應用程式

[DirectLineAPI]: http://docs.botframework.com/en-us/restapi/directline/
[Support]: bot-service-resources-links-help.md
[WebChat]: bot-service-channel-connect-webchat.md
