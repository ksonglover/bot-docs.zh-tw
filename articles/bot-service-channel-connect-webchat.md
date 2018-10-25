---
title: 將 Bot 連線至網路聊天頻道 | Microsoft Docs
description: 了解如何針對連線到網路聊天頻道的 Bot，使用網頁中的網路聊天控制項。
keywords: 網路聊天, bot 通道, 網頁, 祕密金鑰, iframe, HTML
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 10/10/2018
ms.openlocfilehash: 6e81b51243afc15714653aed7b9ca6513314071c
ms.sourcegitcommit: 54ed5000c67a5b59e23b667547565dd96c7302f9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/13/2018
ms.locfileid: "49315154"
---
# <a name="connect-a-bot-to-web-chat"></a>將 Bot 連線至「網路聊天」

[!INCLUDE pre-release-label]

當您使用 Bot Service [建立 Bot](bot-service-quickstart.md) 時，會自動為您設定網路聊天頻道。 網路聊天頻道包含網路聊天控制項，能夠讓您的使用者直接在網頁上與 Bot 進行互動。

![網路聊天範例](./media/bot-service-channel-webchat/create-a-bot.png)

Bot Framework 入口網站中的網路聊天頻道，包含您在網頁中內嵌網路聊天控制項所需的所有項目。 如需使用網路聊天控制項，您只需要取得 Bot 的祕密金鑰，並在網頁中內嵌控制項即可。

## <a id="step-1"></a> 取得您的 bot 祕密金鑰

1. 在 [Azure 入口網站](http://portal.azure.com)中開啟您的 Bot，然後按一下 [通道] 刀鋒視窗。

2. 針對 **網路聊天**頻道，按一下 [編輯]。  
![網路聊天頻道](./media/bot-service-channel-webchat/bot-service-channel-list.png)

3. 在 [祕密金鑰] 底下，按一下第一個金鑰的 [顯示]。  
![祕密金鑰](./media/bot-service-channel-webchat/secret-key.png)

4. 複製 [祕密金鑰] 和 [內嵌程式碼]。

5. 按一下 [完成] 。

## <a name="embed-the-web-chat-control-in-your-website"></a>在您的網站中內嵌網路聊天控制項

您可以使用下列兩個選項之一，在您的網站中內嵌網路聊天控制項。

### <a name="option-1---keep-your-secret-hidden-exchange-your-secret-for-a-token-and-generate-the-embed"></a>選項 1 - 保持隱藏您的祕密、將您的祕密與權杖交換，並產生內嵌

如果您可以執行伺服器對伺服器要求，將您的網路聊天祕密與暫存權杖交換，且想要讓其他開發人員難以在他們的網站中內嵌您的 Bot，請使用此選項。 雖然使用此選項無法完全防止其他開發人員在其網站中內嵌您的 Bot，但確實會增加難度。

若要將您的祕密與權杖交換並產生內嵌：

1. 請發出 **GET** 要求給 `https://webchat.botframework.com/api/tokens`，並透過 `Authorization` 標頭傳遞您的網路聊天祕密。 `Authorization` 標頭會使用 `BotConnector` 配置並包含您的祕密，如下列範例要求中所示。

2. 您 **GET** 要求的回應會包含權杖 (以引號括住)，可透過轉譯 **iframe** 內的 網路聊天控制項用來開始對話。 權杖僅對一個對話有效；若要開始另一個對話，您必須產生新的權杖。

3. 在您從 Bot Framework 入口網站中的網路聊天頻道複製的 `iframe` **內嵌程式碼**內 (如上方[取得您的 bot 祕密金鑰](#step-1)中所述)，將 `s=` 參數變更為 `t=`，並以您的權杖來取代 "YOUR_SECRET_HERE"。

> [!NOTE]
> 權杖在到期前會自動更新。 

##### <a name="example-request"></a>範例要求

```requestGET https://webchat.botframework.com/api/tokens Authorization: BotConnector YOUR_SECRET_HERE
```

##### Example response 

```response
"IIbSpLnn8sA.dBB.MQBhAFMAZwBXAHoANgBQAGcAZABKAEcAMwB2ADQASABjAFMAegBuAHYANwA.bbguxyOv0gE.cccJjH-TFDs.ruXQyivVZIcgvosGaFs_4jRj1AyPnDt1wk1HMBb5Fuw"
```

##### <a name="example-iframe-using-token"></a>範例 iframe (使用權杖)

```html
<iframe src="https://webchat.botframework.com/embed/YOUR_BOT_ID?t=YOUR_TOKEN_HERE"></iframe>
```

##### <a name="example-html-code"></a>範例 html 程式碼
```html
<!DOCTYPE html>
<html>
<body>
  <iframe id="chat" style="width: 400px; height: 400px;" src=''></iframe>
</body>
<script>

    var xhr = new XMLHttpRequest();
    xhr.open('GET', "https://webchat.botframework.com/api/tokens", true);
    xhr.setRequestHeader('Authorization', 'BotConnector ' + 'YOUR SECRET HERE');
    xhr.send();
    xhr.onreadystatechange = processRequest;

    function processRequest(e) {
      if (xhr.readyState == 4  && xhr.status == 200) {
        var response = JSON.parse(xhr.responseText);
        document.getElementById("chat").src="https://webchat.botframework.com/embed/lucas-direct-line?t="+response
      }
    }

  </script>
</html>
```

### <a id="option-2"></a> 選項 2 - 在您的網站中使用祕密來內嵌網路聊天控制項

如果您想要讓其他開發人員輕鬆地在其網站中內嵌您的 Bot，請使用此選項。 

> [!WARNING]
> 如果您使用此選項，其他開發人員只需複製您的內嵌程式碼，即可在其網站中內嵌您的 Bot。

若要藉由指定 `iframe` 標記內的祕密，在您的網站中內嵌 Bot：

1. 請從 Bot Framework 入口網站內的網路聊天頻道，複製 `iframe` **內嵌程式碼** (如上方[取得您的 bot 祕密金鑰](#step-1)中所述)。

2. 在**內嵌程式碼**內，將 "YOUR_SECRET_HERE" 取代為您從相同頁面中複製的**祕密金鑰**值。

##### <a name="example-iframe-using-secret"></a>範例 iframe (使用祕密)

```html
<iframe src="https://webchat.botframework.com/embed/YOUR_BOT_ID?s=YOUR_SECRET_HERE"></iframe>
```

## <a name="style-the-web-chat-control"></a>設計網路聊天控制項樣式

您可以使用 `iframe` 的 `style` 屬性來指定 `height` 和 `width`，以變更網路聊天控制項的大小。

```html
<iframe style="height:480px; width:402px" src="... SEE ABOVE ..."></iframe>
```

![聊天控制項用戶端](./media/chatwidget-client.png)

## <a name="additional-resources"></a>其他資源

您可以在 GitHub 上[下載 Web 聊天控制項的原始程式碼](https://aka.ms/BotFramework-WebChat-V4)。
