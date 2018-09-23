---
title: メッセージへの入力ヒントの追加 | Microsoft Docs
description: Bot Connector サービスを使用してメッセージに入力ヒントを追加する方法について説明します。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ms.openlocfilehash: 66c6bc20013ff2de82e29af76e9c99898c8b13d9
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/18/2018
ms.locfileid: "39303217"
---
# <a name="add-input-hints-to-messages"></a>メッセージへの入力ヒントの追加
> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-add-input-hints.md)
> - [Node.js](../nodejs/bot-builder-nodejs-send-input-hints.md)
> - [REST](../rest-api/bot-framework-rest-connector-add-input-hints.md)

メッセージの入力ヒントを指定することにより、メッセージがクライアントに配信された後、ボットがユーザー入力を受け付けるか、期待するか、または無視するかを指示できます。 多くのチャネルでは、これによってクライアントが適宜、ユーザー入力コントロールの状態を設定できます。 たとえば、ボットがユーザーの入力を無視していることをメッセージの入力ヒントが示している場合、クライアントはマイクを閉じて入力ボックスを無効にし、ユーザーが入力を提供するのを防ぐことができます。

## <a name="accepting-input"></a>入力の受け付け

ボットが受動的に入力の準備ができているが、ユーザーからの応答を待っていないことを示すには、メッセージを表す [Activity][Activity] オブジェクト内で `inputHint` プロパティを **acceptingInput** に設定します。 多くのチャネルでは、これによってクライアントの入力ボックスが有効になり、マイクは閉じられますがユーザーはまだマイクにアクセスできます。 たとえば、ユーザーがマイクボタンを押し下げたままにすると、Cortana はマイクを開いてユーザーからの入力を受け付けます。 

次の例は、メッセージを送信し、ボットが入力を受け付けていることを指定する要求を示しています。 この要求の例で、`https://smba.trafficmanager.net/apis` はベース URI を示しています。ご利用のボットによって発行される要求に対するベース URI は、これとは異なる場合があります。 ベース URI の設定の詳細については、[API リファレンス](bot-framework-rest-connector-api-reference.md#base-uri)に関する記事をご覧ください。

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
        "name": "sender's name"
    },
    "conversation": {
        "id": "abcd1234",
        "name": "conversation's name"
   },
   "recipient": {
        "id": "1234abcd",
        "name": "recipient's name"
    },
    "text": "Here's a picture of the house I was telling you about.",
    "inputHint": "acceptingInput",
    "replyToId": "5d5cdc723"
}
```

## <a name="expecting-input"></a>入力の期待

ボットがユーザーからの応答を待っていることを示すには、メッセージを表す [Activity][Activity] オブジェクト内で `inputHint` プロパティを **expectingInput** に設定します。 多くのチャネルでは、これによってクライアントの入力ボックスが有効になり、マイクが開きます。 

次の例は、メッセージを送信し、ボットが入力を期待していることを指定する要求を示しています。 この要求の例で、`https://smba.trafficmanager.net/apis` はベース URI を示しています。ご利用のボットによって発行される要求に対するベース URI は、これとは異なる場合があります。 ベース URI の設定の詳細については、[API リファレンス](bot-framework-rest-connector-api-reference.md#base-uri)に関する記事をご覧ください。

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
        "name": "sender's name"
    },
    "conversation": {
        "id": "abcd1234",
        "name": "conversation's name"
   },
   "recipient": {
        "id": "1234abcd",
        "name": "recipient's name"
    },
    "text": "What is your favorite color?",
    "inputHint": "expectingInput",
    "replyToId": "5d5cdc723"
}
```

## <a name="ignoring-input"></a>入力の無視
 
ボットがユーザーから入力を受け取る準備ができていないことを示すには、メッセージを表す [Activity][Activity] オブジェクト内で `inputHint` プロパティを **ignoringInput** に設定します。 多くのチャネルでは、これによってクライアントの入力ボックスが無効になり、マイクが閉じられます。 

次の例は、メッセージを送信し、ボットが入力を無視していることを指定する要求を示しています。 この要求の例で、`https://smba.trafficmanager.net/apis` はベース URI を示しています。ご利用のボットによって発行される要求に対するベース URI は、これとは異なる場合があります。 ベース URI の設定の詳細については、[API リファレンス](bot-framework-rest-connector-api-reference.md#base-uri)に関する記事をご覧ください。

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
        "name": "sender's name"
    },
    "conversation": {
        "id": "abcd1234",
        "name": "conversation's name"
   },
   "recipient": {
        "id": "1234abcd",
        "name": "recipient's name"
    },
    "text": "Please hold while I perform the calculation.",
    "inputHint": "ignoringInput",
    "replyToId": "5d5cdc723"
}
```

## <a name="additional-resources"></a>その他のリソース

- [メッセージの作成](bot-framework-rest-connector-create-messages.md)
- [メッセージを送受信する](bot-framework-rest-connector-send-and-receive-messages.md)

[Activity]: bot-framework-rest-connector-api-reference.md#activity-object