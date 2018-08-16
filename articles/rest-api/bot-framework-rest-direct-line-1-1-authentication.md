---
title: 認証 | Microsoft Docs
description: Direct Line API v1.1 で API 要求を認証する方法について説明します。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ms.openlocfilehash: 6e83cc45e925f5d94b70260de6ac54e6f4052ca4
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/18/2018
ms.locfileid: "39301905"
---
# <a name="authentication"></a>Authentication

> [!IMPORTANT]
> この記事では、Direct Line API 1.1 での認証について説明します。 クライアント アプリケーションとボットの間の新しい接続を作成する場合は、代わりに [Direct Line API 3.0](bot-framework-rest-direct-line-3-0-authentication.md) を使用します。

クライアントは、Direct Line API 1.1 に対する要求を、[Bot Framework Portal](../bot-service-channel-connect-directline.md) の Direct Line チャネル構成ページから取得する**シークレット**を使用するか、ランタイム時に取得する**トークン**を使用して認証できます。

各要求の `Authorization` ヘッダーで、"Bearer" スキームまたは "BotConnector" スキームを使用して、シークレットまたはトークンを指定する必要があります。 

**Bearer スキーム**:
```http
Authorization: Bearer SECRET_OR_TOKEN
```

**BotConnector スキーム**:
```http
Authorization: BotConnector SECRET_OR_TOKEN
```

## <a name="secrets-and-tokens"></a>シークレットとトークン

Direct Line **シークレット**は、関連付けられているボットに属するすべての会話にアクセスするために使用できるマスター キーです。 **シークレット**は、**トークン**を取得するために使用することもできます。 シークレットには有効期限がありません。 

Direct Line **トークン**は、1 つの会話にアクセスするために使用できるキーです。 トークンには有効期限がありますが、更新できます。 

サービス間アプリケーションを作成する場合は、Direct Line API 要求の `Authorization` ヘッダー内に**シークレット**を指定することが最も簡単な方法です。 クライアントが Web ブラウザーまたはモバイル アプリで実行されるアプリケーションを記述する場合は、シークレットをトークン (1 つの会話でのみ機能し、更新されない限り有効期限が切れます) と交換し、その**トークン**を、Direct Line API 要求の `Authorization` ヘッダー内に指定できます。 自分にとって最適なセキュリティ モデルを選択してください。

> [!NOTE]
> Direct Line クライアント資格情報は、ボットの資格情報とは異なります。 これにより、キーを個別に変更でき、ボットのパスワードを公開せずにクライアント トークンを共有できます。 

## <a name="get-a-direct-line-secret"></a>Direct Line シークレットを取得する

<a href="https://dev.botframework.com/" target="_blank">Bot Framework Portal</a> で、ボット用の Direct Line チャネル構成ページを使用して、[Direct Line シークレットを取得](../bot-service-channel-connect-directline.md)できます。

![Direct Line 構成](../media/direct-line-configure.png)

## <a id="generate-token"></a> Direct Line トークンを生成する

1 つの会話にアクセスするために使用できる Direct Line トークンを生成するには、まず <a href="https://dev.botframework.com/" target="_blank">Bot Framework Portal</a> の Direct Line チャネル構成ページから Direct Line シークレットを取得します。 その後、次の要求を発行して、Direct Line シークレットを Direct Line トークンと交換します。

```http
POST https://directline.botframework.com/api/tokens/conversation
Authorization: Bearer SECRET
```

この要求の `Authorization` ヘッダーの **SECRET** を、Direct Line シークレットの値に置き換えます。

以下のスニペットは、トークン生成要求と応答の例を示しています。

### <a name="request"></a>Request

```http
POST https://directline.botframework.com/api/tokens/conversation
Authorization: Bearer RCurR_XV9ZA.cwA.BKA.iaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0
```

### <a name="response"></a>Response

要求が成功した場合、応答には 1 つの会話で有効なトークンが含まれます。 トークンの有効期限は 30 分です。 トークンを有効のままにするには、有効期限が切れる前に[トークンを更新する](#refresh-token)必要があります。

```http
HTTP/1.1 200 OK
[other headers]

"RCurR_XV9ZA.cwA.BKA.iaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0y8qbOF5xPGfiCpg4Fv0y8qqbOF5x8qbOF5xn"
```

### <a name="generate-token-versus-start-conversation"></a>トークンの生成と会話の開始

トークンの生成操作 (`POST /api/tokens/conversation`) と[会話の開始](bot-framework-rest-direct-line-1-1-start-conversation.md)操作 (`POST /api/conversations`) は、どちらの操作も、1 つの会話にアクセスするために使用できる `token` を返すという点で類似しています。 ただし、会話の開始操作とは異なり、トークンの生成操作では、会話は開始されず、ボットへの接触は行われません。 

トークンをクライアントに配布し、クライアントに会話を開始してほしい場合は、トークンの生成操作を使用します。 会話をすぐに開始するつもりの場合は、[会話の開始](bot-framework-rest-direct-line-1-1-start-conversation.md)操作を使用します。

## <a id="refresh-token"></a> Direct Line トークンを更新する

Direct Line トークンは、生成されてから 30 分間有効であり、期限が切れていない限り、時間制限なしに更新できます。 期限が切れたトークンは更新できません。 Direct Line トークンを更新するには、次の要求を発行します。

```http
POST https://directline.botframework.com/api/tokens/{conversationId}/renew
Authorization: Bearer TOKEN_TO_BE_REFRESHED
```

この要求では、URI の **{conversationId}** をトークンが有効な会話の ID に置き換え、`Authorization` ヘッダーの **TOKEN_TO_BE_REFRESHED** を更新する Direct Line トークンに置き換えます。

以下のスニペットは、トークン更新要求と応答の例を示しています。

### <a name="request"></a>Request

```http
POST https://directline.botframework.com/api/tokens/abc123/renew
Authorization: Bearer CurR_XV9ZA.cwA.BKA.iaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0y8qbOF5xPGfiCpg4Fv0y8qqbOF5x8qbOF5xn
```

### <a name="response"></a>Response

要求が成功した場合、応答には前のトークンと同じ会話で有効な新しいトークンが含まれます。 新しいトークンの有効期限は 30 分です。 新しいトークンを有効のままにするには、有効期限が切れる前に[トークンを更新する](#refresh-token)必要があります。

```http
HTTP/1.1 200 OK
[other headers]

"RCurR_XV9ZA.cwA.BKA.y8qbOF5xPGfiCpg4Fv0y8qqbOF5x8qbOF5xniaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0"
```

## <a name="additional-resources"></a>その他のリソース

- [主要な概念](bot-framework-rest-direct-line-1-1-concepts.md)
- [ボットを Direct Line に接続する](../bot-service-channel-connect-directline.md)