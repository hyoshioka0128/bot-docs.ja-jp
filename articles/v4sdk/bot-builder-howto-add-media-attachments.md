---
title: メッセージにメディアを追加する | Microsoft Docs
description: Bot Builder SDK を使用してメッセージにメディアを追加する方法について説明します。
keywords: メディア、メッセージ、イメージ、オーディオ、ビデオ、ファイル、MessageFactory、リッチ メッセージ
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 08/24/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 5883b31df95da26fa0432f4cfe195f12fc3089ad
ms.sourcegitcommit: 86ddf3ebe6cc3385d1c4d30b971ac9c3e1fc5a77
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 08/27/2018
ms.locfileid: "43055996"
---
# <a name="add-media-to-messages"></a>メッセージにメディアを追加する

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

ユーザーとボットの間のメッセージ交換には、イメージ、ビデオ、オーディオ、ファイルなどのメディア添付ファイルを含めることができます。 Bot Builder SDK には、ユーザーへのリッチ メッセージの送信タスクが容易になるように設計された新しいクラス `Microsoft.Bot.Builder.MessageFactory` が含まれています。

## <a name="send-attachments"></a>添付ファイルを送信する

イメージやビデオなどのユーザー コンテンツを送信するには、添付ファイルまたは添付ファイルのリストをメッセージに追加します。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

`Activity` オブジェクトの `Attachments` プロパティには、メッセージに添付するメディア添付ファイルやリッチ カードを表す `Attachment` オブジェクトが格納されます。 メディア添付ファイルをメッセージに追加するには、`message` アクティビティ用の `Attachment` オブジェクトを作成し、`ContentType`、`ContentUrl`、`Name` の各プロパティを設定します。 

```csharp
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Core.Extensions;
using Microsoft.Bot.Schema;

// Create the activity and add an attachment.
var activity = MessageFactory.Attachment(
    new Attachment { ContentUrl = "imageUrl.png", ContentType = "image/png" }
);

// Send the activity to the user.
await context.SendActivity(activity);
```

メッセージ ファクトリの `Attachment` メソッドを使用して、積み重ねられた添付ファイルのリストを送信することもできます。

```csharp
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Core.Extensions;
using Microsoft.Bot.Schema;

// Create the activity and add an attachment.
var activity = MessageFactory.Attachment(new Attachment[]
{
    new Attachment { ContentUrl = "imageUrl1.png", ContentType = "image/png" },
    new Attachment { ContentUrl = "imageUrl2.png", ContentType = "image/png" },
    new Attachment { ContentUrl = "imageUrl3.png", ContentType = "image/png" }
});

// Send the activity to the user.
await context.SendActivity(activity);
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

イメージやビデオのような単一のコンテンツをユーザーに送信するには、URL に含まれているメディアを送信します。

```javascript
const {MessageFactory} = require('botbuilder');
let imageOrVideoMessage = MessageFactory.contentUrl('imageUrl.png', 'image/jpeg')

// Send the activity to the user.
await context.sendActivity(imageOrVideoMessage);
```

積み重ねられた添付ファイルのリストを送信するには: <!-- TODO: Convert the hero cards to image attachments in this example. -->

```javascript
// require MessageFactory and CardFactory from botbuilder.
const {MessageFactory, CardFactory} = require('botbuilder');

let messageWithCarouselOfCards = MessageFactory.list([
    CardFactory.heroCard('title1', ['imageUrl1'], ['button1']),
    CardFactory.heroCard('title2', ['imageUrl2'], ['button2']),
    CardFactory.heroCard('title3', ['imageUrl3'], ['button3'])
]);

await context.sendActivity(messageWithCarouselOfCards);
```

---

添付ファイルがイメージ、オーディオ、またはビデオの場合、コネクタ サービスにより、[チャネル](~/v4sdk/bot-builder-channeldata.md)での会話内の添付ファイルがレンダリングできる方法で、添付ファイルのデータがチャネルに通信されます。 添付ファイルがファイルである場合は、ファイルの URL が会話内にハイパーリンクとしてレンダリングされます。

## <a name="send-a-hero-card"></a>ヒーロー カードを送信する

シンプルなイメージ添付ファイルまたはビデオ添付ファイルだけでなく、**ヒーロー カード**を添付することもできます。これにより、1 つのオブジェクトに含まれるイメージとボタンを結合してユーザーに送信することができます。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

ヒーロー カードとボタンを使用してメッセージを作成するには、`HeroCard` をメッセージに添付します。

```csharp
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Core.Extensions;
using Microsoft.Bot.Schema;

