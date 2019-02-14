---
title: QnA Maker を使用して質問に回答する | Microsoft Docs
description: ボットで QnA Maker を使用する方法について説明します。
keywords: 質問と回答, QnA, FAQ, QnA Maker
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: cognitive-services
ms.date: 01/15/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 6722a9c49961857ab53a2d80fca926775e712aae
ms.sourcegitcommit: 7f418bed4d0d8d398f824e951ac464c7c82b8c3e
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/14/2019
ms.locfileid: "56240168"
---
# <a name="use-qna-maker-to-answer-questions"></a>QnA Maker を使用して質問に回答する

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

QnA Maker サービスを使用すると、質問と回答のサポートをボットに追加することができます。 お客様独自の QnA Maker サービスを作成するときの基本的な要件の 1 つは、質問と回答でサービスをシードすることです。 多くの場合、質問と回答は、FAQ や他のドキュメントなどのコンテンツに既に存在しています。 また、より自然な会話になるように質問に対する回答をカスタマイズしたいこともあります。

このトピックでは、ナレッジ ベースを作成し、ボットで使用します。

## <a name="prerequisites"></a>前提条件
- [QnA Maker](https://www.qnamaker.ai/) アカウント
- この記事のコードは、**QnA Maker** サンプルをベースにしています。 [C# サンプル](https://aka.ms/cs-qna)または [Javascript サンプル](https://aka.ms/js-qna-sample)のコピーが必要です。
- [Bot Framework Emulator](https://github.com/Microsoft/BotFramework-Emulator/blob/master/README.md#download)
- [ボットの基本](bot-builder-basics.md)、[QnA Maker](https://docs.microsoft.com/en-us/azure/cognitive-services/qnamaker/overview/overview)、および [.bot](bot-file-basics.md) ファイルの知識。

## <a name="create-a-qna-maker-service-and-publish-a-knowledge-base"></a>QnA Maker サービスの作成とナレッジ ベースの発行
1. まず、[QnA Maker サービス](https://docs.microsoft.com/en-us/azure/cognitive-services/qnamaker/how-to/set-up-qnamaker-service-azure)を作成する必要があります。
1. 次に、プロジェクトの CognitiveModels フォルダーにある `smartLightFAQ.tsv` ファイルを使用して、ナレッジ ベースを作成します。 ご自身の QnA Maker [ナレッジ ベース](https://docs.microsoft.com/en-us/azure/cognitive-services/qnamaker/quickstarts/create-publish-knowledge-base)を作成、トレーニング、および発行する手順は、QnA Maker のドキュメントに記載されています。 次の手順では、ご自身の KB `qna` に名前を付け、`smartLightFAQ.tsv` ファイルを使用して KB を設定します。
> 注: この記事は、お客様のユーザーが作成した QnA Maker ナレッジ ベースにアクセスする際にも使用できます。

## <a name="obtain-values-to-connect-your-bot-to-the-knowledge-base"></a>ボットをナレッジ ベースに接続するための値を取得する
1. [QnA Maker](https://www.qnamaker.ai/) サイトで、目的のナレッジ ベースを選択します。
1. ナレッジ ベースを開いた状態で **[設定]** を選択します。 "_サービス名_" に表示されている値を記録します。 この値は、QnA Maker ポータル インターフェイスを使用するときに、対象となる自分のナレッジ ベースを見つけるのに役立ちます。 この値を使用して、ボット アプリをこのナレッジ ベースに接続するわけではありません。 
1. 下へスクロールして **[デプロイの詳細]** を探し、次の値を記録します。
   - POST /knowledgebases/<Your_Knowledge_Base_Id>/getAnswers
   - Host: <Your_Hostname>/qnamaker
   - Authorization:EndpointKey <Your_Endpoint_Key>
   
この 3 つの値によって、アプリが Azure QnA サービスを介して QnA Maker ナレッジ ベースに接続する際に必要な情報が提供されます。  

## <a name="update-the-bot-file"></a>.bot ファイルを更新する
まず、ナレッジ ベースにアクセスするために必要な情報 (ホスト名、エンドポイント キー、ナレッジ ベース ID (KbId) など) を `qnamaker.bot` に追加します。 これらは、QnA Maker でナレッジ ベースの **[設定]** から保存した値です。 
> 注: 既存のボット アプリケーションに QnA Maker ナレッジ ベースへのアクセスを追加する場合は、次に示すような "type": "qna" セクションを .bot ファイルに必ず追加してください。 このセクション内の "name" 値が、アプリ内からこの情報にアクセスするために必要なキーになります。

```json
{
  "name": "qnamaker",
  "services": [
    {
      "type": "endpoint",
      "name": "development",
      "endpoint": "http://localhost:3978/api/messages",
      "appId": "",
      "appPassword": "",
      "id": "25"    
    },
    {
      "type": "qna",
      "name": "QnABot",
      "KbId": "<Your_Knowledge_Base_Id>",
      "subscriptionKey": "",
      "endpointKey": "<Your_Endpoint_Key>",
      "hostname": "<Your_Hostname>",
      "id": "117"
    }
  ],
  "padlock": "",
   "version": "2.0"
}
```

# <a name="ctabcs"></a>[C#](#tab/cs)
次に、**BotServices.cs** の BotService クラスの新しいインスタンスを初期化します。これにより、.bot ファイルから上記の情報が取得されます。 外部サービスは、BotConfiguration クラスを使用して構成されます。

```csharp
private static BotServices InitBotServices(BotConfiguration config)
{
    var qnaServices = new Dictionary<string, QnAMaker>();
    foreach (var service in config.Services)
    {
        switch (service.Type)
        {
            case ServiceTypes.QnA:
            {
                // Create a QnA Maker that is initialized and suitable for passing
                // into the IBot-derived class (QnABot).
                var qna = (QnAMakerService)service;
                if (qna == null)
                {
                    throw new InvalidOperationException("The QnA service is not configured correctly in your '.bot' file.");
                }

                if (string.IsNullOrWhiteSpace(qna.KbId))
                {
                    throw new InvalidOperationException("The QnA KnowledgeBaseId ('kbId') is required to run this sample. Please update your '.bot' file.");
                }

                if (string.IsNullOrWhiteSpace(qna.EndpointKey))
                {
                    throw new InvalidOperationException("The QnA EndpointKey ('endpointKey') is required to run this sample. Please update your '.bot' file.");
                }

                if (string.IsNullOrWhiteSpace(qna.Hostname))
                {
                    throw new InvalidOperationException("The QnA Host ('hostname') is required to run this sample. Please update your '.bot' file.");
                }

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
    var connectedServices = new BotServices(qnaServices);
    return connectedServices;
}
```

次に、**QnABot.cs** で、この QnAMaker インスタンスをボットに渡します。 お客様独自のナレッジ ベースにアクセスする場合は、以下の "_ウェルカム_" メッセージを変更して、お客様のユーザーの役に立つ初期手順を設定します。 このクラスには、静的変数 _QnAMakerKey_ も定義されています。 これは、QnA Mkaer ナレッジ ベースにアクセスするための接続情報を含む、.bot ファイル内のセクションを参照します。

```csharp
public class QnABot : IBot
{
    public static readonly string QnAMakerKey = "QnABot";
    private const string WelcomeText = @"This bot will introduce you to QnA Maker.
                                         Ask a question to get started.";
    private readonly BotServices _services;
    public QnABot(BotServices services)
    {
        _services = services ?? throw new System.ArgumentNullException(nameof(services));
        Console.WriteLine($"{_services}");
        if (!_services.QnAServices.ContainsKey(QnAMakerKey))
        {
            throw new System.ArgumentException($"Invalid configuration. Please check your '.bot' file for a QnA service named '{QnAMakerKey}'.");
        }
    }
}
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

このサンプルでは、スタートアップ コードが **index.js** ファイルにあります。ボット ロジックのコードは **bot.js** ファイルに、また、追加の構成情報は **qnamaker.bot** ファイルに含まれています。

**index.js** ファイルで、構成情報を読み取って QnA Maker サービスを生成し、ボットを初期化します。

`QNA_CONFIGURATION` の値を .bot ファイル内の "name": 値に更新します。 これは、QnA Maker ナレッジ ベースにアクセスするための接続パラメーターを含む、.bot ファイルの "type": "qna" セクションへのキーです。

```js
// Name of the QnA Maker service in the .bot file. 
const QNA_CONFIGURATION = '<BOT_FILE_NAME>';

// Get endpoint and QnA Maker configurations by service name.
const endpointConfig = botConfig.findServiceByNameOrId(BOT_CONFIGURATION);
const qnaConfig = botConfig.findServiceByNameOrId(QNA_CONFIGURATION);

// Map the contents to the required format for `QnAMaker`.
const qnaEndpointSettings = {
    knowledgeBaseId: qnaConfig.kbId,
    endpointKey: qnaConfig.endpointKey,
    host: qnaConfig.hostname
};

// Create adapter...

// Create the QnAMakerBot.
let bot;
try {
    bot = new QnAMakerBot(qnaEndpointSettings, {});
} catch (err) {
    console.error(`[botInitializationError]: ${ err }`);
    process.exit();
}
```

次に、HTTP サーバーを作成し、受信要求をリッスンして、ボット ロジックに対する呼び出しを生成します。

```js
// Create HTTP server.
let server = restify.createServer();
server.listen(process.env.port || process.env.PORT || 3978, function() {
    console.log(`\n${ server.name } listening to ${ server.url }.`);
    console.log(`\nGet Bot Framework Emulator: https://aka.ms/botframework-emulator.`);
    console.log(`\nTo talk to your bot, open qnamaker.bot file in the emulator.`);
});

// Listen for incoming requests.
server.post('/api/messages', (req, res) => {
    adapter.processActivity(req, res, async (turnContext) => {
        await bot.onTurn(turnContext);
    });
});
```

---

## <a name="calling-qna-maker-from-your-bot"></a>ボットからの QnA Maker の呼び出し

# <a name="ctabcs"></a>[C#](#tab/cs)

QnA Maker からの回答をボットが必要とする場合、ボットのコードから `GetAnswersAsync()` を呼び出して、現在のコンテキストに基づいて適切な回答を取得します。 お客様独自のナレッジ ベースにアクセスする場合は、以下の "_回答なし_" メッセージを変更して、お客様のユーザーの役に立つ手順を設定します。

```csharp
// Check QnA Maker model
var response = await _services.QnAServices[QnAMakerKey].GetAnswersAsync(turnContext);
if (response != null && response.Length > 0)
{
    await turnContext.SendActivityAsync(response[0].Answer, cancellationToken: cancellationToken);
}
else
{
    var msg = @"No QnA Maker answers were found. This example uses a QnA Maker Knowledge Base that focuses on smart light bulbs.
                To see QnA Maker in action, ask the bot questions like 'Why won't it turn on?' or 'I need help'.";
    await turnContext.SendActivityAsync(msg, cancellationToken: cancellationToken);
}

    /// ...
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

**bot.js** ファイルで、ユーザーの入力を QnA Maker サービスの `getAnswers` メソッドに渡して、ナレッジ ベースから回答を取得します。 お客様独自のナレッジ ベースにアクセスする場合は、以下の "_回答なし_" メッセージと "_ウェルカム_" メッセージを変更して、お客様のユーザーの役に立つ手順を設定します。

```javascript
const { ActivityTypes, TurnContext } = require('botbuilder');
const { QnAMaker, QnAMakerEndpoint, QnAMakerOptions } = require('botbuilder-ai');

/**
 * A simple bot that responds to utterances with answers from QnA Maker.
 * If an answer is not found for an utterance, the bot responds with help.
 */
class QnAMakerBot {
    /**
     * The QnAMakerBot constructor requires one argument (`endpoint`) which is used to create an instance of `QnAMaker`.
     * @param {QnAMakerEndpoint} endpoint The basic configuration needed to call QnA Maker. In this sample the configuration is retrieved from the .bot file.
     * @param {QnAMakerOptions} config An optional parameter that contains additional settings for configuring a `QnAMaker` when calling the service.
     */
    constructor(endpoint, qnaOptions) {
        this.qnaMaker = new QnAMaker(endpoint, qnaOptions);
    }

    /**
     * Every conversation turn for our QnAMakerBot will call this method.
     * @param {TurnContext} turnContext Contains all the data needed for processing the conversation turn.
     */
    async onTurn(turnContext) {
        // By checking the incoming Activity type, the bot only calls QnA Maker in appropriate cases.
        if (turnContext.activity.type === ActivityTypes.Message) {
            // Perform a call to the QnA Maker service to retrieve matching Question and Answer pairs.
            const qnaResults = await this.qnaMaker.getAnswers(turnContext);

            // If an answer was received from QnA Maker, send the answer back to the user.
            if (qnaResults[0]) {
                await turnContext.sendActivity(qnaResults[0].answer);

            // If no answers were returned from QnA Maker, reply with help.
            } else {
                await turnContext.sendActivity('No QnA Maker answers were found. This example uses a QnA Maker Knowledge Base that focuses on smart light bulbs. To see QnA Maker in action, ask the bot questions like "Why won\'t it turn on?" or "I need help."');
            }

        // If the Activity is a ConversationUpdate, send a greeting message to the user.
        } else if (turnContext.activity.type === ActivityTypes.ConversationUpdate &&
                   turnContext.activity.recipient.id !== turnContext.activity.membersAdded[0].id) {
            await turnContext.sendActivity('Welcome to the QnA Maker sample! Ask me a question and I will try to answer it.');

        // Respond to all other Activity types.
        } else if (turnContext.activity.type !== ActivityTypes.ConversationUpdate) {
            await turnContext.sendActivity(`[${ turnContext.activity.type }]-type activity detected.`);
        }
    }
}

module.exports.QnAMakerBot = QnAMakerBot;
```

---

## <a name="test-the-bot"></a>ボットのテスト

ご自身のマシンを使ってローカルでサンプルを実行します。 手順については、readme ファイルで [C#](https://github.com/Microsoft/BotBuilder-Samples/tree/master/samples/csharp_dotnetcore/11.qnamaker) または [JS](https://github.com/Microsoft/BotBuilder-Samples/blob/master/samples/javascript_nodejs/11.qnamaker/README.md) サンプルを参照してください。

次に示すように、エミュレーターでボットにメッセージを送信します。

![QnA サンプルをテストする](~/media/emulator-v4/qna-test-bot.png)


## <a name="next-steps"></a>次の手順

他の Cognitive Services と QnA Maker を組み合わせて、ボットをさらに強力にすることができます。 ディスパッチ ツールを使用すると、ボットで QnA と Language Understanding (LUIS) を結合できます。

> [!div class="nextstepaction"]
> [ディスパッチ ツールを使用して LUIS と QnA サービスを組み合わせる](./bot-builder-tutorial-dispatch.md)
