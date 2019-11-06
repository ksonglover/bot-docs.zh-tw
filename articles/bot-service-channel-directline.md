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
ms.openlocfilehash: cba508d0895b2e2657abd37073bc5c348c1eef44
ms.sourcegitcommit: 4751c7b8ff1d3603d4596e4fa99e0071036c207c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 11/02/2019
ms.locfileid: "73441604"
---
# <a name="about-direct-line"></a>關於 Direct Line

Bot Framework Direct Line 通道是能輕鬆將 Bot 整合到您行動應用程式、網頁或其他應用程式的方式。
Direct Line 有三種形式：
- Direct Line 服務 – 全域且健全的服務，適用於大部分開發人員
- Direct Line App Service 擴充功能 - 著重於安全性和效能的 Direct Line 功能 (2019 年 5 月起以個人預覽版提供)
- Direct Line Speech – 適用於高效能語音 (2019 年 5 月起以個人預覽版提供)

您可以藉由評估每個供應項目的功能，以及您方案的需求，來選擇最適合您的 Direct Line 供應項目。 這些供應項目會隨著時間簡化。

|                            | Direct Line | Direct Line App Service 擴充功能 | Direct Line Speech |
|----------------------------|-------------|-----------------------------------|--------------------|
| 可用性和授權    | 正式運作 | 個人預覽版，無 SLA  | 個人預覽版，無 SLA |
| 語音辨識和文字轉語音的效能 | 標準 | 標準 | 高效能 |
| 整合式 OAuth           | yes | 是 | 否 |
| 整合式遙測       | yes | 是 | 否 |
| 支援舊版網頁瀏覽器 | yes | 否 | 否 |
| Bot Framework SDK 支援 | 所有 v3、v4 | 需要 v4.5+ | 需要 v4.5+ |
| 用戶端 SDK 支援    | JS、C# | JS、C# | C++、C#、Unity |
| 與網路聊天搭配使用  | yes | 是 | 否|
| 對話記錄 API | yes | 是| 否|
| VNET | 否 | 預覽* | 否 |

_* Direct Line App Service 擴充功能可用在 VNET 中，但還不允許限制輸出呼叫。_

## <a name="addtional-resources"></a>其他資源
- [將 Bot 連線至 Direct Line](bot-service-channel-connect-directline.md)
- [將 Bot 連線至 Direct Line Speech](bot-service-channel-connect-directlinespeech.md)
