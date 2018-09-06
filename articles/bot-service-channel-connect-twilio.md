---
title: 將 Bot 連線至 Twilio | Microsoft Docs
description: 了解如何設定 Bot 與 Twilio 的連線。
keywords: Twilio, bot 通道, SMS, 應用程式, 電話, 設定 Twilio, 雲端通訊, 文字
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ms.openlocfilehash: 7e09126d50cfbebfc0aad0ee7fcb71b4e7551a7d
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/18/2018
ms.locfileid: "39298903"
---
# <a name="connect-a-bot-to-twilio"></a>將 Bot 連線至 Twilio

您可以設定您的 Bot 使用 Twilio 雲端通訊平台與其他人通訊。

## <a name="log-in-to-or-create-a-twilio-account-for-sending-and-receiving-sms-messages"></a>登入或建立 Twilio 帳戶來傳送和接收 SMS 訊息

如果您沒有 Twilio 帳戶，請<a href="https://www.twilio.com/try-twilio" target="_blank">建立新的帳戶</a>

## <a name="create-a-twiml-application"></a>建立 TwiML 應用程式

<a href="https://www.twilio.com/user/account/messaging/dev-tools/twiml-apps/add" target="_blank">建立 TwiML 應用程式</a>

![建立應用程式](~/media/channels/twi-StepTwiml.png)

 在 [傳訊] 底下，「要求 URL」應該是 **https://sms.botframework.com/api/sms** 。

## <a name="select-or-add-a-phone-number"></a>選取或新增電話號碼

<a href="https://www.twilio.com/user/account/phone-numbers/incoming" target="_blank">選取或新增電話號碼</a>。 按一下數字，將其新增至您所建立的 TwiML 應用程式中。

![設定電話號碼](~/media/channels/twi-StepPhone.png)

## <a name="specify-application-to-use-for-messaging"></a>指定要用於傳訊的應用程式
在 [傳訊] 區段中，將 [TwiML 應用程式] 設定為您剛才所建立 TwiML 應用程式的名稱。
複製**電話號碼**值以供日後使用。

![指定應用程式](~/media/channels/twi-StepPhone2.png)

## <a name="gather-credentials"></a>收集認證

<a href="https://www.twilio.com/user/account/settings" target="_blank">收集認證</a>，然後按一下「眼睛」圖示以查看驗證權杖。

![收集應用程式認證](~/media/channels/twi-StepAuth.png)

## <a name="submit-credentials"></a>提交認證

輸入電話號碼、accountSID 以及您稍早複製的驗證權杖，然後按一下 [提交 Twilio 認證]。

## <a name="enable-the-bot"></a>啟用 Bot
勾選 [在 SMS 上啟用此 Bot]。 然後按一下 [我已完成設定 SMS]。

當您完成這些步驟時，您的 Bot 即已成功設定為使用 Twilio 與使用者進行通訊。

