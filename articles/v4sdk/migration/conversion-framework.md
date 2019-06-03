---
title: 同じ .NET Framework プロジェクト内での既存のボットの移行 | Microsoft Docs
description: 同じプロジェクトを使用して、既存の v3 ボットを取得し、それを v4 SDK に移行します。
keywords: bot の移行, formflow, ダイアログ, v3 ボット
author: JonathanFingold
ms.author: v-jofing
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 05/23/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: aca21d9af94f274936900f1d73c1b340272cd089
ms.sourcegitcommit: ea64a56acfabc6a9c1576ebf9f17ac81e7e2a6b7
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 05/24/2019
ms.locfileid: "66215611"
---
# <a name="migrate-a-net-v3-bot-to-a-framework-v4-bot"></a>.NET v3 ボットを Framework v4 ボットに移行する

この記事では、"_プロジェクトの種類を変換せずに_"、[v3 ContosoHelpdeskChatBot](https://github.com/microsoft/BotBuilder-Samples/tree/master/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V3) を v4 ボットに変換します。 ボットは、.NET Framework プロジェクトのままになります。
この変換は次の手順に分けられます。

1. NuGet パッケージを更新してインストールする
1. Global.asax.cs ファイルを更新する
1. MessagesController クラスを更新する
1. ダイアログを変換する

この変換の結果が、[.NET Framework v4 ContosoHelpdeskChatBot](https://github.com/microsoft/BotBuilder-Samples/tree/master/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework) です。

Bot Framework SDK v4 は、SDK v3 と同じ基になる REST API に基づいています。 ただし、SDK v4 は、開発者がより柔軟にボットを制御できるように、以前のバージョンの SDK をリファクタリングしたものです。 この SDK の主な変更点は次のとおりです。

- 状態は、状態管理オブジェクトとプロパティ アクセサーを使用して管理されます。
- ターン ハンドラーの設定とターン ハンドラーへのアクティビティの受け渡しが変更されました。
- scoreable は存在しなくなりました。 ダイアログに制御を渡す前に、ターン ハンドラー内で "グローバル" コマンドの有無を確認できます。
- 以前のバージョンのものとは大きく異なる新しい Dialogs ライブラリ。 コンポーネント ダイアログ、ウォーターフォール ダイアログ、v4 用の FormFlow ダイアログのコミュニティ実装を使用して、古いダイアログを新しいダイアログ システムに変換する必要があります。

具体的な変更点の詳細については、[.NET SDK v3 と v4 の違い](migration-about.md)に関する記事をご覧ください。

## <a name="update-and-install-nuget-packages"></a>NuGet パッケージを更新してインストールする

1. **Microsoft.Bot.Builder.Azure** と **Microsoft.Bot.Builder.Integration.AspNet.WebApi** を最新の安定したバージョンに更新します。

    **Microsoft.Bot.Builder** パッケージと **Microsoft.Bot.Connector** は依存関係であるため、これらのパッケージも更新されます。

1. **Microsoft.Bot.Builder.History** パッケージを削除します。 これは v4 SDK には含まれていません。
1. **Autofac.WebApi2** を追加します。

    ASP.NET の依存関係挿入に役立つため、これを使用します。

1. **Bot.Builder.Community.Dialogs.Formflow** を追加します。

    これは、v3 の FormFlow 定義ファイルから v4 のダイアログを構築するためのコミュニティ ライブラリです。 その依存関係の 1 つとして、**Microsoft.Bot.Builder.Dialogs** があるので、これも自動的にインストールされます。

この時点でビルドすると、コンパイラ エラーが発生します。 これらは無視してかまいません。 変換が完了すると、作業コードが作成されます。

## <a name="update-your-globalasaxcs-file"></a>Global.asax.cs ファイルを更新する

スキャフォールディングの一部が変更されたので、v4 では[状態管理](../bot-builder-concept-state.md)インフラストラクチャの一部を自分で設定する必要があります。 たとえば、v4 ではボット アダプターを使用して認証を処理し、アクティビティをボット コードに転送します。また、状態プロパティを事前に宣言しておきます。

`DialogState` の状態プロパティを作成します。これは、v4 のダイアログのサポートに必要になります。 依存関係挿入を使用して、必要な情報をコントローラーとボット コードに取得します。

**Global.asax.cs** 内で、次の手順を実行します。

1. `using` ステートメントを更新します。[!code-csharp[Using statements](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Global.asax.cs?range=4-13)]

1. **Application_Start** メソッドから次の行を削除します。[!code-csharp[Removed lines](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V3/ContosoHelpdeskChatBot/Global.asax.cs?range=23-24)]

    次の行を挿入します。[!code-csharp[Reference BotConfig.Register](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Global.asax.cs?range=22)]

1. 参照されなくなった **RegisterBotModules** メソッドを削除します。

1. **BotConfig.UpdateConversationContainer** メソッドを、この **BotConfig.Register** メソッドに置き換えます。このメソッドを使用して、依存関係挿入をサポートするために必要なオブジェクトを登録します。 このボットでは "_ユーザー_" 状態も "_個人的な会話_" 状態も使用しないため、会話状態管理オブジェクトのみを作成します。
    [!code-csharp[Define BotConfig.Register](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Global.asax.cs?range=31-61)]

## <a name="update-your-messagescontroller-class"></a>MessagesController クラスを更新する

v4 のボットでは、ここでターン ハンドラーが開始されるので、多くの変更を行う必要があります。 ボットのターン ハンドラー自体を除き、このほとんどは定型句と考えることができます。 **Controllers\MessagesController.cs** ファイル内で、次の手順を実行します。

1. `using` ステートメントを更新します。[!code-csharp[Using statements](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Controllers/MessagesController.cs?range=4-8)]

1. クラスから `[BotAuthentication]` 属性を削除します。 v4 では、ボットのアダプターが認証を処理します。

1. これらのフィールドと、これらのフィールドを初期化するコンス トラクターを追加します。 ASP.NET と Autofac では、依存関係挿入を使用してパラメーター値を取得します (これに対応するため、アダプター オブジェクトとボット オブジェクトを **Global.asax.cs** に登録しました)。 [!code-csharp[Fields and constructor](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Controllers/MessagesController.cs?range=14-21)]

1. **Post** メソッドの本体を置き換えます。 アダプターを使用して、ボットのメッセージ ループ (ターン ハンドラー) を呼び出します。
    [!code-csharp[Post method](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Controllers/MessagesController.cs?range=23-31)]

### <a name="delete-the-cancelscorable-and-globalmessagehandlersbotmodule-classes"></a>CancelScorable クラスと GlobalMessageHandlersBotModule クラスを削除する

v4 には scorable が存在せず、`cancel` メッセージに対応するようにターン ハンドラーを更新したので、**CancelScorable** (**Dialogs\CancelScorable.cs** 内) クラスと **GlobalMessageHandlersBotModule** クラスを削除できます。

## <a name="create-your-bot-class"></a>ボット クラスを作成する

v4 では、基本的に、ターン ハンドラーまたはメッセージ ループのロジックはボット ファイル内にあります。 `ActivityHandler` から派生させます。これは、一般的な種類のアクティビティのハンドラーを定義します。

1. **Bots\DialogBots.cs** ファイルを作成します。

1. `using` ステートメントを更新します。[!code-csharp[Using statements](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Bots/DialogBot.cs?range=4-8)]

1. `ActivityHandler` から `DialogBot` を派生させて、ダイアログ用のジェネリック パラメーターを追加します。
    [!code-csharp[Class definition](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Bots/DialogBot.cs?range=19)]

1. これらのフィールドと、これらのフィールドを初期化するコンス トラクターを追加します。 同様に、ASP.NET と Autofac では、依存関係挿入を使用してパラメーター値を取得します。
    [!code-csharp[Fields and constructor](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Bots/DialogBot.cs?range=21-28)]

1. `OnMessageActivityAsync` をオーバーライドしてメイン ダイアログを呼び出します。 (この後すぐに `Run` 拡張メソッドを定義します) [!code-csharp[OnMessageActivityAsync](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Bots/DialogBot.cs?range=38-47)]

1. `OnTurnAsync` をオーバーライドして、ターンの最後に会話状態を保存します。 v4 では、状態を永続化レイヤーに書き込むために、明示的にこれを行う必要があります。 `ActivityHandler.OnTurnAsync` メソッドでは、受信したアクティビティの種類に基づいて特定のアクティビティ ハンドラー メソッドが呼び出されます。そのため、基本メソッドの呼び出し後の状態を保存します。
    [!code-csharp[OnTurnAsync](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Bots/DialogBot.cs?range=30-36)]

### <a name="create-the-run-extension-method"></a>Run 拡張メソッドを作成する

拡張メソッドを作成します。この拡張メソッドで、ボットから最小限のコンポーネント ダイアログを実行するために必要なコードを統合します。

**DialogExtensions.cs** ファイルを作成し、`Run` 拡張メソッドを実装します。
[!code-csharp[The extension](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/DialogExtensions.cs?range=4-41)]

## <a name="convert-your-dialogs"></a>ダイアログを変換する

v4 SDK に移行するために、元のダイアログに多くの変更を加えます。 現時点では、コンパイラ エラーは気にしないでください。 これらは変換が完了したら解決します。
元のコードを必要以上に変更しないようにするために、移行の完了後もコンパイラの警告がいくつか残ります。

すべてのダイアログは、v3 の `IDialog<object>` インターフェイスを実装するのではなく、`ComponentDialog` から派生します。

このボットには、変換する必要があるダイアログが 4 つあります。

| | |
|---|---|
| [RootDialog](#update-the-root-dialog) | オプションを提示し、他のダイアログを開始します。 |
| [InstallAppDialog](#update-the-install-app-dialog) | コンピューターにアプリをインストールする要求を処理します。 |
| [LocalAdminDialog](#update-the-local-admin-dialog) | ローカル コンピューターの管理者権限の要求を処理します。 |
| [ResetPasswordDialog](#update-the-reset-password-dialog) | パスワードをリセットする要求を処理します。 |

これらは入力を収集しますが、コンピューター上でこれらの操作を実行するわけではありません。

### <a name="make-solution-wide-dialog-changes"></a>ソリューション全体のダイアログを変更する

1. ソリューション全体で、出現するすべての `IDialog<object>` を `ComponentDialog` に置き換えます。
1. ソリューション全体で、出現するすべての `IDialogContext` を `DialogContext` に置き換えます。
1. 各ダイアログ クラスで、`[Serializable]` 属性を削除します。

ダイアログ内の制御フローとメッセージングは同じ方法で処理されなくなったので、各ダイアログを変換するときにこれを変更する必要があります。

| Operation | v3 コード | v4 コード |
| :--- | :--- | :--- |
| ダイアログの開始を処理する | `IDialog.StartAsync` を実装する | これをウォーターフォール ダイアログの最初のステップにするか、`Dialog.BeginDialogAsync` を実装する |
| ダイアログの継続を処理する | `IDialogContext.Wait` を呼び出す | ウォーターフォール ダイアログにステップを追加するか、`Dialog.ContinueDialogAsync` を実装する |
| ユーザーにメッセージを送信する | `IDialogContext.PostAsync` を呼び出す | `ITurnContext.SendActivityAsync` を呼び出す |
| 子ダイアログを開始する | `IDialogContext.Call` を呼び出す | `DialogContext.BeginDialogAsync` を呼び出す |
| 現在のダイアログが完了したことを通知する | `IDialogContext.Done` を呼び出す | `DialogContext.EndDialogAsync` を呼び出す |
| ユーザーの入力を取得する | `IAwaitable<IMessageActivity>` パラメーターを使用する | ウォーターフォール内からプロンプトを使用するか、`ITurnContext.Activity` を使用する |

v4 コードに関する注意事項を次に示します。

- ダイアログ コード内で、`DialogContext.Context` プロパティを使用して現在のターン コンテキストを取得します。
- ウォーターフォール ステップには、`DialogContext` から派生した `WaterfallStepContext` パラメーターがあります。
- 具象ダイアログ クラスおよびプロンプト クラスはすべて抽象 `Dialog` クラスから派生します。
- コンポーネント ダイアログを作成するときに ID を割り当てます。 ダイアログ セットの各ダイアログには、そのセット内で一意の ID を割り当てる必要があります。

### <a name="update-the-root-dialog"></a>ルート ダイアログを更新する

このボットでは、ルート ダイアログがユーザーに一連のオプションからの選択を要求し、その選択に基づいて子ダイアログを開始します。 会話の有効期間中、これがループします。

- メイン フローはウォーターフォール ダイアログとして設定できます。これは、v4 SDK の新しい概念です。 固定された一連のステップが順番に実行されます。 詳細については、「[連続して行われる会話フローの実装](~/v4sdk/bot-builder-dialog-manage-conversation-flow.md)」をご覧ください。
- プロンプトは、プロンプト クラスで処理されるようになりました。プロンプト クラスは、入力を要求し、最小限の処理と検証を実行して値を返す短い子ダイアログです。 詳細については、[ダイアログ プロンプトを使用したユーザー入力の収集](~/v4sdk/bot-builder-prompts.md)に関するページをご覧ください。

**Dialogs/RootDialog.cs** ファイルで、次の手順を実行します。

1. `using` ステートメントを更新します。[!code-csharp[Using statements](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Dialogs/RootDialog.cs?range=4-10)]

1. `HelpdeskOptions` オプションを文字列のリストから選択肢のリストに変換する必要があります。 これは選択プロンプトで使用されます。選択プロンプトは、(リスト内で) 選択された番号、選択された値、または選択肢のいずれかの同義語を有効な入力として受け入れます。
    [!code-csharp[HelpDeskOptions](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Dialogs/RootDialog.cs?range=28-33)]

1. コンストラクターを追加します。 このコードは、次の処理を実行します。
   - ダイアログの各インスタンスの作成時に ID を割り当てます。 ダイアログ ID は、ダイアログの追加先となるダイアログ セットに含まれます。 ボットが **MessageController** クラス内でダイアログ オブジェクトを使用して初期化されたことを思い出してください。 各 `ComponentDialog` には、一連の独自のダイアログ ID が割り当てられた独自の内部ダイアログ セットがあります。
   - 選択プロンプトなど、他のダイアログを子ダイアログとして追加します。 ここでは、各ダイアログ ID にクラス名を使用します。
   - 3 ステップのウォーターフォール ダイアログを定義します。 このダイアログはこの後すぐに実装します。
     - ダイアログでは、まず、ユーザーに実行するタスクを選択するよう求めます。
     - 次に、その選択肢に関連付けられている子ダイアログを開始します。
     - 最後に、ダイアログ自体を再開します。
   - ウォーターフォールの各ステップはデリゲートです。次にそれらを実装します。可能であれば、元のダイアログから既存のコードを取得します。
   - コンポーネント ダイアログを開始すると、その _initial dialog_ が開始されます。 既定では、これはコンポーネント ダイアログに追加された最初の子ダイアログです。 `InitialDialogId` プロパティを明示的に設定します。つまり、メインのウォーターフォール ダイアログが、セットに追加する最初のダイアログである必要はありません。 たとえば、最初にプロンプトを追加する方がよい場合は、実行時の問題を引き起こさずに、これを行うことが可能になります。
    [!code-csharp[Constructor](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Dialogs/RootDialog.cs?range=35-49)]

1. **StartAsync** メソッドは削除できます。 コンポーネント ダイアログが開始されると、"_初期_" ダイアログが自動的に開始されます。 ここでは、コンストラクターで定義したウォーターフォール ダイアログがこれに該当します。 このダイアログも、その最初のステップで自動的に開始されます。

1. **MessageReceivedAsync** メソッドと **ShowOptions** メソッドを削除し、それらをウォーターフォールの最初のステップに置き換えます。 この 2 つのメソッドは、ユーザーにあいさつし、利用可能なオプションのいずれかを選択するよう求めていました。
   - ここでは、選択リストが表示されます。あいさつメッセージとエラー メッセージは、選択プロンプトの呼び出しでオプションとして提供されます。
   - 選択プロンプトが完了すると、ウォーターフォールは次のステップに進むため、ダイアログで次に呼び出すメソッドを指定する必要はありません。
   - 選択プロンプトは、有効な入力を受け取るか、ダイアログ スタック全体がキャンセルされるまでループします。
    [!code-csharp[PromptForOptionsAsync](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Dialogs/RootDialog.cs?range=51-65)]

1. **OnOptionSelected** をウォーターフォールの 2 番目のステップに置き換えることができます。 置換後も、ユーザーの入力に基づいて子ダイアログを開始します。
   - 選択プロンプトは `FoundChoice` 値を返します。 これは、ステップ コンテキストの `Result` プロパティに示されます。 ダイアログ スタックでは、すべての戻り値をオブジェクトとして扱います。 戻り値がダイアログのいずれかから返されている場合、そのオブジェクトがどの種類の値であるかがわかります。 各プロンプトの種類で返される値の一覧については、[プロンプトの種類](../bot-builder-concept-dialog.md#prompt-types)に関するセクションをご覧ください。
   - 選択プロンプトは例外をスローしないため、try-catch ブロックを削除できます。
   - このメソッドが常に適切な値を返すように、フォールスルーを追加する必要があります。 このコードへの到達は回避する必要がありますが、到達しても、ダイアログを "正常に失敗にする" ことができます。
    [!code-csharp[ShowChildDialogAsync](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Dialogs/RootDialog.cs?range=67-102)]

1. 最後に、古い **ResumeAfterOptionDialog** メソッドをウォーターフォールの最後のステップに置き換えます。
    - 元のダイアログで行っていたように、ダイアログを終了してチケット番号を返すのではなく、スタック上で元のインスタンスをそれ自体の新しいインスタンスに置き換えることでウォーターフォールを再開します。 これが可能なのは、元のアプリでは常に戻り値 (チケット番号) が無視され、ルート ダイアログが再開されていたためです。
    [!code-csharp[ResumeAfterAsync](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Dialogs/RootDialog.cs?range=104-138)]

### <a name="update-the-install-app-dialog"></a>アプリのインストール ダイアログを更新する

アプリのインストール ダイアログでは、いくつかの論理タスクを実行します。これを 4 ステップのウォーターフォール ダイアログとして設定します。 既存のコードをウォーターフォール ステップにどのように組み込むかは、ダイアログごとの論理的な作業です。 ステップごとに、そのコードが含まれていた元のメソッドに留意します。

1. ユーザーに検索文字列を要求します。
1. 一致候補をデータベースに照会します。
   - 一致候補が 1 つの場合は、これを選択して続行します。
   - 一致候補が複数ある場合は、ユーザーにいずれかを選択するよう求めます。
   - 一致候補がない場合、ダイアログは終了します。
1. ユーザーにアプリをインストールするコンピューター名の入力を求めます。
1. 情報をデータベースに書き込み、確認メッセージを送信します。

**Dialogs/InstallAppDialog.cs** ファイルで、次の手順を実行します。

1. `using` ステートメントを更新します。[!code-csharp[Using statements](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Dialogs/InstallAppDialog.cs?range=4-11)]

1. 収集した情報を追跡するために使用するキーの定数を定義します。
    [!code-csharp[Key ID](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Dialogs/InstallAppDialog.cs?range=17-18)]

1. コンストラクターを追加し、コンポーネントのダイアログ セットを初期化します。
    [!code-csharp[Constructor](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Dialogs/InstallAppDialog.cs?range=20-34)]

1. **StartAsync** をウォーターフォールの最初のステップに置き換えることができます。
    - 状態を自分で管理する必要があるので、ダイアログの状態でアプリのインストール オブジェクトを追跡します。
    - ユーザーに入力を求めるメッセージは、プロンプトの呼び出しでのオプションになります。
    [!code-csharp[GetSearchTermAsync](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Dialogs/InstallAppDialog.cs?range=35-50)]

1. **appNameAsync** と **multipleAppsAsync** をウォーターフォールの 2 番目のステップに置き換えることができます。
    - ユーザーの最後のメッセージを単に確認するのではなく、プロンプトの結果を取得します。
    - データベース クエリと if ステートメントは、**appNameAsync** の場合と同様に構成されています。 if ステートメントの各ブロック内のコードは、v4 ダイアログで動作するように更新されました。
        - 一致候補が 1 つの場合は、ダイアログの状態を更新し、次のステップに進みます。
        - 一致候補が複数ある場合は、選択プロンプトを使用して、オプションの一覧から選択するようユーザーに求めます。 つまり、**multipleAppsAsync** は削除できます。
        - 一致候補がない場合は、このダイアログを終了し、ルート ダイアログに null を返します。
    [!code-csharp[ResolveAppNameAsync](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Dialogs/InstallAppDialog.cs?range=52-91)]

1. **appNameAsync** では、クエリの解決後にユーザーにコンピューター名の入力も求めていました。 ウォーターフォールの次のステップに、ロジックのこの部分を取り込みます。
    - 繰り返しますが、v4 では状態を自分で管理する必要があります。 ここで唯一注意が必要なことは、前のステップで 2 つの異なるロジック分岐を通過してこのステップに到達できることです。
    - 今回は別のオプションを指定して、以前と同じテキスト プロンプトを使ってユーザーにコンピューター名の入力を求めます。
    [!code-csharp[GetMachineNameAsync](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Dialogs/InstallAppDialog.cs?range=93-114)]

1. **machineNameAsync** のロジックは、ウォーターフォールの最後のステップにまとめられています。
    - テキスト プロンプトの結果からコンピューター名を取得し、ダイアログの状態を更新します。
    - サポート コードは別のプロジェクトにあるため、データベースを更新する呼び出しを削除します。
    - その後、成功メッセージをユーザーに送信し、ダイアログを終了します。
    [!code-csharp[SubmitRequestAsync](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Dialogs/InstallAppDialog.cs?range=116-135)]

1. データベースの呼び出しをシミュレートするために、データベースではなく静的リストを照会するように **getAppsAsync** をモックアップします。
    [!code-csharp[GetAppsAsync](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Dialogs/InstallAppDialog.cs?range=137-200)]

### <a name="update-the-local-admin-dialog"></a>ローカル管理者ダイアログを更新する

v3 では、このダイアログでユーザーにあいさつし、FormFlow ダイアログを開始して、結果をデータベースに保存していました。 これは 2 ステップのウォーターフォールに簡単に変換できます。

1. `using` ステートメントを更新します。 このダイアログには、v3 の FormFlow ダイアログが含まれています。 v4 では、FormFlow コミュニティ ライブラリを使用できます。
    [!code-csharp[Using statements](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Dialogs/LocalAdminDialog.cs?range=4-8)]

1. 結果はダイアログの状態で提供されるので、`LocalAdmin` のインスタンス プロパティを削除できます。

1. コンストラクターを追加し、コンポーネントのダイアログ セットを初期化します。 FormFlow ダイアログも同じ方法で作成されます。 コンストラクター内でコンポーネントのダイアログ セットに追加するだけです。
    [!code-csharp[Constructor](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Dialogs/LocalAdminDialog.cs?range=14-23)]

1. **StartAsync** をウォーターフォールの最初のステップに置き換えることができます。 コンストラクター内で FormFlow を既に作成しているので、他の 2 つのステートメントはこれに変換されます。 なお、`FormBuilder` では、生成されたダイアログの ID としてモデルの種類の名前が割り当てられますが、このモデルでは `LocalAdminPrompt` がそれにあたります。
    [!code-csharp[BeginFormflowAsync](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Dialogs/LocalAdminDialog.cs?range=25-35)]

1. **ResumeAfterLocalAdminFormDialog** をウォーターフォールの 2 番目のステップに置き換えることができます。 インスタンス プロパティからではなく、ステップ コンテキストから戻り値を取得する必要があります。
    [!code-csharp[SaveResultAsync](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Dialogs/LocalAdminDialog.cs?range=37-50)]

1. FormFlow でインスタンス プロパティを更新しない点を除き、**BuildLocalAdminForm** の変更はほとんどありません。
    [!code-csharp[BuildLocalAdminForm](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Dialogs/LocalAdminDialog.cs?range=52-76)]

### <a name="update-the-reset-password-dialog"></a>パスワードのリセット ダイアログを更新する

v3 では、このダイアログでユーザーにあいさつし、パス コードでユーザーを承認して、失敗にするかまたは FormFlow ダイアログを開始した後、パスワードをリセットしていました。 これもウォーターフォールに適切に変換できます。

1. `using` ステートメントを更新します。 このダイアログには、v3 の FormFlow ダイアログが含まれています。 v4 では、FormFlow コミュニティ ライブラリを使用できます。
    [!code-csharp[Using statements](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Dialogs/ResetPasswordDialog.cs?range=4-9)]

1. コンストラクターを追加し、コンポーネントのダイアログ セットを初期化します。 FormFlow ダイアログも同じ方法で作成されます。 コンストラクター内でコンポーネントのダイアログ セットに追加するだけです。
    [!code-csharp[Constructor](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Dialogs/ResetPasswordDialog.cs?range=15-25)]

1. **StartAsync** をウォーターフォールの最初のステップに置き換えることができます。 ここでは、コンストラクター内で FormFlow を既に作成しました。 それ以外は、v3 の呼び出しから v4 の対応する呼び出しへの変換だけを行い、同じロジックを維持します。
    [!code-csharp[BeginFormflowAsync](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Dialogs/ResetPasswordDialog.cs?range=27-45)]

1. **sendPassCode** は主に演習として残してあります。 元のコードはコメント アウトされています。このメソッドは true を返すだけです。 また、メール アドレスは、元のボットで使用されていなかったので削除することもできます。
    [!code-csharp[SendPassCode](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Dialogs/ResetPasswordDialog.cs?range=47-81)]

1. **BuildResetPasswordForm** に変更はありません。

1. **ResumeAfterLocalAdminFormDialog** をウォーターフォールの 2 番目のステップに置き換えることができます。戻り値はステップ コンテキストから取得します。 元のダイアログで何にも使用されていなかった電子メール アドレスを削除しました。また、データベースに照会する代わりにダミーの結果を用意しました。 v3 の呼び出しから v4 の対応する呼び出しへの変換だけを行い、同じロジックを維持します。
    [!code-csharp[ProcessRequestAsync](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Dialogs/ResetPasswordDialog.cs?range=90-113)]

### <a name="update-models-as-necessary"></a>必要に応じてモデルを更新する

FormFlow ライブラリを参照する一部のモデルで、`using` ステートメントを更新する必要があります。

1. `LocalAdminPrompt` で、これらを次のように変更します。[!code-csharp[Using statements](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Models/LocalAdminPrompt.cs?range=4)]

1. `ResetPasswordPrompt` で、これらを次のように変更します。[!code-csharp[Using statements](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Models/ResetPasswordPrompt.cs?range=4-5)]

## <a name="update-webconfig"></a>Web.config の更新

**MicrosoftAppId** と **MicrosoftAppPassword** の構成キーをコメント アウトします。 これにより、これらの値をエミュレーターに提供しなくても、ローカルでボットをデバッグできるようになります。

## <a name="run-and-test-your-bot-in-the-emulator"></a>ボットを実行してエミュレーターでテストする

この時点で、ボットをローカルの IIS で実行し、エミュレーターを使用してボットに接続できます。

1. IIS でボットを実行します。
1. エミュレーターを起動し、ボットのエンドポイント (例: **http://localhost:3978/api/messages** ) に接続します。
    - 初めてボットを実行する場合は、 **[File]\(ファイル\) > [New Bot]\(新しいボット\)** をクリックして、画面の指示に従います。 それ以外の場合は、 **[File]\(ファイル\) > [Open Bot]\(ボットを開く\)** をクリックして既存のボットを開きます。
    - 構成のポート設定を再確認します。 たとえば、ブラウザーで `http://localhost:3979/` にアクセスしてボットを開いている場合は、エミュレーターでボットのエンドポイントを `http://localhost:3979/api/messages` に設定します。
1. 4 つのダイアログはすべて機能するはずです。ウォーターフォール ステップにブレークポイントを設定して、これらのポイントでダイアログ コンテキストとダイアログの状態を確認できます。

## <a name="additional-resources"></a>その他のリソース

v4 の概念トピック

- [ボットのしくみ](../bot-builder-basics.md)
- [状態の管理](../bot-builder-concept-state.md)
- [ダイアログ ライブラリ](../bot-builder-concept-dialog.md)

v4 の使用方法のトピック

- [テキスト メッセージを送受信する](../bot-builder-howto-send-messages.md)
- [ユーザーと会話データを保存する](../bot-builder-howto-v4-state.md)
- [連続して行われる会話フローの実装](../bot-builder-dialog-manage-conversation-flow.md)
