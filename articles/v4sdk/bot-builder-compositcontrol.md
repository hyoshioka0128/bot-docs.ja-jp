---
title: ダイアログの再利用 - Bot Service
description: Bot Framework SDK でコンポーネント ダイアログを使用して、ご自身のボット ロジックをモジュール化する方法について説明します。
keywords: 複合コントロール、モジュラー ボット ロジック
author: v-ducvo
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 11/05/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 2e40e6a9a8da2e884be0469fb00f0da5f011f22d
ms.sourcegitcommit: f8b5cc509a6351d3aae89bc146eaabead973de97
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/09/2020
ms.locfileid: "75798896"
---
# <a name="reuse-dialogs"></a>ダイアログの再利用

[!INCLUDE[applies-to](../includes/applies-to.md)]

コンポーネント ダイアログを使用すると、大規模なダイアログ セットをより管理しやすい要素に分割して、特定のシナリオを処理する独立したダイアログを作成できます。 各要素には独自のダイアログ セットがあり、その要素外にあるダイアログ セットとの名前の競合を回避しています。

## <a name="prerequisites"></a>前提条件

- [ボットの基本][concept-basics]、[ダイアログ ライブラリ][concept-dialogs]、および[会話を管理][simple-flow]する方法に関する知識。
- マルチターン プロンプト サンプルのコピー ([**C#** ][cs-sample]、[**JavaScript**][js-sample]、または [**Python**][python-sample])。

## <a name="about-the-sample"></a>サンプルについて

マルチターン プロンプト サンプルでは、ウォーターフォール ダイアログ、いくつかのプロンプト、およびコンポーネント ダイアログを使用して、ユーザーに一連の質問を行う単純なインタラクションを作成します。 コードはダイアログを使用して、これらの手順を順番に切り替えます。

| 手順        | プロンプトの種類  |
|:-------------|:-------------|
| 移動手段をユーザーに聞きます | 選択プロンプト |
| 名前をユーザーに聞きます | テキスト プロンプト |
| 年齢を指定するかどうかをユーザーに聞きます | 確認プロンプト |
| "はい" と回答した場合は、年齢を聞きます  | 数値プロンプト。数値を検証し、0 より大きく 150 未満の場合のみ受け入れます。 |
| 収集された情報が正しいかどうかを聞きます | 再利用確認プロンプト |

最後に、"はい" と答えたら、収集された情報を表示します。それ以外の場合は、ユーザー情報が保持されないことをユーザーに通知します。

## <a name="implement-the-component-dialog"></a>コンポーネント ダイアログを実装する

マルチターン プロンプト サンプルでは、_ウォーターフォール ダイアログ_、いくつかの_プロンプト_、および _コンポーネント ダイアログ_を使用して、ユーザーに一連の質問を行う単純なインタラクションを作成します。

コンポーネント ダイアログによって 1 つ以上のダイアログがカプセル化されます。 コンポーネント ダイアログには内部ダイアログ セットがあり、この内部ダイアログ セットに追加したダイアログとプロンプトは独自の ID を持っています。これらの ID は、コンポーネント ダイアログ内からのみ表示できます。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

ダイアログを使用するには、**Microsoft.Bot.Builder.Dialogs** NuGet パッケージをインストールします。

**Dialogs\UserProfileDialog.cs**

ここでは `UserProfileDialog` クラスは、`ComponentDialog` クラスから派生しています。

[!code-csharp[Class](~/../botbuilder-samples/samples/csharp_dotnetcore/05.multi-turn-prompt/Dialogs/UserProfileDialog.cs?range=17)]

コンストラクター内で、`AddDialog` メソッドによって、ダイアログとプロンプトがコンポーネント ダイアログに追加されます。 このメソッドを使用して追加した最初の項目が初期ダイアログとして設定されますが、これは、`InitialDialogId` プロパティを明示的に設定することで変更できます。 コンポーネント ダイアログを開始すると、その _initial dialog_ が開始されます。

[!code-csharp[Constructor](~/../botbuilder-samples/samples/csharp_dotnetcore/05.multi-turn-prompt/Dialogs/UserProfileDialog.cs?range=21-48)]

これは、ウォーターフォール ダイアログの最初のステップの実装です。

[!code-csharp[First step](~/../botbuilder-samples/samples/csharp_dotnetcore/05.multi-turn-prompt/Dialogs/UserProfileDialog.cs?range=50-60)]

ウォーターフォール ダイアログの実装の詳細については、[連続して行われる会話フローを実装する](bot-builder-dialog-manage-complex-conversation-flow.md)方法をご覧ください。

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

ダイアログを使用するには、ご自身のプロジェクトで **botbuilder-dialogs** npm パッケージをインストールする必要があります。

**dialogs/userProfileDialog.js**

ここでは `UserProfileDialog` クラスによって `ComponentDialog` が拡張されます。

[!code-javascript[Class](~/../botbuilder-samples/samples/javascript_nodejs/05.multi-turn-prompt/dialogs/userProfileDialog.js?range=28)]

コンストラクター内で、`AddDialog` メソッドによって、ダイアログとプロンプトがコンポーネント ダイアログに追加されます。 このメソッドを使用して追加した最初の項目が初期ダイアログとして設定されますが、これは、`InitialDialogId` プロパティを明示的に設定することで変更できます。 コンポーネント ダイアログを開始すると、その _initial dialog_ が開始されます。

[!code-javascript[Constructor](~/../botbuilder-samples/samples/javascript_nodejs/05.multi-turn-prompt/dialogs/userProfileDialog.js?range=29-51)]

これは、ウォーターフォール ダイアログの最初のステップの実装です。

[!code-javascript[First step](~/../botbuilder-samples/samples/javascript_nodejs/05.multi-turn-prompt/dialogs/userProfileDialog.js?range=70-77)]

ウォーターフォール ダイアログの実装の詳細については、[連続して行われる会話フローを実装する](bot-builder-dialog-manage-complex-conversation-flow.md)方法をご覧ください。

# <a name="pythontabpython"></a>[Python](#tab/python)

ダイアログを使用するには、ターミナルから `pip install botbuilder-dialogs` と `pip install botbuilder-ai` を実行して **botbuilder-dialogs** および **botbuilder-ai** pypi パッケージをインストールします。

**dialogs/user_profile_dialog.py**

ここでは `UserProfileDialog` クラスによって `ComponentDialog` が拡張されます。

[!code-python[Class](~/../botbuilder-python/samples/python/05.multi-turn-prompt/dialogs/user_profile_dialog.py?range=25)]

コンストラクター内で、`add_dialog` メソッドによって、ダイアログとプロンプトがコンポーネント ダイアログに追加されます。 このメソッドを使用して追加した最初の項目が初期ダイアログとして設定されますが、これは、`initial_dialog_id` プロパティを明示的に設定することで変更できます。 コンポーネント ダイアログを開始すると、その _initial dialog_ が開始されます。

[!code-python[Constructor](~/../botbuilder-python/samples/python/05.multi-turn-prompt/dialogs/user_profile_dialog.py?range=25-57)]

これは、ウォーターフォール ダイアログの最初のステップの実装です。

[!code-python[First step](~/../botbuilder-python/samples/python/05.multi-turn-prompt/dialogs/user_profile_dialog.py?range=59-71)]

ウォーターフォール ダイアログの実装の詳細については、[連続して行われる会話フローを実装する](bot-builder-dialog-manage-complex-conversation-flow.md)方法をご覧ください。

---

実行時、コンポーネント ダイアログに独自のダイアログ スタックが保持されます。 コンポーネント ダイアログが開始すると、以下が行われます。

- インスタンスが作成され、外部ダイアログ スタックに追加されます
- 内部ダイアログ スタックが作成され、その状態が追加されます
- 初期ダイアログが開始され、内部ダイアログ スタックに追加されます。

親コンテキストからは、コンポーネントがアクティブなダイアログのように見えます。 コンポーネント内からは、初期ダイアログがアクティブなダイアログのように見えます。

### <a name="implement-the-rest-of-the-dialog-and-add-it-to-the-bot"></a>ダイアログの残りの部分を実装し、ボットに追加する

コンポーネント ダイアログの追加先の外部ダイアログ セットでは、コンポーネント ダイアログの ID は、それを作成したときに使用したものです。 外部セットでは、コンポーネントは 1 つのダイアログのように見えます。これはプロンプトの動作と似ています。

コンポーネント ダイアログを使用するには、そのインスタンスをボットのダイアログ セットに追加します。これが外部ダイアログ セットです。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

**Bots\DialogBot.cs**

サンプルでは、これはボットの `OnMessageActivityAsync` メソッドから呼び出される `RunAsync` メソッドを使用して行います。

[!code-csharp[OnMessageActivityAsync](~/../botbuilder-samples/samples/csharp_dotnetcore/05.multi-turn-prompt/Bots/DialogBot.cs?range=42-48&highlight=6)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**dialogs/userProfileDialog.js**

サンプルでは、`run` メソッドをユーザー プロファイル ダイアログに追加しました。

[!code-javascript[run method](~/../botbuilder-samples/samples/javascript_nodejs/05.multi-turn-prompt/dialogs/userProfileDialog.js?range=59-68)]

**bots/dialogBot.js**

`run` メソッドは、ボットの `onMessage` メソッドから呼び出されます。

[!code-javascript[onMessage](~/../botbuilder-samples/samples/javascript_nodejs/05.multi-turn-prompt/bots/dialogBot.js?range=24-31&highlight=5)]

# <a name="pythontabpython"></a>[Python](#tab/python)

**helpers/dialog_helper.py**

サンプルでは、`run_dialog` メソッドをユーザー プロファイル ダイアログに追加しました。

[!code-python[First step](~/../botbuilder-python/samples/python/05.multi-turn-prompt/helpers/dialog_helper.py?range=8-19)]

ボットの `on_message_activity` メソッドから呼び出される `run_dialog` メソッド。

**bots/dialog_bot.py** [!code-python[First step](~/../botbuilder-python/samples/python/05.multi-turn-prompt/bots/dialog_bot.py?range=46-51)]

---

## <a name="to-test-the-bot"></a>ボットをテストする

1. [Bot Framework Emulator](https://aka.ms/bot-framework-emulator-readme) をインストールします (まだインストールしていない場合)。
1. ご自身のマシンを使ってローカルでサンプルを実行します。
1. 以下に示すように、エミュレーターを起動し、お使いのボットに接続して、メッセージを送信します。

![マルチターン プロンプト ダイアログの実行サンプル](../media/emulator-v4/multi-turn-prompt.png)

## <a name="additional-information"></a>関連情報

### <a name="how-cancellation-works-for-component-dialogs"></a>コンポーネント ダイアログのキャンセルのしくみ

コンポーネント ダイアログのコンテキストから _cancel all dialogs_ を呼び出すと、コンポーネント ダイアログによって、その内部スタックのすべてのダイアログがキャンセルされた後、終了します。そして外部スタックの次のダイアログに制御が戻ります。

外部コンテキストから _cancel all dialogs_ を呼び出すと、コンポーネントは、外部コンテキストの残りのダイアログとともにがキャンセルされます。

ボットで入れ子になったコンポーネント ダイアログを管理するときは、この点に注意してください。

## <a name="next-steps"></a>次のステップ

ボットを強化して、"ヘルプ"、"キャンセル" など、会話の通常のフローを中断する可能性がある追加入力に対応できるようにします。

> [!div class="nextstepaction"]
> [ユーザーによる割り込みの処理](bot-builder-howto-handle-user-interrupt.md)

<!-- Footnote-style links -->

[concept-basics]: bot-builder-basics.md
[concept-state]: bot-builder-concept-state.md
[concept-dialogs]: bot-builder-concept-dialog.md

[simple-flow]: bot-builder-dialog-manage-conversation-flow.md
[prompting]: bot-builder-prompts.md
[component-dialogs]: bot-builder-compositcontrol.md

[cs-sample]: https://aka.ms/cs-multi-prompts-sample
[js-sample]: https://aka.ms/js-multi-prompts-sample
[python-sample]: https://aka.ms/python-multi-prompts-sample
