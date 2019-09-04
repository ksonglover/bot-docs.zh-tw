---
title: Bot Service 案例概觀 | Microsoft Docs
description: 針對使用 Bot Service 所建置的強大且成功 Bot，探索其重要案例。
author: BrianRandell
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: c4a0af8bfd6496b82ddcc9219e948395d87e4ff3
ms.sourcegitcommit: eacf1522d648338eebefe2cc5686c1f7866ec6a2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/30/2019
ms.locfileid: "70167348"
---
# <a name="bot-scenarios"></a>Bot 案例

[!INCLUDE [pre-release-label](includes/pre-release-label-v3.md)]

本主題會針對使用 Bot Service 所建置強大且成功的 Bot，探索其重要案例。

您可以從[常見的 Bot Framework 案例範例](https://aka.ms/abs-scenarios)下載或複製所有案例 Bot 範例的原始程式碼。

## <a name="commerce-bot-scenario"></a>商務 Bot 案例
[商務 Bot](bot-service-scenario-commerce.md) 案例說明 Bot 如何取代傳統電子郵件和電話互動等常見的旅館接待服務。 Bot 會利用認知服務，以更順暢地處理客戶的文字和語音要求，並搭配蒐集來自與後端服務整合的內容。

在商務 Bot 案例中，客戶可以向旅館提出門房服務要求。 其會透過 Azure Active Directory v2 驗證端點進行驗證。 Bot 可以查閱客戶的預約，並提供不同服務選項。 例如，客戶可能預訂泳池畔的小屋。 Bot 會使用 Language Understanding Intelligent Services (LUIS) 剖析要求，然後 Bot 會逐步引導使用者進行針對現有預約，來預訂小屋的程序。

## <a name="cortana-skill-bot-scenario"></a>Cortana 技能 Bot 案例
[Cortana 技能 Bot](bot-service-scenario-cortana-skill.md) 案例會利用 Cortana。 使用語音的自然介面和自訂的 Cortana 技能 Bot，您可以讓 Cortana 與組織 (如行動汽車美容公司) 交談，以協助您根據撥打電話時的所在地進行預約。 此 Bot 可以提供一份服務、可用時間和持續時間的清單。

## <a name="enterprise-productivity-bot-scenario"></a>企業生產力 Bot 案例
[企業生產力 Bot](bot-service-scenario-enterprise-productivity.md) 案例會說明如何整合 Bot 與 Office 365 行事曆和其他服務，以提高生產力。

Bot 會與 Office 365 整合，以便更快速且更輕鬆地建立與其他人開會的要求。 在這麼做的過程中，您可以存取 Dynamics CRM 等其他服務。 此範例會提供所需程式碼，以供透過 Azure Active Directory 進行驗證來與 Office 365 整合。 它會為外部服務提供模擬進入點，以作為讀者的練習。

## <a name="information-bot-scenario"></a>資訊 Bot 案例
[資訊 Bot](bot-service-scenario-informational.md) 可使用認知服務 QnA Maker 回答在知識集或常見問題集中定義的問題，也會使用 Azure 搜尋服務回答更開放式的問題。

通常資訊會藏在 SQL Server 等結構化的資料存放區，可以透過搜尋輕鬆地呈現。 想像一下使用簡單的交談式命令查閱客戶的訂單狀態。 使用認知服務 QnA Maker，使用者會看到一組有效的搜尋選項，例如查閱客戶、檢閱客戶的最新訂單等。使用者可以利用 QnA 格式定義，輕鬆詢問 Azure 搜尋支援的問題，並在 SQL Database 中查閱儲存的資料。

## <a name="iot-bot-scenario"></a>IoT Bot 案例
這個[物聯網 (IoT) Bot](bot-service-scenario-internet-things.md) 可讓您輕鬆地使用互動式聊天命令控制住家內的裝置，例如 Philips Hue 燈光。

您可以使用這個簡單的 Bot，搭配免費的 If This Then That (IFTTT) 服務來控制 Philips Hue 燈光。 身為 IoT 裝置，Philips Hue 可透過其公開的 API 在本機進行控制。 不過，此 API 並未公開用於從本機網路外部所進行的一般存取。 不過，IFTTT 是 "[Friend of Hue](http://www2.meethue.com/friends-of-hue/ifttt/)"，因此，已公開多個您可以發出的控制命令，例如，開燈和關燈、變更燈色或光源強度。

## <a name="next-steps"></a>後續步驟
您現已大致了解各種案例，接下來請深入了解每個案例。

> [!div class="nextstepaction"]
> [商務 Bot](bot-service-scenario-commerce.md)
