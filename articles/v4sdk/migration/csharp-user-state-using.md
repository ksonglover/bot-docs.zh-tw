---
title: 在 v4 Bot 中使用 .NET v3 使用者狀態 | Microsoft Docs
description: 如何在 v4 Bot 中使用 v3 使用者狀態的範例
keywords: Csharp, bot 移轉, v3 bot
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 08/21/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 1bae27b9cf2bdc6a53a5c55fe7458802729160c1
ms.sourcegitcommit: 008aa6223aef800c3abccda9a7f72684959ce5e7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/26/2019
ms.locfileid: "70026697"
---
# <a name="using-net-v3-user-state-in-a-v4-bot"></a>在 v4 bot 中使用 .NET v3 使用者狀態

本文說明 v4 Bot 如何對 v3 使用者狀態資訊執行讀取、寫入和刪除作業的範例。
Bot 會使用 `MemoryStorage` 維護對話狀態以追蹤及引導對話，並向使用者詢問問題。  其會使用名為 `V3V4Storage` 的自訂 `IStorage` 類別以 v3 格式維護**使用者狀態**，以追蹤使用者的答案。  此類別的其中一個引數是 `IBotDataStore`。 V3 SDK 程式碼基底已複製到 `Bot.Builder.Azure.V3V4` 中，且同時包含三個 v3 SDK 儲存體提供者 (Azure Sql、Azure 資料表和 Cosmos Db)。  其目的是要允許現有的 v3 **使用者狀態**導入已遷移的 v4 Bot 中。

您可以在[這裡](https://github.com/microsoft/BotBuilder-Samples/tree/master/MigrationV3V4/CSharp/V4StateBotFromV3Providers)找到程式碼範例。

## <a name="prerequisites"></a>必要條件

- [.NET Core SDK](https://dotnet.microsoft.com/download) 2.1 版

    ```bash
    # determine dotnet version
    dotnet --version
    ```

## <a name="setup"></a>設定

1. 複製儲存機制

    ```bash
    git clone https://github.com/microsoft/botbuilder-samples.git
    ```

1. 在終端機中，瀏覽至 `MigrationV3V4/CSharp/V4StateBotFromV3Providers`

    ```bash
    cd BotBuilder-Samples/MigrationV3V4/CSharp/V4StateBotFromV3Providers
    ```

1. 選擇選項 A 或 B，從終端機或 Visual Studio 執行 Bot。

    - 從終端機

        ```bash
        # run the bot
        dotnet run
        ```

    - 或從 Visual Studio

        - 啟動 Visual Studio，選取 [檔案] -> [開啟] -> [專案/解決方案]
        - 瀏覽至 `BotBuilder-Samples/MigrationV3V4/CSharp/V4StateBotFromV3Providers` 資料夾
        - 選取 `V4StateBot.sln` 檔案
        - 按 F5 以執行專案


## <a name="storage-provider-setup"></a>儲存體提供者設定

我們假設您有已設定的 v3 狀態存放區，且正在使用中。 在此情況下，要將此範例設定為使用現有的狀態存放區，只需將儲存體提供者的連線資訊新增至 `web.config` 檔案即可，如下所示。

- Cosmos DB

```json
  "v3CosmosEndpoint": "https://yourcosmosdb.documents.azure.com:443/",
  "v3CosmosKey": "YourCosmosDbKey",
  "v3CosmosDatataseName": "v3botdb",
  "v3CosmosCollectionName": "v3botcollection",
```

- Azure Sql

```json
 "ConnectionStrings": {
    "SqlBotData": "Server=YourServer;Initial Catalog=BotData;Persist Security Info=False;User ID=YourUserName;Password=YourUserPassword;MultipleActiveResultSets=False;Encrypt=True;TrustServerCertificate=True;Connection Timeout=30;"
  },
```

- Azure 資料表

```json
 "ConnectionStrings": {
    "AzureTable": "DefaultEndpointsProtocol=https;AccountName=YourAccountName;AccountKey=YourAccountKey;EndpointSuffix=core.windows.net"
  },
```

- 設定 Bot 的儲存體提供者

    開啟 `V4V3StateBot` 專案根目錄中的 `Startup.cs` 檔案。 在檔案的中段 (52-76 行左右)，您會看到每個儲存體提供者的組態。 其會從 web.config 讀取組態值。 

    [!code-csharp[Storage configuration](~/../botbuilder-samples/MigrationV3V4/CSharp/V4StateBotFromV3Providers/V4V3StateBot/Startup.cs?range=52-76)]

    藉由將您選擇的執行個體所對應的數行取消註解，以指定您要讓 Bot 使用的儲存體提供者。 正確設定提供者之後，請確定提供者類別已傳至 `V3V4Storage` (72-75 行左右)。 

    [!code-csharp[Storage provider](~/../botbuilder-samples/MigrationV3V4/CSharp/V4StateBotFromV3Providers/V4V3StateBot/Startup.cs?range=72-75)]

    依預設會設定 Cosmos DB (先前稱為 Document DB)。 可能的值包括：

    ```bash
    documentDbBotDataStore
    tableBotDataStore
    tableBotDataStore2
    sqlBotDataStore
    ```

- 啟動應用程式。 

## <a name="v3v4-storage-and-state-classes"></a>V3V4 儲存體和狀態類別

### <a name="v3v4storage"></a>V3V4Storage

`V3V4Storage` 類別包含主要儲存體對應功能。 其會實作 v4 `IStorage` 介面，並將儲存體提供者方法 (讀取、寫入和刪除) 對應回 v3 儲存體提供者類別，讓您可以從 v4 Bot 使用 v3 格式的使用者狀態。

### <a name="v3v4state"></a>V3V4State

此類別繼承自 v4 `BotState` 類別，並使用 v3 樣式的金鑰 (`IAddress`)。 這可讓您以 V3 狀態儲存體一貫採用的相同方式，對 v3 儲存體進行讀取、寫入和刪除。


## <a name="testing-the-bot-using-bot-framework-emulator"></a>使用 Bot Framework Emulator 測試 Bot

[Bot Framework Emulator][5] 是一個桌面應用程式，可讓 Bot 開發人員在本機或透過通道從遠端對其 Bot 進行測試和偵錯。

- 從[這裡][6]安裝 Bot Framework Emulator 4.3.0 版或更高版本


### <a name="connect-to-the-bot-using-bot-framework-emulator"></a>使用 Bot Framework Emulator 連線至 Bot

- 啟動 Bot Framework Emulator
- 檔案 -> 開啟 Bot
- 輸入 Bot URL `http://localhost:3978/api/messages`


## <a name="further-reading"></a>進階閱讀

- [Azure Bot 服務簡介][21]
- [Bot 狀態][7]
- [直接寫入儲存體][8]
- [管理對話和使用者狀態][9]

[3]: https://aka.ms/botframework-emulator
[5]: https://github.com/microsoft/botframework-emulator
[6]: https://github.com/Microsoft/BotFramework-Emulator/releases
[7]: https://docs.microsoft.com/azure/bot-service/bot-builder-storage-concept
[8]: https://docs.microsoft.com/azure/bot-service/bot-builder-howto-v4-storage?tabs=csharp
[9]: https://docs.microsoft.com/azure/bot-service/bot-builder-howto-v4-state?tabs=csharp
[21]: https://docs.microsoft.com/azure/bot-service/bot-service-overview-introduction?view=azure-bot-service-4.0
[40]: https://aka.ms/azuredeployment
