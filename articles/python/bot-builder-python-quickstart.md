---
title: 使用適用於 Python 的 Bot Builder SDK 建立 Bot | Microsoft Docs
description: 使用適用於 Python 的 Bot Builder SDK 快速建立 Bot。
keywords: Bot Builder SDK, create a bot, quickstart, python, getting started, 建立 Bot, 快速入門, python, 開始使用
author: jonathanfingold
ms.author: jonathanfingold
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 08/30/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 4536bef820bc1e6e21ba2905fb643fe5608b3788
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/24/2018
ms.locfileid: "49999015"
---
# <a name="create-a-bot-with-the-bot-builder-sdk-for-python"></a>使用適用於 Python 的 Bot Builder SDK 建立 Bot

>[!NOTE] 
> Python SDK 處於**預覽**狀態，如需詳細資訊，請前往 Python [GitHub 存放庫](https://github.com/Microsoft/botbuilder-python)。 

本快速入門會逐步引導您建置 Bot，然後使用 Bot Framework Emulator 來測試它。 

## <a name="pre-requisite"></a>必要條件
- [Python 3.6.4](https://www.python.org/downloads/) 
- [Bot Framework Emulator](https://github.com/Microsoft/BotFramework-Emulator/releases)

## <a name="create-a-bot"></a>建立 Bot
在 main.py 檔案中，匯入下列標準模組：

```python
import http.server
import json
import asyncio
```

以及下列 SDK 模組：
```python
from botbuilder.schema import (Activity, ActivityTypes)
from botframework.connector import ConnectorClient
from botframework.connector.auth import (MicrosoftAppCredentials,
                                         JwtTokenValidation, SimpleCredentialProvider)
```
接下來，新增下列程式碼以使用 ConnectorClient 來建立 Bot：
```python
APP_ID = ''
APP_PASSWORD = ''


class BotRequestHandler(http.server.BaseHTTPRequestHandler):

    @staticmethod
    def __create_reply_activity(request_activity, text):
        return Activity(
            type=ActivityTypes.message,
            channel_id=request_activity.channel_id,
            conversation=request_activity.conversation,
            recipient=request_activity.from_property,
            from_property=request_activity.recipient,
            text=text,
            service_url=request_activity.service_url)

    def __handle_conversation_update_activity(self, activity):
        self.send_response(202)
        self.end_headers()
        if activity.members_added[0].id != activity.recipient.id:
            credentials = MicrosoftAppCredentials(APP_ID, APP_PASSWORD)
            reply = BotRequestHandler.__create_reply_activity(activity, 'Hello and welcome to the echo bot!')
            connector = ConnectorClient(credentials, base_url=reply.service_url)
            connector.conversations.send_to_conversation(reply.conversation.id, reply)

    def __handle_message_activity(self, activity):
        self.send_response(200)
        self.end_headers()
        credentials = MicrosoftAppCredentials(APP_ID, APP_PASSWORD)
        connector = ConnectorClient(credentials, base_url=activity.service_url)
        reply = BotRequestHandler.__create_reply_activity(activity, 'You said: %s' % activity.text)
        connector.conversations.send_to_conversation(reply.conversation.id, reply)

    def __handle_authentication(self, activity):
        credential_provider = SimpleCredentialProvider(APP_ID, APP_PASSWORD)
        loop = asyncio.new_event_loop()
        try:
            loop.run_until_complete(JwtTokenValidation.authenticate_request(
                activity, self.headers.get("Authorization"), credential_provider))
            return True
        except Exception as ex:
            self.send_response(401, ex)
            self.end_headers()
            return False
        finally:
            loop.close()

    def __unhandled_activity(self):
        self.send_response(404)
        self.end_headers()

    def do_POST(self):
        body = self.rfile.read(int(self.headers['Content-Length']))
        data = json.loads(str(body, 'utf-8'))
        activity = Activity.deserialize(data)

        if not self.__handle_authentication(activity):
            return

        if activity.type == ActivityTypes.conversation_update.value:
            self.__handle_conversation_update_activity(activity)
        elif activity.type == ActivityTypes.message.value:
            self.__handle_message_activity(activity)
        else:
            self.__unhandled_activity()


try:
    SERVER = http.server.HTTPServer(('localhost', 9000), BotRequestHandler)
    print('Started http server on localhost:9000')
    SERVER.serve_forever()
except KeyboardInterrupt:
    print('^C received, shutting down server')
    SERVER.socket.close()
```


儲存 main.py。 若要在 Windows 上執行範例，請在命令列視窗中輸入下列程式碼：
```
python main.py
```
在您的本機終端機中，您應該會看到 'Started http server on localhost:9000' (已在 localhost:9000 上啟動 http 伺服器) 的訊息

## <a name="start-the-emulator-and-connect-your-bot"></a>啟動模擬器並連線到您的 Bot

接下來，請啟動模擬器，然後在模擬器中連線至您的 Bot：

1. 按一下模擬器 [歡迎] 索引標籤中的 [開啟 Bot] 連結。 
2. 在您建立專案的目錄中選取 .bot 檔案。

## <a name="interact-with-your-bot"></a>與您的 Bot 互動

傳送訊息給 Bot，Bot 就會以訊息回應。
![模擬器執行中](../media/emulator-v4/emulator-running.png)


## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [Bot 概念](../v4sdk/bot-builder-basics.md)
