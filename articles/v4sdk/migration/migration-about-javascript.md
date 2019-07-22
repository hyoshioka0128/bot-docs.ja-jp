---
title: NodeJS SDK v3 と v4 の違い | Microsoft Docs
description: NodeJS SDK v3 と v4 の違いについて説明します。
keywords: ボットの移行、ダイアログ、状態
author: mmiele
ms.author: v-mimiel
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 07/08/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 24725df2f109e73df142194c05ee27c2d8ba5da4
ms.sourcegitcommit: 4f78e68507fa3594971bfcbb13231c5bfd2ba555
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/17/2019
ms.locfileid: "68292212"
---
# <a name="differences-between-the-v3-and-v4-javascript-sdk"></a>JavaScript SDK v3 とv4 の違い

Bot Framework SDK のバージョン 4 では、バージョン 3 と同じ基になる Bot Framework Service がサポートされています。 ただし、v4 は、開発者がより柔軟にボットを制御できるように、以前のバージョンの SDK をリファクタリングしたものです。 この SDK の主な変更点は次のとおりです。

- ボット アダプターの導入
  - アクティビティ処理スタックに含まれています
  - Bot Framework 認証が処理されます
  - Bot Framework Connector の呼び出しがカプセル化され、チャネルとボットのターン ハンドラー間の受信トラフィックと送信トラフィックが管理されます
  - ターンごとにコンテキストが初期化されます
  - 詳細については、[ボットのしくみ](../bot-builder-basics.md)に関する記事を参照してください。
- 状態管理のリファクタリング
  - 状態データは、ボット内で自動的に使用できなくなりました
  - 状態管理オブジェクトとプロパティ アクセサーを使用して管理されるようになりました。
  - 詳細については、[状態の管理](../bot-builder-concept-state.md)に関する記事を参照してください。
- 新しいダイアログ ライブラリ
  - v3 ダイアログは、新しいダイアログ ライブラリ用に書き換える必要があります
  - scoreable は存在しなくなりました。 ダイアログに制御を渡す前に、"グローバル" コマンドの有無を確認できます。 ご自身の v4 ボットの設計方法に応じて、これはメッセージ ハンドラーまたは親ダイアログに存在します。 例については、[ユーザーによる割り込みを処理](../bot-builder-howto-handle-user-interrupt.md)する方法を参照してください。
  - 詳細については、[ダイアログ ライブラリ](../bot-builder-concept-dialog.md)に関する記事を参照してください。

## <a name="activity-processing"></a>アクティビティの処理

ボットのアダプターを作成するときは、チャネルとユーザーからの受信アクティビティを受け取るメッセージ ハンドラー デリゲートも用意します。 アダプターは、受け取ったアクティビティごとにターン コンテキスト オブジェクトを作成します。 ターン コンテキスト オブジェクトがボットのターン ハンドラーに渡され、ターンが完了するとオブジェクトは破棄されます。

