---
title: 連続して行われる会話フローの実装 | Microsoft Docs
description: Bot Framework SDK でダイアログを使用して単純な会話フローを管理する方法について説明します。
keywords: 単純な会話フロー, 連続して行われる会話フロー, ダイアログ, プロンプト, ウォーターフォール, ダイアログ セット
author: JonathanFingold
ms.author: v-jofing
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 05/23/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 680d9148b463bbb5d10f4a6a06cc7b32b824b66e
ms.sourcegitcommit: ea64a56acfabc6a9c1576ebf9f17ac81e7e2a6b7
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 05/24/2019
ms.locfileid: "66215421"
---
# <a name="implement-sequential-conversation-flow"></a>連続して行われる会話フローの実装

[!INCLUDE[applies-to](../includes/applies-to.md)]

質問を投稿して情報を収集することは、ボットがユーザーとやり取りする主な手段の 1 つです。 ダイアログ ライブラリを使うことで、質問が行いやすくなるだけでなく、応答が検証され、確実に特定のデータ型と一致するように、またはカスタム検証ルールを満たすようになります。

ダイアログ ライブラリを使用して、単純な会話フローと複雑な会話フローを管理できます。 単純なインタラクションでは、ボットは決まった一連のステップを順番に実行していき、最後に会話が終了します。 一般的に、ダイアログは、ボットがユーザーから情報を収集する必要がある場合に役立ちます。 このトピックでは、プロンプトを作成してウォーターフォール ダイアログから呼び出すことで、単純な会話フローを実装する方法について詳しく説明します。 

## <a name="prerequisites"></a>前提条件

- [ボットの基本][concept-basics]、[状態の管理][concept-state]、および[ダイアログ ライブラリ][concept-dialogs]に関する知識。
- **マルチターン プロンプト** サンプルのコピー ([**CSharp**][cs-sample] または [**JavaScript**][js-sample])。

## <a name="about-this-sample"></a>このサンプルについて

マルチターン プロンプト サンプルでは、ウォーターフォール ダイアログ、いくつかのプロンプト、およびコンポーネント ダイアログを使用して、ユーザーに一連の質問を行う単純なインタラクションを作成します。 コードはダイアログを使用して、これらの手順を順番に切り替えます。

| 手順        | プロンプトの種類  |
|:-------------|:-------------| 
| 移動手段をユーザーに聞きます | 選択プロンプト |
| 名前をユーザーに聞きます | テキスト プロンプト |
| 年齢を指定するかどうかをユーザーに聞きます | 確認プロンプト |
| "はい" と回答した場合は、年齢を聞きます  | 数値プロンプト。数値を検証し、0 より大きく 150 未満の場合のみ受け入れます。 |
| 収集された情報が正しいかどうかを聞きます | 再利用確認プロンプト |

最後に、"はい" と答えたら、収集された情報を表示します。それ以外の場合は、ユーザー情報が保持されないことをユーザーに通知します。

## <a name="create-the-main-dialog"></a>メイン ダイアログを作成する

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

ダイアログを使用するには、**Microsoft.Bot.Builder.Dialogs** NuGet パッケージをインストールします。

ボットは `UserProfileDialog` を介してユーザーと対話します。 ボットの `DialogBot` クラスを作成するときに、`UserProfileDialog` をメイン ダイアログとして設定します。 その後、ボットは `Run` ヘルパー メソッドを使用して、ダイアログにアクセスします。

![ユーザー プロファイル ダイアログ](media/user-profile-dialog.png)

**Dialogs\UserProfileDialog.cs**

最初に、`ComponentDialog` クラスから派生し、6 つのステップで構成される `UserProfileDialog` を作成します。

`UserProfileDialog` コンストラクターで、ウォーターフォール ステップ、プロンプト、およびウォーターフォール ダイアログを作成し、ダイアログ セットに追加します。 プロンプトは、それが使用されるダイアログ セットに追加する必要があります。

