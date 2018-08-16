---
title: Bot Connector API を使用したメッセージの作成 | Microsoft Docs
description: Bot Connector API 内の、一般的に使用されるメッセージ プロパティについて説明します。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ms.openlocfilehash: 2fbff9c6d7fe1e06fa87e5b2695056dbc1414570
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/18/2018
ms.locfileid: "39304393"
---
# <a name="create-messages"></a>メッセージの作成

ボットは、**message** タイプの [Activity][Activity] オブジェクトを送信してユーザーに情報を伝達し、同様にユーザーからの**メッセージ** アクティビティも受信します。 プレーン テキストだけで構成されているシンプルなメッセージもあれば、[読み上げテキスト](bot-framework-rest-connector-text-to-speech.md)、[推奨されるアクション](bot-framework-rest-connector-add-suggested-actions.md)、[メディアの添付ファイル](bot-framework-rest-connector-add-media-attachments.md)、[リッチ カード](bot-framework-rest-connector-add-rich-cards.md)、[チャネル固有のデータ](bot-framework-rest-connector-channeldata.md)など、よりリッチなコンテンツを含むメッセージもあります。 この記事では、一般的に使用されるメッセージ プロパティの一部について説明します。

## <a name="message-text-and-formatting"></a>メッセージ テキストと書式設定

メッセージ テキストは **plain**、**markdown**、または **xml** を使用して書式設定できます。 `textFormat` プロパティの既定の書式設定は **markdown** で、Markdown 書式設定標準を使用してテキストを解釈します。 テキスト書式のサポートのレベルは、チャネルによって異なります。 使用する機能がターゲットのチャネルでサポートされているかどうかを確認するには、[Channel Inspector][ChannelInspector] を使用して機能をプレビューします。 

[Activity][Activity] オブジェクトの `textFormat` プロパティを使用して、テキストの書式を指定することができます。 たとえば、プレーン テキストのみを含む基本的なメッセージを作成するには、[Activity][Activity] オブジェクトの `textFormat` プロパティを **plain** に設定し、`text` プロパティをメッセージの内容に設定し、`locale` プロパティを送信者のロケールに設定します。 

一般的にサポートされているテキストの書式設定の一覧については、「[Text formatting](../bot-service-channel-inspector.md#text-formatting)」 (テキストの書式設定) を参照してください。

## <a name="attachments"></a>[添付ファイル]

[Activity][Activity] オブジェクトの `attachments` プロパティは、単純なメディア添付ファイル (画像、オーディオ、ビデオ、ファイル) やリッチ カードの送信に使用できます。 詳細については、「[メッセージへのメディア添付ファイルの追加](bot-framework-rest-connector-add-media-attachments.md)」および「[メッセージへのリッチ カードの追加](bot-framework-rest-connector-add-rich-cards.md)」を参照してください。

## <a name="entities"></a>エンティティ

[Activity][Activity] オブジェクトの `entities` プロパティは、拡張可能な <a href="http://schema.org/" target="_blank">schema.org</a> オブジェクトの配列であり、チャネルとボットの間で共通のコンテキスト メタデータを交換するために使用します。

### <a name="mention-entities"></a>メンション エンティティ

多くのチャネルは、ボットまたはユーザーが会話のコンテキスト内で誰かを "メンション" する能力をサポートしています。 メッセージでユーザーをメンションするには、メッセージの `entities` プロパティに [Mention][Mention] オブジェクトを設定します。 

### <a name="place-entities"></a>場所エンティティ

メッセージ内で<a href="https://schema.org/Place" target="_blank">場所に関連した情報</a>を伝達するには、メッセージの `entities` プロパティに [Place][Place] オブジェクトを設定します。 

## <a name="channel-data"></a>チャネル データ

[Activity][Activity] オブジェクトの `channelData` プロパティは、チャネル固有の機能を実装するために使用できます。 詳細については、「[チャネル固有の機能の実装](bot-framework-rest-connector-channeldata.md)」を参照してください。

## <a name="text-to-speech"></a>テキストを音声に変換する

[Activity][Activity] オブジェクトの `speak` プロパティを使用して、音声に対応したチャネルでボットが話すテキストを指定できます。また、[Activity][Activity] オブジェクトの `inputHint` プロパティを使用して、クライアントのマイクの状態に影響を与えることができます。 詳細については、「[メッセージへの音声の追加](bot-framework-rest-connector-text-to-speech.md)」および「[メッセージへの入力ヒントの追加](bot-framework-rest-connector-add-input-hints.md)」を参照してください。

## <a name="suggested-actions"></a>推奨されるアクション

[Activity][Activity] オブジェクトの `suggestedActions` プロパティは、ユーザーが入力を提供するためにタップできるボタンを表示するために使用できます。 (タップされた後でも表示されたままとなり、ユーザーがアクセスできる) リッチ カード内に表示されるボタンとは異なり、推奨される操作ウィンドウ内に表示されるボタンは、ユーザーが選択を行った後は表示されなくなります。 詳細については、「[メッセージへの推奨されるアクションの追加](bot-framework-rest-connector-add-suggested-actions.md)」を参照してください。

## <a name="additional-resources"></a>その他のリソース

- [Channel Inspector を使用して機能をプレビューする][ChannelInspector]
- [アクティビティの概要](bot-framework-rest-connector-activities.md)
- [メッセージを送受信する](bot-framework-rest-connector-send-and-receive-messages.md)
- [メッセージへのメディア添付ファイルの追加](bot-framework-rest-connector-add-media-attachments.md)
- [メッセージへのリッチ カードの追加](bot-framework-rest-connector-add-rich-cards.md)
- [メッセージへの音声の追加](bot-framework-rest-connector-text-to-speech.md)
- [メッセージへの入力ヒントの追加](bot-framework-rest-connector-add-input-hints.md)
- [メッセージへの推奨されるアクションの追加](bot-framework-rest-connector-add-suggested-actions.md)
- [チャネル固有の機能の実装](bot-framework-rest-connector-channeldata.md)

[Mention]: bot-framework-rest-connector-api-reference.md#mention-object
[Place]: bot-framework-rest-connector-api-reference.md#place-object
[Activity]: bot-framework-rest-connector-api-reference.md#activity-object
[ChannelInspector]: ../bot-service-channel-inspector.md
[textFormating]: ../bot-service-channel-inspector.md#text-formatting
