---
title: アクティビティの概要 | Microsoft Docs
description: Bot Builder SDK for .NET 内で使用できるさまざまなアクティビティの種類について説明します。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: f7fe3181a4c361b47a7ef6fbdf815b4c495c6f76
ms.sourcegitcommit: 67445b42796d90661afc643c6bb6533e9a662cbc
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 08/06/2018
ms.locfileid: "39574638"
---
# <a name="activities-overview"></a>アクティビティの概要

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

[!INCLUDE [Activity concept overview](../includes/snippet-dotnet-concept-activity.md)]

## <a name="activity-types-in-the-bot-builder-sdk-for-net"></a>Bot Builder SDK for .NET のアクティビティの種類

次のアクティビティの種類は、Bot Builder SDK for .NET でサポートされています。

| Activity.Type | インターフェイス | 説明 |
|------|------|------|
| [message](#message) | IMessageActivity | ボットとユーザー間の通信を表します。 |
| [conversationUpdate](#conversationupdate) | IConversationUpdateActivity | ボットが会話に追加されたこと、他のメンバーが会話に追加された、または会話から削除されたこと、会話のメタデータが変更されたことを示します。 |
| [contactRelationUpdate](#contactrelationupdate) | IContactRelationUpdateActivity | ユーザーの連絡先リストに対してボットが追加または削除されたことを示します。 |
| [typing](#typing) | ITypingActivity | 会話の相手側のユーザーまたはボットが応答をコンパイルしていることを示します。 | 
| [ping](#ping) | 該当なし | ボットのエンドポイントがアクセス可能かどうかを判断しようとしていることを表します。 | 
| [deleteUserData](#deleteuserdata) | 該当なし | ボットに保存されている可能性のあるユーザー データを削除するようにユーザーが要求したことをボットに示します。 |
| [endOfConversation](#endofconversation) | IEndOfConversationActivity | 会話の終了を示します。 |
| [event](#event) | IEventActivity | ボットに送信される、ユーザーに表示されない通信を表します。 |
| [invoke](#invoke) | IInvokeActivity | ボットに送信される、特定の操作の実行を要求する通信を表します。 このアクティビティの種類は、Microsoft Bot Framework による内部使用のために予約されてます。 |
| [messageReaction](#messagereaction) | IMessageReactionActivity | ユーザーが既存のアクティビティに反応したことを示します。 たとえば、ユーザーがメッセージの "いいね！" ボタンをクリックしたような場合です。 |

## <a name="message"></a>message

ボットは、**message** アクティビティを送信してユーザーに情報を伝達し、ユーザーから **message** アクティビティを受信します。 プレーン テキストだけで構成されているシンプルなメッセージもあれば、[読み上げテキスト](bot-builder-dotnet-text-to-speech.md)、[推奨されるアクション](bot-builder-dotnet-add-suggested-actions.md)、[メディアの添付ファイル](bot-builder-dotnet-add-media-attachments.md)、[リッチ カード](bot-builder-dotnet-add-rich-card-attachments.md)、[チャネル固有のデータ](bot-builder-dotnet-channeldata.md)など、よりリッチなコンテンツを含むメッセージもあります。 一般的に使用されるメッセージのプロパティについては、「[メッセージの作成](bot-builder-dotnet-create-messages.md)」をご参照ください。

## <a name="conversationupdate"></a>conversationUpdate

ボットは、ボットが会話に追加されたり、他のメンバーが会話に追加または会話から削除されたり、会話メタデータが変更されたりするたびに、**conversationUpdate** アクティビティを受信します。 

メンバーが会話に追加された場合、アクティビティの `MembersAdded` プロパティには、新しいメンバーを示す `ChannelAccount` オブジェクトの配列が含まれます。 

ボットが会話に追加されたか (つまり、新しいメンバーの 1 つか) どうかを判断するには、アクティビティの `Recipient.Id` 値 (つまり、ボットの ID) が、`MembersAdded` 配列のアカウントのいずれかの `Id` プロパティと一致するかどうかを評価します。

メンバーが会話から削除された場合、`MembersRemoved` プロパティには削除されたメンバーを示す `ChannelAccount` オブジェクトの配列が含まれます。 

> [!TIP]
> ユーザーが会話に参加したことを示す **conversationUpdate** アクティビティをボットが受信した場合、そのユーザーにウェルカム メッセージを送信して応答することを選択できます。 

## <a name="contactrelationupdate"></a>contactRelationUpdate

ボットがユーザーの連絡先リストに対して追加または削除されるたびに、ボットは **contactRelationUpdate** アクティビティを受信します。 アクティビティの `Action` プロパティ (追加 | 削除) の値は、ボットがユーザーの連絡先リストに対して追加または削除されているかを示します。

## <a name="typing"></a>typing

ボットは **typing** アクティビティを受信して、ユーザーが応答を入力中であることを示します。 ボットは、**typing** アクティビティを送信して、要求を満たすため、または応答をコンパイルするためにボットが動作中であることをユーザーに示すことがあります。 

## <a name="ping"></a>ping

ボットは **ping** アクティビティを受信して、エンドポイントにアクセス可能かどうかを判断します。 ボットは、HTTP 状態コード 200 (OK)、403 (許可されていません) または 401 (権限がありません) で応答する必要があります。

## <a name="deleteuserdata"></a>deleteUserData

ボットは、ボットが以前に保持していたユーザーのデータを削除するようにユーザーが要求すると、**deleteUserData** アクティビティを受信します。 ボットがこの種類のアクティビティを受信した場合、ボットはその要求を実行したユーザー用に以前保存した個人を特定できる情報 (PII) を削除する必要があります。

## <a name="endofconversation"></a>endOfConversation 

ボットは **endOfConversation** アクティビティを受信して、ユーザーが会話を終了したことを示します。 ボットは **endOfConversation** アクティビティを送信して、ユーザーが会話を終了したことを示すことがあります。 

## <a name="event"></a>event

ボットは、外部プロセスまたは外部サービスから、ボットに情報を通信することを望むが、その情報をユーザーには表示しないことを示す、**event** アクティビティを受信する場合があります。 通常、**event** アクティビティの送信者は、ボットがいかなる方法でも受信確認することを望みません。

## <a name="invoke"></a>invoke

ボットは、ボットに対する特定の操作の実行要求を表す **invoke** アクティビティを受信することがあります。 通常、**invoke** アクティビティの送信者は、ボットが HTTP 応答で受信確認することを期待しています。 このアクティビティの種類は、Microsoft Bot Framework による内部使用のために予約されてます。

## <a name="messagereaction"></a>messageReaction

一部のチャネルは、ユーザーが既存のアクティビティに反応したときに、**messageReaction** アクティビティをボットに送信します。 たとえば、ユーザーがメッセージの "いいね！" ボタンをクリックしたような場合です。 **ReplyToId** プロパティは、ユーザーが反応したアクティビティを示します。

**messageReaction** アクティビティは、チャネルで定義されている任意の数の **messageReactionTypes** に対応することができます。 たとえば、チャネルは反応の種類として "Like" や "PlusOne" を送信できます。 

## <a name="additional-resources"></a>その他のリソース

- [アクティビティを送受信する](bot-builder-dotnet-connector.md)
- [メッセージの作成](bot-builder-dotnet-create-messages.md)
- <a href="https://docs.botframework.com/en-us/csharp/builder/sdkreference/dc/d2f/class_microsoft_1_1_bot_1_1_connector_1_1_activity.html" target="_blank">Activity クラス</a>
