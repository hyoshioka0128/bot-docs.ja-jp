---
title: メッセージに音声を追加する - Bot Service
description: Bot Connector サービスを使用してメッセージに音声を追加する方法について説明します。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 02/20/2020
ms.openlocfilehash: f5bf6cf254b4c095ce9f49619bb39ef5b48fa5ed
ms.sourcegitcommit: 9d77f3aff9521d819e88efd0fbd19d469b9919e7
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/16/2020
ms.locfileid: "77519931"
---
# <a name="add-speech-to-messages"></a>メッセージに音声を追加する
> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-text-to-speech.md)
> - [Node.js](../nodejs/bot-builder-nodejs-text-to-speech.md)
> - [REST](../rest-api/bot-framework-rest-connector-text-to-speech.md)

Cortana などの音声対応チャネルのボットを作成している場合は、ボットが読み上げるテキストを指定するメッセージを構成できます。 また、[入力ヒント](bot-framework-rest-connector-add-input-hints.md)を指定して、ボットがユーザー入力を受け入れるか、期待するか、無視するかを示し、クライアントのマイクの状態に影響を与えることを試すこともできます。 

クライアント アプリケーションが [Direct Line Speech チャンネル](https://docs.microsoft.com/azure/bot-service/bot-service-channel-connect-directlinespeech?view=azure-bot-service-4.0)を介してボットと通信できるように、そのボットを構成できます。

## <a name="specify-text-to-be-spoken-by-your-bot"></a>ボットが読み上げるテキストを指定する

音声対応チャネル上でボットが読み上げるテキストを指定するには、メッセージを表す[Activity][Activity] オブジェクト内の `speak` プロパティを設定します。 `speak` プロパティは、プレーンテキスト文字列か、または声、速さ、音量、発音、ピッチなどのボットの音声のさまざまな特性を制御できる XML ベースのマークアップ言語である<a href="https://docs.microsoft.com/azure/cognitive-services/speech-service/speech-synthesis-markup" target="_blank">音声合成マークアップ言語 (SSML)</a> として書式設定された文字列のどちらかに設定できます。 チャンネルがサポートされていない場合、メッセージはテキストとして配信されます。


次の要求では、表示されるテキストと読み上げられるテキストを指定し、ボットが[ユーザー入力を想定している](bot-framework-rest-connector-add-input-hints.md)ことを示すメッセージを送信します。 ここでは、<a href="https://docs.microsoft.com/azure/cognitive-services/speech-service/speech-synthesis-markup" target="_blank">SSML</a> 形式を使用した `speak` プロパティを指定して、単語 "sure" を適度な強調の量で読み上げる必要があることを示しています。 この要求の例で、Direct Line はベース URI を示しています。ご利用のボットによって発行される要求に対するベース URI は、これとは異なる場合があります。 ベース URI の設定の詳細については、[API リファレンス](bot-framework-rest-connector-api-reference.md#base-uri)に関する記事をご覧ください。

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
    "speak": "<speak version=\"1.0\" xmlns=\"http://www.w3.org/2001/10/synthesis\" xml:lang=\"en-US\">Are you <emphasis level=\"moderate\">sure</emphasis> that you want to cancel this transaction?</speak>",
    "inputHint": "expectingInput",
    "replyToId": "5d5cdc723"
}
```

## <a name="input-hints"></a>入力ヒント

音声対応チャンネルでメッセージを送信する場合、入力ヒントを含めて、ボットがユーザー入力を受け入れているか、期待しているか、無視しているかを示すことで、クライアントのマイクの意図した状態を表すことができます。 詳細については、「[メッセージへの入力ヒントの追加](bot-framework-rest-connector-add-input-hints.md)」を参照してください。

## <a name="additional-resources"></a>その他のリソース

- [メッセージの作成](bot-framework-rest-connector-create-messages.md)
- [メッセージを送受信する](bot-framework-rest-connector-send-and-receive-messages.md)
- [メッセージへの入力ヒントの追加](bot-framework-rest-connector-add-input-hints.md)
- <a href="https://docs.microsoft.com/azure/cognitive-services/speech-service/speech-synthesis-markup" target="_blank">音声合成マークアップ言語 (SSML)</a>

[Activity]: bot-framework-rest-connector-api-reference.md#activity-object
