---
title: API リファレンス - Direct Line API 1.1 | Microsoft Docs
description: Direct Line API 1.1 のヘッダー、HTTP 状態コード、スキーマ、操作、およびオブジェクトについて説明します。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
ms.openlocfilehash: 3607957cd5cb8738e8268ece6eba4417250bc596
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/24/2018
ms.locfileid: "49997961"
---
# <a name="api-reference---direct-line-api-11"></a>API リファレンス - Direct Line API 1.1

> [!IMPORTANT]
> この記事には、Direct Line API 1.1 に関するリファレンス情報が含まれています。 クライアント アプリケーションとボット間の新しい接続を作成する場合は、[Direct Line API 3.0](bot-framework-rest-direct-line-3-0-api-reference.md) を使用してください。

Direct Line API 1.1 を使用することで、クライアント アプリケーションとボットの通信を有効にできます。 Direct Line API 1.1 では、HTTPS で業界標準の REST および JSON を使用します。

## <a name="base-uri"></a>ベース URI

Direct Line API 1.1 にアクセスするには、すべての API 要求用の次の基本 URI を使用します。

`https://directline.botframework.com`

## <a name="headers"></a>headers

標準 HTTP 要求ヘッダーに加えて、Direct Line API 要求には、要求を発行しているクライアントを認証するシークレットまたはトークンを指定する `Authorization` ヘッダーを含める必要があります。 `Authorization` ヘッダーは、"Bearer" スキームまたは "BotConnector" スキームを使用して指定できます。 

**Bearer スキーム**:
```http
Authorization: Bearer SECRET_OR_TOKEN
```

**BotConnector スキーム**:
```http
Authorization: BotConnector SECRET_OR_TOKEN
```

クライアントが Direct Line API 要求を認証するために使用できるシークレットまたはトークンを取得する方法の詳細については、「[Authentication](bot-framework-rest-direct-line-1-1-authentication.md)」(認証) を参照してください。

## <a name="http-status-codes"></a>HTTP 状態コード

各応答で返される <a href="http://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html" target="_blank"> HTTP 状態コード</a> は、対応する要求の結果を示しています。 

| HTTP 状態コード | 意味 |
|----|----|
| 200 | 要求は成功しました。 |
| 204 | 要求は成功しましたが、返されたコンテンツはありませんでした。 |
| 400 | 要求の形式その他が正しくありませんでした。 |
| 401 | クライアントは要求を行うことを許可されていません。 多くの場合、この状態コードの原因は、`Authorization` ヘッダーが見つからないか、形式が正しくないことです。 |
| 403 | 要求された操作の実行がクライアントに許可されていません。 多くの場合、この状態コードの原因は、`Authorization` ヘッダーに無効なトークンまたはシークレットが指定されていることです。 |
| 404 | 要求されたリソースが見つかりませんでした。 通常、この状態コードは、要求 URI が無効であることを示します。 |
| 500 | Direct Line サービス内で内部サーバー エラーが発生しました |
| 502 | ボット内でエラーが発生しました。ボットが利用できないか、ボットからエラーが返されました。  **これは一般的なエラー コードです。** |

## <a name="token-operations"></a>トークンの操作 
クライアントが 1 つの会話にアクセスするために使用できるトークンを作成または更新するには、次の操作を使用します。

