---
title: 管理 Bot |Microsoft Docs
description: 了解如何透過 Bot 服務入口網站管理 Bot。
keywords: Azure 入口網站, Bot 管理, 在網路聊天中測試, MicrosoftAppID, MicrosoftAppPassword, 應用程式設定
author: v-ducvo
ms.author: rstand
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: abs
ms.date: 4/13/2019
ms.openlocfilehash: f3e0ac52a3bfe5759202af6c704626acafef617b
ms.sourcegitcommit: 4086189a9c856fbdc832eb1a1d205e5f1b4e3acd
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/16/2019
ms.locfileid: "65733307"
---
# <a name="manage-a-bot"></a>管理 Bot

[!INCLUDE [applies-to-both](includes/applies-to-both.md)]

本主題說明如何使用 Azure 入口網站管理 Bot。

## <a name="bot-settings-overview"></a>Bot 設定概觀

![Bot 設定概觀](~/media/azure-manage-a-bot/overview.png)

在 [概觀] 刀鋒視窗中，您可以找到關於 Bot 的高階資訊。 例如，您可以看到 Bot 的 [訂用帳戶 ID]、[定價層] 和 [傳訊端點]。

## <a name="bot-management"></a>Bot 管理

 您可以在＜**BOT 管理**＞一節下找到大部分的 Bot 管理選項。 以下是可協助您管理 Bot 的選項清單。

![Bot 管理](~/media/azure-manage-a-bot/bot-management.png)

| 選項 |  說明 |
| ---- | ---- |
| **建置** | [建置] 索引標籤可提供選項來變更 Bot。 此選項不適用於**僅限註冊 Bot**。 |
| **在網路聊天中測試** | 使用整合式網路聊天控制項來協助您快速測試 Bot。 |
| **分析** | 如果 Bot 已開啟分析，您可以檢視 Application Insights 針對該 Bot 收集的分析資料。 |
| **Channels** | 設定 Bot 用來與使用者進行通訊的通道。 |
| **設定** | 管理各種 Bot 設定檔設定，例如顯示名稱、分析和傳訊端點。 |
| **語音預備** | 管理 LUIS 應用程式與 Bing 語音服務之間的連線。 |
| **Bot 服務定價** | 管理 Bot 服務的定價層。 |

## <a name="app-service-settings"></a>App service 設定

![App Service 設定](~/media/azure-manage-a-bot/app-service-settings.png)

[應用程式設定] 刀鋒視窗中包含關於 Bot 的詳細資訊，例如 Bot 的環境、偵錯設定和應用程式設定金鑰。

### <a name="microsoftappid-and-microsoftapppassword"></a>MicrosoftAppID 和 MicrosoftAppPassword

**MicrosoftAppID** 和 **MicrosoftAppPassword** 會保存在 Bot 的設定檔 (`appsettings.json` 或 `.env`) 或 Azure Key Vault 內。 若要擷取，請下載您 Bot 的設定檔或組態檔 (如果存在的話，適用於舊版 Bot)，或存取 Azure Key Vault。 您可能需要使用識別碼和密碼在本機進行測試。

> [!NOTE]
> **Bot 通道註冊** Bot 服務隨附 *MicrosoftAppID*，但因為沒有應用程式服務與這種類型的服務建立關聯，所以沒有任何 [應用程式設定] 刀鋒視窗可供您查閱 *MicrosoftAppPassword*。 若要取得密碼，您必須先產生密碼。 若要產生 **Bot 通道註冊**的密碼，請參閱 [Bot 通道註冊密碼](bot-service-quickstart-registration.md#bot-channels-registration-password)

## <a name="next-steps"></a>後續步驟
由於您已在 Azure 入口網站中探索 Bot 服務刀鋒視窗，請了解如何使用線上程式碼編輯器來自訂 Bot。
> [!div class="nextstepaction"]
> [使用線上程式碼編輯器](bot-service-build-online-code-editor.md)
