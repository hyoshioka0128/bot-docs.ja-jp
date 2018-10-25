---
title: メッセージをボットに送信する | Microsoft Docs
description: Direct Line API v1.1 を利用してボットにメッセージを送信する方法について説明します。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
ms.openlocfilehash: 360ec3a6a6c9a3be16370aaf445f24a237a702e3
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/24/2018
ms.locfileid: "49998019"
---
# <a name="send-a-message-to-the-bot"></a>メッセージをボットに送信する

> [!IMPORTANT]
> この記事では、Direct Line API 1.1 を利用してボットにメッセージを送信する方法について説明します。 クライアント アプリケーションとボットの間の新しい接続を作成する場合は、代わりに [Direct Line API 3.0](bot-framework-rest-direct-line-3-0-send-activity.md) を使用します。

クライアントは、Direct Line 1.1 プロトコルを使用してボットとメッセージを交換できます。 これらのメッセージは、ボットがサポートするスキーマ (Bot Framework v1 または Bot Framework v3) に変換されます。 クライアントは、要求ごとに 1 つのメッセージを送信できます。 

## <a name="send-a-message"></a>メッセージの送信

ボットにメッセージを送信するには、クライアントで [Message](bot-framework-rest-direct-line-1-1-api-reference.md#message-object) オブジェクトを作成してメッセージを定義し、`POST` 要求を `https://directline.botframework.com/api/conversations/{conversationId}/messages` に発行し、要求の本文でその Message オブジェクトを指定する必要があります。

次のスニペットは、メッセージ送信要求と応答の例を示しています。

### <a name="request"></a>Request

```http
POST https://directline.botframework.com/api/conversations/abc123/messages
Authorization: Bearer RCurR_XV9ZA.cwA.BKA.iaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0
[other headers]
```

```json
{
  "text": "hello",
  "from": "user1"
}
```

### <a name="response"></a>Response

メッセージがボットに配信されると、サービスは、ボットの状態コードを反映する HTTP 状態コードで応答します。 ボットでエラーが生成されると、クライアントからのメッセージ送信要求への応答として、HTTP 500 応答 ("内部サーバー エラー") が返されます。 POST が正常に実行されると、サービスから HTTP 204 状態コードが返されます。 応答の本文でデータは返されません。 クライアントのメッセージと、ボットからのメッセージは、[ポーリング](bot-framework-rest-direct-line-1-1-receive-messages.md)で取得できます。 

```http
HTTP/1.1 204 No Content
[other headers]
```

### <a name="total-time-for-the-send-message-requestresponse"></a>メッセージ送信要求/応答の合計時間

Direct Line 会話にメッセージを POST するための合計時間は次を足したものになります。

- HTTP 要求がクライアントから Direct Line サービスに至るための移動時間
- Direct Line 内の内部処理時間 (通常、120 ミリ秒未満)
- Direct Line サービスからボットまでの移動時間
- ボット内の処理時間
- HTTP 応答がクライアントに戻るための移動時間

## <a name="send-attachments-to-the-bot"></a>ボットに添付ファイルを送信する

状況によっては、クライアントから画像や文書などの添付ファイルをボットに送信しなければならないことがあります。 `POST /api/conversations/{conversationId}/messages` を利用して送信される [Message](bot-framework-rest-direct-line-1-1-api-reference.md#message-object) オブジェクト内で添付ファイルの [URL を指定する](#send-by-url)か、`POST /api/conversations/{conversationId}/upload` を利用して[添付ファイルをアップロードする](#upload-attachments)ことで、クライアントから添付ファイルを送信できます。

## <a id="send-by-url"></a> URL で添付ファイルを送信する

`POST /api/conversations/{conversationId}/messages` を利用して [Message](bot-framework-rest-direct-line-1-1-api-reference.md#message-object) オブジェクトの一部として 1 つ以上の添付ファイルを送信するには、メッセージの `images` 配列または `attachments` 配列 (あるいはその両方) に添付ファイル URL を指定します。

## <a id="upload-attachments"></a> アップロードで添付ファイルを送信する

クライアントから送信する画像や文書がデバイスにあるものの、それらのファイルに対応する URL がないことがしばしばあります。 そのような場合、クライアントでは、アップロードでボットに添付ファイルを送信する `POST /api/conversations/{conversationId}/upload` 要求を発行できます。 要求の形式とコンテンツは、クライアントから [1 つの添付ファイルを送信する](#upload-one-attachment)か、[複数の添付ファイルを送信する](#upload-multiple-attachments)かによって変わります。

### <a id="upload-one-attachment"></a> アップロードで 1 つの添付ファイルを送信する

アップロードで 1 つの添付ファイルを送信するには、次のような要求を発行します。 

```http
POST https://directline.botframework.com/api/conversations/{conversationId}/upload?userId={userId}
Authorization: Bearer SECRET_OR_TOKEN
Content-Type: TYPE_OF_ATTACHMENT
Content-Disposition: ATTACHMENT_INFO
[other headers]

[file content]
```

この要求 URI で、**{conversationId}** を会話の ID に置換し、**{userId}** をメッセージを送信するユーザーの ID に置換します。 要求ヘッダーで、`Content-Type` を設定して添付ファイルの種類を指定し、`Content-Disposition` を設定して添付ファイルのファイル名を指定します。

次のスニペットは、(単一の) 添付ファイル送信要求と応答の例を示しています。

#### <a name="request"></a>Request

```http
POST https://directline.botframework.com/api/conversations/abc123/upload?userId=user1
Authorization: Bearer RCurR_XV9ZA.cwA.BKA.iaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0
Content-Type: image/jpeg
Content-Disposition: name="file"; filename="badjokeeel.jpg"
[other headers]

[JPEG content]
```

#### <a name="response"></a>Response

要求が成功すると、アップロードの完了時にメッセージがボットに送信され、サービスから HTTP 204 状態コードが返されます。

```http
HTTP/1.1 204 No Content
[other headers]
```

### <a id="upload-multiple-attachments"></a> アップロードで複数の添付ファイルを送信する

アップロードで複数の添付ファイルを送信するには、`/api/conversations/{conversationId}/upload` エンドポイントに対して、マルチパート要求の `POST` を実行します。 要求の `Content-Type` ヘッダーを `multipart/form-data` に設定し、パートごとに `Content-Type` ヘッダーと `Content-Disposition` ヘッダーを含めて各添付ファイルの種類とファイル名を指定します。 要求 URI で、`userId` パラメーターを、メッセージを送信するユーザーの ID に設定します。 

`Content-Type` ヘッダー値 `application/vnd.microsoft.bot.message` を指定するパートを追加することで、要求内に [Message](bot-framework-rest-direct-line-1-1-api-reference.md#message-object) オブジェクトを含めることができます。 これにより、クライアントは、添付ファイルが含まれるメッセージをカスタマイズできます。 要求にメッセージが含まれる場合、送信前に、ペイロードの他のパートで指定された添付ファイルが、そのメッセージに添付ファイルとして追加されます。 

次のスニペットは、(複数の) 添付ファイル送信要求と応答の例を示しています。 この例では、要求により、テキストと 1 つの画像添付ファイルが含まれるメッセージが送信されます。 要求にさらにパートを追加して、このメッセージに複数の添付ファイルを含めることができます。

#### <a name="request"></a>Request

```http
POST https://directline.botframework.com/api/conversations/abc123/upload?userId=user1
Authorization: Bearer RCurR_XV9ZA.cwA.BKA.iaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0
Content-Type: multipart/form-data; boundary=----DD4E5147-E865-4652-B662-F223701A8A89
[other headers]

----DD4E5147-E865-4652-B662-F223701A8A89
Content-Type: image/jpeg
Content-Disposition: form-data; name="file"; filename="badjokeeel.jpg"
[other headers]

[JPEG content]

----DD4E5147-E865-4652-B662-F223701A8A89
Content-Type: application/vnd.microsoft.bot.message
[other headers]

{
  "text": "Hey I just IM'd you\n\nand this is crazy\n\nbut here's my webhook\n\nso POST me maybe",
  "from": "user1"
}

----DD4E5147-E865-4652-B662-F223701A8A89
```

#### <a name="response"></a>Response

要求が成功すると、アップロードの完了時にメッセージがボットに送信され、サービスから HTTP 204 状態コードが返されます。

```http
HTTP/1.1 204 No Content
[other headers]
```

## <a name="additional-resources"></a>その他のリソース

- [主要な概念](bot-framework-rest-direct-line-1-1-concepts.md)
- [認証](bot-framework-rest-direct-line-1-1-authentication.md)
- [会話の開始](bot-framework-rest-direct-line-1-1-start-conversation.md)
- [ボットからメッセージを受け取る](bot-framework-rest-direct-line-1-1-receive-messages.md)