[!code-csharp[Constructor snippet](~/../botbuilder-samples/samples/csharp_dotnetcore/05.multi-turn-prompt/Dialogs/UserProfileDialog.cs?range=22-41)]

次に、ダイアログで使用されるステップを実装します。 プロンプトを使用するには、そのプロンプトをご自身のダイアログのステップから呼び出し、`stepContext.Result` を使用して、次のステップでプロンプトの結果を取得します。 バックグラウンドでは、プロンプトは 2 つのステップから成るダイアログです。 最初のステップでプロンプトは入力を要求します。そして 2 番目のステップで有効な値を返すか、最初からやり直して、有効な入力を受信するまでユーザーに再入力を要求します。

ウォーターフォール ステップからは常に null 以外の `DialogTurnResult` を返す必要があります。 そうしないと、ご自身のダイアログは設計どおりに機能しません。 ここでは、ウォーターフォール ダイアログの `NameStepAsync` に対する実装を示します。

[!code-csharp[Name step](~/../botbuilder-samples/samples/csharp_dotnetcore/05.multi-turn-prompt/Dialogs/UserProfileDialog.cs?range=56-61)]

`AgeStepAsync` で、ユーザー入力を検証できない場合の再試行プロンプトを指定します。入力を検証できない原因は、その入力がプロンプトで解析できない形式であるか、検証基準を満たしていないかのいずれかです。 この状況で、再試行プロンプトが指定されていないと、プロンプトは最初のプロンプト テキストを使用して、ユーザーに再入力を求めます。

[!code-csharp[Age step](~/../botbuilder-samples/samples/csharp_dotnetcore/05.multi-turn-prompt/Dialogs/UserProfileDialog.cs?range=74-93&highlight=10)]

**UserProfile.cs**

ユーザーの移動手段、名前、および年齢は `UserProfile` クラスのインスタンスに保存されています。

[!code-csharp[UserProfile class](~/../botbuilder-samples/samples/csharp_dotnetcore/05.multi-turn-prompt/UserProfile.cs?range=9-16)]

**Dialogs\UserProfileDialog.cs**

最後のステップで、前のウォーターフォール ステップで呼び出されたダイアログによって返された `stepContext.Result` を確認します。 戻り値が true の場合は、ユーザー プロファイル アクセサーを使用して、ユーザー プロファイルを取得し、更新します。 ユーザー プロファイルを取得するには、`GetAsync` メソッドを呼び出して、`userProfile.Transport`、`userProfile.Name`、`userProfile.Age` の各プロパティの値を設定します。 最後に、ダイアログを終了する `EndDialogAsync` を呼び出す前に、ユーザーの情報をまとめます。 ダイアログを終了すると、そのダイアログはダイアログ スタックから取り出され、ダイアログの親に省略可能な結果が返されます。 この親は、終了したばかりのダイアログを開始したダイアログまたはメソッドです。

