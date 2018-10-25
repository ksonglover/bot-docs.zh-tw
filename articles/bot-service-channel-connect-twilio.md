---
title: 將 Bot 連線至 Twilio | Microsoft Docs
description: 了解如何設定 Bot 與 Twilio 的連線。
keywords: Twilio, bot 通道, SMS, 應用程式, 電話, 設定 Twilio, 雲端通訊, 文字
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 10/9/2018
ms.openlocfilehash: 1c85c59591313f675e53af7cffc4751a05ce4bc4
ms.sourcegitcommit: 54ed5000c67a5b59e23b667547565dd96c7302f9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/13/2018
ms.locfileid: "49315164"
---
# <a name="connect-a-bot-to-twilio"></a>將 Bot 連線至 Twilio

您可以設定您的 Bot 使用 Twilio 雲端通訊平台與其他人通訊。

## <a name="log-in-to-or-create-a-twilio-account-for-sending-and-receiving-sms-messages"></a>登入或建立 Twilio 帳戶來傳送和接收 SMS 訊息

如果您沒有 Twilio 帳戶，請<a href="https://www.twilio.com/try-twilio" target="_blank">建立新的帳戶</a>。

## <a name="create-a-twiml-application"></a>建立 TwiML 應用程式

依照指示<a href="https://support.twilio.com/hc/en-us/articles/223180928-How-Do-I-Create-a-TwiML-App-" target="_blank">建立 TwiML 應用程式</a>。

![建立應用程式](~/media/channels/twi-StepTwiml.png)

在 [屬性] 之下，輸入**易記名稱**。 在本教學課程中，我們以「我的 TwiML 應用程式」為例。 語音之下的 [要求 URL] 可以保留空白。 在 [傳訊] 底下，[要求 URL] 應該是 `https://sms.botframework.com/api/sms`。

## <a name="select-or-add-a-phone-number"></a>選取或新增電話號碼

請依照<a href = "https://support.twilio.com/hc/en-us/articles/223180048-Adding-a-Verified-Phone-Number-or-Caller-ID-with-Twilio" target="_blank">這裡</a>的指示，透過主控台網站新增已驗證的呼叫端識別碼。 完成之後，您會在 [管理號碼] 之下的 [作用中號碼] 中看見您已驗證的號碼。

![設定電話號碼](~/media/channels/twi-StepPhone.png)

## <a name="specify-application-to-use-for-voice-and-messaging"></a>指定要用於語音和傳訊的應用程式

按一下號碼並移至 [設定]。 在 [語音] 和 [傳訊] 之下，將 [設定方式] 設定為 TwiML 應用程式，以及將 [TWIML 應用程式] 設定為 [我的 TwiML 應用程式]。 完成之後，按一下 [儲存]。

![指定應用程式](~/media/channels/twi-StepPhone2.png)

請返回 [管理號碼]，您會看見 [語音] 和 [傳訊] 的組態都變更為 TwiML 應用程式。

![指定的號碼](~/media/channels/twi-StepPhone3.png)


## <a name="gather-credentials"></a>收集認證

請返回[主控台首頁](https://www.twilio.com/console/)，您會在專案儀表板上看見您的帳戶 SID 和驗證權杖，如下所示。

![收集應用程式認證](~/media/channels/twi-StepAuth.png)

## <a name="submit-credentials"></a>提交認證

在另一個視窗中，返回 Bot Framework 網站 https://dev.botframework.com/。 

- 選取 [我的 Bot]，然後選擇要連線至 Twilio 的 Bot。 這會將您導向 Azure 入口網站。
- 選取 [Bot 管理] 之下的 [通道]。 按一下 Twilio (SMS) 圖示。
- 輸入電話號碼、帳戶 SID，以及您先前記錄的驗證權杖。 完成之後，按一下 [儲存]。

![提交認證](~/media/channels/twi-StepSubmit.png)

當您完成這些步驟時，您的 Bot 即已成功設定為使用 Twilio 與使用者進行通訊。