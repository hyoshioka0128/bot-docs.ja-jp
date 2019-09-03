---
title: メッセージへのメディア添付ファイルの追加 | Microsoft Docs
description: Bot Connector サービスを使用してメッセージにメディア添付ファイルを追加する方法について説明します。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 10/25/2018
ms.openlocfilehash: be56700664e7626c247bb77899dc89f3cac32469
ms.sourcegitcommit: c200cc2db62dbb46c2a089fb76017cc55bdf26b0
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 08/27/2019
ms.locfileid: "70037214"
---
# <a name="add-media-attachments-to-messages"></a>メッセージへのメディア添付ファイルの追加
> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-add-media-attachments.md)
> - [Node.JS](../nodejs/bot-builder-nodejs-send-receive-attachments.md)
> - [REST](../rest-api/bot-framework-rest-connector-add-media-attachments.md)

通常、ボットとチャネルが交換するのはテキスト文字列ですが、添付ファイルの交換もサポートされる一部のチャネルでは、ボットはよりリッチなメッセージをユーザーに送信できます。 たとえば、ボットは、メディア添付ファイル (画像、動画、オーディオ、ファイルなど) と[リッチ カード](bot-framework-rest-connector-add-rich-cards.md)を送信できます。 この記事では、Bot Connector サービスを使用してメッセージにメディア添付ファイルを追加する方法について説明します。

> [!TIP]
> チャネルでサポートされている添付ファイルの種類と数、およびチャネルによって添付ファイルがどのようにレンダリングされるかを確認するには、[Channel Inspector][ChannelInspector] に関するページを参照してください。

## <a name="add-a-media-attachment"></a>メディア添付ファイルの追加  

メディア添付ファイルをメッセージに追加するには、[Attachment][] オブジェクトを作成し、`name` プロパティを設定します。さらに、`contentUrl` プロパティをメディア ファイルの URL に設定し、`contentType` プロパティを適切なメディアの種類 (たとえば、**image/png**、**audio/wav**、**video/mp4**) に設定します。 次に、メッセージを表す [Activity][] オブジェクト内で、`Attachment` オブジェクトを `attachments` 配列内に指定します。

次の例に、テキストと 1 つの画像添付ファイルを含むメッセージを送信する要求を示します。 この要求の例で、`https://smba.trafficmanager.net/apis` はベース URI を示しています。ご利用のボットによって発行される要求に対するベース URI は、これとは異なる場合があります。 ベース URI の設定の詳細については、「[API Reference (API リファレンス)](bot-framework-rest-connector-api-reference.md#base-uri)」を参照してください。

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
    "text": "Here's a picture of the duck I was telling you about.",
    "attachments": [
        {
            "contentType": "image/png",
            "contentUrl": "https://aka.ms/DuckOnARock",
            "name": "duck-on-a-rock.jpg"
        }
    ],
    "replyToId": "5d5cdc723"
}
```

画像のインライン バイナリをサポートするチャネルの場合、`Attachment` の `contentUrl` プロパティを画像の base64 バイナリに設定できます (たとえば、**data:image/png;base64,iVBORw0KGgo...** )。 このチャネルでは、メッセージのテキスト文字列の横に画像または画像の URL が表示されます。

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
    "text": "Here's a picture of the duck I was telling you about.",
    "attachments": [
        {
            "contentType": "image/png",
            "contentUrl": "data:image/png;base64,iVBORw0KGgo…",
            "name": "duck-on-a-rock.jpg"
        }
    ],
    "replyToId": "5d5cdc723"
}
```

上記の画像ファイル用の手順と同じ手順を使用して、動画ファイルまたはオーディオ ファイルをメッセージに添付することができます。 チャネルによっては、動画やオーディオがインラインで再生されたり、リンクとして表示されたりすることがあります。

> [!NOTE] 
> ボットがメディア添付ファイルを含むメッセージを受け取ることもあります。
> たとえば、分析する写真や保存するドキュメントをユーザーがアップロードすることがチャネルによって許可されている場合、ボットが受け取るメッセージに添付ファイルが含まれる場合があります。

## <a name="add-an-audiocard-attachment"></a>AudioCard 添付ファイルの追加

[AudioCard][] または [VideoCard][] 添付ファイルを追加する方法は、メディア添付ファイルを追加する方法と同じです。 たとえば、次の JSON は、メディア添付ファイルにオーディオ カードを追加する方法を示しています。

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
    "attachments": [
    {
      "contentType": "application/vnd.microsoft.card.audio",
      "content": {
        "title": "Allegro in C Major",
        "subtitle": "Allegro Duet",
        "text": "No Image, No Buttons, Autoloop, Autostart, Sharable",
        "duration": "PT2M55S",
        "media": [
          {
            "url": "https://contoso.com/media/AllegrofromDuetinCMajor.mp3"
          }
        ],
        "shareable": true,
        "autoloop": true,
        "autostart": true,
        "value": {
            // Supplementary parameter for this card
        }
      }
    }],
    "replyToId": "5d5cdc723"
}
```

チャネルがこの添付ファイルを受け取ると、オーディオ ファイルの再生が開始されます。 たとえば、 **[一時停止]** ボタンをクリックしてユーザーがオーディオとやりとりすると、チャネルは次のような JSON でコールバックをボットに送信します。

```json
{
    ...
    "type": "event",
    "name": "media/pause",
    "value": {
        "url": // URL for media
        "cardValue": {
            // Supplementary parameter for this card
        }
    }
}
```

`activity.name` フィールドにはメディア イベント名 **media/pause** が表示されます。 すべてのメディア イベント名の一覧については、次の表を参照してください。

| Event | 説明 |
| ---- | ---- |
| **media/next** | クライアントが次のメディアまでスキップしました |
| **media/pause** | クライアントがメディアの再生を一時停止しました |
| **media/play** | クライアントがメディアの再生を開始しました |
| **media/previous** | クライアントが前のメディアまでスキップしました |
| **media/resume** | クライアントがメディアの再生を再開しました |
| **media/stop** | クライアントがメディアの再生を停止しました |

## <a name="additional-resources"></a>その他のリソース

- [メッセージの作成](bot-framework-rest-connector-create-messages.md)
- [メッセージを送受信する](bot-framework-rest-connector-send-and-receive-messages.md)
- [メッセージへのリッチ カードの追加](bot-framework-rest-connector-add-rich-cards.md)
- [Bot Framework のアクティビティ スキーマ](https://aka.ms/botSpecs-activitySchema)
- [Bot Framework のカード スキーマ](https://aka.ms/botSpecs-cardSchema)

[Activity]: bot-framework-rest-connector-api-reference.md#activity-object
[Attachment]: bot-framework-rest-connector-api-reference.md#attachment-object
[AudioCard]: bot-framework-rest-connector-api-reference.md#audiocard-object
[VideoCard]: bot-framework-rest-connector-api-reference.md#videocard-object
