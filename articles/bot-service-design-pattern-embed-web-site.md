---
title: 將 Bot 嵌入網站 | Microsoft Docs
description: 了解如何設計內嵌於網站中的 Bot。
author: matvelloso
ms.author: mateusv
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
ms.openlocfilehash: 03c7e1316e463caf84b8dfd503e1502bb66469e6
ms.sourcegitcommit: b15cf37afc4f57d13ca6636d4227433809562f8b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/11/2019
ms.locfileid: "54224843"
---
# <a name="embed-a-bot-in-a-website"></a>將 Bot 嵌入網站

Bot 通常位於網站外部，但也可能內嵌於網站中。 例如，您可以將[知識 Bot](~/bot-service-design-pattern-knowledge-base.md) 內嵌在網站中，以便使用者快速尋找資訊，否則在複雜的網站結構內往往不容易找到。 您也可以將 Bot 內嵌在技術支援中心的網站內， 做為傳入使用者要求的第一個回應程式。 Bot 可以獨立解決簡單的問題，並將較為複雜的問題[遞交](~/bot-service-design-pattern-handoff-human.md)給人類專員。 

本文章探討 Bot 與網站的整合，以及使用 *backchannel* 機制的過程，以便在網頁和 Bot 之間進行私密通訊。 

Microsoft 提供兩種不同的 Bot 與網站整合方式：Skype web 控制項和開放原始碼 web 控制項。

## <a name="skype-web-control"></a>Skype web 控制項

Skype web 控制項本質上是已啟用 web 控制項中的 Skype 用戶端。 內建的 Skype 驗證可讓 Bot 驗證和辨識使用者，而不需要開發人員撰寫任何自訂程式碼。 Skype 會自動辨識其網頁用戶端中使用的 Microsoft 帳戶。 

由於 Skype web 控制項僅做為 Skype 前端，使用者的 Skype 用戶端會自動存取任何 web 控制項促成的完整對話內容。 即使在網頁瀏覽器關閉之後，使用者也可以繼續與 Skype 用戶端的 Bot 進行互動。 

## <a name="open-source-web-control"></a>開放原始碼 web 控制項

<a href="https://aka.ms/BotFramework-WebChat" target="_blank">開放原始碼網路聊天控制項</a>是以 ReactJS 為基礎，並使用 [Direct Line API] [ directLineAPI]與 Bot Framework 進行通訊。 網路聊天控制項提供實作網路聊天所需的空白畫布，讓您完整掌控其行為和其所提供的使用者體驗。 

*Backchannel* 機制可讓裝載控制項的網頁，運用使用者完全無法察覺的方式與 Bot 直接進行通訊。 這項功能可讓數個實用案例變得可行︰ 

- 網頁可以傳送相關資料至 Bot (例如 GPS 位置)。
- 網頁可以向 Bot 通知使用者動作 (例如「使用者剛才從下拉式清單中選取了選項 A)。
- 網頁可將登入使用者的驗證權杖傳送給 Bot。
- Bot 可以傳送相關資料至網頁 (例如使用者組合目前的值)。
- Bot 可以傳送「命令」至網頁 (例如變更背景色彩)。

## <a name="using-the-backchannel-mechanism"></a>使用 backchannel 機制

[!INCLUDE [Introduction to backchannel mechanism](~/includes/snippet-backchannel.md)]

## <a name="sample-code"></a>範例程式碼

<a href="https://aka.ms/BotFramework-WebChat" target="_blank">開放原始碼網路聊天控制項</a>可透過 GitHub 取得。 如要深入了解如何使用開放原始碼網路聊天控制項實作 backchannel 機制和適用於 Node.js 的 Bot Framework SDK，請參閱[使用 backchannel 機制](~/nodejs/bot-builder-nodejs-backchannel.md)。

## <a name="additional-resources"></a>其他資源

- [Direct Line API][directLineAPI]
- [開放原始碼網路聊天控制項](https://github.com/Microsoft/BotFramework-WebChat)
- [使用 backchannel 機制](~/nodejs/bot-builder-nodejs-backchannel.md)

[directLineAPI]: https://docs.botframework.com/en-us/restapi/directline3/#navtitle
