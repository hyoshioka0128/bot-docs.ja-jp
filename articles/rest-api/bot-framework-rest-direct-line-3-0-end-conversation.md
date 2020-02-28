---
title: 会話を終了する - Bot Service
description: Direct Line API v3.0 を使用して会話を終了する方法について説明します。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
ms.openlocfilehash: 67c74bc3f51328370e4d6ba207855b20e4d79497
ms.sourcegitcommit: 308e6df385b9bac9c8d60f8b75eabc813b823c38
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/20/2020
ms.locfileid: "77520001"
---
# <a name="end-a-conversation"></a>会話を終了する

**endOfConversation** [アクティビティ](https://aka.ms/botSpecs-activitySchema)は、チャネルまたはボットが会話を終了したことを意味します。 

> [!NOTE] 
> **endOfConversation** イベントを送信するチャネルはごく少数に限られており、Cortana チャネルだけがこのイベントを受け入れます。 Direct Line などの他のチャネルはこの機能を実装しておらず、代わりにアクティビティを破棄または転送します。endOfConversation アクティビティへの対応は、チャネルごとに決定されます。

## <a name="send-an-endofconversation-activity"></a>endOfConversation アクティビティを送信する

Cortana チャンネルでの会話の終了を要求するには、チャンネルのメッセージング エンドポイントに会話の終了アクティビティを POST します。

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
- [ボットへのアクティビティの送信](bot-framework-rest-direct-line-3-0-send-activity.md)
