---
title: ユーザー入力を収集するために独自のプロンプトを作成する - Bot Service
description: Bot Framework SDK でプリミティブなプロンプトを使用して会話フローを管理する方法について説明します。
keywords: 会話フロー, プロンプト, 会話状態, ユーザー状態, カスタム プロンプト
author: JonathanFingold
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 2/7/2020
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 65aafe3eac9b9b6f023d77c8442bb3a37823789d
ms.sourcegitcommit: 772b9278d95e4b6dd4afccf4a9803f11a4b09e42
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/22/2020
ms.locfileid: "80117554"
---
# <a name="create-your-own-prompts-to-gather-user-input"></a>ユーザー入力を収集するために独自のプロンプトを作成する

[!INCLUDE[applies-to](../includes/applies-to.md)]

多くの場合、ボットとユーザー間の会話では、ユーザーに情報の入力を求め、ユーザーの応答を解析し、その情報に基づいてアクションを実行する必要があります。 お使いのボットは会話のコンテキストを追跡する必要があります。これにより、自身の動作を管理し、以前の質問に対する回答を記憶することができます。 ボットの "*状態*" は、受信メッセージに適切に応答するためにボットが追跡する情報です。

> [!TIP]
> ダイアログ ライブラリにはプロンプトが組み込まれており、ユーザーが使用できる機能がさらに用意されています。 これらのプロンプトの例については、「[連続して行われる会話フローの実装](bot-builder-dialog-manage-conversation-flow.md)」を参照してください。

## <a name="prerequisites"></a>前提条件

