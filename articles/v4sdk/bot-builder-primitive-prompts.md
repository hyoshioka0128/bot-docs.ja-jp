---
title: ユーザー入力を収集するために独自のプロンプトを作成する | Microsoft Docs
description: Bot Framework SDK でプリミティブなプロンプトを使用して会話フローを管理する方法について説明します。
keywords: 会話フロー, プロンプト, 会話状態, ユーザー状態, カスタム プロンプト
author: JonathanFingold
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 05/23/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 4ae38a7b4f6a7769f8839fb44a82b3600abc6318
ms.sourcegitcommit: a547192effb705e4c7d82efc16f98068c5ba218b
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/25/2019
ms.locfileid: "75491417"
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

## <a name="ctabcsharp"></a>[C#](#tab/csharp)
![custom-prompts](media/CustomPromptBotSample-Overview.png)

- ボットによって収集されるユーザー情報を表す `UserProfile` クラス。
- ユーザー情報を収集しているときに、会話状態を制御する `ConversationFlow` クラス。
- 会話の進行状況を追跡するための内部 `ConversationFlow.Question` 列挙型。

## <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)
![custom-prompts](media/CustomPromptBotSample-JS-Overview.png)

- ボットによって収集されるユーザー情報を表す `userProfile` クラス。
- ユーザー情報を収集しているときに、会話状態を制御する `conversationFlow` クラス。
- 会話の進行状況を追跡するための内部 `conversationFlow.question` 列挙型。

## <a name="pythontabpython"></a>[Python](#tab/python)
![custom-prompts](media/CustomPromptBotSample-Python-Overview.png)

- ボットによって収集されるユーザー情報を表す `UserProfile` クラス。
- ユーザー情報を収集しているときに、会話状態を制御する `ConversationFlow` クラス。
- 会話の進行状況を追跡するための内部 `ConversationFlow.Question` 列挙型。

---

ユーザー状態では、ユーザーの名前、年齢、および選択した日付が追跡されます。また、会話状態では、ユーザーに質問した内容が追跡されます。
このボットはデプロイする予定がないため、"_メモリ ストレージ_" を使用するように、ユーザー状態と会話情報の両方を構成します。 

会話のフローと入力のコレクションの管理には、ボットのメッセージ ターン ハンドラーと、ユーザーおよび会話状態のプロパティを使用します。 ボットでは、メッセージ ターン ハンドラーの各イテレーション中に受信した状態のプロパティ情報を記録します。

## <a name="create-conversation-and-user-objects"></a>会話およびユーザー オブジェクトを作成する

## <a name="ctabcsharp"></a>[C#](#tab/csharp)

ユーザーおよび会話状態オブジェクトはスタートアップ時に作成され、依存関係がボット コンストラクターに挿入されます。 

**Startup.cs**  
[!code-csharp[Startup](~/../botbuilder-samples/samples/csharp_dotnetcore/44.prompt-users-for-input/Startup.cs?range=28-35)]

**Bots/CustomPromptBot.cs**  
[!code-csharp[custom prompt bot](~/../botbuilder-samples/samples/csharp_dotnetcore/44.prompt-users-for-input/Bots/CustomPromptBot.cs?range=21-28)]

## <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**Index.js** で、状態プロパティとボットを作成し、`run` ボット メソッドを `processActivity` 内から呼び出します。

[!code-javascript[custom prompt bot](~/../botbuilder-samples/samples/javascript_nodejs/44.prompt-for-user-input/index.js?range=33-39)]

[!code-javascript[custom prompt bot](~/../botbuilder-samples/samples/javascript_nodejs/44.prompt-for-user-input/index.js?range=63-69)]

## <a name="pythontabpython"></a>[Python](#tab/python)

**app.py** で、状態プロパティとボットを作成します。

[!code-python[custom prompt bot](~/../botbuilder-python/samples/python/44.prompt-for-user-input/app.py?range=66-72)]

[!code-python[custom prompt bot](~/../botbuilder-python/samples/python/44.prompt-for-user-input/app.py?range=75-76)]

---

## <a name="create-property-accessors"></a>プロパティ アクセサーを作成する

## <a name="ctabcsharp"></a>[C#](#tab/csharp)

最初に、プロパティ アクセサーを作成します。このプロパティ アクセサーにより、`OnMessageActivityAsync` メソッド内の `BotState` へのハンドルが提供されます。 次に、`GetAsync` メソッドを呼び出して、適切に範囲指定されたキーを取得します。

**Bots/CustomPromptBot.cs**  
[!code-csharp[custom prompt bot](~/../botbuilder-samples/samples/csharp_dotnetcore/44.prompt-users-for-input/Bots/CustomPromptBot.cs?range=30-37)]

最後に、`SaveChangesAsync` メソッドを使用してデータを保存します。

[!code-csharp[custom prompt bot](~/../botbuilder-samples/samples/csharp_dotnetcore/44.prompt-users-for-input/Bots/CustomPromptBot.cs?range=41-44)]

## <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

コンストラクターで状態プロパティ アクセサーを作成し、目的の会話に対して、(上記で作成した) 状態管理オブジェクトを設定します。

**bots/customPromptBot.js**  
[!code-javascript[custom prompt bot](~/../botbuilder-samples/samples/javascript_nodejs/44.prompt-for-user-input/bots/customPromptBot.js?range=23-29)]

その後、(次のセクションで説明する) メイン メッセージ ハンドラーの後に実行する 2 つ目のハンドラー `onDialog` を定義します。 この 2 つ目のハンドラーにより、ターンごとに状態が確実に保存されます。

