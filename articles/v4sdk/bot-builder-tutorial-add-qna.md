---
title: 質問に回答するボットの Azure Bot Service チュートリアル | Microsoft Docs
description: QnA Maker を使用して質問に回答するボットに関するチュートリアルです。
keywords: QnA Maker, 質問と回答, ナレッジ ベース
author: JonathanFingold
ms.author: v-jofing
manager: kamrani
ms.topic: tutorial
ms.service: bot-service
ms.subservice: sdk
ms.date: 01/15/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: b1c34531ee60b2ce9037f42e4f5a7093501cf83a
ms.sourcegitcommit: bdb981c0b11ee99d128e30ae0462705b2dae8572
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/17/2019
ms.locfileid: "54360955"
---
# <a name="tutorial-use-qna-maker-in-your-bot-to-answer-questions"></a>チュートリアル:ボットで QnA Maker を使用して質問に回答する

QnA Maker サービスとナレッジ ベースを使用して、質問と回答のサポートをボットに追加できます。 ナレッジ ベースを作成したら、それに質問と回答をシードします。

このチュートリアルでは、以下の内容を学習します。

> [!div class="checklist"]
> * QnA Maker サービスとナレッジ ベースを作成する
> * ナレッジ ベースの情報を .bot ファイルに追加する
> * ナレッジ ベースに対してクエリを実行するようにボットを更新する
> * ボットを再公開する

Azure サブスクリプションがない場合は、開始する前に[無料アカウント](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)を作成してください。

## <a name="prerequisites"></a>前提条件

* [前のチュートリアル](bot-builder-tutorial-basic-deploy.md)で作成したボット。 そのボットに質問と回答の機能を追加します。
* QnA Maker についてある程度理解していると役に立ちます。 QnA Maker ポータルを使用して、ボットで使用できるように、ナレッジ ベースを作成、トレーニング、公開します。

前のチュートリアルに対する前提条件が既に揃っている必要があります。

[!INCLUDE [deployment prerequisites snippet](~/includes/deploy/snippet-prerequisite.md)]

## <a name="sign-in-to-qna-maker-portal"></a>QnA Maker ポータルにサインインする

<!-- This and the next step are close duplicates of what's in the QnA How-To -->

