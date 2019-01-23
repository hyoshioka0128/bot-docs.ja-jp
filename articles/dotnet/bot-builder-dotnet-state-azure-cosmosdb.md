---
title: Azure Cosmos DB を使用してカスタム状態データを管理する | Microsoft Docs
description: Bot Framework SDK for .NET で Azure Cosmos DB を使用して状態データを保存および取得する方法を説明します
author: kaiqb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 19b44fbb09af69bde803b9b884d58108c248ebe5
ms.sourcegitcommit: b15cf37afc4f57d13ca6636d4227433809562f8b
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/11/2019
ms.locfileid: "54224834"
---
# <a name="manage-custom-state-data-with-azure-cosmos-db-for-net"></a>.NET 用の Azure Cosmos DB を使用してカスタム状態データを管理する

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

この記事では、ボットの状態データを保存および管理するための Azure Cosmos DB ストレージを実装します。 ボットで使用される既定の Connector State Service は、運用環境用ではありません。 GitHub で利用可能な [Azure 拡張機能](https://github.com/Microsoft/BotBuilder-Azure)を使用するか、自分で選択したデータ ストレージ プラットフォームを使用してカスタム状態クライアントを実装する必要があります。 カスタム状態ストレージを使用する理由のいくつかを次に示します。
 - State API のスループットが高い (パフォーマンスをより強力に制御できる)
 - geo 分布に伴う待ち時間が少ない
 - データの格納場所を制御できる
 - 実際の状態データにアクセスできる
 - 32 KB を超えるデータを格納できる
 
## <a name="prerequisites"></a>前提条件
必要なものは次のとおりです。
 - [Microsoft Azure アカウント](https://azure.microsoft.com/en-us/free/)
 - [Visual Studio 2015 またはそれ以降](https://www.visualstudio.com/)
 - [Bot Builder Azure NuGet パッケージ](https://www.nuget.org/packages/Microsoft.Bot.Builder.Azure/)
 - [Autofac Web Api2 NuGet パッケージ](https://www.nuget.org/packages/Autofac.WebApi2/)
 - [Bot Framework Emulator](~/bot-service-debug-emulator.md)
 
## <a name="create-azure-account"></a>Azure アカウントを作成する
Azure アカウントを持っていない場合は、[こちら](https://azure.microsoft.com/en-us/free/)をクリックして、無料アカウントにサインアップしてください。

## <a name="set-up-the-azure-cosmos-db-database"></a>Azure Cosmos DB データベースを設定する
1. Azure Portal にログインしたら、**[新規]** をクリックして新しい *Azure Cosmos DB* データベースを作成します。 
2. **[データベース]** をクリックします。 
3. **[Azure Cosmos DB]** を見つけて、**[作成]** をクリックします。
4. フィールドに入力します。 **[API]** フィールドで、**[SQL (DocumentDB)]** を選択します。 すべてのフィールドに入力した後、画面下部にある **[作成]** ボタンをクリックして、新しいデータベースをデプロイします。 
5. 新しいデータベースがデプロイされたら、その新しいデータベースに移動します。 **[アクセス キー]** をクリックして、キーと接続文字列を検索します。 ボットはこの情報を使用して、状態データを保存するストレージ サービスを呼び出します。

## <a name="install-nuget-packages"></a>NuGet パッケージのインストール
1. Visual Studio で既存の C# ボット プロジェクトを開くか、ボット テンプレートを使用して新しいボット プロジェクトを作成します。 
2. 次の NuGet パッケージをインストールします。
   - Microsoft.Bot.Builder.Azure
   - Autofac.WebApi2

## <a name="add-connection-string"></a>接続文字列の追加 
Web.config ファイルに次のエントリを追加します。
```XML
<add key="DocumentDbUrl" value="Your DocumentDB URI"/>
<add key="DocumentDbKey" value="Your DocumentDB Key"/>
```
値を、Azure Cosmos DB で見つけた URI とプライマリ キーに置き換えます。 Web.config ファイルを保存します。

## <a name="modify-your-bot-code"></a>ボットのコードを変更する
**Azure Cosmos DB** ストレージを使用するには、次のコード行を、ボットの **Global.asax.cs** ファイル内の **Application_Start()** メソッドに追加します。

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

Global.asax.cs ファイルを保存します。 以上で、エミュレーターでボットをテストする準備が整いました。

## <a name="run-your-bot-app"></a>ボット アプリを実行する
Visual Studio でボットを実行すると、追加したコードによって Azure 内にカスタム **botdata** テーブルが作成されます。

## <a name="connect-your-bot-to-the-emulator"></a>ボットをエミュレーターに接続する
この時点では、ボットはローカルで実行されています。 次に、エミュレーターを起動し、ボットをエミュレーターに接続します。
1. アドレス バーに http://localhost:port-number/api/messages と入力します。port-number は、アプリケーションを実行しているブラウザーに示されているポート番号と同じにします。 <strong>[Microsoft アプリ ID]</strong> フィールドと <strong>[Microsoft アプリ パスワード]</strong> フィールドは、この時点では空白のままで構いません。 この情報は、後ほど、[ボットを登録](~/bot-service-quickstart-registration.md)するときに取得します。
2. **[接続]** をクリックします。 
3. エミュレーターでいくつかのメッセージを入力して、ボットをテストします。 

## <a name="view-state-data-on-azure-portal"></a>Azure Portal で状態データを表示する
状態データを表示するには、Azure Portal にサインインし、データベースに移動します。 **[データ エクスプローラー (プレビュー)]** をクリックして、ボットの状態情報が保存されていることを確認します。 

## <a name="next-steps"></a>次の手順
この記事では、Cosmos DB を使用してボットのデータの保存と管理を行いました。 次に、ダイアログを使用して会話フローをモデル化する方法について学びます。

> [!div class="nextstepaction"]
> [会話フローを管理する](bot-builder-dotnet-manage-conversation-flow.md)

## <a name="additional-resources"></a>その他のリソース
上記のコードで使用されている制御の反転コンテナーと依存関係の挿入パターンの基本的な事柄については、[Autofac](http://autofac.readthedocs.io/en/latest/) サイトを参照してください。 

GitHub から[サンプル](https://github.com/Microsoft/BotBuilder-Azure/tree/master/CSharp/Samples/DocumentDb)をダウンロードして、Cosmos DB を使用した状態管理の詳細を知ることもできます。 
