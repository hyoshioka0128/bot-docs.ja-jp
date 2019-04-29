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
ms.date: 04/18/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: bd29aa1ee56ebf64dc5db2edc47adc3ab250e7d5
ms.sourcegitcommit: aea57820b8a137047d59491b45320cf268043861
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/22/2019
ms.locfileid: "59904945"
---
# <a name="tutorial-use-qna-maker-in-your-bot-to-answer-questions"></a>チュートリアル:ボットで QnA Maker を使用して質問に回答する

[!INCLUDE [applies-to-v4](../includes/applies-to.md)]

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

前のチュートリアルに対するこれらの前提条件が既に揃っている場合もあります。

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

## <a name="add-knowledge-base-information-to-your-bot"></a>ナレッジ ベースの情報をボットに追加する
ボット フレームワーク v4.3 以降、Azure では、ダウンロードされたボットのソース コードの一部として .bot ファイルが提供されなくなりました。 次の手順に従って、CSharp または JavaScript ボットをナレッジベースに接続します。

## <a name="ctabcsharp"></a>[C#](#tab/csharp)

次の値を appsetting.json ファイルに追加します。

```json
{
   "MicrosoftAppId": "",
  "MicrosoftAppPassword": "",
  "ScmType": "None",

  "kbId": "<your-knowledge-base-id>",
  "endpointKey": "<your-knowledge-base-endpoint-key>",
  "hostname": "<your-qna-service-hostname>" // This is a URL
}
```

## <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

次の値を .env ファイルに追加します。

```javascript
MicrosoftAppId=""
MicrosoftAppPassword=""
ScmType=None

kbId="<your-knowledge-base-id>"
endpointKey="<your-knowledge-base-endpoint-key>"
hostname="<your-qna-service-hostname>" // This is a URL

```

---

    | フィールド | 値 |
    |:----|:----|
    | kbId | QnA Maker ポータルで自動的に生成されたナレッジ ベース ID。 |
    | endpointKey | QnA Maker ポータルで自動的に生成されエンドポイント キー。 |
    | hostname | QnA Maker ポータルで生成されたホスト URL。 `https://` で始まって `/qnamaker` で終わる完全な URL を使用します。 |

編集結果を保存します。

## <a name="update-your-bot-to-query-the-knowledge-base"></a>ナレッジ ベースに対してクエリを実行するようにボットを更新する

自分のナレッジ ベースのサービス情報を読み込むように、初期化コードを更新します。

## <a name="ctabcsharp"></a>[C#](#tab/csharp)

1. **Microsoft.Bot.Builder.AI.QnA** NuGet パッケージをプロジェクトに追加します。
1. **Microsoft.Extensions.Configuration** NuGet パッケージをプロジェクトに追加します。
1. **startup.cs** ファイルに、次の名前空間参照を追加します。

   **startup.cs**
   ```csharp
       using Microsoft.Bot.Builder.AI.QnA;
       using Microsoft.Extensions.Configuration;
   ```
1. _ConfigureServices_ メソッドを変更し、**appsettings.json** ファイルに定義されるナレッジ ベースに接続された QnAMkaerEndpoint を作成します。

   **startup.cs**
   ```csharp
   // Create QnAMaker endpoint as a singleton
   services.AddSingleton(new QnAMakerEndpoint
   {
      KnowledgeBaseId = Configuration.GetValue<string>($"kbId"),
      EndpointKey = Configuration.GetValue<string>($"endpointKey"),
      Host = Configuration.GetValue<string>($"hostname")
    });

   ```
1. **EchoBot.cs** ファイルに、次の名前空間参照を追加します。

   **EchoBot.cs**
   ```csharp
   using System.Linq;
   using Microsoft.Bot.Builder.AI.QnA;
   ```

1. `EchoBotQnA` コネクタを追加し、ボットのコンストラクターで初期化します。

   **EchoBot.cs**
   ```csharp
   public QnAMaker EchoBotQnA { get; private set; }
   public EchoBot(QnAMakerEndpoint endpoint)
   {
      // connects to QnA Maker endpoint for each turn
      EchoBotQnA = new QnAMaker(endpoint);
   }
   ```
