---
title: Language Understanding - Bot Service
description: Microsoft Cognitive Services を使って、ボットをより便利で魅力的にするために、人工知能を追加する方法について説明します。
keywords: LUIS, 意図, 認識エンジン, ディスパッチ ツール, QnA, QnA Maker
author: ivorb
ms.author: kamrani
manager: rstand
ms.topic: article
ms.service: bot-service
ms.date: 09/19/2018
ms.reviewer: ''
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 00d2d55c5738712ef91cdee1468deedb3e09c66e
ms.sourcegitcommit: 9d77f3aff9521d819e88efd0fbd19d469b9919e7
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/16/2020
ms.locfileid: "75799076"
---
# <a name="language-understanding"></a>Language Understanding

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

ボットでは、構造化されたガイドのあるスタイルから、自由形式で制約のないスタイルまで、さまざまな会話のスタイルを使用できます。 ボットは、ユーザーの発言に基づいて、会話フローで次に行うことを判断する必要があります。制約のない会話では、ユーザーの応答の幅はさらに広がります。

| ガイド | [ファイル] |
|------|------|
| 私は旅行ボットです。 次のオプションから 1 つ選択してください: フライトを検索する、ホテルを検索する、レンタカーを検索する。 | 旅行の予約のお手伝いができます。 何がしたいですか? |
| 他に必要なことはありますか? [はい] または [いいえ] をクリックしてください。 | 他に必要なことはありますか? |

ユーザーとボットとの対話は、自由形式であることが多く、ボットは文脈から自然に言語を理解する必要があります。 この記事では、ユーザーが望むことを判断し、文章内の概念やエンティティを識別し、最終的には適切なアクションでボットが応答できるようにするために、Language Understanding (LUIS) がどのように役立つかについて説明します。

## <a name="recognize-intent"></a>意図の認識

