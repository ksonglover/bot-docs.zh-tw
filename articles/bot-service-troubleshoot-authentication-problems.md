---
title: 針對 Bot Framework 驗證進行疑難排解 | Microsoft Docs
description: 了解如何使用 Bot 針對驗證錯誤進行疑難排解。
author: DeniseMak
ms.author: v-demak
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/17
ms.openlocfilehash: a64edda73832f4d3fff49b08b5eaf6792c021ece
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/18/2018
ms.locfileid: "39299934"
---
# <a name="troubleshooting-bot-framework-authentication"></a>針對 Bot Framework 驗證進行疑難排解

此指南可協助您透過評估一系列的案例，來判斷問題所在，從而解決 Bot 的驗證問題。 

> [!NOTE]
> 若要完成此指南中的所有步驟，您必須下載並使用 [Bot Framework 模擬器][Emulator]，而且必須能夠存取 <a href="https://dev.botframework.com" target="_blank">Bot Framework 入口網站</a>中的 Bot 註冊設定。

## <a id="PW"></a> 應用程式識別碼和密碼

Bot 安全性由您在使用 Bot Framework 註冊 Bot 時取得的**Microsoft 應用程式識別碼**和 **Microsoft 應用程式密碼**進行設定。 這些值通常在 Bot 的設定檔內指定，且用來擷取來自 Microsoft 帳戶服務的存取權杖。 

如果您還沒有這樣做，請[註冊您的 Bot](~/bot-service-quickstart-registration.md)以取得可用於進行驗證的 **Microsoft 應用程式識別碼**和 **Microsoft 應用程式密碼**。 

