---
title: Bot Connector API を使用したメッセージの作成 - Bot Service
description: Bot Connector API 内の、一般的に使用されるメッセージ プロパティについて説明します。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
ms.openlocfilehash: f3f084f65c1fb4fc84a6c8c75ce36b56487ebd0d
ms.sourcegitcommit: 9d77f3aff9521d819e88efd0fbd19d469b9919e7
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/16/2020
ms.locfileid: "75789965"
---
# <a name="create-messages"></a>メッセージを作成する

ボットは、**message** 型の [Activity][] オブジェクトを送信してユーザーに情報を伝達し、同様にユーザーからの**メッセージ** アクティビティも受信します。 プレーン テキストだけで構成されているシンプルなメッセージもあれば、[読み上げテキスト](bot-framework-rest-connector-text-to-speech.md)、[推奨されるアクション](bot-framework-rest-connector-add-suggested-actions.md)、[メディアの添付ファイル](bot-framework-rest-connector-add-media-attachments.md)、[リッチ カード](bot-framework-rest-connector-add-rich-cards.md)、[チャネル固有のデータ](bot-framework-rest-connector-channeldata.md)など、よりリッチなコンテンツを含むメッセージもあります。 この記事では、一般的に使用されるメッセージ プロパティの一部について説明します。

## <a name="message-text-and-formatting"></a>メッセージ テキストと書式設定

メッセージ テキストは **plain**、**markdown**、または **xml** を使用して書式設定できます。 `textFormat` プロパティの既定の書式設定は **markdown** で、Markdown 書式設定標準を使用してテキストを解釈します。 テキスト書式のサポートのレベルは、チャネルによって異なります。 

[!INCLUDE [Channel Inspector intro](~/includes/snippet-channel-inspector.md)]

[Activity][] オブジェクトの `textFormat` プロパティを使用して、テキストの書式を指定することができます。 たとえば、プレーン テキストのみを含む基本的なメッセージを作成するには、`Activity` オブジェクトの `textFormat` プロパティを **plain** に設定し、`text` プロパティをメッセージの内容に設定し、`locale` プロパティを送信者のロケールに設定します。 

## <a name="attachments"></a>[Attachments]

[Activity][] オブジェクトの `attachments` プロパティは、単純なメディア添付ファイル (画像、オーディオ、ビデオ、ファイル) やリッチ カードの送信に使用できます。 詳細については、「[メッセージへのメディア添付ファイルの追加](bot-framework-rest-connector-add-media-attachments.md)」および「[メッセージへのリッチ カードの追加](bot-framework-rest-connector-add-rich-cards.md)」を参照してください。

## <a name="entities"></a>エンティティ

[Activity][] オブジェクトの `entities` プロパティは、拡張可能な <a href="http://schema.org/" target="_blank">schema.org</a> オブジェクトの配列であり、チャネルとボットの間で共通のコンテキスト メタデータを交換するために使用します。

### <a name="mention-entities"></a>メンション エンティティ

多くのチャネルは、ボットまたはユーザーが会話のコンテキスト内で誰かを "メンション" する能力をサポートしています。 メッセージでユーザーをメンションするには、メッセージの `entities` プロパティに [Mention][] オブジェクトを設定します。 

### <a name="place-entities"></a>場所エンティティ

メッセージ内で<a href="https://schema.org/Place" target="_blank">場所に関連した情報</a>を伝達するには、メッセージの `entities` プロパティに [Place][] オブジェクトを設定します。 

## <a name="channel-data"></a>チャネル データ

[Activity][] オブジェクトの `channelData` プロパティは、チャネル固有の機能を実装するために使用できます。 詳細については、「[チャネル固有の機能の実装](bot-framework-rest-connector-channeldata.md)」を参照してください。

## <a name="text-to-speech"></a>テキストを音声に変換する

[Activity][] オブジェクトの `speak` プロパティを使用して、音声に対応したチャネルでボットが話すテキストを指定できます。また、`Activity` オブジェクトの `inputHint` プロパティを使用して、クライアントのマイクの状態に影響を与えることができます。 詳細については、「[メッセージへの音声の追加](bot-framework-rest-connector-text-to-speech.md)」および「[メッセージへの入力ヒントの追加](bot-framework-rest-connector-add-input-hints.md)」を参照してください。

## <a name="suggested-actions"></a>推奨されるアクション

[Activity][] オブジェクトの `suggestedActions` プロパティは、ユーザーが入力を提供するためにタップできるボタンを表示するために使用できます。 (タップされた後でも表示されたままとなり、ユーザーがアクセスできる) リッチ カード内に表示されるボタンとは異なり、推奨される操作ウィンドウ内に表示されるボタンは、ユーザーが選択を行った後は表示されなくなります。 詳細については、「[メッセージへの推奨されるアクションの追加](bot-framework-rest-connector-add-suggested-actions.md)」を参照してください。

## <a name="additional-resources"></a>その他のリソース

- [チャネル リファレンス][ChannelInspector]
- [アクティビティの概要](https://aka.ms/botSpecs-activitySchema)
- [メッセージを送受信する](bot-framework-rest-connector-send-and-receive-messages.md)
- [メッセージへのメディア添付ファイルの追加](bot-framework-rest-connector-add-media-attachments.md)
- [メッセージへのリッチ カードの追加](bot-framework-rest-connector-add-rich-cards.md)
- [メッセージへの音声の追加](bot-framework-rest-connector-text-to-speech.md)
- [メッセージへの入力ヒントの追加](bot-framework-rest-connector-add-input-hints.md)
- [メッセージへの推奨されるアクションの追加](bot-framework-rest-connector-add-suggested-actions.md)
- [チャネル固有の機能の実装](bot-framework-rest-connector-channeldata.md)

[ChannelInspector]: ../bot-service-channels-reference.md
[textFormating]: ../bot-service-channel-inspector.md#text-formatting

[Activity]: bot-framework-rest-connector-api-reference.md#activity-object
[Mention]: bot-framework-rest-connector-api-reference.md#mention-object
[Place]: bot-framework-rest-connector-api-reference.md#place-object
