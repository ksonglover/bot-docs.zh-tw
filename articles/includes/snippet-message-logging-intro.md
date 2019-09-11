---
ms.openlocfilehash: 12ead266dea859c84450e08ae12ed98d4952d698
ms.sourcegitcommit: fa6e775dcf95a4253ad854796f5906f33af05a42
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/16/2019
ms.locfileid: "70297375"
---
Bot Framework SDK 中的**中介軟體**功能可讓 Bot 攔截使用者與 Bot 之間交換的所有訊息。 針對每個攔截的訊息，您可以選擇執行如下的動作：將訊息儲存到您所指定的資料存放區，以建立對話記錄，或是以某種方式檢查訊息，並執行程式碼所指定的任何動作。 

> [!NOTE]
> Bot Framework 不會自動儲存交談的詳細資料，因此這麼做可能會擷取 Bot 和使用者不想要與外界共用的私人資訊。 如果您的 Bot 儲存了對話的詳細資料，它即應將此情況告知使用者，並說明這些資料的使用方式。