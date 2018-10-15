---
title: 言語理解のための LUIS の使用 | Microsoft Docs
description: 自然言語理解のための LUIS を Bot Builder SDK と共に使用する方法について説明します。
keywords: Language Understanding, LUIS, 意図, 認識エンジン, エンティティ, ミドルウェア
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 09/19/18
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: c4a7cfba6f588c95dbebf4886ffd7e432d99c3ae
ms.sourcegitcommit: 3cb288cf2f09eaede317e1bc8d6255becf1aec61
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/27/2018
ms.locfileid: "47389803"
---
# <a name="using-luis-for-language-understanding"></a>言語理解のための LUIS の使用

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

ユーザーの真意を会話と文脈から理解する能力は難しいタスクかもしれませんが、より自然な会話の印象をボットに与えることができます。 そうした能力は LUIS と呼ばれる Language Understanding によって実現され、ボットがユーザーのメッセージの意図を認識できるように、ユーザーがより自然な言葉を使用できるように、また、会話フローがより適切に管理されるようになります。 LUIS とボットの統合についてさらなる背景情報が必要な場合は、「[language understanding for bots](./bot-builder-concept-LUIS.md)」(ボットのための言語理解) を参照してください。 

このトピックでは、LUIS を使用して数種類の意図を認識する単純なボットの設定手順を示します。

## <a name="installing-packages"></a>パッケージのインストール

まず、LUIS に必要なパッケージがあることを確認します。

# <a name="ctabcs"></a>[C# を選択した場合](#tab/cs)

