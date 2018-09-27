---
title: 使用 Azure Cosmos DB 管理自訂狀態資料 | Microsoft Docs
description: 了解如何使用 Azure Cosmos DB 搭配適用於 .NET 的 Bot Builder SDK 來儲存和擷取狀態資料
author: kaiqb
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 5951df2fa7e89cce03bd8b7f9404585f720a9247
ms.sourcegitcommit: 3bf3dbb1a440b3d83e58499c6a2ac116fe04b2f6
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/23/2018
ms.locfileid: "46707304"
---
# <a name="manage-custom-state-data-with-azure-cosmos-db-for-net"></a>使用適用於 .NET 的 Azure Cosmos DB 管理自訂狀態資料

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

在本文中，您將實作 Azure Cosmos DB 來儲存和管理 Bot 的狀態資料。 Bot 所使用的預設連接器狀態服務不適用於生產環境。 您應使用 GitHub 上提供的 [Azure 延伸模組](https://github.com/Microsoft/BotBuilder-Azure) (英文)，或使用您選擇的資料儲存體平台來實作自訂狀態用戶端。 以下列舉應使用自訂狀態儲存體的若干原因：
 - 可獲得更高的狀態 API 輸送量 (更充分掌控效能)
 - 可降低地理分佈的延遲
 - 可控制資料儲存位置
 - 可存取實際的狀態資料
 - 儲存超過 32 KB 的資料
 
## <a name="prerequisites"></a>必要條件
您需要：
 - [Microsoft Azure 帳戶](https://azure.microsoft.com/en-us/free/)
 - [Visual Studio 2015 或更新版本](https://www.visualstudio.com/)
 - [Bot Builder Azure NuGet 套件](https://www.nuget.org/packages/Microsoft.Bot.Builder.Azure/)
 - [Autofac Web Api2 NuGet 套件](https://www.nuget.org/packages/Autofac.WebApi2/) (英文)
 - [Bot Framework 模擬器](~/bot-service-debug-emulator.md) (英文)
 
## <a name="create-azure-account"></a>建立 Azure 帳戶
如果您沒有 Azure 帳戶，請按一下[這裡](https://azure.microsoft.com/en-us/free/)註冊免費帳戶。

## <a name="set-up-the-azure-cosmos-db-database"></a>設定 Azure Cosmos DB 資料庫
1. 在您登入 Azure 入口網站之後，按一下 [新增] 以建立新的 *Azure Cosmos DB* 資料庫。 
2. 按一下 [資料庫]。 
3. 找到 **Azure Cosmos DB**，然後按一下 [建立]。
4. 填寫欄位。 如果是 [**API**] 欄位，請選取 [**SQL (DocumentDB)**]。 填妥所有欄位後，按一下畫面底部的 [建立] 按鈕來部署新的資料庫。 
5. 新資料庫部署完畢後，接著瀏覽至新資料庫。 按一下 [存取金鑰] 以尋找金鑰和連接字串。 您的 Bot 會使用此資訊來呼叫儲存體服務以儲存狀態資料。

## <a name="install-nuget-packages"></a>安裝 NuGet 套件
1. 開啟現有的 C# Bot 專案，或使用 Visual Studio 中的 Bot 範本建立新專案。 
2. 安裝下列 NuGet 封裝：
   - Microsoft.Bot.Builder.Azure
   - Autofac.WebApi2

## <a name="add-connection-string"></a>新增連接字串 
在 Web.config 檔案中新增下列項目：
```XML
<add key="DocumentDbUrl" value="Your DocumentDB URI"/>
<add key="DocumentDbKey" value="Your DocumentDB Key"/>
```
請將值替換為您的 URI 和您在 Azure Cosmos DB 中的主索引鍵。 儲存 Web.config 檔案。

## <a name="modify-your-bot-code"></a>修改您的 Bot 程式碼
若要使用 **Azure Cosmos DB** 儲存體，將下列程式碼行新增至 **Application_Start()** 方法中 Bot 的 **Global.asax.cs** 檔案。

```cs
using System;
using Autofac;
using System.Web.Http;
using System.Configuration;
using Microsoft.Bot.Connector;
using Microsoft.Bot.Builder.Azure;
using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Bot.Builder.Dialogs.Internals;

namespace SampleApp
{
    public class WebApiApplication : System.Web.HttpApplication
    {
        protected void Application_Start()
        {
            GlobalConfiguration.Configure(WebApiConfig.Register);
            var uri = new Uri(ConfigurationManager.AppSettings["DocumentDbUrl"]);
            var key = ConfigurationManager.AppSettings["DocumentDbKey"];
            var store = new DocumentDbBotDataStore(uri, key);

            Conversation.UpdateContainer(
                        builder =>
                        {
                            builder.Register(c => store)
                                .Keyed<IBotDataStore<BotData>>(AzureModule.Key_DataStore)
                                .AsSelf()
                                .SingleInstance();

                            builder.Register(c => new CachingBotDataStore(store, CachingBotDataStoreConsistencyPolicy.ETagBasedConsistency))
                                .As<IBotDataStore<BotData>>()
                                .AsSelf()
                                .InstancePerLifetimeScope();

                        });

        }
    }
}
```

儲存 global.asax.cs 檔案。 現在您已準備好使用模擬器測試 Bot。

## <a name="run-your-bot-app"></a>執行 Bot 應用程式
在 Visual Studio 中執行 Bot，您所新增的程式碼就會在 Azure 中建立自訂的 **botdata** 表格。

## <a name="connect-your-bot-to-the-emulator"></a>將 Bot 連線至模擬器
此時，Bot 正在本機執行。 接下來，請啟動模擬器，然後在模擬器中連線至您的 Bot：
1. 將 http://localhost:port-number/api/messages 輸入網址列，其中連接埠號碼必須符合應用程式執行所在的瀏覽器中，所顯示的連接埠號碼。 您目前可以將 <strong>Microsoft 應用程式識別碼</strong>和 <strong>Microsoft 應用程式密碼</strong>欄位留白。 稍後[註冊 Bot](~/bot-service-quickstart-registration.md) 時，您會取得這些資訊。
2. 按一下 [ **連接**]。 
3. 在模擬器中輸入一些訊息以測試您的 Bot。 

## <a name="view-state-data-on-azure-portal"></a>在 Azure 入口網站上檢視狀態資料
若要檢視狀態資料，請登入 Azure 入口網站並瀏覽至您的資料庫。 按一下 [資料總管 (預覽)] 來確認正在儲存的 Bot 狀態資訊。 

## <a name="next-steps"></a>後續步驟
在本文中，您可以使用 Cosmos DB 來儲存及管理您的 Bot 資料。 接下來，了解如何使用對話來模型化對話流程。

> [!div class="nextstepaction"]
> [管理對話流程](bot-builder-dotnet-manage-conversation-flow.md)

## <a name="additional-resources"></a>其他資源
如果您不熟悉上述程式碼中所使用的「控制項反轉」容器和「相依性插入」模式，請造訪 [Autofac](http://autofac.readthedocs.io/en/latest/) 網站以取得資訊。 

您也可以從 GitHub 下載[範例](https://github.com/Microsoft/BotBuilder-Azure/tree/master/CSharp/Samples/DocumentDb)，以深入了解如何使用 Cosmos DB 來管理狀態。 
