---
title: SDK v3 と v4 の違い | Microsoft Docs
description: SDK v3 と v4 の違いについて説明します。
keywords: bot の移行, formflow, ダイアログ, 状態
author: JonathanFingold
ms.author: v-jofing
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 03/28/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 55b5a4073340bb29074af5b2ee74dd952ea40f0c
ms.sourcegitcommit: f84b56beecd41debe6baf056e98332f20b646bda
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 05/03/2019
ms.locfileid: "65032241"
---
# <a name="differences-between-the-v3-and-v4-net-sdk"></a>.NET SDK v3 と v4 の違い

Bot Framework SDK のバージョン 4 では、バージョン 3 と同じ基になる Bot Framework Service がサポートされています。 ただし、v4 は、開発者がより柔軟にボットを制御できるように、以前のバージョンの SDK をリファクタリングしたものです。 この SDK の主な変更点は次のとおりです。

- ボット アダプターの導入。 アダプターはアクティビティ処理スタックに含まれます。
  - アダプターが Bot Framework 認証を処理します。
  - アダプターは、Bot Framework Connector の呼び出しをカプセル化して、チャネルとボットのターン ハンドラー間の受信トラフィックと送信トラフィックを管理します。
  - アダプターはターンごとにコンテキストを初期化します。
  - 詳細については、「[ボットのしくみ][about-bots]」をご覧ください。
- 状態管理のリファクタリング。
  - 状態データは、ボット内で自動的に使用できなくなりました。
  - 状態は、状態管理オブジェクトとプロパティ アクセサーを使用して管理されるようになりました。
  - 詳細については、「[状態の管理][about-state]」をご覧ください。
- 新しいダイアログ ライブラリ。
  - v3 ダイアログは、新しいダイアログ ライブラリ用に書き換える必要があります。
  - scoreable は存在しなくなりました。 ダイアログに制御を渡す前に、"グローバル" コマンドの有無を確認できます。 ご自身の v4 ボットの設計方法に応じて、これはメッセージ ハンドラーまたは親ダイアログに存在します。 例については、[ユーザーによる割り込みを処理][interruptions]する方法をご覧ください。
  - 詳細については、「[ダイアログ ライブラリ][about-dialogs]」をご覧ください。
