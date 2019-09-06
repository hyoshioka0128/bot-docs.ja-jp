---
title: API リファレンス - Direct Line API 3.0 | Microsoft Docs
description: Direct Line API 3.0 のヘッダー、HTTP 状態コード、スキーマ、操作、およびオブジェクトについて説明します。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
ms.openlocfilehash: 618b2ffe99114679aa5592b816adf6e1b82be83e
ms.sourcegitcommit: eacf1522d648338eebefe2cc5686c1f7866ec6a2
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 08/30/2019
ms.locfileid: "70167181"
---
# <a name="api-reference---direct-line-api-30"></a>API リファレンス - Direct Line API 3.0

Direct Line API 3.0 を使用することで、クライアント アプリケーションとボットの通信を有効にできます。 Direct Line API 3.0 では、HTTPS で業界標準の REST および JSON を使用します。

## <a name="base-uri"></a>ベース URI

Direct Line API 3.0 にアクセスするには、すべての API 要求用の次の基本 URI を使用します。

`https://directline.botframework.com`

## <a name="headers"></a>headers

標準 HTTP 要求ヘッダーに加えて、Direct Line API 要求には、要求を発行しているクライアントを認証するシークレットまたはトークンを指定する `Authorization` ヘッダーを含める必要があります。 `Authorization` ヘッダーは次の形式を使用して指定します。

```http
Authorization: Bearer SECRET_OR_TOKEN
```

クライアントが Direct Line API 要求を認証するために使用できるシークレットまたはトークンを取得する方法の詳細については、「[Authentication](bot-framework-rest-direct-line-3-0-authentication.md)」(認証) を参照してください。

## <a name="http-status-codes"></a>HTTP 状態コード

各応答で返される <a href="http://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html" target="_blank"> HTTP 状態コード</a> は、対応する要求の結果を示しています。

| HTTP 状態コード | 意味 |
|----|----|
| 200 | 要求は成功しました。 |
| 201 | 要求は成功しました。 |
| 202 | 要求の処理は受理されました。 |
| 204 | 要求は成功しましたが、返されたコンテンツはありませんでした。 |
| 400 | 要求の形式その他が正しくありませんでした。 |
| 401 | クライアントは要求を行うことを許可されていません。 多くの場合、この状態コードの原因は、`Authorization` ヘッダーが見つからないか、形式が正しくないことです。 |
| 403 | 要求された操作の実行がクライアントに許可されていません。 以前は有効であったが期限切れとなったトークンが要求で指定されている場合は、[ErrorResponse][] オブジェクト内で返された [Error][] の `code` プロパティが `TokenExpired` に設定されます。 |
| 404 | 要求されたリソースが見つかりませんでした。 通常、この状態コードは、要求 URI が無効であることを示します。 |
| 500 | Direct Line サービス内で内部サーバー エラーが発生しました。 |
| 502 | ボットが利用できないか、ボットからエラーが返されました。 **これは一般的なエラー コードです。** |

> [!NOTE]
> HTTP 状態コード 101 は WebSocket 接続パス内で使用されます (ただし、多くの場合は WebSocket クライアントによって処理されます)。

### <a name="errors"></a>Errors

4xx 範囲または 5xx 範囲の HTTP 状態コードを指定する応答では、エラーの情報を提供する [ErrorResponse][] オブジェクトが応答の本文に含まれます。 4xx 範囲のエラー応答を受け取った場合、要求を再送信する前に、**ErrorResponse** オブジェクトを検査してエラーの原因を識別し、問題を解決してください。

> [!NOTE]
> **ErrorResponse** オブジェクト内の `code` プロパティで指定された HTTP 状態コードと値は変わりません。 **ErrorResponse** オブジェクト内の `message` プロパティで指定された値は、時間の経過と共に変更される可能性があります。

次のスニペットは、要求の例と結果のエラー応答を示したものです。

#### <a name="request"></a>Request

```http
POST https://directline.botframework.com/v3/directline/conversations/abc123/activities
[detail omitted]
```

#### <a name="response"></a>Response

```http
HTTP/1.1 502 Bad Gateway
[other headers]
```

```json
{
    "error": {
        "code": "BotRejectedActivity",
        "message": "Failed to send activity: bot returned an error"
    }
}
```

## <a name="token-operations"></a>トークン操作

クライアントが 1 つの会話にアクセスするために使用できるトークンを作成または更新するには、次の操作を使用します。

