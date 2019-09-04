---
title: ボットのしくみ | Microsoft Docs
description: Bot Framework SDK におけるアクティビティと HTTP のしくみについて説明します。
keywords: 会話フロー, ターン, ボットの会話, ダイアログ, プロンプト, ウォーターフォール, ダイアログ セット
author: johnataylor
ms.author: johtaylo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 05/23/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 0e8a8275a7ede599b3d25576abcd3c1160873db7
ms.sourcegitcommit: eacf1522d648338eebefe2cc5686c1f7866ec6a2
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 08/30/2019
ms.locfileid: "70167201"
---
# <a name="how-bots-work"></a>ボットのしくみ

[!INCLUDE[applies-to](../includes/applies-to.md)]

ボットとは、テキスト、グラフィックス (カード、画像など)、または音声を使用してユーザーが会話形式で対話するアプリです。 ユーザーとボットとの間で行われるすべての対話では、"*アクティビティ*" が発生します。 Facebook、Skype、Slack など、ボットに接続されたユーザーのアプリ ("*チャネル*") とボットとの間でやり取りされる情報は、Azure Bot Service のコンポーネントである Bot Framework Service によって送信されます。 それらによって送信されるアクティビティには、それぞれのチャネルによって付加的な情報が追加される場合があります。 ボットを作成する前に、ユーザーと通信するためにアクティビティ オブジェクトがボットでどのように使用されているかを理解しておくことが大切です。 まずは、単純なエコー ボットを実行するときにやり取りされるアクティビティを見てみましょう。 

![アクティビティの図](media/bot-builder-activity.png)

ここには、"*会話の更新*" と "*メッセージ*" の 2 種類のアクティビティが示されています。

会話の更新は、当事者が会話に参加したときに Bot Framework サービスによって送信されます。 たとえば、Bot Framework Emulator による会話の開始時には、(会話に参加するユーザーとボットに 1 つずつ) 会話の更新アクティビティが 2 つ生じます。 これらの会話の更新アクティビティを見分けるには、"*追加されたメンバー*" プロパティに、ボット以外のメンバーが含まれているかどうかを確認します。 

メッセージ アクティビティは、当事者間の会話情報を伝達します。 エコー ボットの例では、メッセージ アクティビティによって単純なテキストが伝達され、そのテキストがチャネルによってレンダリングされます。 また、メッセージ アクティビティでは、読み上げられるテキスト、推奨されるアクション、または表示されるカードが伝達される場合があります。

この例では、インバウンド メッセージ アクティビティに応じるメッセージ アクティビティがボットによって作成され、送信されました。 しかし、受信されたメッセージ アクティビティに対するボットの応答には、他にもさまざまな形態が考えられます。たとえば、会話の更新アクティビティに対し、何らかのウェルカム テキストをメッセージ アクティビティで送信することによってボットが応答することも珍しくありません。 詳細については、「[新規ユーザーへのメッセージ](bot-builder-welcome-user.md)」を参照してください。

### <a name="http-details"></a>HTTP の詳細

アクティビティは、Bot Framework サービスから HTTP POST 要求を介してボットに送信されます。 ボットは、インバウンド POST 要求に対し、200 HTTP 状態コードで応答します。 ボットからチャネルに送信されるアクティビティは、別の HTTP POST で Bot Framework サービスに送信されます。 それに対する肯定応答が 200 HTTP の状態コードで返されます。

これらの POST 要求とその肯定応答の順序は、プロトコルでは規定されていません。 しかし、一般的な HTTP サービスのフレームワークに合わせるため、これらの要求は入れ子にするのが通常です。つまり、アウトバウンド HTTP 要求は、ボットからインバウンド HTTP 要求のスコープ内で行われます。 上の図には、このパターンが示されています。 2 つの異なる HTTP 接続が連続して存在するため、セキュリティ モデルには、その両方に対する手立てが必要となります。

### <a name="defining-a-turn"></a>ターンの定義

人が会話するときは、通常、代わる代わる順番に話します。 ボットの場合、一般的には、ボットがユーザー入力に反応します。 Bot Framework SDK では、"_ターン_" は、ボットがユーザーから受信するアクティビティと、ボットが即時応答としてユーザーに送り返すアクティビティで構成されています。 ターンは、特定のアクティビティの到着に関連付けられた処理として考えることができます。

