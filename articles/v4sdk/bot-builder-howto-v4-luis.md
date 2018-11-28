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
ms.date: 11/16/18
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: faf26b1c4ba87061631f217ee074283759f77c97
ms.sourcegitcommit: 392c581aa2f59cd1798ee2136b6cfee56aa3ee6d
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/20/2018
ms.locfileid: "52156701"
---
# <a name="add-natural-language-understanding-to-your-bot"></a>ボットに自然言語の理解を追加する

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

ユーザーの真意を会話と文脈から理解する能力は難しいタスクかもしれませんが、より自然な会話の印象をボットに与えることができます。 そうした能力は LUIS と呼ばれる Language Understanding によって実現され、ボットがユーザーのメッセージの意図を認識できるように、ユーザーがより自然な言葉を使用できるように、また、会話フローがより適切に管理されるようになります。 このトピックでは、LUIS を使用して数種類の意図を認識する単純なボットの設定手順について説明します。 
## <a name="prerequisites"></a>前提条件
- [luis.ai](https://www.luis.ai) アカウント
- [Bot Framework Emulator](https://github.com/Microsoft/BotFramework-Emulator/blob/master/README.md#download)
- この記事のコードは、**LUIS による NLP** のサンプルをベースにしています。 サンプルのコピー ([C#](https://aka.ms/cs-luis-sample) または [JS](https://aka.ms/js-luis-sample)) が必要になります。 
- [ボットの基本](bot-builder-basics.md)、[自然言語処理](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/what-is-luis)、および [.bot](bot-file-basics.md) ファイルに関する知識。

## <a name="create-a-luis-app-in-the-luis-portal"></a>LUIS ポータルでの LUIS アプリの作成
LUIS ポータルにサインインして、ご自身のバージョンのサンプル LUIS アプリを作成します。 アプリケーションは、**[マイ アプリ]** で作成および管理できます。 

1. **[Import new app]\(新しいアプリのインポート\)** を選択します。 
1. **[Choose App file (JSON format)...]\(アプリ ファイル (JSON 形式) を選択...\)** をクリックします。 
1. サンプルの `CognitiveModels` フォルダーにある `reminders.json` ファイルを選択します。 **[Optional Name]\(オプション名\)** に「**LuisBot**」と入力します。 このファイルには、Calendar-Add、Calendar-Find、None の 3 つの意図が含まれています。 これらの意図を使って、ユーザーがどのようなつもりでボットに送メッセージを信しているのかを把握します。 
1. アプリを[トレーニング](https://docs.microsoft.com/en-us/azure/cognitive-services/LUIS/luis-how-to-train)します。
1. アプリを*運用*環境に[発行](https://docs.microsoft.com/en-us/azure/cognitive-services/LUIS/publishapp)します。

### <a name="obtain-values-to-connect-to-your-luis-app"></a>LUIS アプリに接続するための値を取得する

LUIS アプリには、その発行後、ボットからアクセスできるようになります。 ボット内から LUIS アプリにアクセスするためには、いくつかの値を記録する必要があります。 その情報は、LUIS ポータルを使用して取得できます。

#### <a name="retrieve-application-information-from-the-luisai-portal"></a>LUIS.ai ポータルからアプリケーション情報を取得する
.bot ファイルは、すべてのサービス参照を 1 か所にまとめる場所として機能します。 取得した情報は、次のセクションで .bot ファイルに追加されます。 
1. 発行済みの LUIS アプリを [luis.ai](https://www.luis.ai) から選択します。
1. 発行済み LUIS アプリを開いた状態で **[管理]** タブを選択します。
1. 左側の **[アプリケーション情報]** タブで、_[アプリケーション ID]_ に表示される値を <YOUR_APP_ID> として記録します。
1. 左側の **[Keys and Endpoints]\(キーとエンドポイント\)** タブを選択し、_[オーサリング キー]_ に表示される値を <YOUR_AUTHORING_KEY> として記録します。 "*お使いのサブスクリプション キー*" は、"*お使いのオーサリング キー*" と同じであることにご注意ください。 
1. 下へスクロールしてページの最後まで移動し、"_リージョン_" に表示される値を <YOUR_REGION> として記録します。
1. "_エンドポイント_" に表示される値を <YOUR_ENDPOINT> として記録します。

#### <a name="update-the-bot-file"></a>ボット ファイルを更新する
LUIS アプリにアクセスするために必要な情報 (アプリケーション ID、オーサリング キー、サブスクリプション キー、エンドポイント、リージョンなど) を `nlp-with-luis.bot` ファイルに追加します。 これらは、発行済みの LUIS アプリから先ほど保存した値です。

```json
{
    "name": "LuisBot",
    "description": "",
    "services": [
        {
            "type": "endpoint",
            "name": "development",
            "endpoint": "http://localhost:3978/api/messages",
            "appId": "",
            "appPassword": "",
            "id": "166"
        },
        {
            "type": "luis",
            "name": "LuisBot",
            "appId": "<luis appid>",
            "version": "0.1",
            "authoringKey": "<luis authoring key>",
            "subscriptionKey": "<luis subscription key>",
            "region": "<luis region>",
            "id": "158"
        }
    ],
    "padlock": "",
    "version": "2.0"
}
```
# <a name="ctabcs"></a>[C#](#tab/cs)

## <a name="configure-your-bot-to-use-your-luis-app"></a>LUIS アプリを使用するためのボットの構成

次に、`BotServices.cs` の BotService クラスの新しいインスタンスを初期化します。この初期化により、前述の情報が `.bot` ファイルから取得されます。 外部サービスは、`BotConfiguration` クラスを使用して構成されます。

```csharp
public class BotServices
{
    // Initializes a new instance of the BotServices class
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

    // Gets the set of LUIS Services used. LuisServices is represented as a dictionary.  
    public Dictionary<string, LuisRecognizer> LuisServices { get; } = new Dictionary<string, LuisRecognizer>();
}
```

次に、`ConfigureServices` メソッド内で次のコードを使用して、LUIS アプリをシングルトンとして `Startup.cs` ファイルに登録します。

```csharp
public void ConfigureServices(IServiceCollection services)
{
    var secretKey = Configuration.GetSection("botFileSecret")?.Value;
    var botFilePath = Configuration.GetSection("botFilePath")?.Value;

    // Loads .bot configuration file and adds a singleton that your Bot can access through dependency injection.
    var botConfig = BotConfiguration.Load(botFilePath ?? @".\nlp-with-luis.bot", secretKey);
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
        
        // ...
    });
}
```

次に、`Luis.cs` ファイルで、ボットがこの LUIS インスタンスを取得します。

```csharp
public class LuisBot : IBot
{
    public static readonly string LuisKey = "LuisBot";
    private const string WelcomeText = "This bot will introduce you to natural language processing with LUIS. Type an utterance to get started";

    // Services configured from the ".bot" file.
    private readonly BotServices _services;

    // Initializes a new instance of the LuisBot class.
    public LuisBot(BotServices services)
    {
        _services = services ?? throw new System.ArgumentNullException(nameof(services));
        if (!_services.LuisServices.ContainsKey(LuisKey))
        {
            throw new System.ArgumentException($"Invalid configuration....");
        }
    }
    // ...
}
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

このサンプルでは、スタートアップ コードが **index.js** ファイルにあります。ボット ロジックのコードは **bot.js** ファイルに、また、追加の構成情報は **nlp-with-luis.bot** ファイルに含まれています。

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

<!--
## Extract entities

Besides recognizing intent, a LUIS app can also extract entities, which are important words for fulfilling a user's request. For example, for a weather bot, the LUIS app might be able to extract the location for the weather report from the user's message.

A common way to structure your conversation is to identify any entities in the user's message, and prompt for any of the required entities that are not found. Then, the subsequent steps handle the response to the prompt.


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


When gathering information like entities from multiple steps in a conversation, it can be helpful to save the information you need in your state. If an entity is found, it can be added to the appropriate state field. In your conversation if the current step already has the associated field completed, the step to prompt for that information can be skipped.

/Snip -->

## <a name="test-the-bot"></a>ボットのテスト

1. ご自身のマシンを使ってローカルでサンプルを実行します。 手順については、readme ファイルで [C#](https://github.com/Microsoft/BotBuilder-Samples/blob/master/samples/csharp_dotnetcore/12.nlp-with-luis/README.md) または [JS](https://github.com/Microsoft/BotBuilder-Samples/blob/master/samples/javascript_nodejs/12.nlp-with-luis/README.md) サンプルを参照してください。

1. エミュレーターで、次に示すようにメッセージを入力します。 

![NLP サンプルをテストする](~/media/emulator-v4/nlp-luis-sample-testing.png)

ボットは、最もスコアが高い意図を使用して応答します。ここでは `Calendar-Add` 意図です。 LUIS.ai ポータルでインポートした `reminders.json` ファイルによって意図が定義されていることを思い出してください。

予測スコアは、予測結果についての LUIS の信頼度を示します。 予測スコアは、0 と 1 の間です。 十分に信頼できる LUIS スコアの例は 0.99 です。 信頼度の低いのスコアの例は 0.01 です。 

## <a name="next-steps"></a>次の手順

> [!div class="nextstepaction"]
> [QnA Maker を使用して質問に回答する](./bot-builder-howto-qna.md)
