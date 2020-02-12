---
title: メッセージにリッチ カード添付ファイルを追加する (v3 JS) - Bot Service
description: Bot Framework SDK for Node.js を使用して魅力的な対話型リッチ カードを送信する方法について説明します。
author: v-ducvo
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: cd2997bc4795a922d25f6c4b4f5fef3d7dd4f366
ms.sourcegitcommit: 4e1af50bd46debfdf9dcbab9a5d1b1633b541e27
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/25/2020
ms.locfileid: "76752914"
---
# <a name="add-rich-card-attachments-to-messages"></a>メッセージにリッチ カード添付ファイルを追加する

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-add-rich-card-attachments.md)
> - [Node.js](../nodejs/bot-builder-nodejs-send-rich-cards.md)
> - [REST](../rest-api/bot-framework-rest-connector-add-rich-cards.md)

Skype、Facebook などの複数のチャネルで、ユーザーがクリックすることでアクションが開始される対話型ボタンを使用して、グラフィカルなリッチ カードをユーザーに送信できます。
SDK には、カードを作成および送信するときに使用できる、メッセージとカード ビルダー クラスが複数用意されています。 これらのカードは、チャネルにネイティブなスキーマを使用して、Bot Framework Connector サービスによってレンダリングされ、クロスプラットフォーム通信がサポートされます。 チャネルで SMS などのカードがサポートされていない場合、Bot Framework では、ユーザーに対して妥当なエクスペリエンスが提供されるように最善が尽くされます。

## <a name="types-of-rich-cards"></a>リッチ カードの種類

Bot Framework では、現在 8 種類のリッチ カードがサポートされています。