- ASP.NET Core のサポート。
  - 新しい C# ボットを作成するためのテンプレートは、ASP.NET Core フレームワークをターゲットとしています。
  - ボットに ASP.NET を引き続き使用することはできますが、v4 では、ASP.NET Core フレームワークのサポートに重点を置いています。
  - このフレームワークの詳細については、「[ASP.NET Core の概要](https://docs.microsoft.com/aspnet/core/)」をご覧ください。

## <a name="activity-processing"></a>アクティビティの処理

ボットのアダプターを作成するときは、チャネルとユーザーからの受信アクティビティを受け取るメッセージ ハンドラー デリゲートも用意します。 アダプターは、受け取ったアクティビティごとにターン コンテキスト オブジェクトを作成します。 ターン コンテキスト オブジェクトがボットのターン ハンドラーに渡され、ターンが完了するとオブジェクトは破棄されます。

ターン ハンドラーは、さまざまな種類のアクティビティを受け取ることができます。 一般に、ボットに含まれているダイアログには "_メッセージ_" アクティビティだけを転送します。 ご自身のボットを `ActivityHandler` から派生させると、すべてのメッセージ アクティビティがボットのターン ハンドラーによって `OnMessageActivityAsync` に転送されます。 このメソッドをオーバーライドして、メッセージ処理ロジックを追加します。 アクティビティの種類の詳細については、[アクティビティ スキーマ][]に関するページをご覧ください。

### <a name="handling-turns"></a>ターンの処理

メッセージを処理するときは、ターン コンテキストを使用して受信アクティビティに関する情報を取得し、アクティビティをユーザーに送信します。

| | |
|-|-|
| 受信アクティビティを取得する | ターン コンテキストの `Activity` プロパティを取得します。 |
| アクティビティを作成してユーザーに送信する | ターン コンテキストの `SendActivityAsync` メソッドを呼び出します。<br/>詳細については、「[テキスト メッセージを送受信する][send-messages]」および「[メッセージにメディアを追加する][send-media]」をご覧ください。 |

`MessageFactory` クラスには、アクティビティを作成し、その形式を設定するためのヘルパー メソッドがいくつか用意されています。

### <a name="scorables-is-gone"></a>使用されなくなった scorable

これらはボットのメッセージ ループ内で処理します。 v4 ダイアログでこれを行う方法については、「[ユーザーによる割り込みを処理する][interruptions]」をご覧ください。

"_既定の例外_" など、構成可能な scorable ディスパッチ ツリーと構成可能なチェーン ダイアログもなくなりました。 この機能を再現する 1 つの方法として、ボットのターン ハンドラー内に実装します。

## <a name="state-management"></a>状態管理

v3 では、会話データを、Bot Framework によって提供される大規模なサービス スイートの一部である Bot State Service に格納できました。 ただし、このサービスは、2018 年 3 月 31 日に廃止されました。 v4 以降、状態の管理に関する設計上の考慮事項は、Web アプリと同じで、さまざまなオプションを使用できます。 これをメモリおよび同じプロセス内にキャッシュするのが通常は最も簡単なのですが、運用環境のアプリについては、状態をより永続的に、たとえば SQL、NoSQL データベース、BLOB などに格納する必要があります。

v4 では、状態を管理する際に、`UserData`、`ConversationData`、`PrivateConversationData` の各プロパティとデータ バッグは使用されません。
状態は、「[状態の管理][about-state]」で説明する状態管理オブジェクトとプロパティ アクセサーを使用して管理されるようになりました。

v4 では、ボットの状態データを管理する、`UserState`、`ConversationState`、`PrivateConversationState` の各クラスが定義されています。 定義済みのデータ バッグを単に読み書きするのではなく、保持するプロパティごとに状態プロパティ アクセサーを作成する必要があります。

### <a name="setting-up-state"></a>状態の設定

**Startup.cs** (.NET Core の場合) または **Global.asax.cs** (.NET Framework の場合) で、可能であれば状態をシングルトンとして構成します。

1. 1 つ以上の `IStorage` オブジェクトを初期化します。 これは、ボットのデータのバッキング ストアを表します。
    v4 SDK では、[ストレージ レイヤー](../bot-builder-concept-state.md#storage-layer)がいくつか提供されます。
    別の種類のストアに接続するために、独自のレイヤーを実装することもできます。
1. 必要に応じて、[状態管理](../bot-builder-concept-state.md#state-management)オブジェクトを作成し、登録します。
    v3 の場合と同じスコープを使用できます。また、必要に応じて、他のスコープを作成することもできます。
1. ボットに必要なプロパティの[状態プロパティ アクセサー](../bot-builder-concept-state.md#state-property-accessors)を作成し、登録します。
    状態管理オブジェクト内では、各プロパティ アクセサーに一意の名前が必要です。

### <a name="using-state"></a>状態の使用

ボットが作成されるたびに、依存関係挿入を使用してこれらにアクセスできます 
(ASP.NET では、ボットまたはメッセージ コントローラーの新しいインスタンスがターンごとに作成されます)。状態プロパティ アクセサーを使用してプロパティを取得および更新し、状態管理オブジェクトを使用して変更をストレージに書き込みます。 コンカレンシーの問題を考慮する必要があることを理解しているという前提で、一般的なタスクを実行する方法を次に示します。

| | |
|-|-|
| 状態プロパティ アクセサーを作成する | `BotState.CreateProperty<T>` を呼び出します。<br/>`BotState` は、会話状態、個人的な会話状態、ユーザー状態を表す抽象基底クラスです。 |
| プロパティの現在の値を取得する | `IStatePropertyAccessor<T>.GetAsync` を呼び出します。<br/>以前に値が設定されていない場合は、出荷時の既定のパラメーターを使用して値が生成されます。 |
| プロパティの現在のキャッシュ値を更新する | `IStatePropertyAccessor<T>.SetAsync` を呼び出します。<br/>これにより、キャッシュだけが更新され、バッキング ストレージ レイヤーは更新されません。 |
| 状態の変更をストレージに保持する | ターン ハンドラーを終了する前に、状態が変わった状態管理オブジェクトに対して `BotState.SaveChangesAsync` を呼び出します。 |

### <a name="managing-concurrency"></a>コンカレンシーを管理する

ボットで状態のコンカレンシーを管理することが必要な場合があります。 詳細については、「**状態の管理**」の「[状態の保存](../bot-builder-concept-state.md#saving-state)」、および「**ストレージに直接書き込む**」の「[eTag を使用してコンカレンシーを管理する](../bot-builder-howto-v4-storage.md#manage-concurrency-using-etags)」をご覧ください。

## <a name="dialogs-library"></a>ダイアログ ライブラリ

ダイアログの主な変更点を次に示します。

- ダイアログ ライブラリが独立した NuGet パッケージ (**Microsoft.Bot.Builder.Dialogs**) になりました。
- ダイアログ クラスがシリアル化可能である必要はなくなりました。 ダイアログの状態は、`DialogState` 状態プロパティ アクセサーを使用して管理されます。
  - ダイアログ オブジェクト自体とは異なり、ダイアログの状態プロパティはターン間で保持されるようになりました。
- `IDialogContext` インターフェイスが `DialogContext` クラスに置き換えられました。 ターン内で、"_ダイアログ セット_" のダイアログ コンテキストを作成します。
  - このダイアログ コンテキストにより、ダイアログ スタック (古いスタック フレーム) をカプセル化されます。 この情報は、ダイアログの状態プロパティ内に保持されます。
- `IDialog` インターフェイスが抽象 `Dialog` クラスに置き換えられました。

### <a name="defining-dialogs"></a>ダイアログの定義

v3 では `IDialog` インターフェイスを使用して柔軟にダイアログを実装できましたが、検証などの機能に対して、独自のコードを実装する必要がありました。 v4 では、現在、ユーザー入力を自動検証し、特定の型 (整数など) に制限して、有効な入力が提供されるまでユーザーに入力を求めるプロンプト クラスがあります。 つまり、一般的に、開発者として書き込むコードが少なくなります。

ダイアログを定義する方法について、いくつかのオプションを使用できるようになりました。

| | |
|:--|:--|
| `ComponentDialog` クラスから派生したコンポーネント ダイアログ | 外部コンテキストとの名前の競合なしにダイアログ コードをカプセル化できます。 「[ダイアログの再利用][reuse-dialogs]」をご覧ください。 |
| `WaterfallDialog` クラスのインスタンスであるウォーターフォール ダイアログ | さまざまな種類のユーザー入力を要求して検証するプロンプト ダイアログと適切に連携するように設計されています。 ウォーターフォールによりプロセスの大部分が自動化されますが、ダイアログ コードに特定の形式が適用されます。[連続して行われる会話フロー][sequential-flow]に関する記事をご覧ください。 |
| 抽象 `Dialog` クラスから派生したカスタム ダイアログ | このダイアログでは、ダイアログの動作に最大限の柔軟性が得られますが、ダイアログ スタックの実装の詳細を把握している必要があります。 |

v3 では、`FormFlow` を使用して、設定されたステップ数を実行していました。 v4 では、ウォーターフォール ダイアログによって FormFlow が置き換えられます。 ウォーターフォール ダイアログを作成するときに、コンストラクターでダイアログのステップを定義します。 実行されるステップの順序は、それを宣言した方法に厳密に従い、1 つずつ自動的に先に進みます。

複数のダイアログを使用して、複雑な制御フローを作成することもできます。[高度な会話フロー][complex-flow]に関する記事をご覧ください。

ダイアログにアクセスするには、そのインスタンスを "_ダイアログ セット_" に追加し、そのセットの "_ダイアログ コンテキスト_" を生成する必要があります。 ダイアログ セットを作成するときには、ダイアログの状態プロパティ アクセサーを提供する必要があります。 これにより、フレームワークはターン間でダイアログの状態を保持できるようになります。 v4 で状態を管理する方法については、「[状態の管理][about-state]」をご覧ください。

### <a name="using-dialogs"></a>ダイアログの使用

v3 での一般的な操作と、ウォーターフォール ダイアログ内でそれらを実行する方法を次に示します。 ウォーターフォール ダイアログの各ステップでは、`DialogTurnResult` 値を返す必要があります。 そうしないと、ウォーターフォールが早期に終了する可能性があります。

| Operation | v3 | v4 |
|:---|:---|:---|
| ダイアログの開始を処理する | `IDialog.StartAsync` を実装する | これをウォーターフォール ダイアログの最初のステップにします。 |
| アクティビティを送信する | `IDialogContext.PostAsync` を呼び出します。 | `ITurnContext.SendActivityAsync` を呼び出します。<br/>ステップ コンテキストの `Context` プロパティを使用して、ターン コンテキストを取得します。  |
| ユーザーの応答を待機する | `IAwaitable<IMessageActivity>` パラメーターを使用し、`IDialogContext.Wait` を呼び出す | プロンプト ダイアログを開始する `ITurnContext.PromptAsync` を待機して制御を戻します。 次に、ウォーターフォールの次のステップで結果を取得します。 |
| ダイアログの継続を処理する | `IDialogContext.Wait` を呼び出します。 | ウォーターフォール ダイアログにステップを追加するか、`Dialog.ContinueDialogAsync` を実装する |
| ユーザーの次のメッセージまで処理が終了したことを伝える | `IDialogContext.Wait` を呼び出します。 | `Dialog.EndOfTurn` を返します。 |
| 子ダイアログを開始する | `IDialogContext.Call` を呼び出します。 | ステップ コンテキストの `BeginDialogAsync` メソッドを待機して制御を戻します。<br/>子ダイアログから値が返された場合、ステップ コンテキストの `Result` プロパティを使用して、ウォーターフォールの次のステップでその値を使用できます。 |
| 現在のダイアログを新しいダイアログに置き換える | `IDialogContext.Forward` を呼び出します。 | `ITurnContext.ReplaceDialogAsync` を待機して制御を戻します。 |
| 現在のダイアログが完了したことを通知する | `IDialogContext.Done` を呼び出します。 | ステップ コンテキストの `EndDialogAsync` メソッドを待機して制御を戻します。 |
| ダイアログを失敗にする | `IDialogContext.Fail` を呼び出します。 | キャッチされた例外をボットの別のレベルでスローするか、`Cancelled` の状態でステップを終了するか、ステップまたはダイアログ コンテキストの `CancelAllDialogsAsync` を呼び出します。<br/>v4 では、ダイアログ内の例外は、ダイアログ スタックではなく C# スタック内で伝達されます。 |

v4 コードに関するその他の注意事項を次に示します。

- v4 のさまざまな `Prompt` 派生クラスでは、ユーザー プロンプトを独立した 2 ステップのダイアログとして実装します。 [連続して行われる会話フローを実装][sequential-flow]する方法をご覧ください。
- `DialogSet.CreateContextAsync` を使用して、現在のターンのダイアログ コンテキストを作成します。
- ダイアログ内から、`DialogContext.Context` プロパティを使用して現在のターン コンテキストを取得します。
- ウォーターフォール ステップには、`DialogContext` から派生した `WaterfallStepContext` パラメーターがあります。
- 具象ダイアログ クラスおよびプロンプト クラスはすべて抽象 `Dialog` クラスから派生します。
- コンポーネント ダイアログを作成するときに ID を割り当てます。 ダイアログ セットの各ダイアログには、そのセット内で一意の ID を割り当てる必要があります。

### <a name="passing-state-between-and-within-dialogs"></a>ダイアログ間およびダイアログ内での状態の受け渡し

「**ダイアログ ライブラリ**」の「[ダイアログの状態](../bot-builder-concept-dialog.md#dialog-state)」、「[ウォーターフォール ステップ コンテキスト プロパティ](../bot-builder-concept-dialog.md#waterfall-step-context-properties)」、「[ダイアログの使用](../bot-builder-concept-dialog.md#using-dialogs)」で、v4 でダイアログの状態を管理する方法を説明しています。

### <a name="iawaitable-is-gone"></a>使用されなくなった IAwaitable

ターンでユーザーのアクティビティを取得するには、ターン コンテキストから取得します。

ユーザーに入力を求め、プロンプトの結果を受け取るには、次の手順に従います。

- ダイアログ セットに適切なプロンプト インスタンスを追加します。
- ウォーターフォール ダイアログのステップからプロンプトを呼び出します。
- 次のステップで、ステップ コンテキストの `Result` プロパティから結果を取得します。

### <a name="formflow"></a>FormFlow

v3 では、FormFlow は C# SDK に含まれていましたが、JavaScript SDK には含まれていませんでした。 v4 SDK には含まれていませんが、C# のコミュニティ バージョンがあります。

| NuGet パッケージ名 | コミュニティの GitHub リポジトリ |
|-|-|
| Bot.Builder.Community.Dialogs.Formflow | [BotBuilderCommunity/botbuilder-community-dotnet/libraries/Bot.Builder.Community.Dialogs.FormFlow](https://github.com/BotBuilderCommunity/botbuilder-community-dotnet/tree/master/libraries/Bot.Builder.Community.Dialogs.FormFlow) |

## <a name="additional-resources"></a>その他のリソース

- [.NET SDK v3 のボットを v4 に移行する](conversion-framework.md)

<!-- -->

[about-bots]: ../bot-builder-basics.md
[about-state]: ../bot-builder-concept-state.md
[about-dialogs]: ../bot-builder-concept-dialog.md

[send-messages]: ../bot-builder-howto-send-messages.md
[send-media]: ../bot-builder-howto-add-media-attachments.md

[sequential-flow]: ../bot-builder-dialog-manage-conversation-flow.md
[complex-flow]: ../bot-builder-dialog-manage-complex-conversation-flow.md
[reuse-dialogs]: ../bot-builder-compositcontrol.md
[interruptions]: ../bot-builder-howto-handle-user-interrupt.md

[アクティビティ スキーマ]: https://aka.ms/botSpecs-activitySchema