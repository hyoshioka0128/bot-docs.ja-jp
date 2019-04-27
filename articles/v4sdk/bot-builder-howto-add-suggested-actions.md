---
title: 入力にボタンを使用する | Microsoft Docs
description: Bot Framework SDK for JavaScript を使用してメッセージ内で推奨されるアクションを送信する方法について説明します。
keywords: 推奨されるアクション, ボタン, 追加の入力
author: Kaiqb
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 4/18/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: bf0c5c0bba335c41a268d43014e925f6a9289d75
ms.sourcegitcommit: aea57820b8a137047d59491b45320cf268043861
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/22/2019
ms.locfileid: "59904975"
---
# <a name="use-button-for-input"></a>入力にボタンを使用する

[!INCLUDE[applies-to](../includes/applies-to.md)]

お使いのボットに、ユーザーがタップして入力できるボタンを表示させることができます。 ボタンを使用すると、キーボードを使って返信を入力することなく、ユーザーが質問に回答したり、ボタンをタップするだけで選択したりできるため、ユーザー エクスペリエンスが向上します。 (タップされた後でも表示されたままとなり、ユーザーがアクセスできる) リッチ カード内に表示されるボタンとは異なり、推奨される操作ウィンドウ内に表示されるボタンは、ユーザーが選択を行った後は表示されなくなります。 このため、ユーザーが会話内の古いボタンをタップすることを防ぎ、(お客様はそのシナリオの責任を負う必要がないため) ボット開発が簡略化されます。 

## <a name="suggest-action-using-button"></a>ボタンを使用した推奨されるアクション

"*推奨されるアクション*" では、お使いのボットにボタンを表示させることができます。 会話の 1 つのターンでユーザーに表示される推奨アクション (「クイック返信」とも呼ばれます) のリストを作成できます。 

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

ここで使用するソース コードについては、[GitHub](https://aka.ms/SuggestedActionsCSharp) を参照してください。

```csharp
using Microsoft.Bot.Builder;
using Microsoft.Bot.Schema;

var reply = turnContext.Activity.CreateReply("What is your favorite color?");

reply.SuggestedActions = new SuggestedActions()
{
    Actions = new List<CardAction>()
    {
        new CardAction() { Title = "Red", Type = ActionTypes.ImBack, Value = "Red" },
        new CardAction() { Title = "Yellow", Type = ActionTypes.ImBack, Value = "Yellow" },
        new CardAction() { Title = "Blue", Type = ActionTypes.ImBack, Value = "Blue" },
    },

};
await turnContext.SendActivityAsync(reply, cancellationToken: cancellationToken);
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)
ここで使用するソース コードについては、[GitHub](https://aka.ms/SuggestActionsJS) を参照してください。

```javascript
const { ActivityTypes, MessageFactory, TurnContext } = require('botbuilder');

async sendSuggestedActions(turnContext) {
    var reply = MessageFactory.suggestedActions(['Red', 'Yellow', 'Blue'], 'What is the best color?');
    await turnContext.sendActivity(reply);
}
```

---

## <a name="additional-resources"></a>その他のリソース

ここで示したソース コードの完全版については、GitHub [[C#](https://aka.ms/SuggestedActionsCSharp) | [JS](https://aka.ms/SuggestActionsJS)] を参照してください。
