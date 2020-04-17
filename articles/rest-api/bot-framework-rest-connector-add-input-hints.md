---
title: メッセージへの入力ヒントの追加 - Bot Service
description: Bot Connector サービスを使用してメッセージに入力ヒントを追加する方法について説明します。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
ms.openlocfilehash: 358f5cc7714c2d8803adbf40b60cc4584d3fe506
ms.sourcegitcommit: 9d77f3aff9521d819e88efd0fbd19d469b9919e7
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/16/2020
ms.locfileid: "77520031"
---
# <a name="add-input-hints-to-messages"></a>メッセージへの入力ヒントの追加
> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-add-input-hints.md)
> - [Node.js](../nodejs/bot-builder-nodejs-send-input-hints.md)
> - [REST](../rest-api/bot-framework-rest-connector-add-input-hints.md)

メッセージの入力ヒントを指定すれば、メッセージがクライアントに配信された後、ボットがユーザー入力を受け付けるか、期待するか、無視するかを示すことができます。 このフィールドをサポートするチャンネルでは、これによってクライアントが適宜、ユーザー入力コントロールの状態を設定できます。 たとえば、ボットがユーザーの入力を無視していることをメッセージの入力ヒントが示している場合、クライアントはユーザーが入力を提供できないよう、マイクを閉じて入力ボックスを無効にすることができます。

## <a name="accepting-input"></a>入力の受け付け

ボットが受動的に入力を受け取る準備ができているが、ユーザーからの応答を待っていないことを示すには、メッセージを表す [Activity][] オブジェクト内で `inputHint` プロパティを **acceptingInput** に設定します。 多くのチャネルでは、これによってクライアントの入力ボックスが有効になり、マイクは閉じられますがユーザーはまだマイクにアクセスできます。 たとえば、ユーザーがマイク ボタンを押し下げたままにすると、Cortana はマイクを開いてユーザーからの入力を受け付けます。 

次の例は、メッセージを送信し、ボットが入力を受け付けていることを指定する要求を示しています。 この要求の例で、Direct Line はベース URI を示しています。ご利用のボットによって発行される要求に対するベース URI は、これとは異なる場合があります。 ベース URI の設定の詳細については、[API リファレンス](bot-framework-rest-connector-api-reference.md#base-uri)に関する記事をご覧ください。

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

ボットがユーザーからの応答をアクティブに待っていることを示すには、メッセージを表す [Activity][] オブジェクト内で `inputHint` プロパティを **expectingInput** に設定します。 それをサポートするチャンネルでは、これによってクライアントの入力ボックスが有効になり、マイクが使えるようになります。 

次の例は、メッセージを送信し、ボットが入力を期待していることを指定する要求を示しています。 この要求の例で、Direct Line はベース URI を示しています。ご利用のボットによって発行される要求に対するベース URI は、これとは異なる場合があります。 ベース URI の設定の詳細については、[API リファレンス](bot-framework-rest-connector-api-reference.md#base-uri)に関する記事をご覧ください。

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
 
ボットがユーザーから入力を受け取る準備ができていないことを示すには、メッセージを表す [Activity][] オブジェクト内で `inputHint` プロパティを **ignoringInput** に設定します。 それをサポートするチャンネルでは、これによってクライアントの入力ボックスが無効になり、マイクが切れます。 

次の例は、メッセージを送信し、ボットが入力を無視していることを指定する要求を示しています。 この要求の例で、Direct Line はベース URI を示しています。ご利用のボットによって発行される要求に対するベース URI は、これとは異なる場合があります。 ベース URI の設定の詳細については、[API リファレンス](bot-framework-rest-connector-api-reference.md#base-uri)に関する記事をご覧ください。

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
