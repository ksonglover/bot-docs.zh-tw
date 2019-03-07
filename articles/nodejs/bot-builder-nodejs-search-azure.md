---
title: 使用 Azure 搜尋服務建立資料驅動體驗 | Microsoft Docs
description: 了解如何使用 Azure 搜尋服務建立資料驅動體驗，並協助使用者在採用適用於 Node.js 的 Bot Framework SDK 和 Azure 搜尋服務的 Bot 中巡覽大量內容。
author: matthewshim-ms
ms.author: v-shimma
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 259f709ae460fde13cdf25ce6d7cbf5dd44a333d
ms.sourcegitcommit: cf3786c6e092adec5409d852849927dc1428e8a2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/01/2019
ms.locfileid: "57224856"
---
# <a name="create-data-driven-experiences-with-azure-search"></a>使用 Azure 搜尋服務建立資料驅動體驗 

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-search-azure.md)
> - [Node.js](../nodejs/bot-builder-nodejs-search-azure.md)

您可以將 [Azure 搜尋服務][search]新增至 Bot，以協助使用者巡覽大量內容，並為 Bot 的使用者建立資料驅動的探索體驗。

Azure 搜尋服務是一項 Azure 服務，可提供關鍵字搜尋、內建語言、自訂評分、多面向導覽等等。 Azure 搜尋服務也可以對各種來源 (包括 Azure SQL DB、DocumentDB、Blob 儲存體和表格儲存體) 的內容編製索引。 它支援「推送」其他資料來源的編製索引，而且可以開啟 PDF、Office 文件，以及其他包含非結構化資料的格式。 收集後，內容就會進入 Azure 搜尋服務索引，然後 Bot 就可加以查詢。

## <a name="install-dependencies"></a>安裝相依性

在命令提示字元中，巡覽至 Bot 的專案目錄，並使用 Node Package Manager (NPM) 安裝下列模組：

