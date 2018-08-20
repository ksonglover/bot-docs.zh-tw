---
title: 使用適用於 Java 的 Bot 建立器 SDK 建立 Bot | Microsoft Docs
description: 使用適用於 Java 的 Bot 建立器 SDK 快速建立 Bot。
keywords: Bot 建立器 SDK, 建立 bot, 快速入門, java, 開始使用
author: jonathanfingold
ms.author: jonathanfingold
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 05/02/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: ca4714d3b3988fd08021f55a4905d9426996b7eb
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/18/2018
ms.locfileid: "39299219"
---
# <a name="create-a-bot-with-the-bot-builder-sdk-for-java"></a>使用適用於 Java 的 Bot 建立器 SDK 建立 Bot
[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

適用於 Java 的 Bot 建立器 SDK 為 Java 開發人員提供熟悉的方式來撰寫 Bot。 SDK v4 處於預覽狀態，如需詳細資訊，請造訪 Java [GitHub 存放庫](https://github.com/Microsoft/botbuilder-java)。

> [!NOTE]
> 我們的程式碼範例和文件目前的目標版本是 Java 1.8 版。

## <a name="getting-started"></a>開始使用

v4 SDK 包含一系列的[程式庫](https://github.com/Microsoft/botbuilder-java/tree/master/libraries)。 若要在本機建置，請參閱[建置 SDK](https://github.com/Microsoft/botbuilder-java/wiki/building-the-sdk)。

- 安裝 [Bot Framework 模擬器](https://github.com/Microsoft/BotFramework-Emulator/releases)

### <a name="create-echobot"></a>建立 EchoBot

在 App.java 檔案中新增下列項目：

```Java
package com.microsoft.bot.connector.sample;

import com.fasterxml.jackson.databind.DeserializationFeature;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.microsoft.aad.adal4j.AuthenticationException;
import com.microsoft.bot.connector.customizations.CredentialProvider;
import com.microsoft.bot.connector.customizations.CredentialProviderImpl;
import com.microsoft.bot.connector.customizations.JwtTokenValidation;
import com.microsoft.bot.connector.customizations.MicrosoftAppCredentials;
import com.microsoft.bot.connector.implementation.ConnectorClientImpl;
import com.microsoft.bot.schema.models.Activity;
import com.microsoft.bot.schema.models.ActivityTypes;
import com.microsoft.bot.schema.models.ResourceResponse;
import com.sun.net.httpserver.HttpExchange;
import com.sun.net.httpserver.HttpHandler;
import com.sun.net.httpserver.HttpServer;

import java.io.IOException;
import java.io.InputStream;
import java.net.InetSocketAddress;
import java.net.URLDecoder;
import java.util.logging.Level;
import java.util.logging.Logger;

public class App {
    private static final Logger LOGGER = Logger.getLogger( App.class.getName() );
    private static String appId = "";       // <-- app id -->
    private static String appPassword = ""; // <-- app password -->

    public static void main( String[] args ) throws IOException {
        CredentialProvider credentialProvider = new CredentialProviderImpl(appId, appPassword);
        HttpServer server = HttpServer.create(new InetSocketAddress(3978), 0);
        server.createContext("/api/messages", new MessageHandle(credentialProvider));
        server.setExecutor(null);
        server.start();
    }

    static class MessageHandle implements HttpHandler {
        private ObjectMapper objectMapper;
        private CredentialProvider credentialProvider;
        private MicrosoftAppCredentials credentials;

        MessageHandle(CredentialProvider credentialProvider) {
            this.objectMapper = new ObjectMapper()
                    .configure(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES, false)
                    .findAndRegisterModules();
            this.credentialProvider = credentialProvider;
            this.credentials = new MicrosoftAppCredentials(appId, appPassword);
        }

        public void handle(HttpExchange httpExchange) throws IOException {
            if (httpExchange.getRequestMethod().equalsIgnoreCase("POST")) {
                Activity activity = getActivity(httpExchange);
                String authHeader = httpExchange.getRequestHeaders().getFirst("Authorization");
                try {
                    JwtTokenValidation.assertValidActivity(activity, authHeader, credentialProvider);

                    // send ack to user activity
                    httpExchange.sendResponseHeaders(202, 0);
                    httpExchange.getResponseBody().close();

                    if (activity.type().equals(ActivityTypes.MESSAGE)) {
                        // reply activity with the same text
                        ConnectorClientImpl connector = new ConnectorClientImpl(activity.serviceUrl(), this.credentials);
                        ResourceResponse response = connector.conversations().sendToConversation(activity.conversation().id(),
                                new Activity()
                                        .withType(ActivityTypes.MESSAGE)
                                        .withText("Echo: " + activity.text())
                                        .withRecipient(activity.from())
                                        .withFrom(activity.recipient())
                        );
                    }
                } catch (AuthenticationException ex) {
                    httpExchange.sendResponseHeaders(401, 0);
                    httpExchange.getResponseBody().close();
                    LOGGER.log(Level.WARNING, "Auth failed!", ex);
                } catch (Exception ex) {
                    LOGGER.log(Level.WARNING, "Execution failed", ex);
                }
            }
        }

        private String getRequestBody(HttpExchange httpExchange) throws IOException {
            StringBuilder buffer = new StringBuilder();
            InputStream stream = httpExchange.getRequestBody();
            int rByte;
            while ((rByte = stream.read()) != -1) {
                buffer.append((char)rByte);
            }
            stream.close();
            if (buffer.length() > 0) {
                return URLDecoder.decode(buffer.toString(), "UTF-8");
            }
            return "";
        }

        private Activity getActivity(HttpExchange httpExchange) {
            try {
                String body = getRequestBody(httpExchange);
                LOGGER.log(Level.INFO, body);
                return objectMapper.readValue(body, Activity.class);
            } catch (Exception ex) {
                LOGGER.log(Level.WARNING, "Failed to get activity", ex);
                return null;
            }

        }
    }
}
```

如果您是使用 Maven，可以從這個存放庫中的範例資料夾複製 pom.xml 檔案。 一旦您開始執行可執行檔，就會啟動 Bot Framework 模擬器。

### <a name="start-the-emulator-and-connect-your-bot"></a>啟動模擬器並且連線到您的 Bot

此時，Bot 正在本機執行。
接下來，啟動模擬器，然後在模擬器中連線到您的 Bot：

1. 按一下模擬器 [歡迎使用] 索引標籤中的 [建立新的 Bot 組態] 連結。 

2. 輸入 **Bot 名稱**，然後輸入 Bot 程式碼的目錄路徑。 Bot 組態檔會儲存至這個路徑。

3. 將 `http://localhost:port-number/api/messages` 輸入到 [端點 URL] 欄位，其中連接埠號碼必須符合應用程式執行所在瀏覽器中顯示的連接埠號碼。

4. 按一下 [連線] 來連線到 Bot。 您不需要指定 **Microsoft 應用程式識別碼**和 **Microsoft 應用程式密碼**。 目前可以將這些欄位保留空白。 稍後註冊 Bot 時，您會取得這項資訊。

### <a name="interact-with-your-bot"></a>與您的 Bot 互動
傳送 "Hi" 給您的 Bot，Bot 會回應訊息。

## <a name="next-steps"></a>後續步驟

接下來，請跳至說明 Bot 及其運作方式的概念。

> [!div class="nextstepaction"]
> [基本 Bot 概念](../v4sdk/bot-builder-basics.md)
