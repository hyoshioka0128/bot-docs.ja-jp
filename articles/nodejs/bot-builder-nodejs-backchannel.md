---
title: Web コントロールを使用して情報を交換する | Microsoft Docs
description: Bot Builder SDK for Node.js を使用して、ボットと Web ページの間で情報を交換する方法について説明します。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 5286f0cc6a6a5d7558de997fc776671219c70aa5
ms.sourcegitcommit: 2dc75701b169d822c9499e393439161bc87639d2
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 08/24/2018
ms.locfileid: "42904871"
---
# <a name="use-the-backchannel-mechanism"></a>バックチャネル メカニズムの使用

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

[!INCLUDE [Introduction to backchannel mechanism](../includes/snippet-backchannel.md)]

## <a name="walk-through"></a>チュートリアル

オープン ソースの Web チャット コントロールは、<a href="https://github.com/microsoft/botframework-DirectLinejs" target="_blank">DirectLineJS</a> という JavaScript クラスを使用して Direct Line API にアクセスします。 このコントロールで Direct Line の独自のインスタンスを作成したり、ホスティング ページと共有したりすることもできます。 コントロールが Direct Line のインスタンスをホスティング ページと共有している場合、コントロールとページの両方がアクティビティを送受信できます。 次の図は、オープン ソースの Web (チャット) コントロールと Direct Line API を使用してボット機能をサポートする Web サイトのアーキテクチャの概要を示しています。 

![バックチャネル](../media/designing-bots/patterns/back-channel.png)

### <a name="sample-code"></a>サンプル コード 

この例で、ボットと Web ページはバックチャネル メカニズムを使用して、ユーザーには表示されない情報を交換します。 ボットは、Web ページの背景色を変更することを要求し、ユーザーがページのボタンをクリックすると、Web ページはボットに通知します。 

> [!NOTE]
> この記事のコード スニペットは、<a href="https://github.com/Microsoft/BotFramework-WebChat/blob/master/samples/backchannel/index.html" target="_blank">バックチャネル サンプル</a>と<a href="https://github.com/ryanvolum/backChannelBot" target="_blank">バックチャネル ボット</a>に由来しています。 

#### <a name="client-side-code"></a>クライアント側のコード

まず Web ページで **DirectLine** オブジェクトを作成します。

```javascript
var botConnection = new BotChat.DirectLine(...);
```

次に、WebChat インスタンスの作成時に **DirectLine** オブジェクトを共有します。

```javascript
BotChat.App({
    botConnection: botConnection,
    user: user,
    bot: bot
}, document.getElementById("BotChatGoesHere"));
```

ユーザーが Web ページのボタンをクリックすると、ボタンがクリックされたことをボットに通知するために、Web ページは種類が "event" のアクティビティを通知します。

```javascript
const postButtonMessage = () => {
    botConnection
        .postActivity({type: "event", value: "", from: {id: "me" }, name: "buttonClicked"})
        .subscribe(id => console.log("success"));
    }
```

> [!TIP]
> 属性 `name` および `value` を使用して、ボットがイベントを適切に解釈したり、イベントに応答したりするために必要な可能性がある情報を伝えます。 

最後に、Web ページはボットからの特定のイベントもリッスンします。
この例で、Web ページは type="event" で name="changeBackground" のアクティビティをリッスンします。 この種類のアクティビティを受け取ると、Web ページの背景色はアクティビティで指定された `value` に変更されます。 

```javascript
botConnection.activity$
    .filter(activity => activity.type === "event" && activity.name === "changeBackground")
    .subscribe(activity => changeBackgroundColor(activity.value))
```

#### <a name="server-side-code"></a>サーバー側のコード

<a href="https://github.com/ryanvolum/backChannelBot" target="_blank">バックチャネル ボット</a>で、ヘルパー関数を使用してイベントを作成します。

```javascript
var bot = new builder.UniversalBot(connector, 
    function (session) {
        var reply = createEvent("changeBackground", session.message.text, session.message.address);
        session.endDialog(reply);
    }
);

const createEvent = (eventName, value, address) => {
    var msg = new builder.Message().address(address);
    msg.data.type = "event";
    msg.data.name = eventName;
    msg.data.value = value;
    return msg;
}
```

同様に、ボットはクライアントからのイベントもリッスンします。 この例では、ボットが `name="buttonClicked"` のイベントを受け取った場合、"I see that you clicked a button" (ボタンをクリックしましたね) というメッセージをユーザーに送信します。

```javascript
bot.on("event", function (event) {
    var msg = new builder.Message().address(event.address);
    msg.data.textLocale = "en-us";
    if (event.name === "buttonClicked") {
        msg.data.text = "I see that you clicked a button.";
    }
    bot.send(msg);
})
```

## <a name="additional-resources"></a>その他のリソース

- [Direct Line API][directLineAPI]
- <a href="https://github.com/Microsoft/BotFramework-WebChat" target="_blank">Microsoft Bot Framework WebChat コントロール</a>
- <a href="https://github.com/Microsoft/BotFramework-WebChat/blob/master/samples/backchannel/index.html" target="_blank">バックチャネルのサンプル</a>
- <a href="https://github.com/ryanvolum/backChannelBot" target="_blank">バックチャネル ボット</a>

[directLineAPI]: https://docs.botframework.com/en-us/restapi/directline3/#navtitle
