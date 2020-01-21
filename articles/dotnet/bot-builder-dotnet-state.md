---
title: 状態データの管理 (v3 C#) - Bot Service
description: Bot Framework SDK for .NET を使用して状態データを保存および取得する方法について説明します。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 0856805c285f47f1f219e49d36daf4892af2dcc7
ms.sourcegitcommit: f8b5cc509a6351d3aae89bc146eaabead973de97
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/09/2020
ms.locfileid: "75797862"
---
# <a name="manage-state-data"></a>状態データの管理

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-state.md)
> - [Node.js](../nodejs/bot-builder-nodejs-state.md)

[!INCLUDE [State concept overview](../includes/snippet-dotnet-concept-state.md)]

## <a name="in-memory-data-storage"></a>インメモリ データ ストレージ

メモリ内データ ストレージはテスト専用です。 このストレージは揮発性であり、一時的なものです。 ボットの再起動ごとにデータがクリアされます。 テストのためにインメモリ ストレージを使用するには、以下を行う必要があります。 

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

この方法を使用して、独自のカスタム データ ストレージを設定できます。また、*Azure 拡張機能*のいずれかを使用することもできます。

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
| `SetConversationData` | 会話 | 指定されたチャネル上の会話の状態データを保存します。 <br/><br/>**注**:`SetConversationData` メソッドを使用して保存されたデータは `DeleteStateForUser` メソッドでは削除されないため、このメソッドを使用してユーザーの個人を特定できる情報 (PII) を保存しないでください。 |
| `SetPrivateConversationData` | ユーザーおよび会話 | 指定されたチャネル上の会話内のユーザーの状態データを保存します。 |
| `DeleteStateForUser` | User | `SetUserData` メソッドまたは `SetPrivateConversationData` メソッドを使用して以前に保存されたユーザーの状態データを削除します。 <br/><br/>**注**:ボットでは、ユーザーの連絡先リストからボットが削除されたことを示す [deleteUserData](bot-builder-dotnet-activities.md#deleteuserdata) 型のアクティビティまたは [contactRelationUpdate](bot-builder-dotnet-activities.md#contactrelationupdate) 型のアクティビティを受信したら、このメソッドを呼び出す必要があります。 |

ボットが "**Set...Data**" メソッドのいずれかを使用して状態データを保存すると、ボットが同じコンテキストで受信する今後のメッセージにそのデータが含まれるようになります。データには、対応する "**Get...Data**" メソッドを使用してアクセスできます。

## <a name="useful-properties-for-managing-state-data"></a>状態データの管理に役立つプロパティ

各 [Activity][Activity] オブジェクトには、状態データの管理に使用するプロパティが含まれています。

| プロパティ | [説明] | 使用事例 |
|----|----|----|
| `From` | チャネル上のユーザーを一意に識別する | ユーザーに関連する状態データの保存と取得 |
| `Conversation` | 会話を一意に識別する | 会話に関連する状態データの保存と取得 |
| `From` および `Conversation` | ユーザーと会話を一意に識別する | 特定の会話のコンテキスト内の特定のユーザーに関連する状態データの保存と取得 |

> [!NOTE]
> Bot Framework 状態データ ストアを使用するのではなく、独自のデータベースに状態データを保存する場合でも、これらのプロパティ値をキーとして使用できます。

## <a name="handle-concurrency-issues"></a>コンカレンシーの問題に対処する

ボットの 別のインスタンスがデータを変更していた場合、ボットが状態データを保存しようとすると、HTTP 状態コード **412 Precondition Failed** でエラー応答が返されることがあります。 次のコード例に示すように、このシナリオを考慮してボットを設計できます。

[!code-csharp[Handle exception saving state](../includes/code/dotnet-state.cs#handleException)]

## <a name="additional-resources"></a>その他のリソース

- [Bot Framework トラブルシューティング ガイド](../bot-service-troubleshoot-general-problems.md)
- <a href="/dotnet/api/?view=botbuilder-3.11.0" target="_blank">Bot Framework SDK for .NET リファレンス</a>

[Activity]: https://docs.botframework.com/csharp/builder/sdkreference/dc/d2f/class_microsoft_1_1_bot_1_1_connector_1_1_activity.html
