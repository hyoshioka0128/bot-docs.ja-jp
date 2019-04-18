---
title: ボットのしくみ | Microsoft Docs
description: Bot Framework SDK におけるアクティビティと HTTP のしくみについて説明します。
keywords: 会話フロー, ターン, ボットの会話, ダイアログ, プロンプト, ウォーターフォール, ダイアログ セット
author: johnataylor
ms.author: johtaylo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 1/10/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: a7f6c22f35719eacf66598e79df5fe52ff19dd43
ms.sourcegitcommit: 103aa3316f9ff658cf2b0d341c5e76c3efc581ee
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/12/2019
ms.locfileid: "59540366"
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

"*アダプター*" は SDK の統合コンポーネントで、SDK ランタイムのコアです。 アクティビティは、HTTP POST 本文で JSON として渡されます。 この JSON は逆シリアル化されて Activity オブジェクトが作成され、これが "*アクティビティの処理*" メソッドへの呼び出しによってアダプターに渡されます。 アクティビティを受け取ったアダプターにより、"*ターン コンテキスト*" が作成され、ミドルウェアが呼び出されます。 "*ターン コンテキスト*" という名前は、アクティビティの到着に関連したあらゆる処理の説明に、"ターン" という言葉が使用されていることに由来します。 ターン コンテキストは、SDK の中で最も重要なアブストラクションの 1 つで、インバウンド アクティビティをすべてのミドルウェア コンポーネントおよびアプリケーション ロジックに伝達するだけでなく、ミドルウェア コンポーネントとアプリケーション ロジックからアウトバウンド アクティビティを送信するためのメカニズムも備えています。 ターン コンテキストは、アクティビティに応答するための、"_アクティビティの送信、更新、および削除_" 応答メソッドを備えています。 各応答メソッドは、非同期プロセスで実行されます。 

[!INCLUDE [alert-await-send-activity](../includes/alert-await-send-activity.md)]


