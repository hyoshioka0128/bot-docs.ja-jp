---
title: テキスト メッセージを送受信する | Microsoft Docs
description: Bot Framework SDK 内でテキスト メッセージを送受信する方法について説明します。
keywords: メッセージの送信, メッセージ アクティビティ, 単純なテキスト メッセージ, メッセージ, テキスト メッセージ, メッセージの受信
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 11/08/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 9cfe077c8d8573145625b211c3c1ca05a6a21e19
ms.sourcegitcommit: b15cf37afc4f57d13ca6636d4227433809562f8b
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/11/2019
ms.locfileid: "54224817"
---
# <a name="send-and-receive-text-message"></a>テキスト メッセージを送受信する 

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

ボットでユーザーと通信し、同様にボットで通信を受け取る主要な方法は、**メッセージ** アクティビティを利用することです。 メッセージにはプレーンテキストで構成されている単純なものもあれば、カードや添付など、よりリッチなコンテンツが含まれるものもあります。 ボットのターン ハンドラーでは、ユーザーからメッセージを受け取ります。そこからユーザーに応答を送信できます。 ターン コンテキスト オブジェクトによって、ユーザーにメッセージを戻すためのメソッドが与えられます。 この記事では、単純なテキスト メッセージを送信する方法について説明します。

## <a name="send-a-text-message"></a>テキスト メッセージの送信

単純なテキスト メッセージを送信するには送信する文字列をアクティビティとして指定します。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

ボットの `OnTurnAsync` メソッドで、ターン コンテキスト オブジェクトの `SendActivityAsync` メソッドを使用してメッセージ応答を 1 つ送信します。 オブジェクトの `SendActivitiesAsync` メソッドを使用して一度に複数の応答を送信することもできます。

```cs
await turnContext.SendActivityAsync($"Welcome!");
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

ボットの `onTurn` ハンドラーで、ターン コンテキスト オブジェクトの `sendActivity` メソッドを使用し、メッセージ応答を 1 つ送信します。 オブジェクトの `sendActivities` メソッドを使用して一度に複数の応答を送信することもできます。

```javascript
await context.sendActivity("Welcome!");
```
---
## <a name="receive-a-text-message"></a>テキスト メッセージの受信

単純なテキスト メッセージを受信するには、"*アクティビティ*" オブジェクトの "*テキスト*" プロパティを使用します。 

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

ボットの `OnTurnAsync` メソッドで、次のコードを使用してメッセージを受信します。 

```cs
var responseMessage = turnContext.Activity.Text;
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

ボットの `OnTurnAsync` メソッドで、次のコードを使用してメッセージを受信します。 
```javascript
let text = turnContext.activity.text;
```
---


## <a name="additional-resources"></a>その他のリソース
一般的なアクティビティ処理の詳細については、[アクティビティの処理](~/v4sdk/bot-builder-basics.md#the-activity-processing-stack)に関するページをご覧ください。 よりリッチなコンテンツを送信する場合、[リッチ メディア](bot-builder-howto-add-media-attachments.md)添付ファイルを追加する方法をご覧ください。
