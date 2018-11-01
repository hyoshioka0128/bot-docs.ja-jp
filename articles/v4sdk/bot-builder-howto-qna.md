---
title: QnA Maker を使用して質問に回答する | Microsoft Docs
description: ボットで QnA Maker を使用する方法について説明します。
keywords: 質問と回答, QnA, FAQ, ミドルウェア
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: cognitive-services
ms.date: 10/08/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 4558a90b7d205d416657450224e2ab4892586b25
ms.sourcegitcommit: 6ed90a4c90add925a0a865be1127041b7775fd3d
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/30/2018
ms.locfileid: "50234465"
---
# <a name="use-qna-maker-to-answer-questions"></a>QnA Maker を使用して質問に回答する

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

QnA Maker サービスを使用すると、質問と回答のサポートをボットに追加することができます。 お客様独自の QnA Maker サービスを作成するときの基本的な要件の 1 つは、質問と回答でサービスをシードすることです。 多くの場合、質問と回答は、FAQ や他のドキュメントなどのコンテンツに既に存在しています。 また、より自然な会話になるように質問に対する回答をカスタマイズしたいこともあります。

## <a name="prerequisites"></a>前提条件
- [QnA Maker](https://www.qnamaker.ai/) アカウントを作成する
- QnA Maker サンプルをダウンロードする ([C#](https://aka.ms/cs-qna) | [JavaScript](https://aka.ms/js-qna-sample))

## <a name="create-a-qna-maker-service-and-publish-a-knowledge-base"></a>QnA Maker サービスの作成とナレッジ ベースの発行

QnA Maker アカウントの作成後、[QnA Maker サービス](https://docs.microsoft.com/en-us/azure/cognitive-services/qnamaker/how-to/set-up-qnamaker-service-azure)と[ナレッジ ベース](https://docs.microsoft.com/en-us/azure/cognitive-services/qnamaker/quickstarts/create-publish-knowledge-base)の作成手順に従います。 

プログラムでボットをナレッジ ベースに接続するためには、ナレッジ ベースの発行後、次の値を記録する必要があります。
- [QnA Maker](https://www.qnamaker.ai/) サイトで、目的のナレッジ ベースを選択します。
- ナレッジ ベースを開いた状態で **[設定]** を選択します。 "_サービス名_" に表示される値を <your_kb_name> として記録します。
- 下へスクロールして **[デプロイの詳細]** を探し、次の値を記録します。
   - POST /knowledgebases/<your_knowledge_base_id>/generateAnswer
   - ホスト: https://<you_hostname>.azurewebsites.net/qnamaker
   - 認可: EndpointKey <your_endpoint_key>

## <a name="installing-packages"></a>パッケージのインストール

コーディングの前に、QnA Maker に必要なパッケージがあることを確認します。

# <a name="ctabcs"></a>[C#](#tab/cs)

次の [NuGet パッケージ](https://docs.microsoft.com/en-us/nuget/tools/package-manager-ui)をボットに追加します。

* `Microsoft.Bot.Builder.AI.QnA`

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

QnA Maker の機能は、`botbuilder-ai` パッケージに含まれています。 このパッケージは、npm を使用してプロジェクトに追加できます。

```shell
npm install --save botbuilder-ai
```

---

## <a name="using-cli-tools-to-update-your-bot-configuration"></a>CLI ツールを使用した .bot 構成の更新

[qnamaker](https://aka.ms/botbuilder-tools-qnaMaker) と [msbot](https://aka.ms/botbuilder-tools-msbot-readme) の BotBuilder CLI ツールを使用して、ナレッジ ベースに関するメタデータを取得し、それを .bot ファイルに追加することによってナレッジ ベースのアクセス値を取得する方法もあります。

1. ターミナルまたはコマンド プロンプトを開いて、ボット プロジェクトのルート ディレクトリに移動します。
2. `qnamaker init` を実行して QnA Maker のリソース ファイル (**.qnamakerrc**) を作成します。 QnA Maker のサブスクリプション キーが求められます。
3. 次のコマンドを実行してメタデータをダウンロードし、ボットの構成ファイルに追加します。

    ```shell
    qnamaker get kb --kbId <your-kb-id> --msbot | msbot connect qna --stdin [ --secret <your-secret>]
    ```
構成ファイルを暗号化した場合、ファイルを更新するためには秘密鍵を指定する必要があります。

## <a name="using-qna-maker"></a>QnA Maker の使用
ボットを初期化すると、まず QnA Maker への参照が追加されます。 それをボット ロジック内で呼び出すことができます。

# <a name="ctabcs"></a>[C#](#tab/cs)
先ほどダウンロードした QnA Maker サンプルを開きます。 必要に応じてこのコードに変更を加えます。
まず、ナレッジ ベースにアクセスするために必要な情報 (ホスト名、エンドポイント キー、ナレッジ ベース ID (KbId) など) を `BotConfiguration.bot` に追加します。 これらは、QnA Maker でナレッジ ベースの **[設定]** から保存した値です。

```json
{
  "name": "QnABotSample",
  "services": [
    {
      "type": "endpoint",
      "name": "development",
      "endpoint": "http://localhost:3978/api/messages",
      "appId": "",
      "id": "1",
      "appPassword": ""
    },
    {
      "type": "qna",
      "name": "QnABot",
      "KbId": "<YOUR_KNOWLEDGE_BASE_ID>",
      "Hostname": "https://<YOUR_HOSTNAME>.azurewebsites.net/qnamaker",
      "EndpointKey": "<YOUR_ENDPOINT_KEY>"
    }
  ],
  "version": "2.0",
  "padlock": ""
}
```

次は、QnA Maker のインスタンスを `Startup.cs` に作成します。 これにより、前述の情報が `BotConfiguration.bot` ファイルから取得されます。 テスト目的であれば、これらの文字列をハードコーディングしてもかまいません。

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

次に、この QnA Maker インスタンスをボットに加える必要があります。 `QnABot.cs` を開き、ファイルの先頭に次のコードを追加します。 お客様独自のナレッジ ベースにアクセスする場合は、以下の "_ウェルカム_" メッセージを変更して、お客様のユーザーの役に立つ初期手順を設定します。

```csharp
public class QnABot : IBot
{
    public static readonly string QnAMakerKey = "QnABot";
    private const string WelcomeText = @"This bot will introduce you to QnA Maker.
                                         Ask a quesiton to get started.";
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
先ほどダウンロードした QnA Maker サンプルを開きます。 必要に応じてこのコードに変更を加えます。
このサンプルでは、スタートアップ コードが **index.js** ファイルにあります。ボット ロジックのコードは **bot.js** ファイルに、また、追加の構成情報は **qnamaker.bot** ファイルに含まれています。

ナレッジ ベースの作成手順と **.bot** ファイルの更新手順を終えると、**qnamaker.bot** ファイルには、QnA Maker ナレッジ ベースのサービス エントリが含まれています。

```json
{
    "name": "qnamaker",
    "description": "",
    "services": [
        {
            "type": "endpoint",
            "name": "development",
            "id": "1",
            "appId": "",
            "appPassword": "",
            "endpoint": "http://localhost:3978/api/messages"
        },
        {
            "type": "qna",
            "name": "<YOUR_KB_NAME>",
            "kbId": "<YOUR_KNOWLEDGE_BASE_ID>",
            "endpointKey": "<YOUR_ENDPOINT_KEY>",
            "hostname": "https://<YOUR_HOSTNAME>.azurewebsites.net/qnamaker",
            "id": "221"
        }
    ],
    "padlock": "",
    "version": "2.0"
}
```

**index.js** ファイルで、構成情報を読み取って QnA Maker サービスを生成し、ボットを初期化します。

`QNA_CONFIGURATION` の値は、お客様の構成ファイルにあるナレッジ ベースの名前に更新してください。

```js
// QnA Maker knowledge base name as specified in .bot file.
const QNA_CONFIGURATION = '<YOUR_KB_NAME>';

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

**bot.js** ファイルで、ユーザーの入力を QnA Maker サービスの `generateAnswer` メソッドに渡して、ナレッジ ベースから回答を取得します。 お客様独自のナレッジ ベースにアクセスする場合は、以下の "_回答なし_" メッセージと "_ウェルカム_" メッセージを変更して、お客様のユーザーの役に立つ手順を設定します。

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
            const qnaResults = await this.qnaMaker.generateAnswer(turnContext.activity.text);

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

ボットに質問して、QnA Maker サービスからの応答を表示します。 QnA サービスのテストと発行の詳細については、[ナレッジ ベースのテスト](https://docs.microsoft.com/en-us/azure/cognitive-services/qnamaker/how-to/test-knowledge-base)に関する QnA Maker の記事を参照してください。

## <a name="next-steps"></a>次の手順

他の Cognitive Services と QnA Maker を組み合わせて、ボットをさらに強力にすることができます。 ディスパッチ ツールを使用すると、ボットで QnA と Language Understanding (LUIS) を結合できます。

> [!div class="nextstepaction"]
> [ディスパッチ ツールを使用して LUIS と QnA サービスを組み合わせる](./bot-builder-tutorial-dispatch.md)
