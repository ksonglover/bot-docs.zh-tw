---
title: 建立適用於 Telegram 的 Bot | Microsoft Docs
description: 了解如何設定 Bot 與 Telegram 的連線。
keywords: 設定 bot, Telegram, bot 通道, Telegram bot, 存取權杖
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
ms.openlocfilehash: cdf9a98d77f876fb582432ab9b4704d2ac98d45f
ms.sourcegitcommit: dd12ddf408c010182b09da88e2aac0de124cef22
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/05/2019
ms.locfileid: "70385960"
---
# <a name="connect-a-bot-to-telegram"></a>將 Bot 連線至 Telegram

您可以設定您的 Bot 使用 Telegram 傳訊應用程式與其他人通訊。

[!INCLUDE [Channel Inspector intro](~/includes/snippet-channel-inspector.md)]

## <a name="visit-the-bot-father-to-create-a-new-telegram-bot"></a>請造訪 Bot Father 以建立新的 Telegram Bot

使用 Bot Father <a href="https://telegram.me/botfather" target="_blank">建立新的 Telegram Bot</a>。

![造訪 Bot Father](~/media/channels/tg-StepVisitBotFather.png)

## <a name="create-a-new-telegram-bot"></a>建立新的 Telegram Bot
若要建立新的 Telegram Bot，請傳送命令 `/newbot`。

![建立新的 Bot](~/media/channels/tg-StepNewBot.png)

### <a name="specify-a-friendly-name"></a>指定易記的名稱

給 Telegram Bot 一個易記的名稱。

![給 Bot 一個易記的名稱](~/media/channels/tg-StepNameBot.png)

### <a name="specify-a-username"></a>指定使用者名稱

給 Telegram Bot 一個唯一的使用者名稱。

![給 Bot 一個唯一的名稱](~/media/channels/tg-StepUsername.png)

### <a name="copy-the-access-token"></a>複製存取權杖

複製 Telegram Bot 的存取權杖。

![複製存取權杖](~/media/channels/tg-StepBotCreated.png)

## <a name="enter-the-telegram-bots-access-token"></a>輸入 Telegram Bot 的存取權杖

在 Azure 入口網站中移至 Bot 的 [通道]  區段，然後按一下 [Telegram]  按鈕。 

> [!NOTE]
>  如果您已將 Bot 連線至 Telegram，則 Azure 入口網站 UI 看起來會略有不同。 

![選取通道中的 Telegram](~/media/channels/tg-connectBot-Azure.png)

將您先前複製的權杖貼到 [存取權杖]  欄位中，然後按一下 [儲存]  。

![Telegram 存取權杖](~/media/channels/tg-accessToken-Azure.png)

您的 Bot 現已成功設定為可與 Telegram 中的使用者通訊。 

![已啟用 Telegram Bot](~/media/channels/tg-botEnabled-Azure.png)