次の NuGet パッケージの v4 バージョンへの[参照を追加](https://docs.microsoft.com/en-us/nuget/tools/package-manager-ui)します。


* `Microsoft.Bot.Builder.AI.LUIS`

# <a name="javascripttabjs"></a>[JavaScript を選択した場合](#tab/js)

npm を使用して、プロジェクトに botbuilder パッケージと botbuilder-ai パッケージをインストールします。

* `npm install --save botbuilder`
* `npm install --save botbuilder-ai`

---

## <a name="set-up-your-luis-app"></a>LUIS アプリを設定する

まず、[luis.ai](https://www.luis.ai) で作成するサービスである _LUIS アプリ_を設定します。 特定の意図を認識できるよう、その LUIS アプリをトレーニングすることができます。 LUIS アプリの作成方法について詳しくは、LUIS のサイトをご覧ください。

この例では、ヘルプ、キャンセル、天気の各意図を認識できるデモ LUIS アプリを使用します。アプリ ID は既にサンプル コードに含まれています。 Cognitive Services キーが必要です。このキーは、[luis.ai](https://www.luis.ai) にログインし、**[User settings]\(ユーザー設定\)** > **[Authoring Key]\(オーサリング キー\)** からキーをコピーすることで入手できます。

> [!NOTE]
> この例で使用するパブリック LUIS アプリの独自のコピーを作成するには、[JSON](https://github.com/Microsoft/LUIS-Samples/blob/master/examples/simple-bot-example/FirstSimpleBotExample.json) LUIS ファイルをコピーします。 次に、LUIS アプリを[インポート](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/create-new-app#import-new-app)し、[トレーニングを行い](https://docs.microsoft.com/en-us/azure/cognitive-services/LUIS/luis-how-to-train)、[発行](https://docs.microsoft.com/en-us/azure/cognitive-services/LUIS/publishapp)します。 サンプル コードのパブリック アプリ ID を新しい LUIS アプリのアプリ ID に置き換えます。


### <a name="configure-your-bot-to-call-your-luis-app"></a>LUIS アプリを呼び出すようにボットを構成します。

# <a name="ctabcs"></a>[C# を選択した場合](#tab/cs)

ターンごとに LUIS アプリを作成して呼び出すことが可能である一方で、LUIS サービスをシングルトンとして登録し、その後それらをパラメーターとしてボットのコンストラクターに渡すことをお勧めします。 このほうがやや複雑であるため、ここではそのメソッドを紹介します。

Echo ボット テンプレートから始め、**Startup.cs** を開きます。 

`Microsoft.Bot.Builder.AI.LUIS` の `using` ステートメントを追加します

```csharp
// add this
using Microsoft.Bot.Builder.AI.LUIS;
```

`ConfigureServices` の末尾の、状態の初期化後に、次のコードを追加します。 これは `appsettings.json` ファイルから情報を取得しますが、それらの文字列はこの記事の最後にリンクされているサンプルのように`.bot` ファイルから取得することも、テスト用にハード コーディングすることもできます。

シングルトンは新しい `LuisRecognizer` をコンストラクターに返します。

```csharp
    // Create and register a LUIS recognizer.
    services.AddSingleton(sp =>
    {
        // Get LUIS information from appsettings.json.
        var section = this.Configuration.GetSection("Luis");
        var luisApp = new LuisApplication(
            applicationId: section["AppId"],
            endpointKey: section["SubscriptionKey"],
            azureRegion: section["Region"]);

        // Specify LUIS options. These may vary for your bot.
        var luisPredictionOptions = new LuisPredictionOptions
        {
            IncludeAllIntents = true,
        };

        return new LuisRecognizer(
            application: luisApp,
            predictionOptions: luisPredictionOptions,
            includeApiResults: true);
    });
```

`<subscriptionKey>` の代わりに [luis.ai](https://www.luis.ai) からサブスクリプション キーを貼り付けます。 これを見つける最も簡単な方法は、右上のアカウント名をクリックし、**[設定]** に移動する方法で、**[Authoring Key]\(オーサリング キー\)** と呼ばれます。

> [!NOTE]
> パブリックのものではなく独自の LUIS アプリを使用している場合、LUIS アプリの ID、サブスクリプション キー、および URL は [luis.ai](https://www.luis.ai) から入手できます。 これらはアプリのページの **[Publish]\(発行\)** タブや **[Settings]\(設定\)** タブで見つかります。
>
>[luis.ai](https://www.luis.ai) にログインし、**[Publish]\(発行\)** タブに移動して、**[Resources and Keys]\(リソースとキー\)** の **[Endpoint]\(エンドポイント\)** 列を調べて、`LuisModel` で使用するベース URL を見つけることができます。 ベース URL は、サブスクリプション ID およびその他のパラメーターの前の**エンドポイント URL** の部分です。

次に、ボットにこの LUIS インスタンスを付与する必要があります。 **EchoBot.cs** を開き、ファイルの先頭に次のコードを追加します。 参照用にクラスの見出しや状態の項目も含まれていますが、ここでは説明しません。

```csharp
public class EchoBot : IBot
{
    /// <summary>
    /// Gets the Echo Bot state.
    /// </summary>
    private IStatePropertyAccessor<EchoState> EchoStateAccessor { get; }

    /// <summary>
    /// Gets the LUIS recognizer.
    /// </summary>
    private LuisRecognizer Recognizer { get; } = null;

    public EchoBot(ConversationState state, LuisRecognizer luis)
    {
        EchoStateAccessor = state.CreateProperty<EchoState>("EchoBot.EchoState");

        // The incoming luis variable is the LUIS Recognizer we added above.
        this.Recognizer = luis ?? throw new ArgumentNullException(nameof(luis));
    }

    /// ...
```

# <a name="javascripttabjs"></a>[JavaScript を選択した場合](#tab/js)

まず、JavaScript の [クイックスタート](../javascript/bot-builder-javascript-quickstart.md)のステップに従ってボットを作成します。 ここでは LUIS 情報をボットにハードコーディングしますが、この記事の最後にリンクされているサンプルのように `.bot` ファイルからプルすることもできます。

新しいボットで、`LuisRecognizer` クラスを要求するように **app.js** を編集し、LUIS モデルのインスタンスを作成します。

```javascript
const { ActivityTypes } = require('botbuilder');
const { LuisRecognizer } = require('botbuilder-ai');

const luisApplication = {
    // This appID is for a public app that's made available for demo purposes
    // You can use it or use your own LUIS "Application ID" at https://www.luis.ai under "App Settings".
     applicationId: 'eb0bf5e0-b468-421b-9375-fdfb644c512e',
    // Replace endpointKey with your "Subscription Key"
    // your key is at https://www.luis.ai under Publish > Resources and Keys, look in the Endpoint column
    // The "subscription-key" is embeded in the Endpoint link. 
    endpointKey: '<your subscription key>',
    // You can find your app's region info embeded in the Endpoint link as well.
    // Some examples of regions are `westus`, `westcentralus`, `eastus2`, and `southeastasia`.
    azureRegion: 'westus'
}

// Create configuration for LuisRecognizer's runtime behavior.
const luisPredictionOptions = {
    includeAllIntents: true,
    log: true,
    staging: false
}

// Create the bot that handles incoming Activities.
const luisBot = new LuisBot(luisApplication, luisPredictionOptions);
```

次に、ボット `LuisBot` のコンストラクターで、LuisRecognizer インスタンスを作成するアプリケーションを取得します。

```javascript
    /**
     * The LuisBot constructor requires one argument (`application`) which is used to create an instance of `LuisRecognizer`.
     * @param {object} luisApplication The basic configuration needed to call LUIS. In this sample the configuration is retrieved from the .bot file.
     * @param {object} luisPredictionOptions (Optional) Contains additional settings for configuring calls to LUIS.
     */
    constructor(application, luisPredictionOptions) {
        this.luisRecognizer = new LuisRecognizer(application, luisPredictionOptions, true);
    }
```

> [!NOTE] 
> パブリックのものではなく独自の LUIS アプリを使用している場合、LUIS アプリのアプリケーション ID、サブスクリプション キー、リージョンは [https://www.luis.ai](https://www.luis.ai) から入手できます。 これらはアプリのページの [Publish]\(発行\) タブや [Settings]\(設定\) タブで見つかります。

---

これで、LUIS 言語理解がボットに構成されました。 次は、LUIS から意図を取得する方法を見てみましょう。

## <a name="get-the-intent-by-calling-luis"></a>LUIS を呼び出して意図を取得する

ボットは LUIS 認識エンジンを呼び出すことによって、LUIS から結果を取得します。

# <a name="ctabcs"></a>[C# を選択した場合](#tab/cs)

LUIS アプリが検出した意図に基づいた返信をボットで単純に送信するには、`LuisRecognizer` を呼び出して `RecognizerResult` を取得します。 これは、LUIS の意図を取得する必要があるときはいつでもコード内で行うことができます。

```cs
using System.Threading.Tasks;
using Microsoft.Bot;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Core.Extensions;
using Microsoft.Bot.Schema;
// add this reference
using Microsoft.Bot.Builder.AI.LUIS;

namespace EchoBot
{
    public class EchoBot : IBot
    {
        /// <summary>
        /// Echo bot turn handler 
        /// </summary>
        /// <param name="context">Turn scoped context containing all the data needed
        /// for processing this conversation turn. </param>        
        public async Task OnTurnAsync(ITurnContext context, System.Threading.CancellationToken token)
        {            
            // This bot is only handling Messages
            if (context.Activity.Type == ActivityTypes.Message)
            {
                // Call LUIS recognizer
                var result = this.Recognizer.RecognizeAsync(context, System.Threading.CancellationToken.None);
                
                var topIntent = result?.GetTopScoringIntent();

                switch ((topIntent != null) ? topIntent.Value.intent : null)
                {
                    case null:
                        await context.SendActivity("Failed to get results from LUIS.");
                        break;
                    case "None":
                        await context.SendActivity("Sorry, I don't understand.");
                        break;
                    case "Help":
                        await context.SendActivity("<here's some help>");
                        break;
                    case "Cancel":
                        // Cancel the process.
                        await context.SendActivity("<cancelling the process>");
                        break;
                    case "Weather":
                        // Report the weather.
                        await context.SendActivity("The weather today is sunny.");
                        break;
                    default:
                        // Received an intent we didn't expect, so send its name and score.
                        await context.SendActivity($"Intent: {topIntent.Value.intent} ({topIntent.Value.score}).");
                        break;
                }
            }
        }
    }    
}

```

発話で認識されたすべての意図は、意図名からスコアへのマップとして返され、`result.Intents` からアクセスできます。 結果セットの最高スコアの意図を簡単に見つけるための、静的な `LuisRecognizer.topIntent()` メソッドが提供されます。

認識されたすべてのエンティティは、エンティティ名から値へのマップとして返され、`results.entities` を使用してアクセスされます。 LuisRecognizer の作成時に `verbose=true` の設定を渡すことによって、追加のエンティティ メタデータを返すことができます。 追加されたメタデータには `results.entities.$instance` を使用してアクセスできます。

# <a name="javascripttabjs"></a>[JavaScript を選択した場合](#tab/js)

受信アクティビティをリッスンするようにコードを編集し、`LuisRecognizer` を呼び出して `RecognizerResult` を取得するようにします。

```javascript
const { ActivityTypes } = require('botbuilder');
const { LuisRecognizer } = require('botbuilder-ai');

// Listen for incoming requests 
server.post('/api/messages', (req, res) => {
    // Route received request to adapter for processing
    adapter.processActivity(req, res, async (context) => {
        if (context.activity.type === 'message') {
            // Perform a call to LUIS to retrieve results for the user's message.
            const results = await this.luisRecognizer.recognize(turnContext);

            // Since the LuisRecognizer was configured to include the raw results, get the `topScoringIntent` as specified by LUIS.
            const topIntent = results.luisResult.topScoringIntent;
            
            switch (topIntent) {
                case 'None':
                    await context.sendActivity("Sorry, I don't understand.")
                    break;
                case 'Cancel':
                    await context.sendActivity("<cancelling the process>")
                    break;
                case 'Help':
                    await context.sendActivity("<here's some help>");
                    break;
                case 'Weather':
                    await context.sendActivity("The weather today is sunny.");
                    break;                        
                case 'null':
                    await context.sendActivity("Failed to get results from LUIS.")
                    break;
                default:
                    // Received an intent we didn't expect, so send its name and score.
                    await context.sendActivity(`The top intent was ${topIntent}`);
            }
        }
    });
});
```

発話で認識されたすべての意図は、意図名からスコアへのマップとして返され、`results.intents` からアクセスできます。 結果セットの最高スコアの意図を簡単に見つけるための、静的な `LuisRecognizer.topIntent()` メソッドが提供されます。


---

Bot Framework Emulator でボットを実行し、「天気」、「ヘルプ」、「キャンセル」などと言ってみます。

![ボットを実行する](./media/how-to-luis/run-luis-bot.png)

## <a name="extract-entities"></a>エンティティの抽出

LUIS アプリでは、意図の認識に加え、ユーザーの要求を満たすための重要な単語であるエンティティを抽出することもできます。 たとえば、天気のボットでは、LUIS アプリはユーザーのメッセージから天気予報の場所を抽出できる場合があります。

会話を構成する一般的な方法は、ユーザーのメッセージ内のエンティティを識別し、必要なエンティティが見つからなければプロンプトを表示するというものです。 次に、後続のステップがプロンプトへの応答を処理します。

# <a name="ctabcs"></a>[C# を選択した場合](#tab/cs)

ユーザーからのメッセージが「シアトルの天気は?」だったとします。 `LuisRecognizer` は `Entities` プロパティを持つ `RecognizerResult` を付与します。これは次の構造を持ちます。

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

次のヘルパー関数をボットに追加して、LUIS の `RecognizerResult` からエンティティを取得できます。 これには `Newtonsoft.Json.Linq` ライブラリの使用が要求され、それを **using** ステートメントに追加する必要があります。

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

会話の複数のステップからのエンティティのような情報を収集する場合、必要な情報を状態に保存すると役に立つ可能性があります。 エンティティが見つかった場合は、それを適切な状態フィールドに追加できます。 会話内で現在のステップの関連フィールドが既に入力されている場合、情報を求めるステップはスキップできます。

# <a name="javascripttabjs"></a>[JavaScript を選択した場合](#tab/js)

ユーザーからのメッセージが「シアトルの天気は?」だったとします。 `LuisRecognizer` は `entities` プロパティを持つ `RecognizerResult` を付与します。次の構造を持ちます。

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

この `findEntities` 関数は、LUIS アプリによって認識される 受信 `entityName` と一致するエンティティを探します。


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

会話の複数のステップからのエンティティのような情報を収集する場合、必要な情報を状態に保存すると役に立つ可能性があります。 エンティティが見つかった場合は、それを適切な状態フィールドに追加できます。 会話内で現在のステップの関連フィールドが既に入力されている場合、情報を求めるステップはスキップできます。

## <a name="additional-resources"></a>その他のリソース

LUIS を使用しているサンプルについては、[[C#](https://aka.ms/cs-luis-sample)] または [[JavaScript](https://aka.ms/js-luis-sample)] 用のプロジェクトをご覧ください。

## <a name="next-steps"></a>次の手順

> [!div class="nextstepaction"]
> [ディスパッチ ツールを使用して LUIS と QnA サービスを組み合わせる](./bot-builder-tutorial-dispatch.md)
