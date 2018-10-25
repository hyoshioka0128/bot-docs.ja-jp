---
title: メッセージへのリッチ カード添付ファイルの追加 | Microsoft Docs
description: Bot Builder SDK for .NET を使用してメッセージにリッチなカードを追加する方法について説明します。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: f5687cc7faf4201485ced9535f2e98b0b4c2225a
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/24/2018
ms.locfileid: "49998179"
---
# <a name="add-rich-card-attachments-to-messages"></a>メッセージにリッチ カード添付ファイルを追加する

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-add-rich-card-attachments.md)
> - [Node.js](../nodejs/bot-builder-nodejs-send-rich-cards.md)
> - [REST](../rest-api/bot-framework-rest-connector-add-rich-cards.md)

ユーザーとボットの間のメッセージ交換には、リストまたはカルーセルとしてレンダリングされる 1 つまたは複数のリッチ カードを含めることができます。 <a href="https://docs.botframework.com/en-us/csharp/builder/sdkreference/dc/d2f/class_microsoft_1_1_bot_1_1_connector_1_1_activity.html" target="_blank">Activity</a> オブジェクトの `Attachments` プロパティには、メッセージ内のリッチ カードやメディア添付ファイルを表す <a href="https://docs.microsoft.com/en-us/dotnet/api/microsoft.bot.connector.attachments?view=botconnector-3.12.2.4" target="_blank">Attachment</a> オブジェクトの配列が格納されます。 

> [!NOTE]
> メッセージにメディア添付ファイルを追加する方法については、「[Add media attachments to messages](bot-builder-dotnet-add-media-attachments.md)」 (メッセージにメディア添付ファイルを追加する) を参照してください。

## <a name="types-of-rich-cards"></a>リッチ カードの種類

Bot Framework では、現在 8 種類のリッチ カードがサポートされています。 

