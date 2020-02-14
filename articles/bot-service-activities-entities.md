---
title: エンティティとアクティビティの種類 - Bot Service
description: エンティティとアクティビティの種類。
keywords: メンション エンティティ, アクティビティの種類, エンティティの使用
author: ivorb
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 03/01/2018
ms.openlocfilehash: cd11cc1fbbacb7e555da4e00337d6fd4b79a4df6
ms.sourcegitcommit: f8b5cc509a6351d3aae89bc146eaabead973de97
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/09/2020
ms.locfileid: "75789079"
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
| Type | エンティティの種類 ("mention") |
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
| Type | エンティティの種類 ("Place") |
| Address | 説明または住所オブジェクト (将来) |
| ジオ (主要地域) | 地理座標 |
| HasMap | 地図の URL または地図オブジェクト (将来) |
| Name | 場所の名前 |

geoCoordinates オブジェクトには、次のプロパティが含まれています。

| プロパティ | 説明 |
|----|----|
| Type | エンティティの種類 ("GeoCoordinates") |
| Name | 場所の名前 |
| Longitude | 場所の経度 (<a href="https://en.wikipedia.org/wiki/World_Geodetic_System" target="_blank">WGS 84</a>) |
| Latitude | 場所の緯度 (<a href="https://en.wikipedia.org/wiki/World_Geodetic_System" target="_blank">WGS 84</a>) |
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
<!-- 
This code example show how to process an activity of type **message**:

# [C#](#tab/cs)

```cs
if (context.Activity.Type == ActivityTypes.Message){
    // do something
}
```

# [JavaScript](#tab/js)

```js
if(context.activity.type === 'message'){
    // do something
}
```

--- -->

アクティビティには最も一般的な **message** 以外に複数の種類があります。 さまざまなアクティビティの種類の説明と詳細については、[Bot Framework のアクティビティ スキーマ](https://aka.ms/botSpecs-activitySchema)に関するページを参照してください。

::: moniker range="azure-bot-service-3.0"

## <a name="additional-resources"></a>その他のリソース

- <a href="https://docs.botframework.com/csharp/builder/sdkreference/dc/d2f/class_microsoft_1_1_bot_1_1_connector_1_1_activity.html" target="_blank">Activity クラス</a>
::: moniker-end