// Create the activity and attach a Hero card.
var activity = MessageFactory.Attachment(
    new HeroCard(
        title: "White T-Shirt",
        images: new CardImage[] { new CardImage(url: "imageUrl.png") },
        buttons: new CardAction[]
        {
            new CardAction(title: "buy", type: ActionTypes.ImBack, value: "buy")
        })
    .ToAttachment());

// Send the activity as a reply to the user.
await context.SendActivity(activity);
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

カードとボタンをユーザーに送信するには、メッセージに `heroCard` を添付します。

```javascript
// require MessageFactory and CardFactory from botbuilder.
const {MessageFactory} = require('botbuilder');
const {CardFactory} = require('botbuilder');

const message = MessageFactory.attachment(
     CardFactory.heroCard(
        'White T-Shirt',
        ['https://example.com/whiteShirt.jpg'],
        ['buy']
    )
 );

await context.sendActivity(message);
```

---

<!--Lifted from the RESP API documentation-->

リッチ カードは、タイトル、説明、リンク、およびイメージから構成されます。 メッセージには、リスト形式またはカルーセル形式で表示される複数のリッチ カードを含めることができます。 Bot Builder SDK では、現在、さまざまなリッチ カードがサポートされています。 サポート状況を示すリッチ カードおよびチャネルの一覧を確認するには、[UX 要素の設計](../bot-service-design-user-experience.md)に関するページを参照してください。

