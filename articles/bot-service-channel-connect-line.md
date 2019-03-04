---
title: 將 Bot 連線至 LINE | Microsoft Docs
description: 了解如何設定 Bot 與 LINE 之間的連線。
keywords: 連接 bot, bot通道, LINE bot, 認證, 設定, 電話
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 1/7/2019
ms.openlocfilehash: f6ed40c5ad3999ea5a123bf051de849917f22b42
ms.sourcegitcommit: e41dabe407fdd7e6b1d6b6bf19bef5f7aae36e61
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 02/27/2019
ms.locfileid: "56893508"
---
# <a name="connect-a-bot-to-line"></a>將 Bot 連線至 LINE

您可以設定讓 Bot 透過 LINE 應用程式與人通訊。

## <a name="log-into-the-line-console"></a>登入 LINE 主控台

透過*使用 LINE 登入*來登入 LINE 帳戶的 [LINE 開發人員主控台](https://developers.line.biz/console/register/messaging-api/provider/)。 

> [!NOTE]
> 如果您還未[下載 LINE](https://line.me/)，則請移至您的設定來註冊電子郵件地址。

### <a name="register-as-a-developer"></a>註冊為開發人員

如果您是第一次登入 LINE 開發人員主控台，請輸入名稱和電子郵件地址來建立開發人員帳戶。

![LINE 螢幕擷取畫面：註冊開發人員](./media/channels/LINE-screenshot-1.png)

## <a name="create-a-new-provider"></a>建立新的提供者

請先建立 Bot 的提供者 (如果您還未設定的話)。 提供者是提供應用程式的實體 (個人或公司)。

![LINE 螢幕擷取畫面：建立提供者](./media/channels/LINE-screenshot-2.png)

## <a name="create-a-messaging-api-channel"></a>建立傳訊 API 通道

接下來，請建立新的傳訊 API 通道。 

![LINE 螢幕擷取畫面：通道類型](./media/channels/LINE-channel-type-selection.png)

按一下綠色方塊來建立新的傳訊 API 通道。

![LINE 螢幕擷取畫面：建立通道](./media/channels/LINE-create-channel.png)

名稱中不得包含 "LINE" 或類似字串。 填寫必要欄位，並確認通道的設定。

![LINE 螢幕擷取畫面：通道設定](./media/channels/LINE-screenshot-4.png)

## <a name="get-necessary-values-from-your-channel-settings"></a>從通道設定取得所需值

在確認通道的設定後，系統會將您導向如下所示的頁面。

![LINE 螢幕擷取畫面：通道頁面](./media/channels/LINE-screenshot-5.png)

按一下您所建立的通道來存取通道設定，然後向下捲動來找到 [基本資訊] > [通道祕密]。 在某處儲存該資訊片刻。 確認 [可用功能] 包括 `PUSH_MESSAGE`。

![LINE 螢幕擷取畫面：通道祕密](./media/channels/LINE-screenshot-6.png)

然後，再往下捲動至 [傳訊設定]。 您會在其中看到 [通道存取權杖] 欄位，並有 [核發] 按鈕。 按一下該按鈕來取得存取權杖，並先儲存該權杖片刻。

![LINE 螢幕擷取畫面：通道權杖](./media/channels/LINE-screenshot-8.png)

## <a name="connect-your-line-channel-to-your-azure-bot"></a>將 LINE 通道連線至 Azure Bot

登入 [Azure 入口網站](https://portal.azure.com/)並尋找您的 Bot，然後按一下 [通道]。 

![LINE 螢幕擷取畫面：Azure 設定](./media/channels/LINE-channel-setting-2.png)

在其中選取 LINE 通道，然後將前面取得的通道密碼和存取權杖貼到適當欄位中。 請務必儲存您所做的變更。

複製 Azure 提供給您的自訂 Webhook URL。

![LINE 螢幕擷取畫面：Azure 設定](./media/channels/LINE-channel-setting-1.png)

## <a name="configure-line-webhook-settings"></a>設定 LINE 的 Webhook 設定

接下來，回到 LINE 開發人員主控台，將 Azure 提供給您的 Webhook URL 貼到 [訊息設定] > [Webhook URL]，然後按一下 [驗證] 來驗證連線。 如果您剛剛才在 Azure 中建立通道，可能需要幾分鐘的時間才會生效。

然後，啟用 [訊息設定] > [使用 Webhook]。

> [!IMPORTANT]
> 在 LINE 開發人員主控台中，您必須先設定 Webhook URL，然後才能設定 [使用 Webhook = Enabled]。 具有空白 URL 的第一個啟用 Webhook 不會設定 Enabled 狀態 (即使 UI 這麼說也是一樣)。

在新增 Webhook URL 並啟用 Webhook 之後，請務必重新載入此頁面，並確認這些變更是否已正確設定。

![LINE 螢幕擷取畫面：Webhook](./media/channels/LINE-screenshot-9.png)

## <a name="test-your-bot"></a>測試 Bot

在完成這些步驟後，您的 Bot 將成功設定為會在 LINE 上與使用者通訊，且已準備好接受測試。

### <a name="add-your-bot-to-your-line-mobile-app"></a>將 Bot 新增至 LINE 行動應用程式

在 LINE 開發人員主控台中，瀏覽至 [設定] 頁面，此時您會看到 Bot 的 QR 代碼。 

在 LINE 行動應用程式中，移至最右側有三個點 [**...**] 的瀏覽索引標籤，然後點選 [QR 代碼] 圖示。 

![LINE 螢幕擷取畫面：行動應用程式](./media/channels/LINE-screenshot-12.jpg)

將 QR 代碼讀取器指向開發人員主控台中的 QR 代碼。 您現在應該就能與 LINE 行動應用程式中的 Bot 互動，並測試 Bot 了。

### <a name="automatic-messages"></a>自動訊息

當您開始測試 Bot 時，您可能會注意到 Bot 傳送的訊息與您在 `conversationUpdate` 活動中指定的訊息不符。  您的對話方塊看起來可能像這樣：

![LINE 螢幕擷取畫面：對話](./media/channels/LINE-screenshot-conversation.jpg)

若要避免傳送這些訊息，您必須關閉自動回應訊息。

![LINE 螢幕擷取畫面：自動回應](./media/channels/LINE-screenshot-10.png)

或者，您也可以選擇保留這些訊息。 在此情況下，最好的辦法是按一下 [設定訊息]，並加以編輯。

![LINE 螢幕擷取畫面：設定自動回應](./media/channels/LINE-screenshot-11.png)

## <a name="troubleshooting"></a>疑難排解

* 如果 Bot 完全沒有回應任何訊息，請在 Azure 入口網站中瀏覽至 Bot，然後選擇 [在網路聊天中測試]。  
    * 如果 Bot 可在其中運作，在 LINE 中卻沒有回應，則請重新載入 LINE 開發人員主控台的頁面，並重複上述的 Webhook 指示。 請務必先設定 **Webhook URL** 再啟用 Webhook。
    * 如果 Bot 在網路聊天中無法運作，請針對 Bot 的問題進行偵錯，然後再回來完成 LINE 通道的設定。

