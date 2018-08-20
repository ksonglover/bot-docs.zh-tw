---
title: 通道和 Bot Connector 服務 | Microsoft Docs
description: Bot 建立器 SDK 的重要概念。
keywords: 活動, 對話
author: jonathanfingold
ms.author: jonathanfingold
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 03/28/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: abb2b18d45fb722d7de98c5cd02bd88350aad803
ms.sourcegitcommit: 9a38d76afb0e82fdccc1f36f9b1a65042671e538
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/04/2018
ms.locfileid: "39515088"
---
# <a name="channels-and-the-bot-connector-service"></a>通道和 Bot Connector 服務

[!INCLUDE [pre-release-label](~/includes/pre-release-label.md)]

通道是指使用者連線至 Bot 的端點，例如 Facebook、Skype、電子郵件、Slack 等。透過 [Azure 入口網站](https://portal.azure.com)設定的 Bot Connector 服務，可將您的 Bot 連接至這些通道，以利 Bot 和使用者之間的通訊。 

除了 Bot Connector 服務所提供的標準通道，您也可以使用 [Direct Line](bot-builder-howto-direct-line.md) 作為通道，將 Bot 連接到您自己的用戶端應用程式。

藉由將 Bot 傳送給通道的訊息正規化，Bot Connector 服務可讓您以不限通道的方式開發 Bot。 這牽涉到從 Bot 建立器結構描述轉換成通道的結構描述。 不過，如果通道不支援 Bot 建立器結構描述的所有層面，則服務會嘗試將訊息轉換成通道支援的格式。 比方說，如果 Bot 將訊息傳送至 SMS 通道，而訊息中包含具有動作按鈕的卡片，則連接器可能會將卡片傳送為映像，並在訊息文字中將動作包含為連結。

## <a name="activities-and-conversations"></a>活動和對話


Bot Connector 服務會使用 JSON 來交換 Bot 和使用者之間的資訊，且 Bot 建立器 SDK 會以語言特定的活動物件來包裝這項資訊。 討論[與您的 Bot 互動](bot-builder-basics.md#interaction-with-your-bot)時會提到[活動類型](../bot-service-activities-entities.md)，最常見的活動類型是訊息，但其他活動類型也很重要。 類型包括對話更新、連絡人關係更新、刪除使用者資料、結束對話、輸入、訊息回應，和一些使用者不一定看過的其他 Bot 特定活動。 每一類型和類型出現時機的詳細資料，可在活動參考內容中找到。

每個環節和其相關聯的活動都屬於邏輯**對話**，可代表一或多個 Bot 和一個特定使用者或一群使用者之間的互動。 對話會特定於通道，且具有該通道上唯一的識別碼。

## <a name="next-steps"></a>後續步驟

現在您已熟悉 Bot 的一些重要概念，讓我們深入了解 Bot 可能使用的不同對話形式。

> [!div class="nextstepaction"]
> [對話形式](bot-builder-conversations.md)