"*ターン コンテキスト*" オブジェクトは、アクティビティに関する情報 (送信者と受信者やチャネル、アクティビティの処理に必要なその他のデータなど) を提供します。 また、ターン中に、ボットの各種レイヤーの境界を越えて情報を追加することもできます。

ターン コンテキストは、SDK の中で最も重要なアブストラクションの 1 つです。 インバウンド アクティビティをすべてのミドルウェア コンポーネントおよびアプリケーション ロジックに伝達するだけでなく、ミドルウェア コンポーネントとアプリケーション ロジックからアウトバウンド アクティビティを送信するためのメカニズムも備えています。

## <a name="the-activity-processing-stack"></a>アクティビティの処理スタック

前の図について、メッセージ アクティビティの到着に注目し、さらに詳しく見ていきましょう。

![アクティビティの処理スタック](media/bot-builder-activity-processing-stack.png)

上の例では、ボットがメッセージ アクティビティに対して、同じテキスト メッセージが含まれた別のメッセージ アクティビティで応答しました。 処理は、HTTP POST 要求で始まります。このとき、アクティビティ情報は JSON ペイロードとして伝達されて、Web サーバーに届きます。 C# では、これは通常 ASP.NET プロジェクトになります。また、JavaScript Node.js プロジェクトでは多くの場合、これはいずれかの一般的なフレームワーク (Express、Restify など) です。

"*アダプター*" は SDK の統合コンポーネントで、SDK ランタイムのコアです。 アクティビティは、HTTP POST 本文で JSON として渡されます。 この JSON は逆シリアル化されて Activity オブジェクトが作成され、これが "*アクティビティの処理*" メソッドへの呼び出しによってアダプターに渡されます。 アクティビティを受け取ったアダプターにより、"*ターン コンテキスト*" が作成され、ミドルウェアが呼び出されます。 

上述したように、ターン コンテキストは、アウトバウンド アクティビティを送信するメカニズムをボットに提供します。この送信処理は、多くの場合、インバウンド アクティビティに対する応答として実行されます。 これを実現するために、ターン コンテキストは、"_アクティビティの送信、更新、および削除_" 応答メソッドを備えています。 各応答メソッドは、非同期プロセスで実行されます。 

[!INCLUDE [alert-await-send-activity](../includes/alert-await-send-activity.md)]

## <a name="activity-handlers"></a>アクティビティ ハンドラー

アクティビティを受信したボットは、それを自身の "*アクティビティ ハンドラー*" に渡します。 実際には "*ターン ハンドラー*" と呼ばれる基本ハンドラーが 1 つあり、 すべてのアクティビティがそこを介してルーティングされます。 その後、ターン ハンドラーは、受信したアクティビティの種類に関係なく、個別のアクティビティ ハンドラーを呼び出します。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

たとえば、ボットがメッセージ アクティビティを受信すると、ターン ハンドラーはその受信アクティビティを確認して、`OnMessageActivityAsync` アクティビティ ハンドラーに送信します。 

ご自身のボットを構築するとき、メッセージを処理し、これに応答するためのボット ロジックはこの `OnMessageActivityAsync` ハンドラーに格納されます。 同様に、会話に追加されたメンバーを処理するロジックは、会話にメンバーが追加されると必ず呼び出される `OnMembersAddedAsync` ハンドラーに格納されます。

