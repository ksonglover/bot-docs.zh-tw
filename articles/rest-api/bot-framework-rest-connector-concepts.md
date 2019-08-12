---
title: Bot 連接器服務和 Bot 狀態服務的重要概念 | Microsoft Docs
description: 了解 Bot Framework 的 Bot 連接器服務和 Bot 狀態服務的重要概念。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 01/16/2019
ms.openlocfilehash: a2fa3f5e1363cb155504cdf903df2f6c25877af3
ms.sourcegitcommit: a1eaa44f182a7210197bd793250907df00e9edab
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/03/2019
ms.locfileid: "68756954"
---
# <a name="key-concepts"></a>重要概念

您可以使用 Bot 連接器服務跨多個通道 (例如 Skype、電子郵件、Slack 等等) 與使用者通訊。 本文介紹 Bot 連接器服務中的重要概念。

## <a name="bot-connector-service"></a>Bot 連接器服務

Bot 連接器服務可讓您的 Bot 和在 <a href="https://dev.botframework.com/" target="_blank">Bot Framework 入口網站</a> \(英文\) 中所設定的頻道交換訊息。 它使用業界標準的 HTTPS 上的 REST 和 JSON，且支援使用 JWT Bearer 權杖驗證。 如需如何使用 Bot 連接器服務的詳細資訊，請參閱[驗證](bot-framework-rest-connector-authentication.md)和此節中的其餘文章。

### <a name="activity"></a>活動

Bot 連接器服務會藉由傳遞 `Activity` 物件，在 Bot 與通道 (使用者) 之間交換資訊。 最常見的活動類型是**訊息**，但是還有其他活動類型可用來將各種類型的資訊傳達給 Bot 或通道。 如需 Bot 連接器服務中 Activity 的詳細資訊，請參閱[活動概觀](bot-framework-rest-connector-activities.md)。

## <a name="bot-state-service"></a>Bot 狀態服務

Microsoft Bot Framework 狀態服務已於 2018 年 3 月 30 日淘汰。 之前，建置於 Azure Bot 服務或 Bot Builder SDK 的 Bot 可依預設連線至 Microsoft 所裝載的這項服務，以儲存 Bot 狀態資料。 Bot 將需要更新才能使用其本身的狀態儲存體。

## <a name="authentication"></a>Authentication

Bot 連接器服務支援使用 JWT 持有人權杖進行驗證。 如需如何驗證 Bot 傳送至 Bot Framework 連出要求、如何驗證 Bot 從 Bot Framework 接收的連入要求之詳細資訊，請參閱[驗證](bot-framework-rest-connector-authentication.md)。 

## <a name="client-libraries"></a>用戶端程式庫

Bot Framework 提供可用於在 C# 或 Node.js 中建置 Bot 的用戶端程式庫。 

- 若要使用 C# 建置 Bot，請使用[適用於 C# 的 Bot Framework SDK](../dotnet/bot-builder-dotnet-overview.md)。 
- 若要使用 Node.js 建置 Bot，請使用[適用於 Node.js 的 Bot Framework SDK](../nodejs/index.md)。 

除了建構 Bot 連接器服務的模型，每個 Bot Framework SDK 也都提供強大的功能，可建置封裝交談邏輯的對話方塊、簡單項目的內建提示 (如是/否)、字串、數字和列舉、功能強大的 AI 架構 (如 <a href="https://www.luis.ai/" target="_blank">LUIS</a>) 的支援等等。 

> [!NOTE]
> 除了使用 C# SDK 或 Node.js SDK，您也可以改為使用 <a href="https://aka.ms/connector-swagger-file" target="_blank">Bot 連接器 Swagger 檔案</a>，以您所選的語言產生自己的用戶端程式庫。

## <a name="additional-resources"></a>其他資源

透過檢閱本節中的各文章，深入了解如何使用 Bot 連接器服務，從[驗證](bot-framework-rest-connector-authentication.md)開始。 如果您遇到問題，或有關於 Bot 連接器服務的建議，請參閱[支援](../bot-service-resources-links-help.md)以取得可用資源清單。 
