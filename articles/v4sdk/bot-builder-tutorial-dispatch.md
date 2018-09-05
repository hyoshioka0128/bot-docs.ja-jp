---
title: ディスパッチ ツールを使用して複数の LUIS と QnA サービスを統合する | Microsoft Docs
description: ボットで LUIS や QnA Maker を使用する方法について説明します。
keywords: Luis, QnA, ディスパッチ ツール, 複数のサービス
author: DeniseMak
ms.author: v-demak
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 08/27/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 02dffcdd8cb4fd27f59fa8a763d7f2027aa0bcf2
ms.sourcegitcommit: 0b2be801e55f6baa048b49c7211944480e83ba95
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 08/28/2018
ms.locfileid: "43115037"
---
# <a name="integrate-multiple-luis-and-qna-services-with-the-dispatch-tool"></a>ディスパッチ ツールを使用して複数の LUIS と QnA サービスを統合する

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

このチュートリアルでは、ディスパッチ ツールによって生成された LUIS モデルを使用して、ボットを複数の Language Understanding (LUIS) アプリおよび QnAMaker サービスと統合する方法について説明します。 

次のサービスが開発済みであり、これらすべてのサービスと統合するボットを作成すると仮定します。

| サービスの種類 | Name | 説明 |
|------|------|------|
| LUIS アプリ | HomeAutomation | HomeAutomation.TurnOn、HomeAutomation.TurnOff、および HomeAutomation.None の各意図を認識します。|
| LUIS アプリ | Weather | Weather.GetForecast および Weather.GetCondition の各意図を認識します。|
| QnAMaker サービス | FAQ  | ホーム オートメーション照明システムに関する質問への回答を提供します |

まず、アプリとサービスを作成し、ディスパッチ ツールを使用して、それらを統合しましょう。

## <a name="create-the-luis-apps"></a>LUIS アプリの作成

LUIS アプリ *HomeAutomation* および *Weather* を作成する最も速い方法は、[homeautomation.json][HomeAutomationJSON] および [weather.json][WeatherJSON] ファイルをダウンロードすることです。 その後、次の手順に従って、その 2 つの LUIS アプリをインポートします。

