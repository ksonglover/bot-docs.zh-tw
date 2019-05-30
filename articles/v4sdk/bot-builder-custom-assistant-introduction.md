---
title: 自訂小幫手概觀 | Microsoft Docs
description: 了解如何建立您自己的自訂小幫手
author: darrenj
ms.author: darrenj
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 05/23/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: f4b8243580ee678390177881b136a9016be4a786
ms.sourcegitcommit: ea64a56acfabc6a9c1576ebf9f17ac81e7e2a6b7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/24/2019
ms.locfileid: "66215459"
---
## <a name="custom-assistant-overview"></a>自訂小幫手概觀

## <a name="overview"></a>概觀

我們發現客戶和合作夥伴都極需要提供專屬於其品牌的對話式小幫手，並且還需要為他們的客戶量身打造，以及在各種對話式畫布和裝置上使用。 開放原始碼的自訂個人小幫手會延續針對 Bot Framework SDK 開發的 Microsoft 開放原始碼方法，讓您根據一組基本功能來完整控制終端使用者的體驗。 此外，此體驗可注入關於終端使用者和任何裝置/生態系統資訊的情報，以達到真正的整合及智慧型體驗。

我們深信客戶應該擁有他們的客戶關係和見解，並且加以擴充。 因此，任何自訂小幫手都會將使用者體驗的完整控制權提供給客戶和合作夥伴。 名稱、語音和特質都可根據組織需求加以變更。 自訂小幫手解決方案簡化了自行建立小幫手的程序，可讓您在幾分鐘內開始使用此功能。 

自訂個人小幫手功能的範圍很廣泛，通常會提供一系列功能給使用者。 為提升開發人員的生產力，並啟用可重複使用對話式體驗的活躍生態系統，我們會針對可重複使用的對話式技能，提供初始範例給開發人員。 這些技能可新增到對話式應用程式，以啟用特定對話體驗，例如尋找景點或與行事曆、工作、電子郵件互動等許多其他案例。 技能皆開放為完全自訂，且包含適用於多種語言、對話方塊和程式碼的語言模型。

我們目前執行的是初始預覽版本，並在開放原始碼存放庫方面與初始客戶及合作夥伴密切合作，目標是先將最初的體驗帶入生活，然後在未來幾個月內讓其可以更廣泛地運用。 

![自訂小幫手圖表](media/enterprise-template/CustomAssistantDiagram.jpg)

## <a name="complete-control-of-the-user-experience"></a>完整控制使用者體驗

您可以擁有和控制終端使用者體驗的所有層面。 這包括商標、名稱、語音、特質、回應和虛擬人偶。 自訂小幫手的原始碼和支援技術會完全提供給您，讓您可根據需求來進行調整。

## <a name="complete-ownership-and-control-of-data"></a>完整的資料擁有權和控制權

