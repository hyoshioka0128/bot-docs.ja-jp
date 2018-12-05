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
ms.date: 11/26/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 9b0ddf5cf8af61048ba78f10824c9573da82fc08
ms.sourcegitcommit: a722f960cd0a8513d46062439eb04de3a0275346
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/27/2018
ms.locfileid: "52336272"
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

**README** に記載された [C#](https://github.com/Microsoft/BotBuilder-Samples/blob/master/samples/csharp_dotnetcore/14.nlp-with-dispatch/README.md) または [JS](https://github.com/Microsoft/BotBuilder-Samples/blob/master/samples/javascript_nodejs/14.nlp-with-dispatch/README.md) に関する手順に従って、エミュレーターを使用してサンプルをビルドおよび実行します。 

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

<!--
# [JavaScript](#tab/javascript)

```javascript
```
-->

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

<!--
# [JavaScript](#tab/javascript)

```javascript
```
-->

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

<!--
# [JavaScript](#tab/javascript)

```javascript
```
-->

---

## <a name="evaluate-the-dispatchers-performance"></a>ディスパッチャーのパフォーマンスの評価

LUIS アプリと QnA Maker サービスの両方で例として提供されるユーザー メッセージがあり、Dispatch が生成する結合 LUIS アプリがこれらの入力をうまく処理できない場合があります。 `eval` オプションを使用すると、アプリのパフォーマンスを確認できます。

```shell
dispatch eval
```

`dispatch eval` を実行すると、**Summary.html** ファイルが生成されます。このファイルは、言語モデルの予測されるパフォーマンスに関する統計を提供します。 `dispatch eval` は、ディスパッチ ツールによって作成された LUIS アプリだけでなく、任意の LUIS アプリに対して実行できます。

### <a name="edit-intents-for-duplicates-and-overlaps"></a>意図の重複の編集

**Summary.html** で重複としてフラグ付けされる発話例を見直して、類似または重複した例を削除します。 たとえば、`Home Automation` LUIS アプリで、"ライトを点けて" のような要求は "TurnOnLights" の意図にマップされますが、"なぜライトが点かない?" のような要求は、 QnA Maker に渡すことができるように "None" の意図にマップされます。 ディスパッチを使用して LUIS アプリと QnA Maker サービスを結合するときは、次のいずれかを実行する必要があります。

* 元の `Home Automation` LUIS アプリから "None" の意図を削除し、その意図の発話をディスパッチャー アプリの "None" の意図に追加します。
* 元の LUIS アプリから "None" の意図を削除しない場合、その意図に一致するメッセージを QnA Maker サービスに渡すためのロジックをボットで追加する必要があります。


## <a name="additional-resources"></a>その他のリソース 

**リソースの削除:** このサンプルでは、次の手順を使用して削除できるアプリケーションとリソースが多数作成されますが、"*他のアプリやサービス*" が依存しているリソースは削除しないでください。 

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
