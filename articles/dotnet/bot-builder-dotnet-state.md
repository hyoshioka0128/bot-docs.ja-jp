---
title: 状態データの管理 | Microsoft Docs
description: Bot Builder SDK for .NET を使用して状態データを保存および取得する方法について説明します。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/17
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 3a924503ecadc9f56fa2543881c116f7fbbb4d9a
ms.sourcegitcommit: 2dc75701b169d822c9499e393439161bc87639d2
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 08/24/2018
ms.locfileid: "42904330"
---
# <a name="manage-state-data"></a>状態データの管理

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-state.md)
> - [Node.js](../nodejs/bot-builder-nodejs-state.md)

[!INCLUDE [State concept overview](../includes/snippet-dotnet-concept-state.md)]

## <a name="in-memory-data-storage"></a>インメモリ データ ストレージ

メモリ内データ ストレージはテスト専用です。 このストレージは揮発性であり、一時的なものです。 ボットが再起動されるたびにデータが消去されます。 テストのためにインメモリ ストレージを使用するには、以下を行う必要があります。 

次の NuGet パッケージをインストールします。 
- Microsoft.Bot.Builder.Azure
- Autofac.WebApi2

**Application_Start** メソッドで、インメモリ ストレージの新しいインスタンスを作成し、新しいデータ ストアを登録します。

```cs
// Global.asax file

var store = new InMemoryDataStore();

Conversation.UpdateContainer(
           builder =>
           {
               builder.Register(c => store)
                         .Keyed<IBotDataStore<BotData>>(AzureModule.Key_DataStore)
                         .AsSelf()
                         .SingleInstance();

               builder.Register(c => new CachingBotDataStore(store,
                          CachingBotDataStoreConsistencyPolicy
                          .ETagBasedConsistency))
                          .As<IBotDataStore<BotData>>()
                          .AsSelf()
                          .InstancePerLifetimeScope();


           });
GlobalConfiguration.Configure(WebApiConfig.Register);

```

このメソッドを使用して独自のカスタム データ ストレージを設定することも、*Azure 拡張機能*のいずれかを使用することもできます。

## <a name="manage-custom-data-storage"></a>カスタム データ ストレージを管理する

運用環境でのパフォーマンスとセキュリティ上の理由から、独自のデータ ストレージを実装するか、次のデータ ストレージ オプションのいずれかを実装することを検討できます。

1. [Cosmos DB を使用して状態データを管理する](bot-builder-dotnet-state-azure-cosmosdb.md)

2. [Table Storage を使用して状態データを管理する](bot-builder-dotnet-state-azure-table-storage.md)

