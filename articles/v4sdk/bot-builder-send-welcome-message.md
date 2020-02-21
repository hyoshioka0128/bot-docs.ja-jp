---
title: ユーザーへのウェルカム メッセージの送信 - Bot Service
description: ボットでユーザーへの歓迎メッセージを表示する方法について説明します。
keywords: 概要, 開発, ユーザー エクスペリエンス, ようこそ, パーソナライズされたエクスペリエンス, C#, JS, ウェルカム メッセージ, ボット, あいさつ, グリーティング
author: DanDev33
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 2/7/2020
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 92ec5c927baa93095beaf304fe17024adf81666b
ms.sourcegitcommit: e5bf9a7fa7d82802e40df94267bffbac7db48af7
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/18/2020
ms.locfileid: "77441671"
---
# <a name="send-welcome-message-to-users"></a>ユーザーへのウェルカム メッセージの送信

[!INCLUDE[applies-to](../includes/applies-to.md)]

ボットを作成する主な目的は、有意義な会話によってユーザー エンゲージメントを高めることです。 この目標を達成するための効果的な方法の 1 つは、ユーザーが最初にアクセスした瞬間から、ボットの主な目的と機能 (ボットが作られた理由) をユーザーがわかるようにすることです。 この記事では、ボットのユーザーを歓迎するのに役立つコード サンプルを紹介します。

## <a name="prerequisites"></a>前提条件

