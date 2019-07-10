---
title: 將 Bot 連線至 Teams | Microsoft Docs
description: 了解如何配置 Bot，以透過 Team 進行存取。
keywords: Team, Bot 通道, 設定 Teams
author: kaiqb
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 07/05/2019
ms.openlocfilehash: d2609e4294416691e156ba3dbd09eabc8e0d3423
ms.sourcegitcommit: b498649da0b44f073dc5b23c9011ea2831edb31e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/05/2019
ms.locfileid: "67592227"
---
# <a name="connect-a-bot-to-teams"></a>將 Bot 連線至 Teams

若要新增 Microsoft Teams 通道，請在 [Azure 入口網站](https://portal.azure.com)中開啟 Bot，按一下 [通道]  刀鋒視窗，然後按一下 [Teams]  。

![新增 Teams 通道](media/teams/connect-teams-channel.png)

接著，按一下 [儲存]  。

![儲存 Teams 通道](media/teams/save-teams-channel.png)

新增 Teams 通道後，請移至**通道**頁面，然後按一下**取得 Bot 內嵌程式碼**。

![取得內嵌程式碼](media/teams/get-embed-code.png)

- 在**取得 Bot 內嵌程式碼**對話方塊中，複製所顯示程式碼 _https_ 的部分。 例如： `https://teams.microsoft.com/l/chat/0/0?users=28:b8a22302e-9303-4e54-b348-343232` 。 

- 在瀏覽器中，貼上此位址，然後選擇您用來將 Bot 新增至 Teams 的 Microsoft Teams 應用程式 (用戶端或 Web)。 您應該能夠在 Microsoft Teams 中，看到列為聯絡人的 Bot，並且可以向其傳送訊息和接收訊息。 

## <a name="additional-information"></a>其他資訊
如需 Microsoft Teams 的特定資訊，請參閱 Teams [文件](https://docs.microsoft.com/en-us/microsoftteams/platform/overview)。 
