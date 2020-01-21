---
title: ブランチとループを使用して高度な会話フローを作成する - Bot Service
description: Bot Framework SDK でダイアログを使用して複雑な会話フローを管理する方法について説明します。
keywords: 複雑な会話フロー, 繰り返し, ループ, メニュー, ダイアログ, プロンプト, ウォーターフォール, ダイアログ セット
author: JonathanFingold
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 11/06/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 701eea560d46acc9d3917716366c509e1032e30f
ms.sourcegitcommit: f8b5cc509a6351d3aae89bc146eaabead973de97
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/09/2020
ms.locfileid: "75798558"
---
# <a name="create-advanced-conversation-flow-using-branches-and-loops"></a>ブランチとループを使用して高度な会話フローを作成する

[!INCLUDE[applies-to](../includes/applies-to.md)]

ダイアログ ライブラリを使用して、単純な会話フローと複雑な会話フローを管理できます。
この記事では、分岐およびループする複雑な会話を管理する方法について説明します。
また、ダイアログのさまざまな部分の間で引数を渡す方法についても説明します。

## <a name="prerequisites"></a>前提条件

- [ボットの基本][concept-basics]、[状態の管理][concept-state]、[ダイアログ ライブラリ][concept-dialogs]、および[連続して行われる会話フローを実装][simple-dialog]する方法に関する知識。
- 複雑なダイアログ サンプルのコピー ([**C#** ][cs-sample]、[**JavaScript**][js-sample]、または [**Python**][python-sample])。

## <a name="about-this-sample"></a>このサンプルについて

このボットのサンプルでは、ユーザーがサインアップし、一覧から最大 2 つの会社をレビューできます。

`DialogAndWelcomeBot` によって `DialogBot` が拡張され、さまざまなアクティビティのハンドラーと、ボットのターン ハンドラーが定義されます。 `DialogBot` ではダイアログが実行されます。

- _run_ メソッドは `DialogBot` によって使用され、ダイアログを開始します。
- `MainDialog` は他の 2 つのダイアログの親で、ダイアログで特定の時間に呼び出されます。 これらのダイアログについては、この記事全体を通して詳しく説明します。

ダイアログは `MainDialog`、`TopLevelDialog`、`ReviewSelectionDialog` のコンポーネント ダイアログに分割され、連携して次の処理を実行します。

- ユーザーの名前と年齢を聞いて、ユーザーの年齢に基づいて "_分岐_" します。
  - ユーザーが若すぎる場合、そのユーザーには会社のレビューを求めません。
  - ユーザーが十分な年齢に達している場合は、ユーザーのレビュー結果の収集を開始します。
    - ユーザーは、レビューする会社を選択できます。
    - ユーザーが会社を選択すると、"_ループ_" して、2 つ目の会社を選択できるようにします。
- 最後に、ユーザーが参加してくれたことに対してお礼を述べます。

複雑な会話の管理には、ウォーターフォール ダイアログと、プロンプトをいくつか使用します。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

![複雑なボットのフロー](./media/complex-conversation-flow.png)

ダイアログを使用するには、ご自身のプロジェクトで **Microsoft.Bot.Builder.Dialogs** NuGet パッケージをインストールする必要があります。

**Startup.cs**

`Startup` でボット用のサービスを登録します。 これらのサービスは、依存関係の挿入を通じてコードの他の部分で使用できます。

- ボット用の基本サービス: 資格情報プロバイダー、アダプター、およびボット実装。
- 状態を管理するためのサービス: ストレージ、ユーザー状態、および会話の状態。
- ボットで使用されるダイアログ。

[!code-csharp[ConfigureServices](~/../botbuilder-samples/samples/csharp_dotnetcore/43.complex-dialog/Startup.cs?range=22-36)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

![複雑なボットのフロー](./media/complex-conversation-flow-js.png)

ダイアログを使用するには、ご自身のプロジェクトで **botbuilder-dialogs** npm パッケージをインストールする必要があります。

**index.js**

次に示す、ボット用の必要なサービスを作成します。

- 基本サービス: アダプターおよびボット実装
- 状態管理: ストレージ、ユーザー状態、および会話状態。
- ダイアログ: ボットは、会話を管理するためにこれらを使用します

[!code-javascript[ConfigureServices](~/../botbuilder-samples/samples/javascript_nodejs/43.complex-dialog/index.js?range=26-65)]

# <a name="pythontabpython"></a>[Python](#tab/python)

![複雑なボットのフロー](./media/complex-conversation-flow-python.png)

ダイアログを使用するには、プロジェクトで `pip install botbuilder-dialogs` を実行して **botbuilder-dialogs** pypi パッケージをインストールする必要があります。

**app.py**

コードの他の部分が必要とするボット用サービスを作成します。

- ボット用の基本サービス: アダプターおよびボット実装。
- 状態を管理するためのサービス: ストレージ、ユーザー状態、および会話の状態。
- ボットで使用されるダイアログ。

[!code-python[ConfigureServices](~/../botbuilder-python/samples/python/43.complex-dialog/app.py?range=28-75)]

---

> [!NOTE]
> メモリ ストレージはテストにのみ使用され、実稼働を目的としたものではありません。
> 運用環境のボットでは、必ず永続タイプのストレージを使用してください。

## <a name="define-a-class-in-which-to-store-the-collected-information"></a>収集した情報を格納するクラスを定義する

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

**UserProfile.cs**

[!code-csharp[UserProfile class](~/../botbuilder-samples/samples/csharp_dotnetcore/43.complex-dialog/UserProfile.cs?range=8-16)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**userProfile.js**

[!code-javascript[UserProfile class](~/../botbuilder-samples/samples/javascript_nodejs/43.complex-dialog/userProfile.js?range=4-12)]

# <a name="pythontabpython"></a>[Python](#tab/python)

**data_models/user_profile.py**

[!code-python[UserProfile class](~/../botbuilder-python/samples/python/43.complex-dialog/data_models/user_profile.py?range=7-13)]


---

## <a name="create-the-dialogs-to-use"></a>使用するダイアログを作成する

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

**Dialogs\MainDialog.cs**

コンポーネント ダイアログ `MainDialog` の定義が完了しました。これにはメイン ステップがいくつか含まれ、ダイアログとプロンプトの動作がこれにより指示されます。 最初のステップでは `TopLevelDialog` が呼び出されます。これについては以下で説明します。

[!code-csharp[step implementations](~/../botbuilder-samples/samples/csharp_dotnetcore/43.complex-dialog/Dialogs/MainDialog.cs?range=31-50&highlight=3)]

**Dialogs\TopLevelDialog.cs**

最初の最上位レベルのダイアログには 4 つのステップがあります。

1. ユーザー名の入力を求めます。
1. ユーザーの年齢の入力を求めます。
1. ユーザーの年齢に基づいて分岐します。
1. 最後に、ユーザーが参加してくれたことに対してお礼を述べ、収集した情報を返します。

最初のステップで、ユーザーのプロファイルをクリアします。これによりダイアログは空のプロファイルで毎回開始されます。 最後のステップでは終了時に情報が返されるため、`AcknowledgementStepAsync` は、その情報をユーザー状態に保存し、最後のステップで使用できるようにメイン ダイアログに返すことで終了します。

[!code-csharp[step implementations](~/../botbuilder-samples/samples/csharp_dotnetcore/43.complex-dialog/Dialogs/TopLevelDialog.cs?range=39-96&highlight=3-4,47-49,56-57)]

**Dialogs\ReviewSelectionDialog.cs**

review-selection ダイアログは最上位レベルのダイアログの `StartSelectionStepAsync` から開始され、2 つのステップがあります。

1. ユーザーに対して、レビューする会社を選択するか、`done` を選択して完了するよう求めます。
1. 必要に応じて、このダイアログを繰り返すか終了します。

この設計では、スタック上で、最上位レベルのダイアログが必ず review-selection ダイアログに先行するため、review-selection ダイアログは、最上位レベル ダイアログの子と見なすことができます。

[!code-csharp[step implementations](~/../botbuilder-samples/samples/csharp_dotnetcore/43.complex-dialog/Dialogs/ReviewSelectionDialog.cs?range=42-106)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**dialogs/mainDialog.js**

コンポーネント ダイアログ `MainDialog` の定義が完了しました。これにはメイン ステップがいくつか含まれ、ダイアログとプロンプトの動作がこれにより指示されます。 最初のステップでは `TopLevelDialog` が呼び出されます。これについては以下で説明します。

[!code-javascript[step implementations](~/../botbuilder-samples/samples/javascript_nodejs/43.complex-dialog/dialogs/mainDialog.js?range=43-55&highlight=2)]

**dialogs/topLevelDialog.js**

最初の最上位レベルのダイアログには 4 つのステップがあります。

1. ユーザー名の入力を求めます。
1. ユーザーの年齢の入力を求めます。
1. ユーザーの年齢に基づいて分岐します。
1. 最後に、ユーザーが参加してくれたことに対してお礼を述べ、収集した情報を返します。

最初のステップで、ユーザーのプロファイルをクリアします。これによりダイアログは空のプロファイルで毎回開始されます。 最後のステップでは終了時に情報が返されるため、`acknowledgementStep` は、その情報をユーザー状態に保存し、最後のステップで使用できるようにメイン ダイアログに返すことで終了します。

[!code-javascript[step implementations](~/../botbuilder-samples/samples/javascript_nodejs/43.complex-dialog/dialogs/topLevelDialog.js?range=32-76&highlight=2-3,37-39,43-44)]

**dialogs/reviewSelectionDialog.js**

review-selection ダイアログは最上位レベルのダイアログの `startSelectionStep` から開始され、2 つのステップがあります。

1. ユーザーに対して、レビューする会社を選択するか、`done` を選択して完了するよう求めます。
1. 必要に応じて、このダイアログを繰り返すか終了します。

この設計では、スタック上で、最上位レベルのダイアログが必ず review-selection ダイアログに先行するため、review-selection ダイアログは、最上位レベル ダイアログの子と見なすことができます。

[!code-javascript[step implementations](~/../botbuilder-samples/samples/javascript_nodejs/43.complex-dialog/dialogs/reviewSelectionDialog.js?range=33-78)]

# <a name="pythontabpython"></a>[Python](#tab/python)

**dialogs\main_dialog.py**

コンポーネント ダイアログ `MainDialog` の定義が完了しました。これにはメイン ステップがいくつか含まれ、ダイアログとプロンプトの動作がこれにより指示されます。 最初のステップでは `TopLevelDialog` が呼び出されます。これについては以下で説明します。

[!code-python[step implementations](~/../botbuilder-python/samples/python/43.complex-dialog/dialogs/main_dialog.py?range=29-50&highlight=4)]

**dialogs\top_level_dialog.py**

最初の最上位レベルのダイアログには 4 つのステップがあります。

1. ユーザー名の入力を求めます。
1. ユーザーの年齢の入力を求めます。
1. ユーザーの年齢に基づいて分岐します。
1. 最後に、ユーザーが参加してくれたことに対してお礼を述べ、収集した情報を返します。

最初のステップで、ユーザーのプロファイルをクリアします。これによりダイアログは空のプロファイルで毎回開始されます。 最後のステップでは終了時に情報が返されるため、`acknowledgementStep` は、その情報をユーザー状態に保存し、最後のステップで使用できるようにメイン ダイアログに返すことで終了します。

[!code-python[step implementations](~/../botbuilder-python/samples/python/43.complex-dialog/dialogs/top_level_dialog.py?range=43-95&highlight=2-3,43-44,52)]

**dialogs/review_selection_dialog.py**

review-selection ダイアログは最上位レベルのダイアログの `startSelectionStep` から開始され、2 つのステップがあります。

1. ユーザーに対して、レビューする会社を選択するか、`done` を選択して完了するよう求めます。
1. 必要に応じて、このダイアログを繰り返すか終了します。

この設計では、スタック上で、最上位レベルのダイアログが必ず review-selection ダイアログに先行するため、review-selection ダイアログは、最上位レベル ダイアログの子と見なすことができます。

[!code-python[step implementations](~/../botbuilder-python/samples/python/43.complex-dialog/dialogs/review_selection_dialog.py?range=42-99)]

---

## <a name="implement-the-code-to-manage-the-dialog"></a>ダイアログを管理するコードを実装する

ボットのターン ハンドラーによって、これらのダイアログで定義された 1 つの会話フローが繰り返されます。
ユーザーからメッセージを受信したらに、次の操作を行います。

1. アクティブなダイアログがあれば、続行します。
   - アクティブなダイアログがない場合は、ユーザー プロファイルをクリアし、最上位レベルのダイアログを開始します。
   - アクティブなダイアログが完了した場合は、返された情報を収集して保存し、ステータス メッセージを表示します。
   - それ以外の場合、アクティブなダイアログはまだプロセスの途中です。その時点で何も行う必要はありません。
1. 会話状態を保存します。これにより、ダイアログの状態に対するすべての更新が保持されます。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

<!-- **DialogExtensions.cs**

In this sample, we've defined a `Run` helper method that we will use to create and access the dialog context.
Since component dialog defines an inner dialog set, we have to create an outer dialog set that's visible to the message handler code, and use that to create a dialog context.

- `dialog` is the main component dialog for the bot.
- `turnContext` is the current turn context for the bot.

[!code-csharp[Run method](~/../botbuilder-samples/samples/csharp_dotnetcore/43.complex-dialog/DialogExtensions.cs?range=13-24)]

-->

**Bots\DialogBot.cs**

メッセージ ハンドラーでは、ダイアログ管理のために `RunAsync` メソッドが呼び出されます。ターンの途中に発生した可能性のある会話およびユーザー状態に対する変更を保存するために、ターン ハンドラーをオーバーライドしました。 基本 `OnTurnAsync` によって `OnMessageActivityAsync` メソッドが呼び出されます。これにより、そのターンの最後に保存呼び出しが確実に行われます。

[!code-csharp[Overrides](~/../botbuilder-samples/samples/csharp_dotnetcore/43.complex-dialog/Bots/DialogBot.cs?range=33-48&highlight=5-7)]

**Bots\DialogAndWelcome.cs**

`DialogAndWelcomeBot` によって上記の `DialogBot` が拡張され、ユーザーが会話に参加したときにウェルカム メッセージが示されます。これは `Startup.cs` で作成されます。

[!code-csharp[On members added](~/../botbuilder-samples/samples/csharp_dotnetcore/43.complex-dialog/Bots/DialogAndWelcome.cs?range=21-38)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**dialogs/mainDialog.js**

このサンプルの `run` メソッドの定義が完了しました。このメソッドは、ダイアログ コンテキストの作成およびアクセスに使用します。
コンポーネント ダイアログによって内部ダイアログ セットが定義されているため、メッセージ ハンドラー コードに表示される外部ダイアログ セットを作成し、それを使用してダイアログ コンテキストを作成する必要があります。

- `turnContext` は、ボットの現在のターン コンテキストです。
- `accessor` は、ダイアログの状態を管理するために作成したアクセサーです。

[!code-javascript[run method](~/../botbuilder-samples/samples/javascript_nodejs/43.complex-dialog/dialogs/mainDialog.js?range=32-41)]

**bots/dialogBot.js**

メッセージ ハンドラーでは、ダイアログ管理のために `run` ヘルパー メソッドが呼び出されます。ターンの途中に発生した可能性のある会話およびユーザー状態に対する変更を保存するために、ターン ハンドラーを実装しています。 `next` を呼び出すことで、基本実装によって `onDialog` メソッドが呼び出されます。これにより、そのターンの最後に保存呼び出しが確実に行われます。

[!code-javascript[Overrides](~/../botbuilder-samples/samples/javascript_nodejs/43.complex-dialog/bots/dialogBot.js?range=24-41)]

**bots/dialogAndWelcomeBot.js**

`DialogAndWelcomeBot` によって上記の `DialogBot` が拡張され、ユーザーが会話に参加したときにウェルカム メッセージが示されます。これは `index.js` で作成されます。

[!code-javascript[On members added](~/../botbuilder-samples/samples/javascript_nodejs/43.complex-dialog/bots/dialogAndWelcomeBot.js?range=10-21)]

# <a name="pythontabpython"></a>[Python](#tab/python)

**bots/dialog_bot.py**

メッセージ ハンドラーでは、ダイアログ管理のために `run_dialog` メソッドが呼び出されます。ターンの途中に発生した可能性のある会話およびユーザー状態に対する変更を保存するために、ターン ハンドラーをオーバーライドしました。 基本 `on_turn` によって `on_message_activity` メソッドが呼び出されます。これにより、そのターンの最後に保存呼び出しが確実に行われます。

[!code-python[Overrides](~/../botbuilder-python/samples/python/43.complex-dialog/bots/dialog_bot.py?range=29-41&highlight=32-34)]

**bots/dialog_and_welcome_bot.py**

`DialogAndWelcomeBot` によって上記の `DialogBot` が拡張され、ユーザーが会話に参加したときにウェルカム メッセージが示されます。これは `config.py` で作成されます。

[!code-python[on_members_added](~/../botbuilder-python/samples/python/43.complex-dialog/bots/dialog_and_welcome_bot.py?range=28-39)]

---

## <a name="branch-and-loop"></a>分岐とループ

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

**Dialogs\TopLevelDialog.cs**

"_最上位レベル_" ダイアログのステップの分岐ロジック サンプルを次に示します。

[!code-csharp[branching logic](~/../botbuilder-samples/samples/csharp_dotnetcore/43.complex-dialog/Dialogs/TopLevelDialog.cs?range=68-80)]

**Dialogs\ReviewSelectionDialog.cs**

"_選択内容の確認_" ダイアログのステップのループ ロジック サンプルを次に示します。

[!code-csharp[looping logic](~/../botbuilder-samples/samples/csharp_dotnetcore/43.complex-dialog/Dialogs/ReviewSelectionDialog.cs?range=96-105)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**dialogs/topLevelDialog.js**

"_最上位レベル_" ダイアログのステップの分岐ロジック サンプルを次に示します。

[!code-javascript[branching logic](~/../botbuilder-samples/samples/javascript_nodejs/43.complex-dialog/dialogs/topLevelDialog.js?range=56-64)]

**dialogs/reviewSelectionDialog.js**

"_選択内容の確認_" ダイアログのステップのループ ロジック サンプルを次に示します。

[!code-javascript[looping logic](~/../botbuilder-samples/samples/javascript_nodejs/43.complex-dialog/dialogs/reviewSelectionDialog.js?range=71-77)]

# <a name="pythontabpython"></a>[Python](#tab/python)

**dialogs/top_level_dialog.py**

"_最上位レベル_" ダイアログのステップの分岐ロジック サンプルを次に示します。

[!code-python[branching logic](~/../botbuilder-python/samples/python/43.complex-dialog/dialogs/top_level_dialog.py?range=71-80)]

**dialogs/review_selection_dialog.py**

"_選択内容の確認_" ダイアログのステップのループ ロジック サンプルを次に示します。

[!code-python[looping logic](~/../botbuilder-python/samples/python/43.complex-dialog/dialogs/review_selection_dialog.py?range=93-98)]

---

## <a name="to-test-the-bot"></a>ボットをテストする

1. [Bot Framework Emulator](https://aka.ms/bot-framework-emulator-readme) をインストールします (まだインストールしていない場合)。
1. ご自身のマシンを使ってローカルでサンプルを実行します。
1. 以下に示すように、エミュレーターを起動し、お使いのボットに接続して、メッセージを送信します。

![複雑なダイアログのサンプルをテストする](~/media/emulator-v4/test-complex-dialog.png)

## <a name="additional-resources"></a>その他のリソース

ダイアログの実装方法の概要については、[連続して行われる会話フローの実装][simple-dialog]に関するページをご覧ください。この記事では、1 つのウォーターフォール ダイアログと、いくつかのプロンプトを使用して、ユーザーに一連の質問を行う単純なやり取りを作成します。

ダイアログ ライブラリには、プロンプト用の基本的な検証が含まれます。 カスタム検証を追加することもできます。 詳細については、[ダイアログ プロンプトを使用したユーザー入力の収集][dialog-prompts]に関するページをご覧ください。

ご自身のダイアログ コードを簡素化し、複数のボットで再利用するために、ダイアログ セットの一部を別のクラスとして定義できます。
詳細については、[ダイアログの再利用][component-dialogs]に関するページをご覧ください。

## <a name="next-steps"></a>次のステップ

> [!div class="nextstepaction"]
> [ダイアログの再利用](bot-builder-compositcontrol.md)

<!-- Footnote-style links -->

[concept-basics]: bot-builder-basics.md
[concept-state]: bot-builder-concept-state.md
[concept-dialogs]: bot-builder-concept-dialog.md

[simple-dialog]: bot-builder-dialog-manage-conversation-flow.md
[dialog-prompts]: bot-builder-prompts.md
[component-dialogs]: bot-builder-compositcontrol.md

[cs-sample]: https://aka.ms/cs-complex-dialog-sample
[js-sample]: https://aka.ms/js-complex-dialog-sample
[python-sample]: https://aka.ms/python-complex-dialog-sample
