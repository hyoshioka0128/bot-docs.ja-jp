---
title: Azure Cosmos DB を使用してカスタム状態データを管理する | Microsoft Docs
description: Bot Framework SDK for Node.js で Azure Cosmos DB を使用して状態データを保存および取得する方法を説明します。
author: DucVo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 67e86455aefd000c8a6956a71adcfdb821266196
ms.sourcegitcommit: a295a90eac461f8b96770dd902ba44919acf33fc
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 06/26/2019
ms.locfileid: "67404333"
---
# <a name="manage-custom-state-data-with-azure-cosmos-db-for-nodejs"></a>Node.js 用の Azure Cosmos DB によるカスタム状態データの管理

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

この記事では、ボットの状態データを保存および管理するための Cosmos DB ストレージを実装します。 ボットで使用される既定の Connector State Service は、運用環境用ではありません。 GitHub で利用可能な [Azure 拡張機能](https://www.npmjs.com/package/botbuilder-azure)を使用するか、自分で選択したデータ ストレージ プラットフォームを使用してカスタム状態クライアントを実装する必要があります。 カスタム状態ストレージを使用する理由のいくつかを次に示します。

- State API のスループットが高い (パフォーマンスをより強力に制御できる)
- geo 分布に伴う待ち時間が少ない
- データの格納場所を制御できる (例:米国西部と米国東部)
- 実際の状態データにアクセスできる
- 状態データのデータベースが他のボットと共有されない
- 32 KB を超えるデータを格納できる

## <a name="prerequisites"></a>前提条件

- [Node.js](https://nodejs.org/en/)。
- [Bot Framework Emulator](~/bot-service-debug-emulator.md)
- Node.js ボットが必要です。 ない場合は、[ボットを作成](bot-builder-nodejs-quickstart.md)します。 

## <a name="create-azure-account"></a>Azure アカウントの作成
Azure アカウントを持っていない場合は、[こちら](https://azure.microsoft.com/free/)をクリックして、無料アカウントにサインアップしてください。

## <a name="set-up-the-azure-cosmos-db-database"></a>Azure Cosmos DB データベースを設定する
1. Azure Portal にログインしたら、 **[新規]** をクリックして新しい *Azure Cosmos DB* データベースを作成します。 
2. **[データベース]** をクリックします。 
3. **[Azure Cosmos DB]** を見つけて、 **[作成]** をクリックします。
4. フィールドに入力します。 **[API]** フィールドで、 **[SQL (DocumentDB)]** を選択します。 すべてのフィールドに入力した後、画面下部にある **[作成]** ボタンをクリックして、新しいデータベースをデプロイします。 
5. 新しいデータベースがデプロイされたら、その新しいデータベースに移動します。 **[アクセス キー]** をクリックして、キーと接続文字列を検索します。 ボットはこの情報を使用して、状態データを保存するストレージ サービスを呼び出します。

## <a name="install-botbuilder-azure-module"></a>botbuilder-azure モジュールのインストール

コマンド プロンプトから `botbuilder-azure` モジュールをインストールするには、ボットのディレクトリに移動し、次の npm コマンドを実行します。

```nodejs
npm install --save botbuilder-azure
```

## <a name="modify-your-bot-code"></a>ボット コードの変更

**Azure Cosmos DB** データベースを使用するには、次のコード行をボットの **app.js** ファイルに追加します。

1. 新しくインストールされたモジュールを必須にします。

   ```javascript
   var azure = require('botbuilder-azure'); 
   ```

2. Azure に接続するための接続文字列設定を構成します。
   ```javascript
   var documentDbOptions = {
       host: 'Your-Azure-DocumentDB-URI', 
       masterKey: 'Your-Azure-DocumentDB-Key', 
       database: 'botdocs',   
       collection: 'botdata'
   };
   ```
   `host` 値と `masterKey` 値は、データベースの **[キー]** メニューで確認できます。 `database` エントリと `collection` エントリが Azure データベースにない場合、自動的に作成されます。

3. `botbuilder-azure` モジュールを使用して、Azure データベースに接続するための新しいオブジェクトを 2 つ作成します。 最初に、(上記の `documentDbOptions` として定義されている) 接続構成設定を渡して、`DocumentDBClient` のインスタンスを作成します。 次に、`DocumentDBClient` オブジェクトを渡して `AzureBotStorage` のインスタンスを作成します。 例:
   ```javascript
   var docDbClient = new azure.DocumentDbClient(documentDbOptions);

   var cosmosStorage = new azure.AzureBotStorage({ gzipData: false }, docDbClient);
   ```

4. インメモリ ストレージではなく、カスタム データベースを使用するよう指定します。 例:

   ```javascript
   var bot = new builder.UniversalBot(connector, function (session) {
        // ... Bot code ...
   })
   .set('storage', cosmosStorage);
   ```

以上で、エミュレーターでボットをテストする準備が整いました。

## <a name="run-your-bot-app"></a>ボット アプリを実行する

コマンド プロンプトでボットのディレクトリに移動し、次のコマンドを使用してボットを実行します。

```nodejs
node app.js
```

## <a name="connect-your-bot-to-the-emulator"></a>ボットをエミュレーターに接続する

この時点では、ボットはローカルで実行されています。 エミュレーターを起動し、エミュレーターからボットに接続します。

1. エミュレーターのアドレス バーに「<strong>http://localhost:port-number/api/messages</strong>」と入力します。port-number は、アプリケーションを実行しているブラウザーに示されているポート番号と同じにします。 <strong>[Microsoft App ID]\(Microsoft アプリ ID\)</strong> フィールドと <strong>[Microsoft App Password]\(Microsoft アプリ パスワード\)</strong> フィールドは、この時点では空白のままでかまいません。 この情報は、後ほど、[ボットを登録](~/bot-service-quickstart-registration.md)するときに取得します。
2. **[接続]** をクリックします。
3. ボットにメッセージを送信して、テストします。 通常どおりにボットと対話してください。 対話が終わったら、**Storage Explorer** に移動し、保存された状態データを表示します。

## <a name="view-state-data-on-azure-portal"></a>Azure Portal での状態データの表示

状態データを表示するには、Azure Portal にサインインし、データベースに移動します。 **[データ エクスプローラー (プレビュー)]** をクリックし、ボットの状態の情報が保存されていることを確認します。

## <a name="next-step"></a>次のステップ

ボットの状態データを完全に制御できるようになったので、次は状態データを使用して会話フローの管理を強化する方法を調べます。

> [!div class="nextstepaction"]
> [会話フローの管理](bot-builder-nodejs-dialog-manage-conversation-flow.md)
