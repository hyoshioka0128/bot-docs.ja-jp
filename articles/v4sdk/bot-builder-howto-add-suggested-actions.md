---
title: 入力にボタンを使用する | Microsoft Docs
description: Bot Framework SDK for JavaScript を使用してメッセージ内で推奨されるアクションを送信する方法について説明します。
keywords: 推奨されるアクション, ボタン, 追加の入力
author: Kaiqb
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/10/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: b121a60b72551dd473a880d47868a61763e5afa0
ms.sourcegitcommit: a547192effb705e4c7d82efc16f98068c5ba218b
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/25/2019
ms.locfileid: "75491493"
---
# <a name="use-button-for-input"></a>入力にボタンを使用する

[!INCLUDE[applies-to](../includes/applies-to.md)]

お使いのボットに、ユーザーがタップして入力できるボタンを表示させることができます。 ボタンを使用すると、キーボードを使って返信を入力することなく、ユーザーが質問に回答したり、ボタンをタップするだけで選択したりできるため、ユーザー エクスペリエンスが向上します。 (タップされた後でも表示されたままとなり、ユーザーがアクセスできる) リッチ カード内に表示されるボタンとは異なり、推奨される操作ウィンドウ内に表示されるボタンは、ユーザーが選択を行った後は表示されなくなります。 このため、ユーザーが会話内の古いボタンをタップすることを防ぎ、(お客様はそのシナリオの責任を負う必要がないため) ボット開発が簡略化されます。 

## <a name="suggest-action-using-button"></a>ボタンを使用した推奨されるアクション

"*推奨されるアクション*" では、お使いのボットにボタンを表示させることができます。 会話の 1 つのターンでユーザーに表示される推奨アクション (「クイック返信」とも呼ばれます) のリストを作成できます。 

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

ここで示すソース コードは、[推奨アクションのサンプル](https://aka.ms/SuggestedActionsCSharp)に基づいています。

[!code-csharp[suggested actions](~/../botbuilder-samples/samples/csharp_dotnetcore/08.suggested-actions/Bots/SuggestedActionsBot.cs?range=87-101)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

ここで示すソース コードは、[推奨アクションのサンプル](https://aka.ms/SuggestActionsJS)に基づいています。

[!code-javascript[suggested actions](~/../botbuilder-samples/samples/javascript_nodejs/08.suggested-actions/bots/suggestedActionsBot.js?range=61-64)]


# <a name="pythontabpython"></a>[Python](#tab/python)

ここで示すソース コードは、[推奨アクションのサンプル](https://aka.ms/SuggestActionsPython)に基づいています。

[!code-python[suggested actions](~/../botbuilder-python/samples/python/08.suggested-actions/bots/suggested_actions_bot.py?range=63-81)]


---

## <a name="additional-resources"></a>その他のリソース

ここで示したソース コードの完全版には、次からアクセスできます。
- [C# のサンプル](https://aka.ms/SuggestedActionsCSharp)
- [JavaScript のサンプル](https://aka.ms/SuggestActionsJS)
- [Python のサンプル](https://aka.ms/SuggestActionsPython)

## <a name="next-steps"></a>次のステップ

> [!div class="nextstepaction"]
> [ユーザーと会話データを保存する](./bot-builder-howto-v4-state.md)
