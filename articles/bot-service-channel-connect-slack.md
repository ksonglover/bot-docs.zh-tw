---
title: 將 Bot 連線至 Slack | Microsoft Docs
description: 深入了解如何設定 Bot 與 Slack 的連線。
keywords: 連線 Bot, Bot 通道, Slack Bot, Slack 傳訊應用程式
author: JonathanFingold
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 01/09/2019
ms.openlocfilehash: 9648d1aa9e2da6243f42a5e044451ae4d8fb9d71
ms.sourcegitcommit: a6d02ec4738e7fc90b7108934740e9077667f3c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/04/2019
ms.locfileid: "70298306"
---
# <a name="connect-a-bot-to-slack"></a>將 Bot 連線至 Slack

您可以設定您的 Bot 使用 Slack 傳訊應用程式與其他人通訊。

## <a name="create-a-slack-application-for-your-bot"></a>為您的 Bot 建立 Slack 應用程式

登入 [Slack](https://slack.com/signin)，然後前往[建立 Slack 應用程式](https://api.slack.com/apps)通道。

![設定 Bot](~/media/channels/slack-NewApp.png)

## <a name="create-an-app-and-assign-a-development-slack-team"></a>建立應用程式並指派「開發 Slack 小組」

輸入「應用程式名」稱並選取「開發 Slack 小組」。 如果您還不是「開發 Slack 小組」的成員，[請建立一個或加入一個](https://slack.com/)。

![建立應用程式](~/media/channels/slack-CreateApp.png)

按一下 [建立應用程式]  。 Slack 會建立您的應用程式，並產生用戶端識別碼和用戶端密碼。

## <a name="add-a-new-redirect-url"></a>新增重新導向 URL

接下來，您將新增重新導向 URL。

1. 選取 [OAuth 與權限]  索引標籤。
2. 按一下 [新增重新導向 URL]  。
3. 輸入 https://slack.botframework.com 。
4. 按一下 [新增]  。
5. 按一下 [儲存 URL]  。

![新增重新導向 URL](~/media/channels/slack-RedirectURL.png)

## <a name="create-a-slack-bot-user"></a>建立 Slack Bot 使用者

新增 Bot 使用者可讓您為 Bot 指派使用者名稱，並選擇是否要一直顯示為上線狀態。

1. 選取 [Bot 使用者]  索引標籤。
2. 按一下 [新增 Bot 使用者]  。

![建立 Bot](~/media/channels/slack-CreateBot.png)

按一下 [新增 Bot 使用者]  以確認您的設定，並將 [一律讓 Bot 顯示為上線狀態]  設為 [開啟]  ，然後按一下 [儲存變更]  。

![建立 Bot](~/media/channels/slack-CreateApp-AddBotUser.png)

## <a name="subscribe-to-bot-events"></a>訂閱 Bot 事件

請遵循下列步驟來訂閱六個特定的 Bot 事件。 藉由訂閱 Bot 事件，應用程式將會收到所指定 URL 上的使用者活動通知。

> [!TIP]
> 您的 Bot 控制代碼就是您的 Bot 名稱。 若要尋找 Bot 的控制代碼，請瀏覽 [https://dev.botframework.com/bots](https://dev.botframework.com/bots)，然後選擇 Bot 並記錄 Bot 名稱。

1. 選取 [事件訂用帳戶]  索引標籤。
2. 將 [啟用事件]  設定為 [開啟]  。
3. 在 [要求 URL]  中輸入 `https://slack.botframework.com/api/Events/{YourBotHandle}`，其中 `{YourBotHandle}` 是您的 Bot 控制代碼 (不含大括號)。 本範例中使用的 Bot 控制代碼為 **ContosoBot**。

   ![訂閱事件：頂端](~/media/channels/slack-SubscribeEvents-a.png)

4. 在 [訂閱 Bot 事件]  中，按一下 [新增 Bot 使用者事件]  。
5. 在事件清單中，選取以下六個事件類型：
    * `member_joined_channel`
    * `member_left_channel`
    * `message.channels`
    * `message.groups`
    * `message.im`
    * `message.mpim`

   ![訂閱事件：中間](~/media/channels/slack-SubscribeEvents-b.png)

6. 按一下 [儲存變更]  。

   ![訂閱事件：底部](~/media/channels/slack-SubscribeEvents-c.png)

## <a name="add-and-configure-interactive-messages-optional"></a>新增和設定互動式訊息 (選擇性)

如果您的 Bot 將使用 Slack 特有的功能 (例如按鈕)，請遵循下列步驟：

1. 選取 [互動式元件]  索引標籤，然後按一下 [啟用互動式元件]  。
2. 輸入 `https://slack.botframework.com/api/Actions` 作為**要求 URL**。
3. 按一下 [儲存變更]  按鈕。

![啟用訊息](~/media/channels/slack-MessageURL.png)

## <a name="gather-credentials"></a>收集認證

選取 [基本資訊]  索引標籤，然後捲動至 [應用程式認證]  區段。
設定 Slack Bot 所需的用戶端識別碼、用戶端密碼和驗證權杖會隨即顯示。

![收集認證](~/media/channels/slack-AppCredentials.png)

## <a name="submit-credentials"></a>提交認證

在另一個瀏覽器視窗中，返回 Bot Framework 網站 `https://dev.botframework.com/`。

1. 選取 [我的 Bot]  ，然後選擇要連線至 Slack 的 Bot。
2. 在 [通道]  區段中，按一下 Slack 圖示。
3. 在 [輸入 Slack 認證]  區段中，將 Slack 網站中的「應用程式認證」貼到適當的欄位。
4. **登陸頁面 URL** 是選擇性的。 您可以省略或變更此 URL。
5. 按一下 [檔案]  。

![提交認證](~/media/channels/slack-SubmitCredentials.png)

請依照指示將 Slack 應用程式的存取權授與您的「開發 Slack 小組」。

## <a name="enable-the-bot"></a>啟用 Bot

在 [設定 Slack] 頁面上，確認 [儲存] 按鈕旁的滑桿已設定為 [啟用]  。
您的 Bot 會設定為可與 Slack 中的使用者通訊。

## <a name="create-an-add-to-slack-button"></a>建立 [新增至 Slack] 按鈕

在[本頁](https://api.slack.com/docs/slack-button)的＜新增 Slack 按鈕＞區段  中，Slack 會提供可用來協助 Slack 使用者找到 Bot 的 HTML。
若要讓您的 Bot 搭配此 HTML，請使用在 Bot Slack 通道設定中找到的 URL 取代 href 值 (開頭為 `https://`)。
請依照下列步驟操作，取得替代 URL。

1. 在 [https://dev.botframework.com/bots](https://dev.botframework.com/bots) 上，按一下您的 Bot。
2. 按一下 [通道]  ，以滑鼠右鍵按一下名為 **Slack** 的項目，然後按一下 [複製連結]  。 此 URL 現在在剪貼簿中。
3. 將剪貼簿中的此 URL 貼到為 [Slack] 按鈕提供的 HTML 中。 此 URL 會取代 Slack 為此 Bot 提供的 href 值。

已獲授權的使用者可以按一下 [新增至 Slack]  按鈕 (由此處已修改的 HTML 提供)，來連線到 Slack 上的 Bot。

## <a name="also-available-as-an-adapter"></a>也可作為配接器提供

此通道也可[作為配接器提供](https://botkit.ai/docs/v4/platforms/slack.html)。 為協助您在配接器和通道之間做選擇，請參閱[目前可用的配接器](bot-service-channel-additional-channels.md#currently-available-adapters)。
