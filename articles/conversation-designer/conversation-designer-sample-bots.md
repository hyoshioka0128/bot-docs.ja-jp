---
title: サンプル ボットから Conversation Designer ボットを作成する | Microsoft Docs
description: これらのサンプル ボットの 1 つで新しいボットを起動します。
author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ROBOTS: NoIndex, NoFollow
ms.openlocfilehash: 0ebf0d1b90b03789d8a77710631c83f0914099c5
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/18/2018
ms.locfileid: "39302524"
---
# <a name="create-a-conversation-designer-bot-from-a-sample-bots"></a>サンプル ボットから Conversation Designer ボットを作成する

Conversation Designer は、ビジュアル ボット ビルダー、ボットを定義するための宣言型モデル、そしてボットの作成を簡単にするランタイムを提供する強力なフレームワークです。 ボットを簡単に作成する方法の 1 つは、既存のボットからボットを起動することです。

多くのサンプル ボットのうちの 1 つから、または以前にエクスポートした Conversation Designer ボットから、Conversation Designer ボットを作成することができます。 各サンプル ボットは、完全に機能するボットです。 これにより、独自のボットを構築したり、シナリオに合わせてサンプル ボットをカスタマイズしたりするための確実な出発点が得られます。

## <a name="the-sample-bots"></a>サンプル ボット

Conversation Designer には、**サンプル ボット**のセットが付属しています。 これらの**サンプル ボット**のいずれかに基づいてボットの構築を開始することができます。 これらの**サンプル ボット**のほとんどは、完全に機能するボットの小規模なシナリオに対応しています。 これらすべてのサンプル ボットが一緒に機能するところを見たい場合は、**Complete Contoso Cafe ボット**を作成します。 

以下の表は、作成できるボットの種類、各ボットの簡単な説明、および各ボットが対象とするシナリオを示しています。 

