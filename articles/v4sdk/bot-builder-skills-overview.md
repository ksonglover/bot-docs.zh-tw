---
title: Bot Framework 技能概觀 | Microsoft Docs
description: 深入了解 Bot Framework 技能
author: darrenj
ms.author: darrenj
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 05/23/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 566eb58f98335b9c55499051c6a23a6e0e0b6bc5
ms.sourcegitcommit: eacf1522d648338eebefe2cc5686c1f7866ec6a2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/30/2019
ms.locfileid: "70167124"
---
# <a name="virtual-assistant---skills-overview"></a>虛擬主題 - 技能概觀

> [!NOTE]
> 本主題適用於 SDK 的 v4 版本。 

## <a name="overview"></a>概觀

開發人員藉由將可重複使用的交談式功能 (稱為技能) 拼接在一起，即可撰寫交談式體驗。

在企業內，可以建立父代 Bot，該 Bot 結合了不同小組所擁有的多個子 Bot，或更廣泛地運用其他開發人員所提供的通用功能。 透過預覽技能，開發人員可以建立新的 Bot (通常透過虛擬助理範本)，及以利用一項合併所有分派和組態變更的命令列作業新增/移除技能。     

技能本身就是可從遠端叫用的 Bot，而且有技能開發人員範本 (.NET、TS) 可供加速新技能的建立。

技能的主要設計目標是要維護一致的活動通訊協定，並確保開發體驗盡可能貼近任何一般 V4 SDK Bot。 

![技能案例](./media/enterprise-template/skills-scenarios.png)

## <a name="bot-framework-skills"></a>Bot Framework 技能

此時，我們已提供下列 Bot Framework 技能，這些技能由 Microsoft Graph 提供技能支援並以多種語言提供。

![技能案例](./media/enterprise-template/skills-at-build.png)

| Name | 說明 |
| ---- | ----------- |
|[行事曆技能](https://aka.ms/bf-calendar-skill)|將行事曆功能新增到您的助理。 由 Microsoft Graph 和 Google 提供技術支援。|
|[電子郵件技能](https://aka.ms/bf-email-skill)|將電子郵件功能新增到您的助理。 由 Microsoft Graph 和 Google 提供技術支援。|
|[待辦事項技能](https://aka.ms/bf-todo-skill)|將工作管理功能新增到您的助理。 由 Microsoft Craph 提供技術支援。|
|[景點技能](https://aka.ms/bf-poi-skill)|尋找景點及指示。 由 Azure 地圖服務和 FourSquare 提供技術支援。|
|[汽車技能](https://aka.ms/bf-autos-kill)|產業垂直整合技能，用於展示和啟用汽車功能控制。|
|[實驗性技能](https://aka.ms/bf-experimental-skills)|新聞、餐廳訂位和天氣。|

## <a name="getting-started"></a>開始使用

請參閱[教學課程](https://aka.ms/bfs-tutorials)，以了解如何運用現有的技能並增長自己的技能。