- [ボットの基本](bot-builder-basics.md)を理解する。
- **ウェルカム サンプル**のコピー ([C# サンプル](https://aka.ms/welcome-user-mvc)、[JS サンプル](https://aka.ms/bot-welcome-sample-js)、または [Python サンプル](https://aka.ms/bot-welcome-python-sample-code))。 サンプルのコードは、ウェルカム メッセージの送信方法を説明するために使用されます。

## <a name="about-this-sample-code"></a>このサンプル コードについて

このサンプル コードは、新しいユーザーを検出し、そのユーザーがボットに最初に接続されたときにウェルカム メッセージを表示する方法を示しています。 次の図は、このボットのロジック フローです。

### <a name="c"></a>[C#](#tab/csharp)

ボットの 2 つの主要なイベントを次に示します。

- `OnMembersAddedAsync`: 新しいユーザーがボットに接続されると必ず呼び出されます
- `OnMessageActivityAsync`: 新しいユーザー入力を受信すると必ず呼び出されます。

![ウェルカム ロジック フロー](media/welcome-user-flow.png)

接続された新しいユーザーには必ず、`WelcomeMessage`、`InfoMessage`、および `PatternMessage` がボットから提供されます。
新しいユーザー入力を受信すると、WelcomeUserState で `DidBotWelcomeUser` が _true_ に設定されているかどうかが確認されます。 設定されていない場合、最初のウェルカム メッセージがユーザーに返されます。

### <a name="javascript"></a>[JavaScript](#tab/javascript)

ボットの 2 つの主要なイベントを次に示します。

- `onMembersAdded`: 新しいユーザーがボットに接続されると必ず呼び出されます
- `onMessage`: 新しいユーザー入力を受信すると必ず呼び出されます。

![ウェルカム ロジック フロー](media/welcome-user-flow-js.png)

接続された新しいユーザーには必ず、`welcomeMessage`、`infoMessage`、および `patternMessage` がボットから提供されます。
新しいユーザー入力を受信すると、`welcomedUserProperty` で `didBotWelcomeUser` が _true_ に設定されているかどうかが確認されます。 設定されていない場合、最初のウェルカム メッセージがユーザーに返されます。
`DidBotWelcomeUser` が _true_ の場合は、ユーザーの入力が評価されます。 ユーザー入力のコンテンツに基づいて、このボットでは次のいずれかの処理が実行されます。

- ユーザーから受信したグリーティングをエコー バックします。
- ボットに関する追加情報を提供するヒーロー カードを表示します。
- `WelcomeMessage` を再送信します。これには、このボットで想定される入力の説明が示されています。

### <a name="python"></a>[Python](#tab/python)

ボットの 2 つの主要なイベントを次に示します。

- `on_members_added_activity`: 新しいユーザーがボットに接続されると必ず呼び出されます
- `on_message_activity`: 新しいユーザー入力を受信すると必ず呼び出されます。

![ウェルカム ロジック フロー](media/welcome-user-flow-python.png)

新しいユーザーが接続されるたびに、ボットによって*ウェルカム メッセージ*、*情報メッセージ*、*パターン メッセージ*が表示されます。
新しいユーザー入力を受信すると、`welcome_user_state.did_welcome_user` プロパティが *true* に設定されているかどうかが確認されます。 *true* に設定されていない場合、最初のウェルカム ユーザー メッセージがユーザーに返されます。 *true* に設定されている場合、ユーザー入力のコンテンツに基づいて、このボットでは次のいずれかの処理が実行されます。

- ユーザーから受信したグリーティングをエコー バックします。
- ボットに関する追加情報を提供するヒーロー カードを表示します。

---

## <a name="create-user-state"></a>ユーザーの状態を作成する

### <a name="c"></a>[C#](#tab/csharp)

ユーザー状態オブジェクトはスタートアップ時に作成され、依存関係がボット コンストラクターに挿入されます。

**Startup.cs** [!code-csharp[define state](~/../botBuilder-samples/samples/csharp_dotnetcore/03.welcome-user/Startup.cs?range=30-34)]

**Bots\WelcomeUserBot.cs** [!code-csharp[consume state](~/../BotBuilder-Samples/samples/csharp_dotnetcore/03.welcome-user/bots/WelcomeUserBot.cs?range=41-47)]

### <a name="javascript"></a>[JavaScript](#tab/javascript)

起動時に、ユーザーの状態が `index.js` で定義され、ボット コンストラクターによって使用されます。

**index.js** [!code-javascript[define state](~/../BotBuilder-Samples/samples/javascript_nodejs/03.welcome-users/index.js?range=51-55)]

**bots/welcomeBot.js** [!code-javascript[consume state](~/../BotBuilder-Samples/samples/javascript_nodejs/03.welcome-users/bots/welcomeBot.js?range=16-22)]

### <a name="python"></a>[Python](#tab/python)

起動時に、ユーザーの状態が `app.py` で定義され、ボット コンストラクターによって使用されます。

**app.py** [!code-python[define state](~/../botbuilder-samples/samples/python/03.welcome-user/app.py?range=61-66)]

**bots/welcome-user-bot.py** [!code-python[consume state](~/../botbuilder-samples/samples/python/03.welcome-user/bots/welcome_user_bot.py?range=23-29)]

---

## <a name="create-property-accessors"></a>プロパティ アクセサーを作成する

### <a name="c"></a>[C#](#tab/csharp)

ここで、`WelcomeUserState` へのハンドルを返すプロパティ アクセサーを `OnMessageActivityAsync` メソッド内に作成します。
次に、`GetAsync` メソッドを呼び出して、適切に範囲指定されたキーを取得します。 その後、ユーザー入力イテレーションが終了するたびに、`SaveChangesAsync` メソッドを使用して、ユーザー状態データを保存します。

**Bots\WelcomeUserBot.cs** [!code-csharp[Get state](~/../BotBuilder-Samples/samples/csharp_dotnetcore/03.welcome-user/bots/WelcomeUserBot.cs?range=68-71)]
[!code-csharp[Save state](~/../BotBuilder-Samples/samples/csharp_dotnetcore/03.welcome-user/bots/WelcomeUserBot.cs?range= 103-105)]

### <a name="javascript"></a>[JavaScript](#tab/javascript)

ここでプロパティ アクセサーを作成します。このアクセサーにより、userState 内で保持されている welcomedUserProperty のハンドルが提供されます。

**bots/welcomeBot.js** [!code-javascript[Get state](~/../BotBuilder-Samples/samples/javascript_nodejs/03.welcome-users/bots/welcomebot.js?range=24-27)]
[!code-javascript[Save state](~/../BotBuilder-Samples/samples/javascript_nodejs/03.welcome-users/bots/welcomebot.js?range=89-97)]

### <a name="python"></a>[Python](#tab/python)

このサンプルでは、ボット コンストラクターで、会話状態のプロパティ アクセサー `user_state_accessor`が作成されます。

**bots/welcome-user-bot.py** [!code-python[constructor](~/../botbuilder-samples/samples/python/03.welcome-user/bots/welcome_user_bot.py?range=31)]

これは、`on_message_activity` ハンドラーでプロパティ アクセサーを使用し、`on_turn` ハンドラーをオーバーライドしてターン終了前に状態を保存します。

[!code-python[Get state](~/../botbuilder-samples/samples/python/03.welcome-user/bots/welcome_user_bot.py?range=80-83)]
[!code-python[save state](~/../botbuilder-samples/samples/python/03.welcome-user/bots/welcome_user_bot.py?range=39-43)]

---

## <a name="detect-and-greet-newly-connected-users"></a>新しく接続されたユーザーを検出し、あいさつする

### <a name="c"></a>[C#](#tab/csharp)

**WelcomeUserBot** では、`OnMembersAddedAsync()` を使用してアクティビティ更新をチェックし、会話に新しいユーザーが追加されたかどうかを確認して、そのユーザーに最初の 3 つのウェルカム メッセージ `WelcomeMessage`、`InfoMessage`、および `PatternMessage` を送信します。 このインタラクションの完全なコードを以下に示します。

**Bots\WelcomeUserBot.cs** [!code-csharp[Define messages](~/../BotBuilder-Samples/samples/csharp_dotnetcore/03.welcome-user/bots/WelcomeUserBot.cs?range=20-39)]
[!code-csharp[Send messages](~/../BotBuilder-Samples/samples/csharp_dotnetcore/03.welcome-user/bots/WelcomeUserBot.cs?range=55-66)]

### <a name="javascript"></a>[JavaScript](#tab/javascript)

この JavaScript コードは、ユーザーが追加されたときに最初のウェルカム メッセージを送信します。 これは、会話アクティビティをチェックし、会話に新しいメンバーが追加されたことを確認することで実行されます。

**bots/welcomeBot.js** [!code-javascript[onMembersAdded](~/../BotBuilder-Samples/samples/javascript_nodejs/03.welcome-users/bots/welcomebot.js?range=63-86)]

### <a name="python"></a>[Python](#tab/python)

`on_members_added_activity` は、新しいユーザーが追加されたかどうかを確認してから、3 つのウェルカム メッセージ ("*ウェルカム メッセージ*"、"*情報メッセージ*"、および "*パターン メッセージ*") を送信します。

**bots/welcome-user-bot.py** [!code-python[on_members_added_activity](~/../botbuilder-samples/samples/python/03.welcome-user/bots/welcome_user_bot.py?range=45-74)]

---

## <a name="welcome-new-user-and-discard-initial-input"></a>新しいユーザーにウェルカム メッセージを表示し、最初の入力を破棄する

### <a name="c"></a>[C#](#tab/csharp)

ユーザーの入力に有用な情報が実際に含まれている場合を考慮することも重要です。これはチャネルごとに異なる可能性があります。 ユーザーが使用可能なすべてのチャネルで良好なエクスペリエンスを得られるようにするために、状態フラグ _didBotWelcomeUser_ を確認し、これが "false" の場合、最初のユーザー入力は処理しません。 代わりに、ユーザーに最初のウェルカム メッセージを表示します。 ブール値 _welcomedUserProperty_ が "true" に設定され、UserState に格納されると、すべての追加メッセージ アクティビティからのこのユーザーの入力が、コードによって処理されます。

**Bots\WelcomeUserBot.cs** [!code-csharp[DidBotWelcomeUser](~/../BotBuilder-Samples/samples/csharp_dotnetcore/03.welcome-user/bots/WelcomeUserBot.cs?range=68-84)]
[!code-csharp[DidBotWelcomeUser](~/../BotBuilder-Samples/samples/csharp_dotnetcore/03.welcome-user/bots/WelcomeUserBot.cs?range=101-105)]

### <a name="javascript"></a>[JavaScript](#tab/javascript)

ユーザーの入力に有用な情報が実際に含まれている場合を考慮することも重要です。これはチャネルごとに異なる可能性があります。 ユーザーが使用可能なすべてのチャネルで良好なエクスペリエンスを得られるようにするために、didBotWelcomedUser プロパティを確認し、これが存在しない場合は、"false" に設定し、最初のユーザー入力を処理しません。 代わりに、ユーザーに最初のウェルカム メッセージを表示します。 その後、ブール値 didBotWelcomeUser が "true" に設定され、すべての追加メッセージ アクティビティからのユーザー入力が、コードによって処理されます。

**bots/welcomeBot.js** [!code-javascript[DidBotWelcomeUser](~/../BotBuilder-Samples/samples/javascript_nodejs/03.welcome-users/bots/welcomebot.js?range=24-39)]
[!code-javascript[DidBotWelcomeUser](~/../BotBuilder-Samples/samples/javascript_nodejs/03.welcome-users/bots/welcomebot.js?range=57-61)]

### <a name="python"></a>[Python](#tab/python)

ユーザーの入力に有用な情報が実際に含まれている場合を考慮することも重要です。これはチャネルごとに異なる可能性があります。 ユーザーが使用可能なすべてのチャネルで良好なエクスペリエンスを得られるようにするために、`on_message_activity` は `did_welcome_user` プロパティを確認します。 初回は、*false* に設定し、ユーザー入力を処理しません。 代わりに、ユーザーに最初のウェルカム メッセージを表示します。 次に、`did_welcome_user` を *true* に設定し、すべての追加メッセージ アクティビティからのユーザー入力を処理します。

**bots/welcome-user-bot.py** [!code-python[DidBotWelcomeUser](~/../botbuilder-samples/samples/python/03.welcome-user/bots/welcome_user_bot.py?range=85-95)]

---

## <a name="process-additional-input"></a>追加入力を処理する

新しいユーザーにウェルカム メッセージが表示されると、メッセージ ターンごとにユーザー入力情報が評価され、そのユーザー入力のコンテキストに基づいてボットが応答します。 次のコードは、その応答の生成に使用する意思決定ロジックを示しています。

### <a name="c"></a>[C#](#tab/csharp)

"Intro" または "help" の入力により関数 `SendIntroCardAsync` が呼び出され、情報ヒーロー カードがユーザーに示されます。 そのコードについては、この記事の次のセクションで説明します。

**Bots\WelcomeUserBot.cs** [!code-csharp[SwitchOnUtterance](~/../BotBuilder-Samples/samples/csharp_dotnetcore/03.welcome-user/bots/WelcomeUserBot.cs?range=85-100)]

### <a name="javascript"></a>[JavaScript](#tab/javascript)

"Intro" または "help" の入力では CardFactory が使用され、概要アダプティブ カードがユーザーに示されます。 そのコードについては、この記事の次のセクションで説明します。

**bots/welcomeBot.js** [!code-javascript[SwitchOnUtterance](~/../BotBuilder-Samples/samples/javascript_nodejs/03.welcome-users/bots/welcomebot.js?range=40-56)]

### <a name="python"></a>[Python](#tab/python)

ユーザーが *intro* または *help* を入力すると、ボットが `__send_intro_card` を呼び出して、概要アダプティブ カードがユーザーに示されます。

**bots/welcome-user-bot.py** [!code-python[SwitchOnUtterance](~/../botbuilder-samples/samples/python/03.welcome-user/bots/welcome_user_bot.py?range=101-106&highlight=97-106)]

---

## <a name="using-hero-card-greeting"></a>ヒーロー カード グリーティングを使用する

既に説明したように、一部のユーザー入力では、要求の応答として "*ヒーロー カード*" が生成されます。 ヒーロー カード グリーティングの詳細については、[概要カードの送信](./bot-builder-howto-add-media-attachments.md)に関するページをご覧ください。 このボットのヒーロー カード応答の作成に必要なコードを次に示します。

### <a name="c"></a>[C#](#tab/csharp)

**Bots\WelcomeUserBot.cs** [!code-csharp[SendHeroCardGreeting](~/../BotBuilder-Samples/samples/csharp_dotnetcore/03.welcome-user/bots/WelcomeUserBot.cs?range=107-125)]

### <a name="javascript"></a>[JavaScript](#tab/javascript)

**bots/welcomeBot.js** [!code-javascript[SendIntroCard](~/../BotBuilder-Samples/samples/javascript_nodejs/03.welcome-users/bots/welcomebot.js?range=99-124)]

### <a name="python"></a>[Python](#tab/python)

**bots/welcome-user-bot.py** [!code-python[SendIntroCard](~/../botbuilder-samples/samples/python/03.welcome-user/bots/welcome_user_bot.py?range=108-143)]

---

## <a name="test-the-bot"></a>ボットのテスト

最新の [Bot Framework Emulator](https://aka.ms/bot-framework-emulator-readme) をダウンロードしてインストールします

1. ご自身のマシンを使ってローカルでサンプルを実行します。 手順については、README ファイルで [C# サンプル](https://aka.ms/welcome-user-mvc)または [JS サンプル](https://aka.ms/bot-welcome-sample-js)を参照してください。
1. 次に示すように、エミュレーターを使用してボットをテストします。

![ウェルカム ボットのテストのサンプル](media/welcome-user-emulator-1.png)

ヒーロー カード グリーティングをテストします。

![ウェルカム ボット カードのテスト](media/welcome-user-emulator-2.png)

## <a name="additional-resources"></a>その他のリソース

さまざまなメディアの応答の詳細については、「[メッセージにメディアを追加する](./bot-builder-howto-add-media-attachments.md)」を参照してください。

## <a name="next-steps"></a>次のステップ

> [!div class="nextstepaction"]
> [ユーザー入力の収集](bot-builder-prompts.md)
