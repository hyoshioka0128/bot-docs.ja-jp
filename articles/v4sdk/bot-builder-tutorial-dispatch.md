---
title: 複数の LUIS および QnA モデルを使用する | Microsoft Docs
description: ボットで LUIS や QnA Maker を使用する方法について説明します。
keywords: Luis, QnA, ディスパッチ ツール, 複数のサービス, 意図のルーティング
author: DeniseMak
ms.author: v-demak
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 01/15/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: c798c26f108458e1caeb16aa22c02c6e7c70fb61
ms.sourcegitcommit: 3cc768a8e676246d774a2b62fb9c688bbd677700
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/16/2019
ms.locfileid: "54323658"
---
# <a name="use-multiple-luis-and-qna-models"></a>複数の LUIS および QnA モデルを使用する

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

このチュートリアルでは、ボットでサポートされているさまざまなシナリオについて、複数の LUIS モデル と QnA Maker サービスがある場合に、ディスパッチ サービスを使用して発話をルーティングする方法を示します。 ここでは、ホーム オートメーションと気象情報に関連する会話に複数の LUIS モデルを使用し、また、入力として FAQ テキスト ファイルに基づいて質問に回答する QnA Maker サービスを使用して、ディスパッチを構成します。 このサンプルでは、以下のサービスが組み合わされます。

| Name | 説明 |
|------|------|
| HomeAutomation | 関連付けられているエンティティ データによってホーム オートメーションの意図を認識する LUIS アプリ。|
| Weather | 場所データを使用して `Weather.GetForecast` および `Weather.GetCondition` の意図を認識する LUIS アプリ。|
| FAQ  | ボットに関する簡単な質問への回答を提供する QnA Maker KB。 |

## <a name="prerequisites"></a>前提条件

