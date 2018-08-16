---
title: 会話を開始する | Microsoft Docs
description: Direct Line API v1.1 を使用して会話を開始する方法について説明します。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ms.openlocfilehash: 36645ce3811c77a3ca7ed697eeae63027fa1644a
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/18/2018
ms.locfileid: "39302508"
---
# <a name="start-a-conversation"></a>会話の開始

> [!IMPORTANT]
> この記事では、Direct Line API 1.1 を使用して会話を開始する方法について説明します。 クライアント アプリケーションとボットの間の新しい接続を作成する場合は、[Direct Line API 3.0](bot-framework-rest-direct-line-3-0-start-conversation.md) を使用してください。

Direct Line の会話はクライアントによって明示的に開かれ、ボットとクライアントが参加し、有効な資格情報を持っている限り続きます。 会話が開いている間は、ボットとクライアントの両方がメッセージを送信できます。 複数のクライアントが特定の会話に接続でき、各クライアントは、複数のユーザーの代理として参加できます。

## <a name="open-a-new-conversation"></a>新しい会話を開く

ボットとの新しい会話を開くには、次の要求を発行します。

```http
POST https://directline.botframework.com/api/conversations
Authorization: Bearer SECRET_OR_TOKEN
```

以下のスニペットは、会話の開始要求と応答の例を示しています。

### <a name="request"></a>Request

```http
POST https://directline.botframework.com/api/conversations
Authorization: Bearer RCurR_XV9ZA.cwA.BKA.iaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0y8qbOF5xPGfiCpg4Fv0y8qqbOF5x8qbOF5xn
```

### <a name="response"></a>Response

要求が成功した場合、応答には、その会話の ID、トークン、そのトークンの有効期限が切れるまでの秒数を示す値が含まれます。

```http
HTTP/1.1 200 OK
[other headers]
```

```json
{
  "conversationId": "abc123",
  "token": "RCurR_XV9ZA.cwA.BKA.iaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0y8qbOF5xPGfiCpg4Fv0y8qqbOF5x8qbOF5xn",
  "expires_in": 1800
}
```

## <a name="start-conversation-versus-generate-token"></a>会話の開始とトークンの生成の比較

会話の開始操作 (`POST /api/conversations`) と[トークンの生成](bot-framework-rest-direct-line-1-1-authentication.md#generate-token)操作 (`POST /api/tokens/conversation`) は、どちらの操作も、1 つの会話にアクセスするために使用できる `token` を返すという点で類似しています。 ただし、会話の開始操作は、会話の開始およびボットとの接触も行いますが、トークンの生成操作はそのどちらも行いません。 

会話をすぐに開始するつもりの場合は、会話の開始操作を使用してください。 トークンをクライアントに配布し、クライアントからの会話の開始を求める場合は、[トークンの生成](bot-framework-rest-direct-line-1-1-authentication.md#generate-token)操作を使用してください。 

## <a name="additional-resources"></a>その他のリソース

- [主要な概念](bot-framework-rest-direct-line-1-1-concepts.md)
- [認証](bot-framework-rest-direct-line-1-1-authentication.md)