---
title: 儲存對話設計工具 Bot | Microsoft Docs
description: 了解如何在對話設計工具 Bot 中儲存 Language Understanding 模型並將它定型，以及備妥語音識別。
author: vkannan
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ROBOTS: NoIndex, NoFollow
ms.openlocfilehash: 3a911158c379f879c0be604fb5e8ba30ab22ee44
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/18/2018
ms.locfileid: "39300326"
---
# <a name="saving-your-conversation-designer-bot"></a>儲存對話設計工具 Bot
> [!IMPORTANT]
> 對話設計工具尚未提供給所有客戶。 有關對話設計工具可用性的詳細資訊會於本年度稍後提供。

在對話設計工具中工作時，您的所有工作都會快取於瀏覽器記憶體中。 若要認可您的變更，按一下位於左側導覽功能表左上角的 [儲存] 按鈕。 為了避免遺失工作，建議您經常儲存工作。 除了按一下 [儲存] 按鈕，您可能也會使用 **CTRL+S** 鍵盤快速鍵來儲存工作。

## <a name="trains-luis-and-primes-speech-recognition"></a>將 LUIS 定型並備妥語音辨識

按一下 [儲存] 按鈕會將變更儲存至 Bot，並執行一些額外的工作。 不同於鍵盤快速鍵，按一下 [儲存] 按鈕也會指示對話設計工具執行下列工作：

- 將適用於 Bot 的任何新 LUIS 意圖和實體定型，並在您的 Bot 服務中本機發佈 LUIS 模型 (如有需要)。 這些意圖可能會新增於對話設計工具或 Bot 的對應 [LUIS](https://luis.ai) 應用程式中。
- 更新對話執行階段以使用新的 LUIS 模型。
- 藉由準備您的範例語句並傳送到 Microsoft 認知服務來備妥語音辨識，進而大幅改善這個 Bot 的語音辨識準確度。

## <a name="next-step"></a>後續步驟
> [!div class="nextstepaction"]
> [測試 Bot](conversation-designer-debug-bot.md)
