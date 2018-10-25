---
title: Bot Connector サービスでのボットの作成 | Microsoft Docs
description: Bot Connector サービスを使用してボットを作成します。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
ms.openlocfilehash: 57babac9594118c12805ff9023cf7086e526a273
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/24/2018
ms.locfileid: "49997941"
---
# <a name="create-a-bot-with-the-bot-connector-service"></a>Bot Connector サービスでのボットの作成
> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-quickstart.md)
> - [Node.js](../nodejs/bot-builder-nodejs-quickstart.md)
> - [Bot Service](../bot-service-quickstart.md)
> - [REST](../rest-api/bot-framework-rest-connector-quickstart.md)

Bot Connector サービスを使用すると、お使いのボットで、業界標準の HTTPS 経由 REST および JSON を使用して、<a href="https://dev.botframework.com/" target="_blank">Bot Framework Portal</a> で構成されたチャネルとメッセージ交換できます。 このチュートリアルでは、Bot Framework からアクセス トークンを取得し、Bot Connector サービスを使用してユーザーとメッセージを交換する手順について説明します。

## <a id="get-token"></a> アクセス トークンを取得する

> [!IMPORTANT]
> Bot Framework に[お使いのボットを登録](../bot-service-quickstart-registration.md)して、そのアプリ ID とパスワードを取得する必要があります (この操作をまだ行っていない場合)。 アクセス トークンを取得するには、ボットのアプリ ID とパスワードが必要になります。

Bot Connector サービスと通信するには、次の形式を使用して、各 API 要求の `Authorization` ヘッダーでアクセス トークンを指定する必要があります。 

```http
Authorization: Bearer ACCESS_TOKEN
```

お使いのボット用のアクセス トークンを取得するには、API 要求を発行します。

### <a name="request"></a>Request

Bot Connector サービスに対する要求の認証に使用できるアクセス トークンを要求するには、次を要求を発行します。**MICROSOFT-APP-ID** および **MICROSOFT-APP-PASSWORD** は、ご自身のボットを Bot Framework に[登録](../bot-service-quickstart-registration.md)したときに取得したアプリ ID とパスワードに置き換えてください。

```http
POST https://login.microsoftonline.com/botframework.com/oauth2/v2.0/token
Host: login.microsoftonline.com
Content-Type: application/x-www-form-urlencoded

grant_type=client_credentials&client_id=MICROSOFT-APP-ID&client_secret=MICROSOFT-APP-PASSWORD&scope=https%3A%2F%2Fapi.botframework.com%2F.default
```

### <a name="response"></a>Response

要求が成功すると、アクセス トークンとその有効期限に関する情報が示された HTTP 200 応答が返されます。 

```json
{
    "token_type":"Bearer",
    "expires_in":3600,
    "ext_expires_in":3600,
    "access_token":"eyJhbGciOiJIUzI1Ni..."
}
```

> [!TIP]
> Bot Connector サービスでの認証の詳細については、「[Authentication (認証)](bot-framework-rest-connector-authentication.md)」を参照してください。

## <a name="exchange-messages-with-the-user"></a>ユーザーとメッセージを交換する

会話とは、ユーザーとご自身のボットの間で交換された一連のメッセージのことです。 

### <a name="receive-a-message-from-the-user"></a>ユーザーからメッセージを受信する

ユーザーがメッセージを送信すると、お使いのボットを[登録](../bot-service-quickstart-registration.md)したときに指定したエンドポイントに、Bot Framework Connector によって要求の POST が実行されます。 要求の本文は [Activity][Activity] オブジェクトです。 次の例は、ユーザーが簡単なメッセージをボットに送信したときに、ボット側で受信する要求本文を示しています。 

```json
{
    "type": "message",
    "id": "bf3cc9a2f5de...",
    "timestamp": "2016-10-19T20:17:52.2891902Z",
    "serviceUrl": "https://smba.trafficmanager.net/apis",
    "channelId": "channel's name/id",
    "from": {
        "id": "1234abcd",
        "name": "user's name"
    },
    "conversation": {
        "id": "abcd1234",
        "name": "conversation's name"
    },
    "recipient": {
        "id": "12345678",
        "name": "bot's name"
    },
    "text": "Haircut on Saturday"
}
```

