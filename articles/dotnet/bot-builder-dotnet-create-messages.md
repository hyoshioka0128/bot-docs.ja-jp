---
title: Bot Framework SDK for .NET を使用したメッセージの作成 | Microsoft Docs
description: Bot Framework SDK for .NET 内の、一般的に使用されるメッセージ プロパティについて説明します。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: bb75e49e50a479e0141000ef49d75559148fe43a
ms.sourcegitcommit: a6d02ec4738e7fc90b7108934740e9077667f3c5
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/04/2019
ms.locfileid: "70297301"
---
# <a name="create-messages"></a>メッセージを作成する

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

ボットは、**メッセージ** [アクティビティ](bot-builder-dotnet-activities.md)を送信してユーザーに情報を伝達し、同様にユーザーからの**メッセージ** アクティビティも受信します。 プレーン テキストだけで構成されているシンプルなメッセージもあれば、[読み上げテキスト](bot-builder-dotnet-text-to-speech.md)、[推奨されるアクション](bot-builder-dotnet-add-suggested-actions.md)、[メディアの添付ファイル](bot-builder-dotnet-add-media-attachments.md)、[リッチ カード](bot-builder-dotnet-add-rich-card-attachments.md)、[チャネル固有のデータ](bot-builder-dotnet-channeldata.md)など、よりリッチなコンテンツを含むメッセージもあります。 

この記事では、一般的に使用されるメッセージ プロパティの一部について説明します。

## <a name="customizing-a-message"></a>メッセージのカスタマイズ

