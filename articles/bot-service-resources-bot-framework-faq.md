---
title: Bot Service 常見問題集 | Microsoft Docs
description: Bot Framework 元素的常見問題集清單，以及新功能的上市時間。
author: scheyal
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 11/21/2019
ms.openlocfilehash: 4c8a70e4b82e2bc4a5c10d4cf73abf51f5f904ea
ms.sourcegitcommit: 91a393e885b9ef7e08ceb978ce2f567ea38e7f48
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 11/27/2019
ms.locfileid: "74564410"
---
# <a name="bot-framework-frequently-asked-questions"></a>Bot Framework 常見問題集

本文包含關於一些 Bot Framework 常見問題集的解答。

## <a name="background-and-availability"></a>背景與可用性

### <a name="why-did-microsoft-develop-the-bot-framework"></a>Microsoft 為何要開發 Bot Framework？

在我們使用對話使用者介面 (CUI) 時，目前只有少數開發人員具備必要的專業知識和工具，可以建立新的對話體驗；或是讓現有的應用程式和服務，具備使用者可以享受的對話介面。 我們建立了 Bot Framework，讓開發人員更容易建置良好的 Bot 並且與使用者連線，無論是在哪裡進行對話 (包括 Microsoft 的主要通道)。

### <a name="what-is-the-v4-sdk"></a>什麼是 v4 SDK？
Bot Framework v4 SDK 是根據意見反映而建置，並且參考了先前的 Bot Framework SDK。 它引入了正確層級的抽象概念，同時採用了豐富的 Bot 建置組塊元件化功能。 您可以從簡單的 Bot 開始，並且使用模組化和可延伸的架構，讓您的 Bot 更細緻。 您可以在 GitHub 上找到 SDK 的[常見問題集](https://github.com/Microsoft/botbuilder-dotnet/wiki/FAQ)。

### <a name="running-a-bot-offline"></a>離線執行 Bot

<!-- WIP -->
在討論如何離線使用 Bot 之前 (表示 Bot 未部署在 Azure 上或在其他一些主機服務上，但已內部部署)，讓我們來澄清幾個重點。

- Bot 是一種沒有 UI 的 Web 服務，因此使用者必須透過其他方式、以使用 [Azure 連接器服務](rest-api/bot-framework-rest-connector-concepts.md#bot-connector-service)的通道形式與其互動。 連接器函式可做為 *Proxy*，在用戶端與 Bot 之間轉送訊息。
- **連接器**是裝載於 Azure 節點且散佈各地的全域應用程式，具備可用性和延展性。 
- 您可以使用 [Bot 通道註冊](bot-service-quickstart-registration.md)，向連接器註冊 Bot。
    >[!NOTE]
    > Bot 必須有可由連接器公開觸達的端點。

您可以離線執行 Bot，但功能有限。 例如，如果您想要使用具有 LUIS 功能的離線 Bot，必須建立 Bot 的容器、必要的工具，以及 LUIS 的容器。 兩者都是透過 Docker Compose 橋接網路進行連線。

這是「部份」離線解決方案，因為認知服務容器需要週期性線上連線。

> [!NOTE]
> 離線執行的 Bot 不支援 QnA 服務。

如需詳細資訊，請參閱

- [將 Language Understanding (LUIS) 容器部署至 Azure 容器執行個體](https://docs.microsoft.com/azure/cognitive-services/luis/deploy-luis-on-container-instances)
- [Azure 認知服務中的容器支援](https://docs.microsoft.com/azure/cognitive-services/cognitive-services-container-support)

## <a name="bot-framework-sdk-version-3-lifetime-support-and-deprecation-notice"></a>Bot Framework SDK 第 3 版存留期支援與取代通知
Microsoft Bot Framework SDK V4 已於 2018 年 9 月發行，從那時起，我們已提供了幾個次要版本來改善功能。 如先前所宣佈，V3 SDK 已淘汰。 因此，V3 存放庫中將不再有任何開發。 **現有的 V3 Bot 工作負載將會繼續執行，並不會發生中斷。我們不打算中斷任何正在執行的工作負載**。

如前面所述，Bot Builder SDK V3 Bot 會繼續執行並受 Azure Bot Service 支援。 Bot Builder SDK V3 將只能受到重大安全性錯誤修正、連接器和通訊協定階層相容性更新的支援。  

所有新功能都只會在 [Bot Framework SDK V4](https://github.com/microsoft/botframework-sdk) 上開發。  建議客戶儘快將其 Bot 遷移至 V4。

強烈建議您開始將 V3 聊天機器人遷移至 V4。 為了支援此移轉，我們已製作移轉文件，且會對移轉計劃提供延伸支援 (透過 Stack Overflow 和 Microsoft 客戶支援等標準管道)。

如需詳細資訊，請參閱下列參考文件：
* [基本移轉指引](https://aka.ms/bfv3v4migration)
* 主要 V4 存放庫會用來開發 Bot Framework 的 Bot
  * [適用於 dotnet 的 Botbuilder](https://github.com/microsoft/botbuilder-dotnet)
  * [適用於 JS 的 Botbuilder](https://github.com/microsoft/botbuilder-js) 
* QnA Maker 程式庫已取代為下列 V4 程式庫：
  * [適用於 dotnet 的程式庫](https://github.com/Microsoft/botbuilder-dotnet/tree/master/libraries/Microsoft.Bot.Builder.AI.QnA)
  * [適用於 JS 的程式庫](https://github.com/Microsoft/botbuilder-js/blob/master/libraries/botbuilder-ai/src/qnaMaker.ts)
* Azure 程式庫已取代為下列 V4 程式庫：
  * [適用於 JS Azure 的 Botbuilder](https://github.com/Microsoft/botbuilder-js/tree/master/libraries/botbuilder-azure)
  * [適用於 dotnet Azure 的 Botbuilder](https://github.com/Microsoft/botbuilder-dotnet/tree/master/libraries/Microsoft.Bot.Builder.Azure)

### <a name="v3-status-summary"></a>V3 狀態摘要

#### <a name="abs-service"></a>ABS 服務
1.  ABS 服務端會繼續支援執行 V3 Bot，而且沒有任何計畫的生命週期結束日期，任何執行中的 Bot 都不會中斷運作。 
2.  通道會繼續與 V3 相容，而且沒有中斷或結束生命週期的計畫。
3.  入口網站上已停止建立新的 V3 Bot；不過，若專家使用者不在 ABS 上 (例如作為 webapp 服務)，但想要獨立部署其 V3 Bot，則可以這麼做。

#### <a name="sdk-and-tools"></a>SDK 和工具

1.  我們不會從 SDK 端投資 V3，而且只會針對可預見的未來，對 SDK 分支套用重要的安全性修正 (例外狀況：我們計畫新增技能連接器，以允許 V4 Bot 呼叫舊版的 V3 Bot)。
2.  SDK 和工具開發僅適用於 V4，並沒有針對 V3 執行或規劃 (因此我們已經在進行了)。
3.  但我們不會防止任何人執行舊工具來管理其 V3 Bot。 

## <a name="how-can-i-migrate-azure-bot-service-from-one-region-to-another"></a>要如何在不同區域間遷移 Azure Bot Service？

Azure Bot Service 不支援區域移動。 這是全球性服務，不會與任何特定區域繫結。

## <a name="channels"></a>聲道
### <a name="when-will-you-add-more-conversation-experiences-to-the-bot-framework"></a>何時會將更多的對話體驗新增至 Bot Framework？

我們計劃持續改進 Bot Framework，包括額外的通道，但是目前尚無法提供排程。  
如果您希望將特定通道新增至架構，請[告訴我們][Support]。

### <a name="i-have-a-communication-channel-id-like-to-be-configurable-with-bot-framework-can-i-work-with-microsoft-to-do-that"></a>我有想要能夠使用 Bot Framework 設定的通訊通道。 可以與 Microsoft 合作完成嗎？

我們沒有一般機制可以讓開發人員將新通道新增至 Bot Framework，但是您可以透過[直接線路 API][DirectLineAPI] 將聊天機器人連線到應用程式。 如果您是通訊通道的開發人員，並且想要與我們配合，在 Bot Framework 中啟用您的通道，[歡迎與我們聯繫][Support]。

### <a name="if-i-want-to-create-a-bot-for-microsoft-teams-what-tools-and-services-should-i-use"></a>如果我想要建立適用於 Microsoft Teams 的聊天機器人，應該使用哪些工具和服務？

Bot Framework 的設計訴求，是為 Teams 和其他許多通道建置、連線及部署高品質、回應靈敏、高效能和可調整的聊天機器人。 SDK 可以用來建立文字/簡訊、影像、按鈕和支援卡片的聊天機器人 (涵蓋了現今對話體驗的大多數聊天機器人互動)，以及 Teams 特定的聊天機器人互動，例如豐富的音訊和視訊體驗。

如果您已經有絕佳的聊天機器人，並且想要觸及 Teams 對象，您的聊天機器人可以透過適用於 REST API 的 Bot Framework (前提是具有可存取網際網路的 REST 端點)，輕易地連線到 Teams (或任何受支援的通道)。

### <a name="how-do-i-create-a-bot-that-uses-the-us-government-data-center"></a>要如何建立使用美國政府資料中心的聊天機器人？

若要建立使用美國政府資料中心的聊天機器人，您必須執行 2 個主要步驟。
1. 在 appsettings.json (或 App Service 設定) 中新增 [管道提供者] 設定。 此設定需要具體設定為下列名稱/值常數：ChannelService = "https://botframework.azure.us"。 appsetting.json 的使用範例如下所示。

```json
{
  "MicrosoftAppId": "", 
  "MicrosoftAppPassword": "",
  "ChannelService": "https://botframework.azure.us"
}
```
2. 如果您要使用 .NET Core，則必須在 startup.cs 檔案中新增 ConfigurationChannelProvider。 新增方式取決於您要使用的 SDK 版本。

- 若為 4.3 版和更新版本，您必須在 ConfigureServices 方法中建立 ConfigurationChannelProvider 執行個體。 在使用 BotFrameworkHttpAdapter 類別時，請將此類別單獨插入到服務集合中，如下所示：

```csharp  
services.AddSingleton<IChannelProvider, ConfigurationChannelProvider>();
```
- 若為 4.3 之前的版本，則請在 ConfigureServices 方法中尋找 AddBot 方法。 在設定選項時，請務必新增：

```csharp
options.ChannelProvider = new ConfigurationChannelProvider();
```
您可以在[這裡](https://docs.microsoft.com/azure/azure-government/documentation-government-services-aiandcognitiveservices#azure-bot-service)找到有關政府服務的詳細資訊

## <a name="security-and-privacy"></a>安全性和隱私權
### <a name="do-the-bots-registered-with-the-bot-framework-collect-personal-information-if-yes-how-can-i-be-sure-the-data-is-safe-and-secure-what-about-privacy"></a>向 Bot Framework 註冊的 Bot 是否會收集個人資訊？ 如果是的話，我要如何確定資料是安全的？ 隱私權呢？

每個 Bot 都是它自己的服務，這些服務的開發人員必須根據其開發人員管理辦法，提供服務條款及隱私權聲明。  您可以從 Bot 目錄中的 Bot 卡片，存取這項資訊。

為了提供 I/O 服務，Bot Framework 會從您使用的聊天服務，將您的訊息和訊息內容 (包括您的識別碼) 傳輸到 Bot。

### <a name="can-i-host-my-bot-on-my-own-servers"></a>可以在自己的伺服器上裝載 Bot 嗎？
是。 您的 Bot 可以裝載在網際網路上的任何位置。 在您自己的伺服器上、在 Azure 中，或在任何其他資料中心。 唯一的需求是 Bot 必須公開可公開存取的 HTTPS 端點。

### <a name="how-do-you-ban-or-remove-bots-from-the-service"></a>如何從服務中禁止或移除 Bot？

使用者可以透過目錄中 Bot 的連絡人卡片，回報行為異常的 Bot。 開發人員必須遵守 Microsoft 的服務條款，才能參與服務。

### <a name="which-specific-urls-do-i-need-to-whitelist-in-my-corporate-firewall-to-access-bot-framework-services"></a>我需要將哪個特定 URL 列入公司防火牆的白名單中，才能存取 Bot Framework 服務？
如果您有輸出防火牆會封鎖 Bot 對網際網路的流量，您必須將該防火牆中的下列 URL 列入白名單：
- login.botframework.com (Bot 驗證)
- login.microsoftonline.com (Bot 驗證)
- westus.api.cognitive.microsoft.com (適用於 Luis.ai NLP 整合)
- state.botframework.com (適用於原型設計的 Bot 狀態儲存體)
- cortanabfchanneleastus.azurewebsites.net (Cortana 通道)
- cortanabfchannelwestus.azurewebsites.net (Cortana 通道)
- *.botframework.com (通道)

### <a name="can-i-block-all-traffic-to-my-bot-except-traffic-from-the-bot-connector-service"></a>我可以封鎖所有對 Bot 的流量 (來自 Bot 連接器服務的流量除外) 嗎？
沒有。 這種將 IP 位址或 DNS 列入白名單的做法並不切實際。 Bot Framework Connector Service 裝載於全球的 Azure 資料中心，而 Azure IP 清單會不斷地變更。 將某些 IP 位址列入白名單可能會運作一天，並隨著 Azure IP 位址變更而造成隔天中斷。
 
### <a name="what-keeps-my-bot-secure-from-clients-impersonating-the-bot-framework-connector-service"></a>何者可讓我的 Bot 保持安全，以便用戶端模擬 Bot Framework Connector Service？
1. 每個 Bot 要求隨附的安全性權杖內都有 ServiceUrl 編碼，這表示即使攻擊者可以存取權杖，也無法將對話重新導向新的 ServiceUrl。 這是由 SDK 的所有實作強制執行，並記載於我們的驗證[參考](https://docs.microsoft.com/azure/bot-service/rest-api/bot-framework-rest-connector-authentication?view=azure-bot-service-3.0#bot-to-connector)資料中。

2. 如果傳入權杖遺漏或格式不正確，Bot Framework SDK 不會在回應中產生權杖。 如果 Bot 的設定不正確，這會限制可以造成多少損害。
3. 在 Bot 內，您可以手動檢查權杖中提供的 ServiceUrl。 這可讓 Bot 在發生服務拓撲變更時更容易受損，所以這是可行的，但不建議這麼做。


請注意，這些都是從 Bot 到網際網路的輸出連線。 Bot Framework Connector Service 沒有將用來與 Bot 對話的 IP 位址或 DNS 名稱清單。 不支援將輸入 IP 位址列入白名單。

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

Bot Framework 和[認知服務](https://www.microsoft.com/cognitive)都是在 [Microsoft Build 2016](https://build.microsoft.com) 中引進的新功能，將會在正式上市時整合至 Cortana Intelligence Suite。 這兩項服務都是根據多年的研究所建置，並且用於熱門的 Microsoft 產品中。 這些功能與 ‘Cortana Intelligence’ 合併，可以讓每個組織利用資料、雲端和智慧的能力，建置專屬的智慧系統，以便開啟新的機會、增加營運速度並且成為業界的領先者。

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


## <a name="app-registration"></a>App 註冊

### <a name="i-need-to-manually-create-my-app-registration-how-do-i-create-my-own-app-registration"></a>我需要手動建立「應用程式註冊」。 要如何建立我自己的應用程式註冊？

如有下列情況，就必須建立自己的應用程式註冊：

- 您已在 Bot Framework 入口網站 (例如 https://dev.botframework.com/bots/new) 中建立聊天機器人 
- 您無法在組織中製作應用程式註冊，因此需要其他合作對象來為您建置的聊天機器人建立應用程式識別碼
- 您另外需要手動建立自己的應用程式識別碼 (和密碼)

若要建立您自己的應用程式識別碼，請遵循下列步驟。

1. 登入 [Azure 帳戶](https://portal.azure.com)。 如果您沒有 Azure 帳戶，可以[註冊免費帳戶](https://azure.microsoft.com/free/)。
2. 移至 [[應用程式註冊] 刀鋒視窗](https://portal.azure.com/#blade/Microsoft_AAD_RegisteredApps/ApplicationsListBlade)，然後按一下頂端動作列中的 [新增註冊]  。

    ![新增註冊](media/app-registration/new-registration.png)

3. 在 [名稱]  欄位中輸入應用程式註冊的顯示名稱，然後選取支援的帳戶類型。 名稱不必和聊天機器人識別碼一樣。

    > [!IMPORTANT]
    > 在 [支援的帳戶類型]  中，選取 [任何組織目錄中的帳戶及個人的 Microsoft 帳戶 (例如 Skype、Xbox、Outlook.com)]  選項按鈕。 若有選取任何其他選項，則**聊天機器人的建立將會失敗**。

    ![註冊詳細資料](media/app-registration/registration-details.png)

4. 按一下 [註冊] 

    之後，新建立的應用程式註冊應該會開啟一個刀鋒視窗。 複製 [概觀] 刀鋒視窗中的 [應用程式 (用戶端) 識別碼]  ，並貼到 [應用程式識別碼] 欄位中。

    ![應用程式識別碼](media/app-registration/app-id.png)

如果您要透過 Bot Framework 入口網站來建立聊天機器人，則此時便已完成應用程式註冊的設定；系統會自動產生祕密。 

如果您要在 Azure 入口網站中建立聊天機器人，則必須為應用程式註冊產生祕密。 

1. 在應用程式註冊刀鋒視窗的左側瀏覽資料行中，按一下 [憑證和祕密]  。
2. 在該刀鋒視窗中，按一下 [新增用戶端祕密]  按鈕。 在彈出的對話方塊中，輸入祕密的選擇性說明，然後從 [到期] 選項按鈕群組中選取 [永不]  。 

    ![新增祕密](media/app-registration/new-secret.png)

3. 從 [用戶端祕密]  底下的資料表中複製祕密的值，然後將此值貼到應用程式的 [密碼]  欄位，並按一下該刀鋒視窗底部的 [確定]  。 接著，繼續建立聊天機器人。 

    > [!NOTE]
    > 進入此刀鋒視窗時才能看到祕密，離開該頁面後就無法加以擷取。 請務必將其複製到安全位置。

    ![新增應用程式識別碼](media/app-registration/create-app-id.png)


[DirectLineAPI]: https://docs.microsoft.com/azure/bot-service/rest-api/bot-framework-rest-direct-line-3-0-concepts
[Support]: bot-service-resources-links-help.md
[WebChat]: bot-service-channel-connect-webchat.md
