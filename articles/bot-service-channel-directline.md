---
title: 關於 Direct Line 通道
titleSuffix: Bot Service
description: Direct Line 通道功能
services: bot-service
author: ivorb
manager: kamrani
ms.service: bot-service
ms.topic: conceptual
ms.date: 11/01/2019
ms.author: kamrani
ms.openlocfilehash: 1caf469a7ec37e932ae2b7ffde85d0dbf24938a3
ms.sourcegitcommit: 312a4593177840433dfee405335100ce59aac347
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 11/12/2019
ms.locfileid: "73933482"
---
# <a name="about-direct-line"></a>關於 Direct Line

Bot Framework Direct Line 通道是能輕鬆將 Bot 整合到您行動應用程式、網頁或其他應用程式的方式。
Direct Line 有三種形式：
- Direct Line 服務 – 全域且健全的服務，適用於大部分開發人員
- Direct Line App Service 擴充功能 - 著重於安全性和效能的 Direct Line 功能 (以個人預覽版提供)
- Direct Line Speech –已針對高效能語音最佳化 (GA)

您可以藉由評估每個供應項目的功能，以及您方案的需求，來選擇最適合您的 Direct Line 供應項目。 這些供應項目會隨著時間簡化。

|                            | Direct Line | Direct Line App Service 擴充功能 | Direct Line Speech |
|----------------------------|-------------|-----------------------------------|--------------------|
| 可用性和授權    | 正式運作 | 公開預覽版，無 SLA  | GA |
| 語音辨識和文字轉語音的效能 | 標準 | 標準 | 高效能 |
| 支援舊版網頁瀏覽器 | yes | GA 時 | yes |
| Bot Framework SDK 支援 | 所有 v3、v4 | 需要 v4.63+ | 需要 v4.63+ |
| 用戶端 SDK 支援    | JS、C# | JS、C# | C++、C#、Unity、JS|
| 與網路聊天搭配使用  | yes | 是 | 否|
| VNET | 否 | 預覽 | 否 |


## <a name="addtional-resources"></a>其他資源
- [將 Bot 連線至 Direct Line](bot-service-channel-connect-directline.md)
- [將 Bot 連線至 Direct Line Speech](bot-service-channel-connect-directlinespeech.md)
