---
title: 建立適用於 Telegram 的 Bot | Microsoft Docs
description: 了解如何設定 Bot 與 Telegram 的連線。
keywords: 設定 bot, Telegram, bot 通道, Telegram bot, 存取權杖
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
ms.openlocfilehash: efe38392117fb871b2b98e3f1d8d798bfaef0c41
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/24/2018
ms.locfileid: "49996505"
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

將您先前複製的權杖貼至 [存取權杖] 欄位，然後按一下 [提交]。

## <a name="enable-the-bot"></a>啟用 Bot
勾選 [在 Telegram 上啟用此 Bot]。 然後按一下 [我已完成設定 Telegram]。

當您完成這些步驟時，您的 Bot 即已成功設定為在 Telegram 中與使用者進行通訊。