* [bluebird](https://www.npmjs.com/package/bluebird)
* [lodash](https://www.npmjs.com/package/lodash)
* [要求](https://www.npmjs.com/package/request)

## <a name="prerequisites"></a>必要條件

下列是**必要項目**： 
- 具備 Azure 訂用帳戶和 Azure 搜尋服務主要金鑰。 您可以在 Azure 入口網站中找到這個值。
- 將 [SearchDialogLibrary](https://github.com/Microsoft/botBuilder-Samples/tree/master/Node/demo-Search/SearchDialogLibrary) 程式庫複製到 Bot 的專案目錄。 此程式庫包含供使用者進行搜尋，但可以視需要自訂以符合 Bot 的一般對話。 

- 將 [SearchProviders](https://github.com/Microsoft/botBuilder-Samples/tree/master/Node/demo-Search/SearchProviders) 程式庫複製到 Bot 的專案目錄。 此程式庫包含建立要求並將其提交至 Azure 搜尋服務所需的所有元件。

## <a name="connect-to-the-azure-service"></a>連線到 Azure 服務 

在 Bot 的主程式檔中 (例如：app.js)，建立兩個先前已安裝之程式庫的參考路徑。 

```javascript
var SearchLibrary = require('./SearchDialogLibrary');
var AzureSearch = require('./SearchProviders/azure-search');
```

將下列程範例程式新增至 Bot。 在 `AzureSearch` 物件中，將您自己的 Azure 搜尋服務設定傳入 `.create` 方法。 在執行階段，這會將 Bot 繫結至 Azure 搜尋服務，並等待 `Promise` 形式的已完成使用者查詢。  

```javascript
// Azure Search
var azureSearchClient = AzureSearch.create('Your-Azure-Search-Service-Name', 'Your-Azure-Search-Primary-Key', 'Your-Azure-Search-Service-Index');
var ResultsMapper = SearchLibrary.defaultResultsMapper(ToSearchHit);
```

 `azureSearchClient` 參考會建立 Azure 搜尋服務模型，該模型則透過 Bot 的 `.env` 設定傳遞 Azure 服務的授權設定。 
 `ResultsMapper` 會剖析 Azure 回應物件，並對應我們在 `ToSearchHit` 方法中定義的資料。 如需這個方法的實作範例，請參閱 [Azure 搜尋服務回應之後](#after-azure-search-responds)。

## <a name="register-the-search-library"></a>註冊搜尋程式庫
您可以在 `SearchLibrary` 模組本身中直接自訂搜尋對話方塊。 `SearchLibrary` 會執行大部分的繁重工作，包括呼叫 Azure 搜尋服務。 

請在 Bot 的主程式檔中新增下列程式碼，以便向 Bot 登錄搜尋對話程式庫。 

```javascript
bot.library(SearchLibrary.create({
    multipleSelection: true,
    search: function (query) { return azureSearchClient.search(query).then(ResultsMapper); },
    refiners: ['refiner1', 'refiner2', 'refiner3'], // customize your own refiners 
    refineFormatter: function (refiners) {
        return _.zipObject(
            refiners.map(function (r) { return 'By ' + _.capitalize(r); }),
            refiners);
    }
}));
```
`SearchLibrary` 不僅會儲存所有與搜尋相關的對話，還會採用使用者查詢以提交至 Azure 搜尋服務。 您必須在 `refiners` 陣列中定義自己的精簡器，以指定您想要讓使用者縮小搜尋結果範圍或篩選搜尋結果的實體。  

## <a name="create-a-search-dialog"></a>建立搜尋對話

您可以在需要時選擇建構對話。 設定 Azure 搜尋服務對話的唯一需求是透過 `SearchLibrary` 物件叫用 `.begin` 方法，以傳入 Bot Framework SDK 所產生的 `session` 物件。 

```javascript
function (session) {
        // Trigger Azure Search dialogs 
        SearchLibrary.begin(session);
    },
    function (session, args) {
        // Process selected search results
        session.send(
            'Search Completed!',
            args.selection.map(  ); // format your response 
    }
```
如需對話的詳細資訊，請參閱[使用對話來管理交談](bot-builder-nodejs-dialog-manage-conversation.md)。

## <a name="after-azure-search-responds"></a>Azure 搜尋服務回應之後 

Azure 搜尋服務成功解析之後，您現在需要從回應物件儲存想要的資料，並以對使用者有意義的方式顯示它。

> [!TIP]
> 請考慮包含 [util 模組][NodeUtil]。 它會協助您進行格式化，並對應來自 Azure 搜尋服務的回應。

在 Bot 的主程式檔中，建立 `ToSearchHit` 方法。 這個方法會傳回一個物件，而該物件將格式化 Azure 回應中所需的相關資料。 下列程式碼示範如何在 `ToSearchHit` 方法中定義您自己的參數。 
 
 ```javascript
 function ToSearchHit(azureResponse) {
     return {
         // define your own parameters 
         key: azureResponse.id,
         title: azureResponse.title,
         description: azureResponse.description,
         imageUrl: azureResponse.thumbnail
     };
 }
```
此動作完成之後，您只需要向使用者顯示資料即可。 

 在 **SearchDialogLibrary** 專案的 **index.js** 檔案中，`searchHitAsCard` 方法會剖析來自 Azure 搜尋服務的每個回應，並建立要向使用者顯示的新卡片物件。 您在 Bot 主程式檔之 `ToSearchHit` 方法中所定義的欄位，需要與 `searchHitAsCard` 方法中的屬性進行同步處理。 

以下顯示如何及在何處使用您透過 `ToSearchHit` 方法定義的參數，來建置要呈現給使用者的卡片附件 UI。 

```javascript
function searchHitAsCard(showSave, searchHit) {
    var buttons = showSave
        ? [new builder.CardAction().type('imBack').title('Save').value(searchHit.key)]
        : [];

    var card = new builder.HeroCard()
        .title(searchHit.title) 
        .buttons(buttons);

    if (searchHit.description) {
        card.subtitle(searchHit.description);
    }

    if (searchHit.imageUrl) {
        card.images([new builder.CardImage().url(searchHit.imageUrl)]);
    }

    return card;
}
```

## <a name="sample-code"></a>範例程式碼

如需兩個示範如何使用適用於 Node.js 的 Bot Framework SDK 支援 Azure 搜尋服務的完整範例，請參閱 GitHub 中的[不動產 Bot 範例](https://github.com/Microsoft/BotBuilder-Samples/tree/v3-sdk-samples/Node/demo-Search/RealEstateBot)或[作業清單 Bot 範例](https://github.com/Microsoft/BotBuilder-Samples/tree/v3-sdk-samples/Node/demo-Search/JobListingBot)。 

## <a name="additional-resources"></a>其他資源

* [Azure 搜尋服務][search]
* [Node Util][NodeUtil]
* [對話](bot-builder-nodejs-dialog-manage-conversation.md)

[NodeUtil]: https://nodejs.org/api/util.html
[search]: /azure/search/search-what-is-azure-search
