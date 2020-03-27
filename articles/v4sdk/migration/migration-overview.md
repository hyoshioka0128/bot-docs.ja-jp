---
title: Bot Framework SDK の移行の概要 - Bot Service
description: SDK の変更点と v3 から v4 への移行方法の概要を説明します。
keywords: ボットの移行
author: mmiele
ms.author: v-mimiel
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 06/11/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 0746a277f465979639db5c8a26aaddfee9386ac2
ms.sourcegitcommit: 772b9278d95e4b6dd4afccf4a9803f11a4b09e42
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/22/2020
ms.locfileid: "80117694"
---
# <a name="migration-overview"></a>移行の概要

Bot Framework SDK v4 は、旧バージョンの SDK のお客様のフィードバックと学習エクスペリエンスに基づいて構築されました。 この新しいバージョンでは、適切なレベルの抽象化を導入し、さらにボット コンポーネントの柔軟なアーキテクチャを実現しています。 たとえば、単純なボットを作成し、Bot Framework SDK v4 のモジュール方式と拡張機能を使用してそれを高度に拡張することができます。

> [!NOTE]
> Bot Framework SDK v4 では、単純なことは単純なまま維持し、複雑なことを実現できるように取り組んでいます。

コミュニティの協力を得て Bot Framework v4 SDK を構築するために、オープンなアプローチが採用されました。 最初に pull request を送信すると、[共同作成者ライセンス契約](https://cla.opensource.microsoft.com/) (CLA) によってライセンスが必要かどうかが自動的に判断されます。 この操作は、すべてのリポジトリで 1 回のみ行う必要があります。 通常、達成する一連の目標を設定する期間があります。

## <a name="what-happens-to-bots-built-using-sdk-v3"></a>SDK v3 を使用して構築されたボットへの影響

Bot Framework SDK v3 は廃止されますが、既存の V3 ボット ワークロードは中断することなく引き続き実行されます。 詳細については、次を参照してください。「[Bot Framework SDK バージョン 3 のライフタイム サポート](https://docs.microsoft.com/azure/bot-service/bot-service-resources-bot-framework-faq?view=azure-bot-service-4.0#bot-framework-sdk-version-3-lifetime-support)」。

V3 ボットから V4 への移行を開始することを強くお勧めします。 この移行をサポートするために、Microsoft では関連ドキュメントを作成しました。また、標準チャネルを介して移行イニシアチブに対する拡張サポートを提供する予定です。

V3 ボットから V4 ボットにすぐに移行できない場合でも、V4 SDK で利用可能な追加機能を利用することができます。 V3 ボットをスキルに変換し、V4 SDK に基づくスキル コンシューマー ボットを作成して、V3 ボットにメッセージを渡すことができます。 詳細については、「[V3 ボットをスキルに変換する](convert-to-skill-overview.md)」を参照してください。

## <a name="advantages"></a>長所

- より豊富で柔軟性の高いオープンなアーキテクチャ:より柔軟な会話設計が可能になります
- 可用性:新しいチャネル機能を使用した追加のシナリオが導入されます
- 開発サイクルにおける領域の専門家 (SME) である個人を拡大します新しい GUI デザイナーで、開発者以外が会話設計で共同作業することができます
- 開発の速度:新しいデバッグおよびテスト開発者ツール
- パフォーマンスの分析情報:会話の品質を評価および改善する新しいテレメトリ機能
- インテリジェンス:認知サービス機能の向上

## <a name="why-migrate"></a>移行する理由
<!-- [!] The declarative model introduced with Adaptive Dialogs would go great here (when ready).  -->
- 柔軟で改善された会話管理
  - アクティビティ処理用の Bot Adapter
  - 状態管理のリファクタリング
  - 新しいダイアログ ライブラリ
  - 構成可能で拡張可能な設計のためのミドルウェア:動作をカスタマイズできるクリーンで一貫性のあるフック
- .NET Core 用に構築されています
  - パフォーマンスの向上
  - クロス プラットフォームの互換性 (Windows/Mac/Linux)
- 複数のプログラミング言語にわたる一貫したプログラミング モデル
- 改善されたドキュメント
- Bot Inspector はデバッグ機能が拡張されました
- 仮想アシスタント
  - 基本的な会話の意図、ディスパッチの統合、QnA Maker、Application Insights、および自動デプロイが搭載された、ボットの作成を簡素化する包括的なソリューション。
  - 拡張可能なスキル。 会話エクスペリエンスを作成するには、"スキル" と呼ばれる再利用可能な会話機能を結合します。
- テスト フレームワーク:トランスポートに依存しない新しいアダプター アーキテクチャを使用した、すぐに利用できるテスト機能
- テレメトリ:会話型 AI 分析では、ご自身のボットの正常性と動作に関する主な分析情報を取得します。
- 今後の予定 (プレビュー)
  - アダプティブ ダイアログ:会話の進行に合わせて動的に変更できる会話を構築します
  - 言語の生成:1 つのフレーズに対して複数のバリエーションを定義します
- 予定
  - 宣言型の設計により、設計者に適したレベルの抽象化を実現できます
  - GUI ダイアログ デザイナー
- Azure Bot Service 
  - Direct Line Speech チャネル。 Bot Framework と Microsoft の Speech Services を統合します。 これにより、クライアントからボット アプリケーションへの双方向のストリーミング音声およびテキストを可能にするチャネルが実現します。

## <a name="whats-changed"></a>変更内容

Bot Framework SDK v4 では、v3 と同じ基になる Bot Framework Service がサポートされています。 ただし、v4 は以前の SDK をリファクターしたものであり、ボットの作成をより柔軟に制御できるようになります。 これには、次の内容が含まれます。

- ボット アダプターを導入しました
  - アダプターはアクティビティ処理スタックに含まれます
  - Bot Framework 認証が処理され、ターンごとにコンテキストが初期化されます
  - Bot Framework Connector サービスの呼び出しがカプセル化され、チャネルとボットのターン ハンドラー間の受信トラフィックと送信トラフィックが管理されます
  - 詳細については、「[ボットのしくみ](../bot-builder-basics.md)」を参照してください
- 状態管理のリファクタリング
  - 状態データは、ボット内で自動的に使用できなくなりました
  - 状態は、状態管理オブジェクトとプロパティ アクセサーを使用して管理されるようになりました
  - 詳細については、「[状態の管理](../bot-builder-concept-state.md)」を参照してください
- 新しいダイアログ ライブラリを導入しました
  - v3 ダイアログは、新しいダイアログ ライブラリ用に書き換える必要があります
  - 詳細については、「[ダイアログ ライブラリ](../bot-builder-concept-dialog.md)」を参照してください

## <a name="whats-involved-in-migration-work"></a>移行作業に必要なこと

- セットアップ ロジックを更新する
- 重要なユーザー状態を移植する
  - 注: 機密性の高いユーザー状態をボット状態に維持することはできません。代わりに、お客様の管理下にある別のストレージに保存してください。
- ボットとダイアログ ロジックを移植する (詳細については言語固有のトピックを参照してください)

### <a name="migration-estimation-worksheet"></a>移行見積りワークシート

次のワークシートは、移行ワークロードの見積もりに役立ちます。 「**発生回数**」列の *count* は実際の数値に置き換えてください。 「**T シャツ**」列に、次のような値を入力します。*Small*、*Medium*、*Large* (見積りに基づく)。

# <a name="c"></a>[C#](#tab/csharp)

| 手順 | V3 | V4 | 発生回数 | 複雑さ | T シャツ |
| -- | -- | -- | -- | -- | -- |
受信アクティビティを取得する | IDialogContext.Activity | ITurnContext.Activity | count | Small  
アクティビティを作成してユーザーに送信する | activity.CreateReply(“text”) IDialogContext.PostAsync | MessageFactory.Text(“text”) ITurnContext.SendActivityAsync | count | Small |
状態管理 | UserData、ConversationData、PrivateConversationData context.UserData.SetValue context.UserData.TryGetValue botDataStore.LoadAsyn | プロパティ アクセサーを使用する UserState、ConversationState、PrivateConversationState | context.UserData.SetValue - count context.UserData.TryGetValue - count botDataStore.LoadAsyn - count | Medium から Large (使用できる[ユーザー状態の管理](https://docs.microsoft.com/azure/bot-service/bot-builder-concept-state?view=azure-bot-service-4.0#state-management)に関する記事を参照してください) |
ダイアログの開始を処理する | IDialog.StartAsync を実装します | これをウォーターフォール ダイアログの最初のステップにします。 | count | Small |  
アクティビティを送信する | IDialogContext.PostAsync。 | ITurnContext.SendActivityAsync を呼び出します。 | count | Small |  
ユーザーの応答を待機する | IAwaitable<IMessageActivity>パラメーターを使用し、IDialogContext.Wait を呼び出します | プロンプト ダイアログを開始する ITurnContext.PromptAsync を待機して制御を戻します。 次に、ウォーターフォールの次のステップで結果を取得します。 | count | Medium (フローに依存) |  
ダイアログの継続を処理する | IDialogContext.Wait | ウォーターフォール ダイアログに手順を追加するか、Dialog.ContinueDialogAsync を実装します | count | Large |  
ユーザーの次のメッセージまで処理が終了したことを伝える | IDialogContext.Wait | Dialog.EndOfTurn を返します。 | count | Medium |  
子ダイアログを開始する | IDialogContext.Call | ステップ コンテキストの BeginDialogAsync メソッドを待機して制御を戻します。 子ダイアログから値が返された場合、ステップ コンテキストの Result プロパティを使用して、ウォーターフォールの次のステップでその値を使用できます。 | count | Medium |  
現在のダイアログを新しいダイアログに置き換える | IDialogContext.Forward | ITurnContext.ReplaceDialogAsync を待機して制御を戻します。 | count | Large |  
現在のダイアログが完了したことを通知する | IDialogContext.Done | ステップ コンテキストの EndDialogAsync メソッドを待機して制御を戻します。 | count | Medium |  
ダイアログを失敗にする | IDialogContext.Fail | キャッチされた例外をボットの別のレベルでスローするか、Cancelled の状態でステップを終了するか、ステップまたはダイアログ コンテキストの CancelAllDialogsAsync を呼び出します。 | count | Small |  

# <a name="javascript"></a>[JavaScript](#tab/javascript)

| 手順 | V3 | V4 | 発生回数 | 複雑さ | T シャツ |
| -- | -- | -- | -- | -- | -- |
受信アクティビティを取得する | IMessage | TurnContext.activity | count | Small  
アクティビティを作成してユーザーに送信する | Session.send('message') を呼び出します。 | TurnContext.sendActivity を呼び出します。 | count | Small |
状態管理 | UserState および ConversationState UserState.get()、UserState.saveChanges()、ConversationState.get()、ConversationState.saveChanges() | プロパティ アクセサーを使用する UserState および ConversationState | count | Medium から Large (使用できる[ユーザー状態の管理](https://docs.microsoft.com/azure/bot-service/bot-builder-concept-state?view=azure-bot-service-4.0#state-management)に関する記事を参照してください) |
ダイアログの開始を処理する | ダイアログの ID を渡して session.beginDialog を呼び出します。 | DialogContext.beginDialog を呼び出します。 | count | Small |  
アクティビティを送信する | Session.send を呼び出します。 | TurnContext.sendActivity を呼び出します。 | count | Small |  
ユーザーの応答を待機する | ウォーターフォール ステップ内からプロンプトを呼び出します。例: builder.Prompts.text(session, 'Please enter your destination')。 次のステップで応答を取得します。 | プロンプト ダイアログを開始する TurnContext.prompt を待機して制御を戻します。 次に、ウォーターフォールの次のステップで結果を取得します。 | count | Medium (フローに依存) |  
ダイアログの継続を処理する | 自動 | ウォーターフォール ダイアログにステップを追加するか、Dialog.continueDialog を実装します。 | count | Large |  
ユーザーの次のメッセージまで処理が終了したことを伝える | Session.endDialog | Dialog.EndOfTurn を返します。 | count | Medium |  
子ダイアログを開始する | Session.beginDialog | ステップ コンテキストの beginDialog メソッドを待機して制御を戻します。 子ダイアログから値が返された場合、ステップ コンテキストの Result プロパティを使用して、ウォーターフォールの次のステップでその値を使用できます。 | count | Medium |  
現在のダイアログを新しいダイアログに置き換える | Session.replaceDialog | ITurnContext.replaceDialog | count | Large |  
現在のダイアログが完了したことを通知する | Session.endDialog | ステップ コンテキストの endDialog メソッドを待機して制御を戻します。 | count | Medium |  
ダイアログを失敗にする | Session.pruneDialogStack | キャッチされた例外をボットの別のレベルでスローするか、Cancelled の状態でステップを終了するか、ステップまたはダイアログ コンテキストの cancelAllDialogs を呼び出します。 | count | Small |  

---

# <a name="c"></a>[C#](#tab/csharp)

Bot Framework SDK v4 と v3 は、基になる REST API を共有しています。 ただし、v4 は、より柔軟にボットを制御できるように、以前のバージョンの SDK をリファクターしたものです。

パフォーマンスが大幅に向上したため、.NET Core への移行をお勧めします。 ただし、既存の V3 ロボットには、.NET Core に相当するものがない外部ライブラリを使用しているものもあります。 この場合、Bot Framework SDK v4 は .NET Framework バージョン 4.6.1 以降で使用できます。 例については、[corebot](https://github.com/microsoft/BotBuilder-Samples/tree/master/samples/csharp_webapi) の場所にあります。

プロジェクトを v3 から v4 に移行するときは、 **.NET Framework** に合わせて変換するか、 **.NET Core** 用の新しいプロジェクトに移植するといういずれかの選択肢を選択できます。

#### <a name="net-framework"></a>.NET Framework

- NuGet パッケージを更新してインストールする
- Global.asax.cs ファイルを更新する
- MessagesController クラスを更新する
- ダイアログを変換する

詳細については、「[.NET v3 ボットを .NET Framework v4 ボットに移行する](conversion-framework.md)」を参照してください。

#### <a name="net-core"></a>.NET Core

- テンプレートを使用して新しいプロジェクトを作成する
- 必要に応じて追加の NuGet パッケージをインストールする
- 自分のボットをカスタマイズし、Startup.cs ファイルを更新して、コントローラー クラスを更新する
- ボット クラスを更新する
- ご利用のダイアログとモデルをコピーして更新する

詳細については、「[.NET v3 ボットを .NET Core v4 ボットに移行する](conversion-core.md)」を参照してください。

# <a name="javascript"></a>[JavaScript](#tab/javascript)

**Bot Framework JavaScript SDK v4** では、ボットの作成方法についていくつかの基本的な変更が加えられています。 これらの変更の結果、特に、ボット オブジェクトの作成、ダイアログの定義、イベント処理ロジックのコーディング関連で、JavaScript でボットを開発するための構文が変更されています。 Bot Framework SDK v4 と v3 は、基になる REST API を共有しています。 ただし、v4 は、より柔軟にボットを制御できるように、以前のバージョンの SDK をリファクターしたものです。特に、

- ダイアログとボット インスタンスはさらに分離されました。 v3 では、ダイアログはボットのコンストラクターに直接登録されました。 v4 では、ダイアログを引数としてボット インスタンスに渡すようになったため、構成における柔軟性が向上しています
- v4 では、`ActivityHandler` クラスを提供しており、メッセージ アクティビティ、会話の更新アクティビティ、イベント アクティビティなど、さまざまな種類のアクティビティの処理を自動化することができます
- NodeJS ボットの移行には、TypeScript または JavaScript で新しい v4 NodeJS ボットを作成する必要があります。 関連する移行ドキュメントに記載されている新しい v4 コンストラクトを使用してボット ロジックを再作成します

#### <a name="migrate-from-v3-to-v4"></a>v3 から v4 に移行する

- 新しいプロジェクトを作成し、依存関係を追加する
- エントリポイントを更新し、定数を定義する
- ダイアログを作成し、SDK v4 を使用してこれを再実装する
- ダイアログを実行するようにボットのコードを更新する
- `store.js` ユーティリティ ファイルを移植する

詳細については、「[SDK v3 JavaScript のボットを v4 に移行する](conversion-javascript.md)」を参照してください。

---

## <a name="additional-resources"></a>その他のリソース

次のその他のリソースは、移行中に役立つ詳細情報を提供します。  

### <a name="c"></a>[C#](#tab/csharp)

<!-- _Mini-TOC with explainer for .NET topics_ -->
次のトピックでは、.NET v3 と v4 の Bot Framework SDK の違い、2 つのバージョンの主な変更点、v3 から v4 にボットを移行する手順が説明されています。

| トピック | 説明 |
| :--- | :--- |
| [.NET SDK v3 と v4 の違い](migration-about.md) |v3 と v4 SDK の一般的な違い |
| [.NET 移行クイック リファレンス](net-migration-quickreference.md) |v3 と v4 SDK の主な変更点 |
| [.NET v3 ボットを Framework v4 ボットに移行する](conversion-framework.md) |同じプロジェクトの種類を使用して v3 ボットを v4 ボットに移行する |
| [.NET v3 ボットを Core v4 ボットに移行する](conversion-core.md) | 新しい .NET Core プロジェクトで v3 から v4 に移行する|

### <a name="javascript"></a>[JavaScript](#tab/javascript)

<!-- _Mini-TOC with explainer for JavaScript topics_ -->
次のトピックでは、JavaScript v3 と v4 の Bot Framework SDK の違い、2 つのバージョンの主な変更点、v3 から v4 にボットを移行する手順が説明されています。

| トピック | 説明 |
| :--- | :--- |
| [JavaScript SDK v3 とv4 の違い](migration-about-javascript.md) | v3 と v4 SDK の一般的な違い |
| [JavaScript 移行クイック リファレンス](javascript-migration-quickreference.md)| v3 と v4 SDK の主な変更点|
| [JavaScript v3 ボットを v4 に移行する](conversion-javascript.md) |v3 ボットを v4 ボットに移行する |

---

### <a name="code-samples"></a>コード サンプル

以下は、Bot Framework SDK V4 の学習やプロジェクトの開始に使用できるコード サンプルです。

| サンプル | 説明 |
| :--- | :--- |
| [Bot Framework V3 から V4 への移行サンプル](https://github.com/microsoft/BotBuilder-Samples/tree/master/MigrationV3V4) <img width="200">| Bot Framework V3 SDK から V4 SDK への移行サンプル |
| [Bot Builder .NET のサンプル](https://github.com/microsoft/BotBuilder-Samples/tree/master/samples/csharp_dotnetcore) | Bot Builder の C#.NET Core サンプル |
| [Bot Builder の JavaScript サンプル](https://github.com/microsoft/BotBuilder-Samples/tree/master/samples/javascript_nodejs) | Bot Builder の JavaScript (node.js) サンプル |
| [Bot Builder のすべてのサンプル](https://github.com/microsoft/BotBuilder-Samples/tree/master/samples) | Bot Builder のすべてのサンプル |

### <a name="getting-help"></a>ヘルプの表示

次のリソースは、ボット開発のための追加情報とサポートを提供します。

[Bot Framework のその他のリソース](https://docs.microsoft.com/azure/bot-service/bot-service-resources-links-help?view=azure-bot-service-4.0)

### <a name="references"></a>References

詳細と背景情報については、次のリソースを参照してください。

| トピック | 説明 |
| :--- | :--- |
| [Bot Framework の新機能](https://docs.microsoft.com/azure/bot-service/what-is-new?view=azure-bot-service-4.0) | Bot Framework と Azure Bot Service の主な機能と機能強化|
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
