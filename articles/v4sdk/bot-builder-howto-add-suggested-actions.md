---
title: 入力にボタンを使用する | Microsoft Docs
description: Bot Framework SDK for JavaScript を使用してメッセージ内で推奨されるアクションを送信する方法について説明します。
keywords: 推奨されるアクション, ボタン, 追加の入力
author: Kaiqb
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 05/23/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 603b6d6fe9fed99cb31649539e518651c135241a
ms.sourcegitcommit: a6d02ec4738e7fc90b7108934740e9077667f3c5
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/04/2019
ms.locfileid: "70299304"
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

---

## <a name="additional-resources"></a>その他のリソース

ここで示したソース コードの完全版については、[CSharp サンプル](https://aka.ms/SuggestedActionsCSharp)または [JavaScript サンプル](https://aka.ms/SuggestActionsJS)を参照してください。

## <a name="next-steps"></a>次の手順

> [!div class="nextstepaction"]
> [ユーザーと会話データを保存する](./bot-builder-howto-v4-state.md)