[!code-javascript[custom prompt bot](~/../botbuilder-samples/samples/javascript_nodejs/44.prompt-for-user-input/bots/customPromptBot.js?range=41-48)]

## <a name="pythontabpython"></a>[Python](#tab/python)

コンストラクターで状態プロパティ アクセサーを作成し、目的の会話に対して、(上記で作成した) 状態管理オブジェクトを設定します。

**bots/custom_prompt_bot.py** [!code-python[custom prompt bot](~/../botbuilder-python/samples/python/44.prompt-for-user-input/bots/custom_prompt_bot.py?range=40-44)]

次に、`save_changes()` メソッドを使用してデータを保存します。
[!code-python[custom prompt bot](~/../botbuilder-python/samples/python/44.prompt-for-user-input/bots/custom_prompt_bot.py?range=53-55)]

---

## <a name="the-bots-message-turn-handler"></a>ボットのメッセージ ターン ハンドラー

## <a name="ctabcsharp"></a>[C#](#tab/csharp)

メッセージ アクティビティを処理するために、_SaveChangesAsync()_ を使用して状態を保存する前に、_FillOutUserProfileAsync()_ ヘルパー メソッドを使用します。 完全なコードを次に示します。

**Bots/CustomPromptBot.cs**  
[!code-csharp[custom prompt bot](~/../botbuilder-samples/samples/csharp_dotnetcore/44.prompt-users-for-input/Bots/CustomPromptBot.cs?range=30-44)]

## <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

メッセージ アクティビティを処理するために、会話とユーザー データを設定し、`fillOutUserProfile()` ヘルパー メソッドを使用します。 ターン ハンドラーの完全なコードを次に示します。

**bots/customPromptBot.js**  
[!code-javascript[custom prompt bot](~/../botbuilder-samples/samples/javascript_nodejs/44.prompt-for-user-input/bots/customPromptBot.js?range=31-39)]

## <a name="pythontabpython"></a>[Python](#tab/python)
メッセージ アクティビティを処理するために、会話とユーザー データを設定し、`_fill_out_user_profile` ヘルパー メソッドを使用します。 ターン ハンドラーの完全なコードを次に示します。

**bots/custom_prompt_bot.py** [!code-python[custom prompt bot](~/../botbuilder-python/samples/python/44.prompt-for-user-input/bots/custom_prompt_bot.py?range=46-55)]

---

## <a name="filling-out-the-user-profile"></a>ユーザー プロファイルを入力する

まず情報を収集します。 それぞれに同様のインターフェイスが用意されています。

- 戻り値は、入力が、この質問に対して有効な回答であるかどうか示しています。
- 検証に合格すると、解析および正規化された値が生成され、保存されます。
- 検証に失敗した場合はメッセージが生成され、ボットはこれを使用して情報を再度求めることができます。

 次のセクションでは、ユーザー入力を解析して検証するヘルパー メソッドを定義します。

## <a name="ctabcsharp"></a>[C#](#tab/csharp)

**Bots/CustomPromptBot.cs**  
[!code-csharp[custom prompt bot](~/../botbuilder-samples/samples/csharp_dotnetcore/44.prompt-users-for-input/Bots/CustomPromptBot.cs?range=46-103)]

## <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**bots/customPromptBot.js**  
[!code-javascript[custom prompt bot](~/../botbuilder-samples/samples/javascript_nodejs/44.prompt-for-user-input/bots/customPromptBot.js?range=52-116)]

## <a name="pythontabpython"></a>[Python](#tab/python)

**bots/custom_prompt_bot.py** [!code-python[custom prompt bot](~/../botbuilder-python/samples/python/44.prompt-for-user-input/bots/custom_prompt_bot.py?range=57-126)]

---

## <a name="parse-and-validate-input"></a>入力を解析して検証する

次の条件を使用して、入力を検証します。

- **name** は、空でない文字列にする必要があります。 空白文字を削除することで正規化します。
- **age** は、18 から 120 の値にする必要があります。 整数を返すことで正規化します。
- **date** は、1 時間以上未来の日付または時刻にする必要があります。
  解析された入力の日付部分のみを返すことで正規化します。

> [!NOTE]
> age と date の入力については、[Microsoft/Recognizers-Text](https://github.com/Microsoft/Recognizers-Text/) ライブラリを使用して、初期解析を実行します。
> サンプル コードは提供されますが、テキスト認識エンジン ライブラリの動作方法については説明されていません。また、これは入力を解析する方法の一例にすぎません。
> これらのライブラリの詳細については、リポジトリの **README** を参照してください。

## <a name="ctabcsharp"></a>[C#](#tab/csharp)

次の検証メソッドをお使いのボットに追加します。

**Bots/CustomPromptBot.cs**  
[!code-csharp[custom prompt bot](~/../botbuilder-samples/samples/csharp_dotnetcore/44.prompt-users-for-input/Bots/CustomPromptBot.cs?range=105-203)]

## <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**bots/customPromptBot.cs**  
[!code-javascript[custom prompt bot](~/../botbuilder-samples/samples/javascript_nodejs/44.prompt-for-user-input/bots/customPromptBot.js?range=118-189)]

## <a name="pythontabpython"></a>[Python](#tab/python)

**bots/custom_prompt_bot.py** [!code-python[custom prompt bot](~/../botbuilder-python/samples/python/44.prompt-for-user-input/bots/custom_prompt_bot.py?range=127-189)]
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
