---
title: QnA Maker を使用して質問に回答する | Microsoft Docs
description: ボットで QnA Maker を使用する方法について説明します。
keywords: 質問と回答, QnA, FAQ, QnA Maker
author: ivorb
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 11/01/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 9ac71659e6420f5181aa7e332d8e5806f1edc348
ms.sourcegitcommit: 4751c7b8ff1d3603d4596e4fa99e0071036c207c
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/02/2019
ms.locfileid: "73441545"
---
# <a name="use-qna-maker-to-answer-questions"></a>QnA Maker を使用して質問に回答する

[!INCLUDE[applies-to](../includes/applies-to.md)]

QnA Maker は、データに基づく会話の質疑応答レイヤーを提供します。 これにより、お使いのボットが QnA Maker に質問を送信し、回答を受け取ることができます。その際、その質問の意図を解析および解釈する必要はありません。 

お客様独自の QnA Maker サービスを作成するときの基本的な要件の 1 つは、質問と回答でサービスをシードすることです。 多くの場合、質問と回答は FAQ などのドキュメントに既に存在していますが、質問に対する回答をカスタマイズして、より自然な会話にしたいこともあります。 

## <a name="prerequisites"></a>前提条件

- この記事のコードは、QnA Maker サンプルをベースにしています。 そのコピー ( **[C#](https://aka.ms/cs-qna) または [JavaScript](https://aka.ms/js-qna-sample)** ) が必要になります。
- [QnA Maker](https://www.qnamaker.ai/) アカウント
- [ボットの基本](bot-builder-basics.md)、[QnA Maker](https://docs.microsoft.com/azure/cognitive-services/qnamaker/overview/overview)、および[ボット リソースの管理](bot-file-basics.md)に関する知識。

## <a name="about-this-sample"></a>このサンプルについて

QnA Maker を利用するボットの場合は、最初に [QnA Maker](https://www.qnamaker.ai/) でナレッジ ベースを作成する必要があります。これについては、次のセクションで説明します。 これによりボットは QnA Maker にユーザーのクエリを送信できるようになり、QnA Maker は、その質問に最適な回答を応答で提供します。

## <a name="ctabcs"></a>[C#](#tab/cs)
![QnABot ロジック フロー](./media/qnabot-logic-flow.png)

ユーザー入力を受け取るたびに、`OnMessageActivityAsync` が呼び出されます。 呼び出された時点で、サンプル コードの `appsetting.json` ファイルに格納されている `_configuration` 情報にアクセスし、事前構成済み QnA Maker ナレッジ ベースに接続するための値を検索します。 

## <a name="javascripttabjs"></a>[JavaScript](#tab/js)
![QnABot JS ロジック フロー](./media/qnabot-js-logic-flow.png)

ユーザー入力を受け取るたびに、`OnMessage` が呼び出されます。 呼び出された時点で、指定された値を使用して事前構成された `qnamaker` コネクタに、サンプル コードの `.env` ファイルからアクセスします。  qnamaker メソッド `getAnswers` は、お使いのボットを、ご自身の外部の QnA Maker ナレッジ ベースに接続します。

---
ユーザーの入力はナレッジ ベースに送信され、返された最適な回答が、該当するユーザーに表示されます。

## <a name="create-a-qna-maker-service-and-publish-a-knowledge-base"></a>QnA Maker サービスの作成とナレッジ ベースの発行
最初の手順で、QnA Maker サービスを作成します。 QnA Maker [ドキュメント](https://docs.microsoft.com/azure/cognitive-services/qnamaker/how-to/set-up-qnamaker-service-azure)に記載されている手順に従って、Azure でサービスを作成します。

次に、サンプル プロジェクトの CognitiveModels フォルダーにある `smartLightFAQ.tsv` ファイルを使用して、ナレッジ ベースを作成します。 ご自身の QnA Maker [ナレッジ ベース](https://docs.microsoft.com/azure/cognitive-services/qnamaker/quickstarts/create-publish-knowledge-base)を作成、トレーニング、および発行する手順は、QnA Maker のドキュメントに記載されています。 次の手順では、ご自身の KB `qna` に名前を付け、`smartLightFAQ.tsv` ファイルを使用して KB を設定します。

> 注: この記事は、お客様のユーザーが作成した QnA Maker ナレッジ ベースにアクセスする際にも使用できます。

## <a name="obtain-values-to-connect-your-bot-to-the-knowledge-base"></a>ボットをナレッジ ベースに接続するための値を取得する
1. [QnA Maker](https://www.qnamaker.ai/) サイトで、目的のナレッジ ベースを選択します。
1. ナレッジ ベースを開いた状態で **[設定]** を選択します。 "_サービス名_" に表示されている値を記録します。 この値は、QnA Maker ポータル インターフェイスを使用するときに、対象となる自分のナレッジ ベースを見つけるのに役立ちます。 この値を使用して、ボット アプリをこのナレッジ ベースに接続するわけではありません。 
1. 下へスクロールして **[デプロイの詳細]** を見つけ、Postman サンプル HTTP 要求にある次の値を記録します。
   - POST /knowledgebases/\<knowledge-base-id>/generateAnswer
   - Host: \<your-hostname> // Full URL ending with /qnamaker
   - Authorization:EndpointKey \<your-endpoint-key>
   
ホスト名の完全な URL 文字列は、"https:// < >.azure.net/qnamaker" のようになります。 この 3 つの値によって、アプリが Azure QnA サービスを介して QnA Maker ナレッジ ベースに接続する際に必要な情報が提供されます。  

## <a name="update-the-settings-file"></a>設定ファイルを更新する

まず、ナレッジ ベースにアクセスするために必要な情報 (ホスト名、エンドポイント キー、ナレッジ ベース ID (kbId) など) を設定ファイルに追加します。 これらは、QnA Maker でナレッジ ベースの **[設定]** タブから保存した値です。 

これを運用環境にデプロイしない場合、アプリ ID とパスワードのフィールドは空白にできます。

> [!NOTE]
> 既存のボット アプリケーションに QnA Maker ナレッジ ベースへのアクセスを追加する場合は、必ず QnA エントリに対して有益なタイトルを追加してください。 このセクション内の "name" 値が、アプリ内からこの情報にアクセスするために必要なキーになります。

## <a name="ctabcs"></a>[C#](#tab/cs)

### <a name="update-your-appsettingsjson-file"></a>お使いの appsettings.json ファイルを更新する

```json
{
  "MicrosoftAppId": "",
  "MicrosoftAppPassword": "",
  
  "QnAKnowledgebaseId": "<knowledge-base-id>",
  "QnAAuthKey": "<your-endpoint-key>",
  "QnAEndpointHostName": "<your-hostname>"
}
```

## <a name="javascripttabjs"></a>[JavaScript](#tab/js)

### <a name="update-your-env-file"></a>お使いの .env ファイルを更新する

```file
MicrosoftAppId=""
MicrosoftAppPassword=""

QnAKnowledgebaseId="<knowledge-base-id>"
QnAAuthKey="<your-endpoint-key>"
QnAEndpointHostName="<your-hostname>"
```

---

## <a name="set-up-the-qna-maker-instance"></a>QnA Maker インスタンスを設定する

最初に、QnA Maker ナレッジ ベースにアクセスするためのオブジェクトを作成します。

## <a name="ctabcs"></a>[C#](#tab/cs)

NuGet パッケージ **Microsoft.Bot.Builder.AI.QnA** がプロジェクトにインストールされていることを確認します。

**QnABot.cs** の `OnMessageActivityAsync` メソッドで、QnAMaker インスタンスを作成します。 `QnABot` クラスも、上記の `appsettings.json` に保存されている接続情報の名前がプルされる場所です。 設定ファイルでナレッジ ベース接続情報に別の名前を選択した場合は、ここで必ず名前を更新して、指定した名前を反映させます。

**Bots/QnABot.cs**  
[!code-csharp[qna connection](~/../botbuilder-samples/samples/csharp_dotnetcore/11.qnamaker/Bots/QnABot.cs?range=32-37)]

## <a name="javascripttabjs"></a>[JavaScript](#tab/js)

お使いのプロジェクトに対して npm パッケージ **botbuilder ai** がインストールされていることを確認します。

このサンプルでは、ボット ロジックのコードは **QnABot.js** ファイルにあります。

**QnABot.js** ファイルで、ご自身の .env ファイルによって提供された接続情報を使用して、QnA Maker サービス _this.qnaMaker_ への接続を確立します。

**QnAMaker.js** [!code-javascript[QnAMaker](~/../botbuilder-samples/samples/javascript_nodejs/11.qnamaker/bots/QnABot.js?range=12-16)]
---

## <a name="calling-qna-maker-from-your-bot"></a>ボットからの QnA Maker の呼び出し

## <a name="ctabcs"></a>[C#](#tab/cs)

QnA Maker からの回答をボットが必要とする場合、ボットのコードから `GetAnswersAsync()` を呼び出して、現在のコンテキストに基づいて適切な回答を取得します。 お客様独自のナレッジ ベースにアクセスする場合は、以下の "_回答が見つかりませんでした_" メッセージを変更して、お客様のユーザーの役に立つ手順を設定します。

**QnABot.cs**  
[!code-csharp[qna connection](~/../botbuilder-samples/samples/csharp_dotnetcore/11.qnamaker/Bots/QnABot.cs?range=43-52)]

## <a name="javascripttabjs"></a>[JavaScript](#tab/js)

**QnABot.js** ファイルで、ユーザーの入力を QnA Maker サービスの `getAnswers` メソッドに渡して、ナレッジ ベースから回答を取得します。 QnA Maker によって応答が返されると、これがユーザーに表示されます。 それ以外の場合、ユーザーは "QnA Maker の回答が見つかりませんでした" というメッセージを受信します。 

**QnABot.js**

[!code-javascript[OnMessage](~/../botbuilder-samples/samples/javascript_nodejs/11.qnamaker/bots/QnABot.js?range=43-59)]

---

## <a name="test-the-bot"></a>ボットのテスト

ご自身のマシンを使ってローカルでサンプルを実行します。 [Bot Framework Emulator](https://github.com/Microsoft/BotFramework-Emulator/blob/master/README.md#download) をインストールします (まだインストールしていない場合)。 詳しい手順については、readme ファイルで [C# サンプル](https://aka.ms/cs-qna)または [Javascript サンプル](https://aka.ms/js-qna-sample)を参照してください。

以下に示すように、エミュレーターを起動し、お使いのボットに接続して、メッセージを送信します。

![QnA サンプルをテストする](../media/emulator-v4/qna-test-bot.png)

## <a name="additional-information"></a>追加情報

### <a name="multi-turn-prompts"></a>マルチターン プロンプト

QnA Maker は、マルチターン プロンプトとも呼ばれるフォローアップ プロンプトをサポートしています。
QnA Maker ナレッジベースにユーザーからの追加の応答が必要な場合、ユーザーにプロンプトを表示するために使用できるコンテキスト情報が QnA Maker から送信されます。 この情報は、QnA Maker サービスへのフォローアップの呼び出しを行うためにも使用されます。
バージョン 4.6 では、Bot Framework SDK にこの機能のサポートが追加されました。

このようなナレッジベースを作成するには、[フォローアップ プロンプトを使用して複数ターンの会話を作成する](https://aka.ms/qnamaker-multiturn-conversation)方法に関する QnA Maker のドキュメントを参照してください。 ボットにマルチターン サポートを組み込む方法については、QnA Maker マルチターン [[**C#** ](https://aka.ms/cs-qna-multiturn) | [**JS**](https://aka.ms/js-qna-multiturn)] サンプルをご覧ください。

<!--TODO: Update code based on final sample 
The following code snippets come from the proof-of-concept **multi-turn QnA Maker prompts** sample for
[**C#**](https://github.com/microsoft/BotBuilder-Samples/tree/master/experimental/qnamaker-prompting/csharp_dotnetcore) and
[**JavaScript**](https://github.com/microsoft/BotBuilder-Samples/tree/master/experimental/qnamaker-prompting/javascript_nodejs).
-->
<!--
#### Sample code

This sample uses a custom QnA dialog to track state for QnA Maker and handle the user's input and QnA Maker's response. When the user sends a message to the bot, the bot treats the input as either an initial query or a response to a follow-up question. The bot starts or continues its QnA dialog, which tracks the QnA Maker context information.

1. When the dialog **starts**, it makes an initial call to the QnA Maker service. QnA Maker context information **is not** included in the call.
1. When the dialog **continues**, it makes a follow-up call to the QnA Maker service. QnA Maker context information included **is** included in the call.
1. In either case, the dialog evaluates the query results.
   - Each QnA Maker result includes either an answer or a follow-up question to the user's initial query. (This sample uses only the first result.)
   - If the result includes follow-up prompts, the dialog prompts the user, saves the context information, and stays on the dialog stack, waiting for additional information from the user.
   - Otherwise, the result represents an answer, the dialog sends the answer and ends.

This sample implements this across two dialog classes:

- The base _functional dialog_ defines the begin, continue, and state logic for the dialog, notably, the _run state machine_ method.
- The derived _QnA dialog_ defines the logic to call QnA Maker, evaluate its response, and send an answer or follow-up question to the user.

#### QnA Maker context

You need to track context information for the QnA Maker service.
The dialog funnels incoming activities through its _run state machine_ method, which:

1. Gets previous QnA Maker context, if any, from state.
1. Calls its _process_ method to call QnA Maker and generates a response for the user.
1. If the result was a follow-up question, sends the question, saves new QnA Maker context to state, and waits for more input on the next turn.
1. If the result was an answer, sends the answer and ends the dialog.

##### [C#](#tab/csharp)

**Dialogs\FunctionDialogBase.cs** defines the **RunStateMachineAsync** method.

```csharp
private async Task<DialogTurnResult> RunStateMachineAsync(DialogContext dialogContext, CancellationToken cancellationToken)
{
     // Get the Process function's current state from the dialog state
     var oldState = GetPersistedState(dialogContext.ActiveDialog);

     // Run the Process function.
     var (newState, output, result) = await ProcessAsync(oldState, dialogContext.Context.Activity).ConfigureAwait(false);

     // If we have output to send then send it.
     foreach (var activity in output)
     {
          await dialogContext.Context.SendActivityAsync(activity).ConfigureAwait(false);
     }

     // If we have new state then we must still be running.
     if (newState != null)
     {
          // Add the state returned from the Process function to the dialog state.
          dialogContext.ActiveDialog.State[FunctionStateName] = newState;

          // Return Waiting indicating this dialog is still in progress.
          return new DialogTurnResult(DialogTurnStatus.Waiting);
     }
     else
     {
          // The Process function indicates it's completed by returning null for the state.
          return await dialogContext.EndDialogAsync(result).ConfigureAwait(false);
     }
}
```

##### [JavaScript](#tab/javascript)

**dialogs/functionDialogBase.js** defines the **runStateMachine** method.

```javascript
async runStateMachine(dc) {

     var oldState = this.getPersistedState(dc.activeDialog);

     var processResult = await this.processAsync(oldState, dc.context.activity);

     var newState = processResult[0];
     var output = processResult[1];
     var result = processResult[2];

     await dc.context.sendActivity(output);

     if(newState != null){
          dc.activeDialog.state[functionStateName] = newState;
          return { status: DialogTurnStatus.waiting };
     }
     else{
          return await dc.endDialog();
     }
}
```

---

#### QnA Maker input and response

You need to provide any previous context information when you call the QnA Maker service.
The dialog handles the call to QnA Maker in its _process_ method, which:

1. Makes the call to QnA Maker, passing in the previous context, if any.
   - This sample uses a _query QnA service_ helper method to format the parameters and make the call.
   - Importantly, if this is a follow-up call to QnA Maker, the _QnA Maker options_ should include values for the _QnA request context_ and the _QnA question ID_.
1. Gets QnA Maker's response and any follow-up prompts from the top result.
1. If the result was a follow-up question:
   - Generates a hero card that contains the question and options for the user's response.
   - Includes the hero card in a message activity.
   - Returns the message activity and the new QnA Maker context.
1. If the result was an answer, returns a message activity that contains the answer.

You can format a follow-up activity in many ways. This sample uses a hero card. However, using suggested actions would be another option.

##### [C#](#tab/csharp)

**Dialogs\QnADialog.cs** defines the **RunStateMachineAsync** method.

```csharp
protected override async Task<(object newState, IEnumerable<Activity> output, object result)> ProcessAsync(object oldState, Activity inputActivity)
{
     Activity outputActivity = null;
     QnABotState newState = null;

     var query = inputActivity.Text;
     var qnaResult = await _qnaService.QueryQnAServiceAsync(query, (QnABotState)oldState);
     var qnaAnswer = qnaResult[0].Answer;
     var prompts = qnaResult[0].Context?.Prompts;

     if (prompts == null || prompts.Length < 1)
     {
          outputActivity = MessageFactory.Text(qnaAnswer);
     }
     else
     {
          // Set bot state only if prompts are found in QnA result
          newState = new QnABotState
          {
          PreviousQnaId = qnaResult[0].Id,
          PreviousUserQuery = query
          };

          outputActivity = CardHelper.GetHeroCard(qnaAnswer, prompts);
     }

     return (newState, new Activity[] { outputActivity }, null);
}
```

##### [JavaScript](#tab/javascript)

**dialogs/qnaDialog.js** defines the **runStateMachine** method.

```javascript
async processAsync(oldState, activity){

     var newState = null;
     var query = activity.text;
     var qnaResult = await QnAServiceHelper.queryQnAService(query, oldState);
     var qnaAnswer = qnaResult[0].answer;

     var prompts = null;
     if(qnaResult[0].context != null){
          prompts = qnaResult[0].context.prompts;
     }

     var outputActivity = null;
     if(prompts == null || prompts.length < 1){
          outputActivity = MessageFactory.text(qnaAnswer);
     }
     else{
          var newState = {
               PreviousQnaId: qnaResult[0].id,
               PreviousUserQuery: query
          }

          outputActivity = CardHelper.GetHeroCard(qnaAnswer, prompts);
     }

     return [newState, outputActivity , null];
}  
```

---
-->
## <a name="next-steps"></a>次の手順

他の Cognitive Services と QnA Maker を組み合わせて、ボットをさらに強力にすることができます。 ディスパッチ ツールを使用すると、ボットで QnA と Language Understanding (LUIS) を結合できます。

> [!div class="nextstepaction"]
> [ディスパッチ ツールを使用して LUIS と QnA サービスを組み合わせる](./bot-builder-tutorial-dispatch.md)
