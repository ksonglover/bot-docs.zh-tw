---
title: 適用於 .NET 的 Bot Framework SDK 重要概念 | Microsoft Docs
description: 了解建置和部署適用於 .NET 的 Bot Framework SDK 中可用交談式 Bot 所需的重要概念和工具。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: e01473a06a0cdbef635de33e5734b02351e36cea
ms.sourcegitcommit: b15cf37afc4f57d13ca6636d4227433809562f8b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/11/2019
ms.locfileid: "54223803"
---
# <a name="key-concepts-in-the-bot-framework-sdk-for-net"></a>適用於 .NET 的 Bot Framework SDK 重要概念

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-concepts.md)
> - [Node.js](../nodejs/bot-builder-nodejs-concepts.md)

本文介紹適用於 .NET 的 Bot Framework SDK 重要概念。

## <a name="connector"></a>連接器

[Bot Framework Connector](bot-builder-dotnet-connector.md) 提供單一 REST API，讓 Bot 能夠跨多個通道 (例如 Skype、電子郵件、Slack 等) 進行通訊。 它可藉由將訊息從 Bot 轉送至通道以及從通道轉送至 Bot，讓 Bot 和使用者之間的通訊更為順暢。 

在適用於 .NET 的 Bot Framework SDK 中，[Connector][connectorLibrary] 程式庫可讓您存取 Connector。 

## <a name="activity"></a>活動

[!INCLUDE [Activity concept overview](../includes/snippet-dotnet-concept-activity.md)]

如需適用於 .NET 的 Bot Framework SDK 中的活動詳細資訊，請參閱[活動概觀](bot-builder-dotnet-activities.md)。

## <a name="dialog"></a>對話

當您使用適用於 .NET 的 Bot Framework SDK 建立 Bot 時，您可以使用[對話](bot-builder-dotnet-dialogs.md)來建立交談模型及管理[交談流程](../bot-service-design-conversation-flow.md#dialog-stack)。 對話可由其他對話所組成以便充分地重複使用，而且對話內容會維護[對話的堆疊](../bot-service-design-conversation-flow.md)，這些對話在對話的任何時間點均處於作用中狀態。 由對話 \(dialog\) 組成的對話 \(conversation\) 可移植到不同電腦，這使得您的 Bot 實作能夠加以調整。 

在適用於 .NET 的 Bot Framework SDK 中，[Builder][builderLibrary] 程式庫可讓您管理對話。

## <a name="formflow"></a>FormFlow

您可以使用適用於 .NET 的 Bot Framework SDK 中的 [FormFlow](bot-builder-dotnet-formflow.md)，簡化建置 Bot 以向使用者收集資訊的程序。 例如，接受三明治訂單的 Bot 必須向使用者收集數項資訊，例如麵包類型、配料的選擇、大小等等。 根據基本指導方針，FormFlow 可以自動產生管理這類引導式對話 \(conversation\) 所需的對話 \(dialog\)。

## <a name="state"></a>State

[!INCLUDE [State concept overview](../includes/snippet-dotnet-concept-state.md)]

如需使用適用於 .NET 的 Bot Framework SDK 管理狀態的詳細資訊，請參閱[管理狀態資料](bot-builder-dotnet-state.md)。

## <a name="naming-conventions"></a>命名慣例

適用於 .NET 的 Bot Framework SDK 程式庫會使用強類型並依照 Pascal 命名法的大小寫慣例。 不過，透過網路來回傳輸的 JSON 訊息會使用駝峰式大小寫命名慣例。 例如，C# 屬性 **ReplyToId** 會在透過網路傳輸的 JSON 訊息中序列化為 **replyToId**。

## <a name="security"></a>安全性

您應該確定 Bot 的端點只能由 Bot Framework Connector 服務加以呼叫。 如需有關本主題的詳細資訊，請參閱[保護 Bot](bot-builder-dotnet-security.md)。

## <a name="next-steps"></a>後續步驟

現在您已了解每個 Bot 背後的概念。 您可以使用範本，快速地[使用 Visual Studio 建置 Bot](bot-builder-dotnet-quickstart.md)。 接下來，將從對話開始，更詳細地研究每個重要概念。

> [!div class="nextstepaction"]
> [適用於 .NET 的 Bot Framework SDK 中的對話](bot-builder-dotnet-dialogs.md)

[connectorLibrary]: /dotnet/api/microsoft.bot.connector

[builderLibrary]: /dotnet/api/microsoft.bot.builder.dialogs
