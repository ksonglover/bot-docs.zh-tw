---
title: 將 Bot 連線至 Skype | Microsoft Docs
description: 了解如何配置 Bot，以透過 Skpye 介面存取。
keywords: skype, Bot 頻道, 設定 skype, 發佈, 連接到頻道
author: v-ducvo
ms.author: RobStand
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 2/1/2018
ms.openlocfilehash: 5dc4063125855113f813b8873b01df84c90e197e
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/18/2018
ms.locfileid: "39299194"
---
# <a name="connect-a-bot-to-skype"></a>將 Bot 連線至 Skype

Skype 可讓您透過立即訊息、 電話和視訊通話連接使用者。 藉由建置使用者可以透過 Skype 介面探索與互動的 Bot 來擴充這項功能。

若要新增 Skype 通道，請在 [Azure 入口網站](https://portal.azure.com/)中開啟 Bot，按一下 [**通道**] 刀鋒視窗，然後按一下 [**Skype**]。 系統會將您引導至 [**配置 Skype**] 設定頁面。 填妥關於 Bot 的所有必要資訊，然後按一下 [**儲存**] 以連接 Skype 頻道。 接受**服務條款**，Skype 頻道將會新增至您的 Bot。

![新增 Skype 頻道](~/media/channels/skype-addchannel.png)

## <a name="web-control"></a>Web 控制項

若要將 Bot 嵌入您的網站，您可以按一下 [**Web 控制項**] 區段中的 [**取得內嵌程式碼**] 按鈕。

## <a name="messaging"></a>訊息

此區段會設定 Bot 如何傳送並接收 Skype 中的訊息。

## <a name="calling"></a>呼叫

此區段會在 Bot 中配置 Skype 的呼叫功能。 您可以指定是否要為 Bot 啟動 [**呼叫**]，以及啟用後是否要使用 IVR 功能或即時媒體功能。

## <a name="groups"></a>群組

此區段會設定是否可以將 Bot 加入群組以及其在傳訊群組中的行為，並且也會用來為呼叫 Bot 啟用群組視訊通話。

## <a name="publish"></a>發佈

此區段會配置 Bot 的發佈設定。 所有標示「*」的欄位，皆為必填欄位。

在**預覽**中的 Bot 限制為 100 名連絡人。 如果您需要 100 位以上的連絡人，請提交 Bot 進行審查。 按一下 [**提交以供審查**] 會在接受後自動讓您的 Bot 可在 Skype 中搜尋。 如果您的要求未通過核准，則會通知您需要變更才能通過核准的項目。

## <a name="next-steps"></a>後續步驟

* [商務用 Skype](bot-service-channel-connect-skypeforbusiness.md)
