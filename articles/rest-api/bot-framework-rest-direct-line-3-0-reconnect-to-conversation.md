---
title: 会話への再接続 - Bot Service
description: Direct Line API v3.0 を使用して会話に再接続する方法について説明します。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 2/20/2020
ms.openlocfilehash: fda80c8e0dc4aaf5169730615b5cfc3acda16160
ms.sourcegitcommit: 9d77f3aff9521d819e88efd0fbd19d469b9919e7
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/16/2020
ms.locfileid: "77519911"
---
# <a name="reconnect-to-a-conversation"></a>会話への再接続

クライアントでのメッセージ受信に [WebSocket インターフェイス](bot-framework-rest-direct-line-3-0-receive-activities.md#connect-via-websocket)が使用されている場合、接続が失われたときに、再接続が必要になる場合があります。 このシナリオでは、会話への再接続に使用できる新しい WebSocket ストリーム URL を、クライアント側で生成する必要があります。

## <a name="generate-a-new-websocket-stream-url"></a>新しい WebSocket ストリーム URL を生成する

既存の会話への再接続に使用できる新しい WebSocket ストリーム URL を生成するには、次の要求を発行します。 

```http
GET https://directline.botframework.com/v3/directline/conversations/{conversationId}?watermark={watermark_value}
Authorization: Bearer SECRET_OR_TOKEN
```

この要求 URI では、 **{conversationId}** を会話 ID に、(`watermark` パラメーターが使用可能な場合は) **{watermark_value}** を基準値に置き換えてください。 `watermark` パラメーターは省略可能です。 要求 URI で `watermark` パラメーターが指定されている場合、会話は基準値から再生され、メッセージが失われないことが保証されます。 要求 URI で `watermark` パラメーターが省略されている場合は、再接続要求後の受信メッセージのみが再生されます。

次のスニペットは、再接続要求と応答の例を示しています。

### <a name="request"></a>Request

```http
GET https://directline.botframework.com/v3/directline/conversations/abc123?watermark=0000a-42
Authorization: Bearer RCurR_XV9ZA.cwA.BKA.iaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0y8qbOF5xPGfiCpg4Fv0y8qqbOF5x8qbOF5xn
```

### <a name="response"></a>Response

要求が成功した場合の応答には、会話の ID、トークン、および新しい WebSocket ストリーム URL が含まれます。

```http
HTTP/1.1 200 OK
[other headers]
```

```json
{
  "conversationId": "abc123",
  "token": "RCurR_XV9ZA.cwA.BKA.iaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0y8qbOF5xPGfiCpg4Fv0y8qqbOF5x8qbOF5xn",
  "streamUrl": "https://directline.botframework.com/v3/directline/conversations/abc123/stream?watermark=000a-4&amp;t=RCurR_XV9ZA.cwA..."
}
```

## <a name="reconnect-to-the-conversation"></a>会話に再接続する

クライアントでは、60 秒以内に新しい WebSocket ストリーム URL を使用して、[会話に再接続](bot-framework-rest-direct-line-3-0-receive-activities.md#connect-via-websocket)する必要があります。 この時間内に接続を確立できない場合、クライアントでは別の再接続要求を発行して、新しいストリーム URL を生成する必要があります。

Direct Line 設定で "拡張認証オプション" が有効になっている場合、正しく構成されたトークンが要求に接続されていないと、400 "MissingProperty" エラーが発生することがあります。 

## <a name="additional-resources"></a>その他のリソース

- [主要な概念](bot-framework-rest-direct-line-3-0-concepts.md)
- [認証](bot-framework-rest-direct-line-3-0-authentication.md)
- [WebSocket ストリーム経由のアクティビティの受信](bot-framework-rest-direct-line-3-0-receive-activities.md#connect-via-websocket)