- この記事のコードは、"ユーザーに入力を求める" サンプルをベースにしています。 **[C# サンプル](https://aka.ms/cs-primitive-prompt-sample)、[JavaScript サンプル](https://aka.ms/js-primitive-prompt-sample)、または [Python サンプル](https://aka.ms/python-primitive-prompt-sample)** のコピーが必要になります。
- [状態の管理](bot-builder-concept-state.md)と[ユーザー データおよび会話データの保存](bot-builder-howto-v4-state.md)方法に関する知識。

## <a name="about-the-sample-code"></a>サンプル コードについて

サンプル ボットでは、ユーザーに対して一連の質問を行い、その回答の一部を検証して、入力を保存します。 次の図は、ボット、ユーザー プロファイル、および会話フロー クラスの間の関係を示しています。

## <a name="c"></a>[C#](#tab/csharp)

![custom-prompts](media/CustomPromptBotSample-Overview.png)

- ボットによって収集されるユーザー情報を表す `UserProfile` クラス。
- ユーザー情報を収集しているときに、会話状態を制御する `ConversationFlow` クラス。
- 会話の進行状況を追跡するための内部 `ConversationFlow.Question` 列挙型。

## <a name="javascript"></a>[JavaScript](#tab/javascript)

![custom-prompts](media/CustomPromptBotSample-JS-Overview.png)

- ボットによって収集されるユーザー情報を表す `userProfile` クラス。
- ユーザー情報を収集しているときに、会話状態を制御する `conversationFlow` クラス。
- 会話の進行状況を追跡するための内部 `conversationFlow.question` 列挙型。

## <a name="python"></a>[Python](#tab/python)

![custom-prompts](media/CustomPromptBotSample-Python-Overview.png)

- ボットによって収集されるユーザー情報を表す `UserProfile` クラス。
- ユーザー情報を収集しているときに、会話状態を制御する `ConversationFlow` クラス。
- 会話の進行状況を追跡するための内部 `ConversationFlow.Question` 列挙型。

---

ユーザー状態では、ユーザーの名前、年齢、および選択した日付が追跡されます。また、会話状態では、ユーザーに質問した内容が追跡されます。
このボットはデプロイする予定がないため、"_メモリ ストレージ_" を使用するように、ユーザー状態と会話情報の両方を構成します。

会話のフローと入力のコレクションの管理には、ボットのメッセージ ターン ハンドラーと、ユーザーおよび会話状態のプロパティを使用します。 ボットでは、メッセージ ターン ハンドラーの各イテレーション中に受信した状態のプロパティ情報を記録します。

## <a name="create-conversation-and-user-objects"></a>会話およびユーザー オブジェクトを作成する

## <a name="c"></a>[C#](#tab/csharp)

ユーザーおよび会話状態オブジェクトをスタートアップ時に作成し、ボット コンストラクターで依存関係挿入によりそれらを使用します。

**Startup.cs**  
[!code-csharp[Startup.cs](~/../botbuilder-samples/samples/csharp_dotnetcore/44.prompt-users-for-input/Startup.cs?range=28-35)]

**Bots/CustomPromptBot.cs**  
[!code-csharp[constructor](~/../botbuilder-samples/samples/csharp_dotnetcore/44.prompt-users-for-input/Bots/CustomPromptBot.cs?range=21-28)]

## <a name="javascript"></a>[JavaScript](#tab/javascript)

ユーザーおよび会話状態オブジェクトを **index.js** で作成し、ボット コンストラクターでそれらを使用します。

**index.js**  
[!code-javascript[index.js](~/../botbuilder-samples/samples/javascript_nodejs/44.prompt-for-user-input/index.js?range=33-39)]

**bots/customPromptBot.js**  
[!code-javascript[constructor](~/../botbuilder-samples/samples/javascript_nodejs/44.prompt-for-user-input/bots/customPromptBot.js?range=20-22)]
[!code-javascript[constructor](~/../botbuilder-samples/samples/javascript_nodejs/44.prompt-for-user-input/bots/customPromptBot.js?range=27-29)]

## <a name="python"></a>[Python](#tab/python)

ユーザーおよび会話状態オブジェクトを **app.py** で作成し、ボット コンストラクターでそれらを使用します。

**app.py**  
[!code-python[app.py](~/../botbuilder-samples/samples/python/44.prompt-for-user-input/app.py?range=67-73)]

**bots/custom_prompt_bot.py**  
[!code-python[constructor](~/../botbuilder-samples/samples/python/44.prompt-for-user-input/bots/custom_prompt_bot.py?range=29-41)]

---

## <a name="create-property-accessors"></a>プロパティ アクセサーを作成する

## <a name="c"></a>[C#](#tab/csharp)

ユーザー プロファイルと会話フローのプロパティのプロパティ アクセサーを作成し、`GetAsync` を呼び出して、状態からプロパティ値を取得します。

**Bots/CustomPromptBot.cs**  
[!code-csharp[OnMessageActivityAsync](~/../botbuilder-samples/samples/csharp_dotnetcore/44.prompt-users-for-input/Bots/CustomPromptBot.cs?range=30-37)]

ターンを終了する前に、`SaveChangesAsync` を呼び出して、状態の変更をストレージに書き込みます。

[!code-csharp[OnMessageActivityAsync](~/../botbuilder-samples/samples/csharp_dotnetcore/44.prompt-users-for-input/Bots/CustomPromptBot.cs?range=41-44)]

## <a name="javascript"></a>[JavaScript](#tab/javascript)

ユーザー プロファイルと会話フローのプロパティのプロパティ アクセサーを作成し、`get` を呼び出して、状態からプロパティ値を取得します。

**bots/customPromptBot.js**  
[!code-javascript[onMessage](~/../botbuilder-samples/samples/javascript_nodejs/44.prompt-for-user-input/bots/customPromptBot.js?range=31-33)]

ターンを終了する前に、`saveChanges` を呼び出して、状態の変更をストレージに書き込みます。

[!code-javascript[custom prompt bot](~/../botbuilder-samples/samples/javascript_nodejs/44.prompt-for-user-input/bots/customPromptBot.js?range=42-51)]

## <a name="python"></a>[Python](#tab/python)

コンストラクターで状態プロパティ アクセサーを作成し、目的の会話に対して、(上記で作成した) 状態管理オブジェクトを設定します。

**bots/custom_prompt_bot.py**  
[!code-python[on_message_activity](~/../botbuilder-samples/samples/python/44.prompt-for-user-input/bots/custom_prompt_bot.py?range=46-49)]

ターンを終了する前に、`SaveChangesAsync` を呼び出して、状態の変更をストレージに書き込みます。

[!code-python[on_message_activity](~/../botbuilder-samples/samples/python/44.prompt-for-user-input/bots/custom_prompt_bot.py?range=53-55)]

---

## <a name="the-bots-message-turn-handler"></a>ボットのメッセージ ターン ハンドラー

メッセージ アクティビティを処理する場合、メッセージ ハンドラーではヘルパー メソッドを使用して会話を管理し、ユーザーにメッセージを表示します。 ヘルパー メソッドについては、次のセクションで説明します。

## <a name="c"></a>[C#](#tab/csharp)

**Bots/CustomPromptBot.cs**  
[!code-csharp[message handler](~/../botbuilder-samples/samples/csharp_dotnetcore/44.prompt-users-for-input/Bots/CustomPromptBot.cs?range=30-44)]

## <a name="javascript"></a>[JavaScript](#tab/javascript)

**bots/customPromptBot.js**  
[!code-javascript[message handler](~/../botbuilder-samples/samples/javascript_nodejs/44.prompt-for-user-input/bots/customPromptBot.js?range=31-39)]

## <a name="python"></a>[Python](#tab/python)

**bots/custom_prompt_bot.py**  
[!code-python[message handler](~/../botbuilder-samples/samples/python/44.prompt-for-user-input/bots/custom_prompt_bot.py?range=46-55)]

---

## <a name="filling-out-the-user-profile"></a>ユーザー プロファイルを入力する

ボットは、ボットが前のターンで尋ねた質問 (存在する場合) に基づいて、ユーザーに情報の入力を求めます。 入力は検証メソッドを使用して解析されます。

各検証メソッドは、次のような同様の設計に従います。

- 戻り値は、入力が、この質問に対して有効な回答であるかどうか示しています。
- 検証に合格すると、解析および正規化された値が生成され、保存されます。
- 検証に失敗した場合はメッセージが生成され、ボットはこれを使用して情報を再度求めることができます。

検証メソッドについては、以下のセクションで説明します。

## <a name="c"></a>[C#](#tab/csharp)

**Bots/CustomPromptBot.cs**  
[!code-csharp[FillOutUserProfileAsync](~/../botbuilder-samples/samples/csharp_dotnetcore/44.prompt-users-for-input/Bots/CustomPromptBot.cs?range=46-103)]

## <a name="javascript"></a>[JavaScript](#tab/javascript)

**bots/customPromptBot.js**  
[!code-javascript[fillOutUserProfile](~/../botbuilder-samples/samples/javascript_nodejs/44.prompt-for-user-input/bots/customPromptBot.js?range=53-118)]

## <a name="python"></a>[Python](#tab/python)

**bots/custom_prompt_bot.py**  
[!code-python[_fill_out_user_profile](~/../botbuilder-samples/samples/python/44.prompt-for-user-input/bots/custom_prompt_bot.py?range=57-125)]

---

## <a name="parse-and-validate-input"></a>入力を解析して検証する

ボットでは、次の条件を使用して入力が検証されます。

- **name** は、空でない文字列にする必要があります。 空白文字を削除することで正規化されます。
- **age** は、18 から 120 の値にする必要があります。 整数を返すことで正規化されます。
- **date** は、1 時間以上未来の日付または時刻にする必要があります。
  解析された入力の日付部分のみを返すことで正規化されます。

> [!NOTE]
> age と date の入力については、[Microsoft/Recognizers-Text](https://github.com/Microsoft/Recognizers-Text/) ライブラリを使用して、初期解析を実行します。
> サンプル コードは提供されますが、テキスト認識エンジン ライブラリの動作方法については説明されていません。また、これは入力を解析する方法の一例にすぎません。
> これらのライブラリの詳細については、リポジトリの **README** を参照してください。

## <a name="c"></a>[C#](#tab/csharp)

**Bots/CustomPromptBot.cs**  
[!code-csharp[validation methods](~/../botbuilder-samples/samples/csharp_dotnetcore/44.prompt-users-for-input/Bots/CustomPromptBot.cs?range=105-203)]

## <a name="javascript"></a>[JavaScript](#tab/javascript)

**bots/customPromptBot.cs**  
[!code-javascript[validation methods](~/../botbuilder-samples/samples/javascript_nodejs/44.prompt-for-user-input/bots/customPromptBot.js?range=120-190)]

## <a name="python"></a>[Python](#tab/python)

**bots/custom_prompt_bot.py**  
[!code-python[validation methods](~/../botbuilder-samples/samples/python/44.prompt-for-user-input/bots/custom_prompt_bot.py?range=127-189)]

---

## <a name="test-the-bot-locally"></a>ボットをローカルでテストする

ボットをローカルでテストするための [Bot Framework Emulator](https://aka.ms/bot-framework-emulator-readme) をダウンロードし、インス―ルします。

1. ご自身のマシンを使ってローカルでサンプルを実行します。 手順については、README ファイルで [C# サンプル](https://aka.ms/cs-primitive-prompt-sample)、[JS サンプル](https://aka.ms/js-primitive-prompt-sample)、または [Python サンプル](https://aka.ms/python-primitive-prompt-sample)を参照してください。
1. 次に示すように、エミュレーターを使用してテストします。

![primitive-prompts](media/primitive-prompts.png)

## <a name="additional-resources"></a>その他のリソース

[ダイアログ ライブラリ](bot-builder-concept-dialog.md)には、会話の管理に関するさまざまな側面を自動化するクラスが用意されています。

## <a name="next-step"></a>次のステップ

> [!div class="nextstepaction"]
> [連続して行われる会話フローの実装](bot-builder-dialog-manage-conversation-flow.md)
