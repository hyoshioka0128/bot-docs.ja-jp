---
title: メッセージへの推奨されるアクションの追加 (v3 JS) - Bot Service
description: Bot Framework SDK for Node.js を使用してメッセージ内で推奨されるアクションを送信する方法について説明します。
author: v-ducvo
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 02/19/2019
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: ebad14abfdef2e274562b17ca1945d709a2c8d54
ms.sourcegitcommit: 9d77f3aff9521d819e88efd0fbd19d469b9919e7
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/16/2020
ms.locfileid: "75790387"
---
# <a name="add-suggested-actions-to-messages"></a>メッセージへの推奨されるアクションの追加

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-add-suggested-actions.md)
> - [Node.js](../nodejs/bot-builder-nodejs-send-suggested-actions.md)
> - [REST](../rest-api/bot-framework-rest-connector-add-suggested-actions.md)

[!INCLUDE [Introduction to suggested actions](../includes/snippet-suggested-actions-intro.md)]

## <a name="suggested-actions-example"></a>推奨されるアクションの例

推奨されるアクションをメッセージに追加するには、メッセージの `suggestedActions` プロパティを、ユーザーに表示されるボタンを表す[カード アクション][ICardAction]の一覧に設定します。

次のコード例は、3 つの推奨されるアクションをユーザーに提示するメッセージの送信方法を示しています。

[!code-javascript[Send suggested actions](../includes/code/node-send-suggested-actions.js#sendSuggestedActions)]

ユーザーが推奨されるアクションのいずれかをタップすると、対応するアクションの `value` を含むメッセージがユーザーからボットに送信されます。

`value` は、`imBack` メソッドによって、使用しているチャネルのチャット ウィンドウに投稿されることに注意してください。 この結果を望まない場合は、`postBack` メソッドを使用できます。これにより選択内容がご自身のボットにポスト バックされますが、選択内容はチャット ウィンドウに表示されません。 ただし、一部のチャネルでは `postBack` がサポートされていません。この場合、このメソッドは `imBack` と同じように動作します。

## <a name="additional-resources"></a>その他のリソース

- [サンプル][samples]
- [IMessage][IMessage]
- [ICardAction][ICardAction]
- [session.send][SessionSend]

[IMessage]: http://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.imessage

[SessionSend]: https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.session.html#send

[ICardAction]: https://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.icardaction.html

<!-- The inspector is no longer supported: we're redirecting to the samples for now. -->
[samples]: https://github.com/Microsoft/BotBuilder-Samples/tree/v3-sdk-samples
