---
title: 物聯網 Bot 案例 |Microsoft Docs
description: 藉由 Bot Framework 探索物聯網 Bot 案例。
author: BrianRandell
ms.author: v-brra
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 0cdade289bc7474a3d2597ae54fd2c355a7969f9
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/24/2018
ms.locfileid: "49996995"
---
# <a name="internet-of-things-iot-bot-scenario"></a>物聯網 (IoT) Bot 案例

[!INCLUDE [pre-release-label](includes/pre-release-label-v3.md)]

這個物聯網 (IoT) Bot 可讓您輕鬆地使用語音或互動式聊天命令控制住家內的裝置，例如 Philips Hue 燈光。

人們喜歡以對話來操作自己的裝置。 自從第一台電視遙控器問世以來，人們就愛上這種不需要移動即可影響環境的操作方式。 此 IoT Bot 讓使用者透過簡單的聊天命令或語音來管理 Philips Hue。 此外，使用聊天功能時，使用者可獲得與色彩相關的視覺化選項。

![物聯網 Bot 圖](~/media/scenarios/bot-service-scenario-iot-bot.png)

以下是 IoT Bot 的邏輯流程：

1. 使用者登入 Skype 並存取 IoT Bot。
2. 使用者透過語音要求 Bot 經由 IoT 裝置開啟燈光。
3. 將該要求轉送至可存取 IoT 裝置網路的協力廠商服務。
4. 將該命令結果傳回給使用者。
5. Application Insights 會收集執行階段遙測，以協助開發 Bot 效能和使用量。

## <a name="sample-bot"></a>範例 Bot
IoT Bot 將能讓您快速地使用 Skype 或 Slack 等頻道的聊天命令，來控制您的 Hue。 為了方便遠端存取，您可以呼叫預先定義的 IFTTT 小程式 來搭配 Hue 使用。

您可以從[常見的 Bot Framework 案例範例](https://aka.ms/bot/scenarios)下載或複製此範例 Bot 的原始程式碼。

## <a name="components-youll-use"></a>您將使用的元件
物聯網 (IoT) Bot 會使用下列元件：
-   Philips Hue
-   If This Then That (IFTTT)
-   Application Insights

### <a name="philips-hue"></a>Philips Hue
與 Philips Hue 連接的燈泡和橋接器可讓您充分掌握您的照明設備。 無論您想要如何處理自己的照明設備，Hue 都能辦到。 Hue 備有您可透過區域網路使用的 API。 不過，如果您想要從任何地方使用簡單好用的 Bot 介面存取由 Hue 控制的裝置和照明設備， 請透過 IFTTT 存取 Hue。

### <a name="ifttt"></a>IFTTT
IFTTT 是免費的 Web 型服務，可供使用者建立簡單的條件陳述式鏈結 (稱為小程式)。 您可以從自己的 Bot 觸發小程式來代替您執行一些動作。 有許多預先定義的 Hue 小程式可用來開關燈光、 變更場景等。

### <a name="application-insights"></a>Application Insights
Application Insights 可協助您透過應用程式效能管理和立即分析，取得可操作的見解。 根據預設，您可以取得豐富的效能監視、強大的警示與便於取用的儀表板，以協助確保您的 Bot 可供使用，並如預期般執行。 您可以快速查看是否遇到問題，然後執行根本原因分析以找出問題並加以修正。
