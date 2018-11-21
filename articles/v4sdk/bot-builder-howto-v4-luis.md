---
title: ボットに自然言語の理解を追加する | Microsoft Docs
description: 自然言語理解のための LUIS を Bot Builder SDK と共に使用する方法について説明します。
keywords: Language Understanding, LUIS, 意図, 認識エンジン, エンティティ, ミドルウェア
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: cognitive-services
ms.date: 11/08/18
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: eab8e2f9d437748d0bb0fefd31c03c8fb350c6b1
ms.sourcegitcommit: 8b7bdbcbb01054f6aeb80d4a65b29177b30e1c20
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/14/2018
ms.locfileid: "51645702"
---
# <a name="add-natural-language-understanding-to-your-bot"></a>ボットに自然言語の理解を追加する

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

ユーザーの真意を会話と文脈から理解する能力は難しいタスクかもしれませんが、より自然な会話の印象をボットに与えることができます。 そうした能力は LUIS と呼ばれる Language Understanding によって実現され、ボットがユーザーのメッセージの意図を認識できるように、ユーザーがより自然な言葉を使用できるように、また、会話フローがより適切に管理されるようになります。 LUIS の詳細な背景情報が必要な場合は、ボットの [Language Understanding](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/what-is-luis) に関する記事をご覧ください。

