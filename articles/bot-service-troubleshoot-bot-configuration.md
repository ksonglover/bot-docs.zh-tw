---
title: 針對 Bot 設定問題進行疑難排解 | Microsoft Docs
description: 如何針對已部署 Bot 中的設定問題進行疑難排解。
keywords: 疑難排解, 設定, 網路聊天, 問題。
author: jonathanFingold
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 4/30/2019
ms.openlocfilehash: d71cfc604484b521450465c483201952d4672fcd
ms.sourcegitcommit: a6d02ec4738e7fc90b7108934740e9077667f3c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/04/2019
ms.locfileid: "70298618"
---
# <a name="troubleshoot-bot-configuration-issues"></a>針對 Bot 設定問題進行疑難排解

針對 Bot 進行疑難排解的第一個步驟，是在網路聊天中進行測試。 這可讓您判斷問題是否為 Bot (Bot 不適用於任何通道) 或特定通道 (Bot 適用於有些通道，但不適用於其他通道) 所特有。

## <a name="test-in-web-chat"></a>在網路聊天中測試

1. 在 [Azure 入口網站](http://portal.azure.com/)中開啟 Bot 資源。
1. 開啟 [在網路聊天中測試]  窗格。
1. 將訊息傳送給您的 Bot。

![在網路聊天中測試](./media/test-in-webchat.png)

如果 Bot 並未以預期的輸出回應，請移至 [Bot 不適用於網路聊天](#bot-does-not-work-in-web-chat)。 否則，移至 [Bot 適用於網路聊天，但不適用於其他通道](#bot-works-in-web-chat-but-not-in-other-channels)。

## <a name="bot-does-not-work-in-web-chat"></a>Bot 不適用於網路聊天

Bot 無法運作的可能原因很多。 最可能是 Bot 應用程式已關閉且無法接收訊息，或 Bot 收到訊息但無法回應。

若要查看 Bot 是否正在執行：

1. 開啟 [概觀]  窗格。
1. 複製 [訊息端點]  並將其貼到您的瀏覽器。

如果此端點傳回 HTTP 錯誤 405，則表示 Bot 可觸達且 Bot 能夠回應訊息。 您應該調查 Bot 是否[逾時](https://github.com/daveta/analytics/blob/master/troubleshooting_timeout.md)或[因為 HTTP 5xx 錯誤而失敗](bot-service-troubleshoot-500-errors.md)。

如果此端點傳回錯誤「此站台無法連線」或「無法觸達此頁面」，這表示 Bot 已關閉而且需要重新部署。

## <a name="bot-works-in-web-chat-but-not-in-other-channels"></a>Bot 適用於網路聊天，但不適用於其他通道

如果 Bot 如預期般在網路聊天中運作，但在某些其他通道中失敗，可能原因如下：

- [通道設定問題](#channel-configuration-issues)
- [通道特有的行為](#channel-specific-behavior)
- [通道中斷](#channel-outage)

### <a name="channel-configuration-issues"></a>通道設定問題

通道設定參數的設定可能不正確，或已從外部進行變更。 例如，Bot 已針對特定頁面設定 Facebook 通道，而此頁面後來遭到刪除。 最簡單的解決方式是移除通道並重新進行通道設定。

以下連結提供設定 Bot Framework 所支援通道的指示：

- [Cortana](bot-service-channel-connect-cortana.md)
- [DirectLine](bot-service-channel-connect-directline.md)
- [電子郵件](bot-service-channel-connect-email.md)
- [Facebook](bot-service-channel-connect-facebook.md)
- [GroupMe](bot-service-channel-connect-groupme.md)
- [Kik](bot-service-channel-connect-kik.md)
- [Microsoft Teams](https://docs.microsoft.com/microsoftteams/platform/concepts/bots/bots-overview)
- [Skype](bot-service-channel-connect-skype.md)
- [商務用 Skype](bot-service-channel-connect-skypeforbusiness.md)
- [Slack](bot-service-channel-connect-slack.md)
- [Telegram](bot-service-channel-connect-telegram.md)
- [Twilio](bot-service-channel-connect-twilio.md)

### <a name="channel-specific-behavior"></a>通道特有的行為

有些功能的實作可能因通道而有所不同。 例如，並非所有通道都支援調適型卡片。 大部分通道都支援按鈕，但是會以通道特有方式轉譯。 如果您看到某些訊息類型在不同通道中的運作方式有所差異，請參閱 [Channel Inspector](bot-service-channels-reference.md)。

以下是一些額外的連結，可協助使用個別的通道：

- [將 Bot 新增至 Microsoft Teams 應用程式](https://docs.microsoft.com/microsoftteams/platform/concepts/bots/bots-overview)
- [Facebook：Messenger 平台簡介](https://developers.facebook.com/docs/messenger-platform/introduction)
- [Cortana 技能設計原則](https://docs.microsoft.com/cortana/skills/design-principles)
- [適用於開發人員的 Skype](https://dev.skype.com/bots)
- [Slack：能夠與 Bot 互動](https://api.slack.com/bot-users)

### <a name="channel-outage"></a>通道中斷

偶爾，某些通道可能會中斷服務。 通常，這類中斷不會持續很久。 不過，如果您懷疑發生中斷，請洽詢通道網站或社交媒體。

判斷通道是否中斷的另一種方法是建立測試 Bot (例如簡單的 Echo Bot) 並新增通道。 如果測試 Bot 適用於某些通道，但不適用於其他通道，則表示問題不在您的生產 Bot。

## <a name="additional-resources"></a>其他資源

請參閱[偵錯 Bot](bot-service-debug-bot.md) 操作說明，以及該區段中的其他偵錯文章。