これらのハンドラーのロジックを実装するには、以下の「[ボット ロジック](#bot-logic)」セクションで示すように、お使いのボットでこれらのメソッドをオーバーライドします。 これらのハンドラーそれぞれに基本実装はありません。このため、必要なロジックをご自身のオーバーライドに追加するだけです。

基本ターン ハンドラーのオーバーライドが必要になる場合もあります。たとえば、ターンの最後に[状態を保存](bot-builder-concept-state.md)するような状況です。 これを行う場合は、最初に必ず `await base.OnTurnAsync(turnContext, cancellationToken);` を呼び出して、`OnTurnAsync` の基本実装がご自身の追加コードの前に実行されていることを確認します。 この実装は特に、`OnMessageActivityAsync` などの残りのアクティビティ ハンドラーを呼び出す役割を果たします。

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

たとえば、ボットがメッセージ アクティビティを受信すると、ターン ハンドラーはその受信アクティビティを確認して、`onMessage` アクティビティ ハンドラーに送信します。

ご自身のボットを構築するとき、メッセージを処理し、これに応答するためのボット ロジックはこの `onMessage` ハンドラーに格納されます。 同様に、会話に追加されたメンバーを処理するロジックは、会話にメンバーが追加されると必ず呼び出される `onMembersAdded` ハンドラーに格納されます。

これらのハンドラーのロジックを実装するには、以下の「[ボット ロジック](#bot-logic)」セクションで示すように、お使いのボットでこれらのメソッドをオーバーライドします。 ハンドラーごとにボット ロジックを定義し、**最後に必ず `next()` を呼び出します**。 `next()` を呼び出すことで、次のハンドラーが確実に実行されます。

基本ターン ハンドラーのオーバーライドが必要になる状況はめったにないため、この操作を行う場合は気を付けてください。 ターンの最後に行う必要がある[状態の保存](bot-builder-concept-state.md)などの操作に対しては、`onDialog` と呼ばれる特別なハンドラーがあります。 `onDialog` ハンドラーは、ハンドラーの残りの部分の実行後、最後に実行され、特定のアクティビティの種類には関連付けられていません。 上記のすべてのハンドラーと同様、必ず `next()` を呼び出して、プロセスの残りの部分を終了してください。

---

## <a name="middleware"></a>ミドルウェア

ミドルウェアは、他のメッセージング ミドルウェアとよく似ています。連続する一連のコンポーネントで構成され、各コンポーネントが順に実行されるため、それぞれがアクティビティで動作できます。 ミドルウェア パイプラインの最終ステージは、アプリケーションがアダプターの *process activity* メソッドに登録したボット クラスのターン ハンドラーへのコールバックです。 ターン ハンドラーは、通常、C# の場合は `OnTurnAsync`、JavaScript の場合は `onTurn` です。

ターン ハンドラーはその引数としてターン コンテキストを受け取り、通常はターン ハンドラー関数の内部で実行されているアプリケーション ロジックによってインバウンド アクティビティの内容を処理して、1 つ以上のアクティビティを応答として生成し、ターン コンテキストの "*アクティビティの送信*" 関数を使用してそれらを送信します。 ターン コンテキストで "*アクティビティの送信*" を呼び出すと、ミドルウェア コンポーネントがアウトバウンド アクティビティで呼び出されます。 ミドルウェア コンポーネントは、ボットのターン ハンドラー関数の前後で実行されます。 実行は本質的に入れ子なっているため、マトリョーシカ人形のようであると言われることもあります。 ミドルウェアの詳細については、[ミドルウェアに関するトピック](~/v4sdk/bot-builder-concept-middleware.md)を参照してください。

## <a name="bot-structure"></a>ボットの構造

次のセクションでは、[**CSharp**](../dotnet/bot-builder-dotnet-sdk-quickstart.md) または [**JavaScript**](../javascript/bot-builder-javascript-quickstart.md) 用に提供されたテンプレートを使用して簡単に作成できる EchoBot の "_主要部_" について説明します。

<!--Need to add section calling out the controller in code, and explaining it further-->

ボットは Web アプリケーションで、各言語のテンプレートが用意されています。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

VSIX テンプレートによって生成されるのは [ASP.NET MVC Core](https://dotnet.microsoft.com/apps/aspnet/mvc) Web アプリです。 [ASP.NET](https://docs.microsoft.com/aspnet/core/fundamentals/index?view=aspnetcore-2.1&tabs=aspnetcore2x) の基礎に関する記事を見ると、**Program.cs** や **Startup.cs** などのファイルにあるのと同様のコードが確認できます。 これらのファイルはすべての Web アプリに必須で、ボット固有のものではありません。

### <a name="appsettingsjson-file"></a>appsettings.json ファイル

**appsettings.json** ファイルでは、アプリ ID、パスワードなど、お使いのボットの構成情報が指定されます。 特定のテクノロジを使用している場合、または運用環境でこのボットを使用している場合は、ご自身の特定のキーまたは URL をこの構成に追加する必要があります。 ただし、このエコー ボットについては、今のところ、ここでは何も行う必要はありません。アプリ ID およびパスワードは、現時点では未定義のままにしておくことができます。

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

<!-- TODO: Update this aka link to point to samples/javascript_nodejs/02.echobot (instead of samples/javascript_nodejs/02.a.echobot) once work-in-progress is merged into master. -->

Yeoman ジェネレーターにより、[restify](http://restify.com/) Web アプリケーションが作成されます。 ドキュメントで restify クイック スタートを確認すると、生成された **index.js** ファイルと似たアプリが表示されます。 ここではテンプレートによって生成されるキー ファイルをいくつかを取り上げます。 一部のファイルに含まれるコードはコピーされませんが、ボットを実行すると表示されます。また、[Node.js echobot](https://aka.ms/js-echobot-sample) サンプルを参照できます。

### <a name="packagejson"></a>package.json

**package.json** では、お使いのボットの依存関係とそれに関連付けられているバージョンが指定されます。 これはすべて、テンプレートとご自身のシステムによって設定されます。

### <a name="env-file"></a>.env ファイル

**.env** ファイルでは、ポート番号、アプリ ID、パスワードなど、お使いのボットの構成情報が指定されます。 特定のテクノロジを使用している場合、または運用環境でこのボットを使用している場合は、ご自身の特定のキーまたは URL をこの構成に追加する必要があります。 ただし、このエコー ボットについては、今のところ、ここでは何も行う必要はありません。アプリ ID およびパスワードは、現時点では未定義のままにしておくことができます。

**.env** 構成ファイルを使用するには、テンプレートに、追加のパッケージが含まれている必要があります。  最初に、`dotenv` パッケージを npm から取得します。

`npm install dotenv`

---

### <a name="bot-logic"></a>ボット ロジック

ボット ロジックは 1 つ以上のチャネルからの受信アクティビティを処理し、応答の送信アクティビティを生成します。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

主なボット ロジックはボット コードで定義され、ここでは `Bots/EchoBot.cs` と呼ばれます。 `EchoBot` は `ActivityHandler` から派生し、これは `IBot` インターフェイスから派生しています。 `ActivityHandler` ではさまざまなハンドラーがさまざまな種類のアクティビティに対して定義されます。たとえば、ここでは `OnMessageActivityAsync` および `OnMembersAddedAsync` の 2 つが定義されます。 これらのメソッドは保護されていますが、`ActivityHandler` から派生しているため、上書きできます。

`ActivityHandler` で定義されているハンドラーを次に示します。

| Event | Handler | 説明 |
| :-- | :-- | :-- |
| 任意のアクティビティの種類を受信した | `OnTurnAsync` | 受信したアクティビティの種類に基づいて、他のハンドラーのいずれかを呼び出します。 |
| メッセージ アクティビティを受信した | `OnMessageActivityAsync` | これをオーバーライドして `message` アクティビティを処理します。 |
| 会話の更新アクティビティを受信した | `OnConversationUpdateActivityAsync` | `conversationUpdate` アクティビティで、ボット以外のメンバーが会話に参加した場合、または会話から退出した場合にハンドラーを呼び出します。 |
| ボットではないメンバーが会話に参加した | `OnMembersAddedAsync` | これをオーバーライドして、会話に参加するメンバーを処理します。 |
| ボットではないメンバーが会話から退出した | `OnMembersRemovedAsync` | これをオーバーライドして、会話から退出メンバーを処理します。 |
| イベント アクティビティを受信した | `OnEventActivityAsync` | `event` アクティビティで、イベントの種類に固有のハンドラーを呼び出します。 |
| Token-response イベント アクティビティを受信した | `OnTokenResponseEventAsync` | これをオーバーライドして、トークン応答イベントを処理します。 |
| Non-token-response イベント アクティビティを受信した | `OnEventAsync` | これをオーバーライドして、その他の種類のイベントを処理します。 |
| メッセージの反応アクティビティを受信した | `OnMessageReactionActivityAsync` | `messageReaction` アクティビティで、メッセージに対して 1 つ以上の反応が追加または削除された場合にハンドラーを呼び出します。 |
| メッセージの反応がメッセージに追加された | `OnReactionsAddedAsync` | これをオーバーライドして、メッセージに追加された反応を処理します。 |
| メッセージの反応がメッセージから削除された | `OnReactionsRemovedAsync` | これをオーバーライドして、メッセージから削除された反応を処理します。 |
| 他のアクティビティの種類を受信した | `OnUnrecognizedActivityTypeAsync` | これをオーバーライドして、他の方法では処理されない任意のアクティビティの種類を処理します。 |

このような各種ハンドラーにインバウンド アクティビティに関する情報を提供する `turnContext` があり、これはインバウンド HTTP 要求に対応しています。 アクティビティの種類もさまざまであるため、各ハンドラーが、そのターン コンテキスト パラメーターで厳密に型指定されたアクティビティを提供します。ほとんどの場合、`OnMessageActivityAsync` は常に処理され通常は最も一般的です。

このフレームワークの以前の 4.x バージョンでは、パブリック メソッド `OnTurnAsync` を実装するオプションもあります。 現在、このメソッドの基本実装はエラー チェックを処理し、各受信アクティビティの種類に応じて、特定のハンドラー (たとえば、このサンプルでは定義する 2 つのハンドラー) をそれぞれ呼び出します。 ほとんどの場合、このメソッドはそのままにして、個別のハンドラーを使用できますが、`OnTurnAsync` のカスタム実装が必要な場合でも、これは引き続き使用できます。

> [!IMPORTANT]
> `OnTurnAsync` メソッドをオーバーライドする場合は、`base.OnTurnAsync` を呼び出して、他のすべての `On<activity>Async` ハンドラーを呼び出すための基本実装を取得するか、ご自身でこれらのハンドラーを呼び出す必要があります。 それ以外の場合、これらのハンドラーは呼び出されず、そのコードは実行されません。

このサンプルでは、新しいユーザーを歓迎するか、`SendActivityAsync` 呼び出しを使用してユーザーが送信したメッセージをエコー バックします。 アウトバウンド アクティビティは、アウトバウンド HTTP POST 要求に対応します。

```cs
public class MyBot : ActivityHandler
{
    protected override async Task OnMessageActivityAsync(ITurnContext<IMessageActivity> turnContext, CancellationToken cancellationToken)
    {
        await turnContext.SendActivityAsync(MessageFactory.Text($"Echo: {turnContext.Activity.Text}"), cancellationToken);
    }

    protected override async Task OnMembersAddedAsync(IList<ChannelAccount> membersAdded, ITurnContext<IConversationUpdateActivity> turnContext, CancellationToken cancellationToken)
    {
        foreach (var member in membersAdded)
        {
            await turnContext.SendActivityAsync(MessageFactory.Text($"welcome {member.Name}"), cancellationToken);
        }
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

主なボット ロジックはボット コードで定義され、ここでは `bots\echoBot.js` と呼ばれます。 `EchoBot` は `ActivityHandler` から派生しています。 `ActivityHandler` ではさまざまなハンドラーがさまざまな種類のアクティビティに対して定義されます。ご自身のボットの動作を変更するには、`onMessage`、`onConversationUpdate` などの追加ロジックをここで指定します。

`ActivityHandler` で定義されているハンドラーを次に示します。

| Event | Handler | 説明 |
| :-- | :-- | :-- |
| 任意のアクティビティの種類を受信した | `onTurn` | いずれかのアクティビティを受信したときに呼び出されます。 |
| メッセージ アクティビティを受信した | `onMessage` | `message` アクティビティを受信したときに呼び出されます。 |
| 会話の更新アクティビティを受信した | `onConversationUpdate` | いずれかの `conversationUpdate` アクティビティを受信したときに呼び出されます。 |
| メンバーが会話に参加した | `onMembersAdded` | いずれかのメンバー (ボットを含む) が会話に参加したときに呼び出されます。 |
| メンバーが会話から退出した | `onMembersRemoved` | いずれかのメンバー (ボットを含む) が会話から退出したときに呼び出されます。 |
| メッセージの反応アクティビティを受信した | `onMessageReaction` | いずれかの `messageReaction` アクティビティを受信したときに呼び出されます。 |
| メッセージの反応がメッセージに追加された | `onReactionsAdded` | 反応がメッセージに追加されたときに呼び出されます。 |
| メッセージの反応がメッセージから削除された | `onReactionsRemoved` | 反応がメッセージから削除されたときに呼び出されます。 |
| イベント アクティビティを受信した | `onEvent` | いずれかの `event` アクティビティを受信したときに呼び出されます。 |
| Token-response イベント アクティビティを受信した | `onTokenResponseEvent` | `tokens/response` イベントを受信したときに呼び出されます。 |
| 他のアクティビティの種類を受信した | `onUnrecognizedActivityType` | 特定の種類のアクティビティのハンドラーが定義されていないときに呼び出されます。 |
| アクティビティ ハンドラーが完了した | `onDialog` | 適用可能なハンドラーが完了した後に呼び出されます。 |

各ハンドラーから `next` 関数パラメーターを呼び出して、処理を続行できるようにします。 `next` が呼び出されない場合は、アクティビティの処理が終了します。

それぞれのターンでは、最初に、ボットがメッセージを受信したかどうかを確認します。 ユーザーからメッセージを受信すると、送信されたメッセージをエコー バックします。

```javascript
const { ActivityHandler } = require('botbuilder');

class MyBot extends ActivityHandler {
    constructor() {
        super();
        // See https://aka.ms/about-bot-activity-message to learn more about the message and other activity types.
        this.onMessage(async (context, next) => {
            await context.sendActivity(`You said '${ context.activity.text }'`);
            // By calling next() you ensure that the next BotHandler is run.
            await next();
        });
        this.onConversationUpdate(async (context, next) => {
            await context.sendActivity('[conversationUpdate event detected]');
            // By calling next() you ensure that the next BotHandler is run.
            await next();
        });
    }
}

module.exports.MyBot = MyBot;
```

---

### <a name="access-the-bot-from-your-app"></a>アプリからボットへのアクセス

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

#### <a name="set-up-services"></a>サービスのセットアップ

`Startup.cs` ファイルの `ConfigureServices` メソッドは、接続済みサービス、`appsettings.json` または Azure Key Vault (存在する場合) のキー、接続状態などを読み込みます。 ここでは、MVC を追加し、サービスで互換性バージョンを設定した後、ボット コントローラーへの依存関係の挿入を使用して、アダプターとボットを使用できるように設定します。

<!-- want to explain the singleton vs transient here?-->

```csharp
// This method gets called by the runtime. Use this method to add services to the container.
public void ConfigureServices(IServiceCollection services)
{
    services.AddMvc().SetCompatibilityVersion(CompatibilityVersion.Version_2_1);

    // Create the credential provider to be used with the Bot Framework Adapter.
    services.AddSingleton<ICredentialProvider, ConfigurationCredentialProvider>();

    // Create the Bot Framework Adapter.
    services.AddSingleton<IBotFrameworkHttpAdapter, BotFrameworkHttpAdapter>();

    // Create the bot as a transient. In this case the ASP Controller is expecting an IBot.
    services.AddTransient<IBot, EchoBot>();
}
```

`Configure` メソッドでご自身のアプリの構成を完了するには、そのアプリによって MVC とその他のいくつかのファイルが使用されることを指定します。 Bot Framework を使用するすべてのボットに、その構成呼び出しが必要ですが、これはご自身のボットをビルドするときに、サンプルまたは VSIX テンプレートで定義されます。 `ConfigureServices` と `Configure` は、アプリの起動時にランタイムによって呼び出されます。

#### <a name="bot-controller"></a>ボット コントローラー

標準の MVC 構造に従っているコントローラーを使用すると、メッセージおよび HTTP POST 要求のルーティングを決定できます。 このボットの場合、前の「[アクティビティの処理スタック](#the-activity-processing-stack)」セクションで説明したように、受信要求をアダプターの *process async activity* メソッドに渡します。 その呼び出しで、ボットと、必要になる可能性がある他の認証情報を指定します。

コントローラーは `ControllerBase` を実装し、`Startup.cs` で設定したアダプターとボットを保持して (ここでは依存関係の挿入を使用可能)、ボットが受信 HTTP POST を受信したときに、必要な情報をそのボットに渡します。

ここでは、クラスはルートおよびコントローラー属性によって実行されます。 これらはフレームワークがメッセージを適切にルーティングし、どのコントローラーを使用するかを認識するうえで役に立ちます。 ルート属性の値を変更すると、エンドポイント、エミュレーター、または他のチャネルによって使用されるその変更は、ご自身のボットにアクセスします。

```cs
// This ASP Controller is created to handle a request. Dependency Injection will provide the Adapter and IBot
// implementation at runtime. Multiple different IBot implementations running at different endpoints can be
// achieved by specifying a more specific type for the bot constructor argument.
[Route("api/messages")]
[ApiController]
public class BotController : ControllerBase
{
    private readonly IBotFrameworkHttpAdapter Adapter;
    private readonly IBot Bot;

    public BotController(IBotFrameworkHttpAdapter adapter, IBot bot)
    {
        Adapter = adapter;
        Bot = bot;
    }

    [HttpPost]
    public async Task PostAsync()
    {
        // Delegate the processing of the HTTP POST to the adapter.
        // The adapter will invoke the bot.
        await Adapter.ProcessAsync(Request, Response, Bot);
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

#### <a name="indexjs"></a>index.js

`index.js` では、ボットの設定のほか、ボット ロジックにアクティビティを転送するホスティング サービスの設定が行われます。

#### <a name="required-libraries"></a>必要なライブラリ

`index.js` ファイルの最上部に、必要な一連のモジュールまたはライブラリが表示されます。 これらのモジュールにより、お使いのアプリケーションに含める必要がある関数セットにアクセスできるようになります。

```javascript
const dotenv = require('dotenv');
const path = require('path');
const restify = require('restify');

// Import required bot services.
// See https://aka.ms/bot-services to learn more about the different parts of a bot.
const { BotFrameworkAdapter } = require('botbuilder');

// This bot's main dialog.
const { MyBot } = require('./bot');

// Import required bot configuration.
const ENV_FILE = path.join(__dirname, '.env');
dotenv.config({ path: ENV_FILE });
```

#### <a name="set-up-services"></a>サービスのセットアップ

次のパートでは、サーバーとアダプターの設定が行われます。このサーバーとアダプターにより、お客様のボットがユーザーとやり取りして、応答を送信できます。 サーバーは、構成ファイルで指定したポートでリッスンするか、エミュレーターとの接続用の _3978_ にフォールバックします。 アダプターはご自身のボットのコンダクタとして機能し、送受信の通信、認証などを指示します。

```javascript
// Create HTTP server
const server = restify.createServer();
server.listen(process.env.port || process.env.PORT || 3978, () => {
    console.log(`\n${ server.name } listening to ${ server.url }`);
    console.log(`\nGet Bot Framework Emulator: https://aka.ms/bot-framework-www-portal-emulator`);
    console.log(`\nTo talk to your bot, open the emulator select "Open Bot"`);
});

// Create adapter.
// See https://aka.ms/about-bot-adapter to learn more about how bots work.
const adapter = new BotFrameworkAdapter({
    appId: process.env.MicrosoftAppId,
    appPassword: process.env.MicrosoftAppPassword
});

// Catch-all for errors.
adapter.onTurnError = async (context, error) => {
    // This check writes out errors to console log .vs. app insights.
    console.error(`\n [onTurnError]: ${ error }`);
    // Send a message to the user
    await context.sendActivity(`Oops. Something went wrong!`);
};

// Create the main dialog.
const myBot = new MyBot();
```

#### <a name="forwarding-requests-to-the-bot-logic"></a>ボット ロジックへの要求のを転送

受信アクティビティは、アダプターの `processActivity` によってボット ロジックに送信されます。
`processActivity` 内の 3 番目のパラメーターは、受信された[アクティビティ](#the-activity-processing-stack)がアダプターによって事前処理され、任意のミドルウェアを介してルーティングされた後、ボット ロジックを実行するために呼び出される関数ハンドラーです。 ターン コンテキスト変数は引数として関数ハンドラーに渡され、受信アクティビティ、送信者と受信者、チャネル、会話などに関する情報を提供する目的で使用できます。アクティビティの処理は、ボットの `run` メソッドにルーティングされます。 `run` は `ActivityHandler` に定義され、エラー チェックをいくつか実行します。その後、受信アクティビティの種類に基づいてボットのイベント ハンドラーを呼び出します。

```javascript
// Listen for incoming requests.
server.post('/api/messages', (req, res) => {
    adapter.processActivity(req, res, async (context) => {
        // Route to main dialog.
        await myBot.run(context);
    });
});
```

---

## <a name="manage-bot-resources"></a>ボット リソースの管理

アプリ ID、パスワード、キーまたはシークレットなどのボット リソースは、適切に管理する必要があります。 これを行う方法の詳細について、「[ボットのリソースを管理](bot-file-basics.md)」を参照してください。

## <a name="additional-resources"></a>その他のリソース

- ボットにおける状態の役割を理解するには、「[状態の管理](bot-builder-concept-state.md)」を参照してください。
