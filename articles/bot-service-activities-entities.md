---
title: エンティティとアクティビティの種類 | Microsoft Docs
description: エンティティとアクティビティの種類。
keywords: メンション エンティティ, アクティビティの種類, エンティティの使用
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 03/01/2018
ms.openlocfilehash: 818017a81b497b13ee181dbb6b87c03a0182736d
ms.sourcegitcommit: 75f32b3325dd0fc4d8128dee6c22ebf91e5785b3
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/09/2018
ms.locfileid: "53120679"
---
# <a name="entities-and-activity-types"></a>エンティティとアクティビティの種類

エンティティはアクティビティの一部であり、アクティビティまたは会話に関する追加情報を提供します。

[!include[Entity boilerplate](includes/snippet-entity-boilerplate.md)]

## <a name="entities"></a>エンティティ

メッセージの "*エンティティ*" プロパティは、拡張可能な <a href="http://schema.org/" target="_blank">schema.org</a> オブジェクトの配列であり、チャネルとボットの間で共通のコンテキスト メタデータを交換するために使用します。

### <a name="mention-entities"></a>メンション エンティティ

多くのチャネルは、ボットまたはユーザーが会話のコンテキスト内で誰かを "メンション" する能力をサポートしています。
メッセージでユーザーをメンションするには、メッセージのエンティティ プロパティに *mention* オブジェクトを設定します。
メンション オブジェクトには、次のプロパティが含まれています。

| プロパティ | 説明 |
|----|----|
| type | エンティティの種類 ("mention") |
| Mentioned | メンションされたユーザーを示すチャネル アカウント オブジェクト | 
| Text | メンション自体を表す *activity.text* プロパティ内のテキスト (空または null 値の可能性があります) |

このコード例は、メンション エンティティをエンティティ コレクションに追加する方法を示しています。

