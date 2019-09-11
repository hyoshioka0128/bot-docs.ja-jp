---
title: テキスト メッセージを送受信する | Microsoft Docs
description: Bot Framework SDK 内でテキスト メッセージを送受信する方法について説明します。
keywords: メッセージの送信, メッセージ アクティビティ, 単純なテキスト メッセージ, メッセージ, テキスト メッセージ, メッセージの受信
author: ivorb
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 05/23/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 002f2de104e61be3b0f7e543408c0c0d78018f9b
ms.sourcegitcommit: a6d02ec4738e7fc90b7108934740e9077667f3c5
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/04/2019
ms.locfileid: "70299252"
---
# <a name="send-and-receive-text-message"></a>テキスト メッセージを送受信する

[!INCLUDE[applies-to](../includes/applies-to.md)]

ボットでユーザーと通信し、同様にボットで通信を受け取る主要な方法は、**メッセージ** アクティビティを利用することです。 メッセージにはプレーンテキストで構成されている単純なものもあれば、カードや添付など、よりリッチなコンテンツが含まれるものもあります。 ボットのターン ハンドラーでは、ユーザーからメッセージを受け取ります。そこからユーザーに応答を送信できます。 ターン コンテキスト オブジェクトによって、ユーザーにメッセージを戻すためのメソッドが与えられます。 この記事では、単純なテキスト メッセージを送信する方法について説明します。

Markdown はほとんどのテキスト フィールドでサポートされていますが、サポート状況はチャネルごとに異なる場合があります。

メッセージを送受信する実行中のボットについては、目次の一番上にあるクイックスタートを実行するか、[ボットの動作方法に関するの記事](bot-builder-basics.md#bot-structure)を確認してください。これは、ご自身で実行する場合にご利用いただけるシンプルなサンプルへもリンクされています。

## <a name="send-a-text-message"></a>テキスト メッセージの送信

単純なテキスト メッセージを送信するには送信する文字列をアクティビティとして指定します。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

ボットのアクティビティ ハンドラーで、ターン コンテキスト オブジェクトの `SendActivityAsync` メソッドを使用し、メッセージ応答を 1 つ送信します。 オブジェクトの `SendActivitiesAsync` メソッドを使用して一度に複数の応答を送信することもできます。

```cs
await turnContext.SendActivityAsync($"Welcome!");
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

ボットのアクティビティ ハンドラーで、ターン コンテキスト オブジェクトの `sendActivity` メソッドを使用し、メッセージ応答を 1 つ送信します。 オブジェクトの `sendActivities` メソッドを使用して一度に複数の応答を送信することもできます。

```javascript
await context.sendActivity("Welcome!");
```
---
## <a name="receive-a-text-message"></a>テキスト メッセージの受信

単純なテキスト メッセージを受信するには、"*アクティビティ*" オブジェクトの "*テキスト*" プロパティを使用します。 

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

ボットのアクティビティ ハンドラーで、次のコードを使用してメッセージを受信します。 

```cs
var responseMessage = turnContext.Activity.Text;
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

ボットのアクティビティ ハンドラーで、次のコードを使用してメッセージを受信します。

```javascript
let text = turnContext.activity.text;
```

---

## <a name="additional-resources"></a>その他のリソース

- 一般的なアクティビティ処理の詳細については、[アクティビティの処理](~/v4sdk/bot-builder-basics.md#the-activity-processing-stack)に関するページをご覧ください。
- 書式設定の詳細については、Bot Framework のアクティビティ スキーマに関するページの[メッセージ アクティビティ](https://aka.ms/botSpecs-activitySchema#message-activity)に関するセクションをご覧ください。

## <a name="next-steps"></a>次の手順

> [!div class="nextstepaction"]
> [メッセージにメディアを追加する](./bot-builder-howto-add-media-attachments.md)
