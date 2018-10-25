---
title: メッセージにメディアを追加する | Microsoft Docs
description: Bot Builder SDK を使用してメッセージにメディアを追加する方法について説明します。
keywords: メディア, メッセージ, イメージ, オーディオ, ビデオ, ファイル, MessageFactory, リッチ カード, メッセージ, アダプティブ カード, ヒーロー カード, 推奨されるアクション
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 09/10/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 35ec8968493eb024b2724d0729a8a2cd6e14ba82
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/24/2018
ms.locfileid: "50000371"
---
# <a name="add-media-to-messages"></a>メッセージにメディアを追加する

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

ユーザーとボットの間のメッセージ交換には、イメージ、ビデオ、オーディオ、ファイルなどのメディア添付ファイルを含めることができます。 Bot Builder SDK は、ユーザーにリッチ メッセージを送信するタスクをサポートしています。 チャネル (Facebook、Skype、Slack など) がサポートするリッチ メッセージの種類を確認するには、チャネルのドキュメントで制限事項に関する情報を参照してください。 使用可能なカードの一覧については、[ユーザー エクスペリエンスの設計](../bot-service-design-user-experience.md)に関するページを参照してください。 

## <a name="send-attachments"></a>添付ファイルを送信する

イメージやビデオなどのユーザー コンテンツを送信するには、添付ファイルまたは添付ファイルのリストをメッセージに追加します。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

