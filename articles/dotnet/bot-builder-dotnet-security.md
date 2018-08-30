---
title: 保護您的 Bot | Microsoft Docs
description: 了解如何使用 HTTPS 與 Bot Framework 驗證來保護您的 Bot。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/17
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 376982396ac11cbcf54f26255235b3779e0e1c26
ms.sourcegitcommit: 2dc75701b169d822c9499e393439161bc87639d2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/24/2018
ms.locfileid: "42905710"
---
# <a name="secure-your-bot"></a>保護 Bot

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

透過 Bot Framework 連接器服務，Bot 可以連線到許多不同的通訊通道 (Skype、SMS、電子郵件等其他項目)。 本文將說明如何使用 HTTPS 與 Bot Framework 驗證來保護您的 Bot。

## <a name="use-https-and-bot-framework-authentication"></a>使用 HTTPS 和 Bot Framework 驗證

若要確保您的 Bot 端點僅能透過 Bot Framework [連接器](bot-builder-dotnet-concepts.md#connector)來存取，請將 Bot 的端點設定為只能使用 HTTPS，並[註冊](~/bot-service-quickstart-registration.md) Bot 來取得其 AppID 和密碼，以啟用 Bot Framework 驗證。 

## <a name="configure-authentication-for-your-bot"></a>設定 Bot 的驗證

在 Bot 的 web.config 檔案中指定 Bot AppID 和密碼。 

> [!NOTE]
> 若要尋找 Bot 的 **AppID** 和 **AppPassword**，請參閱 [MicrosoftAppID 和 MicrosoftAppPassword](~/bot-service-manage-overview.md#microsoftappid-and-microsoftapppassword)。

```xml
<appSettings>
    <add key="MicrosoftAppId" value="_appIdValue_" />
    <add key="MicrosoftAppPassword" value="_passwordValue_" />
</appSettings>
```

然後，在使用適用於 .NET 的 Bot 建立器 SDK 來建立 Bot 時，使用 `[BotAuthentication]` 屬性來指定驗證認證。 

若要使用儲存在 web.config 檔案中的驗證認證，請指定 `[BotAuthentication]` (不含任何參數)。

[!code-csharp[Use botAuthentication attribute](../includes/code/dotnet-security.cs#attribute1)]

若要使用其他值作為驗證認證，請指定 `[BotAuthentication]` 屬性，並傳入那些值。

[!code-csharp[Use botAuthentication attribute with parameters](../includes/code/dotnet-security.cs#attribute2)]

## <a name="additional-resources"></a>其他資源

- [適用於 .NET 的 Bot 建立器 SDK](bot-builder-dotnet-overview.md)
- [適用於 .NET 的 Bot 建立器 SDK 重要概念](bot-builder-dotnet-concepts.md)
- [使用 Bot Framework 註冊 Bot](~/bot-service-quickstart-registration.md)
