---
title: ユーザーと会話イベントの処理 | Microsoft Docs
description: Bot Builder SDK for Node.js を使用して、会話に参加しているユーザーなどのイベントを処理する方法について説明します。
author: DucVo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 2e8cb9661079eafa988af4a7639e3e7dc1580bfa
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/24/2018
ms.locfileid: "49998139"
---
# <a name="handle-user-and-conversation-events"></a>ユーザーと会話イベントの処理

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

この記事では、ボットを連絡先リストに追加したり、ボットが会話から削除されるときに「**Goodbye**」と発話するなど、会話に参加しているユーザーなどのイベントを処理する方法について説明します。


## <a name="greet-a-user-on-conversation-join"></a>会話に参加したユーザーにあいさつする
Bot Framework では、メンバーが会話に参加または退出するたびにボットに通知する [conversationUpdate][conversationUpdate] イベントを提供しています。 会話のメンバーになることができるのは、ユーザーまたはボットです。

次のコード スニペットでは、ボットは、会話に参加した新しいメンバーにあいさつしたり、会話から退出する場合に「**Goodbye**」と言ったりできます。

[!INCLUDE [conversationUpdate sample Node.js](../includes/snippet-code-node-conversationupdate-1.md)]

## <a name="acknowledge-add-to-contacts-list"></a>連絡先リストへの追加を認識する

[contactRelationUpdate][contactRelationUpdate] イベントは、ユーザーによって連絡先リストに追加されたことを、ボットに通知します。

[!INCLUDE [contactRelationUpdate sample Node.js](../includes/snippet-code-node-contactrelationupdate-1.md)]

## <a name="add-a-first-run-dialog"></a>初回実行時のダイアログを追加する

**conversationUpdate** および the **contactRelationUpdate** イベントは、すべてのチャネルでサポートされるわけではないので、会話に参加するユーザーにあいさつするための一般的な方法は、初回実行時のダイアログを追加することです。

次の例では、ユーザーが初見の場合は、必ずダイアログをトリガーする関数を追加しました。 アクションに [onFindAction][onFindAction] ハンドラーを指定することで、アクションがトリガーされる方法をカスタマイズできます。 

[!INCLUDE [first-run sample Node.js](../includes/snippet-code-node-first-run-dialog-1.md)]

また、[onSelectAction][onSelectAction] ハンドラーを指定することで、トリガーされた後にアクションが行う動作をカスタマイズできます。 トリガー アクションでは、[onInterrupted][onInterrupted] ハンドラーを指定して、中断が発生する前にインターセプトできます。 詳細については、「[Handle user actions](bot-builder-nodejs-dialog-actions.md)」(ユーザー アクションの処理) を参照してください。

## <a name="additional-resources"></a>その他のリソース

* [conversationUpdate][conversationUpdate]
* [contactRelationUpdate][contactRelationUpdate]

[conversationUpdate]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.iconversationupdate.html
[contactRelationUpdate]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.icontactrelationupdate.html

[onFindAction]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.itriggeractionoptions#onfindaction
[onSelectAction]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.itriggeractionoptions#onselectaction
[onInterrupted]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.itriggeractionoptions#oninterrupted

[SendTyping]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session#sendtyping
[IMessage]: http://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.imessage
[ChatConnector]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.chatconnector.html
[session_userData]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session.html#userdata
