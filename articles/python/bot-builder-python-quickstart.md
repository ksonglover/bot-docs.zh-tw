---
title: 使用適用於 Python 的 Bot Builder SDK 建立 Bot | Microsoft Docs
description: 使用適用於 Python 的 Bot Builder SDK 快速建立 Bot。
keywords: Bot Builder SDK, create a bot, quickstart, python, getting started, 建立 Bot, 快速入門, python, 開始使用
author: jonathanfingold
ms.author: jonathanfingold
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 02/21/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: ba16adebe6bbb9b79949cd9842e975e35c3f2aa6
ms.sourcegitcommit: d486dd088b87a44fc8142f7a08877ff993861a42
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/26/2018
ms.locfileid: "42928407"
---
# <a name="create-a-bot-with-the-bot-builder-sdk-for-python"></a>使用適用於 Python 的 Bot Builder SDK 建立 Bot
[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

適用於 Python 的 Bot Builder SDK 是用來開發 Bot 且易於使用的架構。 本快速入門會逐步引導您建置 Bot，然後使用 Bot Framework Emulator 來測試它。 SDK v4 處於預覽狀態，如需詳細資訊，請前往 Python [GitHub 存放庫](https://github.com/Microsoft/botbuilder-python) \(英文\)。 

## <a name="pre-requisite"></a>必要條件
- [Python 3.6.4](https://www.python.org/downloads/) 
- [Bot Framework Emulator](https://github.com/Microsoft/BotFramework-Emulator/releases)

# <a name="create-a-bot"></a>建立 Bot
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
            loop.run_until_complete(JwtTokenValidation.assert_valid_activity(
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

### <a name="start-the-emulator-and-connect-your-bot"></a>啟動模擬器並連線到您的 Bot

接下來，啟動模擬器，然後在模擬器中連線到您的 Bot：


1. 按一下模擬器 [歡迎使用] 索引標籤中的 [建立新的 Bot 設定] 連結。 

2. 輸入 **Bot 名稱**，然後輸入 Bot 程式碼的目錄路徑。 Bot 設定檔將會儲存至這個路徑。

3. 將 `http://localhost:port-number/api/messages` 輸入至 [端點 URL] 欄位，其中連接埠號碼必須符合應用程式執行所在瀏覽器中顯示的連接埠號碼。

4. 按一下 [連線] 來連線至 Bot。 您不需要指定 **Microsoft 應用程式識別碼**和 **Microsoft 應用程式密碼**。 目前可以將這些欄位保留空白。 稍後註冊 Bot 時，您將會取得這項資訊。

在模擬器中輸入 **Hello** (您好)，而 Bot 會回應 **You said "Hello"** (您說了「您好」)。

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [基本 Bot 概念](../v4sdk/bot-builder-basics.md)
