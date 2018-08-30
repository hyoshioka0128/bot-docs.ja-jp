---
title: 入力インジケーターの送信 | Microsoft Docs
description: Bot Builder SDK for Node.js を使用して、ボットが要求を処理中であることをユーザーに伝えるために "しばらくお待ちください" というインジケーターを追加する方法をご確認ください。
author: DeniseMak
ms.author: v-demak
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 1c6b01b1cab9165b1513bfda5af182f27635290e
ms.sourcegitcommit: 2dc75701b169d822c9499e393439161bc87639d2
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 08/24/2018
ms.locfileid: "42904384"
---
# <a name="send-a-typing-indicator"></a>入力インジケーターの送信 

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

ユーザーはメッセージに対するタイムリーな応答を期待しています。 ボットがユーザーからの指示を受け取ったということをユーザーに表示することなく、サーバーの呼び出しやクエリの実行などの実行時間の長いタスクを実行すると、ユーザーはしびれを切らして追加のメッセージを送信したり、単にボットが壊れていると推測する可能性があります。
多くのチャネルは、メッセージが受信され、処理中であることをユーザに示す入力表示の送信をサポートします。


## <a name="typing-indicator-example"></a>入力インジケーターの例

次の例は、[session.sendTyping()][SendTyping] を使用して入力表示を送信する方法を示しています。  これは Bot Framework Emulator を使用してテストできます。


```javascript

// Create bot and default message handler
var bot = new builder.UniversalBot(connector, function (session) {
    session.sendTyping();
    setTimeout(function () {
        session.send("Hello there...");
    }, 3000);
});
```

また、入力インジケーターは、イメージが含まれているメッセージが誤った順序で送信されないようにメッセージ遅延を挿入する場合にも便利です。

詳細については、「[リッチ カードの送信方法](bot-builder-nodejs-send-rich-cards.md)」をご参照ください。


## <a name="additional-resources"></a>その他のリソース

* [sendTyping][SendTyping]


[SendTyping]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session#sendtyping
[IMessage]: http://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.imessage
