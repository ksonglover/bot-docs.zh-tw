---
title: 使用 Azure 表格儲存體來管理自訂狀態資料 | Microsoft Docs
description: 了解如何使用 Azure 資料表儲存體搭配適用於 Node.js 的 Bot Framework SDK，來儲存和擷取狀態資料。
author: DucVo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 5c2b8832401ccc9260c9aa872c0848b3a3e8445b
ms.sourcegitcommit: b15cf37afc4f57d13ca6636d4227433809562f8b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/11/2019
ms.locfileid: "54225713"
---
# <a name="manage-custom-state-data-with-azure-table-storage-for-nodejs"></a>使用適用於 Node.js 的 Azure 資料表儲存體管理自訂狀態資料

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

在本文中，您將實作 Azure 資料表儲存體來儲存和管理 Bot 的狀態資料。 Bot 所使用的預設「連接器狀態服務」不適用於生產環境。 您應使用 GitHub 上提供的 [Azure 延伸模組](https://www.npmjs.com/package/botbuilder-azure) (英文)，或使用您選擇的資料儲存體平台來實作自訂狀態用戶端。 以下是使用自訂狀態儲存體的一些原因：

- 更高的狀態 API 輸送量 (更充分掌控效能)
- 異地分佈的低延遲
- 掌握資料儲存位置 (例如：美國西部與美國東部)
- 存取實際狀態資料
- 狀態資料資料庫不會與其他 Bot 共用
- 儲存超過 32KB

## <a name="prerequisites"></a>必要條件

- [Node.js](https://nodejs.org/en/).
- [Bot Framework 模擬器](~/bot-service-debug-emulator.md)。
- 必須具備 Node.js Bot。 如果沒有，請[建立一個 Bot](bot-builder-nodejs-quickstart.md)。 
- [儲存體總管](http://storageexplorer.com/)。

## <a name="create-azure-account"></a>建立 Azure 帳戶
如果您沒有 Azure 帳戶，請按一下[這裡](https://azure.microsoft.com/en-us/free/)註冊免費帳戶。

## <a name="set-up-the-azure-table-storage-service"></a>設定 Azure 資料表儲存體服務
1. 在您登入 Azure 入口網站之後，請按一下 [新增] 以建立新的 Azure 表格儲存體服務。 
2. 搜尋實作 Azure 資料表的**儲存體帳戶**。 按一下 [建立] 開始建立儲存體帳戶。 
3. 填寫欄位，按一下畫面底部的 [建立] 按鈕來部署新的儲存體服務。 
4. 部署新的儲存體服務之後，巡覽至您剛才建立的儲存體帳戶。 您會發現它列在 [儲存體帳戶] 刀鋒視窗中。
4. 選取 [存取金鑰]，並複製該金鑰供稍後使用。 您的 Bot 會使用 [儲存體帳戶名稱] 和 [金鑰] 來呼叫儲存體服務以儲存狀態資料。

## <a name="install-botbuilder-azure-module"></a>安裝 botbuilder-azure 模組

若要透過命令提示字元安裝 `botbuilder-azure` 模組，請巡覽至 Bot 的目錄，然後執行下列 npm 命令：

```nodejs
npm install --save botbuilder-azure
```

## <a name="modify-your-bot-code"></a>修改您的 Bot 程式碼

若要使用 **Azure 資料表**資料庫，請將下列這幾行程式碼新增至 Bot 的 **app.js** 檔案。

1. 需要新安裝的模組。

   ```javascript
   var azure = require('botbuilder-azure'); 
   ```

2. 配置連線設定以連線至 Azure。
   ```javascript
   // Table storage
   var tableName = "Table-Name"; // You define
   var storageName = "Table-Storage-Name"; // Obtain from Azure Portal
   var storageKey = "Azure-Table-Key"; // Obtain from Azure Portal
   ```
   `storageName` 和 `storageKay` 值可在 Azure 資料表的 [存取金鑰] 功能表中找到。 如果 `tableName` 不存在於 Azure 資料表中，則會為您建立。

3. 使用 `botbuilder-azure` 模組，建立兩個新的物件以連線至 Azure 資料表。 首先，建立傳入連線組態設定的 `AzureTableClient` 執行個體。 接下來，建立傳入 `AzureTableClient` 物件的 `AzureBotStorage` 執行個體。 例如︰

   ```javascript
   var azureTableClient = new azure.AzureTableClient(tableName, storageName, storageKey);

   var tableStorage = new azure.AzureBotStorage({gzipData: false}, azureTableClient);
   ```

4. 指定您想要使用自訂的資料庫，而非記憶體內部儲存體，並將工作階段資訊新增至該資料庫。 例如︰

   ```javascript
   var bot = new builder.UniversalBot(connector, function (session) {
        // ... Bot code ...

        // capture session user information
        session.userData = {"userId": session.message.user.id, "jobTitle": "Senior Developer"};

        // capture conversation information  
        session.conversationData[timestamp.toISOString().replace(/:/g,"-")] = session.message.text;

        // save data
        session.save();
   })
   .set('storage', tableStorage);
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
3. 傳送訊息給 Bot，藉此測試您的 Bot。 像平常一樣，與 Bot 互動。 完成後，請前往 [儲存體總管] 並檢視已儲存的狀態資料。

## <a name="view-data-in-storage-explorer"></a>在 [儲存體總管] 中檢視資料

若要檢視狀態資料，請開啟 [儲存體總管] 並使用 Azure 入口網站認證連線至 Azure，或使用 `storageName` 和 `storageKey` 直接連線至表格，然後巡覽至您的 `tableName`。 

![含有 botdata 資料表資料列之 [儲存體總管] 的螢幕擷取畫面](~/media/bot-builder-nodejs-state-azure-table-storage/bot-builder-nodejs-state-azure-table-storage-query.png)

[資料] 資料行中交談的一筆記錄如下所示：

```JSON
{
    "2018-05-15T18-23-48.780Z": "I'm the second user",
    "2018-05-15T18-23-55.120Z": "Do you know what time it is?",
    "2018-05-15T18-24-12.214Z": "I'm looking for information about the new process."
}
```

## <a name="next-step"></a>後續步驟

現在，您已經可以充分掌握 Bot 的狀態資料，讓我們看看如何使用才能更有效地管理交談流程。

> [!div class="nextstepaction"]
> [管理交談流程](bot-builder-nodejs-dialog-manage-conversation-flow.md)
