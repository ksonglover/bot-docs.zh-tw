---
title: 將 Bot 設定為在一或多個通道上執行 | Microsoft Docs
description: 了解如何使用 Bot Framework 入口網站，將 Bot 設定為在一或多個通道上執行。
keywords: Bot 通道, 設定, cortana, facebook messenger, kik, slack, skype, azure 入口網站
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 09/22/2018
ms.openlocfilehash: 0430562fd615aef67b4ba95538d390cf2223fb45
ms.sourcegitcommit: c6ce4c42fc56ce1e12b45358d2c747fb77eb74e2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/22/2019
ms.locfileid: "54453902"
---
# <a name="connect-a-bot-to-channels"></a>將 Bot 連線至通道

通道可連接 Bot 與通訊應用程式。 您可以將 Bot 設定為連線到您想要提供此 Bot 的通道。 透過 Azure 入口網站設定的 Bot Framework Service，可將您的 Bot 連接至這些通道，以利 Bot 和使用者之間的通訊。 您可以連線至許多熱門服務，例如 [Cortana](bot-service-channel-connect-cortana.md)、[Facebook Messenger](bot-service-channel-connect-facebook.md)、[Kik](bot-service-channel-connect-kik.md) 和 [Slack](bot-service-channel-connect-slack.md)，以及數個其他服務。 已為您預先設定 [Skype](https://dev.skype.com/bots) 和網路聊天。 除了 Bot Connector Service 所提供的標準通道，您也可以使用 Direct Line 作為通道，將 Bot 連接到您自己的用戶端應用程式。

藉由將 Bot 傳送給通道的訊息正規化，Bot Framework Service 可讓您以不限通道的方式開發 Bot。 這牽涉到從 Bot 架構結構描述轉換成通道的結構描述。 不過，如果通道不支援 Bot 架構結構描述的所有層面，則服務會嘗試將訊息轉換成通道支援的格式。 比方說，如果 Bot 將訊息傳送至電子郵件通道，而訊息中包含具有動作按鈕的卡片，則連接器可能會將卡片傳送為映像，並在訊息文字中將動作包含為連結。


針對大部分的通道，您必須提供通道設定資訊，才能在通道上執行 Bot。 大部分的通道會要求 Bot 要有通道帳戶，Facebook Messenger 等其他通道則會要求 Bot 也要有已向通道註冊的應用程式。

若要設定 Bot 以連線到通道，請完成下列步驟：

1. 登入 <a href="https://portal.azure.com" target="_blank">Azure 入口網站</a>。
1. 選取您想要設定的 Bot。
3. 在 [Bot Service] 刀鋒視窗中，按一下 [Bot 管理] 底下的 [通道]。
4. 按一下想要在 Bot 中新增的通道圖示。

![連線到通道](./media/channels/connect-to-channels.png)

設定通道後，該通道上的使用者就可以開始使用 Bot。

## <a name="publish-a-bot"></a>發佈 Bot

每個通道的發佈程序不同。

[!INCLUDE [publishing](./includes/snippet-publish-to-channel.md)]

## <a name="additional-resources"></a>其他資源
SDK 包含您可用來建置 Bot 的範例。 請瀏覽 [GitHub](https://github.com/Microsoft/BotBuilder-samples) 存放庫，以查看範例清單。
