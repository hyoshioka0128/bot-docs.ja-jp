---
title: QnA Maker を使用して質問に回答する | Microsoft Docs
description: ボットで QnA Maker を使用する方法について説明します。
keywords: 質問と回答, QnA, FAQ, QnA Maker
author: ivorb
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 05/23/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: a0dac2fb38f065c0dab6044b5421918e73a0c563
ms.sourcegitcommit: b8b2776552b15590a453267dd0141a25418fbc0c
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/17/2019
ms.locfileid: "72556458"
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

## <a name="next-steps"></a>次の手順

他の Cognitive Services と QnA Maker を組み合わせて、ボットをさらに強力にすることができます。 ディスパッチ ツールを使用すると、ボットで QnA と Language Understanding (LUIS) を結合できます。

> [!div class="nextstepaction"]
> [ディスパッチ ツールを使用して LUIS と QnA サービスを組み合わせる](./bot-builder-tutorial-dispatch.md)