[LUIS](https://docs.microsoft.com/azure/cognitive-services/luis/home) は、ボットが適切に応答できるように、ユーザーの発言内容からその**意図** (したいこと) を判断することに役立ちます。 特に、ユーザーがボットに話しかけたことが、予測可能な構造または特定のパターンに従っていない場合に、LUIS は役立ちます。 ボットにユーザーが話しかけたり応答を入力したりする会話型ユーザー インターフェイスがある場合は、*発話* (ユーザーからの音声またはテキストの入力) のバリエーションは無限にあります。

たとえば、旅行ボットのユーザーがフライトの予約を依頼する場合に、多くの言い方があることを考えてみてください。

![フライトを予約するためのさまざまな異なる形式の発話](media/cognitive-services-add-bot-language/cognitive-services-luis-utterances.png)

これら発話は、構造が異なっていたり、考えていなかった “フライト” のさまざまなシノニムが含まれている場合があります。 ご自分のボットで、すべての発話に一致するロジックを記述し、しかも同じ単語を含む別の意図と区別するのは容易なことではありません。 さらに、ボットでは、*エンティティ* (場所や時間などの他の重要な単語) を抽出する必要があります。 LUIS は、意図とエンティティを文脈上から識別することで、このプロセスを容易にします。

自然言語入力用のボットを設計する場合は、ボットが認識する必要がある意図とエンティティを決定し、それらをボットが実行するアクションにどのように結び付けるかについて考えます。 [luis.ai](https://www.luis.ai) で、カスタムの意図とエンティティを定義し、それぞれの意図に例を提供し、その中のエンティティにラベルを付けることで、それらの動作を指定します。

ボットは、LUIS で認識される意図を使用して、会話のトピックを判断したり、会話フローを開始します。 たとえば、ユーザーが "フライトを予約したい" と言うと、ボットは BookFlight 意図を検出し、フライトの検索を開始するための会話フローを呼び出します。 LUIS は、目的地や出発日のようなエンティティを検出します。これらはどちらも意図をトリガーする元の発話と、後の会話フローにあります。 ボットは、必要なすべての情報を得ると、ユーザーの意図を満たすことができます。

![ボットとの会話は BookFlight 意図によってトリガーされます。](media/cognitive-services-add-bot-language/cognitive-services-luis-conversation-high-level.png)

### <a name="recognize-intent-in-common-scenarios"></a>一般的なシナリオで意図を認識する

開発時間を節約するため、LUIS には、ボットの一般的なカテゴリの一般的な発話を認識する事前トレーニング済みの言語モデルが用意されています。 

**事前作成済みのドメイン**は、予定、リマインダー、管理、フィットネス、エンターテインメント、通信、予約などの一般的なシナリオで、組み合わせてうまく機能する、事前トレーニング済みのすぐに使用できる意図とエンティティのコレクションです。 **ユーティリティ**事前作成済みドメインは、ボットがキャンセル、確認、ヘルプ、繰り返し、停止などの一般的なタスクを処理するのに役立ちます。 LUIS が提供している[事前作成済みのドメイン](https://docs.microsoft.com/azure/cognitive-services/LUIS/luis-how-to-use-prebuilt-domains)をご覧ください。

**事前作成済みエンティティ**は、ボットが日付、時刻、数値、温度、通貨、地理、年齢などの一般的な情報の種類を認識するのに役立ちます。 LUIS が認識できる型の背景情報については、[事前作成済みエンティティの使用](https://docs.microsoft.com/azure/cognitive-services/LUIS/pre-builtentities)に関するページを参照してください。

## <a name="how-your-bot-gets-messages-from-luis"></a>ボットが LUIS からメッセージを受け取るしくみ

LUIS の設定と接続が完了すると、ボットから LUIS アプリにメッセージを送信できます。LUIS アプリは意図とエンティティを含む JSON 応答を返します。 次に、ボットの "_ターン ハンドラー_" で[ターン コンテキスト](~/v4sdk/bot-builder-basics.md#defining-a-turn)を使用して、LUIS 応答内の意図に基づいて会話フローをルーティングすることができます。 

![意図とエンティティがボットに渡されるしくみ](./media/cognitive-services-add-bot-language/cognitive-services-luis-message-flow-bot-code.png)

ボットで LUIS アプリの使用を開始するには、[Language Understanding での LUIS の使用](https://docs.microsoft.com/azure/bot-service/bot-builder-howto-v4-luis?view=azure-bot-service-4.0)に関するページを参照してください。

## <a name="best-practices-for-language-understanding"></a>Language Understanding のベスト プラクティス

ボットの言語モデルを設計する場合は、次のプラクティスを検討してください。

### <a name="consider-the-number-of-intents"></a>意図の数を検討する

LUIS アプリでは、複数のカテゴリのいずれかに発話を分類することによって意図を認識します。 当然ながら、多数の意図の中から適切なカテゴリを決定すると、それらを区別する LUIS アプリの能力を減らすことができます。

意図の数を減らす 1 つの方法として、階層設計の使用があります。 天気に関連する 3 つの意図、ホーム オートメーションに関連する 3 つの意図、およびその他の 3 つのユーティリティ意図 (ヘルプ、キャンセル、あいさつ) を持つパーソナル アシスタント ボットのケースについて考えてみましょう。 これらの意図をすべて同じ LUIS アプリに配置すると、既に 9 つの意図があることになり、ボットにさらに機能を追加すると、意図の数は最終的に数十になる可能性があります。 代わりに、ディスパッチャー LUIS アプリを使用して、ユーザーの要求が天気、ホーム オートメーション、またはユーティリティに対するものかどうかを判断してから、ディスパッチャーが判断したカテゴリの LUIS アプリを呼び出すことができます。 この場合、各 LUIS アプリは 3 つの意図だけで開始することになります。

### <a name="use-a-none-intent"></a>None 意図を使用する

ボットのユーザーが、予期しない、または現在の会話フローに無関係なことを言う場合がよくあります。 そうしたメッセージを処理するために _None_ 意図が用意されています。

フォールバック、既定値または "上記のいずれでもない" ケースを処理するための意図をトレーニングしない場合、LUIS アプリではメッセージを定義済みの意図に分類することしかできません。 たとえば、`HomeAutomation.TurnOn` と `HomeAutomation.TurnOff` という 2 つの意図を持つ LUIS アプリがあるとします。 意図がこれらしかなく、入力が "金曜日に予定をスケジュール設定する" のような無関係なものである場合、LUIS アプリにはそのメッセージを HomeAutomation.TurnOn または HomeAutomation.TurnOff のいずれかに分類する以外に選択肢がありません。 LUIS アプリにいくつかの例を持つ `None` 意図がある場合、フォールバック ロジックをいくつかボットに提供して、予期しない発話を処理することができます。

`None` 意図は、認識結果を向上させるうえで非常に役に立ちます。 このホーム オートメーション シナリオでは、"金曜日に予定をスケジュール設定する" によって、信頼度の低い `HomeAutomation.TurnOn` 意図が生成される可能性があり、お使いのボットでは、これを拒否する必要があります。 このようなフレーズは、ご自身のモデルの `None` 意図に追加できるため、`None` に適切に解決されます。

### <a name="review-the-utterances-that-luis-app-receives"></a>LUIS アプリが受信する発話を確認する

LUIS アプリでは、ユーザーから送信されたメッセージを確認することでアプリのパフォーマンスを向上させる機能が用意されています。 詳しい手順については、[推奨される発話](https://docs.microsoft.com/azure/cognitive-services/LUIS/label-suggested-utterances)に関するページを参照してください。


## <a name="integrate-multiple-luis-apps-and-qna-services-with-the-dispatch-tool"></a>ディスパッチ ツールを使用して複数の LUIS アプリと QnA サービスを統合する

複数の会話トピックを理解する多目的ボットを作成する場合は、機能ごとにサービスを個別に開発してから、それらを統合することができます。 これらのサービスには、Language Understanding (LUIS) アプリと QnAMaker サービスを含めることができます。 ボットが、複数の LUIS アプリ、複数の QnAMaker サービス、または 2 つの組み合わせを結合できるシナリオの例をいくつか次に示します。

* パーソナル アシスタント ボットでは、ユーザーはさまざまなコマンドを呼び出すことができます。 コマンドの各カテゴリは、個別に開発できる "スキル" を形成し、各スキルには LUIS アプリがあります。
* ボットは、多くのナレッジ ベースを検索して、よくある質問 (FAQ) への回答を見つけます。
* ビジネス向けのボットには、顧客アカウントの作成と発注のための LUIS アプリと、その FAQ 用の QnAMaker サービスもあります。  

### <a name="the-dispatch-tool"></a>ディスパッチ ツール

ディスパッチ ツールは、*ディスパッチ アプリ* (適切な LUIS と QnA Maker サービスにメッセージをルーティングする新しい LUIS アプリ) を作成することで、複数の LUIS アプリと QnA Maker サービスをボットに統合するのに役立ちます。 複数の LUIS アプリと QnA Maker を 1 つのボットに結合する詳しい手順については、[ディスパッチのチュートリアル](./bot-builder-tutorial-dispatch.md)のページを参照してください。

## <a name="use-luis-to-improve-speech-recognition"></a>LUIS を使用して音声認識を向上させる

ユーザーが話しかけるボットの場合、LUIS と統合することで、音声をテキストに変換するときに誤解される可能性がある単語をボットが識別するのに役立ちます。  たとえば、チェスのシナリオで、ユーザーが次のように言ったとします: "Move knight to A 7 (ナイトを A 7 に移動)"。 そのユーザーの意図に対するコンテキストがなければ、その発話は次のように認識される可能性があります: "Move night 287 (ナイトを 287 に移動)"。 チェスの駒と座標を表すエンティティを作成し、それらを発話でラベル付けすることで、それらを識別するための音声認識のコンテキストを提供します。 Bing Speech と統合されている Bot Framework チャネル (Web チャット、Bot Framework エミュレーター、Cortana など) を使用して、[音声認識の準備を有効](https://docs.microsoft.com/azure/bot-service/bot-service-manage-speech-priming?view=azure-bot-service-4.0)にすることができます。  

## <a name="additional-resources"></a>その他のリソース
詳細については、[Cognitive Services](https://docs.microsoft.com/azure/cognitive-services/) のドキュメントを参照してください。
