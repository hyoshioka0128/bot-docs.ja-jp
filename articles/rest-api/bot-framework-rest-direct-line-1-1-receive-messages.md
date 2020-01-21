---
title: ボットからメッセージを受信する - Bot Service
description: Direct Line API v1.1 を使用して、ボットからメッセージを受信する方法について説明します。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
ms.openlocfilehash: c88d3f363bf4bcc40fa7a21aa1fcdd0b764abe1e
ms.sourcegitcommit: f8b5cc509a6351d3aae89bc146eaabead973de97
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/09/2020
ms.locfileid: "75789698"
---
# <a name="receive-messages-from-the-bot"></a>ボットからメッセージを受信する

> [!IMPORTANT]
> この記事では、Direct Line API 1.1 を使用して、ボットからメッセージを受信する方法について説明します。 クライアント アプリケーションとボットの間の新しい接続を作成する場合は、代わりに [Direct Line API 3.0](bot-framework-rest-direct-line-3-0-receive-activities.md) を使用します。

クライアントは、メッセージを受信するには、Direct Line 1.1 プロトコルを使用して `HTTP GET` インターフェイスをポーリングする必要があります。 

## <a name="retrieve-messages-with-http-get"></a>HTTP GET でメッセージを取得する

特定の会話のメッセージを取得するには、`GET` 要求を `api/conversations/{conversationId}/messages` エンドポイントに発行します。オプションで、クライアントが認識した最新のメッセージを示す `watermark` パラメーターを指定できます。 メッセージが含まれていない場合でも、更新された `watermark` 値が JSON 応答内に返されます。

以下のスニペットは、メッセージ取得要求と応答の例を示しています。 メッセージ取得応答には、[MessageSet](bot-framework-rest-direct-line-1-1-api-reference.md#messageset-object) のプロパティとして、`watermark` が含まれます。 クライアントは、メッセージが返らなくなるまで `watermark` 値を進めることで、入手できるメッセージをすべて取得する必要があります。 

### <a name="request"></a>Request

```http
GET https://directline.botframework.com/api/conversations/abc123/messages?watermark=0001a-94
Authorization: Bearer RCurR_XV9ZA.cwA.BKA.iaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0
```

### <a name="response"></a>Response

```http
HTTP/1.1 200 OK
[other headers]
```

```json
{
    "messages": [
        {
            "conversation": "abc123",
            "id": "abc123|0000",
            "text": "hello",
            "from": "user1"
        }, 
        {
            "conversation": "abc123",
            "id": "abc123|0001",
            "text": "Nice to see you, user1!",
            "from": "bot1"
        }
    ],
    "watermark": "0001a-95"
}
```

## <a name="timing-considerations"></a>タイミングに関する考慮事項

Direct Line は潜在的なタイミングのギャップを伴うマルチパート プロトコルですが、そのプロトコルとサービスは信頼性の高いクライアントを簡単に構築できるように設計されています。 メッセージ取得応答内に返される `watermark` プロパティは、信頼できます。 ウォーターマーク文字列を再生する限り、クライアントがメッセージを見逃すことはありません。

クライアントは、その使用目的と一致するポーリング間隔を選択する必要があります。

- サービス間アプリケーションは、多くの場合、5 秒または 10 秒のポーリング間隔を使用します。

- クライアント向けのアプリケーションは、多くの場合、1 秒のポーリング間隔を使用し、クライアントが送信するすべてのメッセージの後で (ボットの応答を迅速に取得するために) 300 ミリ秒以内に追加要求を発行します。 この 300 ミリ秒の遅延は、ボットの速度と転送時間に基づいて調整する必要があります。

## <a name="additional-resources"></a>その他のリソース

- [主要な概念](bot-framework-rest-direct-line-1-1-concepts.md)
- [認証](bot-framework-rest-direct-line-1-1-authentication.md)
- [会話の開始](bot-framework-rest-direct-line-1-1-start-conversation.md)
- [メッセージをボットに送信する](bot-framework-rest-direct-line-1-1-send-message.md)