---
title: ダイアログの状態 | Microsoft Docs
description: Bot Builder SDK 内で状態がどのように機能するかについて説明します。
keywords: 状態, ボットの状態, 会話の状態, ユーザーの状態
author: johnataylor
ms.author: johtaylo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 9/21/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 4da715650eb1c3e7b284aaa47aa5ce4ac7280f4c
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/24/2018
ms.locfileid: "49998759"
---
# <a name="dialog-state"></a>ダイアログの状態

ダイアログはマルチターンの会話ロジックを実装するためのアプローチの 1 つであり、保持された状態に基づく SDK の機能の一例です。 

ダイアログ ベースのボットは通常、そのボットの実装のメンバー変数として DialogSet コレクションを保持します。 DialogSetは、アクセサーというオブジェクトへのハンドルを使用して作成されます。 

アクセサーは、SDK の状態モデルにおける中心的な概念です。 アクセサーは IStatePropertyAccessor インターフェイスを実装します。これは、Get、Set、Delete の各メソッドの実装を提供しなければならないことを意味します。 アクセサーを作成するには、アクセサーに対してプロパティ名を指定します。 

アクセサーを使用するコードは、アクセサーの Get、Set、Delete メソッドを呼び出すことができ、これらはその名前のプロパティを参照します。 より上位コンポーネントで一部の状態の永続化が必要である場合は、アクセサーを使用する必要があります。 この方法なら、さまざまなシナリオで状態ストレージの実装を利用しつつ、同じ上位レベルのコードを使用することができます。 たとえば、ダイアログではアクセサーを使用しますが、これは永続化された状態にアクセスするための唯一の方法です。

![ダイアログの状態](media/bot-builder-dialog-state.png)

ボットの OnTurn が呼び出されると、ボットは DialogSet で CreateContext を呼び出すことにより、ダイアログのサブシステムを初期化します。 DialogContext の作成には状態が必要であるため、DialogSet はアクセサーを使用して適切なダイアログの状態の JSON を取得します。 SDK は BotState クラスの形でアクセサーの実装を提供します。 状態の実装を利用するアプリケーションは、BotState をサブクラス化して、アクセサーの実装を継承できます。 SDK に含まれる BotState には、次の 2 つのサブクラスがあります。

- UserState
- ConversationState

UserState と ConversationState は、基盤となるストレージ用のキーを使用します。 このキーは物理ストレージに渡されます。 論理的には、このキーはアクセサーによって指定されたプロパティの名前空間として機能します。 BotState の実装は、内部的に TurnContext で受信アクティビティを使用して、使用するストレージ キーを動的に生成します。

- UserState は、Channel Id と From Id を使用してキーを生成します。例: _{Activity.ChannelId}/conversations/{Activity.From.Id}#DialogState_
- ConversationState は、Channel Id と Conversation Id を使用してキーを生成します。例: _{Activity.ChannelId}/users/{Activity.Conversation.Id}#YourPropertyName_

アプリケーションは、アクセサーのほか、動的に作成されたストレージ キーとプロパティ名への適切なバインドを提供する必要があります。これらはすべてバックグラウンドで処理されます。 アクセサーの BotState の実装には、いくつかの最適化が含まれます。 

- 1 つ目の最適化は、キャッシュと遅延読み込みです。 実際のストレージ実装での読み込みは、アクセサーに対する最初の Get が呼び出されるまで延期され、読み込みの結果はキャッシュに保持されます。 このキャッシュへの参照は、対応する BotState によって提供されるキーを使用して TurnContext に追加されます。 したがって、UserState に対応するキャッシュは "UserState" というフィールドに、ConversationState に対応するキャッシュは "ConversationState" というフィールドに保持されます。 さまざまなアクセサー オブジェクトに対する呼び出しは TurnContext に渡されるため、適切なキャッシュに移動してそれをフェッチできます。

- 2 つ目の最適化は、状態に何らかの変更が行われたかどうかを判断するロジックが BotState クラスに含まれることです。 変更が行われた場合にのみ、基盤となるストレージで実際の保存操作が実行されます。

- BotState 実装内の 3 つ目の最適化は、BotStateSet というクラスにあります。 BotStateSet は BotState オブジェクトのコレクションであり、その SaveChanges メソッドの呼び出しをコレクションのすべてのメンバーに並列でデリゲートします。

重要なのは、BotState での SaveChanges の呼び出しが IStatePropertyAccessor インターフェイスの一部ではないことです。 これは、SaveChanges がこのモデルの核となる要素ではなく、BotState 実装の固有の最適化であるからです。 具体的には、ダイアログなどのコードは SaveChanges はもちろん、BotState についても関知しません。 事実、ダイアログのコードはアクセサーにのみ結合されます。 これは、SaveChanges メソッドが実行の完了後にダイアログ システムの外部で呼び出されるようにするためです。 たとえば、この図に示すように、ボットの OnTurn メソッド内から呼び出すことができます。

BotState の実装により、いくつか特別なセマンティクスがもたらされることに注意してください。 とりわけ、最後の書き込みが以前に書き込まれた状態を上書きする、"最後の書き込みが有効" という動作をサポートすることに注意が必要です。 これは多くのアプリケーションで問題なく動作しますが、影響を及ぼすこともあります (特に、何らかのレベルのコンカレンシーが機能することのあるスケールアウト シナリオの場合)。 これが問題になる場合は、独自のアクセサーを実装して、ダイアログに渡すとよいでしょう。 別のアプローチも多数あります。たとえば、Azure Storage などのクラウド ストレージ サービスでよく使われる、eTag の条件を利用するソリューションもあります。 この場合のソリューションでは、他の重要なパーツを実装するものと思われます。 たとえば、送信アクティビティをバッファー処理して、保存操作が成功した後にのみそれを送信する場合もあります。 重要な点は、この動作が BotState 実装の動作ではなく、アプリケーションがアクセサー レベルで提供し、組み込むものだということです。

BotState 実装自体に、プラグ可能なストレージ モデルがあります。 これはシンプルな読み込み/保存パターンに従ったものであり、SDK は運用環境向けに 2 つの代替実装を提供します。 1 つは Azure Storage 用で、もう 1 つは CosmosDB 用です。テスト目的のメモリ内実装もあります。 ただし、ここで重要なのは、"最後の書き込みが有効" というセマンティクスが BotState 実装によって指定されるということです。

## <a name="handling-state-in-middleware"></a>ミドルウェアでの状態の処理
上の図は、ボットの OnTurn の最後に発生する SaveAllChanges への呼び出しを示しています。 呼び出しに焦点を当てた同じ図を次に示します。

![状態ミドルウェアの問題](media/bot-builder-dialog-state-problem.png)

このアプローチの問題は、ボットの OnTurn メソッドが戻った後に発生する、カスタム ミドルウェアで行われた状態の更新が永続ストレージに保存されないことです。 これを解決するには、AutoSaveChangesMiddleware をミドルウェア スタックの最後に追加することにより、SaveAllChanges への呼び出しをカスタム ミドルウェアの完了後に移動します。 実行は次のようになります。

![状態ミドルウェアの解決策](media/bot-builder-dialog-state-solution.png)

## <a name="additional-resources"></a>その他のリソース
詳細については、GitHub の Bot Builder SDK [[C#](https://github.com/Microsoft/BotBuilder-dotnet) | [JavaScript](https://github.com/Microsoft/BotBuilder-js)] のページを参照してください。