ターン ハンドラーは、さまざまな種類のアクティビティを受け取ることができます。 一般に、ボットに含まれているダイアログには "_メッセージ_" アクティビティだけを転送します。 ご自身のボットを `ActivityHandler` から派生させると、すべてのメッセージ アクティビティがボットのターン ハンドラーによって `OnMessage` に転送されます。 このメソッドをオーバーライドして、メッセージ処理ロジックを追加します。 アクティビティの種類の詳細については、[アクティビティ スキーマ](https://github.com/Microsoft/botframework-sdk/blob/master/specs/botframework-activity/botframework-activity.md)に関する記事を参照してください。

### <a name="handling-turns"></a>ターンの処理

メッセージを処理するときは、ターン コンテキストを使用して受信アクティビティに関する情報を取得し、アクティビティをユーザーに送信します。

|アクティビティ |説明 |
|:---|:---|
| 受信アクティビティを取得する | ターン コンテキストの `Activity` プロパティを取得します。 |
| アクティビティを作成してユーザーに送信する | ターン コンテキストの `SendActivity` メソッドを呼び出します。<br/> 詳細については、[テキスト メッセージの送受信](../../rest-api/bot-framework-rest-direct-line-1-1-receive-messages.md)と[メッセージへのメディアの追加](../bot-builder-howto-add-media-attachments.md)に関する記事を参照してください。 |

`MessageFactory` クラスには、アクティビティを作成し、その形式を設定するためのヘルパー メソッドがいくつか用意されています。

### <a name="interruptions"></a>割り込み

これらはボットのメッセージ ループ内で処理します。 v4 ダイアログでこれを行う方法については、「[ユーザーによる割り込みを処理する](../bot-builder-howto-handle-user-interrupt.md)」をご覧ください。

## <a name="state-management"></a>状態管理

v3 では、会話データを、Bot Framework によって提供される大規模なサービス スイートの一部である Bot State Service に格納できました。 ただし、このサービスは、2018 年 3 月 31 日に廃止されました。 v4 以降、状態の管理に関する設計上の考慮事項は、Web アプリと同じで、さまざまなオプションを使用できます。 これをメモリおよび同じプロセス内にキャッシュするのが通常は最も簡単なのですが、運用環境のアプリについては、状態をより永続的に、たとえば SQL、NoSQL データベース、BLOB などに格納する必要があります。

v4 では、状態を管理する際に、`UserData`、`ConversationData`、`PrivateConversationData` の各プロパティとデータ バッグは使用されません。
状態は、「[状態の管理](../bot-builder-concept-state.md)」で説明する状態管理オブジェクトとプロパティ アクセサーを使用して管理されるようになりました。

v4 では、ボットの状態データを管理する、`UserState`、`ConversationState`、`PrivateConversationState` の各クラスが定義されています。 定義済みのデータ バッグを単に読み書きするのではなく、保持するプロパティごとに状態プロパティ アクセサーを作成する必要があります。

### <a name="setting-up-state"></a>状態の設定

状態はアプリケーションのエントリ ポイント ファイル、一般的には NodeJS アプリケーションの 'index.js' または 'app.js' で構成する必要があります。 

1. botbuilder-core によって提供される `Storage` インターフェイスを実装する 1 つ以上のオブジェクトを初期化します。 これは、ボットのデータのバッキング ストアを表します。
    v4 SDK では、[ストレージ レイヤー](../bot-builder-concept-state.md#storage-layer)がいくつか提供されます。
    別の種類のストアに接続するために、独自のレイヤーを実装することもできます。
1. 必要に応じて、[状態管理](../bot-builder-concept-state.md#state-management)オブジェクトを作成し、登録します。
    v3 の場合と同じスコープを使用できます。また、必要に応じて、他のスコープを作成することもできます。
1. 最後に、ボットに必要なプロパティの[状態プロパティ アクセサー](../bot-builder-concept-state.md#state-property-accessors)を作成し、登録します。
    状態管理オブジェクト内では、各プロパティ アクセサーに一意の名前が必要です。

### <a name="using-state"></a>状態の使用

状態プロパティ アクセサーを使用してプロパティを取得および更新し、状態管理オブジェクトを使用して変更をストレージに書き込みます。 コンカレンシーの問題を考慮する必要があることを理解しているという前提で、一般的なタスクを実行する方法を次に示します。

| タスク|説明 |
|:---|:---|
| 状態プロパティ アクセサーを作成する | `BotState` オブジェクトの `createProperty` メソッドを呼び出します。 <br/>`BotState` は、会話状態、個人的な会話状態、ユーザー状態を表す抽象基底クラスです。 |
| プロパティの現在の値を取得する | `StatePropertyAccessor.get(TurnContext)` を呼び出します。<br/>以前に値が設定されていない場合は、出荷時の既定のパラメーターを使用して値が生成されます。 |
| 状態の変更をストレージに保持する | ターン ハンドラーを終了する前に、状態が変わった状態管理オブジェクトに対して `BotState.saveChanges(TurnContext, boolean)` を呼び出します。 |

### <a name="managing-concurrency"></a>コンカレンシーを管理する

ボットで状態のコンカレンシーを管理することが必要な場合があります。 詳細については、「**状態の管理**」の「[状態の保存](../bot-builder-concept-state.md#saving-state)」、および「**ストレージに直接書き込む**」の「[eTag を使用してコンカレンシーを管理する](../bot-builder-howto-v4-storage.md#manage-concurrency-using-etags)」をご覧ください。

## <a name="dialogs-library"></a>ダイアログ ライブラリ

ダイアログの主な変更点を次に示します。

- ダイアログ ライブラリは、**botbuilder-dialogs** という独立した npm パッケージになりました。
- ダイアログの状態は、`DialogState` 状態クラスとプロパティ アクセサーを使用して管理されます。
  - ダイアログ オブジェクト自体とは異なり、ダイアログの状態プロパティはターン間で保持されるようになりました。
- ターン内で、"_ダイアログ セット_" のダイアログ コンテキストを作成します。
  - このダイアログ コンテキストによってダイアログ スタックがカプセル化されます。 この情報は、ダイアログの状態プロパティ内に保持されます。
- どちらのバージョンにも抽象 `Dialog` クラスがありますが、v3 では 'ActionSet' クラスが拡張され、v4 では 'Object' が拡張されます。

### <a name="defining-dialogs"></a>ダイアログの定義

v3 では `Dialog` クラスを使用して柔軟にダイアログを実装できましたが、検証などの機能に対して、独自のコードを実装する必要がありました。 v4 では、現在、ユーザー入力を自動検証し、特定の型 (整数など) に制限して、有効な入力が提供されるまでユーザーに入力を求めるプロンプト クラスがあります。 つまり、一般的に、開発者として書き込むコードが少なくなります。

ダイアログを定義する方法について、いくつかのオプションを使用できるようになりました。

|ダイアログの種類| 説明 |
|:---|:---|
| `ComponentDialog` クラスから派生したコンポーネント ダイアログ | 外部コンテキストとの名前の競合なしにダイアログ コードをカプセル化できます。 「[ダイアログの再利用](../bot-builder-concept-dialog.md)」をご覧ください。|
| `WaterfallDialog` クラスのインスタンスであるウォーターフォール ダイアログ | さまざまな種類のユーザー入力を要求して検証するプロンプト ダイアログと適切に連携するように設計されています。 ウォーターフォールによりプロセスの大部分が自動化されますが、ダイアログ コードに特定の形式が適用されます。[連続して行われる会話フロー](../bot-builder-dialog-manage-conversation-flow.md)に関する記事を参照してください。 |
| 抽象 `Dialog` クラスから派生したカスタム ダイアログ | このダイアログでは、ダイアログの動作に最大限の柔軟性が得られますが、ダイアログ スタックの実装の詳細を把握している必要があります。 |

ウォーターフォール ダイアログを作成するときに、コンストラクターでダイアログのステップを定義します。 実行されるステップの順序は、それを宣言した方法に厳密に従い、1 つずつ自動的に先に進みます。

複数のダイアログを使用して、複雑な制御フローを作成することもできます。[高度な会話フロー](../bot-builder-dialog-manage-complex-conversation-flow.md)に関する記事を参照してください。

ダイアログにアクセスするには、そのインスタンスを "_ダイアログ セット_" に追加し、そのセットの "_ダイアログ コンテキスト_" を生成する必要があります。 ダイアログ セットを作成するときには、ダイアログの状態プロパティ アクセサーを提供する必要があります。 これにより、フレームワークはターン間でダイアログの状態を保持できるようになります。 v4 で状態を管理する方法については、「[状態の管理](../bot-builder-concept-state.md)」をご覧ください。

### <a name="using-dialogs"></a>ダイアログの使用

v3 での一般的な操作と、ウォーターフォール ダイアログ内でそれらを実行する方法を次に示します。 ウォーターフォール ダイアログの各ステップでは、`DialogTurnResult` 値を返す必要があります。 そうしないと、ウォーターフォールが早期に終了する可能性があります。

| Operation | v3 | v4 |
|:---|:---|:---|
| ダイアログの開始を処理する | `session.beginDialog` を呼び出し、ダイアログの ID で渡します | `DialogContext.beginDialog` を呼び出して |
| アクティビティを送信する | `session.send` を呼び出します。 | `TurnContext.sendActivity` を呼び出します。<br/>ステップ コンテキストの `Context` プロパティを使用して、ターン コンテキスト (`step.context.sendActivity`) を取得します。  |
| ユーザーの応答を待機する | ウォーターフォール ステップ内からプロンプトを呼び出します (例: `builder.Prompts.text(session, 'Please enter your destination')`)。 次のステップで応答を取得します。 | プロンプト ダイアログを開始する `TurnContext.prompt` を待機して制御を戻します。 次に、ウォーターフォールの次のステップで結果を取得します。 |
| ダイアログの継続を処理する | 自動 | ウォーターフォール ダイアログにステップを追加するか、`Dialog.continueDialog` を実装する |
| ユーザーの次のメッセージまで処理が終了したことを伝える | `session.endDialog` を呼び出します。 | `Dialog.EndOfTurn` を返します。 |
| 子ダイアログを開始する | `session.beginDialog` を呼び出します。 | ステップ コンテキストの `beginDialog` メソッドを待機して制御を戻します。<br/>子ダイアログから値が返された場合、ステップ コンテキストの `Result` プロパティを使用して、ウォーターフォールの次のステップでその値を使用できます。 |
| 現在のダイアログを新しいダイアログに置き換える | `session.replaceDialog` を呼び出します。 | `ITurnContext.replaceDialog` を待機して制御を戻します。 |
| 現在のダイアログが完了したことを通知する | `session.endDialog` を呼び出します。 | ステップ コンテキストの `endDialog` メソッドを待機して制御を戻します。 |
| ダイアログを失敗にする | `session.pruneDialogStack` を呼び出します。 | キャッチされた例外をボットの別のレベルでスローするか、`Cancelled` の状態でステップを終了するか、ステップまたはダイアログ コンテキストの `cancelAllDialogs` を呼び出します。 |

v4 コードに関するその他の注意事項を次に示します。

- v4 のさまざまな `Prompt` 派生クラスでは、ユーザー プロンプトを独立した 2 ステップのダイアログとして実装します。 [連続して行われる会話フローを実装する][sequential-flow]方法を参照してください。
- `DialogSet.createContext` を使用して、現在のターンのダイアログ コンテキストを作成します。
- ダイアログ内から、`DialogContext.context` プロパティを使用して現在のターン コンテキストを取得します。
- ウォーターフォール ステップには、`DialogContext` から派生した `WaterfallStepContext` パラメーターがあります。
- 具象ダイアログ クラスおよびプロンプト クラスはすべて抽象 `Dialog` クラスから派生します。
- コンポーネント ダイアログを作成するときに ID を割り当てます。 ダイアログ セットの各ダイアログには、そのセット内で一意の ID を割り当てる必要があります。

### <a name="passing-state-between-and-within-dialogs"></a>ダイアログ間およびダイアログ内での状態の受け渡し

「**ダイアログ ライブラリ**」の「[ダイアログの状態](../bot-builder-concept-dialog.md#dialog-state)」、「[ウォーターフォール ステップ コンテキスト プロパティ](../bot-builder-concept-dialog.md#waterfall-step-context-properties)」、「[ダイアログの使用](../bot-builder-concept-dialog.md#using-dialogs)」で、v4 でダイアログの状態を管理する方法を説明しています。

### <a name="get-user-response"></a>ユーザーの応答を取得する

ターンでユーザーのアクティビティを取得するには、ターン コンテキストから取得します。

ユーザーに入力を求め、プロンプトの結果を受け取るには、次の手順に従います。

- ダイアログ セットに適切なプロンプト インスタンスを追加します。
- ウォーターフォール ダイアログのステップからプロンプトを呼び出します。
- 次のステップで、ステップ コンテキストの `Result` プロパティから結果を取得します。

## <a name="additional-resources"></a>その他のリソース

詳細と背景情報については、次のリソースを参照してください。

| トピック | 説明 |
| :--- | :--- |
|[JavaScript SDK v3 のボットを v4 に移行する](https://docs.microsoft.com/en-us/azure/bot-service/migration/conversion-javascript?view=azure-bot-service-4.0)| JavaScript v3 ボットの v4 への移植|
| [Bot Framework の新機能](https://docs.microsoft.com/en-us/azure/bot-service/what-is-new?view=azure-bot-service-4.0) | Bot Framework と Azure Bot Service の主な機能と機能強化|
|[ボットのしくみ](../bot-builder-basics.md)|ボットの内部メカニズム|
|[状態の管理](../bot-builder-concept-state.md)|状態の管理を容易にすることができる抽象化|
|[ダイアログ ライブラリ](../bot-builder-concept-dialog.md)| 会話を管理するための中心的な概念|
|[テキスト メッセージを送受信する](../bot-builder-howto-send-messages.md)|ボットがユーザーと通信するための主な方法|
|[メディアを送信する](../bot-builder-howto-add-media-attachments.md)|画像、ビデオ、オーディオ、ファイルなどのメディアの添付ファイル| 
|[連続して行われる会話フロー](../bot-builder-dialog-manage-conversation-flow.md)| ボットがユーザーと対話する主な方法としての質問|
|[ユーザーと会話データを保存する](../bot-builder-howto-v4-state.md)|ステートレスでの会話の追跡|
|[複雑なフロー](../bot-builder-dialog-manage-complex-conversation-flow.md)|複雑な会話フローを管理します |
|[ダイアログの再利用](../bot-builder-compositcontrol.md)|特定のシナリオを処理する独立したダイアログを作成します|
|[割り込み](../bot-builder-howto-handle-user-interrupt.md)| 堅牢なボットを作成するための割り込み処理|
|[アクティビティ スキーマ](https://aka.ms/botSpecs-activitySchema)|人間と自動化されたソフトウェアのためのスキーマ|