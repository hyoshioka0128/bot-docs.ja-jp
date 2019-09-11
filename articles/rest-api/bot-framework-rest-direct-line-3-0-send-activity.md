---
title: ボットにアクティビティを送信する | Microsoft Docs
description: Direct Line API v3.0 を利用してボットにアクティビティを送信する方法について説明します。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
ms.openlocfilehash: 5c7ac61da6c2e0d09fb2f8dc4cd0bf3961bcfc4f
ms.sourcegitcommit: e815e786413296deea0bd78e5a495df329a9a7cb
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/10/2019
ms.locfileid: "70875995"
---
# <a name="send-an-activity-to-the-bot"></a>ボットにアクティビティを送信する

Direct Line 3.0 プロトコルを利用すると、**メッセージ** アクティビティ、**入力**アクティビティ、ボットでサポートされるカスタム アクティビティなど、さまざまな種類の[アクティビティ](https://aka.ms/botSpecs-activitySchema)をクライアントとボットで交換できます。 クライアントでは、要求ごとにアクティビティを 1 つを送信できます。 

## <a name="send-an-activity"></a>アクティビティを送信する

ボットにアクティビティを送信するには、クライアントで [Activity][] オブジェクトを作成してアクティビティを定義し、`POST` 要求を `https://directline.botframework.com/v3/directline/conversations/{conversationId}/activities` に発行し、要求の本文でその Activity オブジェクトを指定する必要があります。

次のスニペットは、アクティビティ送信要求と応答の例を示しています。

### <a name="request"></a>Request

```http
POST https://directline.botframework.com/v3/directline/conversations/abc123/activities
Authorization: Bearer RCurR_XV9ZA.cwA.BKA.iaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0
Content-Type: application/json
[other headers]
```

```json
{
    "type": "message",
    "from": {
        "id": "user1"
    },
    "text": "hello"
}
```

### <a name="response"></a>Response

アクティビティがボットに配信されるとき、サービスでは、ボットのステータス コードを反映する HTTP ステータス コードで応答します。 ボットによりエラーが生成された場合、そのアクティビティ送信要求への応答で、HTTP 502 応答 ("ゲートウェイが不適切です") がクライアントに返されます。

> [!NOTE]
> これは、正しいトークンが使用されていないことが原因で発生する可能性があります。 アクティビティの送信には、"*会話の開始*" に対して受信したトークンのみを使用できます。

投稿が成功した場合、ボットに送信されたアクティビティの ID を指定する JSON ペイロードが応答に含まれます。

```http
HTTP/1.1 200 OK
[other headers]
```

```json
{
    "id": "0001"
}
```

### <a name="total-time-for-the-send-activity-requestresponse"></a>アクティビティ送信要求/応答の合計時間

Direct Line 会話にメッセージを投稿するための合計時間は次を足したものになります。

- HTTP 要求がクライアントから Direct Line サービスに至るための移動時間
- Direct Line 内の内部処理時間 (通常、120 ミリ秒未満)
- Direct Line サービスからボットまでの移動時間
- ボット内の処理時間
- HTTP 応答がクライアントに戻るための移動時間

## <a name="send-attachments-to-the-bot"></a>ボットに添付ファイルを送信する

クライアントでは、画像や文書など、添付ファイルをボットに送信しなければならない状況があります。 `POST /v3/directline/conversations/{conversationId}/activities` を利用して送信される [Activity][] オブジェクト内で添付ファイルの [URL を指定する](#send-by-url)か、`POST /v3/directline/conversations/{conversationId}/upload` を利用して[添付ファイルをアップロードする](#upload-attachments)ことで、クライアントでは添付ファイルを送信できます。

## <a id="send-by-url"></a> URL で添付ファイルを送信する

`POST /v3/directline/conversations/{conversationId}/activities` を利用し、[Activity][] オブジェクトの一部として 1 つまたは複数の添付ファイルを送信するには、Activity オブジェクト内に 1 つまたは複数の [Attachment][] オブジェクトを含め、各 Attachment オブジェクトの `contentUrl` プロパティを設定し、添付ファイルの HTTP、HTTPS、`data` URI を指定します。

## <a id="upload-attachments"></a> アップロードで添付ファイルを送信する

クライアントから送信する画像や文書がデバイスにあるものの、それらのファイルに対応する URL がないことがしばしばあります。 そのような場合、クライアントでは、アップロードでボットに添付ファイルを送信する `POST /v3/directline/conversations/{conversationId}/upload` 要求を発行できます。 要求の形式とコンテンツは、クライアントから [1 つの添付ファイルを送信する](#upload-one-attachment)か、[複数の添付ファイルを送信する](#upload-multiple-attachments)かによって変わります。

### <a id="upload-one-attachment"></a> アップロードで 1 つの添付ファイルを送信する

アップロードで 1 つの添付ファイルを送信するには、次のような要求を発行します。 

```http
POST https://directline.botframework.com/v3/directline/conversations/{conversationId}/upload?userId={userId}
Authorization: Bearer SECRET_OR_TOKEN
Content-Type: TYPE_OF_ATTACHMENT
Content-Disposition: ATTACHMENT_INFO
[other headers]

[file content]
```

この要求 URI で、 **{conversationId}** を会話の ID に置換し、 **{userId}** をメッセージを送信するユーザーの ID に置換します。 `userId` パラメーターは必須です。 要求ヘッダーで、`Content-Type` を設定して添付ファイルの種類を指定し、`Content-Disposition` を設定して添付ファイルのファイル名を指定します。

次のスニペットは、(単一の) 添付ファイル送信要求と応答の例を示しています。

#### <a name="request"></a>Request

```http
POST https://directline.botframework.com/v3/directline/conversations/abc123/upload?userId=user1
Authorization: Bearer RCurR_XV9ZA.cwA.BKA.iaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0
Content-Type: image/jpeg
Content-Disposition: name="file"; filename="badjokeeel.jpg"
[other headers]

[JPEG content]
```

#### <a name="response"></a>Response

要求が成功した場合、アップロードが完了したとき、**メッセージ** アクティビティがボットに送信されます。クライアントにより受け取られる応答には、送信されたアクティビティの ID が含まれます。

```http
HTTP/1.1 200 OK
[other headers]
```

```json
{
  "id": "0003"
}
```

### <a id="upload-multiple-attachments"></a> アップロードで複数の添付ファイルを送信する

アップロードで複数の添付ファイルを送信するには、`/v3/directline/conversations/{conversationId}/upload` エンドポイントに対して、マルチパート要求の `POST` を実行します。 要求の `Content-Type` ヘッダーを `multipart/form-data` に設定し、パートごとに `Content-Type` ヘッダーと `Content-Disposition` ヘッダーを含めて各添付ファイルの種類とファイル名を指定します。 要求 URI で、`userId` パラメーターを、メッセージを送信するユーザーの ID に設定します。 

`Content-Type` ヘッダー値 `application/vnd.microsoft.activity` を指定するパートを追加することで、要求内に `Activity` オブジェクトを含めることができます。 要求にアクティビティが含まれる場合、ペイロードの他のパートで指定される添付ファイルが添付ファイルとして送信前にそのアクティビティに追加されます。 要求にアクティビティが含まれない場合、指定の添付ファイルが送信されるコンテナーとして空のアクティビティが作成されます。

次のスニペットは、添付ファイル (複数) 送信要求と応答の例を示しています。 この例では、要求により、テキストと 1 つの画像添付ファイルが含まれるメッセージが送信されます。 要求にさらにパートを追加して、このメッセージに複数の添付ファイルを含めることができます。

#### <a name="request"></a>Request

```http
POST https://directline.botframework.com/v3/directline/conversations/abc123/upload?userId=user1
Authorization: Bearer RCurR_XV9ZA.cwA.BKA.iaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0
Content-Type: multipart/form-data; boundary=----DD4E5147-E865-4652-B662-F223701A8A89
[other headers]

----DD4E5147-E865-4652-B662-F223701A8A89
Content-Type: image/jpeg
Content-Disposition: form-data; name="file"; filename="badjokeeel.jpg"
[other headers]

[JPEG content]

----DD4E5147-E865-4652-B662-F223701A8A89
Content-Type: application/vnd.microsoft.activity
[other headers]

{
  "type": "message",
  "from": {
    "id": "user1"
  },
  "text": "Hey I just IM'd you\n\nand this is crazy\n\nbut here's my webhook\n\nso POST me maybe"
}

----DD4E5147-E865-4652-B662-F223701A8A89
```

#### <a name="response"></a>Response

要求が成功した場合、アップロードが完了したとき、メッセージ アクティビティがボットに送信されます。クライアントにより受け取られる応答には、送信されたアクティビティの ID が含まれます。

```http
HTTP/1.1 200 OK
[other headers]
```

```json
{
    "id": "0004"
}
```

## <a name="additional-resources"></a>その他のリソース

- [主要な概念](bot-framework-rest-direct-line-3-0-concepts.md)
- [認証](bot-framework-rest-direct-line-3-0-authentication.md)
- [会話の開始](bot-framework-rest-direct-line-3-0-start-conversation.md)
- [会話への再接続](bot-framework-rest-direct-line-3-0-reconnect-to-conversation.md)
- [ボットからのアクティビティの受信](bot-framework-rest-direct-line-3-0-receive-activities.md)
- [会話を終了する](bot-framework-rest-direct-line-3-0-end-conversation.md)
- [Bot Framework のアクティビティ スキーマ](https://aka.ms/botSpecs-activitySchema)

[Activity]: bot-framework-rest-connector-api-reference.md#activity-object
[Attachment]: bot-framework-rest-connector-api-reference.md#attachment-object
