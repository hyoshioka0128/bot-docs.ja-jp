---
title: メッセージを送受信する | Microsoft Docs
description: Bot Connector サービスを使用してメッセージを送受信する方法について説明します。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
ms.openlocfilehash: d694f20985b6bf2875fe28f599ff603512b87757
ms.sourcegitcommit: c200cc2db62dbb46c2a089fb76017cc55bdf26b0
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 08/27/2019
ms.locfileid: "70037344"
---
# <a name="send-and-receive-messages"></a>メッセージを送受信する

Bot Connector サービスを使用すると、ボットで Skype、電子メール、Slack などの複数のチャネル間で通信できます。 これにより、ボットからチャネルへ、またチャネルからボットへと[アクティビティ](bot-framework-rest-connector-activities.md)をリレーすることで、ボットとユーザー間の通信が容易になります。 すべてのアクティビティに、メッセージを作成したユーザーに関する情報、メッセージのコンテキスト、メッセージの受信者と共に、適切な送信先にメッセージをルーティングするために使用される情報が含まれます。 この記事では、Bot Connector サービスを使用して、チャネル上のボットとユーザーの間で **message** アクティビティを交換する方法について説明します。 

## <a id="create-reply"></a> メッセージに応答する

### <a name="create-a-reply"></a>応答を作成する 

ユーザーがボットにメッセージを送信すると、ボットは **message** 型の [Activity][] オブジェクトとしてメッセージを受け取ります。 ユーザーのメッセージへの応答を作成するには、新しい `Activity` オブジェクトを作成し、まず次のプロパティを設定します。

| プロパティ | 値 |
|----|----|
| conversation | ユーザーのメッセージの `conversation` プロパティのコンテンツにこのプロパティを設定します。 |
| from | ユーザーのメッセージの `recipient` プロパティのコンテンツにこのプロパティを設定します。 |
| locale | 指定した場合、ユーザーのメッセージの `locale` プロパティのコンテンツにこのプロパティを設定します。 |
| recipient | ユーザーのメッセージの `from` プロパティのコンテンツにこのプロパティを設定します。 |
| replyToId | ユーザーのメッセージの `id` プロパティのコンテンツにこのプロパティを設定します。 |
| type | このプロパティは **message** に設定します |

次に、ユーザーに伝える情報を指定するプロパティを設定します。 たとえば、メッセージに表示するテキストを指定するには `text` プロパティを設定します。ボットが話すテキストを指定するには `speak` プロパティを設定します。メディアの添付ファイルや、メッセージに含めるリッチ カードを指定するには `attachments` プロパティを設定します。 一般的に使用されるメッセージのプロパティについては、「[Create messages](bot-framework-rest-connector-create-messages.md)」(メッセージの作成) を参照してください。

### <a name="send-the-reply"></a>応答を送信する

