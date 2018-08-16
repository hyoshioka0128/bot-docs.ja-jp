---
title: Azure Table Storage を使用したカスタム状態データの管理 | Microsoft Docs
description: Bot Builder SDK for Node.js で Azure Table Storage を使用して状態データを保存および取得する方法について取り上げます。
author: DucVo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: c77b07801b8eb0168ac3e09d7b271ddfb17a04ac
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/18/2018
ms.locfileid: "39302281"
---
# <a name="manage-custom-state-data-with-azure-table-storage-for-nodejs"></a>Node.js 用 Azure Table Storage を使用したカスタム状態データの管理

この記事では、ボットの状態データを保存および管理するための Azure Table Storage を実装します。 ボットで使用される既定の Connector State Service は、運用環境用ではありません。 GitHub で利用可能な [Azure 拡張機能](https://www.npmjs.com/package/botbuilder-azure)を使用するか、自分で選択したデータ ストレージ プラットフォームを使用してカスタム状態クライアントを実装する必要があります。 カスタム状態ストレージを使用する理由のいくつかを次に示します。

- State API のスループットが高い (パフォーマンスをより強力に制御できる)
- geo 分布に伴う待ち時間が少ない
- データの格納場所を制御できる (例: 米国西部と米国東部)
- 実際の状態データにアクセスできる
- 状態データのデータベースが他のボットと共有されない
- 32 KB を超えるデータを格納できる

## <a name="prerequisites"></a>前提条件

- [Node.js](https://nodejs.org/en/)。
- [Bot Framework Emulator](~/bot-service-debug-emulator.md)。
- Node.js ボットが必要。 ない場合は、[ボットを作成](bot-builder-nodejs-quickstart.md)します。 
- [Storage Explorer](http://storageexplorer.com/)。

## <a name="create-azure-account"></a>Azure アカウントを作成する
Azure アカウントを持っていない場合は、[こちら](https://azure.microsoft.com/en-us/free/)をクリックして、無料アカウントにサインアップしてください。

## <a name="set-up-the-azure-table-storage-service"></a>Azure Table Storage サービスを設定する
1. Azure portal にログインし、**[新規]** をクリックして新しい Azure Table Storage サービスを作成します。 
2. Azure Table を実装する**ストレージ アカウント**を検索します。 **[作成]** をクリックして、ストレージ アカウントの作成を開始します。 
3. フィールドに入力し、画面下部にある **[作成]** ボタンをクリックして、新しいストレージ サービスをデプロイします。 
4. 新しいストレージ サービスがデプロイされたら、作成したストレージ アカウントに移動します。 作成したストレージ アカウントは、**[ストレージ アカウント]** ブレードに表示されます。
4. **[アクセス キー]** を選択し、後で使用するためにそのキーをコピーします。 ボットは、**ストレージ アカウント名**と**キー**を使用して、状態データを保存するストレージ サービスを呼び出します。

## <a name="install-botbuilder-azure-module"></a>botbuilder-azure モジュールをインストールする

コマンド プロンプトから `botbuilder-azure` モジュールをインストールするには、ボットのディレクトリに移動し、次の npm コマンドを実行します。

```nodejs
npm install --save botbuilder-azure
```

## <a name="modify-your-bot-code"></a>ボットのコードを変更する

**Azure Table** ストレージを使用するには、次のコード行をボットの **app.js** ファイルに追加します。

1. 新しくインストールされたモジュールを必須にします。

   ```javascript
   var azure = require('botbuilder-azure'); 
   ```

2. Azure に接続するための接続設定を構成します。
   ```javascript
   // Table storage
   var tableName = "Table-Name"; // You define
   var storageName = "Table-Storage-Name"; // Obtain from Azure Portal
   var storageKey = "Azure-Table-Key"; // Obtain from Azure Portal
   ```
   `storageName` 値と `storageKay` 値は、Azure Table の **[アクセス キー]** メニューで確認できます。 Azure Table に `tableName` が存在しない場合は、作成されます。

3. `botbuilder-azure` モジュールを使用して、Azure Table に接続するための新しいオブジェクトを 2 つ作成します。 最初に、接続構成設定を渡して、`AzureTableClient` のインスタンスを作成します。 次に、`AzureTableClient` オブジェクトを渡して `AzureBotStorage` のインスタンスを作成します。 例: 

   ```javascript
   var azureTableClient = new azure.AzureTableClient(tableName, storageName, storageKey);

   var tableStorage = new azure.AzureBotStorage({gzipData: false}, azureTableClient);
   ```

4. インメモリ ストレージではなくカスタム データベースを使用し、セッション情報をデータベースに追加するよう指定します。 例: 

   ```javascript
   var bot = new builder.UniversalBot(connector, function (session) {
        // ... Bot code ...

        // capture session user information
        session.userData = {"userId": session.message.user.id, "jobTitle": "Senior Developer"};

        // capture conversation information  
        session.conversationData[timestamp.toISOString().replace(/:/g,"-")] = session.message.text;

        // save data
        session.save();
   })
   .set('storage', tableStorage);
   ```
以上で、エミュレーターでボットをテストする準備が整いました。

## <a name="run-your-bot-app"></a>ボット アプリを実行する

コマンド プロンプトでボットのディレクトリに移動し、次のコマンドを使用してボットを実行します。

```nodejs
node app.js
```

## <a name="connect-your-bot-to-the-emulator"></a>ボットをエミュレーターに接続する

この時点では、ボットはローカルで実行されています。 エミュレーターを起動し、エミュレーターからボットに接続します。

1. エミュレーターのアドレス バーに「<strong>http://localhost:port-number/api/messages</strong>」と入力します。port-number は、アプリケーションを実行しているブラウザーに示されているポート番号と同じにします。 <strong>[Microsoft App ID]\(Microsoft アプリ ID\)</strong> フィールドと <strong>[Microsoft App Password]\(Microsoft アプリ パスワード\)</strong> フィールドは、この時点では空白のままでかまいません。 この情報は、後で[ボットを登録](~/bot-service-quickstart-registration.md)するときに取得します。
2. **[接続]** をクリックします。
3. ボットにメッセージを送信して、テストします。 通常どおりにボットと対話してください。 対話が終わったら、**Storage Explorer** に移動し、保存された状態データを表示します。

## <a name="view-data-in-storage-explorer"></a>Storage Explorer でデータを表示する

状態データを表示するには、**Storage Explorer** を開き、Azure portal 資格情報を使用して Azure に接続するか、`storageName` と `storageKey` を使用して Table に直接接続してから `tableName` に移動します。 

![ボット データ テーブルの行を含む Storage Explorer のスクリーンショット](~/media/bot-builder-nodejs-state-azure-table-storage/bot-builder-nodejs-state-azure-table-storage-query.png)

**[データ]** 列の会話の 1 つのレコードは次のようになります。

```JSON
{
    "2018-05-15T18-23-48.780Z": "I'm the second user",
    "2018-05-15T18-23-55.120Z": "Do you know what time it is?",
    "2018-05-15T18-24-12.214Z": "I'm looking for information about the new process."
}
```

## <a name="next-step"></a>次のステップ

ボットの状態データを完全に制御できるようになったので、次は状態データを使用して会話フローの管理を強化する方法を調べます。

> [!div class="nextstepaction"]
> [会話フローの管理](bot-builder-nodejs-dialog-manage-conversation-flow.md)