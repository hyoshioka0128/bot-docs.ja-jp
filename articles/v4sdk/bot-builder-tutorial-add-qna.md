---
title: 質問に回答するボットの Azure Bot Service チュートリアル | Microsoft Docs
description: QnA Maker を使用して質問に回答するボットに関するチュートリアルです。
keywords: QnA Maker, 質問と回答, ナレッジ ベース
author: JonathanFingold
ms.author: kamrani
manager: kamrani
ms.topic: tutorial
ms.service: bot-service
ms.date: 05/23/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: c22e0b8413fc0bcfb4ced330470d88a8a4fbea81
ms.sourcegitcommit: a547192effb705e4c7d82efc16f98068c5ba218b
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/25/2019
ms.locfileid: "75491433"
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
   1. ナレッジ ベースに入力するには、サンプル リポジトリの `BotBuilder-Samples\samples\csharp_dotnetcore\11.qnamaker\CognitiveModels\smartLightFAQ.tsv` ファイルを使用します。 サンプルをダウンロードした場合は、お使いのコンピューターから *smartLightFAQ.tsv* ファイルをアップロードします。
   1. **[Create your kb]\(KB の作成\)** をクリックして、ナレッジ ベースを作成します。
1. ナレッジ ベースを**保存してトレーニング**します。
1. ナレッジ ベースを**公開**します。

QnA Maker アプリが公開されたら、 **[SETTINGS]\(設定\)** タブを選択し、 *[Deployment details]\(デプロイの詳細\)* まで下にスクロールします。 *Postman* HTTP サンプル要求の次の値をコピーします。

```text
POST /knowledgebases/<knowledge-base-id>/generateAnswer
Host: <your-hostname>  // NOTE - this is a URL ending in /qnamaker.
Authorization: EndpointKey <qna-maker-resource-key>
```

ホスト名の完全な URL 文字列は、"https:// < >.azure.net/qnamaker" のようになります。

これらの値は、次のステップで、`appsettings.json` ファイルまたは `.env` ファイル内で使用されます。

ボットでナレッジ ベースを使用できる状態になりました。

## <a name="add-knowledge-base-information-to-your-bot"></a>ナレッジ ベースの情報をボットに追加する
Bot Framework v4.3 以降、Azure では、ダウンロードされたボットのソース コードの一部として .bot ファイルが提供されなくなりました。 次の手順に従って、CSharp、JavaScript、または Python のボットをナレッジ ベースに接続します。

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

# <a name="pythontabpython"></a>[Python](#tab/python)

次の値を `config.py` ファイルに追加します。

```python
class DefaultConfig:
    """ Bot Configuration """
    PORT = 3978
    APP_ID = os.environ.get("MicrosoftAppId", "")
    APP_PASSWORD = os.environ.get("MicrosoftAppPassword", "")

    QNA_KNOWLEDGEBASE_ID = os.environ.get("QnAKnowledgebaseId", "")
    QNA_ENDPOINT_KEY = os.environ.get("QnAEndpointKey", "")
    QNA_ENDPOINT_HOST = os.environ.get("QnAEndpointHostName", "")

```

---

| フィールド | 値 |
|:----|:----|

| QnAKnowledgebaseId | QnA Maker ポータルで自動的に生成されたナレッジ ベース ID。 | | QnAAuthKey (Python の QnAEndpointKey)  | QnA Maker ポータルで自動的に生成されたエンドポイント キー。 | | QnAEndpointHostName | QnA Maker ポータルで生成されたホスト URL。 `https://` で始まって `/qnamaker` で終わる完全な URL を使用します。 完全な URL 文字列は、"https:// < >.azure.net/qnamaker" のようになります。 |

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
      await turnContext.SendActivityAsync(MessageFactory.Text($"Echo: {turnContext.Activity.Text}"), cancellationToken);

      await AccessQnAMaker(turnContext, cancellationToken);
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

# <a name="pythontabpython"></a>[Python](#tab/python)

1. サンプル リポジトリの README ファイルで説明されているようにパッケージをインストールしたことを確認します。
1. 次に示すように、`botbuilder-ai` 参照を `requirements.txt` ファイルに追加します。

   **requirements.txt**
   <!-- Removed version numbers -->
   ```text
      botbuilder-core
      botbuilder-ai
      flask
   ```

   バージョンは異なる場合があることに注意してください。

1. `app.py` ファイルで、次に示すようにボット インスタンスの作成を変更します。

   **app.py**

   ```python
   # Create the main dialog
   BOT = MyBot(APP.config)
   ```

1. 次に示すように、`bot.py` ファイルで `QnAMaker` と `QnAMakerEndpoint` をインポートし、さらに `Config` もインポートします。

   **bot.py**

   ```python
   from flask import Config

   from botbuilder.ai.qna import QnAMaker, QnAMakerEndpoint
   from botbuilder.core import ActivityHandler, MessageFactory, TurnContext
   from botbuilder.schema import ChannelAccount
   ```

