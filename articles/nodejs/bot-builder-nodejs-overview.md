---
title: 適用於 Node.js 的 Bot 建立器 SDK | Microsoft Docs
description: 探索「適用於 Node.js 的 Bot 建立器 SDK 」這個功能強大、簡單易用的 Bot 建立架構。
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 24da2cc90908a1f22bee9d2cd007607b116ac2cc
ms.sourcegitcommit: 2dc75701b169d822c9499e393439161bc87639d2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/24/2018
ms.locfileid: "42904201"
---
# <a name="bot-builder-sdk-for-nodejs"></a>Bot Builder SDK for Node.js

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-overview.md)
> - [Node.js](../nodejs/bot-builder-nodejs-overview.md)
> - [REST](../rest-api/bot-framework-rest-overview.md)

適用於 Node.js 的 Bot 建立器 SDK 是功能強大、簡單易用的架構，可為 Node.js 開發人員提供熟悉的 Bot 撰寫方式。
您可以用它建立各種不同的對話式使用者介面，從簡單的提示到自由形式的對話都在其適用範圍內。

Bot 的對話式邏輯會裝載為 Web 服務。 Bot 建立器 SDK 會使用 <a href="http://restify.com">restify</a> 這個受歡迎的 Web 服務建立架構，來建立 Bot 的 Web 伺服器。 SDK 也與 <a href="http://expressjs.com/">Express</a> 相容，稍做調整即可使用其他 Web 應用程式架構。 

使用 SDK 就可以利用下列 SDK 功能： 

- 功能強大的系統，用於建立對話以封裝對話式邏輯。
- 內建簡單事物提示 (例如，是/否、字串、數字和列舉)，以及支援內含影像和附件的訊息和內含按鈕的複合式資訊卡 (Rich Card)。
- 內建支援功能強大的 AI 架構，例如 <a href="http://luis.ai" target="_blank">LUIS</a>。
- 內建辨識器和事件處理常式，可引導使用者完成對話，視需要提供說明、瀏覽、釐清和確認。

## <a name="get-started"></a>開始使用

如果您未曾撰寫過 Bot，請透過逐步指示[使用 Node.js 建立第一個 Bot](bot-builder-nodejs-quickstart.md)，讓指示協助您設定專案、安裝 SDK 和執行第一個 Bot。 

如果您未曾使用過適用於 Node.js 的 Bot 建立器 SDK，您可以從重要概念來開始，讓其協助您了解 Bot 建立器 SDK 的主要元件，請參閱[重要概念](bot-builder-nodejs-concepts.md)。

若要確保 Bot 能應付最常見的使用者案例，請檢閱[設計原則](../bot-service-design-principles.md)和[瀏覽模式](../bot-service-design-pattern-task-automation.md)以獲得指導方針。

## <a name="get-samples"></a>取得範例

[適用於 Node.js 的 Bot 建立器 SDK 範例](bot-builder-nodejs-samples.md)會示範以工作為主的 Bot，示範如何利用適用於 Node.js 的 Bot 建立器 SDK 功能。 您可使用這些範例，協助您快速開始建置具備豐富功能的絕佳 Bot。

## <a name="next-steps"></a>後續步驟
> [!div class="nextstepaction"]
> [重要概念](bot-builder-nodejs-concepts.md)

## <a name="additional-resources"></a>其他資源

下列以工作為主的使用說明指南會示範「適用於 Node.js 的 Bot 建立器 SDK」的各種功能。

* [回應訊息](bot-builder-nodejs-use-default-message-handler.md)
* [處理使用者動作](bot-builder-nodejs-dialog-actions.md)
* [辨識使用者意圖](bot-builder-nodejs-recognize-intent-messages.md)
* [傳送複合式資訊卡 (Rich Card)](bot-builder-nodejs-send-rich-cards.md)
* [傳送附件](bot-builder-nodejs-send-receive-attachments.md)
* [儲存使用者資料](bot-builder-nodejs-save-user-data.md)


如果您有關於「適用於 Node.js 的 Bot 建立器 SDK」的問題或建議，請參閱[支援](../bot-service-resources-links-help.md)以取得可用資源清單。 


[DesignGuide]: ../bot-service-design-principles.md 
[DesignPatterns]: ../bot-service-design-pattern-task-automation.md 
[HowTo]: bot-builder-nodejs-use-default-message-handler.md 