自訂小幫手將會部署在您的 Azure 訂用帳戶內。 因此，小幫手所產生的所有資料 (詢問的問題、使用者行為等) 會完全包含在您的 Azure 訂用帳戶中。 請參閱[認知服務的 Azure 可信任雲端](https://www.microsoft.com/en-us/trustcenter/cloudservices/cognitiveservices)和[信任中心的 Azure 區段](https://www.microsoft.com/en-us/TrustCenter/CloudServices/Azure)，以更具體地取得詳細資訊。

## <a name="your-assistant-anywhere"></a>小幫手無所不在

自訂小幫手會利用 Microsoft 的對話式 AI 平台，因此可以透過任何 Bot Framework [通道](https://docs.microsoft.com/en-us/azure/bot-service/bot-service-manage-channels?view=azure-bot-service-4.0)呈現 – 例如 WebChat、FaceBook Messenger、Skype 等。此外，透過 [Direct Line](https://docs.microsoft.com/en-us/azure/bot-service/rest-api/bot-framework-rest-direct-line-3-0-concepts?view=azure-bot-service-4.0) 通道，我們可以將體驗內嵌至桌面和行動應用程式，包括汽車、喇叭、鬧鐘等裝置。

## <a name="built-on-enterprise-grade-technology"></a>以企業級技術為基礎

自訂小幫手解決方案以 Azure Bot 服務、Language Understanding 認知服務、整合語音及一整組支援 Azure 的元件作為基礎，這表示您可以從 [Azure 全域基礎結構](https://azure.microsoft.com/en-gb/global-infrastructure/)中獲益。

此外，Language Understanding 支援由 LUIS 認知服務提供，可支援一整組[此處所列](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/luis-supported-languages)的語言。 [翻譯工具認知服務](https://azure.microsoft.com/en-us/services/cognitive-services/translator-text-api/)提供額外的機器翻譯功能，可進一步擴大自訂小幫手的適用範圍。

## <a name="integrated-and-context-aware"></a>整合與情境感知

自訂小幫手可以整合到您的裝置和生態系統中，讓您感受到真正的整合和智慧型體驗。 此情境感知功能可讓您開發更智慧型的體驗，並進一步提供別人沒有的個人化設定。

## <a name="3rd-party-assistant-integration"></a>第三方小幫手整合

自訂小幫手可讓您提供自己獨有的經驗，但也可移交給終端使用者選擇的數位助理，以處理特定類型的問題。

## <a name="flexible-integration"></a>彈性整合

我們的自訂小幫手架構具有彈性，可與您在裝置型語音或自然語言處理功能上所做的投資進行整合，當然也可以與您現有的後端系統和 API 整合。

## <a name="adaptive-cards"></a>調適型卡片

[調適性卡片](https://adaptivecards.io/)可讓自訂小幫手傳回使用者體驗元素 (例如卡片、影像、按鈕) 及文字回應。 如果裝置或對話畫布上有一個畫面，這些調適性卡片可以跨各種支援使用者體驗的裝置與平台進行轉譯 (如果適用)。 您可以在[此處](https://adaptivecards.io/samples/)找到調適性卡片的範例，而[此處](https://docs.microsoft.com/en-us/adaptive-cards/rendering-cards/getting-started)的文件中有轉譯選項的相關資訊。


## <a name="skills"></a>技術

除了基本的小幫手功能，還有一組廣泛的常見功能需要每位開發人員自行建置。 生產力是絕佳範例，每個組織可能都需要建立語言模型 (LUIS)、對話方塊 (程式碼)、整合 (程式碼) 和語言產生 (回應)，以啟用常見案例，例如，景點、電子郵件、行事曆或工作等。

這會因為需要支援大量語言而讓後續作業變得更加複雜，並產生每個組織建置自有小幫手所需的大量工作。

我們的自訂小幫手解決方案包含新的「技能」功能，僅透過設定就可讓新功能插入自訂小幫手。 

每項技能 (語言模型、對話方塊、整合程式碼和語言產生) 的各方面都可完全由開發人員自訂，因為與自訂小幫手有關的完整原始程式碼皆可在 GitHub 取得。

## <a name="example-scenarios"></a>範例案例

自訂小幫手已延伸到廣泛的產業案例中，參考用的範例案例如下所示：

- 汽車業：藉由將啟用語音的個人助理整合到汽車中，可讓終端使用者執行傳統汽車操作 (例如導航和廣播) 及攸關生產力的案例，例如在您快遲到時更改會議時間、新增項目到您的工作清單，以及提供更積極的體驗，例如汽車可以在您啟動引擎、回家或啟用定速巡航時提供要完成的工作建議。 調適性卡片會在透過隨按即說 (Push-To-Talk) 或喚醒字互動來執行的前端單元和語音整合中進行轉譯。

- 旅遊服務業：將啟用語音的個人助理整合到旅館房間裝置，可提供廣泛的住宿相關案例 (例如延長您的住宿時間、要求延後退房、客房服務)，包括門房服務和尋找當地餐廳和景點的功能。 若連結您的生產力帳戶 (選用)，可開啟更多個人化體驗，例如建議的提醒呼叫、天氣警告及留宿期間的模式學習。 目前您已可以在飯店內體驗到電視的個人化發展。

- 企業：藉由將啟用語音和文字的品牌員工助理體驗整合到企業裝置和現有的對話畫布 (例如 Teams、WebChat、Slack) 中，可讓員工管理他們的行事曆、尋找可用的會議室、尋找有特定技術的人員，或執行 HR 相關的作業。 

## <a name="getting-started"></a>開始使用

我們目前執行的是初始預覽版本，並在開放原始碼存放庫方面與初始客戶及合作夥伴密切合作，目標是先將最初的體驗帶入生活，然後在未來幾個月內讓其可以更廣泛地運用。 如果您對此感興趣並想開始使用，請填妥[此表格](https://aka.ms/customassistantpreviewform)，我們將會與您連絡。

