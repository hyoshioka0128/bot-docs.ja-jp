---
title: ボットからアクティビティを受信する | Microsoft Docs
description: Direct Line API v3.0 を使用して、ボットからアクティビティを受信する方法について説明します。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ms.openlocfilehash: 2993b75a26ed987a472c241133a62727e3b285d2
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/18/2018
ms.locfileid: "39303497"
---
# <a name="receive-activities-from-the-bot"></a>ボットからアクティビティを受信する

クライアントは、Direct Line 3.0 プロトコルを使用して、`WebSocket` ストリーム経由でアクティビティを受信するか、`HTTP GET` 要求を発行してアクティビティを取得できます。 

## <a name="websocket-vs-http-get"></a>WebSocket と HTTP GET

WebSocket のストリーミング はメッセージを効率的にクライアントにプッシュし、GET インターフェイスはクライアントがメッセージを明示的に要求できるようにします。 多くの場合、効率の良さから WebSocket メカニズムが推奨されますが、WebSocket を使用できないクライアントまたは会話履歴を取得したいクライアントでは、GET メカニズムが有用です。 

WebSocket と HTTP GET は、どちらも、すべての[アクティビティの種類](bot-framework-rest-connector-activities.md)を利用できるわけではありません。 次の表で、Direct Line プロトコルを使用するクライアントで利用可能なさまざまな種類のアクティビティについて説明します。

| アクティビティの種類 | 利用可用性 | 
|----|----|
| message | HTTP GET と WebSocket |
| typing | WebSocket のみ |
| conversationUpdate | クライアント経由では送信/受信されない |
| contactRelationUpdate | Direct Line ではサポートされない |
| endOfConversation | HTTP GET と WebSocket |
| その他のすべてのアクティビティの種類 | HTTP GET と WebSocket |

## <a id="connect-via-websocket"> </a>WebSocket ストリーム経由でアクティビティを受信する

クライアントが[会話の開始](bot-framework-rest-direct-line-3-0-start-conversation.md)要求を送信してボットとの会話を開くと、サービスの応答には、クライアントが後で WebSocket 経由で接続するために使用できる `streamUrl` プロパティが含まれます。 ストリーム URL は事前に承認されているため、WebSocket 経由で接続するクライアントの要求には、`Authorization` ヘッダーは必要ありません。

次の例は、`streamUrl` を使用して WebSocket 経由で接続する要求を示しています。

```http
-- connect to wss://directline.botframework.com --
GET /v3/directline/conversations/abc123/stream?t=RCurR_XV9ZA.cwA..."
Upgrade: websocket
Connection: upgrade
[other headers]
```

サービスは、状態コード HTTP 101 ("プロトコルの切り替え中") で応答します。

```http
HTTP/1.1 101 Switching Protocols
[other headers]
```

### <a name="receive-messages"></a>メッセージを受信する

WebSocket 経由で接続した後、クライアントは、Direct Line サービスから次の種類のメッセージを受信できます。

