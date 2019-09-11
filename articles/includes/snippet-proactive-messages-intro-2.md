---
ms.openlocfilehash: 4108f02a350d7e02d66db06cd7e65fa9c7a98e7d
ms.sourcegitcommit: fa6e775dcf95a4253ad854796f5906f33af05a42
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/16/2019
ms.locfileid: "70297354"
---
**アドホック型のプロアクティブ メッセージ**は、最も単純な種類のプロアクティブ メッセージです。
ボットは会話が開始されると単純に会話にメッセージを差し挟みます。つまり、ユーザーが現在、ボットと別の話題で会話中であり、話題を変えるつもりがない場合でも考慮しません。

通知をより円滑に処理するには、会話フローに通知を統合するための他の方法を検討してください (会話の状態にフラグを設定する方法や、通知をキューに追加する方法など)。

<!--Snip
A **dialog-based proactive message** is more complex than an ad hoc proactive message. 
Before it can inject this type of proactive message into the conversation, 
the bot must identify the context of the existing conversation and decide how (or if)
it will resume that conversation after the message interrupts. 

For example, consider a bot that needs to initiate a survey at a given point in time. 
When that time arrives, the bot stops the existing conversation with the user and 
redirects the user to a `SurveyDialog`. 
The `SurveyDialog` is added to the top of the dialog stack and takes control of the conversation. 
When the user finishes all required tasks at the `SurveyDialog`, the `SurveyDialog` closes,
 returning control to the previous dialog, where the user can continue with the prior topic of conversation.

A dialog-based proactive message is more than just simple notification. 
In sending the notification, the bot changes the topic of the existing conversation. 
It then must decide whether to resume that conversation later, or to abandon that conversation altogether by resetting the dialog stack. 
/Snip-->