## <a name="prerequisites"></a>前提条件
このトピックでは、LUIS を使用して数種類の意図を認識する単純なボットの設定手順について説明します。 この記事のコードは、LUIS による NLP の [C#](https://aka.ms/cs-luis-sample) サンプルおよび [JavaScript](https://aka.ms/js-luis-sample) サンプルをベースにしています。

## <a name="create-a-luis-app-in-the-luis-portal"></a>LUIS ポータルでの LUIS アプリの作成

まず、[luis.ai](https://www.luis.ai) でアカウントにサインアップし、[こちら](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/luis-how-to-start-new-app)に記載された手順に従い、LUIS ポータルで LUIS アプリを作成します。 この記事で使用されたサンプル LUIS アプリについて、お客様独自のバージョンを作成する場合は、LUIS ポータル内でこの `LUIS.Reminders.json` ファイル ([C#](https://github.com/Microsoft/BotBuilder-Samples/blob/v4/samples/csharp_dotnetcore/12.nlp-with-luis/CognitiveModels/LUIS-Reminders.json) | [JS](https://github.com/Microsoft/BotBuilder-Samples/blob/master/samples/javascript_nodejs/12.nlp-with-luis/cognitiveModels/reminders.json)) を[インポート](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/create-new-app#import-new-app)して LUIS アプリをビルドし、[トレーニング](https://docs.microsoft.com/en-us/azure/cognitive-services/LUIS/luis-how-to-train)して[発行](https://docs.microsoft.com/en-us/azure/cognitive-services/LUIS/publishapp)してください。

### <a name="obtain-values-to-connect-to-your-luis-app"></a>LUIS アプリに接続するための値を取得する

LUIS アプリには、その発行後、ボットからアクセスできるようになります。 ボット内から LUIS アプリにアクセスするためには、いくつかの値を記録する必要があります。 その情報は、LUIS ポータルまたは CLI ツールを使用して取得できます。

#### <a name="using-luis-portal"></a>LUIS ポータルを使用する
- 発行済みの LUIS アプリを [luis.ai](https://www.luis.ai) から選択します。
- 発行済み LUIS アプリを開いた状態で **[管理]** タブを選択します。
- 左側の **[アプリケーション情報]** タブで、_[アプリケーション ID]_ に表示される値を <YOUR_APP_ID> として記録します。
- 左側の **[Keys and Endpoints]\(キーとエンドポイント\)** タブを選択し、_[オーサリング キー]_ に表示される値を <YOUR_AUTHORING_KEY> として記録します。 <YOUR_SUBSCRIPTION_KEY> は、<YOUR_AUTHORING_KEY> と同じであることに注意してください。 下へスクロールしてページの最後まで移動し、_[リージョン]_ に表示される値を <YOUR_REGION> として記録します。さらに、_[エンドポイント]_ に表示される値を <YOUR_ENDPOINT> として記録します。

#### <a name="using-cli-tools"></a>CLI ツールを使用する

[luis](https://aka.ms/botbuilder-tools-luis) と [msbot](https://aka.ms/botbuilder-tools-msbot-readme) の BotBuilder CLI ツールを使用して、LUIS アプリのメタデータを取得し、**.bot** ファイルに追加することができます。

1. ターミナルまたはコマンド プロンプトを開いて、ボット プロジェクトのルート ディレクトリに移動します。
2. `luis` と `msbot` のツールがインストールされていることを確認します。

    ```shell
    npm install luis msbot
    ```

3. `luis init` を実行して LUIS のリソース ファイル (**.luisrc**) を作成します。 確認を求められたら、お客様の LUIS オーサリング キーとリージョンを指定してください。 この時点でアプリ ID を入力する必要はありません。
4. 次のコマンドを実行してメタデータをダウンロードし、ボットの構成ファイルに追加します。
    構成ファイルを暗号化した場合、ファイルを更新するためには秘密鍵を指定する必要があります。

    ```shell
    luis get application --appId <your-app-id> --msbot | msbot connect luis --stdin [--secret <YOUR-SECRET>]
    ```

## <a name="configure-your-bot-to-use-your-luis-app"></a>LUIS アプリを使用するためのボットの構成

ボットを初期化すると、まず LUIS アプリへの参照が追加されます。 それをボット ロジック内で呼び出すことができます。

### <a name="prerequisite"></a>前提条件

コーディングを始める前に、LUIS アプリに必要なパッケージがあることを確認します。

# <a name="ctabcs"></a>[C#](#tab/cs)

次の [NuGet パッケージ](https://docs.microsoft.com/en-us/nuget/tools/package-manager-ui)をボットに追加します。

* `Microsoft.Bot.Builder.AI.Luis`

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

LUIS の機能は、`botbuilder-ai` パッケージに含まれています。 このパッケージは、npm を使用してプロジェクトに追加できます。

```shell
npm install --save botbuilder-ai
```

---

# <a name="ctabcs"></a>[C#](#tab/cs)

[こちら](https://aka.ms/cs-luis-sample)から NLP LUIS サンプル コードをダウンロードして開きます。 必要に応じてコードを変更します。 

まず、LUIS アプリにアクセスするために必要な情報 (アプリケーション ID、オーサリング キー、サブスクリプション キー、エンドポイント、リージョンなど) を `BotConfiguration.bot` ファイルに追加します。 これらは、発行済みの LUIS アプリから先ほど保存した値です。

```csharp
{
  "name": "LuisBot",
  "services": [
    {
      "type": "endpoint",
      "name": "development",
      "endpoint": "http://localhost:3978/api/messages",
      "appId": "",
      "appPassword": "",
      "id": "1"
    },
    {
      "type": "luis",
      "name": "LuisBot",
      "AppId": "<YOUR_APP_ID>",
      "SubscriptionKey": "<YOUR_SUBSCRIPTION_KEY>",
      "AuthoringKey": "<YOUR_AUTHORING_KEY>",
      "GetEndpoint": "<YOUR_ENDPOINT>",
      "Region": "<YOUR_REGION>"
    }
  ],
  "padlock": "",
  "version": "2.0"
}
```

次に、BotService クラス `BotServices.cs` の新しいインスタンスを初期化します。これにより、前述の情報が `.bot` ファイルから取得されます。 次のコードを `BotServices.cs` ファイルに追加します。

```csharp
public class BotServices
{
    /// Initializes a new instance of the BotServices class
    public BotServices(BotConfiguration botConfiguration)
    {
        foreach (var service in botConfiguration.Services)
        {
            switch (service.Type)
            {
                case ServiceTypes.Luis:
                {
                    var luis = (LuisService)service;
                    if (luis == null)
                    {
                        throw new InvalidOperationException("The LUIS service is not configured correctly in your '.bot' file.");
                    }

                    var app = new LuisApplication(luis.AppId, luis.AuthoringKey, luis.GetEndpoint());
                    var recognizer = new LuisRecognizer(app);
                    this.LuisServices.Add(luis.Name, recognizer);
                    break;
                    }
                }
            }
        }

    /// Gets the set of LUIS Services used.
    /// LuisServices is represented as a dictionary.  
    public Dictionary<string, LuisRecognizer> LuisServices { get; } = new Dictionary<string, LuisRecognizer>();
}
```

次に、LUIS アプリをシングルトンとして `Startup.cs` ファイルに登録します。`ConfigureServices` 内に次のコードを追加してください。

```csharp
public void ConfigureServices(IServiceCollection services)
{
    var secretKey = Configuration.GetSection("botFileSecret")?.Value;
    var botFilePath = Configuration.GetSection("botFilePath")?.Value;

    // Loads .bot configuration file and adds a singleton that your Bot can access through dependency injection.
    var botConfig = BotConfiguration.Load(botFilePath ?? @".\BotConfiguration.bot", secretKey);
    services.AddSingleton(sp => botConfig ?? throw new InvalidOperationException($"The .bot config file could not be loaded. ({botConfig})"));

    // Initialize Bot Connected Services clients.
    var connectedServices = new BotServices(botConfig);
    services.AddSingleton(sp => connectedServices);
    services.AddSingleton(sp => botConfig);

    services.AddBot<LuisBot>(options =>
    {
        // Retrieve current endpoint.
        var environment = _isProduction ? "production" : "development";
        var service = botConfig.Services.Where(s => s.Type == "endpoint" && s.Name == environment).FirstOrDefault();
        if (!(service is EndpointService endpointService))
        {
            throw new InvalidOperationException($"The .bot file does not contain an endpoint with name '{environment}'.");
        }

        options.CredentialProvider = new SimpleCredentialProvider(endpointService.AppId, endpointService.AppPassword);

        // Creates a logger for the application to use.
        ILogger logger = _loggerFactory.CreateLogger<LuisBot>();

        // Catches any errors that occur during a conversation turn and logs them.
        options.OnTurnError = async (context, exception) =>
        {
            logger.LogError($"Exception caught : {exception}");
            await context.SendActivityAsync("Sorry, it looks like something went wrong.");
        };
        /// ...
    });
}
```

次に、ボットにこの LUIS インスタンスを付与する必要があります。 `LuisBot.cs` を開いて、ファイルの先頭に次のコードを追加します。

```csharp
public class LuisBot : IBot
{
    public static readonly string LuisKey = "LuisBot";
    private const string WelcomeText = "This bot will introduce you to natural language processing with LUIS. Type an utterance to get started";

    /// Services configured from the ".bot" file.
    private readonly BotServices _services;

    /// Initializes a new instance of the LuisBot class.
    public LuisBot(BotServices services)
    {
        _services = services ?? throw new System.ArgumentNullException(nameof(services));
        if (!_services.LuisServices.ContainsKey(LuisKey))
        {
            throw new System.ArgumentException($"Invalid configuration. Please check your '.bot' file for a LUIS service named '{LuisKey}'.");
        }
    }
    /// ...
}
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

このサンプルでは、スタートアップ コードが **index.js** ファイルにあります。ボット ロジックのコードは **bot.js** ファイルに、また、追加の構成情報は **nlp-with-luis.bot** ファイルに含まれています。

LUIS アプリの作成手順と **.bot** ファイルの更新手順を終えると、**nlp-with-luis.bot** ファイルには、LUIS アプリのサービス エントリが含まれています。

```json
{
    "name": "YOUR_LUIS_APP_NAME",
    "description": "",
    "services": [
        {
            "type": "endpoint",
            "name": "development",
            "endpoint": "http://localhost:3978/api/messages",
            "appId": "",
            "appPassword": "",
            "id": "35"
        },
        {
            "type": "luis",
            "name": "YOUR_LUIS_APP_NAME",
            "appId": "<YOUR_APP_ID>",
            "version": "0.1",
            "authoringKey": "<YOUR_AUTHORING_KEY>",
            "subscriptionKey": "<YOUR_SUBSCRIPTION_KEY>>",
            "region": "<YOUR_REGION>",
            "id": "83"
        }
    ],
    "padlock": "",
    "version": "2.0"
}
```

**bot.js** ファイルで構成情報を読み取って、LUIS サービスを生成し、ボットを初期化します。
`LUIS_CONFIGURATION` の値は、お客様の構成ファイルにある LUIS アプリの名前に更新してください。

```javascript
// Language Understanding (LUIS) service name as defined in the .bot file.YOUR_LUIS_APP_NAME is "LuisBot" in the C# code.
const LUIS_CONFIGURATION = '<YOUR_LUIS_APP_NAME>';

// Get endpoint and LUIS configurations by service name.
const endpointConfig = botConfig.findServiceByNameOrId(BOT_CONFIGURATION);
const luisConfig = botConfig.findServiceByNameOrId(LUIS_CONFIGURATION);

// Map the contents to the required format for `LuisRecognizer`.
const luisApplication = {
    applicationId: luisConfig.appId,
    endpointKey: luisConfig.subscriptionKey || luisConfig.authoringKey,
    azureRegion: luisConfig.region
};

// Create configuration for LuisRecognizer's runtime behavior.
const luisPredictionOptions = {
    includeAllIntents: true,
    log: true,
    staging: false
};

// Create adapter...

// Create the LuisBot.
let bot;
try {
    bot = new LuisBot(luisApplication, luisPredictionOptions);
} catch (err) {
    console.error(`[botInitializationError]: ${ err }`);
    process.exit();
}
```

次に、HTTP サーバーを作成し、受信要求をリッスンして、ボット ロジックに対する呼び出しを生成します。

```javascript
// Create HTTP server.
let server = restify.createServer();
server.listen(process.env.port || process.env.PORT || 3978, function() {
    console.log(`\n${ server.name } listening to ${ server.url }.`);
    console.log(`\nGet Bot Framework Emulator: https://aka.ms/botframework-emulator.`);
    console.log(`\nTo talk to your bot, open nlp-with-luis.bot file in the emulator.`);
});

// Listen for incoming requests.
server.post('/api/messages', (req, res) => {
    adapter.processActivity(req, res, async(turnContext) => {
        await bot.onTurn(turnContext);
    });
});
```

---

お客様のボットで使用するための LUIS の構成が完了しました。 次は、LUIS から意図を取得する方法を見てみましょう。

## <a name="get-the-intent-by-calling-luis"></a>LUIS を呼び出して意図を取得する

ボットは LUIS 認識エンジンを呼び出すことによって、LUIS から結果を取得します。

# <a name="ctabcs"></a>[C#](#tab/cs)

LUIS アプリが検出した意図に基づいた返信をボットで単純に送信するには、`LuisRecognizer` を呼び出して `RecognizerResult` を取得します。 これは、LUIS の意図を取得する必要があるときはいつでもコード内で行うことができます。

```cs
public async Task OnTurnAsync(ITurnContext turnContext, CancellationToken cancellationToken = default(CancellationToken))

{
    if (turnContext.Activity.Type == ActivityTypes.Message)
    {
        // Check LUIS model
        var recognizerResult = await _services.LuisServices[LuisKey].RecognizeAsync(turnContext, cancellationToken);
        var topIntent = recognizerResult?.GetTopScoringIntent();
        if (topIntent != null && topIntent.HasValue && topIntent.Value.intent != "None")
        {
            await turnContext.SendActivityAsync($"==>LUIS Top Scoring Intent: {topIntent.Value.intent}, Score: {topIntent.Value.score}\n");
        }
        else
        {
            var msg = @"No LUIS intents were found.
                        This sample is about identifying two user intents:
                        'Calendar.Add'
                        'Calendar.Find'
                        Try typing 'Add Event' or 'Show me tomorrow'.";
            await turnContext.SendActivityAsync(msg);
        }
        }
        else if (turnContext.Activity.Type == ActivityTypes.ConversationUpdate)
        {
            // Send a welcome message to the user and tell them what actions they may perform to use this bot
            await SendWelcomeMessageAsync(turnContext, cancellationToken);
        }
        else
        {
            await turnContext.SendActivityAsync($"{turnContext.Activity.Type} event detected", cancellationToken: cancellationToken);
        }
}
```

発話で認識されたすべての意図は、意図名からスコアへのマップとして返され、`recognizerResult.Intents` からアクセスできます。 結果セットの最高スコアの意図を簡単に見つけるための、静的な `recognizerResult?.GetTopScoringIntent()` メソッドが提供されます。

認識されたすべてのエンティティは、エンティティ名から値へのマップとして返され、`recognizerResults.entities` を使用してアクセスされます。 LuisRecognizer の作成時に `verbose=true` の設定を渡すことによって、追加のエンティティ メタデータを返すことができます。 追加されたメタデータには `recognizerResults.entities.$instance` を使用してアクセスできます。

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

**bot.js** ファイルで、ユーザーの入力を LUIS 認識エンジンの `recognize` メソッドに渡して、LUIS アプリから回答を取得します。

```javascript
const { ActivityTypes } = require('botbuilder');
const { LuisRecognizer } = require('botbuilder-ai');

/**
 * A simple bot that responds to utterances with answers from the Language Understanding (LUIS) service.
 * If an answer is not found for an utterance, the bot responds with help.
 */
class LuisBot {
    /**
     * The LuisBot constructor requires one argument (`application`) which is used to create an instance of `LuisRecognizer`.
     * @param {LuisApplication} luisApplication The basic configuration needed to call LUIS. In this sample the configuration is retrieved from the .bot file.
     * @param {LuisPredictionOptions} luisPredictionOptions (Optional) Contains additional settings for configuring calls to LUIS.
     */
    constructor(application, luisPredictionOptions, includeApiResults) {
        this.luisRecognizer = new LuisRecognizer(application, luisPredictionOptions, true);
    }

    /**
     * Every conversation turn calls this method.
     * @param {TurnContext} turnContext Contains all the data needed for processing the conversation turn.
     */
    async onTurn(turnContext) {
        // By checking the incoming Activity type, the bot only calls LUIS in appropriate cases.
        if (turnContext.activity.type === ActivityTypes.Message) {
            // Perform a call to LUIS to retrieve results for the user's message.
            const results = await this.luisRecognizer.recognize(turnContext);

            // Since the LuisRecognizer was configured to include the raw results, get the `topScoringIntent` as specified by LUIS.
            const topIntent = results.luisResult.topScoringIntent;

            if (topIntent.intent !== 'None') {
                await turnContext.sendActivity(`LUIS Top Scoring Intent: ${ topIntent.intent }, Score: ${ topIntent.score }`);
            } else {
                // If the top scoring intent was "None" tell the user no valid intents were found and provide help.
                await turnContext.sendActivity(`No LUIS intents were found.
                                                \nThis sample is about identifying two user intents:
                                                \n - 'Calendar.Add'
                                                \n - 'Calendar.Find'
                                                \nTry typing 'Add Event' or 'Show me tomorrow'.`);
            }
        } else if (turnContext.activity.type === ActivityTypes.ConversationUpdate &&
            turnContext.activity.recipient.id !== turnContext.activity.membersAdded[0].id) {
            // If the Activity is a ConversationUpdate, send a greeting message to the user.
            await turnContext.sendActivity('Welcome to the NLP with LUIS sample! Send me a message and I will try to predict your intent.');
        } else if (turnContext.activity.type !== ActivityTypes.ConversationUpdate) {
            // Respond to all other Activity types.
            await turnContext.sendActivity(`[${ turnContext.activity.type }]-type activity detected.`);
        }
    }
}

module.exports.LuisBot = LuisBot;
```

LUIS 認識エンジンからは、候補となる意図に対し、発話がどの程度一致しているかについての情報が返されます。 結果オブジェクトの `luisResult.intents` プロパティには、スコア付けされた意図の配列が格納されます。 結果オブジェクトの `luisResult.topScoringIntent` プロパティには、最もスコアの高い意図とそのスコアが格納されます。

---

## <a name="extract-entities"></a>エンティティの抽出

LUIS アプリでは、意図の認識に加え、ユーザーの要求を満たすための重要な単語であるエンティティを抽出することもできます。 たとえば、天気のボットでは、LUIS アプリはユーザーのメッセージから天気予報の場所を抽出できる場合があります。

会話を構成する一般的な方法は、ユーザーのメッセージ内のエンティティを識別し、必要なエンティティが見つからなければプロンプトを表示するというものです。 次に、後続のステップがプロンプトへの応答を処理します。

<!--Snip
# [C#](#tab/cs)

Let's say the message from the user was "What's the weather in Seattle"? The [LuisRecognizer](https://docs.microsoft.com/en-us/dotnet/api/microsoft.bot.builder.ai.luis.luisrecognizer) gives you a [RecognizerResult](https://docs.microsoft.com/en-us/dotnet/api/microsoft.bot.builder.core.extensions.recognizerresult) with an [`Entities` property](https://docs.microsoft.com/en-us/dotnet/api/microsoft.bot.builder.core.extensions.recognizerresult#properties-) that has this structure:

```json
{
"$instance": {
    "Weather_Location": [
        {
            "startIndex": 22,
            "endIndex": 29,
            "text": "seattle",
            "score": 0.8073087
        }
    ]
},
"Weather_Location": [
        "seattle"
    ]
}
```

The following helper function can be added to your bot to get entities out of the `RecognizerResult` from LUIS. It will require the use of the `Newtonsoft.Json.Linq` library, which you'll have to add to your **using** statements.

```cs
// Get entities from LUIS result
private T GetEntity<T>(RecognizerResult luisResult, string entityKey)
{
    var data = luisResult.Entities as IDictionary<string, JToken>;
    if (data.TryGetValue(entityKey, out JToken value))
    {
        return value.First.Value<T>();
    }
    return default(T);
}
```

When gathering information like entities from multiple steps in a conversation, it can be helpful to save the information you need in your state. If an entity is found, it can be added to the appropriate state field. In your conversation if the current step already has the associated field completed, the step to prompt for that information can be skipped.

# [JavaScript](#tab/js)

Let's say the message from the user was "What's the weather in Seattle"? The [LuisRecognizer](https://docs.microsoft.com/en-us/javascript/api/botbuilder-ai/luisrecognizer) gives you a [RecognizerResult](https://docs.microsoft.com/en-us/javascript/api/botbuilder-core-extensions/recognizerresult) with an `entities` property that has this structure:

```json
{
"$instance": {
    "Weather_Location": [
        {
            "startIndex": 22,
            "endIndex": 29,
            "text": "seattle",
            "score": 0.8073087
        }
    ]
},
"Weather_Location": [
        "seattle"
    ]
}
```

This `findEntities` function looks for any entities recognized by the LUIS app that match the incoming `entityName`.

```javascript
// Helper function for finding a specified entity
// entityResults are the results from LuisRecognizer.get(context)
function findEntities(entityName, entityResults) {
    let entities = []
    if (entityName in entityResults) {
        entityResults[entityName].forEach(entity => {
            entities.push(entity);
        });
    }
    return entities.length > 0 ? entities : undefined;
}
```
/Snip-->

会話の複数のステップからのエンティティのような情報を収集する場合、必要な情報を状態に保存すると役に立つ可能性があります。 エンティティが見つかった場合は、それを適切な状態フィールドに追加できます。 会話内で現在のステップの関連フィールドが既に入力されている場合、情報を求めるステップはスキップできます。

## <a name="additional-resources"></a>その他のリソース

LUIS を使用しているサンプルについては、[[C#](https://aka.ms/cs-luis-sample)] または [[JavaScript](https://aka.ms/js-luis-sample)] 用のプロジェクトをご覧ください。

## <a name="next-steps"></a>次の手順

> [!div class="nextstepaction"]
> [ディスパッチ ツールを使用して LUIS と QnA サービスを組み合わせる](./bot-builder-tutorial-dispatch.md)