- この記事のコードは、**ディスパッチによる NLP** のサンプルをベースにしています。 サンプルのコピー ([C#](https://aka.ms/dispatch-sample-cs) または [JS](https://aka.ms/dispatch-sample-js)) が必要になります。
- [ボットの基本](bot-builder-basics.md)、[自然言語処理](bot-builder-howto-v4-luis.md)、[QnA Maker](bot-builder-howto-qna.md)、および [.bot](bot-file-basics.md) ファイルに関する知識が必要です。
- テスト用の [Bot Framework Emulator](https://github.com/Microsoft/BotFramework-Emulator/blob/master/README.md#download)。

## <a name="create-the-services-and-test-the-bot"></a>サービスの作成とボットのテスト

[C#](https://aka.ms/dispatch-sample-readme-cs) または [JS](https://aka.ms/dispatch-sample-readme-js) の **README** の指示に従って、コマンド ライン インターフェイスの呼び出しを使用してこのボットを作成するか、以下の手順に従って、Azure、LUIS、QnAMaker のユーザー インターフェイスを使用して手動でボットを作成することができます。

 ### <a name="create-your-bot-using-service-ui"></a>サービス UI を使用してボットを作成する
 
手作業でのボットの作成を始めるには、GitHub の [BotFramework-Samples](https://aka.ms/botdispatchgitsamples) リポジトリにある 4 つのファイル [home-automation.json](https://aka.ms/dispatch-home-automation-json)、[weather.json](https://aka.ms/dispatch-weather-json)、[nlp-with-dispatchDispatch.json](https://aka.ms/dispatch-dispatch-json)、[QnAMaker.tsv](https://aka.ms/dispatch-qnamaker-tsv) をローカル フォルダーにダウンロードします。1 つの方法としては、上で示した GitHub リポジトリのリンクを開き、**BotFramework-Samples** をクリックしてから、リポジトリをローカル マシンに "複製またはダウンロード" します。 これらのファイルは、前提条件で説明されているサンプルとは異なるリポジトリにあることに注意してください。

### <a name="manually-create-luis-apps"></a>LUIS アプリを手動で作成する

[LUIS Web ポータル](https://www.luis.ai/)にログインします。 _[My apps]\(マイ アプリ\)_ セクションの _[Import new app]\(新しいアプリのインポート\)_ タブを選択します。 次のダイアログ ボックスが表示されます。

![LUIS JSON ファイルをインポートする](./media/tutorial-dispatch/import-new-luis-app.png)

_[Choose app file]\(アプリ ファイルの選択\)_ ボタンを選択し、ダウンロードしたファイル 'home-automation.json' を選択します。 省略可能な名前のフィールドは空白のままにします。 _[完了]_ を選択します。

LUIS で Home Automation アプリが開いたら、_[Train]\(トレーニング\)_ ボタンを選択します。 これで、'home-automation.json' ファイルでインポートした一連の発話を使用してアプリがトレーニングされます。

トレーニングが完了したら、_[Publish]\(公開\)_ ボタンを選択します。 次のダイアログ ボックスが表示されます。

![LUIS アプリを公開する](./media/tutorial-dispatch/publish-luis-app.png)

'運用' 環境を選択し、_[Publish]\(公開\)_ ボタンを選択します。

新しい LUIS アプリが公開されたら、_[MANAGE]\(管理\)_ タブを選択します。[Application Information]\(アプリケーション情報\) ページで、`Application ID` と `Display name` の値を書き留めます。 [Key and Endpoints]\(キーとエンドポイント\) ページで、`Authoring Key` と `Region` の値を書き留めます。 これらの値は、後で 'nlp-with-dispatch.bot' によって使用されます。

完了したら、ローカルにダウンロードした'weather.json' ファイルと 'nlp-with-dispatchDispatch.json' ファイルに対して同じ手順を繰り返して、自分の LUIS 天気アプリと LUIS ディスパッチ アプリの両方を "_トレーニング_" し、"_公開_" します。

### <a name="manually-create-qna-maker-app"></a>QnA Maker アプリを手動で作成する

QnA Maker ナレッジ ベースを設定する最初の手順は、まず Azure で QnA Maker サービスを設定することです。 そのためには、[こちら](https://aka.ms/create-qna-maker)で説明されている手順に従います。 次に [QnAMaker Web ポータル](https://qnamaker.ai)にログインします。 手順 2 に進みます

![QnA の作成手順 2](./media/tutorial-dispatch/create-qna-step-2.png)

そして選択します
1. Azure AD アカウント。
1. Azure サブスクリプション名。
1. QnA Maker サービス用に作成した名前 (最初の段階でこのプル ダウン リストに Azure QnA サービスが表示されない場合は、ページを更新してみてください)。 

手順 3 に進みます

![QnA の作成手順 3](./media/tutorial-dispatch/create-qna-step-3.png)

QnA Maker ナレッジベースの名前を指定します。 この例では、'sample-qna' という名前を使用します。

手順 4 に進みます

![QnA の作成手順 4](./media/tutorial-dispatch/create-qna-step-4.png)

_[+ Add File]\(ファイルの追加\)_ オプションを選択し、ダウンロードしたファイル 'QnAMaker.tsv' を選択します

ナレッジベースには _Chit-chat_ の性格を追加する選択がもう 1 つありますが、この例にはこのオプションが含まれていません。

_[Save and train]\(保存してトレーニング\)_ を選択し、完了したら、_[PUBLISH]\(公開\)_ タブを選択してアプリを公開します。

QnA Maker アプリが公開されたら、_[SETTINGS]\(設定\)_ タブを選択し、[Deployment details]\(デプロイの詳細\) まで下にスクロールします。 _Postman_ サンプル HTTP 要求の次の値を書き留めます。

```
POST /knowledgebases/<Your_Knowledgebase_Id>/generateAnswer
Host: <Your_Hostname>
Authorization: EndpointKey <Your_Endpoint_Key>
```
これらの値は、後で 'nlp-with-dispatch.bot' によって使用されます。

### <a name="manually-update-your-bot-file"></a>.bot ファイルを手動で更新する

すべてのサービス アプリを作成したら、それぞれの情報を 'nlp-with-dispatch.bot' ファイルに追加する必要があります。 以前にダウンロードした C# または JS サンプル ファイルに含まれるこのファイルを開きます。 各セクションに、"type": "luis" または "type": "dispatch" の値を追加します。

```
"appId": "<Your_Recorded_App_Id>",
"authoringKey": "<Your_Recorded_Authoring_Key>",
"subscriptionKey": "<Your_Recorded_Authoring_Key>",
"version": "0.1",
"region": "<Your_Recorded_Region>",
```

"type": "qna" のセクションの場合は、次の値を追加します。

```
"type": "qna",
"name": "sample-qna",
"id": "201",
"kbId": "<Your_Recorded_Knowledgebase_Id>",
"subscriptionKey": "<Your_Azure_Subscription_Key>", // Used when creating your QnA service.
"endpointKey": "<Your_Recorded_Endpoint_Key>",
"hostname": "<Your_Recorded_Hostname>"
```

すべての変更が完了したら、このファイルを保存します。

### <a name="test-your-bot"></a>ボットをテストする

次はエミュレーターを使用してサンプルを実行します。 エミュレーターを開いたら、'nlp-with-dispatch.bot' ファイルを選択します。

参考のために、サービスに含めた質問とコマンドの一部を次に示します。

* QnA Maker
  * `hi`、`good morning`
  * `what are you`、`what do you do`
* LUIS (ホーム オートメーション)
  * `turn on bedroom light`
  * `turn off bedroom light`
  * `make some coffee`
* LUIS (天気)
  * `whats the weather in redmond washington`
  * `what's the forecast for london`
  * `show me the forecast for nebraska`

### <a name="connecting-to-the-services-from-your-bot"></a>ボットからサービスへの接続

Dispatch、LUIS、および QnA Maker サービスに接続するために、ボットは **.bot** ファイルから情報をプルします。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

**Startup.cs** 内の `ConfigureServices` によって構成ファイルが読み込まれます。`InitBotServices` によって、サービスを初期化するためにその情報が使用されます。 作成されるたびに、ボットは登録された `BotServices` オブジェクトを使用して初期化されます。 これら 2 つのメソッドの関連する部分を次に示します。

```csharp
public void ConfigureServices(IServiceCollection services)
{
    //...
    var botConfig = BotConfiguration.Load(botFilePath ?? @".\nlp-with-dispatch.bot", secretKey);
    services.AddSingleton(sp => botConfig
        ?? throw new InvalidOperationException($"The .bot config file could not be loaded. ({botConfig})"));

    // ...
    
    var connectedServices = InitBotServices(botConfig);
    services.AddSingleton(sp => connectedServices);
    
    services.AddBot<NlpDispatchBot>(options =>
    {
          
          // The Memory Storage used here is for local bot debugging only. 
          // When the bot is restarted, everything stored in memory will be gone.

          Storage dataStore = new MemoryStorage();

          // ...

          // Create Conversation State object.
          // The Conversation State object is where we persist anything at the conversation-scope.

          var conversationState = new ConversationState(dataStore);
          options.State.Add(conversationState);
     });
}

```
次のコードでは、外部サービスへのボットの参照を初期化します。 たとえば、LUIS および QnaMaker サービスがここで作成されます。 これらの外部サービスは、(お使いの ".bot" ファイルのコンテンツに基づいて) `BotConfiguration` クラスを使用して構成されます。

```csharp
private static BotServices InitBotServices(BotConfiguration config)
{
    var qnaServices = new Dictionary<string, QnAMaker>();
    var luisServices = new Dictionary<string, LuisRecognizer>();

    foreach (var service in config.Services)
    {
        switch (service.Type)
        {
            case ServiceTypes.Luis:
                {
                    // ...
                    var app = new LuisApplication(luis.AppId, luis.AuthoringKey, luis.GetEndpoint());
                    var recognizer = new LuisRecognizer(app);
                    luisServices.Add(luis.Name, recognizer);
                    break;
                }

            case ServiceTypes.Dispatch:
                // ...
                var dispatchApp = new LuisApplication(dispatch.AppId, dispatch.AuthoringKey, dispatch.GetEndpoint());

                // Since the Dispatch tool generates a LUIS model, we use the LuisRecognizer to resolve the
                // dispatching of the incoming utterance.
                var dispatchARecognizer = new LuisRecognizer(dispatchApp);
                luisServices.Add(dispatch.Name, dispatchARecognizer);
                break;

            case ServiceTypes.QnA:
                {
                    // ...
                    var qnaEndpoint = new QnAMakerEndpoint()
                    {
                        KnowledgeBaseId = qna.KbId,
                        EndpointKey = qna.EndpointKey,
                        Host = qna.Hostname,
                    };

                    var qnaMaker = new QnAMaker(qnaEndpoint);
                    qnaServices.Add(qna.Name, qnaMaker);
                    break;
                }
        }
    }

    return new BotServices(qnaServices, luisServices);
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

サンプル コードでは、事前に定義された名前付け定数を使用して、.bot ファイルのさまざまなセクションを識別しています。 _nlp-with-dispatch.bot_ ファイルの元の名前付け例からセクション名を変更した場合は、必ず **bot.js**、**homeAutomation.js**、**qna.js**、または **weather.js** ファイルで関連する定数の宣言を探し、そのエントリを変更後の名前に修正します。  
```javascript
// In file bot.js
// this is the LUIS service type entry in the .bot file.
const DISPATCH_CONFIG = 'nlp-with-dispatchDispatch';

// In file homeAutomation.js
// this is the LUIS service type entry in the .bot file.
const LUIS_CONFIGURATION = 'Home Automation';

// In file qna.js
// Name of the QnA Maker service in the .bot file.
const QNA_CONFIGURATION = 'sample-qna';

// In file weather.js
// this is the LUIS service type entry in the .bot file.
const WEATHER_LUIS_CONFIGURATION = 'Weather';
```

**bot.js** では、構成ファイル _nlp-with-dispatch.bot_ に含まれている情報を使用して、ディスパッチ ボットをさまざまなサービスに接続しています。 各コンストラクターでは、前述のセクション名に基づいて構成ファイルの適切なセクションが検索され、使用されます。

```javascript
class DispatchBot {
    constructor(conversationState, userState, botConfig) {
        //...
        this.homeAutomationDialog = new HomeAutomation(conversationState, userState, botConfig);
        this.weatherDialog = new Weather(botConfig);
        this.qnaDialog = new QnA(botConfig);

        this.conversationState = conversationState;
        this.userState = userState;

        // dispatch recognizer
        const dispatchConfig = botConfig.findServiceByNameOrId(DISPATCH_CONFIG);
        //...
```
---

### <a name="calling-the-services-from-your-bot"></a>ボットからのサービスの呼び出し

ボットのロジックにより、結合された Dispatch モデルに対してユーザー入力が確認されます。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

**NlpDispatchBot.cs** ファイルでは、ボットのコンストラクターによって、起動時に登録された `BotServices` オブジェクトが取得されます。

```csharp
private readonly BotServices _services;

public NlpDispatchBot(BotServices services)
{
    _services = services ?? throw new System.ArgumentNullException(nameof(services));

    //...
}
```

ボットの `OnTurnAsync` メソッドでは、ユーザーから受信したメッセージを Dispatch モデルに対して確認します。

```csharp
// Get the intent recognition result
var recognizerResult = await _services.LuisServices[DispatchKey].RecognizeAsync(context, cancellationToken);
var topIntent = recognizerResult?.GetTopScoringIntent();

if (topIntent == null)
{
    await context.SendActivityAsync("Unable to get the top intent.");
}
else
{
    await DispatchToTopIntentAsync(context, topIntent, cancellationToken);
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**bot.js** の `onTurn` メソッドでは、ユーザーから受信したメッセージが確認されます。 種類 _ActivityType.Message_ を受信した場合、このメッセージはボットの _dispatchRecognizer_ を介して送信されます。

```javascript
if (turnContext.activity.type === ActivityTypes.Message) {
    // determine which dialog should fulfill this request
    // call the dispatch LUIS model to get results.
    const dispatchResults = await this.dispatchRecognizer.recognize(turnContext);
    const dispatchTopIntent = LuisRecognizer.topIntent(dispatchResults);
    //...
 }
```
---

### <a name="working-with-the-recognition-results"></a>認識結果の操作

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

モデルによって結果が生成されるとき、どのサービスが発話を最も適切に処理できるかが示されます。 このボットのコードでは、要求を対応するサービスにルーティングし、呼び出されたサービスからの応答を要約します。

```csharp
// Depending on the intent from Dispatch, routes to the right LUIS model or QnA service.
private async Task DispatchToTopIntentAsync(
    ITurnContext context,
    (string intent, double score)? topIntent,
    CancellationToken cancellationToken = default(CancellationToken))
{
    const string homeAutomationDispatchKey = "l_Home_Automation";
    const string weatherDispatchKey = "l_Weather";
    const string noneDispatchKey = "None";
    const string qnaDispatchKey = "q_sample-qna";

    switch (topIntent.Value.intent)
    {
        case homeAutomationDispatchKey:
            await DispatchToLuisModelAsync(context, HomeAutomationLuisKey);

            // Here, you can add code for calling the hypothetical home automation service, passing in any entity
            // information that you need.
            break;
        case weatherDispatchKey:
            await DispatchToLuisModelAsync(context, WeatherLuisKey);

            // Here, you can add code for calling the hypothetical weather service,
            // passing in any entity information that you need
            break;
        case noneDispatchKey:
            // You can provide logic here to handle the known None intent (none of the above).
            // In this example we fall through to the QnA intent.
        case qnaDispatchKey:
            await DispatchToQnAMakerAsync(context, QnAMakerKey);
            break;

        default:
            // The intent didn't match any case, so just display the recognition results.
            await context.SendActivityAsync($"Dispatch intent: {topIntent.Value.intent} ({topIntent.Value.score}).");
            break;
    }
}

// Dispatches the turn to the request QnAMaker app.
private async Task DispatchToQnAMakerAsync(
    ITurnContext context,
    string appName,
    CancellationToken cancellationToken = default(CancellationToken))
{
    if (!string.IsNullOrEmpty(context.Activity.Text))
    {
        var results = await _services.QnAServices[appName].GetAnswersAsync(context).ConfigureAwait(false);
        if (results.Any())
        {
            await context.SendActivityAsync(results.First().Answer, cancellationToken: cancellationToken);
        }
        else
        {
            await context.SendActivityAsync($"Couldn't find an answer in the {appName}.");
        }
    }
}


// Dispatches the turn to the requested LUIS model.
private async Task DispatchToLuisModelAsync(
    ITurnContext context,
    string appName,
    CancellationToken cancellationToken = default(CancellationToken))
{
    await context.SendActivityAsync($"Sending your request to the {appName} system ...");
    var result = await _services.LuisServices[appName].RecognizeAsync(context, cancellationToken);

    await context.SendActivityAsync($"Intents detected by the {appName} app:\n\n{string.Join("\n\n", result.Intents)}");

    if (result.Entities.Count > 0)
    {
        await context.SendActivityAsync($"The following entities were found in the message:\n\n{string.Join("\n\n", result.Entities)}");
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

モデルによって結果が生成されるとき、どのサービスが発話を最も適切に処理できるかが示されます。 このボットのコードによって、要求は対応するサービスにルーティングされます。

```javascript
switch (dispatchTopIntent) {
   case HOME_AUTOMATION_INTENT:
      await this.homeAutomationDialog.onTurn(turnContext);
      break;
   case WEATHER_INTENT:
      await this.weatherDialog.onTurn(turnContext);
      break;
   case QNA_INTENT:
      await this.qnaDialog.onTurn(turnContext);
      break;
   case NONE_INTENT:
      default:
      // Unknown request
       await turnContext.sendActivity(`I do not understand that.`);
       await turnContext.sendActivity(`I can help with weather forecast, turning devices on and off and answer general questions like 'hi', 'who are you' etc.`);
 }
 
 // In homeAutomation.js
 async onTurn(turnContext) {
    // make call to LUIS recognizer to get home automation intent + entities
    const homeAutoResults = await this.luisRecognizer.recognize(turnContext);
    const topHomeAutoIntent = LuisRecognizer.topIntent(homeAutoResults);
    // depending on intent, call turn on or turn off or return unknown
    switch (topHomeAutoIntent) {
       case HOME_AUTOMATION_INTENT:
          await this.handleDeviceUpdate(homeAutoResults, turnContext);
          break;
       case NONE_INTENT:
       default:
         await turnContext.sendActivity(`HomeAutomation dialog cannot fulfill this request.`);
    }
}
    
// In weather.js
async onTurn(turnContext) {
   // Call weather LUIS model.
   const weatherResults = await this.luisRecognizer.recognize(turnContext);
   const topWeatherIntent = LuisRecognizer.topIntent(weatherResults);
   // Get location entity if available.
   const locationEntity = (LOCATION_ENTITY in weatherResults.entities) ? weatherResults.entities[LOCATION_ENTITY][0] : undefined;
   const locationPatternAnyEntity = (LOCATION_PATTERNANY_ENTITY in weatherResults.entities) ? weatherResults.entities[LOCATION_PATTERNANY_ENTITY][0] : undefined;
   // Depending on intent, call "Turn On" or "Turn Off" or return unknown.
   switch (topWeatherIntent) {
      case GET_CONDITION_INTENT:
         await turnContext.sendActivity(`You asked for current weather condition in Location = ` + (locationEntity || locationPatternAnyEntity));
         break;
      case GET_FORECAST_INTENT:
         await turnContext.sendActivity(`You asked for weather forecast in Location = ` + (locationEntity || locationPatternAnyEntity));
         break;
      case NONE_INTENT:
      default:
         wait turnContext.sendActivity(`Weather dialog cannot fulfill this request.`);
   }
}
    
// In qna.js
async onTurn(turnContext) {
   // Call QnA Maker and get results.
   const qnaResult = await this.qnaRecognizer.generateAnswer(turnContext.activity.text, QNA_TOP_N, QNA_CONFIDENCE_THRESHOLD);
   if (!qnaResult || qnaResult.length === 0 || !qnaResult[0].answer) {
       await turnContext.sendActivity(`No answer found in QnA Maker KB.`);
       return;
    }
    // respond with qna result
    await turnContext.sendActivity(qnaResult[0].answer);
}
```
---

## <a name="edit-intents-to-improve-performance"></a>意図を編集してパフォーマンスを向上する

ボットが起動したら、類似の、または重複する発話を削除することで、ボットのパフォーマンスを向上することができます。 たとえば、`Home Automation` LUIS アプリで、"ライトを点けて" のような要求は "TurnOnLights" の意図にマップされますが、"なぜライトが点かない?" のような要求は、 QnA Maker に渡すことができるように "None" の意図にマップされます。 ディスパッチを使用して LUIS アプリと QnA Maker サービスを結合するときは、次のいずれかを実行する必要があります。

* 元の `Home Automation` LUIS アプリから "None" の意図を削除し、代わりにその意図の発話をディスパッチャー アプリの "None" の意図に追加します。
* 元の LUIS アプリから "None" の意図を削除しない場合、代わりに "None" の意図に一致するメッセージを QnA Maker サービスに渡すためのロジックをボットで追加する必要があります。

上記の 2 つのアクションのどちらでも、ボットからユーザーに '回答が見つかりません' というメッセージが返される回数を減らすことができます。 

## <a name="additional-resources"></a>その他のリソース

**LUIS モデルを更新するか、新しく作成する:** このサンプルは、事前構成済みの LUIS モデルが基になっています。 このモデルの更新または新しい LUIS モデルの作成に役立つ追加情報については、[こちら](https://aka.ms/create-luis-model#updating-your-cognitive-models
)をご覧ください。

**リソースを削除する:** このサンプルでは、次の手順を使用して削除できるアプリケーションとリソースが多数作成されますが、"*他のアプリやサービス*" が依存しているリソースは削除しないでください。 

"_LUIS のリソース_"
1. [luis.ai](https://www.luis.ai) ポータルにサインインします。
1. _[マイ アプリ]_ ページに移動します。
1. このサンプルによって作成されたアプリを選択します。
   * `Home Automation`
   * `Weather`
   * `NLP-With-Dispatch-BotDispatch`
1. _[削除]_ をクリックし、_[OK]_ をクリックして確認します。

"_QnA Maker のリソース_"
1. [qnamaker.ai](https://www.qnamaker.ai/) ポータルにサインインします。
1. _[My knowledge bases]\(マイ ナレッジ ベース\)_ ページに移動します。
1. `Sample QnA` ナレッジ ベースの削除ボタンをクリックし、_[削除]_ をクリックして確認します。

**ベスト プラクティス:** このサンプルで使用されているサービスを向上させるには、[LUIS](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/luis-concept-best-practices) および [QnA Maker](https://docs.microsoft.com/en-us/azure/cognitive-services/qnamaker/concepts/best-practices) 向けのベスト プラクティスを参照してください。