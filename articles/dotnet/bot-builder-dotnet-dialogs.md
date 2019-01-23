---
title: ダイアログの概要 | Microsoft Docs
description: Bot Framework SDK for .NET 内でダイアログを使用して会話をモデル化し、会話フローを管理する方法を説明します。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 3089e7a073f6a6d9af3a3720954af3a915106888
ms.sourcegitcommit: b15cf37afc4f57d13ca6636d4227433809562f8b
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/11/2019
ms.locfileid: "54224997"
---
# <a name="dialogs-in-the-bot-framework-sdk-for-net"></a>Bot Framework SDK for .NET のダイアログ

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-dialogs.md)
> - [Node.js](../nodejs/bot-builder-nodejs-dialog-overview.md)

Bot Framework SDK for .NET を使用してボットを作成する場合、ダイアログを使用して会話をモデル化し、[会話フロー](../bot-service-design-conversation-flow.md)を管理できます。 各ダイアログは、`IDialog` を実装する C# クラスに独自の状態をカプセル化する抽象化です。 ダイアログは、他のダイアログと組み合わせて最大限に再利用することができます。ダイアログ コンテキストは、任意の時点における会話内のアクティブな[ダイアログのスタック](../bot-service-design-conversation-flow.md#dialog-stack)を保持します。 

ダイアログを構成する会話はコンピューター間で移植可能です。これにより、ボットの実装をスケーリングすることができます。 Bot Framework SDK for .NET でダイアログを使用すると、会話状態 (ダイアログ スタックとスタック内の各ダイアログの状態) が、ユーザーが選択した[状態データ](bot-builder-dotnet-state.md) ストレージに自動的に格納されます。 これにより、ボットのサービス コードをステートレスにすることができます。これは、Web サーバーのメモリにセッション状態を保存する必要のない Web アプリケーションと似ています。 

## <a name="echo-bot-example"></a>エコー ボットの例

このエコー ボットの例では、[クイック スタート](bot-builder-dotnet-quickstart.md) チュートリアルで作成したボットを、ダイアログを使用してユーザーとメッセージを交換するように変更する方法を示します。

> [!TIP]
> この例を実行するには、[クイック スタート](bot-builder-dotnet-quickstart.md) チュートリアルの手順を使用してボットを作成した後、次に説明するように **MessagesController.cs** ファイルを更新します。

### <a name="messagescontrollercs"></a>MessagesController.cs 

Bot Framework SDK for .NET では、[Builder][builderLibrary] ライブラリを使用してダイアログを実装できます。 関連するクラスにアクセスするには、`Dialogs` 名前空間をインポートします。

