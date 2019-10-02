---
title: 在 v4 Bot 中使用 JavaScript v3 使用者狀態 | Microsoft Docs
description: 如何在 v4 Bot 中使用 v3 使用者狀態的範例
keywords: JavaScript, bot 移轉, v3 bot
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 08/14/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 4697ecf47464114de68ec6c0d872b45ff1ee5e54
ms.sourcegitcommit: d493caf74b87b790c99bcdaddb30682251e3fdd4
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/26/2019
ms.locfileid: "71278978"
---
<!-- This article is on hold -->

# <a name="using-javascript-v3-user-state-in-a-v4-bot"></a>在 v4 Bot 中使用 JavaScript v3 使用者狀態

本文說明 v4 Bot 如何對 v3 使用者狀態資訊執行讀取、寫入和刪除作業的範例。

您可以在[這裡](https://github.com/microsoft/BotBuilder-Samples/tree/master/MigrationV3V4/Node/V4V3-user-state-adapter-sample-bot)找到程式碼範例。

> [!NOTE]
> Bot 會維護**對話狀態**以追蹤及引導對話，並向使用者詢問問題。 其會維護**使用者狀態**，以追蹤使用者的答案。

## <a name="prerequisites"></a>必要條件

- npm 6.9.0 版或更高版本 (用以支援套件別名)。

- Node.js 10.14.1 版或更高版本。

    若要檢查您的電腦上安裝的版本，請在終端機中執行下列命令：

    ```bash
    # determine node version
    node --version
    ```

## <a name="setup"></a>設定

1. 複製儲存機制

    ```bash
    git clone https://github.com/microsoft/botbuilder-samples.git
    ```

1. 在終端機中，瀏覽至 `BotBuilder-Samples/MigrationV3V4/Node/V4V3-user-state-adapter-sample-bot`

    ```bash
    cd BotBuilder-Samples/MigrationV3V4/Node/V4V3-user-state-adapter-sample-bot
    ```

1. 在下列位置執行 `npm install`：

    ```bash
    root
    /V4V3StorageMapper
    /V4V3UserState
    ```

1. 執行 ``npm run build`` 或 ``tsc``，以在下列位置中建置 `StorageMapper` 和 `UserState` 模組：

    ```bash
    /V4V3StorageMapper
    /V4V3UserState
    ```

1. 設定資料基底

    1. 複製 `.env.example` 檔案的內容。
    1. 建立名為 `.env` 的新檔案，並將先前的內容貼入其中。 
    1. 填入您儲存體提供者的值。
        請注意，您可以透過 Azure入口網站，在您的儲存體提供者 (例如 *Cosmos DB*、*資料表儲存體*或 *SQL 資料庫*) 的區段下找到*使用者名稱*、*密碼*和*主機資訊*。 資料表和集合名稱由使用者所定義。
  
1. 設定 Bot 的儲存體提供者

    1. 開啟專案根目錄中的 `index.js` 檔案。 在檔案的開頭處 (38-98 行左右)，您會看到每個儲存體提供者的組態，如註解所批註。 其會透過節點 `process.env` 從 `.env` 檔案讀取組態值。 下列程式碼片段顯示如何設定 SQL Database。

        [!code-javascript[Storage configuration](~/../botbuilder-samples/MigrationV3V4/Node/V4V3-user-state-adapter-sample-bot/index.js?range=77-92)]

    1. 藉由將您選擇的儲存體用戶端執行個體傳至 `StorageMapper` 配接器 (107 行左右)，以指定您要讓 Bot 使用的儲存體提供者。  

        [!code-javascript[StorageMapper](~/../botbuilder-samples/MigrationV3V4/Node/V4V3-user-state-adapter-sample-bot/index.js?range=105-107)]

        預設設定為 *Cosmos DB*。 可能的值包括：

        ```bash
            cosmosStorageClient
            tableStorage
            sqlStorage
        ```

1. 啟動應用程式。 從專案根目錄執行下列命令︰

    ```bash
    npm run start
    ```

## <a name="adapter-classes"></a>配接器類別

### <a name="v4v3storagemapper"></a>V4V3StorageMapper

`StorageMapper` 類別包含主要配接器功能。 其會實作 v4 儲存介面，並將儲存體提供者方法 (讀取、寫入和刪除) 對應回 v3 儲存體提供者類別，讓您可以從 v4 Bot 使用 v3 格式的使用者狀態。

### <a name="v4v3userstate"></a>V4V3UserState

此類別擴充了 v4 `BotState` 類別 (`botbuilder-core`) 而使其使用 v3 樣式的金鑰，以允許對 v3 儲存體執行讀取、寫入和刪除。

## <a name="testing-the-bot-using-bot-framework-emulator"></a>使用 Bot Framework Emulator 測試 Bot

[Bot Framework Emulator][5] 是一個桌面應用程式，可讓您在本機或透過通道從遠端 Bot 進行測試和偵錯。

- 從[這裡][6]安裝 Bot Framework Emulator 4.3.0 版或更高版本

### <a name="connect-to-the-bot-using-bot-framework-emulator"></a>使用 Bot Framework Emulator 連線至 Bot

1. 啟動 Bot Framework Emulator。
1. 輸入下列 URL (端點)：`http://localhost:3978/api/messages`。

### <a name="testing-steps"></a>測試步驟

1. 在模擬器中開啟 Bot，並傳送訊息。 出現提示時，請提供您的名稱。
1. 在此回合結束後，將另一則訊息傳送至 Bot。
1. 確定系統不會再次提示您提供名稱。 Bot 應該會從儲存體讀取該名稱，並認知其已提示過您。
1. Bot 應該會回應您的訊息。
1. 移至您在 Azure 中的儲存體提供者，並確認您的名稱已在資料庫中儲存為使用者資料。

## <a name="deploy-the-bot-to-azure"></a>將 Bot 部署至 Azure

若要深入了解如何將 Bot 部署至 Azure，請參閱[將 Bot 部署至 Azure][40]，以取得部署指示的完整清單。

## <a name="further-reading"></a>進階閱讀

- [Azure Bot 服務簡介][21]
- [Bot 狀態][7]
- [直接寫入儲存體][8]
- [管理對話和使用者狀態][9]
- [Restify][30]
- [dotenv][31]

[3]: https://aka.ms/botframework-emulator
[5]: https://github.com/microsoft/botframework-emulator
[6]: https://github.com/Microsoft/BotFramework-Emulator/releases
[7]: https://docs.microsoft.com/azure/bot-service/bot-builder-storage-concept
[8]: https://docs.microsoft.com/azure/bot-service/bot-builder-howto-v4-storage?tabs=javascript
[9]: https://docs.microsoft.com/azure/bot-service/bot-builder-howto-v4-state?tabs=javascript
[21]: https://docs.microsoft.com/azure/bot-service/bot-service-overview-introduction?view=azure-bot-service-4.0
[30]: https://www.npmjs.com/package/restify
[31]: https://www.npmjs.com/package/dotenv
[40]: https://aka.ms/azuredeployment