1. _OnMembersAddedAsync( )_ メソッドの下に _AccessQnAMaker( )_ メソッドを作成します。そのためには、次のコードを追加します。

   **EchoBot.cs**
   ```csharp
   private async Task AccessQnAMaker(ITurnContext<IMessageActivity> turnContext, CancellationToken cancellationToken)
   {
      var results = await EchoBotQnA.GetAnswersAsync(turnContext);
      if (results.Any())
      {
         await turnContext.SendActivityAsync(MessageFactory.Text("QnA Maker Returned: " + results.First().Answer), cancellationToken);
      }
      else
      {
         await turnContext.SendActivityAsync(MessageFactory.Text("Sorry, could not find an answer in the Q and A system."), cancellationToken);
      }
   }
   ```
1. 次に、_OnMessageActivityAsync ()_ 内で、次のように新しいメソッド _AccessQnAMaker ()_ を呼び出します。

   **EchoBot.cs**
   ```csharp
   protected override async Task OnMessageActivityAsync(ITurnContext<IMessageActivity> turnContext, CancellationToken cancellationToken)
   {
      // First send the user input to your QnA Maker knowledgebase
      await AccessQnAMaker(turnContext, cancellationToken);
      ...
   }
   ```

## <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

1. ターミナルまたはコマンド プロンプトを開いて、プロジェクトのルート ディレクトリに移動します。
1. **botbuilder-ai** npm パッケージをプロジェクトに追加します。
   ```shell
   npm i botbuilder-ai
   ```

1. **Index.js** で、// アダプターの作成セクションに続けて、次のコードを追加して QnA Maker サービスを生成するために必要な .env ファイル構成情報を読み取ります。

   **index.js**
   ```javascript
   // Map knowledgebase endpoint values from .env file into the required format for `QnAMaker`.
   const configuration = {
      knowledgeBaseId: process.env.kbId,
      endpointKey: process.env.endpointKey,
      host: process.env.hostname
   };

   ```

1. QnA servicesconfiguration 情報を渡すようにボットの構造を更新します。

   **index.js**
   ```javascript
   // Create the main dialog.
   const myBot = new MyBot(configuration, {}, logger);
   ```

1. **bot.js** ファイルに、QnAMaker に必要なこれを追加します。

   **bot.js**
   ```javascript
   const { QnAMaker } = require('botbuilder-ai');
   ```

1. QnAMaker コネクタを作成するのに必要な渡された構成パラメーターを受けとって、これらのパラメーターが指定されていない場合にエラーをスローするようにコンストラクターを変更します。

   **bot.js**
   ```javascript
      class MyBot extends ActivityHandler {
         constructor(configuration, qnaOptions) {
            super();
            if (!configuration) throw new Error('[QnaMakerBot]: Missing parameter. configuration is required');
            // now create a qnaMaker connector.
            this.qnaMaker = new QnAMaker(configuration, qnaOptions);
   ```

1. 最後に、各ユーザーに QnA Maker ナレッジベースへの入力を渡し、ユーザーに QnA Maker 応答を返す onMessage( ) 呼び出しを追加します。  回答を得るためにナレッジ ベースにクエリを実行します。
 
    **bot.js**
    ```javascript
   // send user input to QnA Maker.
   const qnaResults = await this.qnaMaker.getAnswers(turnContext);

   // If an answer was received from QnA Maker, send the answer back to the user.
   if (qnaResults[0]) {
      await turnContext.sendActivity(`QnAMaker returned response: ' ${ qnaResults[0].answer}`);
   } 
   else { 
      // If no answers were returned from QnA Maker, reply with help.
      wait turnContext.sendActivity('No QnA Maker response was returned.'
           + 'This example uses a QnA Maker Knowledge Base that focuses on smart light bulbs. '
           + `Ask the bot questions like "Why won't it turn on?" or "I need help."`);
   }
   ```
---

### <a name="test-the-bot-locally"></a>ボットをローカルでテストする

この時点で、ボットはいくつかの質問に回答できるようになっているはずです。 ローカル環境でボットを実行し、エミュレーターで開きます。

![QnA サンプルをテストする](./media/qna-test-bot.png)

## <a name="re-publish-your-bot"></a>ボットを再公開する

これで、ボットを Azure に再公開できる状態になりました。

## <a name="ctabcsharp"></a>[C#](#tab/csharp)

[!INCLUDE [publish snippet](~/includes/deploy/snippet-publish.md)]

## <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

[!INCLUDE [publish snippet](~/includes/deploy/snippet-publish-js.md)]

---

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