| カードの種類 | [説明] |
|------|------|
| [アダプティブ カード](/adaptive-cards/get-started/bots) | テキスト、音声、画像、ボタン、および入力フィールドの任意の組み合わせを含めることができる、カスタマイズ可能なカード。  [チャネルごとのサポート](/adaptive-cards/get-started/bots#channel-status)に関するページをご覧ください。 |
| [アニメーション カード][animationCard] | アニメーション GIF または短い動画を再生できるカード。 |
| [オーディオ カード][audioCard] | オーディオ ファイルを再生できるカード。 |
| [ヒーロー カード][heroCard] | 通常 1 つの大きなイメージ、1 つまたは複数のボタン、およびテキストが含まれるカード。 |
| [サムネイル カード][thumbnailCard] | 通常 1 つのサムネイル イメージ、1 つまたは複数のボタン、およびテキストが含まれるカード。|
| [領収書カード][receiptCard] | ボットからユーザーに領収書を提供できるようにするカード。 通常は、領収書に含める項目の一覧、税金と合計の情報、およびその他のテキストが含まれます。 |
| [サインイン カード][signinCard] | ボットでユーザーのサインインを要求できるようにするカード。 通常は、テキストと、ユーザーがクリックしてサインイン プロセスを開始できる 1 つ以上のボタンが含まれます。 |
| [ビデオ カード][videoCard] | 動画を再生できるカード。 |

## <a name="send-a-carousel-of-hero-cards"></a>ヒーロー カードのカルーセルを送信する

次の例は、架空の T シャツ会社のボットと、ユーザーが "シャツを表示" と言ったときの応答としてカードのカルーセルを送信する方法を示しています。

```javascript
// Create your bot with a function to receive messages from the user
// Create bot and default message handler
var bot = new builder.UniversalBot(connector, function (session) {
    session.send("Hi... We sell shirts. Say 'show shirts' to see our products.");
});

// Add dialog to return list of shirts available
bot.dialog('showShirts', function (session) {
    var msg = new builder.Message(session);
    msg.attachmentLayout(builder.AttachmentLayout.carousel)
    msg.attachments([
        new builder.HeroCard(session)
            .title("Classic White T-Shirt")
            .subtitle("100% Soft and Luxurious Cotton")
            .text("Price is $25 and carried in sizes (S, M, L, and XL)")
            .images([builder.CardImage.create(session, 'http://petersapparel.parseapp.com/img/whiteshirt.png')])
            .buttons([
                builder.CardAction.imBack(session, "buy classic white t-shirt", "Buy")
            ]),
        new builder.HeroCard(session)
            .title("Classic Gray T-Shirt")
            .subtitle("100% Soft and Luxurious Cotton")
            .text("Price is $25 and carried in sizes (S, M, L, and XL)")
            .images([builder.CardImage.create(session, 'http://petersapparel.parseapp.com/img/grayshirt.png')])
            .buttons([
                builder.CardAction.imBack(session, "buy classic gray t-shirt", "Buy")
            ])
    ]);
    session.send(msg).endDialog();
}).triggerAction({ matches: /^(show|list)/i });
```

この例では、[Message][Message] クラスを使用してカルーセルを作成します。  
カルーセルは、画像、テキスト、および項目の購入をトリガーする 1 つのボタンが含まれる [HeroCard][heroCard] クラスの一覧で構成されます。  
**[購入]** ボタンをクリックするとメッセージの送信がトリガーされるため、2 つ目のダイアログを追加して、そのボタン クリックをキャッチする必要があります。

## <a name="handle-button-input"></a>ボタン入力を処理する

"buy" または "add" で始まり、"shirt" というワードを含む何かが後に続くメッセージを受信するたびに、`buyButtonClick` ダイアログがトリガーされます。
このダイアログは、ユーザーが求める色と、必要に応じてシャツのサイズを検索するために、いくつかの正規表現を使用して開始されます。
これにより柔軟性が増し、ボタン クリックと "大きな灰色のシャツをカートに追加してください" といった自然言語メッセージの両方に対応することができます。
色が有効で、サイズが不明の場合、ユーザーは、項目をカートに追加する前に一覧からサイズを選択するようにボットから求められます。
必要なすべての情報がボットに入力されたら、その項目は、永続化されているカートに **session.userData** を使用して追加され、確認メッセージがユーザーに送信されます。

```javascript
// Add dialog to handle 'Buy' button click
bot.dialog('buyButtonClick', [
    function (session, args, next) {
        // Get color and optional size from users utterance
        var utterance = args.intent.matched[0];
        var color = /(white|gray)/i.exec(utterance);
        var size = /\b(Extra Large|Large|Medium|Small)\b/i.exec(utterance);
        if (color) {
            // Initialize cart item
            var item = session.dialogData.item = {
                product: "classic " + color[0].toLowerCase() + " t-shirt",
                size: size ? size[0].toLowerCase() : null,
                price: 25.0,
                qty: 1
            };
            if (!item.size) {
                // Prompt for size
                builder.Prompts.choice(session, "What size would you like?", "Small|Medium|Large|Extra Large");
            } else {
                //Skip to next waterfall step
                next();
            }
        } else {
            // Invalid product
            session.send("I'm sorry... That product wasn't found.").endDialog();
        }
    },
    function (session, results) {
        // Save size if prompted
        var item = session.dialogData.item;
        if (results.response) {
            item.size = results.response.entity.toLowerCase();
        }

        // Add to cart
        if (!session.userData.cart) {
            session.userData.cart = [];
        }
        session.userData.cart.push(item);

        // Send confirmation to users
        session.send("A '%(size)s %(product)s' has been added to your cart.", item).endDialog();
    }
]).triggerAction({ matches: /(buy|add)\s.*shirt/i });
```

<!-- 
> [!NOTE]
> When sending a message that contains images, keep in mind that some channels download images before displaying a message to the user. >  
> As a result, a message containing an image followed immediately by a message without images may sometimes be flipped in the user's feed.
> For information on how to avoid messages being sent out of order, see [Message ordering][MessageOrder].  
-->

## <a name="add-a-message-delay-for-image-downloads"></a>画像のダウンロードためにメッセージの遅延を追加する

一部のチャネルでは、画像がダウンロードされた後にメッセージが表示される傾向があるため、画像が含まれるメッセージの直後に画像が含まれないメッセージを送信すると、これらのメッセージはユーザーのフィードで逆になることがあります。 これが発生する可能性を最小限に抑えるには、お使いの画像が必ずコンテンツ配信ネットワーク (CDN) から配信されるようにします。また、過度に大きい画像の使用は避けるようにしてください。 極端な場合、画像が含まれるメッセージと、それに続くメッセージの間に 1 ～ 2 秒の遅延を追加しなければならないこともあります。 ユーザーがこの遅延を若干自然に感じられるようにするには、その遅延を開始する前に、**session.sendTyping()** を呼び出して typing インジケーターを送信します。

<!-- 
To learn more about sending a typing indicator, see [How to send a typing indicator](bot-builder-nodejs-send-typing-indicator.md).
-->

Bot Framework では、ボットからの複数のメッセージが順不同で表示されるのを防ぐために、バッチ処理が実装されています。 <!-- Unfortunately, not all channels can guarantee this. --> ご利用のボットによって複数の応答がユーザーに送信されるとき、メッセージの元の順序を維持するために、個別のメッセージが自動的にバッチにグループ化され、1 つのセットとしてユーザーに配信されます。 この自動バッチ処理では、毎回 **session.send()** が呼び出された後、既定の 250 ミリ秒の待機時間を経てから、次の **send()** の呼び出しが開始されます。

メッセージのバッチ処理の遅延は構成可能です。 SDK の自動バッチ処理ロジックを無効にするには、既定の遅延を大きな数値に設定して、バッチが配信された後に呼び出すコールバックによって **sendBatch()** を手動で呼び出します。

## <a name="send-an-adaptive-card"></a>アダプティブ カードを送信する

アダプティブ カードには、テキスト、音声、画像、ボタン、および入力フィールドの任意の組み合わせを含めることができます。
アダプティブ カードは、[アダプティブ カード](http://adaptivecards.io)で指定された JSON 形式を使用して作成され、カードのコンテンツと形式をフル コントロールできます。

Node.js を使用してアダプティブ カードを作成するには、[アダプティブ カード](http://adaptivecards.io) サイト内の情報を活用して、アダプティブ カードのスキーマを理解し、アダプティブ カードの要素について調べてください。また、さまざまな構成や複雑さを備えたカードの作成に使用できる JSON のサンプルもご覧ください。 さらに、Interactive Visualizer を使用して、アダプティブ カードのペイロードを設計し、カードの出力をプレビューできます。

次のコード例では、カレンダー アラーム用のアダプティブ カードが含まれるメッセージを作成する方法を示します。

[!code-javascript[Add Adaptive Card attachment](../includes/code/node-send-card-buttons.js#addAdaptiveCardAttachment)]

結果のカードには、3 つのブロック (テキスト、入力フィールド (選択リスト)、および 3 つのボタン) が含まれています。

![アダプティブ カードのカレンダー アラーム](../media/adaptive-card-reminder.png)

## <a name="additional-resources"></a>その他のリソース

- [チャネル リファレンス][inspector]
- [アダプティブ カード](http://adaptivecards.io)
- [AnimationCard][animationCard]
- [AudioCard][audioCard]
- [HeroCard][heroCard]
- [ThumbnailCard][thumbnailCard]
- [ReceiptCard][receiptCard]
- [SigninCard][signinCard]
- [VideoCard][videoCard]
- [メッセージ][Message]
- [添付ファイルの送信方法](bot-builder-nodejs-send-receive-attachments.md)

[MessageOrder]: bot-builder-nodejs-manage-conversation-flow.md#message-ordering
[Message]: https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.message
[IMessage]: http://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.imessage

[animationCard]: https://docs.microsoft.com/javascript/api/botbuilder/animationcard?view=botbuilder-ts-3.0
[audioCard]: https://docs.microsoft.com/javascript/api/botbuilder/audiocard?view=botbuilder-ts-3.0
[heroCard]: https://docs.microsoft.com/javascript/api/botbuilder/herocard?view=botbuilder-ts-3.0
[thumbnailCard]: https://docs.microsoft.com/javascript/api/botbuilder/thumbnailcard?view=botbuilder-ts-3.0
[receiptCard]: https://docs.microsoft.com/javascript/api/botbuilder/receiptcard?view=botbuilder-ts-3.0
[signinCard]: https://docs.microsoft.com/javascript/api/botbuilder/signincard?view=botbuilder-ts-3.0
[videoCard]: https://docs.microsoft.com/javascript/api/botbuilder/videocard?view=botbuilder-ts-3.0

[inspector]: ../bot-service-channels-reference.md
