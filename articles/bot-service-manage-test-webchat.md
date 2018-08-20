---
title: 在網路聊天中測試 Bot Service | Microsoft Docs
description: 了解如何在 Azure 入口網站中使用 webchat 控制項來測試 Bot Service。
keywords: 在網路聊天中測試, azure 入口網站
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ms.openlocfilehash: 0c358f4e53f3fd64cce3635f644cc0f2f612e983
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/18/2018
ms.locfileid: "39297567"
---
# <a name="test-in-web-chat"></a>在網路聊天中測試
Bot Service 包含[網路聊天控制項](bot-service-channel-connect-webchat.md)，可協助您輕鬆地測試 Bot。 

## <a name="test-a-bot-in-the-azure-portal-with-web-chat"></a>在 Azure 入口網站中，用網路聊天來測試 Bot
登入 [Azure 入口網站](https://portal.azure.com)，並開啟 Bot 的刀鋒視窗。 在 [Bot 管理] 區段中，按一下 [在網路聊天中測試]。 Bot Service 會將網路聊天控制項載入，並連線至您的 Bot。

![在網路聊天 UI 中測試](~/media/test-in-webchat/test-in-webchat.png)

您可以輸入文字來與您的 Bot 聊天。 如果您的 Bot 支援語音，您可以按一下麥克風按鈕來與您的 Bot 對話。 如果您的 Bot 支援附件，您可以上傳附件 (例如影像)。 了解如何使用 [Bot 建立器 SDK](bot-builder-overview-getstarted.md) 將功能新增至 Bot。

> [!NOTE]
> 如果在幾分鐘後，網路聊天控制項未完全載入，請嘗試重新整理頁面。

## <a name="next-steps"></a>後續步驟
現在您已知道如何在 Azure 中測試您的 Bot，請了解如何利用 Bot Framework 模擬器來進行更深入的測試和偵錯功能。

> [!div class="nextstepaction"]
> [Bot Framework 模擬器](bot-service-debug-emulator.md)