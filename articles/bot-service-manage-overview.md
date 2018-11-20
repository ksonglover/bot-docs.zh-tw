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
ms.date: 12/13/2017
ms.openlocfilehash: e68db6c1fe9d3d136a8643652df034fb6df2858f
ms.sourcegitcommit: 8b7bdbcbb01054f6aeb80d4a65b29177b30e1c20
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 11/14/2018
ms.locfileid: "51645598"
---
# <a name="manage-a-bot"></a>管理 Bot

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

[應用程式設定] 刀鋒視窗中包含關於 Bot 的詳細資訊，例如 Bot 的環境識別碼、Application Insights 金鑰、Microsoft 應用程式識別碼和 Microsoft 應用程式密碼。

### <a name="microsoftappid-and-microsoftapppassword"></a>MicrosoftAppID 和 MicrosoftAppPassword

您可以在 [設定] 刀鋒視窗中找到 Bot 的 **MicrosoftAppID**。 Bot 的 **MicrosoftAppPassword** 只會在您第一次建立 Bot 時顯示。

![Microsoft 應用程式識別碼和密碼](~/media/azure-manage-a-bot/app-settings.png)

> [!NOTE]
> **Bot 通道註冊** Bot 服務隨附 *MicrosoftAppID*，但因為沒有應用程式服務與這種類型的服務建立關聯，所以沒有任何 [應用程式設定] 刀鋒視窗可供您查閱 *MicrosoftAppPassword*。 若要取得密碼，您必須先產生密碼。 若要產生 **Bot 通道註冊**的密碼，請參閱 [Bot 通道註冊密碼](bot-service-quickstart-registration.md#bot-channels-registration-password)

## <a name="next-steps"></a>後續步驟
由於您已在 Azure 入口網站中探索 Bot 服務刀鋒視窗，請了解如何使用線上程式碼編輯器來自訂 Bot。
> [!div class="nextstepaction"]
> [使用線上程式碼編輯器](bot-service-build-online-code-editor.md)
