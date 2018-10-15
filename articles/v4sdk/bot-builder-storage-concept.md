---
redirect_url: /bot-framework/bot-builder-howto-v4-state
ms.openlocfilehash: e5da105e32ae748383d376f90afd9aebbf4c7aa5
ms.sourcegitcommit: 3bf3dbb1a440b3d83e58499c6a2ac116fe04b2f6
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/23/2018
ms.locfileid: "46708118"
---
<a name="--"></a><!--
---
title: 状態とストレージ | Microsoft Docs description: 状態マネージャー、会話状態、およびユーザー状態の Bot Builder SDK 内での意味について説明します。
keywords: LUIS, 会話状態, ユーザー状態, ストレージ, 状態の管理 author: DeniseMak ms.author: v-demak manager: kamrani ms.topic: article ms.prod: bot-framework ms.date: 02/15/2018 monikerRange: 'azure-bot-service-4.0'
---

# <a name="state-and-storage"></a>状態とストレージ
[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

ボットを適切に設計するための鍵は、メッセージ交換のコンテキストを追跡することです。そうすることで、以前の質問に対する回答などの事柄をボットで記憶することができます。
ボットの使用目的によっては、会話の有効期間よりも長い間、状態を追跡または情報を保存することが必要な場合もあります。
ボットの*状態*は、受信メッセージに適切に応答するためにボットが記憶する情報です。 Bot Builder SDK は、ユーザーまたは会話に関連付けられたオブジェクトとして状態データを保存および取得するためのクラスを提供します。

* **会話プロパティ**は、ボットがユーザーと交わしている現在の会話をボットが追跡するために役立ちます。 ボットがステップのシーケンスを完了する、または会話トピックを切り替える必要がある場合、会話プロパティを使用してシーケンス内のステップを管理、または現在のトピックを追跡することができます。 会話プロパティには現在の会話の状態が反映されるため、通常は、会話の終了時、ボットが "_会話終了_" アクティビティを受信したときにプロパティをクリアします。
* **ユーザー プロパティ**は、ユーザーの以前の会話がどこで中断されたかを判断したり、戻ってきたユーザーにそのユーザーの名前を使って挨拶したりするなど、さまざまな目的で使用できます。 ユーザーの設定を保存すると、次回チャットするときにその情報を使用して会話をカスタマイズできます。 たとえば、ユーザーが関心を持っているトピックに関するニュース記事についてユーザーに通知したり、予定が利用可能になった場合にユーザーに通知したりできます。 ボットが_ユーザー データの削除_アクティビティを受信した場合は、ユーザー プロパティをクリアしてください。

[Storage](bot-builder-howto-v4-storage.md) を使用して永続的ストレージの読み書きができます。 これにより、共有リソースの更新、RSVP や投票の記録、天気の履歴データの読み取りなどをボットで実行できます。 アプリがその目的を達成するためにストレージを使用するのと同様に、ボットもユーザーとの会話内でストレージを使用できます。

<!-- 
*Conversation state* pertains to the current conversation that the user is having with your bot. When the conversation ends, your bot deletes this data.

You can also store *user state* that persists after a conversation ends. For example, if you store a user's preferences, you can use that information to customize the conversation the next time you chat. For example, you might alert the user to a news article about a topic that interests her, or alert a user when an appointment becomes available. 
-->

<!-- You should generally avoid saving state using a global variable or function closures.
Doing so will create issues when you want to scale out your bot. Instead, use the conversation state and user state middleware that the BotBuilder SDK provides --> 

<!--
## Types of underlying storage

The SDK provides bot state manager middleware to persist conversation and user state. State can be accessed using the bot's context. This state manager can use Azure Table Storage, file storage, or memory storage as the underlying data storage. You can also create your own storage components for your bot.

Bots built using Azure Table Storage can be designed to be stateless and scalable across multiple compute nodes.

> [!NOTE] 
> File and memory storage won't scale across nodes.

## Writing directly to storage

You can also use the Bot Builder SDK to read and write data directly to storage, without using middleware or without using the bot context. This can be appropriate to data that your bot uses, that comes from a source outside your bot's conversation flow.

For example, let's say your bot allows the user to ask for the weather report, and your bot retrieves the weather report for a specified date, by reading it from an external database. The content of the weather database isn't dependent on user information or the conversation context, so you could just read it directly from storage instead of using the state manager.  See [How to write directly to storage](bot-builder-howto-v4-storage.md) for an example.

## Next steps

Next, lets get into how activities are processed, in depth, and how we respond to them.

> [!div class="nextstepaction"]
> [Activity Processing](bot-builder-concept-activity-processing.md)

## Additional resources

- [How to save state](bot-builder-howto-v4-state.md)
- [How to write directly to storage](bot-builder-howto-v4-storage.md)

-->
