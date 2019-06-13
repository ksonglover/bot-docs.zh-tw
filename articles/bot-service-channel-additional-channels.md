---
title: 其他通道 | Microsoft Docs
description: 了解如何為 Bot 設定其他通道。
keywords: bot 通道, hangouts, Twilio, facebook, azure 入口網站
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 02/08/2019
ms.openlocfilehash: d6cf068f3cd4d57ff9cde2be6d41e084cee6524d
ms.sourcegitcommit: e276008fb5dd7a37554e202ba5c37948954301f1
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/05/2019
ms.locfileid: "66693738"
---
# <a name="additional-channels"></a>其他通道

除了這些文件中所列的通道外，還有一些可作為配接器的其他通道，都可藉由我們[提供的平台](https://botkit.ai/docs/v4/platforms/)來透過 Botkit 使用，或者透過[社群存放庫](https://github.com/BotBuilderCommunity/)來存取。 以下是所提供的其他通道：

- [Webex Teams](https://botkit.ai/docs/v4/platforms/webex.html)
- [Websocket 和 Webhook](https://botkit.ai/docs/v4/platforms/web.html)
- [Google Hangouts 與 Google 助理](https://github.com/BotBuilderCommunity/) (可透過社群取得)
- [Amazon Alexa](https://github.com/BotBuilderCommunity/) (可透過社群取得)

## <a name="currently-available-adapters"></a>目前可用的配接器

您可以在[這裡找到](https://botkit.ai/docs/v4/platforms/)完整的可用配接器清單。 您會發現某些通道也可作配接器使用，而您可以自行決定使用通道和配接器的時機。

### <a name="when-to-use-an-adapter"></a>使用配接器的時機

1. 服務不支援您想要的通道
2. 部署的安全性與合規性需求指出您不能依賴外部服務
3. 您在特定通道中需要的功能深度未受到支援

### <a name="when-to-use-a-channel"></a>使用通道的時機

1. 您需要跨通道的相容性：您的 Bot 應作用於一個以上的可用通道
2. 內建支援：每當第三方進行更新時，Microsoft 會為您維護、修補及無縫地處理每個通道
3. 可讓您存取更多專屬的 Microsoft 通路，例如快速成長的 Microsoft Teams
4. 如果您想要依賴 GUI 介面來為 Bot 啟用其他通道
