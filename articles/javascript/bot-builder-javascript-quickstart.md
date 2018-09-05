---
title: Bot Builder SDK for JavaScript を使用してボットを作成する | Microsoft Docs
description: Bot Builder SDK for JavaScript を使用してボットをすばやく作成します。
keywords: クイック スタート, Bot Builder SDK, 使用の開始
author: jonathanfingold
ms.author: jonathanfingold
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 07/12/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 21a9aa5b1d108b5d03b108278a81229e16b5bc99
ms.sourcegitcommit: a2f3d87c0f252e876b3e63d75047ad1e7e110b47
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 08/25/2018
ms.locfileid: "42928128"
---
# <a name="create-a-bot-with-the-bot-builder-sdk-v4-preview-for-javascript"></a>Bot Builder SDK v4 (プレビュー) for JavaScript を使用してボットを作成する

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

このクイック スタートでは、Yeoman Bot Builder ジェネレーターと Bot Builder SDK for JavaScript を使用してボットを構築し、Bot Framework Emulator でテストする方法を説明します。 これは、[Microsoft Bot Builder SDK v4](https://github.com/Microsoft/botbuilder-js) に基づいています。

## <a name="prerequisites"></a>前提条件

- [Visual Studio Code](https://www.visualstudio.com/downloads)
- [Node.js](https://nodejs.org/en/)
- [Yeoman](http://yeoman.io/) (ジェネレーターを使用してボットを作成できます)
- [ボット エミュレーター](https://github.com/Microsoft/BotFramework-Emulator)
- [restify](http://restify.com/) および JavaScript での非同期プログラミングに関する知識

> [!NOTE]
> 一部のインストールでは、restify のインストール手順を実行すると、node-gyp に関するエラーが発生します。
> この場合は、`npm install -g windows-build-tools` を実行してみてください。

Bot Builder SDK for JavaScript は、特殊な `@preview` タグを使用して NPM からインストールできる一連の[パッケージ](https://github.com/Microsoft/botbuilder-js/tree/master/libraries)で構成されています。

## <a name="create-a-bot"></a>ボットの作成

管理者特権でのコマンド プロンプトを開き、ディレクトリを作成し、自分のボット用にパッケージを初期化します。

```bash
md myJsBots
cd myJsBots
```

次に、Yeoman と JavaScript のジェネレーターをインストールします。

```bash
npm install -g yo
npm install -g generator-botbuilder@preview
```

次に、ジェネレーターを使用してエコー ボットを作成します。

```bash
yo botbuilder
```

Yeoman により、作成するボットに関する情報の入力が求められます。

- ボットの名前を入力します。
- 説明を入力します。
- ボットの言語を `JavaScript` または `TypeScript` から選択します。
- 使用するテンプレートを選択します。 現時点では、テンプレートは `Echo` しかありませんが、まもなく他のテンプレートも追加される予定です。

Yeoman により、新しいフォルダーにボットが作成されます。

## <a name="explore-code"></a>コードの確認

新しく作成されたボット フォルダーを開くと、`app.js` ファイルがあります。 この `app.js` ファイルには、ボット アプリの実行に必要なすべてのコードが含まれます。 このファイルには、どんな入力もエコー バックして、カウンターをインクリメントするエコー ボットが含まれています。

次のコードでは、会話の状態ミドルウェアでメモリ内ストレージを使用します。 このミドルウェアはストレージに対して状態オブジェクトの読み取りと書き込みを実行します。 カウント変数はボットに送信されるメッセージ数を記録します。 同様の手法を使用して、ターン間の状態を維持することができます。

**app.js**
```javascript
// Packages are installed for you
const { BotFrameworkAdapter, MemoryStorage, ConversationState } = require('botbuilder');
const restify = require('restify');

// Create server
let server = restify.createServer();
server.listen(process.env.port || process.env.PORT || 3978, function () {
    console.log(`${server.name} listening to ${server.url}`);
});

// Create adapter
const adapter = new BotFrameworkAdapter({
    appId: process.env.MICROSOFT_APP_ID,
    appPassword: process.env.MICROSOFT_APP_PASSWORD
});

// Add conversation state middleware
const conversationState = new ConversationState(new MemoryStorage());
adapter.use(conversationState);
```

次のコードでは、受信要求をリッスンし、ユーザーへの応答を送信する前に、受信アクティビティの種類を確認します。

```javascript
// Listen for incoming requests
server.post('/api/messages', (req, res) => {
    // Route received request to adapter for processing
    adapter.processActivity(req, res, (context) => {
        // This bot is only handling Messages
        if (context.activity.type === 'message') {

            // Get the conversation state
            const state = conversationState.get(context);

            // If state.count is undefined set it to 0, otherwise increment it by 1
            const count = state.count === undefined ? state.count = 0 : ++state.count;

            // Echo back to the user whatever they typed.
            return context.sendActivity(`${count}: You said "${context.activity.text}"`);
        } else {
            // Echo back the type of activity the bot detected if not of type message
            return context.sendActivity(`[${context.activity.type} event detected]`);
        }
    });
});
```

## <a name="start-your-bot"></a>ボットの起動

ディレクトリをボット用に作成したディレクトリに変更して、ボットを起動します。

```bash
cd <bot directory>
node app.js
```

## <a name="start-the-emulator-and-connect-your-bot"></a>エミュレーターの起動とボットの接続

この時点では、ボットはローカルで実行されています。 次に、エミュレーターを起動し、エミュレーターのボットに接続します。

1. エミュレーターの [ようこそ] タブにある **[新しいボット構成を作成する]** リンクをクリックします。
1. **ボット名**を入力してから、ボット コードへのディレクトリ パスを入力します。 ボットの構成ファイルはこのパスに保存されます。
1. **[エンドポイント URL]** フィールドに `http://localhost:port-number/api/messages` と入力します。*port-number* は、アプリケーションを実行しているブラウザーに示されているポート番号と同じにします。
1. **[接続]** をクリックしてボットに接続します。 **[Microsoft アプリ ID]** と **[Microsoft アプリ パスワード]** を指定する必要はありません。 現段階では、これらのフィールドは空白のままにしておくことができます。 この情報は、後ほどボットを登録するときに取得します。

ボットに「こんにちは」を送信すると、ボットはメッセージに対して「0: こんにちはを送信しました」と応答します。

## <a name="next-steps"></a>次の手順

次に、[ボットを Azure に配置](../bot-builder-howto-deploy-azure.md)するか、またはボットとそのしくみを説明する概念をご覧ください。

> [!div class="nextstepaction"]
> [ボットの基本的な概念](../v4sdk/bot-builder-basics.md)
