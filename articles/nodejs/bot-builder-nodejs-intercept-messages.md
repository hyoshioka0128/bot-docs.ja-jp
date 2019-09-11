---
title: メッセージをインターセプトする | Microsoft Docs
description: Bot Framework SDK for Node.js を使用して、情報交換をインターセプトして処理することで、ログやその他のレコードを作成する方法について説明します。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 11/02/2018
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 2ca85c598d5515e8a785326ba12fd872ffce741f
ms.sourcegitcommit: a6d02ec4738e7fc90b7108934740e9077667f3c5
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/04/2019
ms.locfileid: "70299822"
---
# <a name="intercept-messages"></a>メッセージをインターセプトする

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-middleware.md)
> - [Node.JS](../nodejs/bot-builder-nodejs-intercept-messages.md)

[!INCLUDE [Introduction to message logging](../includes/snippet-message-logging-intro.md)]

## <a name="example"></a>例

次のサンプル コードは、ユーザーとボット間で交換されるメッセージを、Bot Framework SDK for Node.js の**ミドルウェア**の概念を使用してインターセプトする方法を示しています。 

最初に、受信メッセージ (`botbuilder`) と送信メッセージ (`send`) 用のハンドラーを構成します。

```javascript
server.post('/api/messages', connector.listen());
var bot = new builder.UniversalBot(connector);
bot.use({
    botbuilder: function (session, next) {
        myMiddleware.logIncomingMessage(session, next);
    },
    send: function (event, next) {
        myMiddleware.logOutgoingMessage(event, next);
    }
})
```

次に、各ハンドラーを実装して、インターセプトされた各メッセージに対するアクションを定義します。

```javascript
module.exports = {
    logIncomingMessage: function (session, next) {
        console.log(session.message.text);
        next();
    },
    logOutgoingMessage: function (event, next) {
        console.log(event.text);
        next();
    }
}
```

これで、すべての受信メッセージ (ユーザーからボットへのメッセージ) によって `logIncomingMessage` がトリガーされ、すべての送信メッセージ (ボットからユーザーへのメッセージ) によって `logOutgoingMessage`がトリガーされます。
この例では、ボットは、単純に各メッセージに関する情報を出力しますが、必要に応じて `logIncomingMessage` と `logOutgoingMessage` を更新して、各メッセージに対して実行するアクションを定義できます。 

## <a name="sample-code"></a>サンプル コード

Bot Framework SDK for Node.js を使用してメッセージをインターセプトしてログに記録する方法を示す完全なサンプルについては、GitHub で<a href="https://aka.ms/v3-js-capability-middlewareLogging" target="_blank">ミドルウェアとログ記録のサンプル</a>を参照してください。