| カードの種類 | 説明 |
|----|----|
| <a href="/adaptive-cards/get-started/bots">アダプティブ カード</a> | テキスト、音声、画像、ボタン、および入力フィールドの任意の組み合わせを含めることができる、カスタマイズ可能なカード。 [チャネルごとのサポート](/adaptive-cards/get-started/bots#channel-status)に関するページをご覧ください。  |
| [アニメーション カード][animationCard] | アニメーション Gif または短い動画を再生できるカード。 |
| [オーディオ カード][audioCard] | オーディオ ファイルを再生できるカード。 |
| [ヒーロー カード][heroCard] | 通常 1 つの大きな画像、1 つ以上のボタン、およびテキストが含まれるカード。 |
| [サムネイル カード][thumbnailCard] | 通常 1 つのサムネイル画像、1 つ以上のボタン、およびテキストが含まれるカード。 |
| [領収書カード][receiptCard] | ボットが領収書をユーザーに提供できるようにするカード。 通常は、受信確認に含める項目の一覧、税金と合計の情報、およびその他のテキストが含まれます。 |
| [サインイン カード][signinCard] | ボットでユーザーのサインインを要求できるようにするカード。 通常は、テキストと、ユーザーがクリックしてサインイン プロセスを開始できる 1 つまたは複数のボタンが含まれます。 |
| [動画カード][videoCard] | ビデオを再生できるカード。 |

> [!TIP]
> 複数のリッチ カードをリスト形式で表示するには、アクティビティの `AttachmentLayout` プロパティを "list" に設定します。 複数のリッチ カードをカルーセル形式で表示するには、アクティビティの `AttachmentLayout` プロパティを "carousel" に設定します。 カルーセル形式がチャネルでサポートされていない場合、`AttachmentLayout` プロパティに "carousel" が指定されていたとしても、リッチ カードはリスト形式で表示されます。

## <a name="process-events-within-rich-cards"></a>リッチ カード内のイベントを処理する

リッチ カード内のイベントを処理するには、`CardAction` オブジェクトを定義して、ユーザーがボタンをクリックするか、またはカードのセクションをタップしたときのアクションを指定します。 各 `CardAction` オブジェクトには、次のプロパティが含まれています。

| プロパティ | type | 説明 | 
|----|----|----|
| type | string | アクションの種類 (下の表に示されている値のいずれか) |
| タイトル | string | ボタンのタイトル |
| イメージ | string | ボタン用のイメージ URL |
| 値 | string | 指定された種類のアクションを実行するために必要な値 |

> [!NOTE]
> アダプティブ カード内のボタンは、`CardAction` オブジェクトではなく、<a href="http://adaptivecards.io" target="_blank">アダプティブ カード</a>によって定義されているスキーマを使用して作成されます。 アダプティブ カードにボタンを追加する方法の例については、「[Add an Adaptive Card to a message](#adaptive-card)」(メッセージにアダプティブ カードを追加する) を参照してください。

次の表では、`CardAction.Type` において有効な値を一覧すると共に、種類ごとに `CardAction.Value` の想定される内容を説明します。

| CardAction.Type | CardAction.Value | 
|----|----|
| openUrl | 組み込みのブラウザーで開かれる URL |
| imBack | (ボタンをクリックまたはカードをタップしたユーザーから) ボットに送信されるメッセージのテキスト。 会話の参加者すべてが、会話をホストしているクライアント アプリケーションを介して、このメッセージ (ユーザーからボットへの) を表示することができます。 |
| postBack | (ボタンをクリックまたはカードをタップしたユーザーから) ボットに送信されるメッセージのテキスト。 クライアント アプリケーションによっては、このテキストがメッセージ フィードに表示される場合があります。そこでは、会話の参加者のすべてにそのテキストが表示されます。 |
| 以下を呼び出します。 | 電話の呼び出し先であり、**tel:123123123123** の形式となります。 |
| playAudio | 再生されるオーディオの URL |
| playVideo | 再生されるビデオの URL |
| showImage | 表示されるイメージの URL |
| downloadFile | ダウンロードされるファイルの URL |
| signin | 開始される OAuth フローの URL |

## <a name="add-a-hero-card-to-a-message"></a>メッセージにヒーロー カードを追加する

通常 1 つの大きなイメージ、1 つまたは複数のボタン、およびテキストが含まれるヒーロー カード。 

次のコード例では、カルーセル形式でレンダリングされた 3 つのヒーロー カードが含まれる応答メッセージを作成する方法を示します。 

[!code-csharp[Add HeroCard attachment](../includes/code/dotnet-add-attachments.cs#addHeroCardAttachment)]

## <a name="add-a-thumbnail-card-to-a-message"></a>メッセージにサムネイル カードを追加する

通常 1 つのサムネイル イメージ、1 つまたは複数のボタン、およびテキストが含まれるサムネイル カード。 

次のコード例では、リスト形式でレンダリングされた 2 つのサムネイル カードが含まれる応答メッセージを作成する方法を示します。 

[!code-csharp[Add ThumbnailCard attachment](../includes/code/dotnet-add-attachments.cs#addThumbnailCardAttachment)]

## <a name="add-a-receipt-card-to-a-message"></a>メッセージに受信確認カードを追加する

受信確認カードを使用すると、ボットからユーザーに受信確認を提供できるようになります。 通常は、受信確認に含める項目の一覧、税金と合計の情報、およびその他のテキストが含まれます。 

次のコード例では、受信確認カードが含まれている応答メッセージを作成する方法を示します。 

[!code-csharp[Add ReceiptCard attachment](../includes/code/dotnet-add-attachments.cs#addReceiptCardAttachment)]

## <a name="add-a-sign-in-card-to-a-message"></a>メッセージにサインイン カードを追加する

サインイン カードを使用すると、ボットでユーザーのサインインを要求できるようになります。 通常は、テキストと、ユーザーがクリックしてサインイン プロセスを開始できる 1 つまたは複数のボタンが含まれます。 

次のコード例では、サインイン カードが含まれている応答メッセージを作成する方法を示します。

[!code-csharp[Add SignInCard attachment](../includes/code/dotnet-add-attachments.cs#addSignInCardAttachment)]

## <a id="adaptive-card"></a> メッセージにアダプティブ カードを追加する

アダプティブ カードには、テキスト、音声、イメージ、ボタン、および入力フィールドの任意の組み合わせを含めることができます。 アダプティブ カードは、<a href="http://adaptivecards.io" target="_blank">アダプティブ カード</a>で指定された JSON 形式を使用して作成されます。これにより、カードのコンテンツと形式の完全な制御が可能になります。 

.NET を使用してアダプティブ カードを作成するには、`Microsoft.AdaptiveCards` NuGet パッケージをインストールします。 次に、<a href="http://adaptivecards.io" target="_blank">アダプティブ カード</a> サイト内の情報を活用して、アダプティブ カードのスキーマを理解し、アダプティブ カードの要素について調べてください。また、さまざまな構成や複雑さを備えたカードの作成に使用できる JSON のサンプルもご覧ください。 さらに、対話型のビジュアライザーを使用すると、アダプティブ カード ペイロードを設計し、カードの出力をプレビューできます。

次のコード例では、カレンダー アラーム用のアダプティブ カードが含まれるメッセージを作成する方法を示します。 

[!code-csharp[Add Adaptive Card attachment](../includes/code/dotnet-add-attachments.cs#addAdaptiveCardAttachment)]

結果のカードには、3 つのブロック (テキスト、入力フィールド (選択リスト)、および 3 つのボタン) が含まれています。

![アダプティブ カードのカレンダー アラーム](../media/adaptive-card-reminder.png)

## <a name="additional-resources"></a>その他のリソース

- [Channel Inspector を使用して機能をプレビューする][inspector]
- <a href="http://adaptivecards.io" target="_blank">アダプティブ カード</a>
- [アクティビティの概要](bot-builder-dotnet-activities.md)
- [メッセージの作成](bot-builder-dotnet-create-messages.md)
- [メッセージへのメディア添付ファイルの追加](bot-builder-dotnet-add-media-attachments.md)
- <a href="https://docs.botframework.com/en-us/csharp/builder/sdkreference/dc/d2f/class_microsoft_1_1_bot_1_1_connector_1_1_activity.html" target="_blank">Activity クラス</a>
- <a href="https://docs.microsoft.com/en-us/dotnet/api/microsoft.bot.connector.attachments?view=botconnector-3.12.2.4" target="_blank">Attachment クラス</a>

[animationCard]: /dotnet/api/microsoft.bot.connector.animationcard

[audioCard]: /dotnet/api/microsoft.bot.connector.audiocard 

[heroCard]: /dotnet/api/microsoft.bot.connector.herocard 

[thumbnailCard]: /dotnet/api/microsoft.bot.connector.thumbnailcard 

[receiptCard]: /dotnet/api/microsoft.bot.connector.receiptcard 

[signinCard]: /dotnet/api/microsoft.bot.connector.signincard 

[videoCard]: /dotnet/api/microsoft.bot.connector.videocard

[inspector]: ../bot-service-channel-inspector.md
