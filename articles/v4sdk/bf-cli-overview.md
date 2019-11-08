---
title: Azure Bot Framework 命令列介面 (CLI) 概觀 | Microsoft Docs
description: 了解 Bot Framework 命令列介面 (CLI)。
keywords: Bot Framework 命令列介面, Bot Framework CLI
author: JonathanFingold
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 11/01/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 4780d5258af7d2c93fafece361326fd2b0f8df77
ms.sourcegitcommit: 4751c7b8ff1d3603d4596e4fa99e0071036c207c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 11/02/2019
ms.locfileid: "73443186"
---
<!--TODO:
- [?] Add to TOC: Reference/Bot Framework CLI/Reference
- [?] Add other topics to the same node for each of the command groups
-->
# <a name="bot-framework-cli-overview"></a>Bot Framework CLI 概觀

[!INCLUDE[applies-to](../includes/applies-to.md)]

Bot Framework 命令列介面 (CLI) 是一種跨平台工具，可讓您管理 Bot 和相關服務。 其藉由將一組工具彙總為單一工具的方式，取代較舊的獨立 CLI 工具集合。 

## <a name="prerequisites"></a>必要條件

* [Node.js](https://nodejs.org/)，版本 10.14.1 或更新版本。

## <a name="installation"></a>安裝

從命令列安裝 BF CLI。

~~~cmd
npm i -g @microsoft/botframework-cli
~~~

## <a name="available-commands"></a>可用的命令

以下是目前可用的命令。

| 舊工具 | BF 命令集 | 說明 |
| :--- | :--- | :--- |
| ChatDown | [`bf chatdown`](bf-cli-reference.md#bf-chatdown) | 用於處理聊天對話 ( **.chat**) 檔案的命令。 |
| NA | [`bf config`](bf-cli-reference.md#bf-config) | 設定 CLI 內的各種設定。 |
| LuDown, LuisGen | [`bf luis`](bf-cli-reference.md#bf-luis) | 用於處理 LUIS 資源檔和管理 LUIS 模型的命令。 |
| QnAMaker | [`bf qnamaker`](bf-cli-reference.md#bf-qnamaker) | 用於處理 QnA Maker 資源檔和管理知識庫的命令。 |

下列工具會在即將推出的版本中移植：
- LUIS (API)
- 分派

如需舊版和新版工具之間的對應參考，請參閱[移植對應](https://github.com/microsoft/botframework-cli/blob/master/PortingMap.md)。

_注意：舊版 CLI 工具將在即將發行的版本中淘汰，並在未來停止提供支援。此區域中的所有新投資、錯誤 (bug) 修正和新功能將僅以 BF CLI 為目標。_

## <a name="overview"></a>概觀

BF CLI 會管理 Bot 和相關服務。 Microsoft Bot Framework 是建置企業級對話式 AI 體驗的全方位架構，而此功能是其中的一部份。 除了管理 Bot 的相關資源之外，BF CLI 也可以作為持續整合和持續部署 (CI/CD) 管線的一部分使用。 當您建立 Bot 時，可能也需要整合 AI 服務，例如使用 LUIS 來理解語言，以及使用 QnAMaker 來讓 Bot 回應問與答格式的簡單問題等等。 若要在您的 Bot 中整合 AI 服務，請使用：

* [`bf luis`](bf-cli-reference.md#bf-luis) 命令，用於使用 LUIS **.lu** 資源檔和管理 LUIS 模型。 該命令也可以產生對應的原始 (C# 或 JavaScript) 程式碼。
* [LUIS API 工具](https://github.com/microsoft/botbuilder-tools/tree/master/packages/LUIS/readme.md)，用於部署本機檔案，並將這些檔案定型、測試及發佈為 LUIS 服務內的 Language Understanding 模型。
* [`bf qnamaker`](bf-cli-reference.md#bf-qnamaker) 命令，用於使用 QnAMaker 知識庫。 此命令可以在本機和 QnA Maker 服務上建立和管理 QnA Maker 資產。

* 請參閱 lu 程式庫[文件](https://github.com/microsoft/botframework-cli/tree/master/packages/lu/README.md)，以了解如何使用 **.lu** 和 **.qna** 檔案格式。

當您的 Bot 變得更精密時，請使用[分派](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/Dispatch) CLI 工具，在多個 LUIS 模型和 QnA Maker 知識庫之間建立、評估和分派意圖。

若要測試並精簡您的 Bot，您可以使用 [Bot Framework Emulator](https://github.com/Microsoft/BotFramework-Emulator/releases)。 Emulator 可讓您對本機電腦或雲端中的 Bot 進行測試和偵錯。

在初期設計階段中，您可以針對 Bot 將支援的特定案例，建立使用者與 Bot 之間的模擬對話。 使用 [`bf chatdown`](bf-cli-reference.md#bf-chatdown) 命令來撰寫交談原型 **.chat** 檔案，將其轉換成豐富的文字記錄，然後在 Emulator 中檢視對話。

最後，透過 [Azure CLI](https://github.com/microsoft/botframework-cli/blob/master/AzureCli.md) (`az bot` 命令)，您可以使用 [Azure Bot 服務](https://azure.microsoft.com/services/bot-service/)來建立、下載、發佈和設定通道。 這是一個可擴充 [Azure CLI](https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest) 功能來管理您 Azure Bot 服務資產的外掛程式。

## <a name="privacy-and-instrumentation"></a>隱私權及檢測
BF CLI 包含檢測選項，其設計目的是為了協助我們根據**匿名**使用模式來改善工具。 __此功能預設為停用、表示不參加__。 如果您選擇加入，Microsoft 會收集一些使用資料：

* 命令群組呼叫
* 使用的旗標，**不包括**特定值。 例如，針對參數 `--folder:name`，我們只會收集 `--folder` 用法，而不會收集資料夾名稱。

若要修改資料收集行為，請使用 [`bf config`](bf-cli-reference.md#bf-config) 命令。

如需詳細資訊，請參閱 [Microsoft 隱私權聲明](https://privacy.microsoft.com/privacystatement)。  

## <a name="issues-and-feature-requests"></a>問題和功能要求
- 您可以在[這裡](https://github.com/microsoft/botframework-cli/issues)提出問題和功能要求。
- 您可以在[這裡](https://github.com/microsoft/botframework-cli/labels/known-issues)找到已知問題。

## <a name="next-steps"></a>後續步驟
- [BF CLI 參考](bf-cli-reference.md)
