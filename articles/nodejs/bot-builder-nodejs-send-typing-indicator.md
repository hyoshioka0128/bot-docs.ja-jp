---
title: 入力インジケーターの送信 - Bot Service
description: Bot Framework SDK for Node.js を使用して、ボットが要求を処理中であることをユーザーに伝えるために "しばらくお待ちください" というインジケーターを追加する方法を説明します。
author: DeniseMak
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 5f4e67818288c5dc1a5fbb78261dea11c8b3a4c5
ms.sourcegitcommit: 9d77f3aff9521d819e88efd0fbd19d469b9919e7
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/16/2020
ms.locfileid: "75790377"
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


[SendTyping]: https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.session#sendtyping
[IMessage]: http://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.imessage