> [!NOTE]
> 若要尋找您 Bot 的 **AppID** 和 **AppPassword**，請參閱 [MicrosoftAppID 和 MicrosoftAppPassword](bot-service-manage-overview.md#microsoftappid-and-microsoftapppassword)。

## <a name="step-1-disable-security-and-test-on-localhost"></a>步驟 1：停用安全性，並在 localhost 上進行測試

在此步驟中，您將在停用安全性時驗證您的 Bot 是否可在 localhost 上存取與執行。 

> [!WARNING]
> 停用 Bot 的安全性可能會允許未知的攻擊者冒充使用者。 只有當您在受保護的偵錯環境中操作時，才實作下列程序。

### <a id="disable-security-localhost"></a> 停用安全性

若要停用 Bot 的安全性，請編輯其組態設定，以移除應用程式識別碼和密碼的值。 

如果您使用 Bot Builder SDK for .NET，請在 Web.config 檔案中編輯這些設定：

```xml
<appSettings>
  <add key="MicrosoftAppId" value="" />
  <add key="MicrosoftAppPassword" value="" />
</appSettings>
```

如果您使用 Bot Builder SDK for Node.js，請編輯這些值 (或更新對應的環境變數)：

```javascript
var connector = new builder.ChatConnector({
  appId: null,
  appPassword: null
});
```

### <a name="test-your-bot-on-localhost"></a>在 localhost 上測試您的 Bot 

接下來，使用 Bot Framework 模擬器在 localhost 上測試您的 Bot。

1. 在 localhost 上啟動您的 Bot。
2. 啟動 Bot Framework 模擬器。
3. 使用模擬器連線至您的 Bot。
    - 將 `http://localhost:port-number/api/messages` 輸入模擬器的網址列，其中**連接埠號碼**必須符合應用程式執行所在瀏覽器中顯示的連接埠號碼。 
    - 請確定 **Microsoft 應用程式識別碼**和 **Microsoft 應用程式密碼**欄位皆留白。
    - 按一下 [連線]。
4. 若要測試與 Bot 的連線，請在模擬器中輸入一些文字，然後按 Enter。

如果 Bot 回應輸入且聊天視窗中沒有任何錯誤，則您已驗證在停用安全性時，您的 Bot 可在 localhost 上存取和執行。 請繼續進行[步驟 2](#step-2)。

如果聊天視窗中顯示一或多個錯誤，請按一下錯誤以取得詳細資料。 常見問題包括：

* 模擬器設定為 Bot 指定了不正確的端點。 請確定您在 URL 中包含適當的連接埠號碼，以及在 URL 結尾包含適當的路徑 (例如，`/api/messages`)。
* 模擬器設定指定以 `https` 開頭的 Bot 端點。 在 localhost 上，端點應以 `http` 開頭。
* 模擬器設定指定 **Microsoft 應用程式識別碼**欄位和/或 **Microsoft 應用程式密碼**欄位的值。 這兩個欄位應為空白。
* Bot 的安全性尚未停用。 [驗證](#disable-security-localhost) Bot 未指定應用程式識別碼或密碼的值。

## <a id="step-2"></a> 步驟 2︰驗證 Bot 的應用程式識別碼和密碼

在此步驟中，您將驗證 Bot 將用於驗證的應用程式識別碼和密碼是否有效 (如果您不知道這些值，[現在就取得](#PW))。 

> [!WARNING]
> 下列指示會停用 `login.microsoftonline.com` 的 SSL 驗證。 僅在安全的網路上執行此程序，並考慮之後變更您的應用程式密碼。

### <a name="issue-an-http-request-to-the-microsoft-login-service"></a>向 Microsoft 登入服務發出 HTTP 要求

這些指示會說明如何使用 [cURL](https://curl.haxx.se/download.html) 向 Microsoft 登入服務發出 HTTP 要求。 您可以使用替代工具 (例如 Postman)，只需確定要求符合 Bot Framework [驗證通訊協定](~/rest-api/bot-framework-rest-connector-authentication.md)。

若要驗證 Bot 的應用程式識別碼和密碼是否有效，請使用 **cURL** 發出下列要求，將 `APP_ID` 和 `APP_PASSWORD` 取代為 Bot 的應用程式識別碼和密碼。

```cmd
curl -k -X POST https://login.microsoftonline.com/botframework.com/oauth2/v2.0/token -d "grant_type=client_credentials&client_id=APP_ID&client_secret=APP_PASSWORD&scope=https%3A%2F%2Fapi.botframework.com%2F.default"
```

此要求會嘗試交換 Bot 的應用程式識別碼和密碼，以取得存取權杖。 如果要求成功，您將收到包含 `access_token` 屬性的 JSON 承載。 

```json
{"token_type":"Bearer","expires_in":3599,"ext_expires_in":0,"access_token":"eyJ0eXAJKV1Q..."}
```

如果要求成功，則您已驗證在要求中指定的應用程式識別碼和密碼有效。 請繼續進行[步驟 3](#step-3)。

如果在回應要求時收到錯誤，請檢查回應以找出錯誤的原因。 如果回應指出應用程式識別碼或密碼不正確，請從 Bot Framework 入口網站[取得正確的值](#PW)，並使用新值重新發出要求，以確認其是否有效。 

## 步驟 3：啟用安全性，並在 localhost 上進行測試 <a id="step-3"></a>

此時，您已驗證在停用安全性時，您的 Bot 可在 localhost 上存取與執行，並確認 Bot 將用於驗證的應用程式識別碼和密碼有效。 在此步驟中，您將在啟用安全性時驗證您的 Bot 是否可在 localhost 上存取與執行。

### <a id="enable-security-localhost"></a> 啟用安全性

您的 Bot 的安全性依賴 Microsoft 服務，即使您的 Bot 只在 localhost 上執行。 若要啟用 Bot 的安全性，請編輯其組態設定，以使用您在[步驟 2](#step-2) 中驗證的值填入應用程式識別碼和密碼。

如果您使用 Bot Builder SDK for .NET，請在 Web.config 檔案中填入這些設定：

```xml
<appSettings>
  <add key="MicrosoftAppId" value="APP_ID" />
  <add key="MicrosoftAppPassword" value="PASSWORD" />
</appSettings>
```

如果您使用 Bot Builder SDK for Node.js，請填入這些設定 (或更新對應的環境變數)：

```javascript
var connector = new builder.ChatConnector({
  appId: 'APP_ID',
  appPassword: 'PASSWORD'
});
```

> [!NOTE]
> 若要尋找您 Bot 的 **AppID** 和 **AppPassword**，請參閱 [MicrosoftAppID 和 MicrosoftAppPassword](bot-service-manage-overview.md#microsoftappid-and-microsoftapppassword)。

### <a name="test-your-bot-on-localhost"></a>在 localhost 上測試您的 Bot 

接下來，使用 Bot Framework 模擬器在 localhost 上測試您的 Bot。

1. 在 localhost 上啟動您的 Bot。
2. 啟動 Bot Framework 模擬器。
3. 使用模擬器連線至您的 Bot。
    - 將 `http://localhost:port-number/api/messages` 輸入模擬器的網址列，其中**連接埠號碼**必須符合應用程式執行所在瀏覽器中顯示的連接埠號碼。 
    - 在 **Microsoft 應用程式識別碼**欄位中輸入您 Bot 的應用程式識別碼。
    - 在 **Microsoft 應用程式密碼**欄位中輸入您的 Bot 密碼。
    - 按一下 [連線]。
4. 若要測試與 Bot 的連線，請在模擬器中輸入一些文字，然後按 Enter。

如果 Bot 回應輸入且聊天視窗中沒有任何錯誤，則您已驗證在啟用安全性時，您的 Bot 可在 localhost 上存取和執行。  請繼續進行[步驟 4](#step-4)。

如果聊天視窗中顯示一或多個錯誤，請按一下錯誤以取得詳細資料。 常見問題包括：

* 模擬器設定為 Bot 指定了不正確的端點。 請確定您在 URL 中包含適當的連接埠號碼，以及在 URL 結尾包含適當的路徑 (例如，`/api/messages`)。
* 模擬器設定指定以 `https` 開頭的 Bot 端點。 在 localhost 上，端點應以 `http` 開頭。
* 在模擬器設定中，**Microsoft 應用程式識別碼**欄位和/或 **Microsoft 應用程式密碼**欄位未包含有效的值。 應填入這兩個欄位，而且每個欄位應包含您在[步驟 2](#step-2) 中驗證的對應值。
* Bot 的安全性尚未啟用。 [驗證](#enable-security-localhost) Bot 組態設定指定應用程式識別碼和密碼的值。

## 步驟 4：在雲端測試您的 Bot <a id="step-4"></a>

此時，您已驗證在停用安全性時，您的 Bot 在 localhost 上可存取並正常執行、已確認您的 Bot 應用程式識別碼和密碼有效，且已驗證在啟用安全性時您的 Bot 在 localhost 上可存取並正常執行。 在此步驟中，您會將您的 Bot 部署到雲端，並在啟用安全性的情況下驗證它是否可存取並正常執行。 

### <a name="deploy-your-bot-to-the-cloud"></a>將您的 Bot 部署至雲端

Bot Framework 要求可從網際網路存取 Bot，因此您必須將您的 Bot 部署至裝載平台，例如 Azure 雲端。 確保在部署之前為您的 Bot 啟用安全性，如[步驟 3](#step-3) 中所述。

> [!NOTE]
> 若您還沒有雲端裝載提供者，則可以註冊<a href="https://azure.microsoft.com/en-us/free/" target="_blank">免費帳戶</a>。 

如果您將 Bot 部署至 Azure，系統會自動為您的應用程式設定 SSL，以啟用 Bot Framework 所需的 **HTTPS** 端點。 如果您部署至其他雲端裝載提供者，請務必確認您的應用程式已設定 SSL，讓 Bot 能夠有 **HTTPS** 端點。

### <a name="test-your-bot"></a>測試 Bot 

若要在啟用安全性的情況下在雲端中測試 Bot，請完成下列步驟。

1. 請確定您的 Bot 已成功部署和執行。 
2. 登入 <a href="https://dev.botframework.com" target="_blank">Bot Framework 入口網站</a>。
3. 按一下 [我的 Bot]。
4. 選取您想要測試的 Bot。
5. 按一下 [測試] 以在內嵌的 Web 聊天控制項中開啟 Bot。
6. 若要測試與 Bot 的連線，請在網路聊天控制項中輸入一些文字，然後按 Enter。

如果聊天視窗中指出錯誤，請使用錯誤訊息來判斷錯誤的原因。 常見問題包括： 

* 在 Bot Framework 入口網站中針對 Bot 的 [設定] 頁面上指定的**傳訊端點**不正確。 請確定您在 URL 結尾包含適當的路徑 (例如，`/api/messages`)。
* 在 Bot Framework 入口網站中針對 Bot 的 [設定] 頁面上指定的**傳訊端點**未以 `https` 開頭，也不受 Bot Framework 信任。 您的 Bot 必須具有有效的鏈結信任憑證。
* 為 Bot 設定之應用程式識別碼或密碼的值遺失或不正確。 [驗證](#enable-security-localhost) Bot 組態設定指定應用程式識別碼和密碼的有效值。

如果 Bot 對輸入做出了適當的回應，那麼您已驗證您的 Bot 在啟用安全性的情況下可以在雲端中存取與執行。 此時，您的 Bot 已準備好安全地[連線到通道](~/bot-service-manage-channels.md)，例如 Skype、Facebook Messenger，直接線路等。

## <a name="additional-resources"></a>其他資源

如果在完成上述步驟之後仍然遇到問題，您可以：

* 使用 Bot Framework 模擬器和<a href="https://ngrok.com/" target="_blank">ngrok</a> [在雲端針對您的 Bot 進行偵錯](~/bot-service-debug-emulator.md)。
* 使用像 [Fiddler](https://www.telerik.com/fiddler) 這樣的 Proxy 處理工具，來檢查進出 Bot 的 HTTPS 流量。 Fiddler 並非 Microsoft 產品。
* 若要了解 Bot Framework 所使用的驗證技術，請檢閱 [Bot 連接器驗證指南][BotConnectorAuthGuide]。
* 使用 Bot Framework [支援][Support]資源向其他人請求協助。 

[BotConnectorAuthGuide]: ~/rest-api/bot-framework-rest-connector-authentication.md
[Support]: bot-service-resources-links-help.md
[Emulator]: bot-service-debug-emulator.md