| Operation | 説明 |
|----|----|
| [トークンの生成](#generate-token) | 新しい会話用のトークンを生成します。 | 
| [トークンの更新](#refresh-token) | トークンを更新します。 | 

### <a name="generate-token"></a>トークンの生成
1 つの会話で有効なトークンを生成します。 
```http 
POST /api/tokens/conversation
```

| | |
|----|----|
| **要求本文** | 該当なし |
| **戻り値** | トークンを表す文字列 | 

### <a name="refresh-token"></a>トークンの更新
トークンを更新します。 
```http 
GET /api/tokens/{conversationId}/renew
```

| | |
|----|----|
| **要求本文** | 該当なし |
| **戻り値** | 新しいトークンを表す文字列 | 

## <a name="conversation-operations"></a>会話操作 
ボットとの会話を開いてクライアントとボット間でメッセージを交換するには、次の操作を使用します。

| Operation | 説明 |
|----|----|
| [会話の開始](#start-conversation) | ボットと新しい会話を開きます。 | 
| [Get Messages](#get-messages) | ボットからメッセージを受信します。 |
| [メッセージの送信](#send-a-message) | メッセージをボットに送信します。 | 
| [ファイルのアップロードと送信](#upload-send-files) | ファイルを添付ファイルとしてアップロードして送信します。 |

### <a name="start-conversation"></a>会話を開始する
ボットと新しい会話を開きます。 
```http 
POST /api/conversations
```

| | |
|----|----|
| **要求本文** | 該当なし |
| **戻り値** | [Conversation](#conversation-object) オブジェクト | 

### <a name="get-messages"></a>メッセージの取得
指定された会話用のボットからメッセージを取得します。 クライアントによって認識された最新のメッセージを示すには、要求 URI 内に `watermark` パラメーターを設定します。 

```http
GET /api/conversations/{conversationId}/messages?watermark={watermark_value}
```

| | |
|----|----|
| **要求本文** | 該当なし |
| **戻り値** | [MessageSet](#messageset-object) オブジェクト。 応答には、`MessageSet` オブジェクトのプロパティとして `watermark` が含まれます。 クライアントは、メッセージが返らなくなるまで `watermark` 値を進めることで、入手できるメッセージをすべて取得する必要があります。 | 

### <a name="send-a-message"></a>メッセージの送信
メッセージをボットに送信します。 
```http 
POST /api/conversations/{conversationId}/messages
```

| | |
|----|----|
| **要求本文** | [Message](#message-object) オブジェクト |
| **戻り値** | 応答の本文で返されるデータはありません。 サービスは、メッセージが正常に送信された場合は、HTTP 204 状態コードで応答します。 クライアントは、[Get Messages](#get-messages) 操作を使用して、送信済みのメッセージを (ボットがクライアントに送信しているメッセージと共に) 取得できます。 |

### <a id="upload-send-files"></a> ファイルのアップロードと送信
ファイルを添付ファイルとしてアップロードして送信します。 添付ファイルを送信しているユーザーの ID を指定するには、要求 URI 内に `userId` パラメーターを設定します。
```http 
POST /api/conversations/{conversationId}/upload?userId={userId}
```

| | |
|----|----|
| **要求本文** | 1 つの添付ファイルの場合は、要求本文にファイルのコンテンツを設定します。 複数の添付ファイルの場合は、各パートが 1 つの添付ファイルを指定するマルチパートを含む要求本文を作成します。(必要に応じて) 指定された添付ファイルのコンテナーとして機能する [Message](#message-object) オブジェクト用の 1 つのパートを記述することもできます。 詳細については、「[ボットにアクティビティを送信する](bot-framework-rest-direct-line-1-1-send-message.md)」を参照してください。 |
| **戻り値** | 応答の本文で返されるデータはありません。 サービスは、メッセージが正常に送信された場合は、HTTP 204 状態コードで応答します。 クライアントは、[Get Messages](#get-messages) 操作を使用して、送信済みのメッセージを (ボットがクライアントに送信しているメッセージと共に) 取得できます。 | 

> [!NOTE]
> アップロードされたファイルは、24 時間後に削除されます。

## <a name="schema"></a>スキーマ

Direct Line 1.1 スキーマは、次のオブジェクトを含む Bot Framework v1 スキーマの簡略化されたコピーです。

### <a name="message-object"></a>Message オブジェクト

クライアントがボットに送信するメッセージ、またはボットから受信するメッセージを定義します。

| プロパティ | type | 説明 |
|----|----|----|
| **id** | string | メッセージを一意に識別する ID (Direct Line によって割り当てられます)。 | 
| **conversationId** | string | 会話を識別する ID。  | 
| **created** | string | メッセージの作成日時。<a href="https://en.wikipedia.org/wiki/ISO_8601" target="_blank">ISO-8601</a> で表わされます。 | 
| **from** | string | メッセージの送信者であるユーザーを識別する ID。 クライアントは、メッセージを作成するときに、このプロパティを安定したユーザー ID に設定する必要があります。 Direct Line はユーザー ID が指定されない場合はそれを割り当てますが、通常は、これによって予期しない動作が発生します。 | 
| **text** | string | ユーザーからボットに、またはボットからユーザーに送信されるメッセージのテキスト。 | 
| **channelData** | オブジェクト | チャネル固有のコンテンツを格納するオブジェクト。 一部のチャネルには、添付ファイル スキーマでは表現できない追加情報を必要とする機能があります。 そのような場合は、このプロパティを、チャネルのドキュメントで定義されているチャネル固有のコンテンツに設定します。 このデータは、クライアントとボット間で変更されずに送信されます。 このプロパティは、複雑なオブジェクトが設定されるか、空のままにしておく必要があります。 文字列、数値、またはその他の単純型を設定しないでください。 | 
| **images** | string[] | メッセージに含まれているイメージの URL を格納する文字列の配列。 この配列内の文字列は、相対 URL である場合があります。 この配列内の任意の文字列が "http" または "https" で始まっていない場合は、文字列の前に `https://directline.botframework.com` を追加して、完全な URL 形式にします。 | 
| **attachments** | [Attachment](#attachment-object)[] | メッセージに含まれるイメージ以外の添付ファイルを表す **Attachment** オブジェクトの配列。 配列内の各オブジェクトには、`url` プロパティと `contentType` プロパティが格納されます。 クライアントがボットから受信するメッセージでは、`url` プロパティは、相対 URL を指定している場合があります。 `url` プロパティの値が "http" または "https" で始まっていない場合は、文字列の前に `https://directline.botframework.com` を追加して、完全な URL 形式にします。 | 

次の例は、すべての可能なプロパティを含む Message オブジェクトを示しています。 ほとんどの場合、メッセージを作成するときに、クライアントが指定する必要があるのは、`from` プロパティと少なくとも 1 つのコンテンツ プロパティ (例: `text`、`images`、`attachments`、または`channelData`) だけです。

```json
{
    "id": "CuvLPID4kDb|000000000000000004",
    "conversationId": "CuvLPID4kDb",
    "created": "2016-10-28T21:19:51.0357965Z",
    "from": "examplebot",
    "text": "Hello!",
    "channelData": {
        "examplefield": "abc123"
    },
    "images": [
        "/attachments/CuvLPID4kDb/0.jpg?..."
    ],
    "attachments": [
        {
            "url": "https://example.com/example.docx",
            "contentType": "application/vnd.openxmlformats-officedocument.wordprocessingml.document"
        }, 
        {
            "url": "https://example.com/example.doc",
            "contentType": "application/msword"
        }
    ]
}
```

### <a name="messageset-object"></a>MessageSet オブジェクト 
メッセージのセットを定義します。<br/><br/>

| プロパティ | type | 説明 |
|----|----|----|
| **messages** | [Message](#message-object)[] | **Message** オブジェクトの配列。 |
| **watermark** | string | セット内のメッセージの最大ウォーターマーク。 クライアントは、`watermark` を使用して、[ボットからメッセージを取得](bot-framework-rest-direct-line-1-1-receive-messages.md)したときに認識した最新のメッセージを示すことができます。 |

### <a name="attachment-object"></a>Attachment オブジェクト
イメージ以外の添付ファイルを定義します。<br/><br/> 

| プロパティ | type | 説明 |
|----|----|----|
| **contentType** | string | 添付ファイル内のコンテンツのメディアの種類。 |
| **URL** | string | 添付ファイルのコンテンツの URL。 |

### <a name="conversation-object"></a>Conversation オブジェクト
Direct Line 会話を定義します。<br/><br/>

| プロパティ | type | 説明 |
|----|----|----|
| **conversationId** | string | 指定されたトークンが有効な会話を一意に識別する ID。 |
| **token** | string | 指定された会話で有効なトークン。 |
| **expires_in** | number | トークンの有効期限が切れるまでの秒数。 |

### <a name="error-object"></a>Error オブジェクト
エラーを定義します。<br/><br/> 

| プロパティ | type | 説明 |
|----|----|----|
| **code** | string | エラー コード。 次の値のいずれか。**MissingProperty**、**MalformedData****NotFound****ServiceError****Internal****InvalidRange****NotSupported****NotAllowed****BadCertificate** |
| **message** | string | エラーの説明。 |
| **statusCode** | number | 状態コード。 |

### <a name="errormessage-object"></a>ErrorMessage オブジェクト
標準化されたメッセージ エラー ペイロード。<br/><br/> 


|        プロパティ        |          type          |                                 説明                                 |
|------------------------|------------------------|-----------------------------------------------------------------------------|
| <strong>error</strong> | [Error](#error-object) | エラーに関する情報を含む <strong>Error</strong> オブジェクト。 |