Azure 資格情報を使用して [QnA Maker ポータル](https://qnamaker.ai/)にサインインします。

## <a name="create-a-qna-maker-service-and-knowledge-base"></a>QnA Maker サービスとナレッジ ベースを作成する

[Microsoft/BotBuilder-Samples](https://github.com/Microsoft/BotBuilder-Samples) リポジトリにある QnA Maker のサンプルから、既存のナレッジ ベース定義をインポートします。

1. サンプル リポジトリをお使いのコンピューターに複製するかコピーします。
1. QnA Maker ポータルで**ナレッジ ベースを作成**します。
   1. 必要な場合は、QnA サービスを作成します。 (既存の QnA Maker サービスを使用しても、このチュートリアル用に新しく作成してもかまいません。)QnA Maker での作業について詳しくは、「[QnA Maker サービスを作成する](https://docs.microsoft.com/en-us/azure/cognitive-services/qnamaker/how-to/set-up-qnamaker-service-azure)」および「[QnA Maker ナレッジ ベースの作成、トレーニング、発行](https://docs.microsoft.com/en-us/azure/cognitive-services/qnamaker/quickstarts/create-publish-knowledge-base)」をご覧ください。
   1. QnA サービスをナレッジ ベースに接続します。
   1. ナレッジ ベースの名前を設定します。
   1. ナレッジ ベースにデータを追加するには、サンプル リポジトリの **BotBuilder-Samples\samples\csharp_dotnetcore\11.qnamaker\CognitiveModels\smartLightFAQ.tsv** ファイルを使用します。
   1. **[Create your kb]\(KB の作成\)** をクリックして、ナレッジ ベースを作成します。
1. ナレッジ ベースを**保存してトレーニング**します。
1. ナレッジ ベースを**公開**します。

   ボットでナレッジ ベースを使用できる状態になりました。 ナレッジ ベース ID、エンドポイント キー、およびホスト名を記録しておきます。 これらは次のステップで必要になります。

## <a name="add-knowledge-base-information-to-your-bot-file"></a>ナレッジ ベースの情報を .bot ファイルに追加する

ナレッジ ベースにアクセスするために必要な情報を .bot ファイルに追加します。

1. エディターで .bot ファイルを開きます。
1. `services` 配列に `qna` 要素を追加します。

    ```json
    {
        "type": "qna",
        "name": "<your-knowledge-base-name>",
        "kbId": "<your-knowledge-base-id>",
        "hostname": "<your-qna-service-hostname>",
        "endpointKey": "<your-knowledge-base-endpoint-key>",
        "subscriptionKey": "<your-azure-subscription-key>",
        "id": "<a-unique-id>"
    }
    ```

    | フィールド | 値 |
    |:----|:----|
    | type | `qna`である必要があります。 これは、このサービス エントリで QnA のナレッジ ベースが記述されていることを示します。 |
    | name | ナレッジ ベースに割り当てた名前。 |
    | kbId | QnA Maker ポータルで自動的に生成されたナレッジ ベース ID。 |
    | hostname | QnA Maker ポータルで生成されたホスト URL。 `https://` で始まって `/qnamaker` で終わる完全な URL を使用します。 |
    | endpointKey | QnA Maker ポータルで自動的に生成されエンドポイント キー。 |
    | subscriptionKey | Azure で QnA Maker サービスを作成するときに使用したサブスクリプションの ID。 |
    | id | .bot ファイルに列記されている他のサービスのいずれでもまだ使用されていない一意の ID (例: "201")。 |

1. 編集結果を保存します。

## <a name="update-your-bot-to-query-the-knowledge-base"></a>ナレッジ ベースに対してクエリを実行するようにボットを更新する

自分のナレッジ ベースのサービス情報を読み込むように、初期化コードを更新します。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

1. **Microsoft.Bot.Builder.AI.QnA** NuGet パッケージをプロジェクトに追加します。
1. **IBot** を実装するクラスの名前を `QnaBot` に変更します。
1. ボットのアクセサーを含むクラスの名前を `QnaBotAccessors` に変更します。
1. **Startup.cs** ファイルに、次の名前空間参照を追加します。
    ```csharp
    using System.Collections.Generic;
    using System.Linq;
    using Microsoft.Bot.Builder.AI.QnA;
    using Microsoft.Bot.Builder.Integration;
    ```
1. そして、**.bot** ファイルで定義されているナレッジ ベースを初期化して登録するように、**ConfigureServices** メソッドを変更します。 以下の最初の数行は、その前に出現する `services.AddBot<QnaBot>(options =>` の呼び出しの本文から移動されたことに注意してください。
    ```csharp
    public void ConfigureServices(IServiceCollection services)
    {
        var secretKey = Configuration.GetSection("botFileSecret")?.Value;
        var botFilePath = Configuration.GetSection("botFilePath")?.Value;

        // Loads .bot configuration file and adds a singleton that your Bot can access through dependency injection.
        var botConfig = BotConfiguration.Load(botFilePath ?? @".\jfEchoBot.bot", secretKey);
        services.AddSingleton(sp => botConfig ?? throw new InvalidOperationException($"The .bot config file could not be loaded. ({botConfig})"));

        // Initialize the QnA knowledge bases for the bot.
        services.AddSingleton(sp => {
            var qnaServices = new List<QnAMaker>();
            foreach (var qnaService in botConfig.Services.OfType<QnAMakerService>())
            {
                qnaServices.Add(new QnAMaker(qnaService));
            }
            return qnaServices;
        });

        services.AddBot<QnaBot>(options =>
        {
            // Retrieve current endpoint.
            // ...
        });

        // Create and register state accessors.
        // ...
    }
    ```
1. **QnaBot.cs** ファイルに、次の名前空間参照を追加します。
    ```csharp
    using System.Collections.Generic;
    using Microsoft.Bot.Builder.AI.QnA;
    ```
1. `_qnaServices` プロパティを追加し、ボットのコンストラクターで初期化します。
    ```csharp
    private readonly List<QnAMaker> _qnaServices;

    /// ...
    public QnaBot(QnaBotAccessors accessors, List<QnAMaker> qnaServices, ILoggerFactory loggerFactory)
    {
        // ...
        _qnaServices = qnaServices;
    }
    ```
1. ユーザーの入力に対して登録済みのナレッジ ベースのクエリを実行するように、ターン ハンドラーを変更します。 QnA Maker からの回答をボットが必要とする場合、ボットのコードから `GetAnswersAsync` を呼び出して、現在のコンテキストに基づいて適切な回答を取得します。 お客様独自のナレッジ ベースにアクセスする場合は、以下の "_回答なし_" メッセージを変更して、お客様のユーザーの役に立つ手順を設定します。
    ```csharp
    public async Task OnTurnAsync(ITurnContext turnContext, CancellationToken cancellationToken = default(CancellationToken))
    {
        if (turnContext.Activity.Type == ActivityTypes.Message)
        {
            foreach(var qnaService in _qnaServices)
            {
                var response = await qnaService.GetAnswersAsync(turnContext);
                if (response != null && response.Length > 0)
                {
                    await turnContext.SendActivityAsync(
                        response[0].Answer,
                        cancellationToken: cancellationToken);
                    return;
                }
            }

            var msg = "No QnA Maker answers were found. This example uses a QnA Maker knowledge base that " +
                "focuses on smart light bulbs. Ask the bot questions like 'Why won't it turn on?' or 'I need help'.";

            await turnContext.SendActivityAsync(msg, cancellationToken: cancellationToken);
        }
        else
        {
            await turnContext.SendActivityAsync($"{turnContext.Activity.Type} event detected");
        }
    }
    ```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

1. ターミナルまたはコマンド プロンプトを開いて、プロジェクトのルート ディレクトリに移動します。
1. **botbuilder-ai** npm パッケージをプロジェクトに追加します。
    ```shell
    npm i botbuilder-ai
    ```
1. **index.js** ファイルに、次の require ステートメントを追加します。
    ```javascript
    const { QnAMaker } = require('botbuilder-ai');
    ```
1. 構成情報を読み取って、QnA Maker サービスを生成します。
    ```javascript
    // Read bot configuration from .bot file.
    // ...

    // Initialize the QnA knowledge bases for the bot.
    // Assume each QnA entry in the .bot file is well defined.
    const qnaServices = [];
    botConfig.services.forEach(s => {
        if (s.type == 'qna') {
            const endpoint = {
                knowledgeBaseId: s.kbId,
                endpointKey: s.endpointKey,
                host: s.hostname
            };
            const options = {};
            qnaServices.push(new QnAMaker(endpoint, options));
        }
    });

    // Get bot endpoint configuration by service name
    // ...
    ```
1. QnA サービスを渡すようにボットの構造を更新します。
    ```javascript
    // Create the bot.
    const myBot = new MyBot(qnaServices);
    ```
1. **bot.js** ファイルに、コンストラクターを追加します。
    ```javascript
    constructor(qnaServices) {
        this.qnaServices = qnaServices;
    }
    ```
1. そして、ナレッジ ベースで回答のクエリを実行するように、ターン ハンドラーを更新します。
    ```javascript
    async onTurn(turnContext) {
        if (turnContext.activity.type === ActivityTypes.Message) {
            for (let i = 0; i < this.qnaServices.length; i++) {
                // Perform a call to the QnA Maker service to retrieve matching Question and Answer pairs.
                const qnaResults = await this.qnaServices[i].getAnswers(turnContext);

                // If an answer was received from QnA Maker, send the answer back to the user and exit.
                if (qnaResults[0]) {
                    await turnContext.sendActivity(qnaResults[0].answer);
                    return;
                }
            }
            // If no answers were returned from QnA Maker, reply with help.
            await turnContext.sendActivity('No QnA Maker answers were found. '
                + 'This example uses a QnA Maker Knowledge Base that focuses on smart light bulbs. '
                + `Ask the bot questions like "Why won't it turn on?" or "I need help."`);
        } else {
            await turnContext.sendActivity(`[${ turnContext.activity.type } event detected]`);
        }
    }
    ```

---

### <a name="test-the-bot-locally"></a>ボットをローカルでテストする

この時点で、ボットはいくつかの質問に回答できるようになっているはずです。 ローカル環境でボットを実行し、エミュレーターで開きます。

![QnA サンプルをテストする](~/media/emulator-v4/qna-test-bot.png)

## <a name="re-publish-your-bot"></a>ボットを再公開する

ボットを再公開できる状態になりました。

[!INCLUDE [publish snippet](~/includes/deploy/snippet-publish.md)]

### <a name="test-the-published-bot"></a>公開したボットをテストする

ボットを公開した後、Azure でボットが更新されて開始されるまで、1、2 分待ちます。

1. エミュレーターを使用してボットの運用エンドポイントをテストするか、または Azure portal を使用して Web チャットでボットをテストします。

   いずれの場合も、ローカル環境でテストしたときと同じ動作になるはずです。

## <a name="clean-up-resources"></a>リソースのクリーンアップ

<!-- In the first tutorial, we should tell them to use a new resource group, so that it is easy to clean up resources. We should also mention in this step in the first tutorial not to clean up resources if they are continuing with the sequence. -->

このアプリケーションを引き続き使用しない場合は、次の手順で関連付けられているリソースを削除します。

1. Azure portal で、ボットのリソース グループを開きます。
1. **[リソース グループの削除]** をクリックして、グループとそれに含まれるすべてのリソースを削除します。
1. 確認ウィンドウでリソース グループ名を入力して、**[削除]** をクリックします。

## <a name="next-steps"></a>次の手順

ボットに機能を追加する方法については、開発ハウツー セクションの記事をご覧ください。
> [!div class="nextstepaction"]
> [次のステップのボタン](bot-builder-howto-send-messages.md)
