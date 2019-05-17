---
title: 認証 |Microsoft Docs
description: Direct Line API v3.0 で API 要求を認証する方法について説明します。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 04/10/2019
ms.openlocfilehash: 717a95d580bad218ade9a884522724f1c6b96ad7
ms.sourcegitcommit: f84b56beecd41debe6baf056e98332f20b646bda
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 05/03/2019
ms.locfileid: "65032641"
---
# <a name="authentication"></a>Authentication

クライアントは、Direct Line API 3.0 に対する要求を、[Bot Framework Portal](../bot-service-channel-connect-directline.md) の Direct Line チャネル構成ページから取得する**シークレット**を使用するか、ランタイム時に取得する**トークン**を使用して認証できます。 シークレットまたはトークンは、次の書式で各要求の `Authorization` ヘッダー内に指定する必要があります。 

```http
Authorization: Bearer SECRET_OR_TOKEN
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
POST https://directline.botframework.com/v3/directline/tokens/generate
Authorization: Bearer SECRET
```

この要求の `Authorization` ヘッダーの **SECRET** を、Direct Line シークレットの値に置き換えます。

以下のスニペットは、トークン生成要求と応答の例を示しています。

### <a name="request"></a>Request

```http
POST https://directline.botframework.com/v3/directline/tokens/generate
Authorization: Bearer RCurR_XV9ZA.cwA.BKA.iaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0
```

要求ペイロードにはトークン パラメーターが含まれています。これは省略可能ですが、使用することをお勧めします。 Direct Line サービスに送り返すことができるトークンを生成するときに、次のペイロードを提供して、接続セキュリティを強化します。 これらの値を含めることによって、Direct Line によるユーザー ID と名前の追加セキュリティ検証が実行可能になり、悪意のあるクライアントによるこれらの値の改ざんが禁止されます。 これらの値を含めて、Direct Line の "_会話更新_" アクティビティ送信機能を向上させることもできます。これにより、ユーザーが会話に参加したときに直ちに会話更新を生成できるようになります。 この情報が指定されていない場合、ユーザーは、Direct Line が会話更新を送信する前に、コンテンツを送信する必要があります。

```json
{
  "user": {
    "id": "string",
    "name": "string"
  },
  "trustedOrigins": [
    "string"
  ]
}
```

| パラメーター | Type | 説明 |
| :--- | :--- | :--- |
| `user.id` | string | 省略可能。 トークン内でエンコードするためのチャネル固有のユーザー ID。 Direct Line ユーザーの場合、`dl_` で始まる必要があります。 会話ごとに一意のユーザー ID を作成できます。セキュリティを強化するために、この ID は推測できないものにします。 |
| `user.name` | string | 省略可能。 トークン内でエンコードするためのユーザーの表示用フレンドリ名。 |
| `trustedOrigins` | 文字列配列 | 省略可能。 トークン内に埋め込む信頼されたドメインの一覧。 これらはボットの Web チャット クライアントをホストできるドメインです。 これはご自身のボット用の Direct Line 構成ページの一覧と一致する必要があります。 |

### <a name="response"></a>Response

要求が成功した場合、応答には、1 つの会話で有効な `token` と、このトークンの有効期限が切れるまでの秒数を示す `expires_in` 値が含まれます。 トークンを有効のままにするには、有効期限が切れる前に[トークンを更新する](#refresh-token)必要があります。

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

### <a name="generate-token-versus-start-conversation"></a>トークンの生成と会話の開始

トークンの生成操作 (`POST /v3/directline/tokens/generate`) と[会話の開始](bot-framework-rest-direct-line-3-0-start-conversation.md)操作 (`POST /v3/directline/conversations`) は、どちらの操作も、1 つの会話にアクセスするために使用できる `token` を返すという点で類似しています。 ただし、会話の開始操作とは異なり、トークンの生成操作では、会話は開始されず、ボットへの接触は行われず、WebSocket のストリーミング URL は作成されません。 

トークンをクライアントに配布し、クライアントに会話を開始してほしい場合は、トークンの生成操作を使用します。 会話をすぐに開始するつもりの場合は、[会話の開始](bot-framework-rest-direct-line-3-0-start-conversation.md)操作を使用します。

## <a id="refresh-token"></a> Direct Line トークンを更新する

Direct Line トークンは、有効期限が切れていない限り、無制限に更新できます。 期限が切れたトークンは更新できません。 Direct Line トークンを更新するには、次の要求を発行します。 

```http
POST https://directline.botframework.com/v3/directline/tokens/refresh
Authorization: Bearer TOKEN_TO_BE_REFRESHED
```

この要求の `Authorization` ヘッダーの **TOKEN_TO_BE_REFRESHED** を、更新する Direct Line トークンに置き換えます。

以下のスニペットは、トークン更新要求と応答の例を示しています。

### <a name="request"></a>Request

```http
POST https://directline.botframework.com/v3/directline/tokens/refresh
Authorization: Bearer CurR_XV9ZA.cwA.BKA.iaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0y8qbOF5xPGfiCpg4Fv0y8qqbOF5x8qbOF5xn
```

### <a name="response"></a>Response

要求が成功した場合、応答には、前のトークンと同じ会話で有効な、新しい `token` と、新しいトークンの有効期限が切れるまでの秒数を示す `expires_in` 値が含まれます。 新しいトークンを有効のままにするには、有効期限が切れる前に[トークンを更新する](#refresh-token)必要があります。

```http
HTTP/1.1 200 OK
[other headers]
```

```json
{
  "conversationId": "abc123",
  "token": "RCurR_XV9ZA.cwA.BKA.y8qbOF5xPGfiCpg4Fv0y8qqbOF5x8qbOF5xniaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0",
  "expires_in": 1800
}
```

## <a name="additional-resources"></a>その他のリソース

- [主要な概念](bot-framework-rest-direct-line-3-0-concepts.md)
- [ボットを Direct Line に接続する](../bot-service-channel-connect-directline.md)