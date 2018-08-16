---
title: メッセージに音声を追加する | Microsoft Docs
description: Bot Connector サービスを使用してメッセージに音声を追加する方法について説明します。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ms.openlocfilehash: 91385fa3e8ae1410679ca5274e40db7fe38bafea
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/18/2018
ms.locfileid: "39304220"
---
# <a name="add-speech-to-messages"></a>メッセージに音声を追加する
> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-text-to-speech.md)
> - [Node.js](../nodejs/bot-builder-nodejs-text-to-speech.md)
> - [REST](../rest-api/bot-framework-rest-connector-text-to-speech.md)

Cortana などの音声対応チャネルのボットを構築している場合は、ボットが読み上げるテキストを指定するメッセージを構成できます。 また、ボットがユーザー入力を受け付けるか、想定しているか、または無視するかどうかを示す[入力ヒント](bot-framework-rest-connector-add-input-hints.md)を指定することによって、クライアントのマイクの状態に影響を与えようと試みることもできます。

## <a name="specify-text-to-be-spoken-by-your-bot"></a>ボットが読み上げるテキストを指定する

音声対応チャネル上でボットが読み上げるテキストを指定するには、メッセージを表す[アクティビティ][Activity] オブジェクト内の `speak` プロパティを設定します。 `speak` プロパティは、プレーンテキスト文字列か、または声、速さ、音量、発音、ピッチなどのボットの音声のさまざまな特性を制御できる XML ベースのマークアップ言語である<a href="https://msdn.microsoft.com/en-us/library/hh378377(v=office.14).aspx" target="_blank">音声合成マークアップ言語 (SSML)</a> として書式設定された文字列のどちらかに設定できます。 

次の要求では、表示されるテキストと読み上げられるテキストを指定し、ボットが[ユーザー入力を想定している](bot-framework-rest-connector-add-input-hints.md)ことを示すメッセージを送信します。 ここでは、<a href="https://msdn.microsoft.com/en-us/library/hh378377(v=office.14).aspx" target="_blank">SSML</a> 形式を使用した `speak` プロパティを指定して、単語 "sure" を適度な強調の量で読み上げる必要があることを示しています。 この要求の例で、`https://smba.trafficmanager.net/apis` はベース URI を示しています。ご利用のボットによって発行される要求に対するベース URI は、これとは異なる場合があります。 ベース URI の設定の詳細については、[API リファレンス](bot-framework-rest-connector-api-reference.md#base-uri)に関する記事をご覧ください。

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
    "text": "Are you sure that you want to cancel this transaction?",
    "speak": "Are you <emphasis level='moderate'>sure</emphasis> that you want to cancel this transaction?",
    "inputHint": "expectingInput",
    "replyToId": "5d5cdc723"
}
```

## <a name="input-hints"></a>入力ヒント

音声対応チャネル上でメッセージを送信する場合は、ボットがユーザー入力を受け付けるか、想定しているか、または無視するかどうかを示す入力ヒントも含めることによって、クライアントのマイクの状態に影響を与えようと試みることができます。 詳細については、「[メッセージへの入力ヒントの追加](bot-framework-rest-connector-add-input-hints.md)」を参照してください。

## <a name="additional-resources"></a>その他のリソース

- [メッセージの作成](bot-framework-rest-connector-create-messages.md)
- [メッセージを送受信する](bot-framework-rest-connector-send-and-receive-messages.md)
- [メッセージへの入力ヒントの追加](bot-framework-rest-connector-add-input-hints.md)
- <a href="https://msdn.microsoft.com/en-us/library/hh378377(v=office.14).aspx" target="_blank">音声合成マークアップ言語 (SSML)</a>

[Activity]: bot-framework-rest-connector-api-reference.md#activity-object
