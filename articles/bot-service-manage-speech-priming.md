---
title: 設定語音預備 | Microsoft Docs
description: 了解如何使用 Azure 入口網站來設定您 Bot Service 的語音預備。
keywords: 語音預備, 語音辨識, LUIS
author: v-royhar
ms.author: v-royhar
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ms.openlocfilehash: ba2aec255cf160a72c11c3ddfda021baae304568
ms.sourcegitcommit: 3cb288cf2f09eaede317e1bc8d6255becf1aec61
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/27/2018
ms.locfileid: "47389637"
---
# <a name="configure-speech-priming"></a>設定語音預備

語音預備會改善您 Bot 中常用的口語單字和片語辨識。 對於已啟用語音且使用 網路聊天和 Cortana 通道的 Bot，語音預備會使用 Language Understanding (LUIS) 應用程式中指定的範例，來改善重要單字的語音辨識準確度。

您的 Bot 可能已經與 LUIS 應用程式整合，或您可以選擇建立 LUIS 應用程式，來與語音預備的 Bot 相關聯。 LUIS 應用程式包含您預期使用者會對 Bot 說的範例。 您需要 Bot 辨識的重要單字都應標示為實體。 例如，在下棋 Bot 中，您要確定當使用者說 "Move knight" 時，Bot 不會解讀為 "Move night"。 LUIS 應用程式應該包含將 "knight" 標示為實體的範例。

> [!NOTE]
> 若要使用語音預備搭配網路聊天頻道，您必須使用 Bing 語音服務。 請參閱[啟用網路聊天頻道中的語音](~/bot-service-channel-connect-webchat-speech.md)，以了解如何使用 Bing 語音服務。

> [!IMPORTANT]
> 語音預備僅適用於針對 Cortana 通道或網路聊天頻道設定的 Bot。

> [!IMPORTANT]
> 非美國地區的 LUIS 應用程式不支援預備，包括：eu.luis.ai 和 au.luis.ai

## <a name="change-the-list-of-luis-apps-your-bot-uses"></a>變更您 Bot 所使用的 LUIS 應用程式清單

若要變更 Bing 語音搭配您 Bot 使用的 LUIS 應用程式清單，請執行下列作業：

1. 在 Bot Service 刀鋒視窗上按一下 [語音預備]。 可提供給您的 LUIS 應用程式清單隨即顯示。
2. 選取您想要 Bing 語音使用的 LUIS 應用程式。
 
    a. 若要在清單中選取 LUIS 應用程式，請將滑鼠停在 LUIS 模型上，直到出現一個核取方塊，然後勾選該核取方塊。
     
    b. 若要選取不在清單中的 LUIS 應用程式，請捲動到底部，並將 LUIS 應用程式識別碼 GUID 輸入文字方塊中。
     
3. 按一下 [儲存]，來儲存與您 Bot 的 Bing 語音相關聯的 LUIS 應用程式清單。

![語音預備面板](~/media/bot-service-manage-speech-priming/speech-priming.png)

## <a name="additional-resources"></a>其他資源

- 若要深入了解在網路聊天中啟用語音，請參閱[在網路聊天頻道中啟用語音](~/bot-service-channel-connect-webchat-speech.md)。
- 若要深入了解語音預備，請參閱 [Bot Framework 中的語音支援 – 與 Directline、與 Cortana 進行 Web 聊天](https://blog.botframework.com/2017/06/26/Speech-To-Text/) (英文)。
- 若要深入了解 LUIS 應用程式，請參閱 [Language Understanding Intelligent Service](https://www.luis.ai)。
