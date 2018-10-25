---
title: 藉由 Azure Cosmos DB 管理自訂狀態資料 | Microsoft Docs
description: 了解如何使用 Azure Cosmos DB 搭配 Bot Builder SDK for Node.js 來儲存和擷取狀態資料。
author: DucVo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 0c0d91a7ec9fd1d72c7c51c042b0f52e28798778
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/24/2018
ms.locfileid: "49998115"
---
# <a name="manage-custom-state-data-with-azure-cosmos-db-for-nodejs"></a>使用適用於 Node.js 的 Azure Cosmos DB 管理自訂狀態資料

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

在本文中，您將實作 Cosmos DB 儲存體來儲存和管理 Bot 的狀態資料。 Bot 所使用的預設連接器狀態服務不適用於生產環境。 您應使用 GitHub 上提供的 [Azure 延伸模組](https://www.npmjs.com/package/botbuilder-azure) (英文)，或使用您選擇的資料儲存體平台來實作自訂狀態用戶端。 以下是使用自訂狀態儲存體的一些原因：

- 更高的狀態 API 輸送量 (更充分掌控效能)
- 異地分佈的低延遲
- 掌握資料儲存位置 (例如：美國西部與美國東部)
- 存取實際狀態資料
- 狀態資料資料庫不會與其他 Bot 共用
- 儲存超過 32KB

## <a name="prerequisites"></a>必要條件

- [Node.js](https://nodejs.org/en/).
- [Bot Framework 模擬器](~/bot-service-debug-emulator.md) (英文)
- 必須具備 Node.js Bot。 如果沒有，請[建立一個 Bot](bot-builder-nodejs-quickstart.md)。 

## <a name="create-azure-account"></a>建立 Azure 帳戶
如果您沒有 Azure 帳戶，請按一下[這裡](https://azure.microsoft.com/en-us/free/)註冊免費帳戶。

## <a name="set-up-the-azure-cosmos-db-database"></a>設定 Azure Cosmos DB 資料庫
1. 在您登入 Azure 入口網站之後，按一下 [新增] 以建立新的 *Azure Cosmos DB* 資料庫。 
2. 按一下 [資料庫]。 
3. 找到 **Azure Cosmos DB**，然後按一下 [建立]。
4. 填寫欄位。 如果是 [**API**] 欄位，請選取 [**SQL (DocumentDB)**]。 填妥所有欄位後，按一下畫面底部的 [建立] 按鈕來部署新的資料庫。 
5. 新資料庫部署完畢後，接著瀏覽至新資料庫。 按一下 [存取金鑰] 以尋找金鑰和連接字串。 您的 Bot 會使用此資訊來呼叫儲存體服務以儲存狀態資料。

## <a name="install-botbuilder-azure-module"></a>安裝 botbuilder-azure 模組

若要透過命令提示字元安裝 `botbuilder-azure` 模組，請巡覽至 Bot 的目錄，然後執行下列 npm 命令：

```nodejs
npm install --save botbuilder-azure
```

## <a name="modify-your-bot-code"></a>修改您的 Bot 程式碼

若要使用 **Azure Cosmos DB** 資料庫，請將下列這幾行程式碼新增至 Bot 的 **app.js** 檔案。

1. 需要新安裝的模組。

   ```javascript
   var azure = require('botbuilder-azure'); 
   ```

2. 配置連線設定以連線至 Azure。
   ```javascript
   var documentDbOptions = {
       host: 'Your-Azure-DocumentDB-URI', 
       masterKey: 'Your-Azure-DocumentDB-Key', 
       database: 'botdocs',   
       collection: 'botdata'
   };
   ```
   `host` 和 `masterKey` 值可在您資料庫的 [**金鑰**] 功能表中找到。 如果 Azure 資料庫不存在 `database` 和 `collection` 項目於，系統將會為您建立。

3. 使用 `botbuilder-azure` 模組，建立兩個新的物件，以連線至 Azure 資料庫。 首先，建立 `DocumentDBClient` 的執行個體，傳入連線的組態設定 (定義為上述的 `documentDbOptions`)。 接下來，建立傳入 `DocumentDBClient` 物件的 `AzureBotStorage` 執行個體。 例如︰
   ```javascript
   var docDbClient = new azure.DocumentDbClient(documentDbOptions);

   var cosmosStorage = new azure.AzureBotStorage({ gzipData: false }, docDbClient);
   ```

4. 指定您想要使用自訂的資料庫，而非記憶體內部儲存體。 例如︰

   ```javascript
   var bot = new builder.UniversalBot(connector, function (session) {
        // ... Bot code ...
   })
   .set('storage', cosmosStorage);
   ```

現在您已準備就緒，可以使用模擬器測試 Bot。

## <a name="run-your-bot-app"></a>執行 Bot 應用程式

透過命令提示字元，巡覽至 Bot 的目錄並使用下列命令執行 Bot：

```nodejs
node app.js
```

## <a name="connect-your-bot-to-the-emulator"></a>將 Bot 連線至模擬器

此時，Bot 正在本機執行。 啟動模擬器，然後從模擬器連線至您的 Bot：

1. 將 <strong>http://localhost:port-number/api/messages</strong> 鍵入模擬器的網址列，其中連接埠號碼必須符合應用程式執行所在之瀏覽器中顯示的連接埠號碼。 您目前可以將 <strong>Microsoft 應用程式識別碼</strong>和 <strong>Microsoft 應用程式密碼</strong>欄位留白。 稍後[註冊 Bot](~/bot-service-quickstart-registration.md) 時，您會取得這些資訊。
2. 按一下 [ **連接**]。
3. 傳送訊息給 Bot，藉此測試您的 Bot。 像平常一樣，與 Bot 互動。 完成後，請前往**儲存體總管**並檢視已儲存的狀態資料。

## <a name="view-state-data-on-azure-portal"></a>檢視 Azure 入口網站的狀態資料

若要檢視狀態資料，請登入 Azure 入口網站並瀏覽至您的資料庫。 按一下 [**資料總管 (預覽)**] 以確認正在儲存的 Bot 狀態資訊。

## <a name="next-step"></a>後續步驟

現在，您已經可以充分掌握 Bot 的狀態資料，讓我們看看如何使用才能更有效地管理交談流程。

> [!div class="nextstepaction"]
> [管理交談流程](bot-builder-nodejs-dialog-manage-conversation-flow.md)