受信アクティビティの `serviceUrl` プロパティを使用して、ボットがその応答の発行に使用する[ベース URI を指定](bot-framework-rest-connector-api-reference.md#base-uri)します。 

応答を送信するには、次の要求を発行します。 

```http
POST /v3/conversations/{conversationId}/activities/{activityId}
```

この要求 URI の **{conversationId}** を (応答) Activity 内の `conversation` オブジェクトの `id`プロパティの値で置き換え、 **{activityId}** を (応答) Activity 内の `replyToId` プロパティの値で置き換えます。 応答を表すために作成した [Activity][] オブジェクトに要求の本文を設定します。

次の例は、単純なテキストのみの応答をユーザーのメッセージに送信する要求を示しています。 この要求の例で、`https://smba.trafficmanager.net/apis` はベース URI を示しています。ご利用のボットによって発行される要求に対するベース URI は、これとは異なる場合があります。 ベース URI の設定の詳細については、[API リファレンス](bot-framework-rest-connector-api-reference.md#base-uri)に関する記事をご覧ください。

```http
POST https://smba.trafficmanager.net/apis/v3/conversations/abcd1234/activities/5d5cdc723 
Authorization: Bearer ACCESS_TOKEN 
Content-Type: application/json 
```

```json
{
    "type": "message",
    "from": {
        "id": "12345678",
        "name": "Pepper's News Feed"
    },
    "conversation": {
        "id": "abcd1234",
        "name": "Convo1"
   },
   "recipient": {
        "id": "1234abcd",
        "name": "SteveW"
    },
    "text": "My bot's reply",
    "replyToId": "5d5cdc723"
}
```

## <a id="send-message"></a> (非応答) メッセージを送信する

ボットが送信するメッセージの多くは、ユーザーから受信したメッセージへの応答に含まれます。 ただし、ボットが、ユーザーからのメッセージに対する直接の返信ではない会話にメッセージを送信する必要がある場合もあります。 たとえば、ボットから新しいトピックの会話を開始する場合や、会話の最後にさようならのメッセージを送信する場合があります。 

ユーザーからのメッセージに対する直接の返信ではない会話にメッセージを送信するには、次の要求を発行します。 

```http
POST /v3/conversations/{conversationId}/activities
```

この要求 URI では、 **{conversationId}** を会話の ID で置き換えます。 
    
応答を表すために作成する [Activity][] オブジェクトに要求の本文を設定します。

> [!NOTE]
> Bot Framework では、ボットで送信できるメッセージ数の制限を指定していません。 しかし、ほとんどのチャネルでは、ボットが短時間で多数のメッセージを送信しないように、調整制限を強制します。 さらに、ボットで立て続けに複数のメッセージを送信する場合は、チャネルでは適切な順序でメッセージがレンダリングされない可能性があります。

## <a name="start-a-conversation"></a>会話を開始する

ボットで 1 人または複数のユーザーと会話を開始する必要がある場合があります。 チャネル上でユーザーとの会話を開始するには、ボットがそのアカウント情報と、そのチャネル上のユーザーのアカウント情報を知っている必要があります。 

> [!TIP]
> ボットが今後ユーザーとの会話を開始する必要がある場合は、ユーザーのアカウント情報、ユーザーの設定やロケールなどの関連情報、サービス URL をキャッシュします (そして会話の開始要求のベース URI として使用します)。 

会話を開始するには、次の要求を発行します。 

```http
POST /v3/conversations
```

ボットのアカウント情報と、会話に含めるユーザーのアカウント情報を指定する [ConversationParameters][] オブジェクトに、要求の本文を設定します。

> [!NOTE]
> すべてのチャネルが、グループ会話をサポートするわけではありません。 チャネルがグループの会話をサポートしているかどうかを判断し、チャネルが会話で許可している参加者の最大数を特定するには、チャネルのドキュメントを参照してください。

次の例は、会話を開始する要求を示しています。 この要求の例で、`https://smba.trafficmanager.net/apis` はベース URI を示しています。ご利用のボットによって発行される要求に対するベース URI は、これとは異なる場合があります。 ベース URI の設定の詳細については、[API リファレンス](bot-framework-rest-connector-api-reference.md#base-uri)に関する記事をご覧ください。

```http
POST https://smba.trafficmanager.net/apis/v3/conversations 
Authorization: Bearer ACCESS_TOKEN
Content-Type: application/json
```

```json
{
    "bot": {
        "id": "12345678",
        "name": "bot's name"
    },
    "isGroup": false,
    "members": [
        {
            "id": "1234abcd",
            "name": "recipient's name"
        }
    ],
    "topicName": "News Alert"
}
```

指定されたユーザーとの会話が確立されている場合、応答には会話を識別する ID が含まれます。 

```json
{
    "id": "abcd1234"
}
```

ボットはその会話 ID を使用して、会話内のユーザーに[メッセージを送信](#send-message)します。

## <a name="additional-resources"></a>その他のリソース

- [アクティビティの概要](bot-framework-rest-connector-activities.md)
- [メッセージの作成](bot-framework-rest-connector-create-messages.md)
- [Bot Framework のアクティビティ スキーマ](https://aka.ms/botSpecs-activitySchema)

[Activity]: bot-framework-rest-connector-api-reference.md#activity-object
[ConversationParameters]: bot-framework-rest-connector-api-reference.md#conversationparameters-object
