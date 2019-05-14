---
title: Azure Bot Service 簡介 | Microsoft Docs
description: 深入了解 Bot 服務；這是一項建立、連接、測試、部署、監視及管理 Bot 的服務。
keywords: 概覽, 簡介, SDK, 概述
author: Kaiqb
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 05/05/2019
ms.openlocfilehash: 569438e43a64a96239f7d9e490563498e7f6f279
ms.sourcegitcommit: 3e3c9986b95532197e187b9cc562e6a1452cbd95
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/06/2019
ms.locfileid: "65039771"
---
# <a name="about-azure-bot-service"></a>關於 Azure Bot Service

[!INCLUDE [applies-to-both](includes/applies-to-both.md)]

Azure Bot 服務和 Bot Framework 提供可在單一位置建立、測試、部署及管理智慧型 Bot 的工具。 利用 SDK 提供的模組化可延伸架構，工具、範本和 AI 服務開發人員可以建立可使用語音、理解自然語言、處理問與答等功能的 Bot。

## <a name="what-is-a-bot"></a>什麼是 Bot？
Bot 提供的體驗比較不像使用電腦，比較像是與人溝通，或至少是與智慧型機器人溝通。 在不再需要直接人為介入的自動化系統上，Bot 可用於輪替簡單、重複性工作，例如預訂晚餐或蒐集設定檔資訊。 使用者可使用文字、互動式卡片和語音來與 Bot 交談。 Bot 互動可以是快速的問與答，也可以是以智慧方式提供服務存取權的複雜對話。

Bot 很類似現代化 Web 應用程式，在網際網路上運作，並使用 API 來傳送和接收訊息。 視 Bot 的種類而定，Bot 的功能差異很大。 現代化 Bot 軟體依賴一些技術和工具，可在各種平台上提供日益複雜的體驗。 不過，簡單的 Bot 可能只會收到訊息，並以極少的相關程式碼來回應使用者。 

Bot 可執行其他類型的軟體可以執行的作業：讀取和寫入檔案、使用資料庫和 API，以及進行一般計算工作。 Bot 的特點就是其使用通常保留給人與人通訊的機制。 

Azure Bot 服務和 Bot Framework 可提供：
- 可供開發 Bot 的 Bot Framework SDK
- Bot Framework 工具，可涵蓋端對端 Bot 開發工作流程
- Bot Framework Service (BFS)，可在 Bot 與通道之間傳送及接收訊息和事件
- Azure 中的 Bot 部署和通道組態

此外，Bot 可使用其他 Azure 服務，例如：
- 用以建置智慧型應用程式的 Azure 認知服務 
- 適用於雲端儲存體解決方案的 Azure 儲存體

## <a name="building-a-bot"></a>建立 Bot 

Azure Bot 服務和 Bot Framework 可提供一組整合式工具與服務，可加快此程序。 選擇您慣用的開發環境或命令列工具來建立 Bot。 C#、JavaScript 和 Typescript 均有 SDK。 (適用於 Java 和 Python 的 SDK 正在開發中。)我們在 Bot 各種開發階段皆提供各項工具，協助您設計及建置 Bot。

![Bot 概觀](media/bot-service-overview.png) 

### <a name="plan"></a>規劃
如同任何類型的軟體，務必徹底了解目標、程序及使用者需求，才能建立成功的 Bot。 撰寫程式碼之前，請先參閱 Bot [設計指南](bot-service-design-principles.md) 了解最佳作法，並找出您的 Bot 需求。 您可以建立簡單的 Bot 或納入更複雜的功能，例如語音、自然語言理解或問題解答。

