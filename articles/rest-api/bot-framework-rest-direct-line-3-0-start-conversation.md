---
title: 会話の開始 - Bot Service
description: Direct Line API v3.0 を使用して会話を開始する方法について説明します。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
ms.openlocfilehash: db6fc2049cbf5fe44ac4f8713c17b2081019fd82
ms.sourcegitcommit: 308e6df385b9bac9c8d60f8b75eabc813b823c38
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/20/2020
ms.locfileid: "77519921"
---
# <a name="start-a-conversation"></a>会話の開始

Direct Line の会話はクライアントによって明示的に開かれ、ボットとクライアントが参加し、有効な資格情報を持っている限り続きます。 会話が開いている間は、ボットとクライアントの両方がメッセージを送信できます。 複数のクライアントが特定の会話に接続でき、各クライアントは、複数のユーザーの代理として参加できます。

## <a name="open-a-new-conversation"></a>新しい会話を開く

クライアントから新しい会話を開始するには、/v3/directline/conversations エンドポイントに POST を発行します。

```http
POST https://directline.botframework.com/v3/directline/conversations
Authorization: Bearer SECRET_OR_TOKEN
```

以下のスニペットは、会話の開始要求と応答の例を示しています。

### <a name="request"></a>Request

```http
POST https://directline.botframework.com/v3/directline/conversations
Authorization: Bearer RCurR_XV9ZA.cwA.BKA.iaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0y8qbOF5xPGfiCpg4Fv0y8qqbOF5x8qbOF5xn
```

### <a name="response"></a>Response

要求が成功すると、応答には、会話の ID、トークン、トークンの有効期限が切れるまでの秒数を示す値、およびクライアントが [WebSocket ストリーム経由でアクティビティを受信する](bot-framework-rest-direct-line-3-0-receive-activities.md#connect-via-websocket)ために使用できるストリーム URL が含まれます。

```http
HTTP/1.1 201 Created
[other headers]
```

```json
{
  "conversationId": "abc123",
  "token": "RCurR_XV9ZA.cwA.BKA.iaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0y8qbOF5xPGfiCpg4Fv0y8qqbOF5x8qbOF5xn",
  "expires_in": 1800,
  "streamUrl": "https://directline.botframework.com/v3/directline/conversations/abc123/stream?t=RCurR_XV9ZA.cwA..."
}
```

通常、会話の開始要求は新しい会話を開始するために使用され、新しい会話が正常に開始された場合は、**HTTP 201** 状態コードが返されます。 ただし、クライアントが `Authorization` ヘッダー内で、会話の開始操作を使用して会話を開始するために以前使用されたことがある Direct Line トークンを使用して会話の開始要求を送信した場合、要求は受理できたが、会話は (すでに存在しているために) 作成されなかったことを示す **HTTP 200** 状態コードが返されます。

> [!TIP]
> WebSocket stream URL に接続されるまで 60 秒の猶予があります。 この時間内に接続を確立できない場合は、[会話に再接続して](bot-framework-rest-direct-line-3-0-reconnect-to-conversation.md)新しいストリーム URL を生成する必要があります。

## <a name="start-conversation-versus-generate-token"></a>会話の開始とトークンの生成の比較

会話の開始操作 (`POST /v3/directline/conversations`) と[トークンの生成](bot-framework-rest-direct-line-3-0-authentication.md#generate-token)操作 (`POST /v3/directline/tokens/generate`) は、どちらの操作も、1 つの会話にアクセスするために使用できる `token` を返すという点で類似しています。 ただし、会話の開始操作は、会話の開始、ボットとの接触、および WebSocket stream URL の作成も行いますが、トークンの生成操作はこれらの操作を行いません。 

クライアントですぐに会話を開始する場合は、会話の開始操作を使用します。 トークンをクライアントに配布し、クライアントからの会話の開始を求める場合は、[トークンの生成](bot-framework-rest-direct-line-3-0-authentication.md#generate-token)操作を使用してください。 

## <a name="additional-resources"></a>その他のリソース

- [主要な概念](bot-framework-rest-direct-line-3-0-concepts.md)
- [認証](bot-framework-rest-direct-line-3-0-authentication.md)
- [WebSocket ストリーム経由のアクティビティの受信](bot-framework-rest-direct-line-3-0-receive-activities.md#connect-via-websocket)
- [会話への再接続](bot-framework-rest-direct-line-3-0-reconnect-to-conversation.md)