- [ActivitySet](bot-framework-rest-direct-line-3-0-api-reference.md#activityset-object) を含むメッセージ。これには、1 つまたは複数のアクティビティとウォーターマーク (後述) が含まれています。
- 接続がまだ有効であることを確認するために Direct Line サービスが使用する空のメッセージ。
- その他の種類。後で定義されます。 これらの種類は、JSON ルートのプロパティによって識別されます。

`ActivitySet` には、ボットと会話のすべてのユーザーによって送信されたメッセージが含まれます。 次の例は、1 つのメッセージを含む `ActivitySet` を示しています。

```json
{
    "activities": [
        {
            "type": "message",
            "channelId": "directline",
            "conversation": {
                "id": "abc123"
            },
            "id": "abc123|0000",
            "from": {
                "id": "user1"
            },
            "text": "hello"
        }
    ],
    "watermark": "0000a-42"
}
```

### <a name="process-messages"></a>メッセージを処理する

クライアントは、各 [ActivitySet](bot-framework-rest-direct-line-3-0-api-reference.md#activityset-object) で受信する `watermark` 値を追跡して、接続が失われたために[会話を再接続する](bot-framework-rest-direct-line-3-0-reconnect-to-conversation.md)必要がある場合に、ウォーターマークを使用してメッセージが失われていないことを保証できるようにする必要があります。 クライアントは、`watermark` プロパティが `null` である `ActivitySet` を受信した場合は、その値を無視する必要があり、前に受信したウォーターマークを上書きしてはなりません。

クライアントは、Direct Line サービスから受信する空のメッセージを無視する必要があります。

クライアントは、接続を確認するために Direct Line サービスに空のメッセージを送信できます。 Direct Line サービスは、クライアントから受信する空のメッセージを無視します。

Direct Line サービスが、特定の条件下で WebSocket 接続を強制的に閉じる場合があります。 クライアントは、`endOfConversation` アクティビティが受信されない場合は、会話に再接続するために使用できる[新しい WebSocket ストリーム URL を生成](bot-framework-rest-direct-line-3-0-reconnect-to-conversation.md)できます。 

WebSocket ストリームにはライブ更新とごく最近のメッセージが含まれています (これは WebSocket 経由で接続するための呼び出しが発行されたためです) が、最新の `POST` の前に `/v3/directline/conversations/{id}` に送信されたメッセージは含まれていません。 会話内でこれまでに送信されたメッセージを取得するには、次に説明する `HTTP GET` を使用します。

## <a id="http-get"></a> HTTP GET を使用してアクティビティを取得する

Websocket を使用できないクライアント、または会話の履歴を取得したいクライアントは、`HTTP GET` を使用してアクティビティを取得できます。

特定の会話のメッセージを取得するには、`GET` 要求を `/v3/directline/conversations/{conversationId}/activities` エンドポイントに発行します。オプションで、クライアントが認識した最新のメッセージを示す `watermark` パラメーターを指定できます。 

以下のスニペットは、会話アクティビティ取得要求と応答の例を示しています。 会話アクティビティ取得応答には、[ActivitySet](bot-framework-rest-direct-line-3-0-api-reference.md#activityset-object) のプロパティとして `watermark` が含まれます。 クライアントは、アクティビティが返らなくなるまで `watermark` 値を進めることで、入手できるアクティビティをすべて取得する必要があります。

### <a name="request"></a>要求

```http
GET https://directline.botframework.com/v3/directline/conversations/abc123/activities?watermark=0001a-94
Authorization: Bearer RCurR_XV9ZA.cwA.BKA.iaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0
```

### <a name="response"></a>応答

```http
HTTP/1.1 200 OK
[other headers]
```

```json
{
    "activities": [
        {
            "type": "message",
            "channelId": "directline",
            "conversation": {
                "id": "abc123"
            },
            "id": "abc123|0000",
            "from": {
                "id": "user1"
            },
            "text": "hello"
        }, 
        {
            "type": "message",
            "channelId": "directline",
            "conversation": {
                "id": "abc123"
            },
            "id": "abc123|0001",
            "from": {
                "id": "bot1"
            },
            "text": "Nice to see you, user1!"
        }
    ],
    "watermark": "0001a-95"
}
```

## <a name="timing-considerations"></a>タイミングに関する考慮事項

ほとんどのクライアントは、完全なメッセージの履歴を保持することを望んでいます。 Direct Line は潜在的なタイミングのギャップを伴うマルチパート プロトコルですが、そのプロトコルとサービスは信頼性の高いクライアントを簡単に構築できるように設計されています。

- WebSocket ストリームと会話アクティビティ取得応答で送信される `watermark` プロパティは信頼できます。 ウォーターマーク文字列を再生する限り、クライアントがメッセージを見逃すことはありません。

- クライアントが会話を開始して WebSocket ストリームに接続すると、POST の後で送信されたが、ソケットが開く前に送信されたすべてのアクティビティは、新しいアクティビティの前に再生されます。

- クライアントが WebSocket ストリームに接続されているときに (履歴を更新するために) 会話アクティビティ取得要求を発行した場合、両方のチャネルでアクティビティが重複することがあります。 クライアントは、既知のすべてのアクティビティ ID を追跡して、重複するアクティビティが発生した場合にそれを拒否できるようにする必要があります。

`HTTP GET` を使用してポーリングを行うクライアントは、その使用目的と一致するポーリング間隔を選択する必要があります。

- サービス間アプリケーションは、多くの場合、5 秒または 10 秒のポーリング間隔を使用します。

- クライアント向けのアプリケーションは、多くの場合、1 秒のポーリング間隔を使用し、クライアントが送信するすべてのメッセージの後で (ボットの応答を迅速に取得するために) 300 ミリ秒以内に追加要求を発行します。 この 300 ミリ秒の遅延は、ボットの速度と転送時間に基づいて調整する必要があります。

## <a name="additional-resources"></a>その他のリソース

- [主要な概念](bot-framework-rest-direct-line-3-0-concepts.md)
- [認証](bot-framework-rest-direct-line-3-0-authentication.md)
- [会話の開始](bot-framework-rest-direct-line-3-0-start-conversation.md)
- [会話への再接続](bot-framework-rest-direct-line-3-0-reconnect-to-conversation.md)
- [ボットへのアクティビティの送信](bot-framework-rest-direct-line-3-0-send-activity.md)
- [会話の終了](bot-framework-rest-direct-line-3-0-end-conversation.md)