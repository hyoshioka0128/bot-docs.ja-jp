---
title: Azure Search を使用してデータドリブン エクスペリエンスを作成する | Microsoft Docs
description: Azure Search を使用してデータドリブン エクスペリエンスを作成し、ユーザーが Bot Builder SDK for Node.js および Azure Search を使用してボット内の大量のコンテンツ間を移動できるように支援する方法について説明します。
author: matthewshim-ms
ms.author: v-shimma
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 1a50bb8af6556830ee9f9b047d7c5a2d3399a6b9
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/18/2018
ms.locfileid: "39302441"
---
# <a name="create-data-driven-experiences-with-azure-search"></a>Azure Search を使用してデータドリブン エクスペリエンスを作成する 
> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-search-azure.md)
> - [Node.js](../nodejs/bot-builder-nodejs-search-azure.md)

ボットに [Azure Search][search] を追加すると、ユーザーが大量のコンテンツ間を移動するのが容易になり、ボットのユーザーのためのデータドリブン探索エクスペリエンスを作成できます。

Azure Search は、キーワード検索、組み込みの言語機能、カスタム スコアリング、ファセット ナビゲーションなど多くの機能を提供する Azure サービスです。 Azure Search はまた、Azure SQL DB、DocumentDB、Blob Storage、Table Storage など、さまざまなソースのコンテンツのインデックスを作成することもできます。 これは他のデータ ソースの "プッシュ" インデックス作成をサポートしており、PDF、Office ドキュメント、および非構造化データが含まれたその他の形式を開くことができます。 収集されると、これらのコンテンツは Azure Search によってインデックス化され、ボットが照会できるようになります。

## <a name="install-dependencies"></a>依存関係をインストールする

コマンド プロンプトから、ボットのプロジェクト ディレクトリに移動し、ノード パッケージ マネージャー (NPM) を使用して次のモジュールをインストールします。

* [bluebird](https://www.npmjs.com/package/bluebird)
* [lodash](https://www.npmjs.com/package/lodash)
* [request](https://www.npmjs.com/package/request)

## <a name="prerequisites"></a>前提条件

**要件**を次に示します。 
- Azure サブスクリプションと Azure Search 主キーを用意します。 これは、Azure Portal で確認できます。
- [SearchDialogLibrary](https://github.com/Microsoft/botBuilder-Samples/tree/master/Node/demo-Search/SearchDialogLibrary) ライブラリをボットのプロジェクト ディレクトリにコピーします。 このライブラリにはユーザーが検索するための一般的なダイアログが含まれていますが、必要に応じてボットに合わせてカスタマイズできます。 

- [SearchProviders](https://github.com/Microsoft/botBuilder-Samples/tree/master/Node/demo-Search/SearchProviders) ライブラリをボットのプロジェクト ディレクトリにコピーします。 このライブラリには、要求を作成して Azure Search に送信するために必要なすべてのコンポーネントが含まれています。

## <a name="connect-to-the-azure-service"></a>Azure サービスに接続する 

ボットのメイン プログラム ファイル (app.js など) に、前にインストールした 2 つのライブラリへの参照パスを作成します。 

```javascript
var SearchLibrary = require('./SearchDialogLibrary');
var AzureSearch = require('./SearchProviders/azure-search');
```

次のサンプル コードをボットに追加します。 `AzureSearch` オブジェクトで、独自の Azure Search 設定を `.create` メソッドに渡します。 これにより、ボットは実行時に Azure Search サービスにバインドされ、`Promise` の形式で完了したユーザー クエリを待ちます。  

```javascript
// Azure Search
var azureSearchClient = AzureSearch.create('Your-Azure-Search-Service-Name', 'Your-Azure-Search-Primary-Key', 'Your-Azure-Search-Service-Index');
var ResultsMapper = SearchLibrary.defaultResultsMapper(ToSearchHit);
```

 `azureSearchClient` 参照によって Azure Search モデルが作成され、このモデルにより、ボットの `.env` 設定からの Azure サービスの認可設定が渡されます。 
 `ResultsMapper` は Azure 応答オブジェクトを解析し、`ToSearchHit` メソッドに定義したとおりにデータをマップします。 このメソッドの実装例については、「[Azure Search が応答した後](#after-azure-search-responds)」を参照してください。

## <a name="register-the-search-library"></a>検索ライブラリを登録する
検索ダイアログは、`SearchLibrary` モジュール内で直接カスタマイズできます。 `SearchLibrary` は、Azure Search の呼び出しを含む、複雑な処理のほとんどを実行します。 

ボットのメイン プログラム ファイルに次のコードを追加して、検索ダイアログ ライブラリをボットに登録します。 

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
`SearchLibrary` は、すべての検索関連のダイアログを格納するだけでなく、Azure Search に送信するユーザー クエリも受け取ります。 `refiners` 配列に独自の絞り込み条件を定義して、ユーザーが検索結果の絞り込みまたはフィルタリングを許可するエンティティを指定する必要があります。  

## <a name="create-a-search-dialog"></a>検索ダイアログを作成する

必要に応じて、ダイアログを自由に構成することができます。 Azure Search ダイアログを設定するための唯一の要件は、`SearchLibrary` オブジェクトから `.begin` メソッドを呼び出し、Bot Builder SDK によって生成された `session` オブジェクトを渡すことです。 

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
ダイアログの詳細については、[ダイアログを使用した会話の管理](bot-builder-nodejs-dialog-manage-conversation.md)に関するページを参照してください。

## <a name="after-azure-search-responds"></a>Azure Search が応答した後 

Azure Search が正常に解決されたら、応答オブジェクトから必要なデータを格納し、ユーザーにわかりやすい方法で表示する必要があります。

> [!TIP]
> [util モジュール][NodeUtil]を含めることを検討してください。 このモジュールは、Azure Search の応答を書式設定してマップするのに役立ちます。

ボットのメイン プログラム ファイルに、`ToSearchHit` メソッドを作成します。 このメソッドは、Azure 応答から必要な関連データを書式設定するオブジェクトを返します。 `ToSearchHit` メソッドに独自のパラメーターを定義する方法を次のコードに示します。 
 
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
これが完了したら、残りの必要な操作はデータをユーザーに表示することだけです。 

 **SearchDialogLibrary** プロジェクトの **index.js** ファイルで、`searchHitAsCard` メソッドは Azure Search からの各応答を解析し、ユーザーに表示する新しいカード オブジェクトを作成します。 ボットのメイン プログラム ファイルの `ToSearchHit` メソッドで定義したフィールドは、`searchHitAsCard` メソッドのプロパティと同期する必要があります。 

`ToSearchHit` メソッドの定義済みのパラメーターを使用して、ユーザーに表示するカード添付 UI を作成する方法と場所を次に示します。 

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

## <a name="sample-code"></a>サンプル コード

Bot Builder SDK for Node.js を使用してボットで Azure Search をサポートする方法を示す 2 つの完全なサンプルについては、GitHub の[不動産ボットのサンプル](https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/demo-Search/RealEstateBot)または[求人情報ボットのサンプル](https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/demo-Search/JobListingBot)を参照してください。 

## <a name="additional-resources"></a>その他のリソース

* [Azure Search][search]
* [Node Util][NodeUtil]
* [ダイアログ](bot-builder-nodejs-dialog-manage-conversation.md)

[NodeUtil]: https://nodejs.org/api/util.html
[search]: /azure/search/search-what-is-azure-search