---
title: メッセージにリッチ カード添付ファイルを追加する - Bot Service
description: Bot Connector サービスを使用して、メッセージにリッチ カードを追加する方法について説明します。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
ms.openlocfilehash: e51d4bcc7059e1130932ca6a8b956dd3b097ef8c
ms.sourcegitcommit: 4e1af50bd46debfdf9dcbab9a5d1b1633b541e27
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/25/2020
ms.locfileid: "76752904"
---
# <a name="add-rich-card-attachments-to-messages"></a>メッセージにリッチ カード添付ファイルを追加する

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-add-rich-card-attachments.md)
> - [Node.js](../nodejs/bot-builder-nodejs-send-rich-cards.md)
> - [REST](../rest-api/bot-framework-rest-connector-add-rich-cards.md)

通常、ボットとチャネルが交換するのはテキスト文字列ですが、添付ファイルの交換もサポートされる一部のチャネルでは、ボットはよりリッチなメッセージをユーザーに送信できます。 たとえば、ボットは、リッチ カードとメディア添付ファイル (画像、動画、音声、ファイルなど) を送信できます。 この記事では、Bot Connector サービスを使用してメッセージにリッチ カード添付ファイルを追加する方法について説明します。

> [!NOTE]
> メッセージにメディア添付ファイルを追加する方法については、「[Add media attachments to messages](bot-framework-rest-connector-add-media-attachments.md)」 (メッセージにメディア添付ファイルを追加する) を参照してください。

## <a name="types-of-rich-cards"></a>リッチ カードの種類

リッチ カードは、タイトル、説明、リンク、およびイメージから構成されます。
メッセージには、リスト形式またはカルーセル形式で表示される複数のリッチ カードを含めることができます。
Bot Framework では、現在 8 種類のリッチ カードがサポートされています。

