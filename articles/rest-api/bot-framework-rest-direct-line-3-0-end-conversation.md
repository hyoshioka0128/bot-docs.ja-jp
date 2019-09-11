---
title: 会話を終了する | Microsoft Docs
description: Direct Line API v3.0 を使用して会話を終了する方法について説明します。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
ms.openlocfilehash: 3006672ba8459ebec7d754a7ec31679debdba4aa
ms.sourcegitcommit: e815e786413296deea0bd78e5a495df329a9a7cb
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/10/2019
ms.locfileid: "70876162"
---
# <a name="end-a-conversation"></a>会話を終了する

**endOfConversation** [アクティビティ](https://aka.ms/botSpecs-activitySchema)は、チャネルまたはボットが会話を終了したことを意味します。 

> [!NOTE] 
> **endOfConversation** イベントを送信するチャネルはごく少数に限られており、Cortana チャネルだけがこのイベントを受け入れます。 Direct Line などの他のチャネルはこの機能を実装しておらず、代わりにアクティビティを破棄または転送します。endOfConversation アクティビティへの対応は、チャネルごとに決定されます。 DirectLine クライアントを設計している場合は、既に終了した会話にボットがアクティビティを送信したときにエラーを生成するなど、クライアントが適切に動作するよう更新します。

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
- [ボットへのアクティビティの送信](bot-framework-rest-direct-line-3-0-send-activity.md)
