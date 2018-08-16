---
title: 会話を終了する | Microsoft Docs
description: Direct Line API v3.0 を使用して会話を終了する方法について説明します。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ms.openlocfilehash: ac984609acfdd8f85088bd47ccded1f45e953b2c
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/18/2018
ms.locfileid: "39302825"
---
# <a name="end-a-conversation"></a>会話を終了する

クライアントまたはボットは、**endOfConversation** [アクティビティ](bot-framework-rest-connector-activities.md)を送信することによって、Direct Line 会話の終了を通知することができます。 

## <a name="send-an-endofconversation-activity"></a>endOfConversation アクティビティを送信する

**endOfConversation** アクティビティは、ボットとクライアント間の通信を終了します。 **endOfConversation** アクティビティが送信された後もまだ、クライアントは `HTTP GET` を使って[メッセージを取得](bot-framework-rest-direct-line-3-0-receive-activities.md#http-get)できますが、クライアントもボットも、会話にそれ以上メッセージを送信することはできません。 

会話を終了するには、POST 要求を発行して **endOfConversation** アクティビティを送信するだけです。

### <a name="request"></a>Request

```http
POST https://directline.botframework.com/v3/directline/conversations/abc123/activities
Authorization: Bearer RCurR_XV9ZA.cwA.BKA.iaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0
[other headers]
```

```json
{
    "type": "endOfConversation",
    "from": {
        "id": "user1"
    }
}
```

### <a name="response"></a>Response

要求が成功した場合、応答には送信されたアクティビティの ID が含まれます。

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
- [ボットにアクティビティを送信する](bot-framework-rest-direct-line-3-0-send-activity.md)
