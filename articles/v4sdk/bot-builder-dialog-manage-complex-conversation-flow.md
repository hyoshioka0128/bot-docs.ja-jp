---
title: ブランチとループを使用して高度な会話フローを作成する - Bot Service
description: Bot Framework SDK でダイアログを使用して複雑な会話フローを管理する方法について説明します。
keywords: 複雑な会話フロー, 繰り返し, ループ, メニュー, ダイアログ, プロンプト, ウォーターフォール, ダイアログ セット
author: JonathanFingold
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 01/30/2020
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: f8854469b131b11d53c90047d438af4e26b06f97
ms.sourcegitcommit: e5bf9a7fa7d82802e40df94267bffbac7db48af7
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/18/2020
ms.locfileid: "77441630"
---
# <a name="create-advanced-conversation-flow-using-branches-and-loops"></a>ブランチとループを使用して高度な会話フローを作成する

[!INCLUDE[applies-to](../includes/applies-to.md)]

ダイアログ ライブラリを使用して、複雑な会話フローを作成できます。
この記事では、分岐およびループする複雑な会話を管理する方法と、ダイアログのさまざまな部分の間で引数を渡す方法について説明します。

## <a name="prerequisites"></a>前提条件

- [ボットの基本][concept-basics]、[状態の管理][concept-state]、[ダイアログ ライブラリ][concept-dialogs]、および[連続して行われる会話フローを実装][simple-dialog]する方法に関する知識。
- 複雑なダイアログ サンプルのコピー ([**C#** ][cs-sample]、[**JavaScript**][js-sample]、または [**Python**][python-sample])。

## <a name="about-this-sample"></a>このサンプルについて

このボットのサンプルでは、ユーザーがサインアップし、一覧から最大 2 つの会社をレビューできます。
ボットは 3 つのコンポーネント ダイアログを使用して、会話フローを管理します。
各コンポーネント ダイアログには、ウォーターフォール ダイアログと、ユーザー入力を収集するために必要なプロンプトが含まれています。
これらのダイアログについては、以下のセクションで詳しく説明します。
会話状態を使用してダイアログを管理し、ユーザー状態を使用してユーザーおよびユーザーがレビューする会社に関する情報を保存します。

ボットはアクティビティ ハンドラーから派生します。 多くのサンプル ボットと同様に、ユーザーに歓迎の意を示し、ダイアログを使用してユーザーからのメッセージを処理し、ターンが終わる前にユーザーと会話状態を保存します。

### <a name="c"></a>[C#](#tab/csharp)

ダイアログを使用するには、**Microsoft.Bot.Builder.Dialogs** NuGet パッケージをインストールします。

![複雑なボットのフロー](./media/complex-conversation-flow.png)

### <a name="javascript"></a>[JavaScript](#tab/javascript)

ダイアログを使用するには、ご自身のプロジェクトで **botbuilder-dialogs** npm パッケージをインストールする必要があります。

![複雑なボットのフロー](./media/complex-conversation-flow-js.png)

### <a name="python"></a>[Python](#tab/python)

ダイアログを使用するには、ご自身のプロジェクトで `pip install botbuilder-dialogs` を実行して **botbuilder-dialogs** PyPI パッケージをインストールする必要があります。

![複雑なボットのフロー](./media/complex-conversation-flow-python.png)

---

## <a name="define-the-user-profile"></a>ユーザー プロファイルを定義する

ユーザー プロファイルには、ダイアログによって収集された情報、ユーザーの名前、年齢、およびレビュー対象として選択された会社が含まれます。

### <a name="c"></a>[C#](#tab/csharp)

**UserProfile.cs**

[!code-csharp[UserProfile class](~/../botbuilder-samples/samples/csharp_dotnetcore/43.complex-dialog/UserProfile.cs?range=8-16)]

### <a name="javascript"></a>[JavaScript](#tab/javascript)

**userProfile.js**

[!code-javascript[UserProfile class](~/../botbuilder-samples/samples/javascript_nodejs/43.complex-dialog/userProfile.js?range=4-12)]

### <a name="python"></a>[Python](#tab/python)

**data_models/user_profile.py**

[!code-python[UserProfile class](~/../botbuilder-samples/samples/python/43.complex-dialog/data_models/user_profile.py?range=7-13)]

---

## <a name="create-the-dialogs"></a>ダイアログを作成する

このボットには、次の 3 つのダイアログが含まれています。

- メイン ダイアログでは、プロセス全体が開始され、収集された情報が要約されます。
- 最上位レベルのダイアログでは、ユーザー情報が収集され、ユーザーの年齢に基づく分岐ロジックが含まれます。
- review-selection ダイアログでは、ユーザーはレビューする会社を繰り返し選択できます。 ループ ロジックを使用してこれを行います。

### <a name="the-main-dialog"></a>メイン ダイアログ

メイン ダイアログには、次の 2 つのステップがあります。

1. 最上位レベルのダイアログを開始します。
1. 最上位レベルのダイアログによって収集されたユーザー プロファイルを取得して要約し、その情報をユーザー状態に保存して、メイン ダイアログの終了を通知します。

#### <a name="c"></a>[C#](#tab/csharp)

**Dialogs\MainDialog.cs**

[!code-csharp[step implementations](~/../botbuilder-samples/samples/csharp_dotnetcore/43.complex-dialog/Dialogs/MainDialog.cs?range=31-50)]

#### <a name="javascript"></a>[JavaScript](#tab/javascript)

**dialogs/mainDialog.js**

[!code-javascript[step implementations](~/../botbuilder-samples/samples/javascript_nodejs/43.complex-dialog/dialogs/mainDialog.js?range=43-55)]

#### <a name="python"></a>[Python](#tab/python)

**dialogs\main_dialog.py**

[!code-python[step implementations](~/../botbuilder-samples/samples/python/43.complex-dialog/dialogs/main_dialog.py?range=29-50)]

---

### <a name="the-top-level-dialog"></a>最上位レベルのダイアログ

最上位レベルのダイアログには、次の 4 つのステップがあります。

1. ユーザー名の入力を求めます。
1. ユーザーの年齢の入力を求めます。
1. ユーザーの年齢に基づいて、review-selection ダイアログを開始するか、次のステップに進みます。
1. 最後に、ユーザーが参加してくれたことに対してお礼を述べ、収集した情報を返します。

最初のステップでは、ダイアログ状態の一部として空のユーザー プロファイルが作成されます。 ダイアログは空のプロファイルで開始され、その進行とともにプロファイルに情報が追加されます。 終了すると、最後のステップで、収集された情報が返されます。

3 番目 (選択の開始) のステップでは、ユーザーの年齢に基づいて会話フローが分岐します。

#### <a name="c"></a>[C#](#tab/csharp)

**Dialogs\TopLevelDialog.cs**

[!code-csharp[step implementations](~/../botbuilder-samples/samples/csharp_dotnetcore/43.complex-dialog/Dialogs/TopLevelDialog.cs?range=39-96&highlight=30-42)]

#### <a name="javascript"></a>[JavaScript](#tab/javascript)

**dialogs/topLevelDialog.js**

[!code-javascript[step implementations](~/../botbuilder-samples/samples/javascript_nodejs/43.complex-dialog/dialogs/topLevelDialog.js?range=32-76&highlight=25-33)]

#### <a name="python"></a>[Python](#tab/python)

**dialogs\top_level_dialog.py**

[!code-python[step implementations](~/../botbuilder-samples/samples/python/43.complex-dialog/dialogs/top_level_dialog.py?range=43-95&highlight=29-38)]

---

### <a name="the-review-selection-dialog"></a>review-selection ダイアログ

review-selection ダイアログには、次の 2 つのステップがあります。

1. ユーザーに対して、レビューする会社を選択するか、`done` を選択して完了するよう求めます。
   - ダイアログが初期情報を使用して開始された場合、その情報はウォーターフォール ステップ コンテキストの _options_ プロパティを通じて使用可能になります。 review-selection ダイアログは、自動的に再起動することができ、これを使用して、レビューする会社をユーザーが複数選択できるようにします。
   - レビューする会社をユーザーが既に選択している場合、その会社は使用可能な選択肢から削除されます。
   - ユーザーがループを早期に終了できるように `done` の選択肢が追加されます。
1. 必要に応じて、このダイアログを繰り返すか終了します。
   - レビューする会社をユーザーが選択した場合は、それを一覧に追加します。
   - ユーザーが 2 つの会社を選択した場合、または終了を選択した場合は、ダイアログを終了し、収集された一覧を返します。
   - それ以外の場合は、ダイアログを再起動し、それを一覧の内容で初期化します。

#### <a name="c"></a>[C#](#tab/csharp)

**Dialogs\ReviewSelectionDialog.cs**

[!code-csharp[step implementations](~/../botbuilder-samples/samples/csharp_dotnetcore/43.complex-dialog/Dialogs/ReviewSelectionDialog.cs?range=42-106&highlight=55-64)]

#### <a name="javascript"></a>[JavaScript](#tab/javascript)

**dialogs/reviewSelectionDialog.js**

[!code-javascript[step implementations](~/../botbuilder-samples/samples/javascript_nodejs/43.complex-dialog/dialogs/reviewSelectionDialog.js?range=33-78&highlight=39-45)]

#### <a name="python"></a>[Python](#tab/python)

**dialogs/review_selection_dialog.py**

[!code-python[step implementations](~/../botbuilder-samples/samples/python/43.complex-dialog/dialogs/review_selection_dialog.py?range=42-99&highlight=51-58)]

---

## <a name="run-the-dialogs"></a>ダイアログを実行する

_dialog bot_ クラスは、アクティビティ ハンドラーを拡張します。また、ダイアログを実行するためのロジックを含んでいます。
_dialog and welcome bot_ クラスは、ダイアログ ボットを拡張し、ユーザーが会話に参加したときに歓迎の意も示します。

ボットのターン ハンドラーによって、3 つのダイアログで定義された会話フローが繰り返されます。
ユーザーからメッセージを受信すると、次の操作を行います。

1. メイン ダイアログを実行します。
   - ダイアログ スタックが空の場合は、メイン ダイアログが開始されます。
   - それ以外の場合は、ダイアログはまだ処理中であり、アクティブなダイアログが継続されます。
1. 状態を保存します。これにより、ユーザー、会話、およびダイアログの状態に対するすべての更新が保持されます。

### <a name="c"></a>[C#](#tab/csharp)

**Bots\DialogBot.cs**

[!code-csharp[Overrides](~/../botbuilder-samples/samples/csharp_dotnetcore/43.complex-dialog/Bots/DialogBot.cs?range=33-48&highlight=5-7,14-15)]

### <a name="javascript"></a>[JavaScript](#tab/javascript)

**bots/dialogBot.js**

[!code-javascript[onMessage](~/../botbuilder-samples/samples/javascript_nodejs/43.complex-dialog/bots/dialogBot.js?range=24-32&highlight=4-5)]
[!code-javascript[run](~/../botbuilder-samples/samples/javascript_nodejs/43.complex-dialog/bots/dialogBot.js?range=35-44&highlight=7-9)]

### <a name="python"></a>[Python](#tab/python)

**bots/dialog_bot.py**

[!code-python[Overrides](~/../botbuilder-samples/samples/python/43.complex-dialog/bots/dialog_bot.py?range=29-41&highlight=4-6,9-13)]

---

## <a name="register-services-for-the-bot"></a>ボット用のサービスを登録する

必要に応じてサービスを作成および登録します。

- ボット用の基本サービス: アダプターおよびボット実装。
- 状態を管理するためのサービス: ストレージ、ユーザー状態、および会話の状態。
- ボットが使用するルート ダイアログ。

### <a name="c"></a>[C#](#tab/csharp)

**Startup.cs**

[!code-csharp[ConfigureServices](~/../botbuilder-samples/samples/csharp_dotnetcore/43.complex-dialog/Startup.cs?range=18-37)]

### <a name="javascript"></a>[JavaScript](#tab/javascript)

**index.js**

[!code-javascript[ConfigureServices](~/../botbuilder-samples/samples/javascript_nodejs/43.complex-dialog/index.js?range=26-43)]

### <a name="python"></a>[Python](#tab/python)

**app.py**

[!code-python[ConfigureServices](~/../botbuilder-samples/samples/python/43.complex-dialog/app.py?range=29-32)]
[!code-python[ConfigureServices](~/../botbuilder-samples/samples/python/43.complex-dialog/app.py?range=70-77)]

---

> [!NOTE]
> メモリ ストレージはテストにのみ使用され、実稼働を目的としたものではありません。
> 運用環境のボットでは、必ず永続タイプのストレージを使用してください。

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
