---
title: 状態の保存とデータへのアクセス | Microsoft Docs
description: 状態マネージャー、会話状態、およびユーザー状態の Bot Builder SDK 内での意味について説明します。
keywords: LUIS, 会話状態, ユーザー状態, ストレージ, 状態の管理
author: DeniseMak
ms.author: v-demak
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 02/15/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 56814ab12a85d18e52b0d5ec83fd81682f3b9f60
ms.sourcegitcommit: f95702d27abbd242c902eeb218d55a72df56ce56
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/19/2018
ms.locfileid: "39304625"
---
# <a name="save-state-and-access-data"></a>状態の保存とデータへのアクセス
[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

ボットを適切に設計するための鍵は、メッセージ交換のコンテキストを追跡することです。そうすることで、以前の質問に対する回答などの事柄をボットで記憶することができます。
ボットの使用目的によっては、会話の有効期間よりも長い間、状態を追跡または情報を保存することが必要な場合もあります。
ボットの*状態*は、受信メッセージに適切に応答するためにボットが記憶する情報です。 Bot Builder SDK は、ユーザーまたは会話に関連付けられたオブジェクトとして状態データを保存および取得するためのクラスを提供します。

* **会話プロパティ**は、ボットがユーザーと交わしている現在の会話をボットが追跡するために役立ちます。 ボットがステップのシーケンスを完了する、または会話トピックを切り替える必要がある場合、会話プロパティを使用してシーケンス内のステップを管理、または現在のトピックを追跡することができます。 会話プロパティは現在の会話の状態を反映するので、通常はセッションの終了時、ボットが_会話終了_アクティビティを受信したときにプロパティをクリアします。
* **ユーザー プロパティ**は、ユーザーの以前の会話がどこで中断されたかを判断したり、戻ってきたユーザーにそのユーザーの名前を使って挨拶したりするなど、さまざまな目的で使用できます。 ユーザーの設定を保存すると、次回チャットするときにその情報を使用して会話をカスタマイズできます。 たとえば、ユーザーが関心を持っているトピックに関するニュース記事についてユーザーに通知したり、予定が利用可能になった場合にユーザーに通知したりできます。 ボットが_ユーザー データの削除_アクティビティを受信した場合は、ユーザー プロパティをクリアしてください。

[Storage](bot-builder-howto-v4-storage.md) を使用して永続的ストレージの読み書きができます。 これにより、共有リソースの更新、RSVP や投票の記録、天気の履歴データの読み取りなどをボットで実行できます。 アプリがその目的を達成するためにストレージを使用するのと同様に、ボットもユーザーとの会話内でストレージを使用できます。

<!-- 
*Conversation state* pertains to the current conversation that the user is having with your bot. When the conversation ends, your bot deletes this data.

You can also store *user state* that persists after a conversation ends. For example, if you store a user's preferences, you can use that information to customize the conversation the next time you chat. For example, you might alert the user to a news article about a topic that interests her, or alert a user when an appointment becomes available. 
-->

<!-- You should generally avoid saving state using a global variable or function closures.
Doing so will create issues when you want to scale out your bot. Instead, use the conversation state and user state middleware that the BotBuilder SDK provides --> 


## <a name="types-of-underlying-storage"></a>基盤となるストレージの種類

SDK は、会話とユーザーの状態を保持するためのボット状態マネージャー ミドルウェアを提供します。 状態にはボットのコンテキストを使用してアクセスできます。 この状態マネージャーは、基盤となるデータ ストレージとして Azure Table Storage、ファイル ストレージ、またはメモリ ストレージを使用できます。 独自のストレージ コンポーネントをボット用に作成することもできます。

Azure Table Storage を使用して構築するボットは、複数のコンピュート ノード間でステートレスかつスケーラブルな設計にすることができます。

> [!NOTE] 
> ファイル ストレージとメモリ ストレージはノード間でスケーリングしません。

## <a name="writing-directly-to-storage"></a>ストレージへの直接書き込み

Bot Builder SDK を使用して、ミドルウェアやボット コンテキストは使用せずに、ストレージのデータを直接読み書きすることもできます。 ボットの会話フローの外にあるソースからのデータをボットが使用する場合は、この方法が適切な可能性があります。

たとえば、ユーザーが天気予報を尋ねるボットで、外部データベースからの読み取りによって指定日の天気予報を取得するとします。 天気データベースの内容はユーザーの情報または会話のコンテキストに依存しないため、状態マネージャーを使用する代わりに、単純にストレージから予報を直接読み取ることができます。  例については「[ストレージに直接書き込む方法](bot-builder-howto-v4-storage.md)」を参照してください。

## <a name="next-steps"></a>次の手順

次に、アクティビティが処理されるしくみの詳細と、ボットでアクティビティに応答する方法を理解しましょう。

> [!div class="nextstepaction"]
> [アクティビティの処理](bot-builder-concept-activity-processing.md)

## <a name="additional-resources"></a>その他のリソース

- [How to save state](bot-builder-howto-v4-state.md)(状態を保存する方法)
- [ストレージに直接書き込む方法](bot-builder-howto-v4-storage.md)
