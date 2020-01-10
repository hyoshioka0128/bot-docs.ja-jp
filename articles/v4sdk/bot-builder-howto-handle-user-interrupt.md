---
title: ユーザーによる割り込みを処理する | Microsoft Docs
description: ユーザーによる割り込みを処理し、会話フローを転送する方法について説明します。
keywords: 割り込む, 割り込み, トピックの切り替え, 中断
author: ivorb
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 11/05/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 63ecbd2aaef40d5d25c49ef93aa665853ff0054c
ms.sourcegitcommit: a547192effb705e4c7d82efc16f98068c5ba218b
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/25/2019
ms.locfileid: "75491736"
---
# <a name="handle-user-interruptions"></a>ユーザーによる割り込みを処理する

[!INCLUDE[applies-to](../includes/applies-to.md)]

割り込みの処理は、堅牢なボットの重要な側面です。 自分が定義した会話フローの手順に従ってユーザーが操作するとは限りません。 プロセスの途中で質問しようとする場合も、操作を完了せずに、途中でキャンセルしたいだけの場合もあります。 このトピックでは、ボットでのユーザーによる中断の一般的な処理方法をいくつか取り上げます。

## <a name="prerequisites"></a>前提条件

- [ボットの基本][concept-basics]、[状態の管理][concept-state]、[ダイアログ ライブラリ][concept-dialogs]、および[ダイアログを再利用][component-dialogs]する方法に関する知識。
- コア ボット サンプルのコピー ([**CSharp**][cs-sample]、[**JavaScript**][js-sample]、または [**Python**][python-sample])。

## <a name="about-this-sample"></a>このサンプルについて

この記事のサンプルでは、ダイアログを使用してユーザーからフライト情報を取得する航空券予約ボットをモデル化します。 ボットとの会話中、ユーザーはいつでも _help_ コマンドまたは _cancel_ コマンドを発行できます。 ここでは 2 種類の中断を処理します。

- **ターン レベル**: ターン レベルで処理をバイパスしますが、スタック上のダイアログはそのままで、提供された情報は保持されます。 次のターンで、中断された場所から再開されます。 
- **ダイアログ レベル**: 処理が完全にキャンセルされるため、ボットは最初からやり直すことができます。

## <a name="define-and-implement-the-interruption-logic"></a>中断ロジックを定義して実装する

最初に、_help_ と _cancel_ 中断を定義して実装します。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

ダイアログを使用するには、**Microsoft.Bot.Builder.Dialogs** NuGet パッケージをインストールします。

**Dialogs\CancelAndHelpDialog.cs**

最初に、ユーザーによる中断を処理する `CancelAndHelpDialog` クラスを実装します。

[!code-csharp[Class signature](~/../botbuilder-samples/samples/csharp_dotnetcore/13.core-bot/Dialogs/CancelAndHelpDialog.cs?range=12)]

`CancelAndHelpDialog` クラスでは、`OnContinueDialogAsync` メソッドが `InerruptAsync` メソッドを呼び出して、ユーザーが通常のフローを中断をしたかどうかを確認します。 フローが中断された場合は、基底クラス メソッドが呼び出されます。それ以外の場合は、戻り値が `InterruptAsync` から返されます。

[!code-csharp[Overrides](~/../botbuilder-samples/samples/csharp_dotnetcore/13.core-bot/Dialogs/CancelAndHelpDialog.cs?range=22-31)]

ユーザーが「help」と入力すると、`InterrupAsync` メソッドはメッセージを送信し、`DialogTurnResult (DialogTurnStatus.Waiting)` を呼び出します。これは、上部のダイアログがユーザーからの応答を待っていることを示します。 この方法では、ターンについてのみ会話フローが中断され、次のターンでは、中断された場所から再開されます。

ユーザーが「cancel」と入力すると、内部ダイアログ コンテキストで `CancelAllDialogsAsync` が呼び出され、そのダイアログ スタックがクリアされ処理が終了します。この場合、取り消し済み状態になり、結果値は返されません。 `MainDialog` (後で説明します) の場合は、予約ダイアログが終了し、null を返したように見えます。これはユーザーが予約を確認しないことを選択した場合の処理と似ています。

