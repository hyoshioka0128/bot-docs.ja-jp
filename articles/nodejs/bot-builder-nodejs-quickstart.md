---
title: Bot Builder SDK for Node.js を使用したボットの作成 | Microsoft Docs
description: ボットの強力な構築フレームワークである Bot Builder SDK for Node.js を使用してボットを作成します。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: get-started-article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 7259e1336965733b860a6129d51b3c2eb10362c7
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/18/2018
ms.locfileid: "39302145"
---
# <a name="create-a-bot-with-the-bot-builder-sdk-for-nodejs"></a>Bot Builder SDK for Node.js を使用したボットの作成
> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-quickstart.md)
> - [Node.js](../nodejs/bot-builder-nodejs-quickstart.md)
> - [Bot Service](../bot-service-quickstart.md)
> - [REST](../rest-api/bot-framework-rest-connector-quickstart.md)

Bot Builder SDK for Node.js は、ボット開発用のフレームワークです。 使いやすく、Express や Restify などのフレームワークをモデル化して、JavaScript 開発者がボットを記述するためのより身近な方法を提供しています。

このチュートリアルでは、Bot Builder SDK for Node.js を使用してボットを作成します。 ボットは、コンソール ウィンドウおよび Bot Framework Emulator でテストできます。

## <a name="prerequisites"></a>前提条件
前提条件となる次のタスクを完了して、使用を開始します。

1. [Node.js](https://nodejs.org) をインストールします。
2. ボットのフォルダーを作成します。
3. コマンド プロンプトまたはターミナルから、先ほど作成したフォルダーに移動します。
4. 次の **npm** コマンドを実行します。

```nodejs
npm init
```

画面上のプロンプトに従ってボットに関する情報を入力すると、npm によって提供された情報を含む **package.json** ファイルが作成されます。 

## <a name="install-the-sdk"></a>SDK のインストール
次に、以下の **npm** コマンドを実行して、Bot Builder SDK for Node.js をインストールします。

```nodejs
npm install --save botbuilder
```

SDK をインストールしたので、初めてのボットを記述する準備ができました。

初めてのボットでは、任意のユーザー入力を単純に再現するボットを作成します。 ボットを作成するには、次の手順を実行します。

1. ボット用に事前に作成したフォルダーに、**app.js** という名前の新しいファイルを作成します。
2. テキスト エディターまたは選択した IDE で、**app.js** を開きます。 次のコードをファイルに追加します。 

   [!code-javascript[consolebot code sample Node.js](../includes/code/node-getstarted.js#consolebot)]

3. ファイルを保存します。 これで、ボットを実行してテストする準備ができました。

### <a name="start-your-bot"></a>ボットの起動

コンソール ウィンドウでボットのディレクトリに移動し、ボットを起動します。

```nodejs
node app.js
```

これで、ボットがローカルで実行されました。 コンソール ウィンドウでいくつかのメッセージを入力して、ボットを試行します。
*"You said:"* というテキストが先頭に付与されているメッセージを再現して、送信した各メッセージにボットが応答していることが確認できます。

## <a name="install-restify"></a>Restify をインストールする

コンソール ボットは優れたテキスト ベース クライアントですが、いずれかの Bot Framework チャネルを使用する (または、エミュレーターでボットを実行する) ためには、ボットが API エンドポイント上で実行される必要があります。 次の **npm** コマンドを実行して <a href="http://restify.com/" target="_blank">Restify</a> をインストールします。

```nodejs
npm install --save restify
```

Restify を配置したので、ボットに変更を加える準備ができました。

## <a name="edit-your-bot"></a>ボットを編集する

**app.js** ファイルに変更を加える必要があります。 

1. `restify` モジュールを要求する行を追加します。
2. `ConsoleConnector` を `ChatConnector` に変更します。
3. Microsoft アプリ ID とアプリ パスワードを含めます。
4. コネクターで API エンドポイントをリッスンします。

   [!code-javascript[echobot code sample Node.js](../includes/code/node-getstarted.js#echobot)]

5. ファイルを保存します。 これで、エミュレーターでボットを実行してテストする準備ができました。

> [!NOTE] 
> Bot Framework Emulator でボットを実行するために、**Microsoft アプリ ID** や **Microsoft アプリ パスワード**は必要ありません。

## <a name="test-your-bot"></a>ボットをテストする
次に、[Bot Framework Emulator](../bot-service-debug-emulator.md) を使用してボットをテストし、動作を確認します。 このエミュレーターは、ローカルホストで、またはトンネルを介したリモート実行で、ボットをテストおよびデバッグするデスクトップ アプリケーションです。

まず、エミュレーターを[ダウンロード](https://emulator.botframework.com)してインストールする必要があります。 ダウンロードが完了したら、実行可能ファイルを起動し、インストール プロセスを完了します。

### <a name="start-your-bot"></a>ボットの起動

エミュレーターをインストールした後、コンソール ウィンドウでボットのディレクトリに移動し、ボットを起動します。

```nodejs
node app.js
```
   
これで、ボットがローカルで実行されました。

### <a name="start-the-emulator-and-connect-your-bot"></a>エミュレーターの起動とボットの接続
ボットを起動した後、次の手順に従って、エミュレーターを起動してボットを接続します。

1. エミュレーター ウィンドウの **[新しいボット構成を作成する]** リンクをクリックします。 

2. アドレス バーに `http://localhost:port-number/api/messages` と入力します。*port-number* は、アプリケーションを実行しているブラウザーに示されているポート番号と同じにします。

3. [保存および接続] をクリックします。 [Microsoft アプリ ID] と [Microsoft アプリ パスワード] を指定する必要はありません。 これらのフィールドは、この時点では空白のままで構いません。 この情報は、後ほどボットを登録するときに取得します。

### <a name="try-out-your-bot"></a>ボットを試行する

ボットがローカルで実行され、エミュレーターに接続されたので、エミュレーターにいくつかのメッセージを入力してボットを試行します。
*"You said:"* というテキストが先頭に付与されているメッセージを再現して、送信した各メッセージにボットが応答していることが確認できます。

Bot Builder SDK for Node.js を使用して、初めてのボットを正常に作成できました。

## <a name="next-steps"></a>次の手順

> [!div class="nextstepaction"]
> [Bot Builder SDK for Node.js](bot-builder-nodejs-overview.md)
