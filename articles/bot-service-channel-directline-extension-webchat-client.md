---
title: 搭配使用 WebChat 與 Direct Line App Service 擴充功能
titleSuffix: Bot Service
description: 搭配使用 WebChat 與 Direct Line App Service 擴充功能
services: bot-service
manager: kamrani
ms.service: bot-service
ms.topic: conceptual
ms.author: kamrani
ms.date: 07/25/2019
ms.openlocfilehash: a11870dfa728621d346a7376363c7e31ddb1596c
ms.sourcegitcommit: 6a83b2c8ab2902121e8ee9531a7aa2d85b827396
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/09/2019
ms.locfileid: "68866429"
---
# <a name="use-webchat-with-the-direct-line-app-service-extension"></a>搭配使用 WebChat 與 Direct Line App Service 擴充功能

本文說明如何搭配使用 WebChat 與 Direct Line App Service 擴充功能。

## <a name="get-your-direct-line-secret"></a>取得您的 Direct Line 祕密

第一個步驟是尋找您的 Direct Line 秘密。 您可以依照 [將 Bot 連線至 Direct Line](bot-service-channel-connect-directline.md) 一文中的指示執行此操作。

## <a name="get-the-preview-version-of-directlinejs"></a>取得 DirectLineJS 的預覽版本
您可以在這裡找到 DirectLineJS 的預覽版本： https://github.com/Jeffders/DirectLineAppServiceExtensionPreview/tree/master/libraries

## <a name="integrate-webchat-client"></a>整合 WebChat 用戶端

此方法與之前大致相同。 差別在於，已建立的新版 **WebChat** 支援雙向 **WebSocket** 流量，且不會連線至 https://directline.botframework.com/ ，而是直接連線至裝載的 Bot。
Bot 的 Direct Line URL 會是 `https://<your_app_service>.azurewebsites.net/.bot/`，其中，`/.bot/` 擴充功能是您 App Service 上的 Direct Line **端點**。
如果您可以設定自己的網域名稱，您仍須附加 `/.bot/` 路徑以存取 Direct Line REST API。

1. 依照[驗證](https://docs.microsoft.com/en-us/azure/bot-service/rest-api/bot-framework-rest-direct-line-3-0-authentication?view=azure-bot-service-4.0)一文中的指示交換權杖的秘密。 但請不要在 `https://directline.botframework.com/v3/directline/tokens/generate` 取得權杖，而是應在下列位置直接從您的 Direct Line App Service 擴充功能產生權杖：`https://<your_app_service>.azurewebsites.net/.bot/v3/directline/tokens/generate`。  

1. 取得權杖後，您可以用下列變更來更新使用 WebChat 的網頁：

```html
<!DOCTYPE html>
<html lang="en-US">
<head>
    <title>Direct Line Streaming Sample</title>
    <script src="~/directLine.js"></script>
    <script src="https://cdn.botframework.com/botframework-webchat/master/webchat.js"></script>
    <style>
        html, body {
            height: 100%
        }

        body {
            margin: 0
        }

        #webchat,

        #webchat > * {
            height: 100%;
            width: 100%;
        }
    </style>
</head>

<body>
    <div id="webchat" role="main"></div>
    <script>
        const activityMiddleware = () => next => card => {
            if (card.activity.type === 'trace') {
                // Return false means, don't render the trace activities
                return () => false;
            } else {
                return children => next(card)(children);
            }
        };


        var dl = new DirectLine.DirectLine({
            secret: '<your token>',
            domain: 'https://<your_site>.azurewebsites.net/.bot/v3/directline',
            webSocket: true,
            conversationId: '<your conversation id>'
        });
        window.WebChat.renderWebChat({
            activityMiddleware,
            directLine: dl,
            userID: '<your generated user id>'
        }, document.getElementById('webchat'));
    </script>
</body>
</html>

```
