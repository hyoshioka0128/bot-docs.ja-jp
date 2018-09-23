---
title: メッセージを送信する | Microsoft Docs
description: Bot Builder SDK 内でメッセージを送信する方法について説明します。
keywords: メッセージを送信する, メッセージ アクティビティ, シンプル テキスト メッセージ, 音声, 音声メッセージ
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 08/23/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: b56ad56e19691cfbd7b39606832ed10fce951aa3
ms.sourcegitcommit: f89ed979eb6321232fb21100ef376d9b0d5113c9
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 08/24/2018
ms.locfileid: "42914592"
---
# <a name="send-messages"></a>メッセージを送信する

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

ボットでユーザーと通信し、同様にボットで通信を受け取る主要な方法は、**メッセージ** アクティビティを利用することです。 メッセージにはプレーンテキストで構成されている単純なものもあれば、カードや添付など、よりリッチなコンテンツが含まれるものもあります。 ボットのターン ハンドラーでは、ユーザーからメッセージを受け取ります。そこからユーザーに応答を送信できます。 [ターン コンテキスト](bot-builder-concept-activity-processing.md#turn-context) オブジェクトによって、ユーザーにメッセージを戻すためのメソッドが与えられます。 一般的なアクティビティ処理に関する詳細については、「[アクティビティの処理](bot-builder-concept-activity-processing.md)」を参照してください。

この記事では、単純なテキストと音声メッセージを送信する方法について説明します。 よりリッチなコンテンツを送信する場合、[リッチ メディア添付ファイルを追加する](bot-builder-howto-add-media-attachments.md)方法をご覧ください。 プロンプト オブジェクトを使用する方法の詳細について、[ユーザーに入力を求める](bot-builder-prompts.md)方法をご覧ください。

## <a name="send-a-simple-text-message"></a>単純なテキスト メッセージを送信する

単純なテキスト メッセージを送信するには送信する文字列をアクティビティとして指定します。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

ボットの **OnTurn** メソッドで、ターン コンテキスト オブジェクトの **SendActivity** メソッドを使用し、メッセージ応答を 1 つ送信します。 オブジェクトの **SendActivities** メソッドを使用し、一度に複数の応答を送信することもできます。

```cs
await context.SendActivity("Greetings from sample message.");
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

ボットのターン ハンドラーで、ターン コンテキスト オブジェクトの **sendActivity** メソッドを使用し、メッセージ応答を 1 つ送信します。 オブジェクトの **sendActivities** メソッドを使用し、一度に複数の応答を送信することもできます。

```javascript
await context.sendActivity("Greetings from sample message.");
```

---

## <a name="send-a-spoken-message"></a>音声メッセージを送信する

特定のチャネルは音声対応ボットに対応しており、ユーザーに話しかけることができます。 メッセージには、文字コンテンツと音声コンテンツの両方を含めることができます。

> [!NOTE]
> 音声に対応していないチャネルについては、音声コンテンツは無視されます。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

オプションの **speak** パラメーターを使用し、応答の一部として音声テキストを提供します。

```cs
await context.SendActivity(
    "This is the text to be displayed.",
    "This is the text to be spoken.");
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

音声を追加するには、メッセージをビルドする `Microsoft.Bot.Builder.MessageFactory` が必要です。 `MessageFactory` は[リッチ メディア](bot-builder-howto-add-media-attachments.md)で使用されることが多く、リッチ メディアではもう少し多く説明されますが、ここでは単純にそれを必要とし、使用します。

```javascript
// Require MessageFactory from botbuilder
const {MessageFactory} = require('botbuilder');

const basicMessage = MessageFactory.text('This is the text that will be displayed.', 'This is the text that will be spoken.');
await context.sendActivity(basicMessage);
```

---