| Operation | 説明 |
|----|----|
| [トークンの生成](#generate-token) | 新しい会話用のトークンを生成します。 |
| [トークンの更新](#refresh-token) | トークンを更新します。 |

### <a name="generate-token"></a>トークンの生成

1 つの会話で有効なトークンを生成します。

```http
POST /v3/directline/tokens/generate
```

| | |
|----|----|
| **要求本文** | [TokenParameters](#tokenparameters-object) オブジェクト |
| **戻り値** | [Conversation](#conversation-object) オブジェクト |

### <a name="refresh-token"></a>トークンの更新

トークンを更新します。

```http
POST /v3/directline/tokens/refresh
```

| | |
|----|----|
| **要求本文** | 該当なし |
| **戻り値** | [Conversation](#conversation-object) オブジェクト |

## <a name="conversation-operations"></a>会話操作

ボットとの会話を開いてクライアントとボット間でアクティビティを交換するには、次の操作を使用します。

| Operation | 説明 |
|----|----|
| [会話の開始](#start-conversation) | ボットと新しい会話を開きます。 |
| [会話情報の取得](#get-conversation-information) | 既存の会話に関する情報を取得します。 この操作を実行すると、クライアントが会話に[再接続](bot-framework-rest-direct-line-3-0-reconnect-to-conversation.md)するために使用できる、新しい WebSocket ストリーム URL が生成されます。 |
| [アクティビティの取得](#get-activities) | ボットからアクティビティを取得します。 |
| [アクティビティの送信](#send-an-activity) | ボットにアクティビティを送信します。 |
| [ファイルのアップロードと送信](#upload-and-send-files) | ファイルを添付ファイルとしてアップロードして送信します。 |

### <a name="start-conversation"></a>会話を開始する

ボットと新しい会話を開きます。

```http
POST /v3/directline/conversations
```

| | |
|----|----|
| **要求本文** | [TokenParameters](#tokenparameters-object) オブジェクト |
| **戻り値** | [Conversation](#conversation-object) オブジェクト |

### <a name="get-conversation-information"></a>会話情報の取得

既存の会話に関する情報を取得し、クライアントが会話に[再接続](bot-framework-rest-direct-line-3-0-reconnect-to-conversation.md)するために使用できる、新しい WebSocket ストリーム URL を生成します。 クライアントによって認識された最新のメッセージを示す必要がある場合は、要求 URI 内に `watermark` パラメーターを指定することもできます。

```http
GET /v3/directline/conversations/{conversationId}?watermark={watermark_value}
```

| | |
|----|----|
| **要求本文** | 該当なし |
| **戻り値** | [Conversation](#conversation-object) オブジェクト |

### <a name="get-activities"></a>アクティビティの取得

指定された会話用のボットからアクティビティを取得します。 クライアントによって認識された最新のメッセージを示す必要がある場合は、要求 URI 内に `watermark` パラメーターを指定することもできます。

```http
GET /v3/directline/conversations/{conversationId}/activities?watermark={watermark_value}
```

| | |
|----|----|
| **要求本文** | 該当なし |
| **戻り値** | [ActivitySet](#activityset-object) オブジェクト。 応答には、`ActivitySet` オブジェクトのプロパティとして `watermark` が含まれます。 クライアントは、アクティビティが返らなくなるまで `watermark` 値を進めることで、すべてのアクティビティを取得する必要があります。 |

### <a name="send-an-activity"></a>アクティビティの送信

ボットにアクティビティを送信します。

```http
POST /v3/directline/conversations/{conversationId}/activities
```

| | |
|----|----|
| **要求本文** | [Activity][] オブジェクト |
| **戻り値** | ボットに送信されたアクティビティの ID を指定する `id` プロパティを格納した [ResourceResponse][]。 |

### <a name="upload-and-send-files"></a>ファイルのアップロードと送信

ファイルを添付ファイルとしてアップロードして送信します。 添付ファイルを送信しているユーザーの ID を指定するには、要求 URI 内に `userId` パラメーターを設定します。

```http
POST /v3/directline/conversations/{conversationId}/upload?userId={userId}
```

| | |
|----|----|
| **要求本文** | 1 つの添付ファイルの場合は、要求本文にファイルのコンテンツを設定します。 複数の添付ファイルの場合は、各パートが 1 つの添付ファイルを指定するマルチパートを含む要求本文を作成します。(必要に応じて) 指定された添付ファイルのコンテナーとして機能する [Activity][] オブジェクト用の 1 つのパートを記述することもできます。 詳しくは、「[ボットにアクティビティを送信する](bot-framework-rest-direct-line-3-0-send-activity.md)」をご覧ください。 |
| **戻り値** | ボットに送信されたアクティビティの ID を指定する `id` プロパティを格納した [ResourceResponse][]。 |

> [!NOTE]
> アップロードされたファイルは、24 時間後に削除されます。

## <a name="schema"></a>スキーマ

Direct Line 3.0 スキーマには、[Bot Framework スキーマ](bot-framework-rest-connector-api-reference.md#schema)によって定義されるすべてのオブジェクトに加え、Direct Line に固有のオブジェクトがいくつか含まれています。

### <a name="activityset-object"></a>ActivitySet オブジェクト

アクティビティのセットを定義します。

| プロパティ | Type | 説明 |
|----|----|----|
| **アクティビティ** | [Activity][][] | **Activity** オブジェクトの配列です。 |
| **watermark** | string | セット内のアクティビティの最大ウォーターマークです。 クライアントは、[ボットからアクティビティを取得する](bot-framework-rest-direct-line-3-0-receive-activities.md#http-get)際、または[新しい WebSocket ストリーム URL を生成する](bot-framework-rest-direct-line-3-0-reconnect-to-conversation.md)際に、`watermark` 値を使用して、直近に認識したメッセージを示すことができます。 |

### <a name="conversation-object"></a>Conversation オブジェクト

Direct Line 会話を定義します。

| プロパティ | Type | 説明 |
|----|----|----|
| **conversationId** | string | 指定されたトークンが有効な会話を一意に識別する ID。 |
| **eTag** | string | HTTP ETag (エンティティ タグ)。 |
| **expires_in** | number | トークンの有効期限が切れるまでの秒数。 |
| **referenceGrammarId** | string | このボットの参照文法の ID。 |
| **streamUrl** | string | 会話のメッセージ ストリームの URL。 |
| **token** | string | 指定された会話で有効なトークン。 |

### <a name="tokenparameters-object"></a>TokenParameters オブジェクト

トークンを作成するためのパラメーター。

| プロパティ | Type | 説明 |
|----|----|----|
| **eTag** | string | HTTP ETag (エンティティ タグ)。 |
| **trustedOrigins** | string[] | トークン内に埋め込む信頼された発行元。 |
| **user** | [ChannelAccount][] | トークン内に埋め込むユーザー アカウント。 |

## <a name="activities"></a>Activities

クライアントが Direct Line を通じてボットから受信した各[Activity][]には、次の操作が適用されます。

- 添付ファイル カードは保持されます。
- アップロードされた添付ファイルの URL は、プライベート リンクを使用して非表示にされます。
- `channelData` プロパティは変更せずに保持されます。

クライアントは、[ActivitySet](#activityset-object) の一部として、ボットから複数のアクティビティを[受信](bot-framework-rest-direct-line-3-0-receive-activities.md)することがあります。

クライアントから Direct Line を通じてボットに `Activity` を送信するときには、次のことに注意してください。

- `type` プロパティでは、送信するアクティビティの種類を指定します (通常は**メッセージ**)。
- `from` プロパティには、クライアントによって選択されたユーザー ID を指定する必要があります。
- 添付ファイルには、既存のリソースの URL や、Direct Line 添付ファイル エンドポイントを通じてアップロードされた URL を含めることができます。
- `channelData` プロパティは変更せずに保持されます。
- JSON にシリアル化されており暗号化されている場合、アクティビティの合計サイズは 256,000 文字以内にする必要があります。 そのため、アクティビティを 150,000 未満に抑えることをお勧めします。 さらにデータが必要な場合、アクティビティを複数に分割するか、添付ファイルを使用することを検討してください。

クライアントでは、要求ごとにアクティビティを 1 つ[送信](bot-framework-rest-direct-line-3-0-send-activity.md)できます。

## <a name="additional-resources"></a>その他のリソース

- [Bot Framework Activity の仕様](https://aka.ms/botSpecs-activitySchema)

[Activity]: bot-framework-rest-connector-api-reference.md#activity-object
[ChannelAccount]: bot-framework-rest-connector-api-reference.md#channelaccount-object
[Error]: bot-framework-rest-connector-api-reference.md#error-object
[ErrorResponse]: bot-framework-rest-connector-api-reference.md#errorresponse-object
[ResourceResponse]: bot-framework-rest-connector-api-reference.md#resourceresponse-object