### <a name="reply-to-the-users-message"></a>ユーザーのメッセージに返信する

お使いのボットのエンドポイントで、ユーザーからのメッセージを表す `POST` 要求 (つまり `type` = **メッセージ**) を受信したら、その要求の情報を使用して、ご自身の応答用の [Activity][Activity] オブジェクトを作成します。

1. **conversation** プロパティを、ユーザーのメッセージの **conversation** プロパティのコンテンツに設定します。
2. **from** プロパティを、ユーザーのメッセージの **recipient** プロパティのコンテンツに設定します。
3. **recipient** プロパティを、ユーザーのメッセージの **from** プロパティのコンテンツに設定します。
4. 必要に応じて、**text** プロパティと **attachments** プロパティを設定します。

着信要求の `serviceUrl` プロパティを使用して、その応答を発行するときに、ご自身のボットによって使用される[ベース URI を特定](bot-framework-rest-connector-api-reference.md#base-uri)します。 

応答を送信するには、次の例に示すように、`/v3/conversations/{conversationId}/activities/{activityId}` に対して、ご自身の [Activity][Activity] オブジェクトの `POST` を実行します。 この要求の本文は、ユーザーに使用可能な予定の時刻を選択するように求める [Activity][Activity] オブジェクトです。

```http
POST https://smba.trafficmanager.net/apis/v3/conversations/abcd1234/activities/bf3cc9a2f5de... 
Authorization: Bearer eyJhbGciOiJIUzI1Ni...
Content-Type: application/json
```

```json
{
    "type": "message",
    "from": {
        "id": "12345678",
        "name": "bot's name"
    },
    "conversation": {
        "id": "abcd1234",
        "name": "conversation's name"
   },
   "recipient": {
        "id": "1234abcd",
        "name": "user's name"
    },
    "text": "I have these times available:",
    "replyToId": "bf3cc9a2f5de..."
}
```

この要求の例で、`https://smba.trafficmanager.net/apis` はベース URI を示しています。ご利用のボットによって発行される要求に対するベース URI は、これとは異なる場合があります。 ベース URI の設定の詳細については、[API リファレンス](bot-framework-rest-connector-api-reference.md#base-uri)に関する記事をご覧ください。 

> [!IMPORTANT]
> この例で示すように、送信する各 API 要求の `Authorization` ヘッダーには、**ベアラー**というワードに続いて、[Bot Framework から取得した](#get-token)アクセス トークンが含まれている必要があります。

別のメッセージを送信して、使用可能な予定の時刻をユーザーがクリックによって選択できるようにするには、同じエンドポイントに対して他の要求の `POST` を実行します。

```http
POST https://smba.trafficmanager.net/apis/v3/conversations/abcd1234/activities/bf3cc9a2f5de... 
Authorization: Bearer eyJhbGciOiJIUzI1Ni...
Content-Type: application/json
```

```json
{
    "type": "message",
    "from": {
        "id": "12345678",
        "name": "bot's name"
    },
    "conversation": {
        "id": "abcd1234",
        "name": "conversation's name"
   },
   "recipient": {
        "id": "1234abcd",
        "name": "user's name"
    },
    "attachmentLayout": "list",
    "attachments": [
      {
        "contentType": "application/vnd.microsoft.card.thumbnail",
        "content": {
          "buttons": [
            {
              "type": "imBack",
              "title": "10:30",
              "value": "10:30"
            },
            {
              "type": "imBack",
              "title": "11:30",
              "value": "11:30"
            },
            {
              "type": "openUrl",
              "title": "See more",
              "value": "http://www.contososalon.com/scheduling"
            }
          ]
        }
      }
    ],
    "replyToId": "bf3cc9a2f5de..."
}
```   

## <a name="next-steps"></a>次の手順

このチュートリアルでは、Bot Framework からアクセス トークンを取得し、Bot Connector サービスを使用してユーザーとメッセージを交換しました。 [Bot Framework エミュレーター](../bot-service-debug-emulator.md)を使用すると、お使いのボットのテストおよびデバッグを行うことができます。 ご自身のボットを他のユーザーと共有したい場合は、そのボットが 1 つまたは複数のチャネルで実行されるように[構成](../bot-service-manage-channels.md)する必要があります。


[Activity]: bot-framework-rest-connector-api-reference.md#activity-object