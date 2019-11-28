---
title: 將 Bot 連結到 Office 365 電子郵件 | Microsoft Docs
description: 了解如何設定 Bot 以使用 Office 365 傳送及接收電子郵件。
keywords: Office 365, bot channels, email, email credentials, azure portal, custom email, Office 365, Bot 通道, 電子郵件, 電子郵件認證, azure 入口網站, 自訂電子郵件
author: kamrani
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 11/15/2019
ms.openlocfilehash: 3ad7829af9dc24ca9e10ebab352b4513b3bcad28
ms.sourcegitcommit: 78da125eafe93b3c6d254708b82bdd01de7c3afb
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 11/25/2019
ms.locfileid: "74464282"
---
# <a name="connect-a-bot-to-office-365-email"></a>將 Bot 連結到 Office 365 電子郵件

除了其他[通道](~/bot-service-manage-channels.md)之外，Bot 還可透過 Office 365 電子郵件與使用者通訊。 當 Bot 設定為存取電子郵件帳戶時，它會在新的電子郵件送達時收到訊息。 然後，Bot 可以按照其商務邏輯進行回應。 例如，Bot 可以傳送電子郵件回覆，確認收到的電子郵件中有訊息「嗨！ 感謝您的訂單！ 我們將立即處理。」

> [!WARNING]
> 建立 "spambot" 違反 Bot Framework [管理辦法](https://www.botframework.com/Content/Microsoft-Bot-Framework-Preview-Online-Services-Agreement.htm)，包括傳送不需要或未經請求的大量電子郵件的 Bot。

> [!NOTE]
> 如果您使用的是 Microsoft Exchange Server，請先確定您已先啟用[自動探索](https://docs.microsoft.com/exchange/client-developer/exchange-web-services/autodiscover-for-exchange)，然後再設定電子郵件通道。 

## <a name="configure-email-credentials"></a>設定電子郵件認證

您可以透過在電子郵件通道設定中輸入 Office 365 認證，將 Bot 連結到電子郵件通道。
不支援使用取代 AAD 的任何供應商進行同盟驗證。

> [!NOTE]
> 您自己的個人電子郵件帳戶不應該用於 Bot，因為傳送至電子郵件帳戶的每個訊息都將轉送給 Bot。 這可能導致 Bot 不當地向寄件者傳送回應。 因此，Bot 應該只使用專用的 O365 電子郵件帳戶。

若要新增電子郵件通道，請在 [Azure 入口網站](https://portal.azure.com/)中開啟 Bot，按一下 [通道]  刀鋒視窗，然後按一下 [電子郵件]  。 輸入有效的電子郵件認證，然後按一下 [儲存]  。

![輸入電子郵件認證](~/media/bot-service-channel-connect-email/bot-service-channel-connect-email-credentials.png)

電子郵件通道目前僅適用於 Office 365。 請注意，目前不支援電子郵件服務。

## <a name="customize-emails"></a>自訂電子郵件

電子郵件通道支援使用 `channelData` 屬性傳送自訂的屬性，以建立更進階的自訂電子郵件。

[!INCLUDE [Email channelData table](~/includes/snippet-channelData-email.md)]

下列範例訊息顯示一個包含這些 `channelData` 屬性的 JSON 檔案。

```json
{
    "type": "message",
    "locale": "en-Us",
    "channelID": "email",
    "from": { "id": "mybot@mydomain.com", "name": "My bot"},
    "recipient": { "id": "joe@otherdomain.com", "name": "Joe Doe"},
    "conversation": { "id": "123123123123", "topic": "awesome chat" },
    "channelData":
    {
        "htmlBody": "<html><body style = /"font-family: Calibri; font-size: 11pt;/" >This is more than awesome.</body></html>",
        "subject": "Super awesome message subject",
        "importance": "high",
        "ccRecipients": "Yasemin@adatum.com;Temel@adventure-works.com"
    }
}
```

::: moniker range="azure-bot-service-3.0"
如需有關使用 `channelData` 的詳細資訊，請參閱 [Node.js](https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/core-ChannelData) 範例或 [.NET](~/dotnet/bot-builder-dotnet-channeldata.md) 文件。
::: moniker-end

::: moniker range="azure-bot-service-4.0"
如需有關使用 `channelData` 的詳細資訊，請參閱[如何實作通道專屬功能](~/v4sdk/bot-builder-channeldata.md)。
::: moniker-end

## <a name="other-considerations"></a>其他考量

如果 Bot 未在 15 秒內傳回 200 OK HTTP 狀態碼以回應傳入的電子郵件訊息，電子郵件通道將會嘗試重新傳送訊息，因此 Bot 可能會收到相同的電子郵件訊息活動好幾次。 如需詳細資訊，請參閱 **Bot 的運作方式**中的 [HTTP 詳細資料](v4sdk/bot-builder-basics.md#http-details)一節，以及如何[未逾時錯誤進行疑難排解](https://github.com/daveta/analytics/blob/master/troubleshooting_timeout.md)一文。

## <a name="additional-resources"></a>其他資源

<!-- Put whole list in monikers, even though it's just the second item that needs to be different. -->
::: moniker range="azure-bot-service-3.0"
* 將 Bot 連結到[通道](~/bot-service-manage-channels.md)
* 使用適用於 .NET 的 Bot Framework SDK [實作通道特定功能](dotnet/bot-builder-dotnet-channeldata.md)
* 使用[通道偵測器](bot-service-channel-inspector.md)查看通道如何轉譯 Bot 應用程式的特定功能
::: moniker-end
::: moniker range="azure-bot-service-4.0"
* 將 Bot 連結到[通道](~/bot-service-manage-channels.md)
* 使用適用於 .NET 的 Bot Framework SDK [實作通道特定功能](~/v4sdk/bot-builder-channeldata.md)
* 使用[通道偵測器](bot-service-channel-inspector.md)查看通道如何轉譯 Bot 應用程式的特定功能
::: moniker-end
