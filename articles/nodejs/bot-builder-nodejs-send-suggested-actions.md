---
title: メッセージへの推奨されるアクションの追加 | Microsoft Docs
description: Bot Builder SDK for Node.js を使用してメッセージ内で推奨されるアクションを送信する方法について説明します。
author: v-ducvo
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 06/06/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 15313b6938c01fe84d4fea93c3dc9f607eeb95d2
ms.sourcegitcommit: 2dc75701b169d822c9499e393439161bc87639d2
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 08/24/2018
ms.locfileid: "42904299"
---
# <a name="add-suggested-actions-to-messages"></a>メッセージへの推奨されるアクションの追加

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-add-suggested-actions.md)
> - [Node.js](../nodejs/bot-builder-nodejs-send-suggested-actions.md)
> - [REST](../rest-api/bot-framework-rest-connector-add-suggested-actions.md)

[!INCLUDE [Introduction to suggested actions](../includes/snippet-suggested-actions-intro.md)]

> [!TIP]
> さまざまなチャネルにおける推奨されるアクションの外観と動作を確認するには、[Channel Inspector][channelInspector] を使用します。

## <a name="suggested-actions-example"></a>推奨されるアクションの例

推奨されるアクションをメッセージに追加するには、メッセージの `suggestedActions` プロパティを、ユーザーに表示されるボタンを表す[カード アクション][ICardAction]の一覧に設定します。

次のコード例は、3 つの推奨されるアクションをユーザーに提示するメッセージの送信方法を示しています。

[!code-javascript[Send suggested actions](../includes/code/node-send-suggested-actions.js#sendSuggestedActions)]

ユーザーが推奨されるアクションのいずれかをタップすると、対応するアクションの `value` を含むメッセージがユーザーからボットに送信されます。

`value` は、`imBack` メソッドによって、使用しているチャネルのチャット ウィンドウに投稿されることに注意してください。 この結果を望まない場合は、`postBack` メソッドを使用できます。これにより選択内容がご自身のボットにポスト バックされますが、選択内容はチャット ウィンドウに表示されません。 ただし、一部のチャネルでは `postBack` がサポートされていません。この場合、このメソッドは `imBack` と同じように動作します。

## <a name="additional-resources"></a>その他のリソース

* [Channel Inspector を使用して機能をプレビューする][inspector]
* [IMessage][IMessage]
* [ICardAction][ICardAction]
* [session.send][SessionSend]

[IMessage]: http://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.imessage

[SessionSend]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session.html#send

[ICardAction]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.icardaction.html

[inspector]: ../bot-service-channel-inspector.md

[channelInspector]: ../bot-service-channel-inspector.md
