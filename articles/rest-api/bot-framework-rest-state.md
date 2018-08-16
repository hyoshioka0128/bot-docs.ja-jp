---
title: 状態データの管理 | Microsoft Docs
description: Bot State サービスを使用して、状態データを格納および取得する方法について説明します。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ms.openlocfilehash: 1557941d4e5413108ea3ce788f7d5d684252b657
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/18/2018
ms.locfileid: "39300916"
---
# <a name="manage-state-data"></a>状態データの管理

Bot State サービスを使用すると、ユーザー、会話、または特定の会話のコンテキスト内で特定のユーザーに関連付けられている状態データを、ご利用のボットで格納および取得できます。 最大 32 キロバイトのデータを、チャネル上のユーザーまたは会話ごとに格納できます。また、チャネル上の会話コンテキスト内のユーザーごとに格納することもできます。 状態データは、以前の会話が中断された場所を決定したり、単純にユーザーに名前であいさつを返したりするなど、さまざまな目的で使用できます。 ユーザーの設定を保存すると、次回チャットするときにその情報を使用して会話をカスタマイズできます。 たとえば、ユーザーが関心を持っているトピックに関するニュース記事についてユーザーに通知したり、予約が利用可能になった場合にユーザーに通知したりできます。 

> [!IMPORTANT]
> Bot Framework State Service API を運用環境で使用することはお勧めしません。将来のリリースで非推奨となる可能性があります。 テストが目的の場合はインメモリ ストレージを使用するようにボット コードを更新し、運用ボットの場合は **Azure 拡張機能**のいずれかを使用することをお勧めします。 詳細については、[.NET](~/dotnet/bot-builder-dotnet-state.md) または [Node](~/nodejs/bot-builder-nodejs-state.md) の実装に関する「**状態データの管理**」トピックを参照してください。

## <a id="concurrency"></a> データの同時実行

Bot State サービスを使用して格納されているデータの同時実行を制御するために、フレームワークでは `POST` 要求にエンティティ タグ (ETag) が使用されます。 フレームワークによって標準ヘッダーが ETag に対して使用されることはありません。 代わりに、要求と応答の本文で、`eTag` プロパティを使って ETag が指定されます。 

たとえば、`GET` 要求を発行してストアからユーザー データを取得する場合、応答には `eTag` プロパティが含まれます。 データを変更し、`POST` 要求を発行して、更新されたデータをストアに保存する場合、その要求には `eTag` プロパティを含めることができ、そのプロパティにより、`GET` 応答で前に受信した値と同じ値が指定されます。 `POST` 要求で指定されている ETag がストア内の現在の値と一致する場合、サーバーによってユーザーのデータが保存された後、"**HTTP 200 成功**" が返され、応答の本文で新しい `eTag` 値が指定されます。 `POST` 要求で指定されている ETag がストア内の現在の値と一致しない場合は、サーバーによって "**HTTP 412 必須条件に失敗**" が返されます。これは、ストア内のユーザーのデータが、前回そのデータを保存または取得した後に変更されたことを示します。

> [!TIP]
> `GET` 応答には必ず `eTag` プロパティが含まれますが、`GET` 要求で `eTag` プロパティを指定する必要はありません。 `eTag` プロパティ値がアスタリスク (`*`) である場合、指定したチャネル、ユーザー、および会話の組み合わせについて、前にデータを保存していないことを示します。
>
> `POST` 要求での `eTag` プロパティの指定は省略可能です。 
> ご自身のボットで同時実行が問題でない場合は、`POST` 要求で `eTag` プロパティを指定しないでください。 

## <a id="save-user-data"></a> ユーザー データを保存する

特定のチャネルにあるユーザーに関する状態データを保存するには、次の要求を発行します。

```http
POST /v3/botstate/{channelId}/users/{userId}
```

この要求 URI で、**{channelId}** をチャネルの ID に、**{userId}** をそのチャネル上のユーザーの ID に置き換えてください。 これらの ID は、お使いのボットで以前受信した、ユーザーからのすべてのメッセージ内にある `channelId` プロパティと `from` プロパティに追加されます。 メッセージからこれらの値を抽出しなくても、今後ユーザーのデータにアクセスできるように、その値を安全な場所にキャッシュすることもできます。

