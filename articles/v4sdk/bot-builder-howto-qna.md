---
title: QnA Maker を使用して質問に回答する | Microsoft Docs
description: ボットで QnA Maker を使用する方法について説明します。
keywords: 質問と回答, QnA, FAQ, ミドルウェア
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 03/13/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 7dd973e2b5a151e754925e6f19c6e4f82507f745
ms.sourcegitcommit: 2dc75701b169d822c9499e393439161bc87639d2
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 08/24/2018
ms.locfileid: "42906176"
---
# <a name="use-qna-maker-to-answer-questions"></a>QnA Maker を使用して質問に回答する


簡単な質問と回答のサポートをボットに追加するには、[QnA Maker](https://docs.microsoft.com/en-us/azure/cognitive-services/qnamaker/home) サービスを使用できます。


独自の QnA Maker サービスを作成するときの基本的な要件の 1 つは、質問と回答でサービスをシードすることです。 多くの場合、質問と回答は、FAQ や他のドキュメントなどのコンテンツに既に存在しています。 また、より自然な会話になるように質問に対する回答をカスタマイズしたいこともあります。 

## <a name="create-a-qna-maker-service"></a>QnA Maker サービスを作成する
最初にアカウントを作成し、[QnA Maker](https://qnamaker.ai/) にサインインします。 次に、**[Create a knowledge base]\(ナレッジ ベースの作成\)** に移動します。 **[Create a QnA service]\(QnA サービスの作成\)** をクリックし、手順に従って Azure QnA サービスを作成します。

![QnA の画像 1](media/QnA_1.png)

[[Create QnA Maker]\(QnA Maker の作成\)](https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesQnAMaker) にリダイレクトされます。 フォームに入力して **[Create]**(作成) をクリックします。

![QnA の画像 2](media/QnA_2.png)
 
Azure portal で QnA のサービスを作成すると、[Resource Management]\(リソース管理\) という見出しの下にキーが表示されますが、これは無視してかまいません。 ステップ 2 に進み、Azure QnA サービスに接続します。 ページを更新して作成した Azure サービスを選択し、ナレッジ ベースの名前を入力します。

![QnA の画像 3](media/QnA_3.png)

**[Create your KB]\(KB の作成\)** をクリックします。 独自の質問と回答を入力するか、次の例をコピーします。 

![QnA の画像 4](media/QnA_4.png)

または、**[Populate your KB]\(KB の設定\)** を選択し、ファイルまたは URL を指定することもできます。 簡単な QnA Maker サービスを生成するためのソース ファイルのサンプルについては、[こちら](https://aka.ms/qna-tsv) をご覧ください。

新しい QnA ペアを追加するか、KB を取り込んだ後、**[Save and train]\(保存してトレーニング\)** をクリックします。 完了したら、**[PUBLISH]\(発行\)** タブで **[Publish]\(発行\)** をクリックします。

QnA サービスをボットに接続するには、ナレッジ ベース ID と QnA Maker サブスクリプション キーを含む HTTP 要求文字列が必要です。 発行の結果から HTTP 要求の例をコピーします。 

![QnA の画像 5](media/QnA_5.png)

## <a name="installing-packages"></a>パッケージのインストール

コーディングの前に、QnA Maker に必要なパッケージがあることを確認します。

# <a name="ctabcs"></a>[C#](#tab/cs)

次の NuGet パッケージの v4 プレリリース バージョンへの[参照を追加](https://docs.microsoft.com/en-us/nuget/tools/package-manager-ui)します。

* `Microsoft.Bot.Builder.Ai.QnA`

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

botbuilder-ai パッケージを使用して、これらのサービスのいずれかをボットに追加できます。 このパッケージは、npm を使用してプロジェクトに追加できます。

* `npm install --save botbuilder@preview`
* `npm install --save botbuilder-ai@preview`

---


## <a name="using-qna-maker"></a>QnA Maker の使用

QnA Maker は最初にミドルウェアとして追加されます。 その後は、ボットのロジック内で結果を使用できます。

# <a name="ctabcs"></a>[C#](#tab/cs)

`Startup.cs` ファイルの `ConfigureServices` メソッドを更新して `QnAMakerMiddleware` オブジェクトを追加します。 ユーザーからメッセージを受信するたびにナレッジ ベースをチェックするようにボットを構成できます。ボットのミドルウェア スタックにナレッジ ベースを追加するだけです。


```csharp
using Microsoft.AspNetCore.Builder;
using Microsoft.AspNetCore.Hosting;
using Microsoft.Bot.Builder.Ai.Qna;
using Microsoft.Bot.Builder.BotFramework;
using Microsoft.Bot.Builder.Core.Extensions;
using Microsoft.Bot.Builder.Integration.AspNet.Core;
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Logging;
using System;

public void ConfigureServices(IServiceCollection services)
{
    services.AddSingleton(_ => Configuration);
    services.AddBot<AiBot>(options =>
    {
        options.CredentialProvider = new ConfigurationCredentialProvider(Configuration);

        var endpoint = new QnAMakerEndpoint
        {
           KnowledgeBaseId = "YOUR-KB-ID",
           // Get the Host from the HTTP request example at https://www.qnamaker.ai
           // For GA services: https://<Service-Name>.azurewebsites.net/qnamaker
           // For Preview services: https://westus.api.cognitive.microsoft.com/qnamaker/v2.0           
           Host = "YOUR-HTTP-REQUEST-HOST",
           EndpointKey = "YOUR-QNA-MAKER-SUBSCRIPTION-KEY"
        };
        options.Middleware.Add(new QnAMakerMiddleware(endpoint));
    });
}
```



EchoBot.cs ファイルのコードを編集し、QnA Maker のミドルウェアがユーザーの質問への応答を送信しなかった場合に、`OnTurn` がフォールバック メッセージを送信するようにします。

```csharp
using System.Threading.Tasks;
using Microsoft.Bot;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Schema;

namespace Bot_Builder_Echo_Bot_QnA
{
    public class EchoBot : IBot
    {    
        public async Task OnTurn(ITurnContext context)
        {
            // This bot is only handling Messages
            if (context.Activity.Type == ActivityTypes.Message)
            {             
                if (!context.Responded)
                {
                    // QnA didn't send the user an answer
                    await context.SendActivity("Sorry, I couldn't find a good match in the KB.");

                }
            }
        }
    }
}
```


ボットのサンプルについては、[QnA Maker のサンプル](https://aka.ms/qna-cs-bot-sample)に関するページをご覧ください。

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

最初に、[QnAMaker](https://github.com/Microsoft/botbuilder-js/tree/master/doc/botbuilder-ai/classes/botbuilder_ai.qnamaker.md) クラスで要求/インポートします。

```js
const { QnAMaker } = require('botbuilder-ai');
```

QnA サービスの HTTP 要求に基づく文字列で初期化することにより、`QnAMaker` を作成します。 サービスに対する要求の例を、[QnA Maker ポータル](https://qnamaker.ai)の [Settings]\(設定\) > [Deployment details]\(デプロイの詳細\) からコピーできます。


文字列の形式は、QnA Maker サービスが GA またはプレビューのどちらのバージョンの QnA Maker を使用しているかにより異なります。 HTTP 要求の例をコピーし、`QnAMaker` を初期化するときに使用するナレッジ ベース ID、サブスクリプション キー、およびホストを取得します。

<!--
**Preview**
```js
const qnaEndpointString = 
    // Replace xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx with your knowledge base ID
    "POST /knowledgebases/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/generateAnswer\r\n" + 
    "Host: https://westus.api.cognitive.microsoft.com/qnamaker/v2.0\r\n" +
    // Replace xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx with your QnAMaker subscription key
    "Ocp-Apim-Subscription-Key: xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx\r\n"
const qna = new QnAMaker(qnaEndpointString);
```

**GA**
```js
const qnaEndpointString = 
    // Replace xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx with your knowledge base ID
    "POST /knowledgebases/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/generateAnswer\r\n" + 
    // Replace <Service-Name> to match the Azure URL where your service is hosted
    "Host: https://<Service-Name>.azurewebsites.net/qnamaker\r\n" +
    // Replace xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx with your QnAMaker subscription key
    "Authorization: EndpointKey xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx\r\n"
const qna = new QnAMaker(qnaEndpointString);
```
-->
**プレビュー**
```js
const qna = new QnAMaker(
    {
        knowledgeBaseId: '<KNOWLEDGE-BASE-ID>',
        endpointKey: '<QNA-SUBSCRIPTION-KEY>',
        host: 'https://westus.api.cognitive.microsoft.com/qnamaker/v2.0'
    },
    {
        // set this to true to send answers from QnA Maker
        answerBeforeNext: true
    }
);
```
**一般公開**
```js
const qna = new QnAMaker(
    {
        knowledgeBaseId: '<KNOWLEDGE-BASE-ID>',
        endpointKey: '<QNA-SUBSCRIPTION-KEY>',
        host: 'https://<Service-Name>.azurewebsites.net/qnamaker'
    },
    {
        answerBeforeNext: true
    }
);
```
ボットのミドルウェア スタックに QNA Maker を追加するだけで、QNA Maker を自動的に呼び出すようにボットを構成できます。

```js
// Add QnA Maker as middleware
adapter.use(qna);

// Listen for incoming requests 
server.post('/api/messages', (req, res) => {
    // Route received request to adapter for processing
    adapter.processActivity(req, res, async (context) => {
        // If `!context.responded`, that means an answer wasn't found for the user's utterance.
        // In this case, we send the user a fallback message.
        if (context.activity.type === 'message' && !context.responded) {
            await context.sendActivity('No QnA Maker answers were found.');
        } else if (context.activity.type !== 'message') {
            await context.sendActivity(`[${context.activity.type} event detected]`);
        }
    });
});
```

`QnAMaker` を初期化するときに `answerBeforeNext` を `true` に初期化すると、QnA Maker のミドルウェアが回答を発見した場合は、`processActivity` のボットのメイン ロジックを実行する前に、QnA Maker のミドルウェアが自動的に応答することを意味します。 代わりに、`answerBeforeNext` を `false` に設定した場合は、`processActivity` のボットのメイン ロジックがすべて実行された後、ボットがユーザーに返信していない場合にのみ、ボットは QnA Maker を呼び出します。 

## <a name="calling-qna-maker-without-using-middleware"></a>ミドルウェアを使用しないで QnA Maker を呼び出す

前の例の `adapter.use(qna);` ステートメントは、QnA がミドルウェアとして実行しており、したがってボットで受信したすべてのメッセージに応答することを意味します。 QnA Maker を呼び出す方法とタイミングをより細かく制御したい場合は、QnA Maker をミドルウェアの一部としてインストールするのではなく、ボットのロジック内から `qna.answer()` を直接呼び出すことができます。

`adapter.use(qna);` ステートメントを削除し、次のコードを使って QnA Maker を直接呼び出します。

```js
// Listen for incoming activity 
server.post('/api/messages', (req, res) => {
    // Route received activity to adapter for processing
    adapter.processActivity(req, res, async (context) => {
        if (context.activity.type === 'message') {
            var handled = await qna.answer(context)
                if (!handled) {
                    await context.sendActivity(`I'm sorry. I didn't understand.`);
                }
        }
    });
});
```

QnA Maker をカスタマイズするもう 1 つの方法は、`qna.generateAnswer()` を使うことです。 このメソッドを使うと、より詳細な回答を QnA Maker から取得できます。


```js
// Listen for incoming activity 
server.post('/api/messages', (req, res) => {
    // Route received activity to adapter for processing
    adapter.processActivity(req, res, async (context) => {
        if (context.activity.type === 'message') {
            // Get all the answers QnA Maker finds
            var results = await qna.generateAnswer(context.activity.text);
                if (results && results.length > 0) {
                    await context.sendActivity(results[0].answer);
                } else {
                    await context.sendActivity(`I don't know.`);
                }
    
        }
    });
});
```
---

ボットに質問して、QnA Maker サービスからの応答を表示します。

![QnA の画像 6](media/QnA_6.png)



## <a name="next-steps"></a>次の手順

他の Cognitive Services と QnA Maker を組み合わせて、ボットをさらに強力にすることができます。 ディスパッチ ツールを使用すると、ボットで QnA と Language Understanding (LUIS) を結合できます。

> [!div class="nextstepaction"]
> [ディスパッチ ツールを使用して LUIS アプリと QnA サービスを組み合わせる](./bot-builder-tutorial-dispatch.md)