[!code-csharp[Interrupt](~/../botbuilder-samples/samples/csharp_dotnetcore/13.core-bot/Dialogs/CancelAndHelpDialog.cs?range=33-56)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

ダイアログを使用するには、**botbuilder-dialogs** npm パッケージをインストールします。

**dialogs/cancelAndHelpDialog.js**

最初に、ユーザーによる中断を処理する `CancelAndHelpDialog` クラスを実装します。

[!code-javascript[Class signature](~/../botbuilder-samples/samples/javascript_nodejs/13.core-bot/dialogs/cancelAndHelpDialog.js?range=11)]

`CancelAndHelpDialog` クラスでは、`onContinueDialog` メソッドが `interrupt` メソッドを呼び出して、ユーザーが通常のフローを中断をしたかどうかを確認します。 フローが中断された場合は、基底クラス メソッドが呼び出されます。それ以外の場合は、戻り値が `interrupt` から返されます。

[!code-javascript[Overrides](~/../botbuilder-samples/samples/javascript_nodejs/13.core-bot/dialogs/cancelAndHelpDialog.js?range=12-18)]

ユーザーが「help」と入力すると、`interrupt` メソッドはメッセージを送信し、`{ status: DialogTurnStatus.waiting }` オブジェクトを返します。これは、上部のダイアログがユーザーからの応答を待っていることを示します。 この方法では、ターンについてのみ会話フローが中断され、次のターンでは、中断された場所から再開されます。

ユーザーが「cancel」と入力すると、内部ダイアログ コンテキストで `cancelAllDialogs` が呼び出され、そのダイアログ スタックがクリアされ処理が終了します。この場合、取り消し済み状態になり、結果値は返されません。 `MainDialog` (後で説明します) の場合は、予約ダイアログが終了し、null を返したように見えます。これはユーザーが予約を確認しないことを選択した場合の処理と似ています。

[!code-javascript[Interrupt](~/../botbuilder-samples/samples/javascript_nodejs/13.core-bot/dialogs/cancelAndHelpDialog.js??range=20-39)]


## <a name="pythontabpython"></a>[Python](#tab/python)

ダイアログを使用するには、`botbuilder-dialogs` パッケージをインストールし、サンプルの `requirements.txt` ファイルに `botbuilder-dialogs>=4.5.0` などの適切な参照が含まれていることを確認します。 パッケージのインストールの詳細については、サンプル リポジトリの [README](https://github.com/microsoft/botbuilder-python) ファイルを参照してください。
> [!NOTE]
> `pip install botbuilder-dialogs` を実行すると、`botbuilder-core`、`botbulder-connector`、および `botbuilder-schema` もインストールされます。

**dialogs/cancel-and-help-dialog.py**

最初に、ユーザーによる中断を処理する `CancelAndHelpDialog` クラスを実装します。

[!code-python[class signature](~/../botbuilder-python/samples/python/13.core-bot/dialogs/cancel_and_help_dialog.py?range=14)]

`CancelAndHelpDialog` クラスでは、`on_continue_dialog` メソッドが `interrupt` メソッドを呼び出して、ユーザーが通常のフローを中断をしたかどうかを確認します。 フローが中断された場合は、基底クラス メソッドが呼び出されます。それ以外の場合は、戻り値が `InterruptAsync` から返されます。

[!code-python[dialog](~/../botbuilder-python/samples/python/13.core-bot/dialogs/cancel_and_help_dialog.py?range=18-23)]

ユーザーが「*help*」または「 *?* 」と入力すると、`interrupt` メソッドがメッセージを送信し、`DialogTurnResult(DialogTurnStatus.Waiting)` を呼び出します。これは、スタックの上のダイアログがユーザーからの応答を待っていることを示します。 この方法では、ターンについてのみ会話フローが中断され、次のターンでは、中断された場所から再開されます。

ユーザーが「*cancel*」または「*quit*」と入力すると、内部ダイアログ コンテキストで `cancel_all_dialogs()` が呼び出され、そのダイアログ スタックがクリアされ処理が終了します。この場合、取り消し済み状態になり、結果値は返されません。 `MainDialog` (後で説明します) の場合は、予約ダイアログが終了し、null を返したように見えます。これはユーザーが予約を確認しないことを選択した場合の処理と似ています。

[!code-python[interrupt](~/../botbuilder-python/samples/python/13.core-bot/dialogs/cancel_and_help_dialog.py?range=25-47)]

---

## <a name="check-for-interruptions-each-turn"></a>ターンごとに中断を確認する

中断処理クラスの動作方法については説明しました。次は 1 つ戻って、ボットがユーザーから新しいメッセージを受け取ったときの動作を見てみましょう。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

**Dialogs\MainDialog.cs**

新しいメッセージ アクティビティが届くと、ボットは `MainDialog` を実行します。 `MainDialog` は、ユーザーに対して、ボットに役立つ情報を入力するよう要求します。 そして、次のように `MainDialog.ActStepAsync` メソッドで `BookingDialog` を開始して、`BeginDialogAsync` を呼び出します。

[!code-csharp[ActStepAsync](~/../botbuilder-samples/samples/csharp_dotnetcore/13.core-bot/Dialogs/MainDialog.cs?range=58-101&highlight=6,26)]

次に、`MainDialog` クラスの `FinalStepAsync` メソッドで、予約ダイアログが終了し、予約は完了または取り消し済みと見なされます。

[!code-csharp[FinalStepAsync](~/../botbuilder-samples/samples/csharp_dotnetcore/13.core-bot/Dialogs/MainDialog.cs?range=130-150)]

`BookingDialog` のコードは中断処理に直接関連しないため、ここには示されていません。 これは、ユーザーに予約の詳細の入力を求めるときに使用されます。 このコードは、**Dialogs\bookingDialogs.cs** にあります。

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**dialogs/mainDialog.js**

新しいメッセージ アクティビティが届くと、ボットは `MainDialog` を実行します。 `MainDialog` は、ユーザーに対して、ボットに役立つ情報を入力するよう要求します。 そして、次のように `MainDialog.actStep` メソッドで `bookingDialog` を開始して、`beginDialog` を呼び出します。

[!code-javascript[Act step](~/../botbuilder-samples/samples/javascript_nodejs/13.core-bot/dialogs/mainDialog.js?range=71-115&highlight=6,27)]

次に、`MainDialog` クラスの `finalStep` メソッドで、予約ダイアログが終了し、予約は完了または取り消し済みと見なされます。

[!code-javascript[Final step](~/../botbuilder-samples/samples/javascript_nodejs/13.core-bot/dialogs/mainDialog.js?range=142-159)]

`BookingDialog` のコードは中断処理に直接関連しないため、ここには示されていません。 これは、ユーザーに予約の詳細の入力を求めるときに使用されます。 このコードは、**Dialogs\bookingDialogs.js** にあります。

## <a name="pythontabpython"></a>[Python](#tab/python)

**dialogs/main_dialog.py**

新しいメッセージ アクティビティが届くと、ボットは `MainDialog` を実行します。 `MainDialog` は、ユーザーに対して、ボットに役立つ情報を入力するよう要求します。 そして、次のように `MainDialog.act_step` メソッドで `bookingDialog` を開始して、`begin_dialog` を呼び出します。

[!code-python[act step](~/../botbuilder-python/samples/python/13.core-bot/dialogs/main_dialog.py?range=63-100&highlight=4-5,20)]

次に、`MainDialog` クラスの `final_step` メソッドで、予約ダイアログが終了し、予約は完了または取り消し済みと見なされます。

[!code-python[final step](~/../botbuilder-python/samples/python/13.core-bot/dialogs/main_dialog.py?range=102-118)]

---

## <a name="handle-unexpected-errors"></a>予期しないエラーを処理する

次に、発生する可能性があるハンドルされない例外を処理します。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

**AdapterWithErrorHandler.cs**

このサンプルでは、アダプターの `OnTurnError` ハンドラーは、お使いのボットのターン ロジックによってスローされたすべての例外を受け取ります。 例外がスローされると、ハンドラーは、ボットが無効な状態になることでエラー ループに陥らないように、現在の会話の会話状態を削除します。

[!code-csharp[AdapterWithErrorHandler](~/../botbuilder-samples/samples/csharp_dotnetcore/13.core-bot/AdapterWithErrorHandler.cs?range=19-50)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**index.js**

このサンプルでは、アダプターの `onTurnError` ハンドラーは、お使いのボットのターン ロジックによってスローされたすべての例外を受け取ります。 例外がスローされると、ハンドラーは、ボットが無効な状態になることでエラー ループに陥らないように、現在の会話の会話状態を削除します。

[!code-javascript[AdapterWithErrorHandler](~/../botbuilder-samples/samples/javascript_nodejs/13.core-bot/index.js?range=35-57)]

## <a name="pythontabpython"></a>[Python](#tab/python)

**adapter_with_error_handler.py**

このサンプルでは、アダプターの `on_error` ハンドラーは、お使いのボットのターン ロジックによってスローされたすべての例外を受け取ります。 例外がスローされると、ハンドラーは、ボットが無効な状態になることでエラー ループに陥らないように、現在の会話の会話状態を削除します。

[!code-python[adapter_with_error_handler](~/../botbuilder-python/samples/python/13.core-bot/adapter_with_error_handler.py?range=15-54)]

---

## <a name="register-services"></a>サービスを登録する

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

**Startup.cs**

最後に、`Startup.cs` では、ボットは一時的なものとして作成され、ターンごとに、ボットの新しいインスタンスが作成されます。

[!code-csharp[Add transient bot](~/../botbuilder-samples/samples/csharp_dotnetcore/13.core-bot/Startup.cs?range=43-44)]

参照用に、上記のボットを作成するときに呼び出しで使用されるクラスの定義を次に示します。

[!code-csharp[MainDialog signature](~/../botbuilder-samples/samples/csharp_dotnetcore/13.core-bot/Dialogs/MainDialog.cs?range=17)]
[!code-csharp[DialogAndWelcomeBot signature](~/../botbuilder-samples/samples/csharp_dotnetcore/13.core-bot/Bots/DialogAndWelcomeBot.cs?range=16)]
[!code-csharp[DialogBot signature](~/../botbuilder-samples/samples/csharp_dotnetcore/13.core-bot/Bots/DialogBot.cs?range=18)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**index.js**

最後に、`index.js` で、ボットが作成されます。

[!code-javascript[Create bot](~/../botbuilder-samples/samples/javascript_nodejs/13.core-bot/index.js?range=75-78)]

参照用に、上記のボットを作成するときに呼び出しで使用されるクラスの定義を次に示します。

[!code-javascript[MainDialog signature](~/../botbuilder-samples/samples/javascript_nodejs/13.core-bot/dialogs/mainDialog.js?range=11)]
[!code-javascript[DialogAndWelcomeBot signature](~/../botbuilder-samples/samples/javascript_nodejs/13.core-bot/bots/dialogAndWelcomeBot.js?range=8)]
[!code-javascript[DialogBot signature](~/../botbuilder-samples/samples/javascript_nodejs/13.core-bot/bots/dialogBot.js?range=6)]

## <a name="pythontabpython"></a>[Python](#tab/python)

**app.py** 最後に、`app.py` で、ボットが作成されます。

[!code-python[create bot](~/../botbuilder-python/samples/python/13.core-bot/app.py?range=44-48)]

参照用に、ボットを作成するための呼び出しで使用されるクラスの定義を次に示します。

[!code-python[main dialog](~/../botbuilder-python/samples/python/13.core-bot/dialogs/main_dialog.py?range=20)]

[!code-python[dialog and welcome](~/../botbuilder-python/samples/python/13.core-bot/bots/dialog_and_welcome_bot.py?range=21)]

[!code-python[dialog](~/../botbuilder-python/samples/python/13.core-bot/bots/dialog_bot.py?range=9)]

---

## <a name="to-test-the-bot"></a>ボットをテストする

1. [Bot Framework Emulator](https://aka.ms/bot-framework-emulator-readme) をインストールします (まだインストールしていない場合)。
1. ご自身のマシンを使ってローカルでサンプルを実行します。
1. 以下に示すように、エミュレーターを起動し、お使いのボットに接続して、メッセージを送信します。

<!--![test dialog prompt sample](~/media/emulator-v4/test-dialog-prompt.png)-->

## <a name="additional-information"></a>関連情報

- [認証サンプル](https://aka.ms/logout)には、同様の中断処理パターンが使用されるログアウトの処理方法が示されています。

- 何も行わない、または何が起こっているかとユーザーに思わせたままにするのではなく、既定の応答を送信する必要があります。 既定の応答では、ユーザーが本来の会話に戻ることができるように、ボットで認識されるコマンドをユーザーに伝える必要があります。

- ターンの任意の時点で、ターン コンテキストの _responded_ プロパティは、ボットがこのターンでユーザーにメッセージを送信したかどうかを示します。 ターンの終了前に、お使いのボットは、何らかのメッセージをユーザーに送信する必要があります。そのメッセージは、入力の簡単な受信確認でもかまいません。

<!-- Footnote-style links -->

[concept-basics]: bot-builder-basics.md
[concept-state]: bot-builder-concept-state.md
[concept-dialogs]: bot-builder-concept-dialog.md

[using-luis]: bot-builder-howto-v4-luis.md
[using-qna]: bot-builder-howto-qna.md

[simple-flow]: bot-builder-dialog-manage-conversation-flow.md
[prompting]: bot-builder-prompts.md
[component-dialogs]: bot-builder-compositcontrol.md

[cs-sample]: https://aka.ms/cs-core-sample
[js-sample]: https://aka.ms/js-core-sample
[python-sample]: https://aka.ms/bot-core-python-sample-code