| カードの種類 | [説明] |
|----|----|
| [AdaptiveCard](/adaptive-cards/get-started/bots) | テキスト、音声、画像、ボタン、および入力フィールドの任意の組み合わせを含めることができる、カスタマイズ可能なカード。 [チャネルごとのサポート](/adaptive-cards/get-started/bots#channel-status)に関するページをご覧ください。 |
| [AnimationCard][] | アニメーション GIF または短い動画を再生できるカード。 |
| [AudioCard][] | オーディオ ファイルを再生できるカード。 |
| [HeroCard][] | 通常 1 つの大きなイメージ、1 つまたは複数のボタン、およびテキストが含まれるカード。 |
| [ThumbnailCard][] | 通常 1 つのサムネイル イメージ、1 つまたは複数のボタン、およびテキストが含まれるカード。 |
| [ReceiptCard][] | ボットからユーザーに領収書を提供できるようにするカード。 通常は、領収書に含める項目の一覧、税金と合計の情報、およびその他のテキストが含まれます。 |
| [SignInCard][] | ボットでユーザーのサインインを要求できるようにするカード。 通常は、テキストと、ユーザーがクリックしてサインイン プロセスを開始できる 1 つ以上のボタンが含まれます。 |
| [VideoCard][] | 動画を再生できるカード。 |

[!INCLUDE [Channel Inspector intro](~/includes/snippet-channel-inspector.md)]

## <a name="process-events-within-rich-cards"></a>リッチ カード内のイベントを処理する

リッチ カード内のイベントを処理するには、[CardAction][] オブジェクトを使用して、ユーザーがボタンをクリックするか、カードのセクションをタップしたときのアクションを指定します。 各 `CardAction` オブジェクトには、次のプロパティが含まれています。

| プロパティ | Type | [説明] |
|----|----|----|
| channelData | string | このアクションに関連付けられているチャネル固有のデータ |
| displayText | string | ボタンがクリックされた場合にチャット フィードに表示するテキスト |
| text | string | アクションのテキスト |
| type | string | アクションの種類 (下の表に示されている値のいずれか) |
| title | string | ボタンのタイトル |
| image | string | ボタン用のイメージ URL |
| value | string | 指定された種類のアクションを実行するために必要な値 |

> [!NOTE]
> アダプティブ カード内のボタンは、`CardAction` オブジェクトではなく、アダプティブ カードによって定義されているスキーマを使用して作成されます。
> アダプティブ カードにボタンを追加する方法の例については、「[Add an Adaptive Card to a message](#add-an-adaptive-card-to-a-message)」(メッセージにアダプティブ カードを追加する) を参照してください。

次の表では、`CardAction` オブジェクトの `type` プロパティの有効値と、各種類の `value` プロパティの期待されるコンテンツを説明します。

| 型 | value |
|----|----|
| openUrl | 組み込みのブラウザーで開かれる URL |
| imBack | (ボタンをクリックまたはカードをタップしたユーザーから) ボットに送信されるメッセージのテキスト。 会話の参加者すべてが、会話をホストしているクライアント アプリケーションを介して、このメッセージ (ユーザーからボットへの) を表示することができます。 |
| postBack | (ボタンをクリックまたはカードをタップしたユーザーから) ボットに送信されるメッセージのテキスト。 クライアント アプリケーションによっては、このテキストがメッセージ フィードに表示される場合があります。そこでは、会話の参加者のすべてにそのテキストが表示されます。 |
| call | 電話の呼び出し先であり、**tel:123123123123** の形式となります。 |
| playAudio | 再生されるオーディオの URL |
| playVideo | 再生されるビデオの URL |
| showImage | 表示されるイメージの URL |
| downloadFile | ダウンロードされるファイルの URL |
| signin | 開始される OAuth フローの URL |

## <a name="add-a-hero-card-to-a-message"></a>メッセージにヒーロー カードを追加する

メッセージにリッチ カード添付ファイルを追加するには、最初に、メッセージに追加する[カードの種類](#types-of-rich-cards)に対応するオブジェクトを作成します。 次に、[Attachment][] オブジェクトを作成し、その `contentType` プロパティをカードのメディアの種類に設定し、その `content` プロパティをカードを表すように作成したオブジェクトに設定します。 メッセージの `attachments` 配列内に `Attachment` オブジェクトを指定します。

> [!TIP]
> 通常、リッチ カード添付ファイルを含むメッセージには `text` を指定しません。

一部のチャネルでは、メッセージ内の `attachments` 配列に複数のリッチ カードを追加できます。 この機能は、複数のオプションをユーザーに提供するシナリオで役立ちます。 たとえば、ユーザーがホテルの部屋を予約できるボットの場合は、利用可能な部屋の種類を示すリッチ カードの一覧をユーザーに表示できます。 各カードには、部屋の種類に対応する写真とアメニティの一覧を含めることができ、ユーザーは、カードをタップするかボタンをクリックすることで部屋の種類を選択できます。

> [!TIP]
> 複数のリッチ カードをリスト形式で表示するには、[Activity][] オブジェクトの `attachmentLayout` プロパティを "list" に設定します。
> 複数のリッチ カードをカルーセル形式で表示するには、`Activity` オブジェクトの `attachmentLayout` プロパティを "carousel" に設定します。
> カルーセル形式がチャネルでサポートされていない場合、`attachmentLayout` プロパティに "carousel" が指定されていたとしても、リッチ カードはリスト形式で表示されます。

次の例に、1 つのヒーロー カード添付ファイルを含むメッセージを送信する要求を示します。 この要求の例で、`https://smba.trafficmanager.net/apis` はベース URI を示しています。ご利用のボットによって発行される要求に対するベース URI は、これとは異なる場合があります。 ベース URI の設定の詳細については、[API リファレンス](bot-framework-rest-connector-api-reference.md#base-uri)に関する記事をご覧ください。

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
    "attachments": [
        {
            "contentType": "application/vnd.microsoft.card.hero",
            "content": {
                "title": "title goes here",
                "subtitle": "subtitle goes here",
                "text": "descriptive text goes here",
                "images": [
                    {
                        "url": "https://aka.ms/DuckOnARock",
                        "alt": "picture of a duck",
                        "tap": {
                            "type": "playAudio",
                            "value": "url to an audio track of a duck call goes here"
                        }
                    }
                ],
                "buttons": [
                    {
                        "type": "playAudio",
                        "title": "Duck Call",
                        "value": "url to an audio track of a duck call goes here"
                    },
                    {
                        "type": "openUrl",
                        "title": "Watch Video",
                        "image": "https://aka.ms/DuckOnARock",
                        "value": "url goes here of the duck in flight"
                    }
                ]
            }
        }
    ],
    "replyToId": "5d5cdc723"
}
```

## <a name="add-an-adaptive-card-to-a-message"></a>メッセージにアダプティブ カードを追加する

アダプティブ カードには、テキスト、音声、画像、ボタン、および入力フィールドの任意の組み合わせを含めることができます。
アダプティブ カードは、[アダプティブ カード](http://adaptivecards.io)で指定された JSON 形式を使用して作成され、カードのコンテンツと形式をフル コントロールできます。

[アダプティブ カード](http://adaptivecards.io) サイト内の情報を活用して、アダプティブ カードのスキーマを理解し、アダプティブ カードの要素について調べてください。また、さまざまな構成や複雑さを備えたカードの作成に使用できる JSON のサンプルもご覧ください。 さらに、Interactive Visualizer を使用して、アダプティブ カードのペイロードを設計し、カードの出力をプレビューできます。 作業の割り当てに使用される単一のアダプティブ カードの例を次に示します。

```json
{
  "$schema": "http://adaptivecards.io/schemas/adaptive-card.json",
  "type": "AdaptiveCard",
  "version": "1.0",
  "body": [
    {
      "type": "Container",
      "items": [
        {
          "type": "TextBlock",
          "text": "Publish Adaptive Card schema",
          "weight": "bolder",
          "size": "medium"
        },
        {
          "type": "ColumnSet",
          "columns": [
            {
              "type": "Column",
              "width": "auto",
              "items": [
                {
                  "type": "Image",
                  "url": "https://pbs.twimg.com/profile_images/3647943215/d7f12830b3c17a5a9e4afcc370e3a37e_400x400.jpeg",
                  "size": "small",
                  "style": "person"
                }
              ]
            },
            {
              "type": "Column",
              "width": "stretch",
              "items": [
                {
                  "type": "TextBlock",
                  "text": "Matt Hidinger",
                  "weight": "bolder",
                  "wrap": true
                },
                {
                  "type": "TextBlock",
                  "spacing": "none",
                  "text": "Created {{DATE(2017-02-14T06:08:39Z, SHORT)}}",
                  "isSubtle": true,
                  "wrap": true
                }
              ]
            }
          ]
        }
      ]
    },
    {
      "type": "Container",
      "items": [
        {
          "type": "TextBlock",
          "text": "Now that we have defined the main rules and features of the format, we need to produce a schema and publish it to GitHub. The schema will be the starting point of our reference documentation.",
          "wrap": true
        },
        {
          "type": "FactSet",
          "facts": [
            {
              "title": "Board:",
              "value": "Adaptive Card"
            },
            {
              "title": "List:",
              "value": "Backlog"
            },
            {
              "title": "Assigned to:",
              "value": "Matt Hidinger"
            },
            {
              "title": "Due date:",
              "value": "Not set"
            }
          ]
        }
      ]
    }
  ],
  "actions": [
    {
      "type": "Action.ShowCard",
      "title": "Comment",
      "card": {
        "type": "AdaptiveCard",
        "body": [
          {
            "type": "Input.Text",
            "id": "comment",
            "isMultiline": true,
            "placeholder": "Enter your comment"
          }
        ],
        "actions": [
          {
            "type": "Action.Submit",
            "title": "OK"
          }
        ]
      }
    },
    {
      "type": "Action.OpenUrl",
      "title": "View",
      "url": "https://adaptivecards.io"
    }
  ]
}

```

完成したカードには、タイトルのほか、カード作成者に関する情報 (名前とアバター)、カードの作成日、作業の割り当ての説明、割り当てに関連した情報が含まれます。 また、クリックして作業の割り当てに関するコメントを入力したり、そのコメントを表示したりすることができるボタンも表示されます。

![アダプティブ カードのカレンダー アラーム](../media/adaptive-card-reminder.png)

## <a name="additional-resources"></a>その他のリソース

- [メッセージの作成](bot-framework-rest-connector-create-messages.md)
- [メッセージを送受信する](bot-framework-rest-connector-send-and-receive-messages.md)
- [メッセージへのメディア添付ファイルの追加](bot-framework-rest-connector-add-media-attachments.md)
- [Bot Framework のアクティビティ スキーマ](https://aka.ms/botSpecs-activitySchema)
- [Channel Inspector][ChannelInspector]

[ChannelInspector]: ../bot-service-channels-reference.md
[Activity]: bot-framework-rest-connector-api-reference.md#activity-object
[Attachment]: bot-framework-rest-connector-api-reference.md#attachment-object
[CardAction]: bot-framework-rest-connector-api-reference.md#cardaction-object
[AnimationCard]: bot-framework-rest-connector-api-reference.md#animationcard-object
[AudioCard]: bot-framework-rest-connector-api-reference.md#audiocard-object
[HeroCard]: bot-framework-rest-connector-api-reference.md#herocard-object
[ThumbnailCard]: bot-framework-rest-connector-api-reference.md#thumbnailcard-object
[ReceiptCard]: bot-framework-rest-connector-api-reference.md#receiptcard-object
[SigninCard]: bot-framework-rest-connector-api-reference.md#signincard-object
[VideoCard]: bot-framework-rest-connector-api-reference.md#videocard-object
