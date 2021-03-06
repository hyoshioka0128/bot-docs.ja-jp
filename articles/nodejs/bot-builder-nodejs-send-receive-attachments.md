---
title: 添付ファイルを送受信する - Bot Service
description: Bot Framework SDK for Node.js を使用して添付ファイルを含むメッセージを送受信する方法について説明します。
author: DeniseMak
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 182fb3ab327b55d8976a607871bb2380a88f8b30
ms.sourcegitcommit: f8b5cc509a6351d3aae89bc146eaabead973de97
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/09/2020
ms.locfileid: "75790560"
---
# <a name="send-and-receive-attachments"></a>添付ファイルを送受信する

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-add-media-attachments.md)
> - [Node.js](../nodejs/bot-builder-nodejs-send-receive-attachments.md)
> - [REST](../rest-api/bot-framework-rest-connector-add-media-attachments.md)

ユーザーとボットの間のメッセージ交換には、イメージ、ビデオ、オーディオ、ファイルなどのメディア添付ファイルを含めることができます。 送信できる添付ファイルの種類はチャネルによって異なりますが、以下に基本的な種類を示します。

* **メディアおよびファイル**:**contentType** に [IAttachment オブジェクト][IAttachment]の MIME の種類を設定し、ファイルへのリンクを **contentUrl** に渡すことで、イメージ、オーディオ、ビデオなどのファイルを送信することができます。
* **カード**: <!-- and custom keyboards --> **contentType** に目的のカードの種類を設定し、カードの JSON を渡すことで、視覚的に優れた一連のカードを送信することができます。 **HeroCard** などのリッチ カード ビルダー クラスを使用する場合、添付ファイルは自動的に追加されます。 この例については、「[リッチ カードを送信する](bot-builder-nodejs-send-rich-cards.md)」をご覧ください。

## <a name="add-a-media-attachment"></a>メディア添付ファイルの追加
メッセージ オブジェクトは、[IMessage][IMessage] のインスタンスである必要があります。これは、イメージなどの添付ファイルを含めたい場合、ユーザーにオブジェクトとしてメッセージを送信する際にとても便利です。 JSON オブジェクト形式でメッセージを送信するには、[session.send()][SessionSend] メソッドを使用します。 

## <a name="example"></a>例

次の例では、ユーザーが添付ファイルを送信したかを確認しています。ユーザーが添付ファイルを送信した場合、添付ファイルに含まれるすべての画像をエコーで返します。 これは、Bot Framework Emulator でボットに画像を送ることでテストできます。

```javascript
// Create your bot with a function to receive messages from the user
var bot = new builder.UniversalBot(connector, function (session) {
    var msg = session.message;
    if (msg.attachments && msg.attachments.length > 0) {
     // Echo back attachment
     var attachment = msg.attachments[0];
        session.send({
            text: "You sent:",
            attachments: [
                {
                    contentType: attachment.contentType,
                    contentUrl: attachment.contentUrl,
                    name: attachment.name
                }
            ]
        });
    } else {
        // Echo back users text
        session.send("You said: %s", session.message.text);
    }
});
```
## <a name="additional-resources"></a>その他のリソース

* [チャネル リファレンス][inspector]
* [IMessage][IMessage]
* [リッチ カードを送信する][SendRichCard]
* [session.send][SessionSend]

[IMessage]: http://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.imessage
[SendRichCard]: bot-builder-nodejs-send-rich-cards.md
[SessionSend]: https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.session.html#send
[IAttachment]: https://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.iattachment.html
[inspector]: ../bot-service-channels-reference.md
