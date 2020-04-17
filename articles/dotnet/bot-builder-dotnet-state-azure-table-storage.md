---
title: Azure Table Storage を使用したカスタム状態データの管理 (v3 C#) - Bot Service
description: Bot Framework SDK for .NET を使用して Azure Table Storage によって状態データを保存および取得する方法について説明します
author: kamrani
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 5fcce457e5365cd54be77812f0b9fd70382a813b
ms.sourcegitcommit: 9d77f3aff9521d819e88efd0fbd19d469b9919e7
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/16/2020
ms.locfileid: "75797933"
---
# <a name="manage-custom-state-data-with-azure-table-storage-for-net"></a>.NET 用 Azure Table Storage を使用したカスタム状態データの管理

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

この記事では、ボットの状態データを保存および管理するための Azure Table Storage を実装します。 ボットで使用される既定の Connector State Service は、運用環境用ではありません。 GitHub で利用可能な [Azure 拡張機能](https://github.com/Microsoft/BotBuilder-Azure)を使用するか、自分で選択したデータ ストレージ プラットフォームを使用してカスタム状態クライアントを実装する必要があります。 カスタム状態ストレージを使用する理由のいくつかを次に示します。
 - State API のスループットが高い (パフォーマンスをより強力に制御できる)
 - geo 分布に伴う待ち時間が少ない
 - データの格納場所を制御できる
 - 実際の状態データにアクセスできる
 - 32 KB を超えるデータを格納できる

## <a name="prerequisites"></a>前提条件
必要なものは次のとおりです。
 - [Microsoft Azure アカウント](https://azure.microsoft.com/free/)
 - [Visual Studio 2015 またはそれ以降](https://www.visualstudio.com/)
 - [Bot Builder Azure NuGet パッケージ](https://www.nuget.org/packages/Microsoft.Bot.Builder.Azure/)
 - [Autofac Web Api2 NuGet パッケージ](https://www.nuget.org/packages/Autofac.WebApi2/)
 - [Bot Framework Emulator](https://emulator.botframework.com/)
 - [Azure 記憶域エクスプローラー](http://storageexplorer.com/)
 
## <a name="create-azure-account"></a>Azure アカウントを作成する
Azure アカウントを持っていない場合は、[こちら](https://azure.microsoft.com/free/)をクリックして、無料アカウントにサインアップしてください。

## <a name="set-up-the-azure-table-storage-service"></a>Azure Table Storage サービスを設定する
1. Azure portal にログインし、 **[新規]** をクリックして新しい Azure Table Storage サービスを作成します。 
2. Azure Table を実装する**ストレージ アカウント**を検索します。 
3. フィールドに入力し、画面下部にある **[作成]** ボタンをクリックして、新しいストレージ サービスをデプロイします。 新しいストレージ サービスがデプロイされると、利用可能な機能とオプションが表示されます。
4. 左側の **[アクセス キー]** タブを選択し、後で使用するために接続文字列をコピーします。 ボットはこの接続文字列を使用して、状態データを保存するストレージ サービスを呼び出します。

## <a name="install-nuget-packages"></a>NuGet パッケージのインストール
1. Visual Studio で既存の C# ボット プロジェクトを開くか、C# ボット テンプレートを使用して新しいボット プロジェクトを作成します。 
2. 次の NuGet パッケージをインストールします。
   - Microsoft.Bot.Builder.Azure
   - Autofac.WebApi2

## <a name="add-connection-string"></a>接続文字列の追加 
Web.config ファイルに次のエントリを追加します。 
```XML
  <connectionStrings>
    <add name="StorageConnectionString"
    connectionString="YourConnectionString"/>
  </connectionStrings>
```
"YourConnectionString" を、前に保存した Azure Table Storage への接続文字列に置き換えます。 Web.config ファイルを保存します。

## <a name="modify-your-bot-code"></a>ボットのコードを変更する
Global.asax.cs ファイルで、次の `using` ステートメントを追加します。
```cs
using Autofac;
using System.Configuration;
using Microsoft.Bot.Connector;
using Microsoft.Bot.Builder.Azure;
using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Bot.Builder.Dialogs.Internals;
```
`Application_Start()` メソッドで、`TableBotDataStore` クラスのインスタンスを作成します。 `TableBotDataStore` クラスによって、`IBotDataStore<BotData>` インターフェイスが実装されます。 `IBotDataStore` インターフェイスを使用すると、既定の Connector State Service 接続を上書きできます。
 ```cs
 var store = new TableBotDataStore(ConfigurationManager.ConnectionStrings["StorageConnectionString"].ConnectionString);
 ```
サービスは次のように登録します。
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
Global.asax.cs ファイルを保存します。

## <a name="run-your-bot-app"></a>ボット アプリを実行する
Visual Studio でボットを実行すると、追加したコードによって Azure 内にカスタム **botdata** テーブルが作成されます。

## <a name="connect-your-bot-to-the-emulator"></a>ボットをエミュレーターに接続する
この時点では、ボットはローカルで実行されています。 次に、エミュレーターを起動し、エミュレーターのボットに接続します。
1. アドレス バーに http://localhost:port-number/api/messages と入力します。port-number は、アプリケーションを実行しているブラウザーに示されているポート番号と同じにします。 <strong>[Microsoft アプリ ID]</strong> フィールドと <strong>[Microsoft アプリ パスワード]</strong> フィールドは、この時点では空白のままで構いません。 この情報は、後ほど、[ボットを登録](~/bot-service-quickstart-registration.md)するときに取得します。
2. **[Connect]** をクリックします。 
3. エミュレーターでいくつかのメッセージを入力して、ボットをテストします。 

## <a name="view-data-in-azure-table-storage"></a>Azure Table Storage のデータの表示
状態データを表示するには、**Storage Explorer** を開き、Azure portal 資格情報を使用して Azure に接続するか、ストレージ名とストレージ キーを使用してテーブルに直接接続してからテーブル名に移動します。  

## <a name="next-steps"></a>次のステップ
この記事では、ボットのデータを保存および管理するために Azure Table Storage を実装しました。 次に、ダイアログを使用して会話フローをモデル化する方法について学びます。

> [!div class="nextstepaction"]
> [会話フローを管理する](bot-builder-dotnet-manage-conversation-flow.md)


## <a name="additional-resources"></a>その他のリソース

上記のコードで使用される制御の反転コンテナーと依存関係の挿入パターンに慣れていない場合は、[Autofac](http://autofac.readthedocs.io/en/latest/) サイトで詳細をご確認ください。 

完全な [Azure Table Storage](https://github.com/Microsoft/BotBuilder-Azure/tree/master/CSharp/Samples/AzureTable) のサンプルを GitHub からダウンロードすることもできます。
