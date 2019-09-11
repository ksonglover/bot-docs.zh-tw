---
title: 使用 Azure 搜尋服務建立資料驅動體驗 | Microsoft Docs
description: 了解如何使用 Azure 搜尋服務建立資料驅動體驗，並協助使用者在採用適用於 .NET 的 Bot Framework SDK 和 Azure 搜尋服務的 Bot 中瀏覽大量內容。
author: matthewshim-ms
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 1/28/2019
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 2328357c372cabc186e589ccbf65f36517d4789c
ms.sourcegitcommit: a6d02ec4738e7fc90b7108934740e9077667f3c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/04/2019
ms.locfileid: "70297817"
---
# <a name="create-data-driven-experiences-with-azure-search"></a>使用 Azure 搜尋服務建立資料驅動體驗 

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-search-azure.md)
> - [Node.js](../nodejs/bot-builder-nodejs-search-azure.md)

您可以將 [Azure 搜尋服務](https://azure.microsoft.com/services/search/)新增至 Bot，以協助使用者瀏覽大量內容並建立資料驅動的探索體驗。

Azure 搜尋服務是一項 Azure 服務，可提供關鍵字搜尋、內建語言、自訂評分、多面向導覽等等。 Azure 搜尋服務也可以對各種來源 (包括 Azure SQL DB、DocumentDB、Blob 儲存體和表格儲存體) 的內容編製索引。 它支援「推送」其他資料來源的編製索引，而且可以開啟 PDF、Office 文件，以及其他包含非結構化資料的格式。 收集後，內容就會進入 Azure 搜尋服務索引，然後 Bot 就可加以查詢。

## <a name="prerequisites"></a>必要條件

在您的 Bot 專案中安裝 [Microsoft.Azure.Search](https://www.nuget.org/packages/Microsoft.Azure.Search/4.0.0-preview) Nuget 套件。

Bot 解決方案需要下列三個 C# 專案。 這些專案會為 Bot 與 Azure 搜尋服務提供額外功能。 從 [GitHub](https://aka.ms/v3-cs-search-demo) 分支處理專案或直接下載原始程式碼。

- **Search.Azure** 專案可定義 Azure 服務呼叫。
- **Search.Contracts** 專案可定義一般介面和資料模型來處理資料。
- **Search.Dialogs** 專案包含用來查詢 Azure 搜尋服務的一般 Bot Builder 對話方塊。

## <a name="configure-azure-search-settings"></a>進行 Azure 搜尋服務設定

在值欄位中使用自有的 Azure 搜尋服務認證，在專案的 **Web.config** 檔案中設定 Azure 搜尋服務設定。 `AzureSearchClient` 類別中的建構函式將使用這些設定來登錄並將 Bot 繫結至 Azure 服務。

```xml
<appSettings>
    <add key="SearchDialogsServiceName" value="Azure-Search-Service-Name" /> <!-- replace value field with Azure Service Name --> 
    <add key="SearchDialogsServiceKey" value="Azure-Search-Service-Primary-Key" /> <!-- replace value field with Azure Service Key --> 
    <add key="SearchDialogsIndexName" value="Azure-Search-Service-Index" /> <!-- replace value field with your Azure Search Index --> 
</appSettings>
```

## <a name="create-a-search-dialog"></a>建立搜尋對話方塊

在 Bot 的專案中，建立新的 `AzureSearchDialog` 類別以在 Bot 中呼叫 Azure 服務。 這個新類別必須從 **Search.Dialogs** 專案繼承 `SearchDialog` 類別，以處理大部分的苦差事。 `GetTopRefiners()` 覆寫可讓使用者縮小/篩選其搜尋結果，而不必從頭開始進行搜尋，並維護搜尋物件的狀態。 您可以在 `TopRefiners` 陣列中新增自有的自訂精簡器，讓使用者篩選或縮小其搜尋結果。 

```cs
[Serializable]
public class AzureSearchDialog : SearchDialog
{
    private static readonly string[] TopRefiners = { "refiner1", "refiner2", "refiner3" }; // define your own custom refiners 

    public AzureSearchDialog(ISearchClient searchClient) : base(searchClient, multipleSelection: true)
    {
    }

    protected override string[] GetTopRefiners()
    {
        return TopRefiners;
    }
}
```

## <a name="define-the-response-data-model"></a>定義回應資料模型

`Search.Contracts` 專案內的 **SearchHit.cs** 類別會定義要從 Azure 搜尋服務回應剖析的相關資料。 對 Bot 而言，唯一強制的包含項目為建構函式中的 `PropertyBag` IDictionary 宣告和建立。 您可以在有關 Bot 需求的這個類別中定義所有其他屬性。 

```cs
[Serializable]
public class SearchHit
{
    public SearchHit()
    {
        this.PropertyBag = new Dictionary<string, object>();
    }

    public IDictionary<string, object> PropertyBag { get; set; }

    // customize the fields below as needed 
    public string Key { get; set; }

    public string Title { get; set; }

    public string PictureUrl { get; set; }

    public string Description { get; set; }
}
```

## <a name="after-azure-search-responds"></a>Azure 搜尋服務回應之後 

成功查詢 Azure 服務時，就必須剖析搜尋結果來擷取相關資料，以便 Bot 向使用者顯示。 若要啟用此功能，您必須建立 `SearchResultMapper` 類別。 建構函式中所建立的 `GenericSearchResult` 物件可定義清單和字典，以在分別對 Bot 資料模型剖析每個結果之後，分別用來儲存結果和 Facet。 

同步處理 `ToSearchHit` 方法中的屬性，以符合 **SearchHit.cs** 中的資料模型。 `ToSearchHit` 方法將會執行，並針對在回應中找到每個結果產生新的 `SearchHit`。  

```cs
public class SearchResultMapper : IMapper<DocumentSearchResult, GenericSearchResult>
{
    public GenericSearchResult Map(DocumentSearchResult documentSearchResult)
    {
        var searchResult = new GenericSearchResult();

        searchResult.Results = documentSearchResult.Results.Select(r => ToSearchHit(r)).ToList();
        searchResult.Facets = documentSearchResult.Facets?.ToDictionary(kv => kv.Key, kv => kv.Value.Select(f => ToFacet(f)));

        return searchResult;
    }

    private static GenericFacet ToFacet(FacetResult facetResult)
    {
        return new GenericFacet
        {
            Value = facetResult.Value,
            Count = facetResult.Count.Value
        };
    }

    private static SearchHit ToSearchHit(SearchResult hit)
    {
        return new SearchHit
        {
            // custom properties defined in SearchHit.cs 
            Key = (string)hit.Document["id"],
            Title = (string)hit.Document["title"],
            PictureUrl = (string)hit.Document["thumbnail"],
            Description = (string)hit.Document["description"]
        };
    }
}
```
剖析並儲存結果之後，仍需要向使用者顯示此資訊。 需要管理 `SearchHitStyler` 類別，以配合 `SearchHit` 類別中您的資料模型。 例如，**SearchHit.cs** 類別中的 `Title`、`PictureUrl` 和 `Description` 屬性會使用於範例程式碼，以建立新的卡片附件。 下列程式碼會針對 `GenericSearchResult` 結果清單中的每個 `SearchHit` 物件，建立新的卡片附件以向使用者顯示。   

```cs
[Serializable]
public class SearchHitStyler : PromptStyler
{
    public override void Apply<T>(ref IMessageActivity message, string prompt, IReadOnlyList<T> options, IReadOnlyList<string> descriptions = null)
    {
        var hits = options as IList<SearchHit>;
        if (hits != null)
        {
            var cards = hits.Select(h => new ThumbnailCard
            {
                Title = h.Title,
                Images = new[] { new CardImage(h.PictureUrl) },
                Buttons = new[] { new CardAction(ActionTypes.ImBack, "Pick this one", value: h.Key) },
                Text = h.Description
            });

            message.AttachmentLayout = AttachmentLayoutTypes.Carousel;
            message.Attachments = cards.Select(c => c.ToAttachment()).ToList();
            message.Text = prompt;
        }
        else
        {
            base.Apply<T>(ref message, prompt, options, descriptions);
        }
    }
}
```
搜尋結果會向使用者顯示，而且您已成功將 Azure 搜尋服務新增至 Bot。

## <a name="samples"></a>範例

如需兩個示範如何使用適用於 .NET 的 Bot Framework SDK 支援 Azure 搜尋服務的完整 Bot 範例，請參閱 GitHub 中的[不動產 Bot 範例](https://github.com/Microsoft/BotBuilder-Samples/tree/v3-sdk-samples/CSharp/demo-Search/RealEstateBot)或[作業清單 Bot 範例](https://github.com/Microsoft/BotBuilder-Samples/tree/v3-sdk-samples/CSharp/demo-Search/JobListingBot)。 

## <a name="additional-resources"></a>其他資源

- [Azure 搜尋服務][search]
- [對話概觀](bot-builder-dotnet-dialogs.md)

[search]: /azure/search/search-what-is-azure-search