1. `config.py` ファイルに指定されている構成パラメーターを 使用して  オブジェクトをインスタンス化するための `config.py`init 関数を追加します。  

   **bot.py**

   ```python
   def __init__(self, config: Config):
      self.qna_maker = QnAMaker(
         QnAMakerEndpoint(
            knowledge_base_id=config["QNA_KNOWLEDGEBASE_ID"],
            endpoint_key=config["QNA_ENDPOINT_KEY"],
            host=config["QNA_ENDPOINT_HOST"],
      )
   )

   ```

1. 回答を得るためにナレッジ ベースに対してクエリを実行するように `on_message_activity` を更新します。 各ユーザー入力を QnA Maker ナレッジ ベースに渡し、最初の QnA Maker の応答をユーザーに返します。

   **bot.py**

   ```python
   async def on_message_activity(self, turn_context: TurnContext):
      # The actual call to the QnA Maker service.
      response = await self.qna_maker.get_answers(turn_context)
      if response and len(response) > 0:
         await turn_context.send_activity(MessageFactory.text(response[0].answer))
      else:
         await turn_context.send_activity("No QnA Maker answers were found.")

   ```

1. 必要に応じて `on_members_added_activity` のウェルカム メッセージを更新します。次に例を示します。

   **bot.py**

   ```python
   await turn_context.send_activity("Hello and welcome to QnA!")
   ```

---

### <a name="test-the-bot-locally"></a>ボットをローカルでテストする

この時点で、ボットはいくつかの質問に回答できるようになっているはずです。 ローカル環境でボットを実行し、エミュレーターで開きます。

![QnA サンプルをテストする](./media/qna-test-bot.png)

## <a name="republish-your-bot"></a>ボットを再公開する
これで、ボットを Azure に再公開できる状態になりました。 プロジェクト フォルダーを zip 圧縮してから、コマンドを実行してボットを Azure にデプロイする必要があります。 詳細については、[ボットのデプロイ](https://docs.microsoft.com/azure/bot-service/bot-builder-deploy-az-cli?view=azure-bot-service-4.0&tabs=csharp)に関する記事を参照してください。 

### <a name="zip-your-project-folder"></a>プロジェクト フォルダーを zip 圧縮する 
[!INCLUDE [zip up code](~/includes/deploy/snippet-zip-code.md)]

<!-- > [!IMPORTANT]
> Before creating a zip of your project files, make sure that you are _in_ the correct folder. 
> - For C# bots, it is the folder that has the .csproj file. 
> - For JS bots, it is the folder that has the app.js or index.js file. 
> - For Python bots, it is the folder that has the app.py file. 
>
> Select all the files and zip them up while in that folder, then run the command while still in that folder.
>
> If your root folder location is incorrect, the **bot will fail to run in the Azure portal**. -->

### <a name="deploy-your-code-to-azure"></a>コードを Azure にデプロイする
[!INCLUDE [deploy code to Azure](~/includes/deploy/snippet-deploy-code-to-az.md)]

<!-- # [C#](#tab/csharp)
```cmd
az webapp deployment source config-zip --resource-group "resource-group-name" --name "bot-name-in-azure" --src "c:\bot\mybot.zip"
```

# [JavaScript](#tab/javascript)

[!INCLUDE [publish snippet](~/includes/deploy/snippet-publish-js.md)]

# [Python](#tab/python)

az webapp deployment source config-zip --resource-group "resource_group_name" --name "unique_bot_name" --src "zi

### Test the published bot

After you publish the bot, give Azure a minute or two to update and start the bot.

Use the Emulator to test the production endpoint for your bot, or use the Azure portal to test the bot in Web Chat.
In either case, you should see the same behavior as you did when you tested it locally.

## Clean up resources

<!-- In the first tutorial, we should tell them to use a new resource group, so that it is easy to clean up resources. We should also mention in this step in the first tutorial not to clean up resources if they are continuing with the sequence. -->

このアプリケーションを引き続き使用しない場合は、次の手順で関連付けられているリソースを削除します。

1. Azure portal で、ボットのリソース グループを開きます。
2. **[リソース グループの削除]** をクリックして、グループとそれに含まれるすべてのリソースを削除します。
3. 確認ウィンドウでリソース グループ名を入力して、 **[削除]** をクリックします。

## <a name="next-steps"></a>次のステップ

ボットに機能を追加する方法については、開発ハウツー セクションの**テキスト メッセージの送受信**に関する記事と、その他の記事をご覧ください。
> [!div class="nextstepaction"]
> [テキスト メッセージを送受信する](bot-builder-howto-send-messages.md)