### <a name="build"></a>建置
Bot 是一項 Web 服務，可實作對話式介面並透過 Bot Framework Service 進行通訊，以傳送和接收訊息和事件。 Bot Framework Service 是 Azure Bot Service 和 Bot Framework 的其中一個元件。 您可以使用任意多個環境和語言建立 Bot。 您可以在 [Azure 入口網站](bot-service-quickstart.md)中開始進行 Bot 開發，或使用 [[C#](dotnet/bot-builder-dotnet-sdk-quickstart.md) | [JavaScript](javascript/bot-builder-javascript-quickstart.md)] 範本進行本機開發。

我們在 Azure Bot Service 和 Bot Framework 中提供可用來增強 Bot 功能的其他元件

| 功能 | 說明 | 連結 |
| --- | --- | --- |
| 新增自然語言處理 | 讓您的 Bot 能夠理解自然語言、理解拼字錯誤、使用語音，以及辨識使用者的意圖 | 如何使用 [LUIS](~/v4sdk/bot-builder-howto-v4-luis.md) 
| 回答問題 | 新增知識庫，以更自然的對話方式回答使用者的問題 | 如何使用 [QnA Maker](~/v4sdk/bot-builder-howto-qna.md) 
| 管理多個模型 | 使用多個模型時 (例如 LUIS 和 nA Maker)，可在 Bot 對話期間，採用智慧方式判斷各個模型的使用時機 | [分派](~/v4sdk/bot-builder-tutorial-dispatch.md)工具|
| 新增卡片和按鈕 | 利用文字以外的媒體 (例如圖形、功能表和卡片) 來強化使用者體驗 | 如何[新增卡片](v4sdk/bot-builder-howto-add-media-attachments.md) |

> [!NOTE]
> 上表並非完整清單。 請參閱左側文章了解更多 Bot 功能，第一篇為[傳送訊息](~/v4sdk/bot-builder-howto-send-messages.md)。

此外，我們會提供命令列工具，協助您建立、管理及測試 Bot 資產。 這些工具可設定 LUIS 應用程式、建置 QnA 知識庫、建置模型以在元件之間分派、模擬對話等等。 您可以在命令列工具[讀我檔案](https://aka.ms/botbuilder-tools-readme)中找到更多詳細資訊。

您也可以存取各種[範例](https://github.com/microsoft/botbuilder-samples)，這些範例展現許多可透過 SDK 取得的功能。 這些很適合尋求更多元用途起點的開發人員。

### <a name="test"></a>測試 
Bot 是將許多不同組件整合在一起運作的複雜應用程式。 和其他複雜的應用程式一樣，這種方式會導致一些有趣的錯誤，或是讓 Bot 產生出乎意料的行為。 在發佈之前，請先測試 Bot。 在發行 Bot 以供使用前，我們會提供數種方式來測試 Bot：

- 使用[模擬器](bot-service-debug-emulator.md)在本機測試 Bot。 Bot Framework 模擬器是獨立的應用程式，不僅提供交談介面，還提供偵錯和訊問工具來協助您了解 Bot 的運作方式和原因。  模擬器可以隨著開發中的 Bot 應用程式在本機執行。 
 
- 在 [Web](bot-service-manage-test-webchat.md) 上測試您的 Bot。 透過 Azure 入口網站進行設定後，Bot 也可透過網路聊天介面觸達。 網路聊天介面適合用來將 Bot 的存取權授與給測試人員，以及其他無法直接存取 Bot 執行中程式碼的人員。

### <a name="publish"></a>發佈 
當您準備在 Web 上提供您的 Bot 時，請將 Bot 發佈至 [Azure](bot-builder-howto-deploy-azure.md) 或自己的 Web 服務或資料中心。 在公用網際網路上有一個位址是 Bot 在您的網站上，或在聊天通道內活化的第一個步驟。

### <a name="connect"></a>連線          
將您的 Bot 連接到 Facebook、Messenger、Kik、Skype、Slack、Microsoft Teams、Telegram、簡訊 /SMS、Twilio、Cortana 及 Skype 等通道中。 Bot Framework 會進行從上述各種平台傳送和接收訊息所需的大部分工作 - 不論所連線到的通道數目和類型為何， Bot 應用程式都會接收統一、正規化的訊息串流。 如需新增通道的資訊，請參閱[通道](bot-service-manage-channels.md)主題。

### <a name="evaluate"></a>評估 
使用在 Azure 入口網站收集的資料，就有機會改善 Bot 功能和效能。 您可以取得服務層級和流量、延遲與整合等檢測資料。 Analytics 提供使用者、訊息和通道資料的相關交談層級報告。 如需詳細資訊，請參閱[如何收集分析資料](bot-service-manage-analytics.md)。


## <a name="next-steps"></a>後續步驟
請查看這些 Bot [個案研究](https://azure.microsoft.com/services/bot-service/)，或按一下下方連結，即可建立 Bot。
> [!div class="nextstepaction"]
> [建立 Bot](bot-service-quickstart.md)