メッセージのテキストの書式設定をさらに制御するため、[Activity](https://docs.botframework.com/csharp/builder/sdkreference/dc/d2f/class_microsoft_1_1_bot_1_1_connector_1_1_activity.html) オブジェクトを使用してカスタム メッセージを作成し、ユーザーに送信する前に必要なプロパティを設定することができます。

このサンプルでは、カスタム `message` オブジェクトを作成し、`Text`、`TextFormat`、`Local` のプロパティを設定する方法を示しています。

[!code-csharp[Set message properties](../includes/code/dotnet-create-messages.cs#setBasicProperties)]

メッセージの `TextFormat` プロパティを使用して、テキストの書式を指定することができます。 `TextFormat` プロパティは、**plain**、**markdown**、**xml** のいずれかに設定できます。 `TextFormat` の既定値は **markdown** です。 

## <a name="attachments"></a>[添付ファイル]

メッセージ アクティビティの `Attachments` プロパティは、単純なメディア添付ファイル (画像、オーディオ、ビデオ、ファイル) やリッチ カードの送受信に使用できます。 詳細については、「[メッセージへのメディア添付ファイルの追加](bot-builder-dotnet-add-media-attachments.md)」および「[メッセージへのリッチ カードの追加](bot-builder-dotnet-add-rich-card-attachments.md)」を参照してください。

## <a name="entities"></a>エンティティ

メッセージの `Entities` プロパティは、拡張可能な <a href="http://schema.org/" target="_blank">schema.org</a> オブジェクトの配列であり、チャネルとボットの間で共通のコンテキスト メタデータを交換するために使用します。

### <a name="mention-entities"></a>メンション エンティティ

多くのチャネルは、ボットまたはユーザーが会話のコンテキスト内で誰かを "メンション" する能力をサポートしています。 メッセージでユーザーをメンションするには、メッセージの `Entities` プロパティに `Mention` オブジェクトを設定します。 `Mention` オブジェクトには、次のプロパティが含まれています。 

| プロパティ | Description | 
|----|----|
| 種類 | エンティティの種類 ("mention") | 
| Mentioned | どのユーザーがメンションされたかを示す `ChannelAccount` オブジェクト | 
| Text | メンション自体を表す `Activity.Text` プロパティ内のテキスト (空または null 値の可能性があります) |

このコード例は、`Mention` エンティティを `Entities` コレクションに追加する方法を示しています。

[!code-csharp[set Mention](../includes/code/dotnet-create-messages.cs#setMention)]

> [!TIP]
> ユーザーの意図を判断しようとするとき、ボットはメッセージの中で自らがメンションされている部分を無視したい場合があります。 `GetMentions` メソッドを呼び出し、応答で返された `Mention` オブジェクトを評価します。

### <a name="place-objects"></a>場所オブジェクト

<a href="https://schema.org/Place" target="_blank">場所関連の情報</a>は、メッセージの `Entities` プロパティに `Place` オブジェクトまたは `GeoCoordinates` オブジェクトを設定することによってメッセージ内で伝達できます。 

`Place` オブジェクトには、次のプロパティが含まれています。

| プロパティ | Description | 
|----|----|
| 種類 | エンティティの種類 ("Place") |
| Address | 説明または `PostalAddress` オブジェクト (将来) | 
| ジオ (主要地域) | GeoCoordinates | 
| HasMap | 地図の URL または `Map` オブジェクト (将来) |
| 名前 | 場所の名前 |

`GeoCoordinates` オブジェクトには、次のプロパティが含まれています。

| プロパティ | Description | 
|----|----|
| 種類 | エンティティの種類 ("GeoCoordinates") |
| 名前 | 場所の名前 |
| Longitude | 場所の経度 (<a href="https://en.wikipedia.org/wiki/World_Geodetic_System" target="_blank">WGS 84</a>) | 
| Latitude | 場所の緯度 (<a href="https://en.wikipedia.org/wiki/World_Geodetic_System" target="_blank">WGS 84</a>) | 
| Elevation | 場所の標高 (<a href="https://en.wikipedia.org/wiki/World_Geodetic_System" target="_blank">WGS 84</a>) | 

このコード例は、`Place` エンティティを `Entities` コレクションに追加する方法を示しています。

[!code-csharp[set GeoCoordinates](../includes/code/dotnet-create-messages.cs#setGeoCoord)]

### <a name="consume-entities"></a>エンティティの使用

エンティティを使用するには、`dynamic` キーワードまたは厳密に型指定されたクラスを使用します。

このコード例は、`dynamic` キーワードを使用してメッセージの `Entities` プロパティ内でエンティティを処理する方法を示しています。

[!code-csharp[examine entity using dynamic keyword](../includes/code/dotnet-create-messages.cs#examineEntity1)]

このコード例は、厳密に型指定されたクラスを使用してメッセージの `Entities` プロパティ内でエンティティを処理する方法を示しています。

[!code-csharp[examine entity using typed class](../includes/code/dotnet-create-messages.cs#examineEntity2)]

## <a name="channel-data"></a>チャネル データ

メッセージ アクティビティの `ChannelData` プロパティは、チャネル固有の機能を実装するために使用できます。 詳細については、「[チャネル固有の機能の実装](bot-builder-dotnet-channeldata.md)」を参照してください。

## <a name="text-to-speech"></a>テキストを音声に変換する

メッセージ アクティビティの `Speak` プロパティは、音声対応チャネルでボットが話すテキストを指定するために使用できます。 メッセージ アクティビティの `InputHint` プロパティは、クライアントのマイクと入力ボックス (ある場合) の状態を制御するために使用できます。 詳細については、「[メッセージへの音声の追加](bot-builder-dotnet-text-to-speech.md)」を参照してください。

## <a name="suggested-actions"></a>推奨されるアクション

メッセージ アクティビティの `SuggestedActions` プロパティは、ユーザーが入力を提供するためにタップできるボタンを表示するために使用できます。 (タップされた後でも表示されたままとなり、ユーザーがアクセスできる) リッチ カード内に表示されるボタンとは異なり、推奨される操作ウィンドウ内に表示されるボタンは、ユーザーが選択を行った後は表示されなくなります。 詳細については、「[メッセージへの推奨されるアクションの追加](bot-builder-dotnet-add-suggested-actions.md)」を参照してください。

## <a name="next-steps"></a>次の手順

ボットとユーザーは互いにメッセージを送信できます。 メッセージが複雑な場合、ボットはメッセージでリッチ カードをユーザーに送信できます。 リッチ カードは、ほとんどのボットで一般的に必要とされる表示と対話のシナリオの多くをカバーします。

> [!div class="nextstepaction"]
> [メッセージでのリッチ カードの送信](bot-builder-dotnet-add-rich-card-attachments.md)

## <a name="additional-resources"></a>その他のリソース

- [アクティビティの概要](bot-builder-dotnet-activities.md)
- [アクティビティを送受信する](bot-builder-dotnet-connector.md)
- [メッセージへのメディア添付ファイルの追加](bot-builder-dotnet-add-media-attachments.md)
- [メッセージへのリッチ カードの追加](bot-builder-dotnet-add-rich-card-attachments.md)
- [メッセージへの音声の追加](bot-builder-dotnet-text-to-speech.md)
- [メッセージへの推奨されるアクションの追加](bot-builder-dotnet-add-suggested-actions.md)
- [チャネル固有の機能の実装](bot-builder-dotnet-channeldata.md)
- <a href="https://docs.botframework.com/csharp/builder/sdkreference/dc/d2f/class_microsoft_1_1_bot_1_1_connector_1_1_activity.html" target="_blank">Activity クラス</a>
- <a href="/dotnet/api/microsoft.bot.connector.imessageactivity" target="_blank">IMessageActivity インターフェイス</a>