[!code-csharp[Using statement](../includes/code/dotnet-dialogs.cs#usingStatement)]

次に、この `EchoDialog` クラスを **MessagesController.cs** に追加して会話を表します。 

[!code-csharp[EchoDialog class](../includes/code/dotnet-dialogs.cs#echobot1)]

次に、`Conversation.SendAsync` メソッドを呼び出すことで、`EchoDialog` クラスを `Post` メソッドに連結します。

[!code-csharp[Post method](../includes/code/dotnet-dialogs.cs#echobot2)]

### <a name="implementation-details"></a>実装の詳細 

Bot Builder では非同期通信の処理に C# の機能を使用するため、`Post` メソッドは `async` とマークされています。 これは、渡されたメッセージに返信する役割を持つタスクを表す `Task` オブジェクトを返します。 例外がある場合、このメソッドによって返される `Task` に例外情報が含まれます。 

`Conversation.SendAsync` メソッドは、Bot Framework SDK for .NET を使用してダイアログを実装するための重要な要素です。 これは、<a href="https://en.wikipedia.org/wiki/Dependency_inversion_principle" target="_blank">依存関係逆転の原則</a>に従い、次のステップを実行します。

1. 必要なコンポーネントをインスタンス化する  
2. `IBotDataStore` からの会話状態 (ダイアログ スタックとスタック内の各ダイアログの状態) を逆シリアル化する
3. ボットが中断状態でメッセージを待っている会話プロセスを再開する
4. 返信を送信する
5. 更新された会話状態をシリアル化し、再び `IBotDataStore` に保存する

会話が最初に開始されたとき、ダイアログには状態が含まれないため、`Conversation.SendAsync` は `EchoDialog` を構築し、`StartAsync` メソッドを呼び出します。 `StartAsync` メソッドは `IDialogContext.Wait` を継続デリゲートと共に呼び出して、新しいメッセージが受信されたときに呼び出すメソッドを指定します (`MessageReceivedAsync`)。 

`MessageReceivedAsync` メソッドは、メッセージを待ち、応答を投稿し、次のメッセージを待ちます。 `IDialogContext.Wait` が呼び出されるたびに、ボットは中断状態になり、メッセージを受信したすべてのコンピューターで再起動できます。 

上記のサンプル コードを使用して作成されたボットは、ユーザーから送信された各メッセージに対し、テキスト "You said:" がプレフィックスとして付加されたユーザーのメッセージを単にエコー バックして応答します。 ボットはダイアログを使用して作成されるため、明示的に状態を管理することなく、より複雑な会話をサポートするように進化させることができます。

## <a name="echo-bot-with-state-example"></a>状態を使用するエコー ボットの例

この次の例は、上記の例にダイアログの状態を追跡する機能を追加したものです。 次のサンプル コードに示すように、`EchoDialog` クラスが更新されると、ボットは、ユーザーから送信された各メッセージに対し、数字 (`count`) に続けてテキスト "You said:" がプレフィックスとして付加されたユーザーのメッセージをエコー バックして応答します。 ボットは、ユーザーがカウントをリセットするまで、返信ごとに `count` の増分を続けます。

### <a name="messagescontrollercs"></a>MessagesController.cs 

[!code-csharp[EchoDialog class](../includes/code/dotnet-dialogs.cs#echobot3)]

### <a name="implementation-details"></a>実装の詳細

最初の例のように、新しいメッセージが受信されると `MessageReceivedAsync` メソッドが呼び出されます。 ただし、今回、`MessageReceivedAsync` メソッドは、応答する前にユーザーのメッセージを評価します。 ユーザーのメッセージが "リセット" されると、ユーザーにカウントのリセットの確認を求めるサブダイアログが組み込みの `PromptDialog.Confirm` プロンプトによって生成されます。 サブダイアログには、親ダイアログの状態に干渉しない独自のプライベート状態があります。 ユーザーがプロンプトに応答すると、サブダイアログの結果が `AfterResetAsync` メソッドに渡されます。このメソッドは、カウントがリセットされたかどうかを示すメッセージをユーザーに送信し、その次のメッセージで `IDialogContext.Wait` を継続と共に `MessageReceivedAsync` にコールバックします。

## <a name="dialog-context"></a>ダイアログ コンテキスト

各ダイアログ メソッドに渡される `IDialogContext` インターフェイスは、ダイアログが状態を保存してチャネルと通信するために必要なサービスへのアクセスを提供します。 `IDialogContext` インターフェイスは、次の 3 つのインターフェイスで構成されます:[Internals.IBotData][iBotData]、[Internals.IBotToUser][iBotToUser]、[Internals.IDialogStack][iDialogStack]。 

### <a name="internalsibotdata"></a>Internals.IBotData

`Internals.IBotData` インターフェイスは、コネクタによって管理されるユーザー単位、会話単位、およびプライベートの会話状態データへのアクセスを提供します。 ユーザー単位の状態データは、特定の会話に関連しないユーザー データを格納するのに便利です。一方、会話単位のデータは、会話に関する一般的なデータを保存するのに便利です。また、プライベートの会話データは、特定の会話に関連するユーザー データを格納するのに便利です。 

### <a name="internalsibottouser"></a>Internals.IBotToUser

`Internals.IBotToUser` は、ボットからユーザーにメッセージを送信するためのメソッドを提供します。 メッセージは、Web API メソッド呼び出しに対する応答と共にインラインで送信することも、[コネクタ クライアント](bot-builder-dotnet-connector.md#create-a-connector-client)を使用して直接送信することもできます。 ダイアログ コンテキストを介してメッセージを送受信することにより、`Internals.IBotData` 状態は必ずコネクタを経由します。

### <a name="internalsidialogstack"></a>Internals.IDialogStack

`Internals.IDialogStack` は、[ダイアログ スタック](../bot-service-design-conversation-flow.md#dialog-stack)を管理するためのメソッドを提供します。 ほとんどの場合、ダイアログ スタックは自動的に管理されます。 ただし、スタックを明示的に管理したい場合もあります。 たとえば、子ダイアログを呼び出してそれをダイアログ スタックの一番上に追加する場合、現在のダイアログを完了としてマークする場合 (これにより、ダイアログをダイアログ スタックから削除して、結果をスタックの前のダイアログに返す)、ユーザーからのメッセージが到着するまで現在のダイアログを中断する場合、ダイアログ スタック全体をリセットする場合などです。

## <a name="serialization"></a>シリアル化

ダイアログ スタックとすべてのアクティブ ダイアログの状態は、ユーザー単位、会話単位の [IBotDataBag][iBotDataBag] にシリアル化されます。 シリアル化された BLOB は、ボットと[コネクタ](bot-builder-dotnet-concepts.md#connector)の間で送受信される各メッセージ内で保持されます。 シリアル化するには、`Dialog` クラスに `[Serializable]` 属性を含める必要があります。 [Builder][builderLibrary] ライブラリ内のすべての `IDialog` 実装は、シリアル化可能とマークされています。 

[Chain メソッド](#dialog-chains)は、LINQ クエリ構文で使用できる fluent インターフェイスをダイアログに提供します。 LINQ クエリ構文のコンパイル済みの形式では、多くの場合匿名メソッドが使用されます。 これらの匿名メソッドがローカル変数の環境を参照しない場合、これらの匿名メソッドは状態を持たず、簡単にシリアル化可能です。 ただし、匿名メソッドが環境内のいずれかのローカル変数を取得した場合、(コンパイラによって生成される) 結果のクロージャ オブジェクトはシリアル化可能とマークされません。 このような状況で、Bot Builder は、問題を識別するために `ClosureCaptureException` をスローします。

リフレクションを使用して、シリアル化可能とマークされていないクラスをシリアル化するために、Builder ライブラリには、[Autofac][autofac] への登録に使用できるリフレクションベースのシリアル化サロゲートが含まれます。

[!code-csharp[Serialization](../includes/code/dotnet-dialogs.cs#serialization)]

## <a id="dialog-chains"></a> Dialog チェーン

`IDialogStack.Call<R>` と `IDialogStack.Done<R>` を使用すると、アクティブ ダイアログのスタックを明示的に管理できます。一方、これらの fluent [Chain][chain] メソッドを使用すると、アクティブ ダイアログのスタックを暗黙的に管理することができます。


|           方法            |  Type   |                                 メモ                                  |
|-----------------------------|---------|------------------------------------------------------------------------|
|     Chain.Select<T, R>      |  LINQ   |           LINQ クエリ構文の "select" と "let" をサポートします。            |
|  Chain.SelectMany<T, C, R>  |  LINQ   |            LINQ クエリ構文の連続する "from" をサポートします。            |
|       Chain.Where<T>        |  LINQ   |                 LINQ クエリ構文の "where" をサポートします。                 |
|        Chain.From<T>        | Chains  |                ダイアログの新しいインスタンスをインスタンス化します。                |
|       Chain.Return<T>       | Chains  |                チェーンに定数値を返します。                |
|         Chain.Do<T>         | Chains  |               チェーン内の副作用を可能にします。                |
|  Chain.ContinueWith<T, R>   | Chains  |                      ダイアログの単純なチェーン。                       |
|       Chain.Unwrap<T>       | Chains  |                  ダイアログに入れ子になっているダイアログをラップ解除します。                   |
| Chain.DefaultIfException<T> | Chains  | 前の結果の例外を受け入れ、default(T) を返します。 |
|        Chain.Loop<T>        | [Branch]\(ブランチ\)  |                   ダイアログのチェーン全体をループします。                   |
|        Chain.Fold<T>        | [Branch]\(ブランチ\)  |   ダイアログの列挙の結果を 1 つの結果に折りたたみます。   |
|     Chain.Switch<T, R>      | [Branch]\(ブランチ\)  |            異なるダイアログ チェーンへの分岐をサポートします。            |
|     Chain.PostToUser<T>     | Message |                      ユーザーにメッセージを送信します。                      |
|     Chain.WaitToBot<T>      | Message |                    ボットへのメッセージを待ちます。                     |
|    Chain.PostToChain<T>     | Message |              ユーザーからのメッセージでチェーンを開始します。              |

### <a name="examples"></a>例 

この LINQ クエリ構文では、`Chain.Select<T, R>` メソッドを使用しています。

[!code-csharp[Chain.Select](../includes/code/dotnet-dialogs.cs#chain1)]

ここでは、`Chain.SelectMany<T, C, R>` メソッドを使用しています。

[!code-csharp[Chain.SelectMany](../includes/code/dotnet-dialogs.cs#chain2)]

`Chain.PostToUser<T>` および `Chain.WaitToBot<T>` メソッドを使用すると、ボットからユーザーにメッセージを送信できます。

[!code-csharp[Chain.PostToUser](../includes/code/dotnet-dialogs.cs#chain3)]

`Chain.Switch<T, R>` メソッドを使用して、会話ダイアログ フローを分岐します。

[!code-csharp[Chain.Switch](../includes/code/dotnet-dialogs.cs#chain4)]

`Chain.Switch<T, R>` が入れ子になった `IDialog<IDialog<T>>` を返した場合、`Chain.Unwrap<T>` を使用して内部の `IDialog<T>` をラップ解除できます。 これにより、会話をチェーンされたダイアログ (長さは異なる可能性がある) の異なるパスに分岐することができます。 fluent チェーン スタイルで記述された、暗黙的なスタック管理を使用してダイアログを分岐するより完全な例を次に示します。

[!code-csharp[Chain.Switch](../includes/code/dotnet-dialogs.cs#chain5)]

## <a name="next-steps"></a>次の手順

ダイアログは、ボットとユーザーの間の会話フローを管理します。 ダイアログは、ユーザーと対話する方法を定義します。 ボットは、スタックで構成された多くのダイアログを使用して、ユーザーとの会話を誘導することができます。 次のセクションでは、スタックのダイアログを作成および破棄するのに応じて拡大縮小するダイアログ スタックを見てみましょう。

> [!div class="nextstepaction"]
> [ダイアログを使用して会話フローを管理する](bot-builder-dotnet-manage-conversation-flow.md)


[builderLibrary]: /dotnet/api/microsoft.bot.builder.dialogs

[iBotData]: /dotnet/api/microsoft.bot.builder.dialogs.internals.ibotdata

[iBotToUser]: /dotnet/api/microsoft.bot.builder.dialogs.internals.ibottouser

[iDialogStack]: /dotnet/api/microsoft.bot.builder.dialogs.internals.idialogstack

[iBotDataBag]: /dotnet/api/microsoft.bot.builder.dialogs.ibotdatabag

[autofac]: /dotnet/api/microsoft.bot.builder.autofac.base

[chain]: /dotnet/api/microsoft.bot.builder.dialogs.chain