# <a name="ctabcs"></a>[C#](#tab/cs)
[!code-csharp[set Mention](includes/code/dotnet-create-messages.cs#setMention)]

> [!TIP]
> ユーザーの意図を判断しようとするとき、ボットはメッセージの中でメンションされている部分を無視したい場合があります。 `GetMentions` メソッドを呼び出し、応答で返された `Mention` オブジェクトを評価します。

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)
```javascript
var entity = context.activity.entities;

const mention = {
    type: "Mention",
    text: "@johndoe",
    mentioned: {
        name: "John Doe",
        id: "UV341235"
    }
}

entity = [mention];
```

---

### <a name="place-objects"></a>場所オブジェクト

<a href="https://schema.org/Place" target="_blank">場所関連の情報</a>は、メッセージのエンティティ プロパティに *Place* オブジェクトまたは *GeoCoordinates* オブジェクトを設定することによってメッセージ内で伝達できます。

place オブジェクトには、次のプロパティが含まれています。

| プロパティ | 説明 |
|----|----|
| type | エンティティの種類 ("Place") |
| Address | 説明または住所オブジェクト (将来) |
| ジオ (主要地域)  | GeoCoordinates |
| HasMap | 地図の URL または地図オブジェクト (将来) |
| Name | 場所の名前 |

geoCoordinates オブジェクトには、次のプロパティが含まれています。

| プロパティ | 説明 |
|----|----|
| type | エンティティの種類 ("GeoCoordinates") |
| Name | 場所の名前 |
| Longitude | 場所の経度 (<a href="https://en.wikipedia.org/wiki/World_Geodetic_System" target="_blank">WGS 84</a>) |
| Longitude | 場所の緯度 (<a href="https://en.wikipedia.org/wiki/World_Geodetic_System" target="_blank">WGS 84</a>) |
| Elevation | 場所の標高 (<a href="https://en.wikipedia.org/wiki/World_Geodetic_System" target="_blank">WGS 84</a>) |

このコード例では、場所エンティティをエンティティ コレクションに追加する方法を示します。

# <a name="ctabcs"></a>[C#](#tab/cs)
[!code-csharp[set GeoCoordinates](includes/code/dotnet-create-messages.cs#setGeoCoord)]

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)
```javascript
var entity = context.activity.entities;

const place = {
    elavation: 100,
    type: "GeoCoordinates",
    name : "myPlace",
    latitude: 123,
    longitude: 234
};

entity = [place];

```

---

### <a name="consume-entities"></a>エンティティの使用

# <a name="ctabcs"></a>[C#](#tab/cs)

エンティティを使用するには、`dynamic` キーワードまたは厳密に型指定されたクラスを使用します。

このコード例は、`dynamic` キーワードを使用してメッセージの `Entities` プロパティ内でエンティティを処理する方法を示しています。

[!code-csharp[examine entity using dynamic keyword](includes/code/dotnet-create-messages.cs#examineEntity1)]

このコード例は、厳密に型指定されたクラスを使用してメッセージの `Entities` プロパティ内でエンティティを処理する方法を示しています。

[!code-csharp[examine entity using typed class](includes/code/dotnet-create-messages.cs#examineEntity2)]

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

このコード例では、メッセージの `entity` プロパティ内でエンティティを処理する方法を示します。

```javascript
if (entity[0].type === "GeoCoordinates" && entity[0].latitude > 34) {
    // do something
}
```

---

## <a name="activity-types"></a>アクティビティの種類

このコード例では、**message** 型のアクティビティの処理方法を示します。

# <a name="ctabcs"></a>[C#](#tab/cs)

```cs
if (context.Activity.Type == ActivityTypes.Message){
    // do something
}
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

```js
if(context.activity.type === 'message'){
    // do something
}
```

---

アクティビティには最も一般的な **message** 以外に複数の種類があります。 アクティビティには複数の種類があります。

| Activity.Type | インターフェイス | 説明 |
|-----|-----|-----|
| [message](#message) | IMessageActivity (C#) <br> Activity (JS) | ボットとユーザー間の通信を表します。 |
| [contactRelationUpdate](#contactrelationupdate) | IContactRelationUpdateActivity (C#) <br> Activity (JS) | ユーザーの連絡先リストに対してボットが追加または削除されたことを示します。 |
| [conversationUpdate](#conversationupdate) | IConversationUpdateActivity (C#) <br> Activity (JS) | ボットが会話に追加されたこと、他のメンバーが会話に追加された、または会話から削除されたこと、会話のメタデータが変更されたことを示します。 |
| [deleteUserData](#deleteuserdata) | 該当なし | ボットに保存されている可能性のあるユーザー データを削除するようにユーザーが要求したことをボットに示します。 |
| [endOfConversation](#endofconversation) | IEndOfConversationActivity (C#) <br> Activity (JS) | 会話の終了を示します。 |
| [event](#event) | IEventActivity (C#) <br> Activity (JS) | ボットに送信される、ユーザーに表示されない通信を表します。 |
| [installationUpdate](#installationupdate) | IInstallationUpdateActivity (C#) <br> Activity (JS) | チャネルの組織単位 (顧客テナントや "チーム" など) 内のボットのインストールまたはアンインストールを表します。 |
| [invoke](#invoke) | IInvokeActivity (C#) <br> Activity (JS) | ボットに送信される、特定の操作の実行を要求する通信を表します。 このアクティビティの種類は、Microsoft Bot Framework による内部使用のために予約されてます。 |
| [messageReaction](#messagereaction) | IMessageReactionActivity (C#) <br> Activity (JS) | ユーザーが既存のアクティビティに反応したことを示します。 たとえば、ユーザーがメッセージの "いいね！" ボタンをクリックしたような場合です。 |
| [typing](#typing) | ITypingActivity (C#) <br> Activity (JS) | 会話の相手側のユーザーまたはボットが応答をコンパイルしていることを示します。 |
| messageUpdate | IMessageUpdateActivity (C#) <br> Activity (JS) | 会話内の以前のメッセージ アクティビティを更新する要求を示します。 |
| messageDelete | IMessageDeleteActivity (C#) <br> Activity (JS) | 会話内の以前のメッセージ アクティビティを削除する要求を示します。 |
| suggestion | ISuggestionActivity (C#) <br> Activity (JS) | 別の特定のアクティビティについての受信者にする個人的な提案を示します。 |
| trace | ITraceActivity (C#) <br> Activity (JS) | ログに記録された会話のトランスクリプトにボットから内部情報を記録できるアクティビティ。 |
| handoff | IHandoffActivity (C#) <br> Activity (JS) | 会話の制御権が移転されているか、会話の制御権の移転を求める要求。 |

## <a name="message"></a>message

<!-- Only the last link is different. -->

::: moniker range="azure-bot-service-3.0"

ボットは、メッセージ アクティビティを送信してユーザーに情報を伝達し、ユーザーからメッセージ アクティビティを受信します。
プレーン テキストだけで構成されているシンプルなメッセージもあれば、読み上げテキスト、[推奨されるアクション](v4sdk/bot-builder-howto-add-suggested-actions.md)、[メディアの添付ファイル](v4sdk/bot-builder-howto-add-media-attachments.md)、[リッチ カード](v4sdk/bot-builder-howto-add-media-attachments.md#send-a-hero-card)、[チャネル固有のデータ](~/dotnet/bot-builder-dotnet-channeldata.md)など、よりリッチなコンテンツを含むメッセージもあります。

::: moniker-end

::: moniker range="azure-bot-service-4.0"

ボットは、メッセージ アクティビティを送信してユーザーに情報を伝達し、ユーザーからメッセージ アクティビティを受信します。
プレーン テキストだけで構成されているシンプルなメッセージもあれば、読み上げテキスト、[推奨されるアクション](v4sdk/bot-builder-howto-add-suggested-actions.md)、[メディアの添付ファイル](v4sdk/bot-builder-howto-add-media-attachments.md)、[リッチ カード](v4sdk/bot-builder-howto-add-media-attachments.md#send-a-hero-card)、[チャネル固有のデータ](~/v4sdk/bot-builder-channeldata.md)など、よりリッチなコンテンツを含むメッセージもあります。

::: moniker-end

## <a name="contactrelationupdate"></a>contactRelationUpdate

ボットがユーザーの連絡先リストに対して追加または削除されるたびに、ボットは連絡先関係更新アクティビティを受信します。 アクティビティのアクション プロパティ (追加 | 削除) の値は、ボットがユーザーの連絡先リストに対して追加または削除されているかを示します。

## <a name="conversationupdate"></a>conversationUpdate

ボットは、ボットが会話に追加されたり、他のメンバーが会話に追加または会話から削除されたり、会話メタデータが変更されたりするたびに、会話更新アクティビティを受信します。

メンバーが会話に追加された場合、アクティビティの追加メンバー プロパティには、新しいメンバーを示すチャネル アカウント オブジェクトの配列が含まれます。

ボットが会話に追加されたか (つまり、新しいメンバーの 1 つか) どうかを判断するには、アクティビティの受信者 ID (つまり、ボットの ID) の値が、追加メンバー配列のアカウントのいずれかの ID プロパティと一致するかどうかを評価します。

メンバーが会話から削除された場合、削除メンバー プロパティには、削除されたメンバーを示すチャネル アカウント オブジェクトの配列が含まれます。

> [!TIP]
> ユーザーが会話に参加したことを示す会話更新アクティビティをボットが受信した場合、そのユーザーにウェルカム メッセージを送信して応答させることを選択できます。

## <a name="deleteuserdata"></a>deleteUserData

ボットは、ボットが以前に保持していたユーザーのデータを削除するようにユーザーが要求すると、ユーザー データ削除アクティビティを受信します。 ボットがこの種類のアクティビティを受信した場合、ボットはその要求を実行したユーザー用に以前保存した個人を特定できる情報 (PII) を削除する必要があります。

## <a name="endofconversation"></a>endOfConversation

ボットは、ユーザーが会話を終了したことを示す会話終了アクティビティを受信します。 ボットは、会話終了アクティビティを送信して、会話を終了していることをユーザーに示すことができます。

## <a name="event"></a>event

ボットは、外部プロセスまたは外部サービスから、ボットに情報を通信することを望むが、その情報をユーザーには表示しないことを示す、イベント アクティビティを受信する場合があります。 通常、イベント アクティビティの送信者は、ボットがいかなる方法でも受信確認を行うことを望みません。

## <a name="installationupdate"></a>installationUpdate

インストール更新アクティビティは、チャネルの組織単位 (顧客テナントや "チーム" など) 内のボットのインストールまたはアンインストールを表します。 インストール更新アクティビティは、一般に、チャネルの追加または削除を表していません。 チャネルは、チャネル内のテナント、チーム、他の組織単位のボットが追加または削除されるときに、インストール アクティビティを送信できます。 ボットがチャネルにインストールされるとき、またはボットがチャネルから削除されるときは、チャネルはインストール アクティビティを送信してはなりません。

## <a name="invoke"></a>invoke

ボットは、ボットに対する特定の操作の実行要求を表す呼び出しアクティビティを受信することがあります。
通常、呼び出しアクティビティの送信者は、ボットが HTTP 応答で受信確認することを期待しています。
このアクティビティの種類は、Microsoft Bot Framework による内部使用のために予約されてます。

## <a name="messagereaction"></a>messageReaction

一部のチャネルは、ユーザーが既存のアクティビティに反応したときに、メッセージ反応アクティビティをボットに送信します。 たとえば、ユーザーがメッセージの "いいね！" ボタンをクリックしたような場合です。 replyToId プロパティは、ユーザーが反応したアクティビティを示します。

メッセージ反応アクティビティは、チャネルで定義されている任意の数のメッセージ反応の種類に対応することができます。 たとえば、チャネルは反応の種類として "Like" や "PlusOne" を送信できます。

## <a name="typing"></a>typing

ボットは、ユーザーが応答を入力していることを示す入力中アクティビティを受信します。
ボットは、入力中アクティビティを送信して、要求を満たしたり応答を作成したりするための作業を行っていることをユーザーに示すことができます。

::: moniker range="azure-bot-service-3.0"

## <a name="additional-resources"></a>その他のリソース

- <a href="https://docs.botframework.com/en-us/csharp/builder/sdkreference/dc/d2f/class_microsoft_1_1_bot_1_1_connector_1_1_activity.html" target="_blank">Activity クラス</a>
::: moniker-end
