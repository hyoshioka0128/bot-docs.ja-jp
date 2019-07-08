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
ms.date: 05/23/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: aff1440f739150181ddc2d65d9b749b4eeda5d79
ms.sourcegitcommit: dbbfcf45a8d0ba66bd4fb5620d093abfa3b2f725
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 06/28/2019
ms.locfileid: "67464695"
---
# <a name="tutorial-use-qna-maker-in-your-bot-to-answer-questions"></a>チュートリアル:ボットで QnA Maker を使用して質問に回答する

[!INCLUDE [applies-to-v4](../includes/applies-to.md)]

QnA Maker サービスとナレッジ ベースを使用して、質問と回答のサポートをボットに追加できます。 ナレッジ ベースを作成したら、それに質問と回答をシードします。

このチュートリアルでは、以下の内容を学習します。

> [!div class="checklist"]
> * QnA Maker サービスとナレッジ ベースを作成する
> * ナレッジ ベースの情報を構成ファイルに追加する
> * ナレッジ ベースに対してクエリを実行するようにボットを更新する
> * ボットを再公開する

Azure サブスクリプションがない場合は、開始する前に[無料アカウント](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)を作成してください。

## <a name="prerequisites"></a>前提条件

* [前のチュートリアル](bot-builder-tutorial-basic-deploy.md)で作成したボット。 そのボットに質問と回答の機能を追加します。
* [QnA Maker](https://qnamaker.ai/) についてある程度理解していると役に立ちます。 QnA Maker ポータルを使用して、ボットで使用できるように、ナレッジ ベースを作成、トレーニング、公開します。
* Azure Bot Service を使用した [QnA ボット作成](https://aka.ms/azure-create-qna)に関する知識。

前のチュートリアルに対する前提条件が既に揃っている場合もあります。

## <a name="sign-in-to-qna-maker-portal"></a>QnA Maker ポータルにサインインする

<!-- This and the next step are close duplicates of what's in the QnA How-To -->

Azure 資格情報を使用して [QnA Maker ポータル](https://qnamaker.ai/)にサインインします。

## <a name="create-a-qna-maker-service-and-knowledge-base"></a>QnA Maker サービスとナレッジ ベースを作成する

[Microsoft/BotBuilder-Samples](https://github.com/Microsoft/BotBuilder-Samples) リポジトリにある QnA Maker のサンプルから、既存のナレッジ ベース定義をインポートします。

1. サンプル リポジトリをお使いのコンピューターに複製するかコピーします。
1. QnA Maker ポータルで**ナレッジ ベースを作成**します。
   1. 必要な場合は、QnA サービスを作成します。 (既存の QnA Maker サービスを使用しても、このチュートリアル用に新しく作成してもかまいません。)QnA Maker での作業について詳しくは、「[QnA Maker サービスを作成する](https://docs.microsoft.com/azure/cognitive-services/qnamaker/how-to/set-up-qnamaker-service-azure)」および「[QnA Maker ナレッジ ベースの作成、トレーニング、発行](https://docs.microsoft.com/azure/cognitive-services/qnamaker/quickstarts/create-publish-knowledge-base)」をご覧ください。
   1. QnA サービスをナレッジ ベースに接続します。
   1. ナレッジ ベースの名前を設定します。
   1. ナレッジ ベースにデータを追加するには、サンプル リポジトリの **BotBuilder-Samples\samples\csharp_dotnetcore\11.qnamaker\CognitiveModels\smartLightFAQ.tsv** ファイルを使用します。
   1. **[Create your kb]\(KB の作成\)** をクリックして、ナレッジ ベースを作成します。
1. ナレッジ ベースを**保存してトレーニング**します。
1. ナレッジ ベースを**公開**します。

QnA Maker アプリが公開されたら、 _[SETTINGS]\(設定\)_ タブを選択し、[Deployment details]\(デプロイの詳細\) まで下にスクロールします。 _Postman_ サンプル HTTP 要求の次の値を書き留めます。

```text
POST /knowledgebases/<knowledge-base-id>/generateAnswer
Host: <your-hostname>  // NOTE - this is a URL ending in /qnamaker.
Authorization: EndpointKey <qna-maker-resource-key>
```

ホスト名の完全な URL 文字列は、"https:// < >.azure.net/qnamaker" のようになります。

これらの値は、次のステップで、`appsettings.json` ファイルまたは `.env` ファイル内で使用されます。

ボットでナレッジ ベースを使用できる状態になりました。

## <a name="add-knowledge-base-information-to-your-bot"></a>ナレッジ ベースの情報をボットに追加する
Bot Framework v4.3 以降、Azure では、ダウンロードされたボットのソース コードの一部として .bot ファイルが提供されなくなりました。 次の手順に従って、CSharp または JavaScript ボットをナレッジ ベースに接続します。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

次の値を appsetting.json ファイルに追加します。

```json
{
  "MicrosoftAppId": "",
  "MicrosoftAppPassword": "",
  "ScmType": "None",
  
  "QnAKnowledgebaseId": "knowledge-base-id",
  "QnAAuthKey": "qna-maker-resource-key",
  "QnAEndpointHostName": "your-hostname" // This is a URL ending in /qnamaker
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

次の値をご自身の .env ファイルに追加します。

```
MicrosoftAppId=""
MicrosoftAppPassword=""
ScmType=None

QnAKnowledgebaseId="knowledge-base-id"
QnAAuthKey="qna-maker-resource-key"
QnAEndpointHostName="your-hostname" // This is a URL ending in /qnamaker
```

---

| フィールド | 値 |
|:----|:----|
| QnAKnowledgebaseId | QnA Maker ポータルで自動的に生成されたナレッジ ベース ID。 |
| QnAAuthKey | QnA Maker ポータルで自動的に生成されエンドポイント キー。 |
| QnAEndpointHostName | QnA Maker ポータルで生成されたホスト URL。 `https://` で始まって `/qnamaker` で終わる完全な URL を使用します。 完全な URL 文字列は、"https:// < >.azure.net/qnamaker" のようになります。 |

編集結果を保存します。

## <a name="update-your-bot-to-query-the-knowledge-base"></a>ナレッジ ベースに対してクエリを実行するようにボットを更新する

自分のナレッジ ベースのサービス情報を読み込むように、初期化コードを更新します。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

1. **Microsoft.Bot.Builder.AI.QnA** NuGet パッケージをプロジェクトに追加します。

   これは、NuGet パッケージ マネージャーまたはコマンド ラインから行うことができます。

   ```cmd
   dotnet add package Microsoft.Bot.Builder.AI.QnA
   ```

   NuGet の詳細については、[NuGet のドキュメント](https://docs.microsoft.com/nuget/#pivot=start&panel=start-all)を参照してください。

1. **Microsoft.Extensions.Configuration** NuGet パッケージをプロジェクトに追加します。

1. **Startup.cs** ファイルに、次の名前空間参照を追加します。

   **Startup.cs**

   ```csharp
   using Microsoft.Bot.Builder.AI.QnA;
   using Microsoft.Extensions.Configuration;
   ```

1. _ConfigureServices_ メソッドを変更し、**appsettings.json** ファイルに定義されるナレッジ ベースに接続された QnAMakerEndpoint を作成します。

   **Startup.cs**

   ```csharp
   // Create QnAMaker endpoint as a singleton
   services.AddSingleton(new QnAMakerEndpoint
   {
      KnowledgeBaseId = Configuration.GetValue<string>($"QnAKnowledgebaseId"),
      EndpointKey = Configuration.GetValue<string>($"QnAAuthKey"),
      Host = Configuration.GetValue<string>($"QnAEndpointHostName")
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
      // First send the user input to your QnA Maker knowledge base
      await AccessQnAMaker(turnContext, cancellationToken);
      ...
   }
   ```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

1. ターミナルまたはコマンド プロンプトを開いて、プロジェクトのルート ディレクトリに移動します。
1. **botbuilder-ai** npm パッケージをプロジェクトに追加します。
   ```shell
   npm i botbuilder-ai
   ```

1. **Index.js** で、// アダプターの作成セクションに続けて、次のコードを追加して QnA Maker サービスを生成するために必要な .env ファイル構成情報を読み取ります。

   **index.js**
   ```javascript
   // Map knowledge base endpoint values from .env file into the required format for `QnAMaker`.
   const configuration = {
      knowledgeBaseId: process.env.QnAKnowledgebaseId,
      endpointKey: process.env.QnAAuthKey,
      host: process.env.QnAEndpointHostName
   };

   ```

1. QnA サービス構成情報を渡すようにボットの構造を更新します。

   **index.js**
   ```javascript
   // Create the main dialog.
   const myBot = new MyBot(configuration, {});
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

1. 最後に、ナレッジ ベースで回答のクエリを実行するように、`onMessage` 関数を更新します。 各ユーザー入力を QnA Maker ナレッジ ベースに渡し、最初の QnA Maker の応答をユーザーに返します。

    **bot.js**

    ```javascript
    this.onMessage(async (context, next) => {
        // send user input to QnA Maker.
        const qnaResults = await this.qnaMaker.getAnswers(context);

        // If an answer was received from QnA Maker, send the answer back to the user.
        if (qnaResults[0]) {
            await context.sendActivity(`QnAMaker returned response: ' ${ qnaResults[0].answer}`);
        }
        else {
            // If no answers were returned from QnA Maker, reply with help.
            await context.sendActivity('No QnA Maker response was returned.'
                + 'This example uses a QnA Maker Knowledge Base that focuses on smart light bulbs. '
                + `Ask the bot questions like "Why won't it turn on?" or "I need help."`);
        }
        await next();
    });
    ```

---

### <a name="test-the-bot-locally"></a>ボットをローカルでテストする

この時点で、ボットはいくつかの質問に回答できるようになっているはずです。 ローカル環境でボットを実行し、エミュレーターで開きます。

![QnA サンプルをテストする](./media/qna-test-bot.png)

## <a name="republish-your-bot"></a>ボットを再公開する

これで、ボットを Azure に再公開できる状態になりました。

> [!IMPORTANT]
> プロジェクト ファイルの zip を作成する前に、必ず適切なフォルダー "_内_" に移動します。 
> - C# ボットの場合は、.csproj ファイルが含まれるフォルダーです。 
> - JS ボットの場合は、app.js ファイルまたは index.js ファイルが含まれるフォルダーです。 
>
> そのフォルダー内ですべてのファイルを選択して zip 圧縮し、そのフォルダー内でコマンドを実行します。
>
> お使いのルート フォルダーの場所が正しくない場合、**ボットは、Azure portal で実行できません**。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)
```cmd
az webapp deployment source config-zip --resource-group "resource-group-name" --name "bot-name-in-azure" --src "c:\bot\mybot.zip"
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

[!INCLUDE [publish snippet](~/includes/deploy/snippet-publish-js.md)]

---

### <a name="test-the-published-bot"></a>公開したボットをテストする

ボットを公開した後、Azure でボットが更新されて開始されるまで、1、2 分待ちます。

エミュレーターを使用してボットの運用エンドポイントをテストするか、または Azure portal を使用して Web チャットでボットをテストします。
いずれの場合も、ローカル環境でテストしたときと同じ動作になるはずです。

## <a name="clean-up-resources"></a>リソースのクリーンアップ

<!-- In the first tutorial, we should tell them to use a new resource group, so that it is easy to clean up resources. We should also mention in this step in the first tutorial not to clean up resources if they are continuing with the sequence. -->

このアプリケーションを引き続き使用しない場合は、次の手順で関連付けられているリソースを削除します。

1. Azure portal で、ボットのリソース グループを開きます。
1. **[リソース グループの削除]** をクリックして、グループとそれに含まれるすべてのリソースを削除します。
1. 確認ウィンドウでリソース グループ名を入力して、 **[削除]** をクリックします。

## <a name="next-steps"></a>次の手順

ボットに機能を追加する方法については、開発ハウツー セクションの**テキスト メッセージの送受信**に関する記事と、その他の記事をご覧ください。
> [!div class="nextstepaction"]
> [テキスト メッセージを送受信する](bot-builder-howto-send-messages.md)
