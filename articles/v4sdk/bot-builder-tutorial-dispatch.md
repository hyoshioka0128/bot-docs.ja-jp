---
title: ディスパッチ ツールで LUIS と QnA サービスを使用する | Microsoft Docs
description: ボットで LUIS や QnA Maker を使用する方法について説明します。
keywords: Luis, QnA, ディスパッチ ツール, 複数のサービス
author: DeniseMak
ms.author: v-demak
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 10/31/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 4d029dc7361ac8a7fadb61141faf60d8a62eab3c
ms.sourcegitcommit: a496714fb72550a743d738702f4f79e254c69d06
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/01/2018
ms.locfileid: "50736680"
---
# <a name="use-luis-and-qna-services-with-the-dispatch-tool"></a>ディスパッチ ツールで LUIS と QnA サービスを使用する

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

<!--TODO: Add the JS sections back in and update them once the JS sample is working.-->

このチュートリアルでは、ディスパッチ ツールによって生成された LUIS モデルを使用して、ボットを複数の Language Understanding (LUIS) アプリおよび QnAMaker サービスと統合する方法について説明します。 このサンプルでは、以下のサービスが組み合わされます。

| サービスの種類 | Name | 説明 |
|------|------|------|
| LUIS アプリ | HomeAutomation | エンティティ データが関連付けられているホーム オートメーションの意図を認識します。|
| LUIS アプリ | Weather | 位置データを含む Weather.GetForecast および Weather.GetCondition の各意図を認識します。|
| QnAMaker サービス | FAQ  | ボットに関するいくつかの簡単な質問への回答を提供します。 |

