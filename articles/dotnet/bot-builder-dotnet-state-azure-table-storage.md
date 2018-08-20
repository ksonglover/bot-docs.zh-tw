---
title: 使用 Azure 表格儲存體來管理自訂狀態資料 | Microsoft Docs
description: 了解如何使用 Azure 表格儲存體，搭配適用於 .NET 的 Bot 建立器 SDK 來儲存和擷取狀態資料
author: kaiqb
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: e2d8e6a5a390a27b61b11ad22f07ce0ab95f1686
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/18/2018
ms.locfileid: "39300223"
---
# <a name="manage-custom-state-data-with-azure-table-storage-for-net"></a>使用適用於 .NET 的 Azure 表格儲存體來管理自訂狀態資料
在本文中，您將實作 Azure 表格儲存體來儲存和管理 Bot 的狀態資料。 Bot 所使用的預設「連接器狀態服務」不適用於生產環境。 您應使用 GitHub 上提供的 [Azure 延伸模組](https://github.com/Microsoft/BotBuilder-Azure) (英文)，或使用您選擇的資料儲存體平台來實作自訂狀態用戶端。 以下是使用自訂狀態儲存體的一些原因：
 - 可獲得更高的狀態 API 輸送量 (更充分掌控效能)
 - 可降低地理分佈的延遲
 - 可控制資料儲存位置
 - 可存取實際的狀態資料
 - 可儲存超過 32KB 的資料

## <a name="prerequisites"></a>必要條件
您需要：
 - [Microsoft Azure 帳戶](https://azure.microsoft.com/en-us/free/)
 - [Visual Studio 2015 或更新版本](https://www.visualstudio.com/)
 - [Bot 建立器 Azure NuGet 套件](https://www.nuget.org/packages/Microsoft.Bot.Builder.Azure/) (英文)
 - [Autofac Web Api2 NuGet 套件](https://www.nuget.org/packages/Autofac.WebApi2/) (英文)
 - [Bot Framework 模擬器](https://emulator.botframework.com/) (英文)
 - [Azure 儲存體總管](http://storageexplorer.com/)
 
## <a name="create-azure-account"></a>建立 Azure 帳戶
如果您沒有 Azure 帳戶，按一下[這裡](https://azure.microsoft.com/en-us/free/)註冊免費帳戶。

## <a name="set-up-the-azure-table-storage-service"></a>設定 Azure 表格儲存體服務
1. 在您登入 Azure 入口網站之後，請按一下 [新增] 以建立新的 Azure 表格儲存體服務。 
2. 搜尋會實作 Azure 表格的**儲存體帳戶**。 
3. 填寫欄位，按一下畫面底部的 [建立] 按鈕來部署新的儲存體服務。 部署新的儲存體服務之後，它會顯示可用的功能和選項。
4. 選取左側的 [存取金鑰] 索引標籤，並複製連接字串以供稍後使用。 您的 Bot 會使用此連接字串，來呼叫儲存體服務以儲存狀態資料。

## <a name="install-nuget-packages"></a>安裝 NuGet 套件
1. 開啟現有的 C# Bot 專案，或使用 Visual Studio 中的 C# Bot 範本建立新專案。 
2. 安裝下列 NuGet 封裝：
   - Microsoft.Bot.Builder.Azure
   - Autofac.WebApi2

## <a name="add-connection-string"></a>新增連接字串 
在 Web.config 檔案中新增下列項目： 
```XML
  <connectionStrings>
    <add name="StorageConnectionString"
    connectionString="YourConnectionString"/>
  </connectionStrings>
```
將 "YourConnectionString" 取代為您稍早儲存的 Azure 表格儲存體連接字串。 儲存 Web.config 檔案。

## <a name="modify-your-bot-code"></a>修改您的 Bot 程式碼
在 Global.asax.cs 檔案中新增下列 `using` 陳述式：
```cs
using Autofac;
using System.Configuration;
using Microsoft.Bot.Connector;
using Microsoft.Bot.Builder.Azure;
using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Bot.Builder.Dialogs.Internals;
```
在 `Application_Start()` 方法中建立 `TableBotDataStore` 類別的執行個體。 `TableBotDataStore` 類別會實作 `IBotDataStore<BotData>` 介面。 `IBotDataStore` 介面可讓您覆寫預設的連接器狀態服務連線。
 ```cs
 var store = new TableBotDataStore(ConfigurationManager.ConnectionStrings["StorageConnectionString"].ConnectionString);
 ```
註冊服務，如下所示：
 ```cs
 Conversation.UpdateContainer(
            builder =>
            {
                builder.Register(c => store)
                          .Keyed<IBotDataStore<BotData>>(AzureModule.Key_DataStore)
                          .AsSelf()
                          .SingleInstance();

                builder.Register(c => new CachingBotDataStore(store,
                           CachingBotDataStoreConsistencyPolicy
                           .ETagBasedConsistency))
                           .As<IBotDataStore<BotData>>()
                           .AsSelf()
                           .InstancePerLifetimeScope();

                
            });
 ```
儲存 Global.asax.cs 檔案。

## <a name="run-your-bot-app"></a>執行 Bot 應用程式
在 Visual Studio 中執行 Bot，您所新增的程式碼會在 Azure 中建立自訂的 **botdata** 表格。

## <a name="connect-your-bot-to-the-emulator"></a>將 Bot 連接到模擬器
此時，Bot 正在本機執行。 接下來，啟動模擬器，然後在模擬器中連接到您的 Bot：
1. 將 http://localhost:port-number/api/messages 輸入網址列，其中連接埠號碼必須符合應用程式執行所在的瀏覽器中，所顯示的連接埠號碼。 您目前可以將 <strong>Microsoft 應用程式識別碼</strong>和 <strong>Microsoft 應用程式密碼</strong>欄位留白。 稍後[註冊 Bot](~/bot-service-quickstart-registration.md) 時，您會取得這些資訊。
2. 按一下 [ **連接**]。 
3. 在模擬器中輸入一些訊息以測試您的 Bot。 

## <a name="view-data-in-azure-table-storage"></a>在 Azure 表格儲存體中檢視資料
若要檢視狀態資料，請開啟**儲存體總管**並使用 Azure 入口網站認證來連線至 Azure，或使用儲存體名稱和儲存體金鑰直接連線至表格，然後瀏覽至您的表格名稱。  

## <a name="next-steps"></a>後續步驟
在本文中，您會實作 Azure 表格儲存體來儲存及管理 Bot 的資料。 接下來，了解如何使用對話方塊將對話流程模型化。

> [!div class="nextstepaction"]
> [管理對話流程](bot-builder-dotnet-manage-conversation-flow.md)


## <a name="additional-resources"></a>其他資源

如果您不熟悉上述程式碼中所使用的「控制項反轉」容器和「相依性插入」模式，請移至 [Autofac](http://autofac.readthedocs.io/en/latest/) (英文) 網站以取得資訊。 

您也可以從 GitHub 下載完整的 [Azure 表格儲存體](https://github.com/Microsoft/BotBuilder-Azure/tree/master/CSharp/Samples/AzureTable) (英文) 範例。