`Activity` オブジェクトの `Attachments` プロパティには、メッセージに添付するメディア添付ファイルやリッチ カードを表す `Attachment` オブジェクトが格納されます。 メディア添付ファイルをメッセージに追加するには、`message` アクティビティ用の `Attachment` オブジェクトを作成し、`ContentType`、`ContentUrl`、`Name` の各プロパティを設定します。 `Activity` オブジェクトの `Attachments` プロパティには、メッセージに添付するメディア添付ファイルやリッチ カードを表す `Attachment` オブジェクトが格納されます。 メディア添付ファイルをメッセージに追加するには、`Attachment` メソッドを使用して `message` アクティビティ用の `Attachment` オブジェクトを作成し、`ContentType`、`ContentUrl`、`Name` の各プロパティを設定します。 ここで示すソース コードは、[添付ファイルの処理](https://aka.ms/bot-attachments-sample-code)のサンプルに基づいています。 

```csharp
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Core.Extensions;
using Microsoft.Bot.Schema;

var reply = turnContext.Activity.CreateReply();

// Create an attachment.
var attachment = new Attachment
    {
        ContentUrl = "imageUrl.png",
        ContentType = "image/png",
        Name = "imageName",
    };
    
// Add the attachment to our reply.
reply.Attachments = new List<Attachment>() { attachment };

// Send the activity to the user.
await turnContext.SendActivityAsync(reply, cancellationToken);
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

ここで示すソース コードは、[JS の添付ファイルの処理](https://aka.ms/bot-attachments-sample-code-js)のサンプルに基づいています。
イメージやビデオのような単一のコンテンツをユーザーに送信するには、URL に含まれているメディアを送信します。

```javascript
const { ActionTypes, ActivityTypes, CardFactory } = require('botbuilder');
// Call function to get an attachment.
reply.attachments = [this.getInternetAttachment()];
reply.text = 'This is an internet attachment.';
// Send the activity to the user.
await turnContext.sendActivity(reply);

/* function getInternetAttachment - Returns an attachment to be sent to the user from a HTTPS URL */
getInternetAttachment() {
        return {
            name: 'imageName.png',
            contentType: 'image/png',
            contentUrl: 'imageUrl.png'}
}
```

---

添付ファイルがイメージ、オーディオ、またはビデオの場合、コネクタ サービスにより、[チャネル](bot-builder-channeldata.md)での会話内の添付ファイルがレンダリングできる方法で、添付ファイルのデータがチャネルに通信されます。 添付ファイルがファイルである場合は、ファイルの URL が会話内にハイパーリンクとしてレンダリングされます。

## <a name="send-a-hero-card"></a>ヒーロー カードを送信する

シンプルなイメージ添付ファイルまたはビデオ添付ファイルだけでなく、**ヒーロー カード**を添付することもできます。これにより、1 つのオブジェクトに含まれるイメージとボタンを結合してユーザーに送信することができます。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

ヒーロー カードとボタンを使用してメッセージを作成するには、`HeroCard` をメッセージに添付します。 ここで示すソース コードは、[添付ファイルの処理](https://aka.ms/bot-attachments-sample-code)のサンプルに基づいています。 

```csharp
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Core.Extensions;
using Microsoft.Bot.Schema;

var reply = turnContext.Activity.CreateReply();

// Create a HeroCard with options for the user to choose to interact with the bot.
var card = new HeroCard
{
    Text = "You can upload an image or select one of the following choices",
    Buttons = new List<CardAction>()
    {
        new CardAction(ActionTypes.ImBack, title: "1. Inline Attachment", value: "1"),
        new CardAction(ActionTypes.ImBack, title: "2. Internet Attachment", value: "2"),
        new CardAction(ActionTypes.ImBack, title: "3. Uploaded Attachment", value: "3"),
    },
};

// Add the card to our reply.
reply.Attachments = new List<Attachment>() { card.ToAttachment() };

await turnContext.SendActivityAsync(reply, cancellationToken);
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

ヒーロー カードとボタンを使用してメッセージを作成するには、`HeroCard` をメッセージに添付します。 ここで示すソース コードは、[JS の添付ファイルの処理](https://aka.ms/bot-attachments-sample-code-js)のサンプルに基づいています。

```javascript
const { ActionTypes, ActivityTypes, CardFactory } = require('botbuilder');
// build buttons to display.
const buttons = [
            { type: ActionTypes.ImBack, title: '1. Inline Attachment', value: '1' },
            { type: ActionTypes.ImBack, title: '2. Internet Attachment', value: '2' },
            { type: ActionTypes.ImBack, title: '3. Uploaded Attachment', value: '3' }
];

// construct hero card.
const card = CardFactory.heroCard('', undefined,
buttons, { text: 'You can upload an image or select one of the following choices.' });

// add card to Activity.
reply.attachments = [card];

// Send hero card to the user.
await turnContext.sendActivity(reply);
```
---

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

var reply = turnContext.Activity.CreateReply();

var card = new HeroCard
{
    Buttons = new List<CardAction>()
    {
        new CardAction(title: "Much Quieter", type: ActionTypes.PostBack, value: "Shh! My Bot friend hears me."),
        new CardAction(ActionTypes.OpenUrl, title: "Azure Bot Service", value: "https://azure.microsoft.com/en-us/services/bot-service/"),
    },
};

```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
const {ActionTypes} = require("botbuilder");

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

アダプティブ カードを使用するには、必ず `Microsoft.AdaptiveCards` NuGet パッケージを追加してください。 


> [!NOTE]
> ご自分のボットで使用されるチャネルにおいてこの機能をテストして、それらのチャネルでアダプティブ カードがサポートされているかどうかを判断する必要があります。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

ここで示すソース コードは、[アダプティブ カードの使用](https://aka.ms/bot-adaptive-cards-sample-code)のサンプルに基づいています。

```csharp
using AdaptiveCards;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Core.Extensions;
using Microsoft.Bot.Schema;

// Creates an attachment that contains an adaptive card
// filePath is the path to JSON file
private static Attachment CreateAdaptiveCardAttachment(string filePath)
{
    var adaptiveCardJson = File.ReadAllText(filePath);
    var adaptiveCardAttachment = new Attachment()
    {
        ContentType = "application/vnd.microsoft.card.adaptive",
        Content = JsonConvert.DeserializeObject(adaptiveCardJson),
    };
    return adaptiveCardAttachment;
}

// Create adaptive card and attach it to the message 
var cardAttachment = CreateAdaptiveCardAttachment(adaptiveCardJsonFilePath);
var reply = turnContext.Activity.CreateReply();
reply.Attachments = new List<Attachment>() { cardAttachment };

await turnContext.SendActivityAsync(reply, cancellationToken);
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

ここで示すソース コードは、[JS のアダプティブ カードの使用](https://aka.ms/bot-adaptive-cards-js-sample-code)のサンプルに基づいています。

```javascript
const { BotFrameworkAdapter } = require('botbuilder');

// Import AdaptiveCard content.
const FlightItineraryCard = require('./resources/FlightItineraryCard.json');
const ImageGalleryCard = require('./resources/ImageGalleryCard.json');
const LargeWeatherCard = require('./resources/LargeWeatherCard.json');
const RestaurantCard = require('./resources/RestaurantCard.json');
const SolitaireCard = require('./resources/SolitaireCard.json');

// Create array of AdaptiveCard content, this will be used to send a random card to the user.
const CARDS = [
    FlightItineraryCard,
    ImageGalleryCard,
    LargeWeatherCard,
    RestaurantCard,
    SolitaireCard
];
// Select a random card to send.
const randomlySelectedCard = CARDS[Math.floor((Math.random() * CARDS.length - 1) + 1)];
// Send adaptive card.
await context.sendActivity({
      text: 'Here is an Adaptive Card:',
       attachments: [CardFactory.adaptiveCard(randomlySelectedCard)]
});
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
サンプル コードについては、こちら (カード: [C#](https://aka.ms/bot-cards-sample-code)/[JS](https://aka.ms/bot-cards-js-sample-code)、アダプティブ カード: [C#](https://aka.ms/bot-adaptive-cards-sample-code)/[JS](https://aka.ms/bot-adaptive-cards-js-sample-code)、添付ファイル: [C#](https://aka.ms/bot-attachments-sample-code)/[JS](https://aka.ms/bot-attachments-sample-code-js)、推奨されるアクション: [C#](https://aka.ms/SuggestedActionsCSharp)/[JS](https://aka.ms/SuggestedActionsJS)) を参照してください。 その他のサンプルについては、[GitHub](https://github.com/Microsoft/BotBuilder-Samples) の Bot Builder サンプル リポジトリを参照してください。