この記事のコードは **NLP と Dispatch** のサンプル ([C#](https://aka.ms/dispatch-sample-cs)) から取得されています。

<!-- | [JS](https://aka.ms/dispatch-sample-js)-->

言語サービスの概要については、「[Language Understanding (言語の理解)](bot-builder-concept-luis.md)」を参照してください。 これらをボットに実装する手順については、[LUIS](bot-builder-howto-v4-luis.md) と [QnA Maker](bot-builder-howto-qna.md) に関するハウツー記事を参照してください。

サンプルの README にある手順に従ってボットを設定してテストするか、[コードに関するメモ](#notes-about-the-code)にジャンプすることができます。

## <a name="create-the-services-and-test-the-bot"></a>サービスの作成とボットのテスト

サンプルの README の手順に従ってください。 CLI ツールを使用して、これらのサービスを作成および発行し、(**.bot**) 構成ファイル内のそれらのサービスに関する情報を更新します。

1. samples リポジトリを複製またはプルします。
1. BotBuilder CLI ツールをインストールします。
1. 必要なサービスを手動で構成します。

### <a name="test-your-bot"></a>ボットをテストする

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Visual Studio または Visual Studio Code を使用して、ボットを起動します。

<!--
# [JavaScript](#tab/javascript)
-->

---

Bot Framework Emulator を使用してボットに接続します。

ここで含めたサービスの対象となる入力の一部を次に示します。

* QnA Maker
  * `hi`、`good morning`
  * `what are you`、`what do you do`
* LUIS (ホーム オートメーション)
  * `turn on bedroom light`
  * `turn off bedroom light`
  * `make some coffee`
* LUIS (天気)
  * `whats the weather in chennai india`
  * `what's the forecast for bangalore`
  * `show me the forecast for nebraska`

## <a name="notes-about-the-code"></a>コードに関するメモ

### <a name="packages"></a>パッケージ

このサンプルでは、次のパッケージを使用します。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

これらの [NuGet パッケージ](https://docs.microsoft.com/en-us/nuget/tools/package-manager-ui#updating-a-package)の最新バージョン (v4)。

* `Microsoft.Bot.Builder`
* `Microsoft.Bot.Builder.AI.Luis`
* `Microsoft.Bot.Builder.AI.QnA`
* `Microsoft.Bot.Builder.Integration.AspNet.Core`
* `Microsoft.Bot.Configuration`

<!--
# [JavaScript](#tab/javascript)

Download the [LUIS Dispatch sample][DispatchBotJs].  Install the required packages, including the `botbuilder-ai` package for LUIS and QnA Maker, using npm:

* `npm install --save botbuilder`
* `npm install --save botbuilder-ai`
-->

---

### <a name="botbuilder-cli-tools"></a>BotBuilder CLI ツール

このサンプルでは、これらの [BotBuilder CLI ツール](https://aka.ms/botbuilder-tools-readme) (npm 経由で入手可能) を使用して、LUIS、QnA Maker、および Dispatch サービスを作成、トレーニング、発行し、これらのサービスに関する情報をボットの (**.bot**) 構成ファイルに記録します。

* [Dispatch](https://aka.ms/botbuilder-tools-dispatch)
* [LUDown](https://aka.ms/botbuilder-tools-ludown-readme)
* [LUIS](https://aka.ms/botbuilder-tools-luis)
* [MSBot](https://aka.ms/botbuilder-tools-msbot-readme)
* [QnAMaker](https://aka.ms/botbuilder-tools-qnaMaker)

> [!TIP]
> npm とこれらの CLI ツールの最新バージョンを使用していることを確認するには、次のコマンドを実行します。
>
> ```shell
> npm i -g npm dispatch ludown luis-apis msbot qnamaker
> ```

ツールを使用してサービスを設定すると、このサンプルの **.bot** ファイルは次のようになります  (`msbot secret -n` を実行してこのファイル内の機密の値を暗号化できます)。

```json
{
    "name": "NLP-With-Dispatch-Bot",
    "description": "",
    "services": [
        {
            "type": "endpoint",
            "name": "development",
            "id": "http://localhost:3978/api/messages",
            "appId": "",
            "appPassword": "",
            "endpoint": "http://localhost:3978/api/messages"
        },
        {
            "type": "luis",
            "name": "Home Automation",
            "appId": "<your-home-automation-luis-app-id>",
            "version": "0.1",
            "authoringKey": "<your-luis-authoring-key>",
            "subscriptionKey": "<your-cognitive-services-subscription-key>",
            "region": "westus",
            "id": "110"
        },
        {
            "type": "luis",
            "name": "Weather",
            "appId": "<your-weather-luis-app-id>",
            "version": "0.1",
            "authoringKey": "<your-luis-authoring-key>",
            "subscriptionKey": "<your-cognitive-services-subscription-key>",
            "region": "westus",
            "id": "92"
        },
        {
            "type": "qna",
            "name": "Sample QnA",
            "kbId": "<your-qna-knowledge-base-id>",
            "subscriptionKey": "<your-cognitive-services-subscription-key>",
            "endpointKey": "<your-qna-endpoint-key>",
            "hostname": "<your-qna-host-name>",
            "id": "184"
        },
        {
            "type": "dispatch",
            "name": "NLP-With-Dispatch-BotDispatch",
            "appId": "<your-dispatch-app-id>",
            "authoringKey": "<your-luis-authoring-key>",
            "subscriptionKey": "<your-cognitive-services-subscription-key>",
            "version": "Dispatch",
            "region": "westus",
            "serviceIds": [
                "110",
                "92",
                "184"
            ],
            "id": "27"
        }
    ],
    "padlock": "",
    "version": "2.0"
}
```

### <a name="connecting-to-the-services-from-your-bot"></a>ボットからサービスへの接続

Dispatch、LUIS、および QnA Maker サービスに接続するために、ボットは **.bot** ファイルから情報をプルします。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

**Startup.cs** 内の `ConfigureServices` によって構成ファイルが読み込まれます。`InitBotServices` によって、サービスを初期化するためにその情報が使用されます。 作成されるたびに、ボットは登録された `BotServices` オブジェクトを使用して初期化されます。 これら 2 つのメソッドの関連する部分を次に示します。

```csharp
public void ConfigureServices(IServiceCollection services)
{
    //...
    var botConfig = BotConfiguration.Load(botFilePath ?? @".\BotConfiguration.bot", secretKey);
    services.AddSingleton(sp => botConfig
        ?? throw new InvalidOperationException($"The .bot config file could not be loaded. ({botConfig})"));

    // Retrieve current endpoint.
    var environment = _isProduction ? "production" : "development";
    var service = botConfig.Services.Where(s => s.Type == "endpoint" && s.Name == environment).FirstOrDefault();
    if (!(service is EndpointService endpointService))
    {
        throw new InvalidOperationException($"The .bot file does not contain an endpoint with name '{environment}'.");
    }

    var connectedServices = InitBotServices(botConfig);

    services.AddSingleton(sp => connectedServices);
    //...
}
```

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
/// <summary>
/// Depending on the intent from Dispatch, routes to the right LUIS model or QnA service.
/// </summary>
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

/// <summary>
/// Dispatches the turn to the request QnAMaker app.
/// </summary>
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

/// <summary>
/// Dispatches the turn to the requested LUIS model.
/// </summary>
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

`dispatch eval` を実行すると、**Summary.html** ファイルが生成されます。このファイルは、言語モデルの予測されるパフォーマンスに関する統計を提供します。

> [!TIP]
> `dispatch eval` は、ディスパッチ ツールによって作成された LUIS アプリだけでなく、任意の LUIS アプリに対して実行できます。

### <a name="edit-intents-for-duplicates-and-overlaps"></a>意図の重複の編集

**Summary.html** で重複としてフラグ付けされる発話例を見直して、類似または重複した例を削除します。 たとえば、`Home Automation` LUIS アプリで、"ライトを点けて" のような要求は "TurnOnLights" の意図にマップされますが、"なぜライトが点かない?" のような要求は、 QnA Maker に渡すことができるように "None" の意図にマップされます。 ディスパッチを使用して LUIS アプリと QnA Maker サービスを結合するときは、次のいずれかを実行する必要があります。

* 元の `Home Automation` LUIS アプリから "None" の意図を削除し、その意図の発話をディスパッチャー アプリの "None" の意図に追加します。
* 元の LUIS アプリから "None" の意図を削除しない場合、その意図に一致するメッセージを QnA Maker サービスに渡すためのロジックをボットで追加する必要があります。

> [!TIP]
> 言語モデルのパフォーマンスの改善に関するヒントは、「[Language Understanding のベスト プラクティス](./bot-builder-concept-luis.md#best-practices-for-language-understanding)」を確認してください。

## <a name="to-clean-up-resources-from-this-sample"></a>このサンプルからリソースをクリーンアップする方法

このサンプルでは、複数のアプリケーションとリソースが作成されます。 次の手順に従って、それらを削除することができます。

### <a name="luis-resources"></a>LUIS のリソース

1. [luis.ai](https://www.luis.ai) ポータルにサインインします。
1. **[マイ アプリ]** ページに移動します。
1. このサンプルによって作成されたアプリを選択します。
   * `Home Automation`
   * `Weather`
   * `NLP-With-Dispatch-BotDispatch`
1. **[削除]** をクリックし、**[OK]** をクリックして確認します。

### <a name="qna-maker-resources"></a>QnA Maker のリソース

1. [qnamaker.ai](https://www.qnamaker.ai/) ポータルにサインインします。
1. **[My knowledge bases]\(マイ ナレッジ ベース\)** ページに移動します。
1. `Sample QnA` ナレッジ ベースの削除ボタンをクリックし、**[削除]** をクリックして確認します。

### <a name="azure-resources"></a>Azure リソース

> [!WARNING]
> これらのリソースのうち、他のアプリやサービスが依存しているものは、いずれも削除しないでください。

1. [Azure Portal](https://portal.azure.com/) にサインインします。
1. サンプル用に作成した Cognitive Services リソースの **[概要]** ページに移動します。
1. **[削除]** をクリックし、**[はい]** をクリックして確認します。

## <a name="additional-resources"></a>その他のリソース

* [言語の理解](bot-builder-concept-luis.md)
* [ナレッジ ボットを設計する](../bot-service-design-pattern-knowledge-base.md)
* [音声プライミングを構成する](../bot-service-manage-speech-priming.md)
* [言語理解のための LUIS の使用](bot-builder-howto-v4-luis.md)
* [QnA Maker を使用して質問に回答する](bot-builder-howto-qna.md)