[!code-csharp[SummaryStepAsync](~/../botbuilder-samples/samples/csharp_dotnetcore/05.multi-turn-prompt/Dialogs/UserProfileDialog.cs?range=108-134&highlight=5-10,25-26)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

ダイアログを使用するには、ご自身のプロジェクトで **botbuilder-dialogs** npm パッケージをインストールする必要があります。

ボットは `UserProfileDialog` を介してユーザーと対話します。 ボットの `DialogBot` を作成するときに、`UserProfileDialog` をメイン ダイアログとして設定します。 その後、ボットは `run` ヘルパー メソッドを使用して、ダイアログにアクセスします。

![ユーザー プロファイル ダイアログ](media/user-profile-dialog-js.png)

**dialogs\userProfileDialog.js**

最初に、`ComponentDialog` クラスから派生し、6 つのステップで構成される `UserProfileDialog` を作成します。

`UserProfileDialog` コンストラクターで、ウォーターフォール ステップ、プロンプト、およびウォーターフォール ダイアログを作成し、ダイアログ セットに追加します。 プロンプトは、それが使用されるダイアログ セットに追加する必要があります。

[!code-javascript[Constructor snippet](~/../botbuilder-samples/samples/javascript_nodejs/05.multi-turn-prompt/dialogs/userProfileDialog.js?range=25-47)]

次に、ダイアログで使用されるステップを実装します。 プロンプトを使用するには、そのプロンプトをご自身のダイアログのステップから呼び出し、この場合は `step.result` を使用して、ステップ コンテキストの次のステップでプロンプトの結果を取得します。 バックグラウンドでは、プロンプトは 2 つのステップから成るダイアログです。 最初のステップでプロンプトは入力を要求します。そして 2 番目のステップで有効な値を返すか、最初からやり直して、有効な入力を受信するまでユーザーに再入力を要求します。

ウォーターフォール ステップからは常に null 以外の `DialogTurnResult` を返す必要があります。 そうしないと、ご自身のダイアログは設計どおりに機能しません。 ここでは、ウォーターフォール ダイアログの `nameStep` に対する実装を示します。

[!code-javascript[name step](~/../botbuilder-samples/samples/javascript_nodejs/05.multi-turn-prompt/dialogs/userProfileDialog.js?range=75-78)]

`ageStep` で、ユーザー入力を検証できない場合の再試行プロンプトを指定します。入力を検証できない原因は、その入力がプロンプトで解析できない形式であるか、上記のコンストラクターで指定した検証基準を満たしていないかのいずれかです。 この状況で、再試行プロンプトが指定されていないと、プロンプトは最初のプロンプト テキストを使用して、ユーザーに再入力を求めます。

[!code-javascript[age step](~/../botbuilder-samples/samples/javascript_nodejs/05.multi-turn-prompt/dialogs/userProfileDialog.js?range=90-101&highlight=5)]

**userProfile.js**

ユーザーの移動手段、名前、および年齢は `UserProfile` クラスのインスタンスに保存されています。

[!code-javascript[user profile](~/../botbuilder-samples/samples/javascript_nodejs/05.multi-turn-prompt/userProfile.js?range=4-10)]

**Dialogs\UserProfileDialog.cs**

最後のステップで、前のウォーターフォール ステップで呼び出されたダイアログによって返された `step.result` を確認します。 戻り値が true の場合は、ユーザー プロファイル アクセサーを使用して、ユーザー プロファイルを取得し、更新します。 ユーザー プロファイルを取得するには、`get` メソッドを呼び出して、`userProfile.transport`、`userProfile.name`、`userProfile.age` の各プロパティの値を設定します。 最後に、ダイアログを終了する `endDialog` を呼び出す前に、ユーザーの情報をまとめます。 ダイアログを終了すると、そのダイアログはダイアログ スタックから取り出され、ダイアログの親に省略可能な結果が返されます。 この親は、終了したばかりのダイアログを開始したダイアログまたはメソッドです。

[!code-javascript[summary step](~/../botbuilder-samples/samples/javascript_nodejs/05.multi-turn-prompt/dialogs/userProfileDialog.js?range=115-136&highlight=4-8,20-21)]

---

## <a name="create-the-extension-method-to-run-the-waterfall-dialog"></a>拡張メソッドを作成してウォーターフォール ダイアログを実行する

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

`Run` 拡張メソッドの定義が完了しました。この拡張メソッドは、ダイアログ コンテキストの作成およびアクセスに使用します。 ここでは `accessor` はダイアログ状態プロパティの状態プロパティ アクセサーで、`dialog` はユーザー プロファイルのコンポーネント ダイアログです。 コンポーネント ダイアログによって内部ダイアログ セットが定義されているため、メッセージ ハンドラー コードに表示される外部ダイアログ セットを作成し、それを使用してダイアログ コンテキストを作成する必要があります。

ダイアログ コンテキストは、`CreateContext` メソッドを呼び出すことで作成され、ボットのターン ハンドラー内からダイアログ セットと対話するために使用されます。 ダイアログ コンテキストには、現在のターン コンテキスト、親ダイアログ、およびダイアログの状態が含まれています。ダイアログの状態が、ダイアログ内の情報を保持する方法を指定します。

ダイアログ コンテキストを使用すると、文字列 ID を使用してダイアログを開始したり、現在のダイアログ (複数のステップが含まれるウォーターフォール ダイアログなど) を続行したりすることができます。 ダイアログ コンテキストは、ボットのすべてのダイアログおよびウォーターフォール ステップに渡されます。

**DialogExtensions.cs**

[!code-csharp[Run method](~/../botbuilder-samples/samples/csharp_dotnetcore/05.multi-turn-prompt/DialogExtensions.cs?range=13-24)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

`userProfileDialog` 内の `run` ヘルパー メソッドの定義が完了しました。このヘルパー メソッドは、ダイアログ コンテキストの作成およびアクセスに使用します。 ここでは `accessor` はダイアログ状態プロパティの状態プロパティ アクセサーで、`this` はユーザー プロファイルのコンポーネント ダイアログです。 コンポーネント ダイアログによって内部ダイアログ セットが定義されているため、メッセージ ハンドラー コードに表示される外部ダイアログ セットを作成し、それを使用してダイアログ コンテキストを作成する必要があります。

ダイアログ コンテキストは、`createContext` メソッドを呼び出すことで作成され、ボットのターン ハンドラー内からダイアログ セットと対話するために使用されます。 ダイアログ コンテキストには、現在のターン コンテキスト、親ダイアログ、およびダイアログの状態が含まれています。ダイアログの状態が、ダイアログ内の情報を保持する方法を指定します。

ダイアログ コンテキストを使用すると、文字列 ID を使用してダイアログを開始したり、現在のダイアログ (複数のステップが含まれるウォーターフォール ダイアログなど) を続行したりすることができます。 ダイアログ コンテキストは、ボットのすべてのダイアログおよびウォーターフォール ステップに渡されます。

[!code-javascript[run method](~/../botbuilder-samples/samples/javascript_nodejs/05.multi-turn-prompt/dialogs/userProfileDialog.js?range=55-64)]

---

## <a name="run-the-dialog"></a>ダイアログを実行する

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

**Bots\DialogBot.cs**

`OnMessageActivityAsync` ハンドラーでは、ダイアログの開始または続行に拡張メソッドが使用されます。 `OnTurnAsync` では、ボットの状態管理オブジェクトを使用して、ストレージに対するすべての状態変更を保持します  (`ActivityHandler.OnTurnAsync` メソッドでは、`OnMessageActivityAsync` など、さまざまなアクティビティ ハンドラー メソッドが呼び出されます。 この方法で、メッセージ ハンドラーの完了後、ターン自体が完了する前に、状態を保存します)。

[!code-csharp[overrides](~/../botbuilder-samples/samples/csharp_dotnetcore/05.multi-turn-prompt/Bots/DialogBot.cs?range=33-48&highlight=5-7)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

`onMessage` ハンドラーでは、ダイアログの開始または続行にヘルパー メソッドが使用されます。 `onDialog` では、ボットの状態管理オブジェクトを使用して、ストレージに対するすべての状態変更を保持します  (`onDialog` メソッドは、`onMessage` など、他の定義済みハンドラーの実行後、最後に呼び出されます。 この方法で、メッセージ ハンドラーの完了後、ターン自体が完了する前に、状態を保存します)。

[!code-javascript[overrides](~/../botbuilder-samples/samples/javascript_nodejs/05.multi-turn-prompt/bots/dialogBot.js?range=30-44)]

---

## <a name="register-services-for-the-bot"></a>ボット用のサービスを登録する

このボットでは、次の "_サービス_" を使用します。

- ボット用の基本サービス: 資格情報プロバイダー、アダプター、およびボット実装。
- 状態を管理するためのサービス: ストレージ、ユーザー状態、および会話の状態。
- ボットで使用されるダイアログ。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

**Startup.cs**

`Startup` でボット用のサービスを登録します。 これらのサービスは、依存関係の挿入を通じてコードの他の部分で使用できます。

[!code-csharp[ConfigureServices](~/../botbuilder-samples/samples/csharp_dotnetcore/05.multi-turn-prompt/Startup.cs?range=17-41)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**index.js**

`index.js` でボット用のサービスを登録します。 

[!code-javascript[overrides](~/../botbuilder-samples/samples/javascript_nodejs/05.multi-turn-prompt/index.js?range=18-49)]

---

> [!NOTE]
> メモリ ストレージはテストにのみ使用され、実稼働を目的としたものではありません。
> 運用環境のボットでは、必ず永続タイプのストレージを使用してください。

## <a name="to-test-the-bot"></a>ボットをテストする

1. [Bot Framework Emulator](https://aka.ms/bot-framework-emulator-readme) をインストールします (まだインストールしていない場合)。
1. ご自身のマシンを使ってローカルでサンプルを実行します。
1. 以下に示すように、エミュレーターを起動し、お使いのボットに接続して、メッセージを送信します。

![マルチターン プロンプト ダイアログの実行サンプル](../media/emulator-v4/multi-turn-prompt.png)

## <a name="additional-information"></a>追加情報

### <a name="about-dialog-and-bot-state"></a>ダイアログとボットの状態について

このボットでは、2 つの状態プロパティ アクセサーを定義しました。

- 1 つはダイアログ状態プロパティ用の会話状態内で作成されます。 ダイアログ状態が追跡するのは、ダイアログ セットのダイアログ内におけるユーザーの場所です。この状態は、begin dialog メソッドや continue dialog メソッドを呼び出すときなど、ダイアログ コンテキストによって更新されます。
- もう 1 つはユーザー プロファイル プロパティ用のユーザー状態内で作成されます。 これは、保持しているユーザー情報を追跡するためにボットによって使用されます。この状態は、ダイアログ コードで明示的に管理します。

状態管理オブジェクトのキャッシュのプロパティ値は、状態プロパティ アクセサーの _get_ メソッドおよび _set_ メソッドによって取得および設定されます。 キャッシュはターンで状態プロパティの値が最初に要求されたときに設定されますが、明示的に保持する必要があります。 この両方の状態プロパティに対する変更を保持するには、対応する状態管理オブジェクトの _save changes_  メソッドを呼び出します。

このサンプルでは、ダイアログ内からユーザー プロファイルの状態が更新されます。 この方法は単純なボットには有効ですが、ボットの枠を越えてダイアログを再利用する場合は機能しません。

ダイアログ ステップとボットの状態を別々に維持するオプションには、さまざまな種類があります。 たとえば、お使いのダイアログで完全な情報を収集すると、以下を行うことができます。

- end dialog メソッドを使用して、収集したデータを戻り値として親コンテキストに提供する。 これは、ボットのターン ハンドラーでもダイアログ スタック上の前のアクティブ ダイアログでもかまいません。 プロンプト クラスはこのように設計されています。
- 適切なサービスへの要求を生成する。 お使いのボットが大規模なサービスへのフロントエンドとして動作している場合にうまく機能することがあります。

## <a name="next-steps"></a>次の手順

> [!div class="nextstepaction"]
> [ボットに自然言語の理解を追加する](bot-builder-howto-v4-luis.md)

<!-- Footnote-style links -->

[concept-basics]: bot-builder-basics.md
[concept-state]: bot-builder-concept-state.md
[concept-dialogs]: bot-builder-concept-dialog.md

[prompting]: bot-builder-prompts.md
[component-dialogs]: bot-builder-compositcontrol.md

[cs-sample]: https://aka.ms/cs-multi-prompts-sample
[js-sample]: https://aka.ms/js-multi-prompts-sample