1. [LUIS Web サイト](https://www.luis.ai/home)に移動してサインインします。 
2. **[マイ アプリ]** ページで、**[Import new app]\(新しいアプリのインポート\)** をクリックします。
   1. *homeautomation* アプリについては、**homeautomation.json** ファイルを選択し、"homeautomation" という名前を付けます。 **[完了]** をクリックして、アプリをインポートします。 アプリがインポートされたら、**[発行]** をクリックして、アプリを発行します。
   1. *weather* アプリについては、**weather.json** ファイルを選択し、"weather" という名前を付けます。 **[完了]** をクリックして、アプリをインポートします。 アプリがインポートされたら、**[発行]** をクリックして、アプリを発行します。

## <a name="create-the-qna-cognitive-service-in-azure"></a>Azure での QnA Cognitive Service の作成

QnA Maker のサービスは、Azure の Cognitive Service と、Cognitive Service を使用して発行する Q&A ペアのナレッジ ベースの 2 つの部分で構成されます。 ご自身のナレッジベースを作成するとき、この手順で作成した **Azure サービス名**を選択することで、それを Azure サービスにリンクできます。

Azure で Cognitive Service を作成するには、[Azure portal](https://portal.azure.com) にログインして、次の手順を実行します。

1. **[すべてのサービス]** をクリックします。
1. `Cognitive` を検索して **Cognitive Services** を選択します。
1. **[追加]** をクリックします。
1. `QnA` を検索して **[QnA Maker]** を選択します。
1. [QnA Maker] ブレードで **[作成]** をクリックします。
1. 情報を入力して QnA Maker サービスを作成します。

    <!-- TODO: Add screenshot.-->

    * サービスの名前を入力します。 このチュートリアルでは、`SmartLightQnA` を使用します。
    * 使用するサブスクリプションを選択します。
    * 使用する管理価格レベルを選択します。F0 (無料) レベルを使用します。
    * 使用する 1 つのリソース グループを作成するか、または新しいリソース グループを選択します。 このチュートリアルでは、新しい `SmartLightQnA` リソース グループを作成します。
    * 検索価格レベルを選択します。B (基本) レベルを使用します。
    * 検索場所を選択します。`West US` を使用します。
    * 使用するアプリケーションの名前を入力します。既定の `SmartLightQnA` のままにします。
    * Web サイトの場所を選択します。`West US` を使用します。
    * App Insights を有効にする既定の設定のままにします。
    * App Insights の場所を選択します。`West US` を使用します。
    * **[作成]** をクリックして QnA Maker サービスを作成します。
    * Azure でサービスが作成され、デプロイが開始します。

1. サービスがデプロイされたら、通知を表示して **[リソースに移動]** をクリックし、サービスのブレードに移動します。
1. **[キー]** をクリックしてキーを取得します。

    * サービスの名前と最初のキーをコピーします。 この名前は、サービスが、お使いのナレッジ ベースにリンクされるように、KB を作成する際に使用する必要があります。 また、以下のディスパッチャーの手順の YOUR-AZURE-QNA-SUBSCRIPTION-KEY の代わりに、このキーを使用します。
    * サービスを中断しなくても一度に 1 つのキーを再生成できるよう、2 つのキーを取得します。

## <a name="create-and-publish-the-qna-maker-knowledge-base"></a>QnA Maker ナレッジ ベースの作成と発行

KB を作成するには、以下の手順を実行します。

1. [QnA Maker Web サイト](https://qnamaker.ai)にアクセスしてサインインします。 
1. **[ナレッジ ベースの作成]** をクリックし、新しいナレッジ ベースを作成します。 Cognitive Service は既に Azure に作成済みのため、**手順 1.** はスキップできます。
1. **手順 2.** では、**[Azure QnA service]\(Azure QnA サービス\)** フィールドに対して、Azure に作成した Cognitive Service 名を選択します。 これにより、この KB が、Azure で作成したサービスに関連付けられます。
1. **手順 3.** では、この KB に "FAQ" という名前を付けます。 
1. **手順 4.** では、KB にこの [サンプル TSV ファイル][FAQ_TSV]を設定します。 または、Web ページのコンテンツを使用します。 その場合は、リンクを **URL** フィールドに貼り付けます。
1. **手順 5.** では、**[Create your KB]\(KB の作成\)** をクリックして、KB を作成します。

KB が作成されたら、その KB を**発行**して、KB ID とエンドポイントを取得する必要があります。 これらは、このプロセスの後半で必要になります。


## <a name="use-the-dispatch-tool-to-create-the-dispatcher-luis-app"></a>ディスパッチ ツールを使用してディスパッチャー LUIS アプリを作成する
これで 2 つの LUIS アプリと 1 つの KB が作成されました。次は、ディスパッチ ツールを使用して、これらを結合して 1 つの LUIS アプリを生成しましょう。 

### <a name="step-1-install-the-dispatch-tool"></a>手順 1: ディスパッチ ツールをインストールする

**コマンド プロンプト** ウィンドウを開き、ディスパッチャー プロジェクトに移動します。 次に、以下の NPM コマンドを実行して、[ディスパッチ ツール][DispatchTool]をインストールします。

```cmd
npm install -g botdispatch
```

### <a name="step-2-initialize-the-dispatch-tool"></a>手順 2: ディスパッチ ツールを初期化する

次のコマンドを実行して、`CombineWeatherAndLights` という名前でディスパッチ ツールを初期化します。 以下のコマンド ラインの `"YOUR-LUIS-AUTHORING-KEY"` の部分はご自身の [LUIS オーサリング キー](https://docs.microsoft.com/en-us/azure/cognitive-services/LUIS/luis-concept-keys)で置き換えます。

```cmd
dispatch init -name CombineWeatherAndLights -luisAuthoringKey "YOUR-LUIS-AUTHORING-KEY" -luisAuthoringRegion westus
```

### <a name="step-3-add-apps-to-dispatcher"></a>手順 3: アプリをディスパッチャーに追加する

作成した LUIS アプリごとに、LUIS アプリ ID を取得します。 各アプリの ID は [LUIS サイト](https://www.luis.ai/home)の **[My Apps]\(マイ アプリ\)** にあります。アプリ名をクリックし、**[設定]** をクリックして**アプリケーション ID** を表示します。 **手順 2.** で取得した "*LUIS オーサリング キー*" と同じものを使用します。

作成した LUIS アプリごとに次のコマンドを実行します (例: **homeautomation** および **weather**)。 以下に示す該当するコマンドの該当する ID を置き換えてください。

```
dispatch add -type luis -id "HOMEAUTOMATION-APP-ID" -name homeautomation -version 0.1 -key "YOUR-LUIS-AUTHORING-KEY"
dispatch add -type luis -id "WEATHER-APP-ID" -name weather -version 0.1 -key "YOUR-LUIS-AUTHORING-KEY"

```

次に、以下のコマンドを実行して、QnAMaker KB を追加します。 このコマンドで、`QNA-KB-ID` は、KB を発行した後に提供された KB ID を指します。また、`YOUR-AZURE-QNA-SUBSCRIPTION-KEY` は、Azure で作成した Cognitive Service から取得したキーを指します。

```
dispatch add -type qna -id "QNA-KB-ID" -name faq -key "YOUR-AZURE-QNA-SUBSCRIPTION-KEY"
```

### <a name="step-4-create-the-dispatcher-luis-app"></a>手順 4: ディスパッチャー LUIS アプリを作成する

すべてのアプリがディスパッチ ツールに追加されたら、次のコマンドを実行して、ディスパッチ アプリを作成します。 ディスパッチ ファイルを調べる必要がある場合は、拡張子が **.dispatch** のファイルに情報があります。

```
dispatch create
```

このコマンドにより、**CombineWeatherAndLights** という名前のディスパッチャー LUIS アプリが作成されます。 [https://www.luis.ai/home](https://www.luis.ai/home) で新しいアプリを確認できます。 

![LUIS.ai でのディスパッチャー アプリ](media/tutorial-dispatch/dispatch-app-in-luis.png)

新しいアプリをクリックします。 **[意図]** に `l_homeautomation`、`l_weather`、`q_faq` の各意図があることが分かります。

![LUIS.ai でのディスパッチャーの意図](media/tutorial-dispatch/dispatch-intents-in-luis.png)

**[Train]\(トレーニング\)** ボタンをクリックして LUIS アプリをトレーニングし、**[PUBLISH]\(発行\)** タブを使用して[発行](https://docs.microsoft.com/en-us/azure/cognitive-services/LUIS/publishapp)します。 **[設定]** をクリックして、新しいアプリの ID をコピーします。 この ID は、ボットがこのアプリに接続する際に必要になります。

## <a name="create-the-bot"></a>ボットの作成

ここで、ディスパッチャー アプリの意図を、元の LUIS アプリと QnAMaker サービスにメッセージをルーティングするボットのロジックにフックできます。

Bot Builder SDK に含まれているサンプルを出発点として使用できます。

# <a name="ctabcsaddref"></a>[C#](#tab/csaddref)

[LUIS Dispatch サンプル][DispatchBotCS]のコードから始めます。 Visual Studio で、次に示す最新のプレリリース バージョンに [Nuget パッケージを更新](https://docs.microsoft.com/en-us/nuget/tools/package-manager-ui#updating-a-package)します。

* `Microsoft.Bot.Builder.Integration.AspNet.Core`
* `Microsoft.Bot.Builder.Ai.QnA` (QnA Maker に必要)
* `Microsoft.Bot.Builder.Ai.Luis` (LUIS に必要)

# <a name="javascripttabjsaddref"></a>[JavaScript](#tab/jsaddref)

[LUIS Dispatch サンプル][DispatchBotJs]をダウンロードします。  npm を使用して、LUIS および QnA Maker 用の `botbuilder-ai` パッケージなど、必要なパッケージをインストールします。

* `npm install --save botbuilder@preview`
* `npm install --save botbuilder-ai@preview`

---


ディスパッチャー アプリを使用するためにサンプルを設定します。

# <a name="ctabcsbotconfig"></a>[C#](#tab/csbotconfig)

[LUIS Dispatch サンプル][DispatchBotCS]の **appsettings.json** で、次のフィールドを編集します。

| Name | 説明 |
|------|------|
| `Luis-SubscriptionKey` |  LUIS サブスクリプション キー。 [こちら](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/luis-concept-keys)で説明するエンドポイント キーまたはオーサリング キーに設定できます。 | 
| `Luis-ModelId-Dispatcher` | ディスパッチ ツールが生成する LUIS アプリのアプリ ID。 | 
| `Luis-ModelId-HomeAutomation` | homeautomation.json から作成したアプリのアプリ ID  | 
| `Luis-ModelId-Weather` | weather.json から作成したアプリのアプリ ID | 
| `QnAMaker-Endpoint-Url` | プレビューの QnA Maker サービスでは、これを https://westus.api.cognitive.microsoft.com/qnamaker/v2.0 に設定してください。 <br/>新しい (GA) QnA Maker サービスでは、これを https://YOUR-QNA-SERVICE-NAME.azurewebsites.net/qnamaker に設定します。|
| `QnAMaker-SubscriptionKey` | ご自身の Azure QnA Maker サブスクリプション キー。 | 
| `QnAMaker-KnowledgeBaseId` | [QnAMaker ポータル](https://qnamaker.ai)で作成するナレッジ ベースの ID。| 



**Startup.cs** の `ConfigureServices` メソッドをご覧ください。 生成したばかりの LUIS アプリのアプリ ID を使用して `LuisRecognizerMiddleware` を初期化するコードが含まれています。

```csharp
// This method gets called by the runtime. Use this method to add services to the container.
public void ConfigureServices(IServiceCollection services)
{
    services.AddSingleton(this.Configuration);
    services.AddBot<LuisDispatchBot>(options =>
    {
        options.CredentialProvider = new ConfigurationCredentialProvider(Configuration);

        var (luisModelId, luisSubscriptionKey, luisUri) = GetLuisConfiguration(this.Configuration, "Dispatcher");

        var luisModel = new LuisModel(luisModelId, luisSubscriptionKey, luisUri);

        // If you want to get all the intents and scores that LUIS recognized, set Verbose = true in luisOptions
        var luisOptions = new LuisRequest { Verbose = true };

        var middleware = options.Middleware;
        middleware.Add(new LuisRecognizerMiddleware(luisModel, luisOptions: luisOptions));
    });
}
```
`GetLuisConfiguration` および `GetQnAMakerConfiguration` メソッドは、LUIS と QnA の構成を appsettings.json から取得します。
```csharp
public static (string modelId, string subscriptionId, Uri uri) GetLuisConfiguration(IConfiguration configuration, string serviceName)
{
    var modelId = configuration.GetSection($"Luis-ModelId-{serviceName}")?.Value;
    var subscriptionId = configuration.GetSection("Luis-SubscriptionKey")?.Value;
    var uri = new Uri(configuration.GetSection("Luis-Url")?.Value);
    return (modelId, subscriptionId, uri);
}

public static (string knowledgeBaseId, string subscriptionKey, string uri) GetQnAMakerConfiguration(IConfiguration configuration)
{
    var knowledgeBaseId = configuration.GetSection("QnAMaker-KnowledgeBaseId")?.Value;
    var subscriptionKey = configuration.GetSection("QnAMaker-SubscriptionKey")?.Value;
    var uri = configuration.GetSection("QnAMaker-Endpoint-Url")?.Value;
    return (knowledgeBaseId, subscriptionKey, uri);
}
```

### <a name="dispatch-the-message"></a>メッセージのディスパッチ

**LuisDispatchBot.cs** をご覧ください。ここでは、サブコンポーネントの LUIS アプリまたは QnA Maker にボットがメッセージをディスパッチします。 

`DispatchToTopIntent` で、ディスパッチャー アプリが `l_homeautomation` または `l_weather` の意図を検出した場合、ディスパッチャー アプリは `DispatchToTopIntent` メソッドを呼び出します。このメソッドは `LuisRecognizer` を作成して、元の `homeautomation` および `weather` アプリを呼び出します。 ボットが `q_faq` の意図、またはフォールバック ケースとして使用される `none` の意図を検出した場合、ボットは QnAMaker をクエリするメソッドを呼び出します。

> [!NOTE] 
> 意図名 `l_homeautomation`、`l_weather` または `q_faq` が、Dispatch を使用して作成した LUIS アプリと一致しない場合、それらを編集して、[LUIS ポータル](https://www.luis.ai)で表示される小文字バージョンの意図名に合わせます。

```csharp
private async Task DispatchToTopIntent(ITurnContext context, (string intent, double score)? topIntent)
{
    switch (topIntent.Value.intent.ToLowerInvariant())
    {
        case "l_homeautomation":
            await DispatchToLuisModel(context, this.luisModelHomeAutomation, "home automation");
            break;
        case "l_weather":
            await DispatchToLuisModel(context, this.luisModelWeather, "weather");
            break;
        case "none":
        // You can provide logic here to handle the known None intent (none of the above).
        // In this example we fall through to the QnA intent.
        case "q_faq":
            await DispatchToQnAMaker(context, this.qnaEndpoint, "FAQ");
            break;
        default:
            // The intent didn't match any case, so just display the recognition results.
            await context.SendActivity($"Dispatch intent: {topIntent.Value.intent} ({topIntent.Value.score}).");

            break;
    }
}
```

この `DispatchToQnAMaker` メソッドは、ユーザーのメッセージを QnA Maker サービスに送信します。 ボットを実行する前に、[QnA Maker ポータル](https://qnamaker.ai)でそのサービスを発行済みであることを確認してください。

```csharp
private static async Task DispatchToQnAMaker(ITurnContext context, QnAMakerEndpoint qnaOptions, string appName)
{
    QnAMaker qnaMaker = new QnAMaker(qnaOptions);
    if (!string.IsNullOrEmpty(context.Activity.Text))
    {
        var results = await qnaMaker.GetAnswers(context.Activity.Text.Trim()).ConfigureAwait(false);
        if (results.Any())
        {
            await context.SendActivity(results.First().Answer);
        }
        else
        {
            await context.SendActivity($"Couldn't find an answer in the {appName}.");
        }
    }
}
```

`DispatchToLuisModel` メソッドは、ユーザーのメッセージを元の `homeautomation` および `weather` LUIS アプリに送信します。 ボットを実行する前に、[LUIS ポータル](https://www.luis.ai)でそれらの LUIS アプリを発行済みであることを確認してください。

```csharp
private static async Task DispatchToLuisModel(ITurnContext context, LuisModel luisModel, string appName)
{
    await context.SendActivity($"Sending your request to the {appName} system ...");
    var (intents, entities) = await RecognizeAsync(luisModel, context.Activity.Text);

    await context.SendActivity($"Intents detected by the {appName} app:\n\n{string.Join("\n\n", intents)}");

    if (entities.Count() > 0)
    {
        await context.SendActivity($"The following entities were found in the message:\n\n{string.Join("\n\n", entities)}");
    }
    
    // Here, you can add code for calling the hypothetical home automation or weather service, 
    // passing in the appName and any intent or entity information that you need 
}
```

`RecognizeAsync` メソッドは `LuisRecognizer` を呼び出して、LUIS アプリから結果を取得します。

```cs
private static async Task<(IEnumerable<string> intents, IEnumerable<string> entities)> RecognizeAsync(LuisModel luisModel, string text)
{
    var luisRecognizer = new LuisRecognizer(luisModel);
    var recognizerResult = await luisRecognizer.Recognize(text, System.Threading.CancellationToken.None);

    // list the intents
    var intents = new List<string>();
    foreach (var intent in recognizerResult.Intents)
    {
        intents.Add($"'{intent.Key}', score {intent.Value}");
    }

    // list the entities
    var entities = new List<string>();
    foreach (var entity in recognizerResult.Entities)
    {
        if (!entity.Key.ToString().Equals("$instance"))
        {
            entities.Add($"{entity.Key}: {entity.Value.First}");
        }
    }

    return (intents, entities);
}
```

# <a name="javascripttabjsbotconfig"></a>[JavaScript](#tab/jsbotconfig)

[Dispatch ボット サンプル][DispatchBotJs]のコードから始めます。 **app.js** を開き、必要に応じて、作成した LUIS アプリの ID で `appId` フィールドを置き換えます。 `appId` フィールドを元のままにする場合、デモ用に作成されたパブリック LUIS アプリを使用することになります。 `LUIS_SUBSCRIPTION_KEY` については、LUIS アプリの [発行] ページにあります。 そのページ上の [エンドポイント] リンク内に埋め込まれています。

```javascript
// Packages
const { BotFrameworkAdapter, MemoryStorage, ConversationState } = require('botbuilder');
const { restify } = require('restify');
const { LuisRecognizer, QnAMaker } = require('botbuilder-ai');
const { DialogSet } = require('botbuilder-dialogs');

// Create LuisRecognizers and QnAMaker
// The LUIS applications are public, meaning you can use your own subscription key to test the applications.
// For QnAMaker, users are required to create their own knowledge base.
// The exported LUIS applications and QnAMaker knowledge base can be found with the sample bot.

// The corresponding LUIS application JSON is `dispatchSample.json`
const dispatcher = new LuisRecognizer({
    appId: '0b18ab4f-5c3d-4724-8b0b-191015b48ea9',
    subscriptionKey: process.env.LUIS_SUBSCRIPTION_KEY,
    serviceEndpoint: 'https://westus.api.cognitive.microsoft.com/',
    verbose: true
});

// The corresponding LUIS application JSON is `homeautomation.json`
const homeAutomation = new LuisRecognizer({
    appId: 'c6d161a5-e3e5-4982-8726-3ecec9b4ed8d',
    subscriptionKey: process.env.LUIS_SUBSCRIPTION_KEY,
    serviceEndpoint: 'https://westus.api.cognitive.microsoft.com/',
    verbose: true
});

// The corresponding LUIS application JSON is `weather.json`
const weather = new LuisRecognizer({
    appId: '9d0c9e9d-ce04-4257-a08a-a612955f2fb5',
    subscriptionKey: process.env.LUIS_SUBSCRIPTION_KEY,
    serviceEndpoint: 'https://westus.api.cognitive.microsoft.com/',
    verbose: true
});
```

次のコードでは、`endpointKey` と `knowledgeBaseID` を、QnAMaker から取得したご自身のキーとナレッジ ベース ID に置き換えます。 `host` は、プレビューの QnA Maker サービスでは `https://westus.api.cognitive.microsoft.com/qnamaker/v2.0` に、新しい (GA) QnA Maker サービスでは `https://YOUR-QNA-SERVICE-NAME.azurewebsites.net/qnamaker` に設定してください。

```javascript
const faq = new QnAMaker(
    {
        knowledgeBaseId: '',
        endpointKey: '',
        host: ''
    },
    {
        answerBeforeNext: true
    }
);

```

**app.js** の残りのコードは、適切なダイアログを呼び出して `l_homeautomation`、`l_weather`、および `None` の各意図を処理します。 `q_faq` の意図の場合、QnAMaker サービスからの回答を返信します。

> [!NOTE] 
> 意図名 `l_homeautomation`、`l_weather` または `q_faq` が、Dispatch を使用して作成した LUIS アプリと一致しない場合、それらを編集して、[LUIS ポータル](https://www.luis.ai)で表示される意図名に合わせます。

```javascript
// create conversation state
const conversationState = new ConversationState(new MemoryStorage());
adapter.use(conversationState);

// register some dialogs for usage with the LUIS apps that are being dispatched to
const dialogs = new DialogSet();

function findEntities(entityName, entityResults) {
    let entities = []
    if (entityName in entityResults) {
        entityResults[entityName].forEach((entity, idx) => {
            entities.push(entity);
        });
    }
    return entities.length > 0 ? entities : undefined;
}

dialogs.add('HomeAutomation_TurnOff', [
    async (dialogContext, args) => {
        const devices = findEntities('HomeAutomation_Device', args.entities);
        const operations = findEntities('HomeAutomation_Operation', args.entities);

        const state = conversationState.get(dialogContext.context);
        state.homeAutomationTurnOff = state.homeAutomationTurnOff ? state.homeAutomationTurnOff + 1 : 1;
        await dialogContext.context.sendActivity(`${state.homeAutomationTurnOff}: You reached the "HomeAutomation.TurnOff" dialog.`);
        if (devices) {
            await dialogContext.context.sendActivity(`Found these "HomeAutomation_Device" entities:\n${devices.join(', ')}`);
        }
        if (operations) {
            await dialogContext.context.sendActivity(`Found these "HomeAutomation_Operation" entities:\n${operations.join(', ')}`);
        }
        await dialogContext.end();
    }
]);

dialogs.add('HomeAutomation_TurnOn', [
    async (dialogContext, args) => {
        const devices = findEntities('HomeAutomation_Device', args.entities);
        const operations = findEntities('HomeAutomation_Operation', args.entities);

        const state = conversationState.get(dialogContext.context);
        state.homeAutomationTurnOn = state.homeAutomationTurnOn ? state.homeAutomationTurnOn + 1 : 1;
        await dialogContext.context.sendActivity(`${state.homeAutomationTurnOn}: You reached the "HomeAutomation.TurnOn" dialog.`);
        if (devices) {
            await dialogContext.context.sendActivity(`Found these "HomeAutomation_Device" entities:\n${devices.join(', ')}`);
        }
        if (operations) {
            await dialogContext.context.sendActivity(`Found these "HomeAutomation_Operation" entities:\n${operations.join(', ')}`);
        }
        await dialogContext.end();
    }
]);

dialogs.add('Weather_GetForecast', [
    async (dialogContext, args) => {
        const locations = findEntities('Weather_Location', args.entities);

        const state = conversationState.get(dialogContext.context);
        state.weatherGetForecast = state.weatherGetForecast ? state.weatherGetForecast + 1 : 1;
        await dialogContext.context.sendActivity(`${state.weatherGetForecast}: You reached the "Weather.GetForecast" dialog.`);
        if (locations) {
            await dialogContext.context.sendActivity(`Found these "Weather_Location" entities:\n${locations.join(', ')}`);
        }
        await dialogContext.end();
    }
]);

dialogs.add('Weather_GetCondition', [
    async (dialogContext, args) => {
        const locations = findEntities('Weather_Location', args.entities);

        const state = conversationState.get(dialogContext.context);
        state.weatherGetCondition = state.weatherGetCondition ? state.weatherGetCondition + 1 : 1;
        await dialogContext.context.sendActivity(`${state.weatherGetCondition}: You reached the "Weather.GetCondition" dialog.`);
        if (locations) {
            await dialogContext.context.sendActivity(`Found these "Weather_Location" entities:\n${locations.join(', ')}`);
        }
        await dialogContext.end();
    }
]);

dialogs.add('None', [
    async (dialogContext) => {
        const state = conversationState.get(dialogContext.context);
        state.noneIntent = state.noneIntent ? state.noneIntent + 1 : 1;
        await dialogContext.context.sendActivity(`${state.noneIntent}: You reached the "None" dialog.`);
        await dialogContext.end();
    }
]);

adapter.use(dispatcher);

// Listen for incoming Activities
server.post('/api/messages', (req, res) => {
    adapter.processActivity(req, res, async (context) => {
        if (context.activity.type === 'message') {
            const state = conversationState.get(context);
            const dc = dialogs.createContext(context, state);

            // Retrieve the LUIS results from our dispatcher LUIS application
            const luisResults = dispatcher.get(context);

            // Extract the top intent from LUIS and use it to select which LUIS application to dispatch to
            const topIntent = LuisRecognizer.topIntent(luisResults);

            const isMessage = context.activity.type === 'message';
            if (isMessage) {
                switch (topIntent) {
                    case 'l_homeautomation':
                        const homeAutoResults = await homeAutomation.recognize(context);
                        const topHomeAutoIntent = LuisRecognizer.topIntent(homeAutoResults);
                        await dc.begin(topHomeAutoIntent, homeAutoResults);
                        break;
                    case 'l_weather':
                        const weatherResults = await weather.recognize(context);
                        const topWeatherIntent = LuisRecognizer.topIntent(weatherResults);
                        await dc.begin(topWeatherIntent, weatherResults);
                        break;
                    case 'q_faq':
                        await faq.answer(context);
                        break;
                    default:
                        await dc.begin('None');
                }
            }

            if (!context.responded) {
                await dc.continue();
                if (!context.responded && isMessage) {
                    await dc.context.sendActivity(`Hi! I'm the LUIS dispatch bot. Say something and LUIS will decide how the message should be routed.`);
                }
            }
        }
    });
});

```

---
## <a name="run-the-bot"></a>ボットの実行

[Bot Framework Emulator](../bot-service-debug-emulator.md) を使ってボットをテストします。 「ライトを点ける」のようなメッセージをボットに送信して、ホーム オートメーション LUIS アプリにメッセージをディスパッチします。また、天気 LUIS アプリにディスパッチする「シアトルの天気を取得する」のようなメッセージをボットに送信します。

> [!NOTE] 
> ボットを実行する前に、[LUIS ポータル](https://www.luis.ai)で作成したすべての LUIS アプリが発行済みであること、また [QnA Maker ポータル](https://qnamaker.ai)で QnA Maker サービスを発行済みであることを確認してください。

![ディスパッチ ボットへのメッセージの送信](media/tutorial-dispatch/run-dispatch-bot.png)

## <a name="evaluate-the-dispatchers-performance"></a>ディスパッチャーのパフォーマンスの評価

LUIS アプリと QnA Maker サービスの両方で例として提供されるユーザー メッセージがあり、Dispatch が生成する結合 LUIS アプリがこれらの入力をうまく処理できない場合があります。 `eval` オプションを使用してアプリのパフォーマンスを確認します。 

```
dispatch eval
```

`dispatch eval` を実行すると、**Summary.html** ファイルが生成されます。このファイルは、新しい言語モデルのパフォーマンスに関する統計を提供します。

> [!TIP] 
> `dispatch eval` は実際には、ディスパッチ ツールによって作成された LUIS アプリだけでなく、任意の LUIS アプリに対して実行できます。

### <a name="edit-intents-for-duplicates-and-overlaps"></a>意図の重複の編集

**Summary.html** で重複としてフラグ付けされる発話例を見直して、類似または重複した例を削除します。 たとえば、ホーム オートメーション用の `homeautomation` LUIS アプリで、「ライトを点けて」のような要求は "TurnOnLights" の意図にマップしますが、「なぜライトが点かない?」のような要求は、 QnA Maker に渡すことができるように "None" の意図にマップします。 ディスパッチを使用して LUIS アプリと QnA Maker サービスを結合するときは、次のいずれかを実行する必要があります。 

* 元の `homeautomation` LUIS アプリから "None" の意図を削除し、その意図の発話をディスパッチャー アプリの "None" の意図に追加します。
* 元の LUIS アプリから "None" の意図を削除しない場合、その意図に一致するメッセージを QnA Maker サービスに渡すためのロジックをボットで追加する必要があります。

> [!TIP] 
> 言語モデルのパフォーマンスの改善に関するヒントは、「[Language Understanding のベスト プラクティス](./bot-builder-concept-luis.md#best-practices-for-language-understanding)」を確認してください。

## <a name="additional-resources"></a>その他のリソース

* [会話フローに LUIS を使用する][luis-v4-how-to]

[luis-v4-how-to]: bot-builder-howto-v4-luis.md
<!-- links -->

[HomeAutomationJSON]: https://aka.ms/dispatch-luis1
[WeatherJSON]: https://aka.ms/dispatch-luis2
[DispatchJSON]: https://aka.ms/dispatch-luis
[FAQ_TSV]: https://aka.ms/dispatch-qna-tsv

[DispatchTool]: https://github.com/Microsoft/botbuilder-tools/tree/master/Dispatch
[DispatchBotCS]: https://aka.ms/dispatch-sample-cs
[DispatchBotJs]: https://aka.ms/dispatch-sample-js