要求の本文を [BotData][BotData] オブジェクトに設定します。この場合、保存するデータは、`data` プロパティによって指定されます。 [同時実行コントロール](#concurrency)にエンティティ タグを使用する場合は、`eTag` プロパティを、前回ユーザーのデータを保存または取得したときに返された ETag (最新のもの) に設定します。 同時実行にエンティティ タグを使用しない場合は、要求で `eTag` プロパティを指定しないでください。

> [!NOTE]
> この `POST` 要求によってストア内にあるユーザー データが上書きされるのは、指定された ETag がサーバーの ETag と一致する場合、または要求で ETag が指定されていない場合のみに限られます。

### <a name="request"></a>Request 

次の例に示す要求では、特定のチャネルにあるユーザーに関するデータが保存されます。 この要求の例で、`https://smba.trafficmanager.net/apis` はベース URI を示しています。ご利用のボットによって発行される要求に対するベース URI は、これとは異なる場合があります。 ベース URI の設定の詳細については、[API リファレンス](bot-framework-rest-connector-api-reference.md#base-uri)に関する記事をご覧ください。

```http
POST https://smba.trafficmanager.net/apis/v3/botstate/abcd1234/users/12345678
Authorization: Bearer ACCESS_TOKEN
Content-Type: application/json
```

```json
{
    "data": [
        {
            "trail": "Lake Serene",
            "miles": 8.2,
            "difficulty": "Difficult",
        },
        {
            "trail": "Rainbow Falls",
            "miles": 6.3,
            "difficulty": "Moderate",
        }
    ],
    "eTag": "a1b2c3d4"
}
```

### <a name="response"></a>Response 

応答には、新しい `eTag` 値を持つ [BotData][BotData] オブジェクトが含まれます。

## <a name="get-user-data"></a>ユーザー データを取得する

特定のチャネルで以前保存されたユーザーに関する状態データを取得するには、次の要求を発行します。

```http
GET /v3/botstate/{channelId}/users/{userId}
```

この要求 URI で、**{channelId}** をチャネルの ID に、**{userId}** をそのチャネル上のユーザーの ID に置き換えてください。 これらの ID は、お使いのボットで以前受信した、ユーザーからのすべてのメッセージ内にある `channelId` プロパティと `from` プロパティに追加されます。 メッセージからこれらの値を抽出しなくても、今後ユーザーのデータにアクセスできるように、その値を安全な場所にキャッシュすることもできます。

### <a name="request"></a>Request

次の例に示す要求では、以前保存されたユーザーに関するデータが取得されます。 この要求の例で、`https://smba.trafficmanager.net/apis` はベース URI を示しています。ご利用のボットによって発行される要求に対するベース URI は、これとは異なる場合があります。 ベース URI の設定の詳細については、[API リファレンス](bot-framework-rest-connector-api-reference.md#base-uri)に関する記事をご覧ください。

```http
GET https://smba.trafficmanager.net/apis/v3/botstate/abcd1234/users/12345678
Authorization: Bearer ACCESS_TOKEN
Content-Type: application/json
```

### <a name="response"></a>Response

応答には [BotData][BotData] オブジェクトが含まれます。この場合、`data` プロパティには、以前保存したユーザーに関するデータが含まれています。また、`eTag` プロパティには、以降の要求でユーザーのデータを保存するときに使用できる ETag が含まれています。 ユーザーに関するデータを前に保存していない場合、`data` プロパティは `null` になり、`eTag` プロパティにはアスタリスク (`*`) が示されます。

次の例は、`GET` 要求に対する応答を示しています。

```json
{
    "data": [
        {
            "trail": "Lake Serene",
            "miles": 8.2,
            "difficulty": "Difficult",
        },
        {
            "trail": "Rainbow Falls",
            "miles": 6.3,
            "difficulty": "Moderate",
        }
    ],
    "eTag": "xyz12345"
}
```

## <a name="save-conversation-data"></a>会話データを保存する

特定のチャネルにある会話に関する状態データを保存するには、次の要求を発行します。

```http
POST /v3/botstate/{channelId}/conversations/{conversationId}
```

この要求 URI で、**{channelId}** をチャネルの ID に、**{conversationId}** を会話の ID に置き換えてください。 これらの ID は、会話のコンテキストでお使いのボットに対して以前送信された、またはそのボットで以前受信したすべてのメッセージ内にある、`channelId` プロパティと `conversation` プロパティに追加されます。 メッセージからこれらの値を抽出しなくても、今後会話のデータにアクセスできるように、その値を安全な場所にキャッシュすることもできます。

「[ユーザー データの保存](#save-user-data)」の説明に従って、要求本文を [BotData][BotData] オブジェクトに設定します。

> [!IMPORTANT]
> **会話データの保存**操作を使用して格納されたデータは、[ユーザー データの削除](#delete-user-data)操作を行っても削除されないため、ユーザーの個人を特定できる情報 (PII) を保存するときは、この操作は絶対に使用しないでください。

### <a name="response"></a>Response

応答には、新しい `eTag` 値を持つ [BotData][BotData] オブジェクトが含まれます。

## <a name="get-conversation-data"></a>会話データを取得する

特定のチャネルで以前保存された会話に関する状態データを取得するには、次の要求を発行します。

```http
GET /v3/botstate/{channelId}/conversations/{conversationId}
```

この要求 URI で、**{channelId}** をチャネルの ID に、**{conversationId}** を会話の ID に置き換えてください。 これらの ID は、会話のコンテキストでお使いのボットに対して以前送信された、またはそのボットで以前受信したすべてのメッセージ内にある、`channelId` プロパティと `conversation` プロパティに追加されます。 メッセージからこれらの値を抽出しなくても、今後会話のデータにアクセスできるように、その値を安全な場所にキャッシュすることもできます。

### <a name="response"></a>Response

応答には、新しい `eTag` 値を持つ [BotData][BotData] オブジェクトが含まれます。

## <a name="save-private-conversation-data"></a>個人的な会話データを保存する

特定の会話のコンテキスト内でユーザーに関する状態データを保存するには、次の要求を発行します。

```http
POST /v3/botstate/{channelId}/conversations/{conversationId}/users/{userId}
```

この要求 URI で、**{channelId}** をチャネルの ID に、**{conversationId}** を会話の ID に置き換え、さらに **{userId}** をそのチャネル上のユーザーの ID に置き換えてください。 これらの ID は、会話のコンテキストでボットに以前届いたそのユーザーからのすべてのメッセージ内にある、`channelId` プロパティ、`conversation` プロパティ、および`from` プロパティに追加されます。 メッセージからこれらの値を抽出しなくても、今後会話のデータにアクセスできるように、その値を安全な場所にキャッシュすることもできます。

「[ユーザー データの保存](#save-user-data)」の説明に従って、要求本文を [BotData][BotData] オブジェクトに設定します。

### <a name="response"></a>Response

応答には、新しい `eTag` 値を持つ [BotData][BotData] オブジェクトが含まれます。

## <a name="get-private-conversation-data"></a>個人的な会話データを取得する

特定の会話のコンテキストで以前保存されたユーザーに関する状態データを取得するには、次の要求を発行します。 

```http
GET /v3/botstate/{channelId}/conversations/{conversationId}/users/{userId}
```

この要求 URI で、**{channelId}** をチャネルの ID に、**{conversationId}** を会話の ID に置き換え、さらに **{userId}** をそのチャネル上のユーザーの ID に置き換えてください。 これらの ID は、会話のコンテキストでボットに以前届いたそのユーザーからのすべてのメッセージ内にある、`channelId` プロパティ、`conversation` プロパティ、および`from` プロパティに追加されます。 メッセージからこれらの値を抽出しなくても、今後会話のデータにアクセスできるように、その値を安全な場所にキャッシュすることもできます。

### <a name="response"></a>Response

応答には、新しい `eTag` 値を持つ [BotData][BotData] オブジェクトが含まれます。

## <a name="delete-user-data"></a>ユーザー データを削除する

特定のチャネルにあるユーザーに関する状態データを削除するには、次の要求を発行します。

```http
DELETE /v3/botstate/{channelId}/users/{userId}
```
この要求 URI で、**{channelId}** をチャネルの ID に、**{userId}** をそのチャネル上のユーザーの ID に置き換えてください。 これらの ID は、お使いのボットで以前受信した、ユーザーからのすべてのメッセージ内にある `channelId` プロパティと `from` プロパティに追加されます。 メッセージからこれらの値を抽出しなくても、今後ユーザーのデータにアクセスできるように、その値を安全な場所にキャッシュすることもできます。

> [!IMPORTANT]
> この操作を実行すると、[ユーザー データの保存](#save-user-data)操作または[個人的な会話データの保存](#save-private-conversation-data)操作のいずれかを使用して以前保存された、ユーザーに関するデータが削除されます。 [会話データの保存](#save-conversation-data)操作を使用して以前保存されたデータは削除されません。 したがって、ユーザーの個人を特定できる情報 (PII) を保存するときは、**会話データの保存**操作は絶対に使用しないでください。

ユーザーの連絡先リストからお使いのボットが削除されたことを示す [deleteUserData](bot-framework-rest-connector-activities.md#deleteuserdata) 型の[Activity][Activity] または [contactRelationUpdate](bot-framework-rest-connector-activities.md#contactrelationupdate) 型のアクティビティを受信したら、そのボットで**ユーザー データの削除**操作を実行する必要があります。

## <a name="additional-resources"></a>その他のリソース

- [主要な概念](bot-framework-rest-connector-concepts.md)
- [アクティビティの概要](bot-framework-rest-connector-activities.md)

[BotData]: bot-framework-rest-connector-api-reference.md#botdata-object
[Activity]: bot-framework-rest-connector-api-reference.md#activity-object
