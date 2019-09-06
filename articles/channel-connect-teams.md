---
title: 將 Bot 連線至 Teams | Microsoft Docs
description: 了解如何配置 Bot，以透過 Team 進行存取。
keywords: Team, Bot 通道, 設定 Teams
author: kamrani
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 08/26/2019
ms.openlocfilehash: 1ba0873c880bfb9b1e0f449039e98859e5571028
ms.sourcegitcommit: 0b647dc6716b0c06f04ee22ebdd7b53039c2784a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/28/2019
ms.locfileid: "70076547"
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

> [!IMPORTANT] 
> 建議您不要針對測試以外的任何用途使用 GUID 新增 Bot。 這麼做會嚴重限制 Bot 的功能。 生產環境中的 Bot 應會隨著應用程式新增至 Teams。 請參閱[建立 Bot](https://docs.microsoft.com/microsoftteams/platform/concepts/bots/bots-create) 和[測試及偵錯您的 Microsoft Teams Bot](https://docs.microsoft.com/microsoftteams/platform/concepts/bots/bots-test)。


## <a name="additional-information"></a>其他資訊
如需 Microsoft Teams 的特定資訊，請參閱 Teams [文件](https://docs.microsoft.com/en-us/microsoftteams/platform/overview)。 