## <a name="middleware"></a>ミドルウェア
ミドルウェアは、他のメッセージング ミドルウェアとよく似ています。連続する一連のコンポーネントで構成され、各コンポーネントが順に実行されるため、それぞれがアクティビティで動作できます。 ミドルウェア パイプラインの最終ステージは、アプリケーションがアダプターに登録したボット クラスでターン ハンドラーを呼び出すコールバック (C# の場合は `OnTurnAsync`、JS の場合は `onTurn`) 関数です。 ターン ハンドラーはその引数としてターン コンテキストを受け取り、通常はターン ハンドラー関数の内部で実行されているアプリケーション ロジックによってインバウンド アクティビティの内容を処理して、1 つ以上のアクティビティを応答として生成し、ターン コンテキストの "*アクティビティの送信*" 関数を使用してそれらを送信します。 ターン コンテキストで "*アクティビティの送信*" を呼び出すと、ミドルウェア コンポーネントがアウトバウンド アクティビティで呼び出されます。 ミドルウェア コンポーネントは、ボットのターン ハンドラー関数の前後で実行されます。 実行は本質的に入れ子なっているため、マトリョーシカ人形のようであると言われることもあります。 ミドルウェアの詳細については、[ミドルウェアに関するトピック](~/v4sdk/bot-builder-concept-middleware.md)を参照してください。

## <a name="bot-structure"></a>ボットの構造
以降のセクションでは、ボットの主な要素について説明します。

### <a name="prerequisites"></a>前提条件
- **EchoBotWithCounter** サンプルのコピー (**[C#](https://aka.ms/EchoBotWithStateCSharp) または [JS](https://aka.ms/EchoBotWithStateJS)**)。 ここでは関連するコードのみを示していますが、サンプルを参照することで完全なソース コードを確認できます。

# <a name="ctabcs"></a>[C#](#tab/cs)

ボットは、一種の [ASP.NET Core](https://docs.microsoft.com/aspnet/core/?view=aspnetcore-2.1) Web アプリケーションです。 [ASP.NET](https://docs.microsoft.com/aspnet/core/fundamentals/index?view=aspnetcore-2.1&tabs=aspnetcore2x) の基礎に関する記事を見ると、**Program.cs** や **Startup.cs** などのファイルにあるのと同様のコードが確認できます。 これらのファイルはすべての Web アプリに必須で、ボット固有のものではありません。 

### <a name="bot-logic"></a>ボット ロジック

ボットのメイン ロジックは、`IBot` インターフェイスから派生した `EchoWithCounterBot` クラスに定義されます。 `IBot` には、`OnTurnAsync` というメソッドが 1 つだけ定義されています。 お客様のアプリケーションでは、このメソッドを実装する必要があります。 受信アクティビティに関する情報は、`OnTurnAsync` の turnContext から得られます。 受信アクティビティは、インバウンド HTTP 要求に相当します。 アクティビティにはさまざまな種類があるため、まず、お客様のボットがメッセージを受信したかどうかを確認します。 それがメッセージである場合は、ターン コンテキストから会話の状態を取得して、ターン カウンターをインクリメントし、新しいターン カウンターの値を会話の状態として保持します。 さらに、SendActivityAsync 呼び出しを使用してユーザーにメッセージを返します。 送信アクティビティは、アウトバウンド HTTP 要求に相当します。

```cs
public async Task OnTurnAsync(ITurnContext turnContext, CancellationToken cancellationToken = default(CancellationToken))
{
    if (turnContext.Activity.Type == ActivityTypes.Message)
    {
        // Get the conversation state from the turn context.
        var oldState = await _accessors.CounterState.GetAsync(turnContext, () => new CounterState());

        // Bump the turn count for this conversation.
        var newState = new CounterState { TurnCount = oldState.TurnCount + 1 };

        // Set the property using the accessor.
        await _accessors.CounterState.SetAsync(turnContext, newState);

        // Save the new turn count into the conversation state.
        await _accessors.ConversationState.SaveChangesAsync(turnContext);

        // Echo back to the user whatever they typed.
        var responseMessage = $"Turn {newState.TurnCount}: You sent '{turnContext.Activity.Text}'\n";
        await turnContext.SendActivityAsync(responseMessage);
    }
    else
    {
        await turnContext.SendActivityAsync($"{turnContext.Activity.Type} event detected");
    }
}
```

### <a name="set-up-services"></a>サービスのセットアップ

startup.cs ファイルの `ConfigureServices` メソッドでは、接続済みサービスを [.bot](bot-builder-basics.md#the-bot-file) ファイルから読み込み、会話のターン中に発生したエラーをキャッチしてログに記録します。さらに、資格情報プロバイダーを設定し、会話データをメモリに格納するための会話の状態オブジェクトを作成します。

```csharp
services.AddBot<EchoWithCounterBot>(options =>
{
    // Creates a logger for the application to use.
    ILogger logger = _loggerFactory.CreateLogger<EchoWithCounterBot>();

    var secretKey = Configuration.GetSection("botFileSecret")?.Value;
    var botFilePath = Configuration.GetSection("botFilePath")?.Value;

    // Loads .bot configuration file and adds a singleton that your Bot can access through dependency injection.
    BotConfiguration botConfig = null;
    try
    {
        botConfig = BotConfiguration.Load(botFilePath ?? @".\BotConfiguration.bot", secretKey);
    }
    catch
    {
        //...
    }

    services.AddSingleton(sp => botConfig);

    // Retrieve current endpoint.
    var environment = _isProduction ? "production" : "development";
    var service = botConfig.Services.Where(s => s.Type == "endpoint" && s.Name == environment).FirstOrDefault();
    if (!(service is EndpointService endpointService))
    {
        throw new InvalidOperationException($"The .bot file does not contain an endpoint with name '{environment}'.");
    }

    options.CredentialProvider = new SimpleCredentialProvider(endpointService.AppId, endpointService.AppPassword);

    // Catches any errors that occur during a conversation turn and logs them.
    options.OnTurnError = async (context, exception) =>
    {
        logger.LogError($"Exception caught : {exception}");
        await context.SendActivityAsync("Sorry, it looks like something went wrong.");
    };

    // The Memory Storage used here is for local bot debugging only. When the bot
    // is restarted, everything stored in memory will be gone.
    IStorage dataStore = new MemoryStorage();

    // ...

    // Create Conversation State object.
    // The Conversation State object is where we persist anything at the conversation-scope.
    var conversationState = new ConversationState(dataStore);

    options.State.Add(conversationState);
});
```

`ConfigureServices` メソッドでは、`EchoBotAccessors` の作成と登録も行います。このアクセサーは、**EchoBotStateAccessors.cs** ファイルで定義され、ASP.NET Core の依存関係挿入フレームワークを使用して、パブリック `EchoWithCounterBot` コンストラクターに渡されます。

```csharp
// Accessors created here are passed into the IBot-derived class on every turn.
services.AddSingleton<EchoBotAccessors>(sp =>
{
    var options = sp.GetRequiredService<IOptions<BotFrameworkOptions>>().Value;
    // ...
    var conversationState = options.State.OfType<ConversationState>().FirstOrDefault();
    // ...

    // Create the custom state accessor.
    // State accessors enable other components to read and write individual properties of state.
    var accessors = new EchoBotAccessors(conversationState)
    {
        CounterState = conversationState.CreateProperty<CounterState>(EchoBotAccessors.CounterStateName),
    };

    return accessors;
});
```

`Configure` メソッドでは、ご自身のアプリで Bot Framework とその他のいくつかのファイルが使用されることを指定します。これにより、アプリの構成が完了します。 Bot Framework を使用するすべてのボットで、その構成の呼び出しが必要です。 `ConfigureServices` と `Configure` は、アプリの起動時にランタイムによって呼び出されます。

### <a name="manage-state"></a>状態の管理

このファイルには、現在の状態を維持するために、お客様のボットによって使用されるシンプルなクラスが含まれます。 これには、お客様のカウンターのインクリメントに使用する `int` のみが含まれます。

```cs
public class CounterState
{
    public int TurnCount { get; set; } = 0;
}
```

### <a name="accessor-class"></a>アクセサー クラス

`EchoBotAccessors` クラスは、`Startup` クラスにシングルトンとして作成され、IBot 派生クラスに渡されます。 例では、 `public class EchoWithCounterBot : IBot`が使用されます。 このアクセサーは、ボットで会話データを保持する際に使用されます。 `EchoBotAccessors` のコンストラクターには、Startup.cs ファイルに作成された会話オブジェクトが渡されます。

```cs
public class EchoBotAccessors
{
    public EchoBotAccessors(ConversationState conversationState)
    {
        ConversationState = conversationState ?? throw new ArgumentNullException(nameof(conversationState));
    }

    public static string CounterStateName { get; } = $"{nameof(EchoBotAccessors)}.CounterState";

    public IStatePropertyAccessor<CounterState> CounterState { get; set; }

    public ConversationState ConversationState { get; }
}
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

Yeoman ジェネレーターにより、[restify](http://restify.com/) Web アプリケーションが作成されます。 ドキュメントで restify クイック スタートを確認すると、生成された **index.js** ファイルと似たアプリが表示されます。 このセクションでは、主に、**package.json**、**.env**、 **index.js**、**bot.js**、および **echobot-with-counter.bot** ファイルについて説明します。 一部のファイルに含まれるコードはここにコピーされませんが、ボットを実行すると表示されます。また、[Node.js echobot-with-counter](https://aka.ms/js-echobot-with-counter) サンプルを参照することができます。

### <a name="packagejson"></a>package.json

**package.json** では、お使いのボットの依存関係とそれに関連付けられているバージョンが指定されます。 これはすべて、テンプレートとご自身のシステムによって設定されます。

### <a name="env-file"></a>.env ファイル

**.env** ファイルでは、ポート番号、アプリ ID、パスワードなど、お使いのボットの構成情報が指定されます。 特定のテクノロジを使用している場合、または運用環境でこのボットを使用している場合は、ご自身の特定のキーまたは URL をこの構成に追加する必要があります。 ただし、このエコー ボットについては、今のところ、ここでは何も行う必要はありません。アプリ ID およびパスワードは、現時点では未定義のままにしておくことができます。

**.env** 構成ファイルを使用するには、テンプレートに、追加のパッケージが含まれている必要があります。  最初に、`dotenv` パッケージを npm から取得します。

`npm install dotenv`

### <a name="indexjs"></a>index.js

`index.js` では、ボットの設定のほか、ボット ロジックにアクティビティを転送するホスティング サービスの設定が行われます。

#### <a name="required-libraries"></a>必要なライブラリ

`index.js` ファイルの最上部に、必要な一連のモジュールまたはライブラリが表示されます。 これらのモジュールにより、お使いのアプリケーションに含める必要がある関数セットにアクセスできるようになります。

```javascript
// Import required packages
const path = require('path');
const restify = require('restify');

// Import required bot services. See https://aka.ms/bot-services to learn more about the different parts of a bot.
const { BotFrameworkAdapter, ConversationState, MemoryStorage } = require('botbuilder');
// Import required bot configuration.
const { BotConfiguration } = require('botframework-config');

const { EchoBot } = require('./bot');

// Read botFilePath and botFileSecret from .env file
// Note: Ensure you have a .env file and include botFilePath and botFileSecret.
const ENV_FILE = path.join(__dirname, '.env');
const env = require('dotenv').config({ path: ENV_FILE });
```

#### <a name="bot-configuration"></a>ボットの構成

次のパートでは、ボットの構成ファイルから情報が読み込まれます。

```javascript
// Get the .bot file path
// See https://aka.ms/about-bot-file to learn more about .bot file its use and bot configuration.
const BOT_FILE = path.join(__dirname, (process.env.botFilePath || ''));
let botConfig;
try {
    // Read bot configuration from .bot file.
    botConfig = BotConfiguration.loadSync(BOT_FILE, process.env.botFileSecret);
} catch (err) {
    console.error(`\nError reading bot file. Please ensure you have valid botFilePath and botFileSecret set for your environment.`);
    console.error(`\n - The botFileSecret is available under appsettings for your Azure Bot Service bot.`);
    console.error(`\n - If you are running this bot locally, consider adding a .env file with botFilePath and botFileSecret.`);
    console.error(`\n - See https://aka.ms/about-bot-file to learn more about .bot file its use and bot configuration.\n\n`);
    process.exit();
}

// For local development configuration as defined in .bot file
const DEV_ENVIRONMENT = 'development';

// Define name of the endpoint configuration section from the .bot file
const BOT_CONFIGURATION = (process.env.NODE_ENV || DEV_ENVIRONMENT);

// Get bot endpoint configuration by service name
// Bot configuration as defined in .bot file
const endpointConfig = botConfig.findServiceByNameOrId(BOT_CONFIGURATION);
```

#### <a name="bot-adapter-http-server-and-bot-state"></a>ボット アダプター、HTTP サーバー、ボット状態

次のパートでは、サーバーとアダプターの設定が行われます。このサーバーとアダプターにより、お客様のボットがユーザーとやり取りして、応答を送信できます。 サーバーは、**BotConfiguration.bot** 構成ファイルで指定されたポートでリッスンするか、エミュレーターとの接続用の _3978_ にフォールバックします。 アダプターはご自身のボットのコンダクタとして機能し、送受信の通信、認証などを指示します。

`MemoryStorage` をストレージ プロバイダーとして使用する状態オブジェクトも作成します。 この状態は `ConversationState` として定義されています。これは、単にご自身の会話の状態が保持されていることを意味します。 `ConversationState` には、ご自身が興味のある情報が保存されます。この場合は、単なるメモリ内のターン カウンターです。

```javascript
// Create bot adapter.
// See https://aka.ms/about-bot-adapter to learn more about bot adapter.
const adapter = new BotFrameworkAdapter({
    appId: endpointConfig.appId || process.env.microsoftAppID,
    appPassword: endpointConfig.appPassword || process.env.microsoftAppPassword
});

// Catch-all for any unhandled errors in your bot.
adapter.onTurnError = async (context, error) => {
    // This check writes out errors to console log .vs. app insights.
    console.error(`\n [onTurnError]: ${ error }`);
    // Send a message to the user
    context.sendActivity(`Oops. Something went wrong!`);
    // Clear out state
    await conversationState.clear(context);
    // Save state changes.
    await conversationState.saveChanges(context);
};

// Define a state store for your bot. See https://aka.ms/about-bot-state to learn more about using MemoryStorage.
// A bot requires a state store to persist the dialog and user state between messages.
let conversationState;

// For local development, in-memory storage is used.
// CAUTION: The Memory Storage used here is for local bot debugging only. When the bot
// is restarted, anything stored in memory will be gone.
const memoryStorage = new MemoryStorage();
conversationState = new ConversationState(memoryStorage);

// Create the main dialog.
const bot = new EchoBot(conversationState);

// Create HTTP server
let server = restify.createServer();
server.listen(process.env.port || process.env.PORT || 3978, function() {
    console.log(`\n${ server.name } listening to ${ server.url }`);
    console.log(`\nGet Bot Framework Emulator: https://aka.ms/botframework-emulator`);
    console.log(`\nTo talk to your bot, open echoBot-with-counter.bot file in the Emulator`);
});
```

#### <a name="bot-logic"></a>ボット ロジック

受信アクティビティは、アダプターの `processActivity` によってボット ロジックに送信されます。
`processActivity` 内の 3 番目のパラメーターは、受信された[アクティビティ](#the-activity-processing-stack)がアダプターによって事前処理され、任意のミドルウェアを介してルーティングされた後、ボット ロジックを実行するために呼び出される関数ハンドラーです。 ターン コンテキスト変数は引数として関数ハンドラーに渡され、受信アクティビティ、送信者と受信者、チャネル、会話などに関する情報を提供する目的で使用できます。アクティビティの処理は、EchoBot の `onTurn` にルーティングされます。

```javascript
// Listen for incoming requests.
server.post('/api/messages', (req, res) => {
    // Route received request to adapter for processing
    adapter.processActivity(req, res, (context) => {
        // Route to main dialog.
        await bot.onTurn(context);
    });
});
```

### <a name="echobot"></a>EchoBot

アクティビティの処理はすべて、このクラスの `onTurn` ハンドラーにルーティングされます。 このクラスの作成時に、状態オブジェクトが渡されます。 この状態オブジェクトを使用することにより、このボットのターン カウンターを保持するための `this.countProperty` アクセサーがコンストラクターによって作成されます。

それぞれのターンでは、最初に、ボットがメッセージを受信したかどうかを確認します。 ボットがメッセージを受信しなかった場合は、受信したアクティビティの種類がエコー バックされます。 次に、ボットの会話に関する情報が保持される状態変数を作成します。 カウント変数が `undefined` である場合、変数は 1 に設定されるか (ボットの初回起動時)、新しいメッセージごとにインクリメントされます。 カウントは、送信メッセージと共にユーザーにエコー バックします。 最後に、カウントを設定し、変更を状態に保存します。

```javascript
const { ActivityTypes } = require('botbuilder');

// Turn counter property
const TURN_COUNTER_PROPERTY = 'turnCounterProperty';

class EchoBot {

    constructor(conversationState) {
        // Creates a new state accessor property.
        // See https://aka.ms/about-bot-state-accessors to learn more about the bot state and state accessors
        this.countProperty = conversationState.createProperty(TURN_COUNTER_PROPERTY);
        this.conversationState = conversationState;
    }

    async onTurn(turnContext) {
        // Handle message activity type. User's responses via text or speech or card interactions flow back to the bot as Message activity.
        // Message activities may contain text, speech, interactive cards, and binary or unknown attachments.
        // see https://aka.ms/about-bot-activity-message to learn more about the message and other activity types
        if (turnContext.activity.type === ActivityTypes.Message) {
            // read from state.
            let count = await this.countProperty.get(turnContext);
            count = count === undefined ? 1 : ++count;
            await turnContext.sendActivity(`${ count }: You said "${ turnContext.activity.text }"`);
            // increment and set turn counter.
            await this.countProperty.set(turnContext, count);
        } else {
            // Generic handler for all other activity types.
            await turnContext.sendActivity(`[${ turnContext.activity.type } event detected]`);
        }
        // Save state changes
        await this.conversationState.saveChanges(turnContext);
    }
}

exports.EchoBot = EchoBot;
```

---

## <a name="the-bot-file"></a>ボット ファイル

**.bot** ファイルには、エンドポイント、アプリ ID、パスワードなどの情報のほか、ボットによって使用されるサービスの参照が格納されます。 このファイルは、テンプレートからボットの作成を開始すると、自動的に作成されますが、エミュレーターまたはその他のツールを使用して、独自のものを作成することもできます。 [エミュレーター](../bot-service-debug-emulator.md)を使ってボットをテストするときに、使用する .bot ファイルを指定できます。

```json
{
    "name": "echobot-with-counter",
    "services": [
        {
            "type": "endpoint",
            "name": "development",
            "endpoint": "http://localhost:3978/api/messages",
            "appId": "",
            "appPassword": "",
            "id": "1"
        }
    ],
    "padlock": "",
    "version": "2.0"
}
```

## <a name="additional-resources"></a>その他のリソース

- ボットにおける状態の役割を理解するには、「[状態の管理](bot-builder-concept-state.md)」を参照してください。
- リソースの管理におけるボット ファイルの役割について理解するには、「[.bot ファイルでリソースを管理する](bot-file-basics.md)」を参照してください。
- ボットを初めて作成するには、[Azure Bot Service の使用](../bot-service-quickstart.md)、[C# の使用](../dotnet/bot-builder-dotnet-sdk-quickstart.md)、または [JavaScript の使用](../javascript/bot-builder-javascript-quickstart.md)のいずれかのクイック スタートを参照してください。