どちらの [Azure 拡張機能](https://www.nuget.org/packages/Microsoft.Bot.Builder.Azure/)オプションでも、Bot Framework SDK for .NET を使用したデータの設定と保持のメカニズムはインメモリ データ ストレージであることに変わりはありません。

## <a name="bot-state-methods"></a>ボットの状態メソッド

次の表に、状態データの管理に使用できるメソッドを示します。

| 方法 | スコープ | 目的 |                                                
|----|----|----|
| `GetUserData` | User | 指定されたチャネル上のユーザーの、以前に保存された状態データを取得します。 |
| `GetConversationData` | 会話 | 指定されたチャネル上の会話の、以前に保存された状態データを取得します。 |
| `GetPrivateConversationData` | ユーザーおよび会話 | 指定されたチャネル上の会話内のユーザーの、以前に保存された状態データを取得します。 |
| `SetUserData` | User | 指定されたチャネル上のユーザーの状態データを保存します。 |
| `SetConversationData` | 会話 | 指定されたチャネル上の会話の状態データを保存します。 <br/><br/>**注**: `SetConversationData` メソッドを使用して保存されたデータは `DeleteStateForUser` メソッドでは削除されないため、このメソッドを使用してユーザーの個人を特定できる情報 (PII) を保存しないでください。 |
| `SetPrivateConversationData` | ユーザーおよび会話 | 指定されたチャネル上の会話内のユーザーの状態データを保存します。 |
| `DeleteStateForUser` | User | `SetUserData` メソッドまたは `SetPrivateConversationData` メソッドを使用して以前に保存されたユーザーの状態データを削除します。 <br/><br/>**注**: ボットでは、ユーザーの連絡先リストからボットが削除されたことを示す [deleteUserData](bot-builder-dotnet-activities.md#deleteuserdata) 型のアクティビティまたは [contactRelationUpdate](bot-builder-dotnet-activities.md#contactrelationupdate) 型のアクティビティを受信したら、このメソッドを呼び出す必要があります。 |

ボットが "**Set...Data**" メソッドのいずれかを使用して状態データを保存すると、ボットが同じコンテキストで受信する今後のメッセージにそのデータが含まれるようになります。データには、対応する "**Get...Data**" メソッドを使用してアクセスできます。

## <a name="useful-properties-for-managing-state-data"></a>状態データの管理に役立つプロパティ

各 [Activity][Activity] オブジェクトには、状態データの管理に使用するプロパティが含まれています。

| プロパティ | 説明 | ユース ケース |
|----|----|----|
| `From` | チャネル上のユーザーを一意に識別する | ユーザーに関連する状態データの保存と取得 |
| `Conversation` | 会話を一意に識別する | 会話に関連する状態データの保存と取得 |
| `From` と `Conversation` | ユーザーと会話を一意に識別する | 特定の会話のコンテキスト内の特定のユーザーに関連する状態データの保存と取得 |

> [!NOTE]
> Bot Framework 状態データ ストアを使用するのではなく、独自のデータベースに状態データを保存する場合でも、これらのプロパティ値をキーとして使用できます。

## <a id="state-client"></a>状態クライアントを作成する

`StateClient` オブジェクトを使用すると、Bot Builder SDK for .NET を使用して状態データを管理できます。 状態データを管理する同じコンテキストに属しているメッセージにアクセスできる場合は、`Activity` オブジェクトに対して `GetStateClient` メソッドを呼び出すことで状態クライアントを作成できます。

[!code-csharp[Get State client](../includes/code/dotnet-state.cs#getStateClient1)]

状態データを管理する同じコンテキストに属しているメッセージにアクセスできない場合は、`StateClient` クラスの新しいインスタンスを作成するだけで、状態クライアントを作成できます。 次の例では、`microsoftAppId` と `microsoftAppPassword` は、[ボット作成](../bot-service-quickstart.md)プロセス中に取得したボットの Bot Framework 認証資格情報です。

> [!NOTE]
> ボットの **AppID** と **AppPassword** を確認する方法については、「[MicrosoftAppID and MicrosoftAppPassword (MicrosoftAppID と MicrosoftAppPassword)](~/bot-service-manage-overview.md#microsoftappid-and-microsoftapppassword)」をご覧ください。

[!code-csharp[Get State client](../includes/code/dotnet-state.cs#getStateClient2)]

> [!NOTE]
> 既定の状態クライアントは、中央のサービスに保存されます。 一部のチャネルでは、そのチャネル内でホストされている状態 API を使用できるので、チャネルが提供する準拠ストアに状態データを保存できます。

## <a name="get-state-data"></a>状態データを取得する

各 "**Get...Data**" メソッドは、指定されたユーザー/会話の状態データを含む `BotData` オブジェクトを返します。 `BotData` オブジェクトから特定のプロパティ値を取得するには、`GetProperty` メソッドを呼び出します。 

次のコード例は、ユーザー データから型指定されたプロパティを取得する方法を示しています。 

[!code-csharp[Get state property](../includes/code/dotnet-state.cs#getProperty1)]

次のコード例は、ユーザー データ内の複合型からプロパティを取得する方法を示しています。

[!code-csharp[Get state property](../includes/code/dotnet-state.cs#getProperty2)]

"**Get...Data**" メソッド呼び出しで指定されたユーザー/会話の状態データが存在しない場合、返される `BotData` オブジェクトには次のプロパティ値が含まれます。 
- `BotData.Data` = null
- `BotData.ETag` = "*"

## <a name="save-state-data"></a>状態データを保存する

状態データを保存するには、まず、適切な "**Get...Data**" メソッドを呼び出して `BotData` オブジェクトを取得します。次に、更新するプロパティごとに `SetProperty` メソッドを呼び出して更新し、適切な "**Set...Data**" メソッドを呼び出してオブジェクトを保存します。 

> [!NOTE]
> チャネル上のユーザーごと、チャネル上の会話ごと、チャネル上の会話のコンテキスト内のユーザーごとに、最大 32 キロバイトのデータを保存できます。 

次のコード例は、型指定されたプロパティをユーザー データに保存する方法を示しています。

[!code-csharp[Set state property](../includes/code/dotnet-state.cs#setProperty1)]

次のコード例は、プロパティをユーザー データ内の複合型に保存する方法を示しています。 

[!code-csharp[Set state property](../includes/code/dotnet-state.cs#setProperty2)]

## <a name="handle-concurrency-issues"></a>同時実行の問題に対処する

ボットの 別のインスタンスがデータを変更していた場合、ボットが状態データを保存しようとすると、HTTP 状態コード **412 Precondition Failed** でエラー応答が返されることがあります。 次のコード例に示すように、このシナリオを考慮してボットを設計できます。

[!code-csharp[Handle exception saving state](../includes/code/dotnet-state.cs#handleException)]

## <a name="additional-resources"></a>その他のリソース

- [Bot Framework トラブルシューティング ガイド](../bot-service-troubleshoot-general-problems.md)
- <a href="/dotnet/api/?view=botbuilder-3.11.0" target="_blank">Bot Builder SDK for .NET リファレンス</a>

[Activity]: https://docs.botframework.com/en-us/csharp/builder/sdkreference/dc/d2f/class_microsoft_1_1_bot_1_1_connector_1_1_activity.html
