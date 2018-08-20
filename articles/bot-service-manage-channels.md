---
title: 將 Bot 設定為在一或多個通道上執行 | Microsoft Docs
description: 了解如何使用 Bot Framework 入口網站，將 Bot 設定為在一或多個通道上執行。
keywords: Bot 通道, 設定, cortana, facebook messenger, kik, slack, skype, azure 入口網站
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ms.openlocfilehash: cb682bf77f801c98d00deffa0fc63249962248cd
ms.sourcegitcommit: dcbc8ad992a3e242a11ebcdf0ee99714d919a877
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/30/2018
ms.locfileid: "39352917"
---
# <a name="connect-a-bot-to-channels"></a>將 Bot 連線至通道

通道會連接 Bot Framework 和通訊應用程式。 您可以將 Bot 設定為連線到您想要提供此 Bot 的通道。 例如，您可以將連線至 Skype 通道的 Bot 新增至連絡人清單，讓人們可以在 Skype 中與其互動。 

通道包含許多熱門服務，例如 [Cortana](bot-service-channel-connect-cortana.md)、[Facebook Messenger](bot-service-channel-connect-facebook.md)、[Kik](bot-service-channel-connect-kik.md) 和 [Slack](bot-service-channel-connect-slack.md)，以及數個其他服務。 已為您預先設定 [Skype](https://dev.skype.com/bots) 和網路聊天。 

在 [Azure 入口網站](https://portal.azure.com)中即可快速又容易地連線到通道。

## <a name="get-started"></a>開始使用

針對大部分的通道，您必須提供通道設定資訊，才能在通道上執行 Bot。 大部分的通道會要求 Bot 要有通道帳戶，Facebook Messenger 等其他通道則會要求 Bot 也要有已向通道註冊的應用程式。

若要設定 Bot 以連線到通道，請完成下列步驟：

1. 登入 <a href="https://portal.azure.com" target="_blank">Azure 入口網站</a>。
1. 選取您想要設定的 Bot。
3. 在 [Bot Service] 刀鋒視窗中，按一下 [Bot 管理] 底下的 [通道]。
4. 按一下想要在 Bot 中新增的通道圖示。

![連線到通道](~/media/channels/connect-to-channels.png)

設定通道後，該通道上的使用者就可以開始使用 Bot。

## <a name="publish-a-bot"></a>發佈 Bot

每個通道的發佈程序不同。

[!INCLUDE [publishing](~/includes/snippet-publish-to-channel.md)]

