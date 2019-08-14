---
title: 將 Bot 連線至直接線路 | Microsoft Docs
description: 了解如何配置 Bot 與直接線路的連線。
keywords: 直接線路, bot 通道, 自訂用戶端, 連線到通道, 設定
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 08/7/2019
ms.openlocfilehash: edfb61a4f4ca33089bce7d4b44ed242f83cbcc0d
ms.sourcegitcommit: 6a83b2c8ab2902121e8ee9531a7aa2d85b827396
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/09/2019
ms.locfileid: "68866628"
---
# <a name="connect-a-bot-to-direct-line"></a>將 Bot 連線至直接線路

您可以使用直接線路通道，讓用戶端應用程式與 Bot 進行通訊。 

## <a name="add-the-direct-line-channel"></a>新增直接線路通道

若要新增直接線路通道，請在 [Azure 入口網站](https://portal.azure.com/)中開啟 Bot，並按一下 [**通道**] 刀鋒視窗，然後按一下 [**直接線路**]。

![新增直接線路通道](media/bot-service-channel-connect-directline/directline-addchannel.png)

## <a name="add-new-site"></a>新增網站

接下來，新增網站以代表您想要連接到 bot 的用戶端應用程式。 按一下 [**新增網站**]，輸入網站名稱，然後按一下 [**完成**]。

![新增直接線路網站](media/bot-service-channel-connect-directline/directline-addsite.png)

## <a name="manage-secret-keys"></a>管理祕密金鑰

當您建立網站時，Bot Framework 會產生祕密金鑰，用戶端應用程式可藉此來[驗證](~/rest-api/bot-framework-rest-direct-line-3-0-authentication.md)其所發出的直接線路 API 要求，以便與 bot 進行通訊。 若要檢視純文字金鑰，按一下 [**顯示**] 以取得對應的索引鍵。

![顯示直接線路金鑰](media/bot-service-channel-connect-directline/directline-showkey.png)

複製並安全地儲存顯示的金鑰。 然後使用此金鑰來[驗證](~/rest-api/bot-framework-rest-direct-line-3-0-authentication.md)您的用戶端發出的直接線路 API 要求，以便與 Bot 進行通訊。
或者，使用直接線路 API [將金鑰交換成權杖](~/rest-api/bot-framework-rest-direct-line-3-0-authentication.md#generate-token)，您的用戶端便可用來驗證單一交談範圍內的後續要求。

![複製直接線路金鑰](media/bot-service-channel-connect-directline/directline-copykey.png)

## <a name="configure-settings"></a>配置設定

最後，配置網站的設定。

- 選取您的用戶端應用程式會用來與 Bot 進行通訊的直接線路通訊協定版本。

> [!TIP]
> 如果要在用戶端應用程式與 Bot 之間建立新連線，請使用 直接線路 API 3.0。

完成後，按一下 [**完成**] 以儲存網站設定。 您可以重複此程序，從為您想要連線至 Bot 的每個用戶端應用程式的 [[新增網站](#add-new-site)] 開始。