> [!TIP]
> チャネルでサポートされているリッチ カードの種類を特定し、チャネルによってリッチ カードがどのようにレンダリングされるかを確認するには、[Channel Inspector](https://docs.microsoft.com/en-us/azure/bot-service/bot-service-channel-inspector?view=azure-bot-service-4.0) に関するページを参照してください。 カードのコンテンツに対する制限事項 (ボタンの最大数またはタイトルの最大長など) については、チャネルのドキュメントを参照してください。

## <a name="process-events-within-rich-cards"></a>リッチ カード内のイベントを処理する

リッチ カード内のイベントを処理するには、_カード アクション_ オブジェクトを使用して、ユーザーがボタンをクリックするか、またはカードのセクションをタップしたときのアクションを指定します。

正常に機能させるために、カード上のクリック可能な各アイテムにアクションの種類を割り当てます。 次の表では、カード アクション オブジェクトの type プロパティについて有効な値を示すと共に、種類ごとに value プロパティの想定される内容を説明します。

| type | 値 |
| :---- | :---- |
| openUrl | 組み込みのブラウザーで開かれる URL。 URL を開くことで、タップまたはクリックに応答します。 |
| imBack | (ボタンをクリックまたはカードをタップしたユーザーから) ボットに送信されるメッセージのテキスト。 会話の参加者すべてが、会話をホストしているクライアント アプリケーションを介して、このメッセージ (ユーザーからボットへの) を表示することができます。 |
| postBack | (ボタンをクリックまたはカードをタップしたユーザーから) ボットに送信されるメッセージのテキスト。 クライアント アプリケーションによっては、このテキストがメッセージ フィードに表示される場合があります。そこでは、会話の参加者のすべてにそのテキストが表示されます。 |
| 以下を呼び出します。 | 電話の呼び出し先であり、`tel:123123123123` の形式となります。呼び出しを開始することで、タップまたはクリックに応答します。|
| playAudio | 再生されるオーディオの URL。 オーディオを再生することで、タップまたはクリックに応答します。 |
| playVideo | 再生されるビデオの URL。 ビデオを再生することで、タップまたはクリックに応答します。 |
| showImage | 表示されるイメージの URL。 イメージを表示することで、タップまたはクリックに応答します。 |
| downloadFile | ダウンロードされるファイルの URL。  ファイルをダウンロードすることで、タップまたはクリックに応答します。 |
| signin | 開始される OAuth フローの URL。 サインインを開始することで、タップまたはクリックに応答します。 |

## <a name="hero-card-using-various-event-types"></a>さまざまな種類のイベントを使用するヒーロー カード

次のコードでは、さまざまなリッチ カード イベントを使用する例を示します。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Core.Extensions;
using Microsoft.Bot.Schema;

// Create the activity and attach a Hero card.
var activity = MessageFactory.Attachment(
    new HeroCard(
        title: "Holler Back Buttons",
        images: new CardImage[] { new CardImage(url: "imageUrl.png") },
        buttons: new CardAction[]
        {
            new CardAction(title: "Shout Out Loud", type: ActionTypes.imBack, value: "You can ALL hear me!"),
            new CardAction(title: "Much Quieter", type: ActionTypes.postBack, value: "Shh! My Bot friend hears me."),
            new CardAction(title: "Show me how to Holler", type: ActionTypes.openURL, value: $"https://en.wikipedia.org/wiki/{cardContent.Key}")
        })
    .ToAttachment());

// Send the activity as a reply to the user.
await context.SendActivity(activity);
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
const {ActionTypes} = require("botbuilder");
```

```javascript
const hero = MessageFactory.attachment(
    CardFactory.heroCard(
        'Holler Back Buttons',
        ['https://example.com/whiteShirt.jpg'],
        [{
            type: ActionTypes.ImBack,
            title: 'ImBack',
            value: 'You can ALL hear me! Shout Out Loud'
        },
        {
            type: ActionTypes.PostBack,
            title: 'PostBack',
            value: 'Shh! My Bot friend hears me. Much Quieter'
        },
        {
            type: ActionTypes.OpenUrl,
            title: 'OpenUrl',
            value: 'https://en.wikipedia.org/wiki/{cardContent.Key}'
        }]
    )
);

await context.sendActivity(hero);

```

---

## <a name="send-an-adaptive-card"></a>アダプティブ カードを送信する
ユーザーとの通信に、テキスト、画像、動画、オーディオ、ファイルなど、多彩なメッセージを送信するために、アダプティブ カードと MessageFactory が使用されます。 ただし、これには違いがいくつかあります。 

まず、アダプティブ カードをサポートするのは一部のチャネルのみで、サポートしていないチャネルでは、アダプティブ カードが部分的にしかサポートされない可能性があります。 たとえば、Facebook でアダプティブ カードを送信する場合、テキストと画像は適切に機能しますが、ボタンは機能しません。 MessageFactory は、作成手順を自動化するための、Bot Builder SDK 内の単なるヘルパー クラスであり、ほとんどのチャネルでサポートされています。 

また、アダプティブ カードではカード形式でメッセージが配信され、チャネルによって、カードのレイアウトが決まります。 MessageFactory によって配信されるメッセージの形式はチャネルによって異なり、アダプティブ カードが添付ファイルに含まれていない限り、必ずしもカード形式であるとは限りません。 

チャネルでのアダプティブ カードのサポートに関する最新情報については、<a href="http://adaptivecards.io/visualizer/">アダプティブ カード ビジュアライザー</a>に関するページを参照してください。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)
アダプティブ カードを使用するには、必ず `Microsoft.AdaptiveCards` NuGet パッケージを追加してください。


> [!NOTE]
> ご自分のボットで使用されるチャネルにおいてこの機能をテストして、それらのチャネルでアダプティブ カードがサポートされているかどうかを判断する必要があります。

```csharp
using AdaptiveCards;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Core.Extensions;
using Microsoft.Bot.Schema;

// Create the activity and attach an Adaptive Card.
var card = new AdaptiveCard();
card.Body.Add(new TextBlock()
{
    Text = "<title>",
    Size = TextSize.Large,
    Wrap = true,
    Weight = TextWeight.Bolder
});
card.Body.Add(new TextBlock() { Text = "<message text>", Wrap = true });
card.Body.Add(new TextInput()
{
    Id = "Title",
    Value = string.Empty,
    Style = TextInputStyle.Text,
    Placeholder = "Title",
    IsRequired = true,
    MaxLength = 50
});
card.Actions.Add(new SubmitAction() { Title = "Submit", DataJson = "{ Action:'Submit' }" });
card.Actions.Add(new SubmitAction() { Title = "Cancel", DataJson = "{ Action:'Cancel'}" });

var activity = MessageFactory.Attachment(new Attachment(AdaptiveCard.ContentType, content: card));

// Send the activity as a reply to the user.
await context.SendActivity(activity);
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
const {CardFactory} = require("botbuilder");

const message = CardFactory.adaptiveCard({
    "$schema": "http://adaptivecards.io/schemas/adaptive-card.json",
    "version": "1.0",
    "type": "AdaptiveCard",
    "speak": "Your flight is confirmed for you from San Francisco to Amsterdam on Friday, October 10 8:30 AM",
    "body": [
        {
            "type": "TextBlock",
            "text": "Passenger",
            "weight": "bolder",
            "isSubtle": false
        },
        {
            "type": "TextBlock",
            "text": "Sarah Hum",
            "separator": true
        },
        {
            "type": "TextBlock",
            "text": "1 Stop",
            "weight": "bolder",
            "spacing": "medium"
        },
        {
            "type": "TextBlock",
            "text": "Fri, October 10 8:30 AM",
            "weight": "bolder",
            "spacing": "none"
        },
        {
            "type": "ColumnSet",
            "separator": true,
            "columns": [
                {
                    "type": "Column",
                    "width": 1,
                    "items": [
                        {
                            "type": "TextBlock",
                            "text": "San Francisco",
                            "isSubtle": true
                        },
                        {
                            "type": "TextBlock",
                            "size": "extraLarge",
                            "color": "accent",
                            "text": "SFO",
                            "spacing": "none"
                        }
                    ]
                },
                {
                    "type": "Column",
                    "width": "auto",
                    "items": [
                        {
                            "type": "TextBlock",
                            "text": " "
                        },
                        {
                            "type": "Image",
                            "url": "http://messagecardplayground.azurewebsites.net/assets/airplane.png",
                            "size": "small",
                            "spacing": "none"
                        }
                    ]
                },
                {
                    "type": "Column",
                    "width": 1,
                    "items": [
                        {
                            "type": "TextBlock",
                            "horizontalAlignment": "right",
                            "text": "Amsterdam",
                            "isSubtle": true
                        },
                        {
                            "type": "TextBlock",
                            "horizontalAlignment": "right",
                            "size": "extraLarge",
                            "color": "accent",
                            "text": "AMS",
                            "spacing": "none"
                        }
                    ]
                }
            ]
        },
        {
            "type": "ColumnSet",
            "spacing": "medium",
            "columns": [
                {
                    "type": "Column",
                    "width": "1",
                    "items": [
                        {
                            "type": "TextBlock",
                            "text": "Total",
                            "size": "medium",
                            "isSubtle": true
                        }
                    ]
                },
                {
                    "type": "Column",
                    "width": 1,
                    "items": [
                        {
                            "type": "TextBlock",
                            "horizontalAlignment": "right",
                            "text": "$1,032.54",
                            "size": "medium",
                            "weight": "bolder"
                        }
                    ]
                }
            ]
        }
    ]
});

// send adaptive card as attachment 
await context.sendActivity({ attachments: [message] })
```

---

## <a name="send-a-carousel-of-cards"></a>カードのカルーセルを送信する

また、メッセージには複数の添付ファイルをカルーセル レイアウトで含めることもできます。このレイアウトでは、添付ファイルが左右に並べて配置され、ユーザーは全体をスクロールすることができます。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Core.Extensions;
using Microsoft.Bot.Schema;

// Create the activity and attach a set of Hero cards.
var activity = MessageFactory.Carousel(
    new Attachment[]
    {
        new HeroCard(
            title: "title1",
            images: new CardImage[] { new CardImage(url: "imageUrl1.png") },
            buttons: new CardAction[]
            {
                new CardAction(title: "button1", type: ActionTypes.ImBack, value: "item1")
            })
        .ToAttachment(),
        new HeroCard(
            title: "title2",
            images: new CardImage[] { new CardImage(url: "imageUrl2.png") },
            buttons: new CardAction[]
            {
                new CardAction(title: "button2", type: ActionTypes.ImBack, value: "item2")
            })
        .ToAttachment(),
        new HeroCard(
            title: "title3",
            images: new CardImage[] { new CardImage(url: "imageUrl3.png") },
            buttons: new CardAction[]
            {
                new CardAction(title: "button3", type: ActionTypes.ImBack, value: "item3")
            })
        .ToAttachment()
    });

// Send the activity as a reply to the user.
await context.SendActivity(activity);
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
// require MessageFactory and CardFactory from botbuilder.
const {MessageFactory, CardFactory} = require('botbuilder');

//  init message object
let messageWithCarouselOfCards = MessageFactory.carousel([
    CardFactory.heroCard('title1', ['imageUrl1'], ['button1']),
    CardFactory.heroCard('title2', ['imageUrl2'], ['button2']),
    CardFactory.heroCard('title3', ['imageUrl3'], ['button3'])
]);

await context.sendActivity(messageWithCarouselOfCards);
```

---

## <a name="additional-resources"></a>その他のリソース

[Channel Inspector を使用して機能をプレビューする](../bot-service-channel-inspector.md)

---
