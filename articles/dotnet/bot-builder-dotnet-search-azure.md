---
title: Azure Search を使用してデータドリブン エクスペリエンスを作成する | Microsoft Docs
description: Azure Search を使用してデータドリブン エクスペリエンスを作成し、ユーザーが Bot Builder SDK for .NET および Azure Search を使用してボット内の大量のコンテンツ間を移動できるように支援する方法について説明します。
author: matthewshim-ms
ms.author: v-shimma
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 5d6c5200fc10bb7c49df7515440daac351640ae0
ms.sourcegitcommit: 2dc75701b169d822c9499e393439161bc87639d2
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 08/24/2018
ms.locfileid: "42904953"
---
# <a name="create-data-driven-experiences-with-azure-search"></a>Azure Search を使用してデータドリブン エクスペリエンスを作成する 

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-search-azure.md)
> - [Node.js](../nodejs/bot-builder-nodejs-search-azure.md)

ボットに [Azure Search](https://azure.microsoft.com/en-us/services/search/) を追加すると、ユーザーが大量のコンテンツ間を移動し、データドリブン探索エクスペリエンスを作成できるように支援できます。

Azure Search は、キーワード検索、組み込みの言語機能、カスタム スコアリング、ファセット ナビゲーションなど多くの機能を提供する Azure サービスです。 Azure Search はまた、Azure SQL DB、DocumentDB、Blob Storage、Table Storage など、さまざまなソースのコンテンツのインデックスを作成することもできます。 これは他のデータ ソースの "プッシュ" インデックス作成をサポートしており、PDF、Office ドキュメント、および非構造化データが含まれたその他の形式を開くことができます。 収集されると、これらのコンテンツは Azure Search によってインデックス化され、ボットが照会できるようになります。


## <a name="prerequisites"></a>前提条件

ボット プロジェクトに [Microsoft.Azure.Search](https://www.nuget.org/packages/Microsoft.Azure.Search/4.0.0-preview) Nuget パッケージをインストールします。 

ボットのソリューションには、次の 3 つの C# プロジェクトが必要です。 これらのプロジェクトは、ボットと Azure Search に追加機能を提供します。 [GitHub](https://github.com/Microsoft/botBuilder-Samples/tree/master/CSharp/demo-Search) からプロジェクトをフォークするか、またはソース コードを直接ダウンロードします。

* [Search.Azure](https://github.com/Microsoft/botBuilder-Samples/tree/master/CSharp/demo-Search/Search.Azure) は、Azure サービスの呼び出しを定義します。 
* [Search.Contracts](https://github.com/Microsoft/botBuilder-Samples/tree/master/CSharp/demo-Search/Search.Contracts) は、データを処理するための汎用インターフェイスとデータ モデルを定義します。
* [Search.Dialogs](https://github.com/Microsoft/botBuilder-Samples/tree/master/CSharp/demo-Search/Search.Dialogs) には、Azure Search にクエリを実行するために使用されるさまざまな汎用の Bot Builder ダイアログが含まれています。

## <a name="configure-azure-search-settings"></a>Azure Search の設定を構成する 

値フィールドで独自の Azure Search 資格情報を使用して、プロジェクトの **Web.config** ファイル内の Azure Search の設定を構成します。 `AzureSearchClient` クラスのコンストラクターは、これらの設定を使用して、ボットを Azure サービスに登録およびバインドします。

```xml
<appSettings>
    <add key="SearchDialogsServiceName" value="Azure-Search-Service-Name" /> <!-- replace value field with Azure Service Name --> 
    <add key="SearchDialogsServiceKey" value="Azure-Search-Service-Primary-Key" /> <!-- replace value field with Azure Service Key --> 
    <add key="SearchDialogsIndexName" value="Azure-Search-Service-Index" /> <!-- replace value field with your Azure Search Index --> 
</appSettings>
```

## <a name="create-a-search-dialog"></a>検索ダイアログを作成する

ボットのプロジェクトで、ボットで Azure サービスを呼び出すための新しい `AzureSearchDialog` クラスを作成します。 この新しいクラスは、負荷の高い作業のほとんどを処理する **Search.Dialogs** プロジェクトから `SearchDialog` クラスを継承する必要があります。 `GetTopRefiners()` のオーバーライドによって、ユーザーは最初から検索を開始しなくても検索結果の絞り込みやフィルター処理が可能になり、それにより検索オブジェクトの状態が保持されます。 `TopRefiners` の配列内に独自のカスタム リファイナーを追加すると、ユーザーは検索結果をフィルター処理したり、絞り込んだりできます。 

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

## <a name="define-the-response-data-model"></a>応答データ モデルを定義する

`Search.Contracts` プロジェクト内の **SearchHit.cs** クラスは、Azure Search の応答から解析される関連データを定義します。 ボット用に含めるべき内容は、`PropertyBag` IDictionary の宣言とコンストラクターでの作成だけです。 このクラスでは、その他のすべてのプロパティをボットのニーズに従って定義できます。 

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

## <a name="after-azure-search-responds"></a>Azure Search が応答した後 

Azure サービスへのクエリが成功したら、ユーザーに表示するボットの関連データを取得するために検索結果を解析する必要があります。 これを可能にするには、`SearchResultMapper` クラスを作成する必要があります。 コンストラクターで作成された `GenericSearchResult` オブジェクトは、ボットのデータ モデルに対応して各結果が解析された後に、それぞれ結果とファセットを格納する一覧と辞書を定義します。 

**SearchHit.cs** のデータ モデルと一致させるために、`ToSearchHit` メソッドでプロパティを同期します。 `ToSearchHit` メソッドは実行されると、応答で見つかったすべての結果の新しい `SearchHit` を生成します。  

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
結果が解析されて格納された後、その情報を引き続きユーザーに表示する必要があります。 `SearchHit` クラスのデータ モデルに対応するには、`SearchHitStyler` クラスを管理する必要があります。 たとえば、サンプル コードでは、新しいカード添付ファイルを作成するために **SearchHit.cs** クラスの `Title`、`PictureUrl`、および `Description` プロパティが使用されています。 次のコードは、ユーザーに表示する `GenericSearchResult` 結果の一覧内のすべての `SearchHit` オブジェクトの新しいカード添付ファイルを作成します。   

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
検索結果がユーザーに表示され、これで Azure Search がボットに正常に追加されました。

## <a name="samples"></a>サンプル

Bot Builder SDK for .NET を使用してボットで Azure Search をサポートする方法を示す 2 つの完全なサンプルについては、GitHub にある[不動産ボット サンプル](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/demo-Search/RealEstateBot)または[求人情報ボット サンプル](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/demo-Search/JobListingBot)を参照してください。 

## <a name="additional-resources"></a>その他のリソース
* [Azure Search][search]
* [ダイアログの概要](bot-builder-dotnet-dialogs.md)
* [Azure Search ボット サンプル](https://github.com/Microsoft/botBuilder-Samples/tree/master/CSharp/demo-Search)

[search]: /azure/search/search-what-is-azure-search