| ボット名 | シナリオ | 導入された機能 |
| ---- | ---- | ---- |
| **基本ボット** | いくつかの基本的な機能を備えているボットです。 たとえば、このボットは、新しいユーザーが会話に追加されたときにウェルカム メッセージを表示し、あいさつメッセージに応答し、ユーザーの要求をボットが理解できない場合はフォールバック動作を実行します。 | - [タスク](conversation-designer-tasks.md) <br/>- タスク トリガーとアクション <br/>- [Language Understanding トリガー](conversation-designer-luis.md)<br/>- [スクリプト トリガー](conversation-designer-code-recognizer.md)<br/>- [応答アクション](conversation-designer-reply.md) |
| **エコー ボット** | ユーザーのメッセージをエコーバックする**基本ボット**です。  ボットへのすべての送信が、"あなたの発言 - ...メッセージ..." という形式のメッセージでエコーバックされます。 | - [スクリプト トリガー](conversation-designer-code-recognizer.md)<br/>- コードのオブジェクト コンテキストの解釈<br/>- コード内のエンティティの作成<br>- ボットの応答でのエンティティの使用 |
| **QnA ボット** | [QnAMaker.ai](http://qnamaker.ai) を使用して 1 ターンの質問と回答のエクスペリエンスを遂行できるボットです。 このサンプルを動作させるには、[QnA Maker](http://qnamaker.ai) アカウントをセットアップして、いくつかの質問と回答でそれを訓練しておく必要があります。 次に、この **QnA ボット**を開いて **[Scripts]\(スクリプト\)** ページに移動し、2 つのプレースホルダー (`<INSERT YOUR KB ID>` と `<INSERT YOUR SUBSCRIPTION KEY>`) を QnA Maker の情報に置き換えます。 この 2 つの情報を入力し、ボットを[保存](conversation-designer-save-bot.md)したら、ボットを[テスト](conversation-designer-debug-bot.md)することができます。 | - [スクリプト トリガー](conversation-designer-code-recognizer.md)<br/>- コードからの HTTP 要求の作成<br/>- カスタム NLP/LU サービス (このサンプルでは QnAMaker.ai) への接続 |
| **会話ボット** | 簡単な会話を行い、ユーザーの名前などのコンテキストを記憶することができるボットです。 | - [ダイアログ アクション](conversation-designer-dialogues.md)<br/>- 基本的な[ダイアログの状態](conversation-designer-dialogues.md#dialogue-states)とプロパティ<br/>- [入力要求ダイアログ](conversation-designer-dialogues.md#prompt-state) (入力要求と再要求の動作の定義) の入門<br/>入力要求ダイアログでのラベル付きエンティティによる- [意図の作成](conversation-designer-luis.md#create-a-new-intent)<br/>- ボットの応答での Language Understanding によって生成されたエンティティの使用<br/>- エンド ユーザー エクスペリエンスを向上させるための[カード](conversation-designer-adaptive-cards.md)の使用<br/>- [入力フォーム](conversation-designer-adaptive-cards.md#input-form)へのボット エンティティのバインド<br/>- [応答テンプレート](conversation-designer-response-templates.md) ボット資産の作成および使用 |
| **メモリ ボット** | ユーザーの独自設定をたずね、後の会話でその情報を呼び出すことができるボットです。 "通信設定を指定する" というようなことを言うと、 このボットは通信を "電話" と "テキスト" のどちらで受信するかをたずねます。 また、このボットに Contoso Cafe の場所情報を問い合わせるには、"カフェの場所を探す" または "ワシントン州レドモンドの Contoso Cafe への道順を教えて" などと言います。 ボットは、情報を見つけたら、ユーザーに電話かテキストで情報を伝えます。 <br/><br/>通信設定が既に指定されている場合、ボットはその設定を使用します。そうでない場合は、ユーザーに指定を求めます。 | - [ダイアログ アクション](conversation-designer-dialogues.md)<br/>- [`context.global`](conversation-designer-context-object.md#context-properties) を使用したボット メモリへのデータの保存<br/>- ダイアログで使用するためのコンテキスト情報のメモリからの再呼び出し |
| **テーブル予約ボット** | ユーザーとの複数ターンの会話を行って、ユーザーが Contoso Cafe ショップのテーブルを予約できるようにするボットです。 <br/><br/>テーブルを予約するために、このボットでは、ユーザーから (1) "*パーティーの規模*"、(2) "*日付/時刻*"、(3) "*場所*" という 3 つの情報を取得する必要があります。 <br/><br/>3 つの情報のすべてを 1 つのメッセージにまとめて指定するには、"レドモンドで火曜日の午後 2 時に 4 人用のテーブルを予約したい" などと言います。 3 つの情報のいずれかを指定しないと、ボットが指定を求めます。 | - [ダイアログ アクション](conversation-designer-dialogues.md)<br/>- 他のダイアログの参照による複雑なダイアログ アクション<br/>- 複数のダイアログの一括作成<br/>- フォワード スロット フィルや修正エクスペリエンスなどを管理するために LUIS をフル活用する、ボットのダイアログの設定<br/>- エンド ユーザー エクスペリエンスを向上させるための[カード](conversation-designer-adaptive-cards.md)の使用<br/>- [入力フォーム](conversation-designer-adaptive-cards.md#input-form)へのボット エンティティのバインド<br/>- [条件付き応答テンプレート](conversation-designer-response-templates.md#conditional-response-templates) ボット資産の作成および使用 |
| **検索絞り込みボット** | 以前の会話から持ち越されたコンテキスト情報を使用して検索を絞り込むために役立つボットです。 たとえば、"今週末、シアトル店は営業していますか?"  と言った後、"レントン店はどうですか" と言うと、ボットは 2 つ目の質問でも今週末についての話であることを覚えています。 "どの店が今週金曜の午後 8 時に営業していますか" と言った後、"午後 10 時では?" と言う ような質問を試すこともできます。 | - [ダイアログ アクション](conversation-designer-dialogues.md)<br/>- [`context.global`](conversation-designer-context-object.md#context-properties) を使用したボットのメモリへのデータの保存<br/>- コンテキストを持ち越す種類の対話をユーザーとするためのボット メモリの使用<br/>- ダイアログで使用するためのコンテキスト情報のメモリからの再呼び出し |
| **サンドイッチ注文ボット** | サンドイッチの注文を受ける際にユーザーに入力を求めるさまざまな方法を示すボットです。 たとえば、このボットは、サンドイッチの注文を受けるために役立ちます。 <br/><br/>サンドイッチを注文するには、(1) *サンドイッチのサイズ*、(2) *パンの種類*、(3) *タンパク質のオプション*、(4) *サンドイッチのトッピング* という 4 つの情報が必要です。 <br/><br/>**テーブル予約ボット**のように、4 つのすべての必須情報を 1 つのメッセージで指定することも、"サンドイッチの注文をお願いします"  または "大きいサイズのサンドイッチにしてください" のように、一度に 1 つずつ指定することもできます。 ボットが、注文の処理に必要な残りの情報の入力をユーザーに依頼します。 | - [ダイアログ アクション](conversation-designer-dialogues.md)<br/>- さまざまな方法で情報を収集するための[プロンプト](conversation-designer-response-templates.md)の設定 |
| **Contoso Cafe ボット** | Contoso Cafe 用に作成された完全に機能するボットです。 このボットでは、その他のサンプル ボットで実行できることのすべてを実行することができます。 ウェルカム メッセージと、対話できる豊富なカードが用意されています。 ヘルプ、案内、およびフォールバックのメッセージがサポートされています。 テーブルの予約、カフェの場所の検索、サンドイッチの注文などを支援します。 | - Conversation Designer のすべての機能<br/>- [`context.global`](conversation-designer-context-object.md#context-properties) を使用したボットのメモリへのデータの保存<br/>- [カード](conversation-designer-adaptive-cards.md)を使用した、ユーザー入力に基づく特定のボット タスクのトリガー (ウェルカム メッセージのボタンを持つカードを参照)<br/>- いくつかの異なる[タスク](conversation-designer-tasks.md)を実行できる複雑なボットの構築。 |

## <a name="explore-sample-bots"></a>サンプル ボットを探す

現在使用しているサンプル ボットが、必要なものと異なる場合は、いつでも別のサンプル ボットに切り替えることができます。 すべてのサンプル ボットの一覧については、**[Explore sample bots]\(サンプル ボットを探す\)** のページを参照してください。

> [!WARNING] 
> サンプル ボットをインポートすると、現在のボットがそのサンプル ボットに置き換えられます。 現在のボットのカスタマイズを失いたくない場合は、.zip ファイルに[エクスポート](conversation-designer-export-import-bot.md#export-a-conversation-designer-bot)します。

別のサンプル ボットに切り替えるには、次の手順に従います。
1. 現在のボットで、**[Explore sample bots]\(サンプル ボットを探す\)** をクリックします。 このオプションは、左側のナビゲーション ウィンドウの下部にあります。 
2. **[Sample bots]\(サンプル ボット\)** の一覧でボットを選択し、**[Import]\(インポート\)** をクリックします。
3. 現在のボットの作業内容を保存する場合は、**[Backup and import]\(バックアップとインポート\)** オプションを選択します。 このオプションは、現在のボットをローカル コンピューターに保存してから、新しい**サンプル ボット**をインポートします。 それ以外の場合は、**[Import without backing up]\(バックアップなしのインポート\)** を選択します。

これで、ボットが、選択した新しいサンプル ボットに置き換えられました。 

## <a name="next-step"></a>次のステップ
> [!div class="nextstepaction"]
> [ボットの保存](conversation-designer-save-bot.md)
