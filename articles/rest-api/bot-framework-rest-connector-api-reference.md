---
title: API リファレンス | Microsoft Docs
description: Bot Connector サービスと Bot State サービスのヘッダー、操作、オブジェクト、およびエラーについて説明します。
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 10/25/2018
ms.openlocfilehash: fd98b1bc8c3aa3b2c9fd716289dfd3ce75bec75b
ms.sourcegitcommit: 8183bcb34cecbc17b356eadc425e9d3212547e27
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/09/2019
ms.locfileid: "55971542"
---
# <a name="api-reference"></a>API リファレンス

> [!NOTE]
> REST API は SDK に相当するものではありません。 REST API は標準の REST 通信を可能にするために提供されていますが、Bot Framework との好ましい対話方法は SDK です。 

Bot Framework の内部で、Bot Connector サービスは、Bot Framework Portal で構成されるチャネルでボットがユーザーとメッセージを交換できるようにします。また Bot State サービスは、ボットが Bot Connector サービスを使用して行う会話に関連した状態データを、ボットで保存および取得できるようにします。 どちらのサービスも、HTTPS 経由で業界標準の REST および JSON を使用します。

> [!IMPORTANT]
> Bot Framework State Service API を運用環境で使用することは推奨されていません。将来のリリースで非推奨となる可能性があります。 テストが目的の場合はメモリ内ストレージを使用するようにボット コードを更新し、運用ボットの場合はいずれかの **Azure 拡張機能**を使用することをお勧めします。 [.NET](~/dotnet/bot-builder-dotnet-state.md) または [Node](~/nodejs/bot-builder-nodejs-state.md) での実装の詳細については、「**状態データの管理**」トピックを参照してください。

## <a name="base-uri"></a>ベース URI

ユーザーがボットにメッセージを送信すると、受信要求には、ボットが応答を送信するエンドポイントを指定する `serviceUrl` プロパティを持つ [Activity](#activity-object) オブジェクトが含まれています。 Bot Connector サービスまたは Bot State サービスにアクセスするには、`serviceUrl` 値を API 要求のベース URI として使用します。 

たとえば、ユーザーがボットにメッセージを送信したときに、ボットが次のアクティビティを受け取るとします。

```json
{
    "type": "message",
    "id": "bf3cc9a2f5de...",
    "timestamp": "2016-10-19T20:17:52.2891902Z",
    "serviceUrl": "https://smba.trafficmanager.net/apis",
    "channelId": "channel's name/id",
    "from": {
        "id": "1234abcd",
        "name": "user's name"
    },
    "conversation": {
        "id": "abcd1234",
        "name": "conversation's name"
    },
    "recipient": {
        "id": "12345678",
        "name": "bot's name"
    },
    "text": "Haircut on Saturday"
}
```

ユーザーのメッセージ内の `serviceUrl` プロパティは、ボットがエンドポイント `https://smba.trafficmanager.net/apis` に応答を送信することを示します。これは、この会話のコンテキストでボットが発行する後続の要求のベース URI になります。 ボットがユーザーに事前対応型のメッセージを送信する必要がある場合は、必ず `serviceUrl` の値を保存してください。

次の例は、ユーザーのメッセージに応答するためにボットが発行する要求を示しています。 

```http
POST https://smba.trafficmanager.net/apis/v3/conversations/abcd1234/activities/bf3cc9a2f5de... 
Authorization: Bearer eyJhbGciOiJIUzI1Ni...
Content-Type: application/json
```

```json
{
    "type": "message",
    "from": {
        "id": "12345678",
        "name": "bot's name"
    },
    "conversation": {
        "id": "abcd1234",
        "name": "conversation's name"
    },
   "recipient": {
        "id": "1234abcd",
        "name": "user's name"
    },
    "text": "I have several times available on Saturday!",
    "replyToId": "bf3cc9a2f5de..."
}
```

## <a name="headers"></a>headers

### <a name="request-headers"></a>要求ヘッダー

発行するすべての API 要求には、標準の HTTP 要求ヘッダーに加えて、ボットを認証するためのアクセス トークンを指定する `Authorization` ヘッダーが含まれている必要があります。 `Authorization` ヘッダーは次の形式を使用して指定します。

```http
Authorization: Bearer ACCESS_TOKEN
```

ボットのアクセス トークンを取得する方法の詳細については、「[Authenticate requests from your bot to the Bot Connector service](bot-framework-rest-connector-authentication.md#bot-to-connector)」(ボットからの要求を Bot Connector サービスに認証する) を参照してください。

### <a name="response-headers"></a>応答ヘッダー

標準の HTTP 応答ヘッダーに加えて、すべての応答には `X-Correlating-OperationId` ヘッダーが含まれます。 このヘッダーの値は、要求の詳細を含む Bot Framework のログ エントリに対応する ID です。 エラー応答を受け取るたびに、このヘッダーの値をキャプチャしてください。 自力で問題を解決できない場合は、問題を報告するときにサポート チームに提供する情報にこの値を含めてください。

## <a name="http-status-codes"></a>HTTP 状態コード

各応答で返される <a href="http://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html" target="_blank"> HTTP 状態コード</a> は、対応する要求の結果を示しています。 

| HTTP 状態コード | 意味 |
|----|----|
| 200 | 要求は成功しました。 |
| 201 | 要求は成功しました。 |
| 202 | 要求の処理は受理されました。 |
| 204 | 要求は成功しましたが、返されたコンテンツはありませんでした。 |
| 400 | 要求の形式その他が正しくありませんでした。 |
| 401 | ボットは要求を行うことが許可されていません。 |
| 403 | 要求された操作の実行がボットに許可されていません。 |
| 404 | 要求されたリソースが見つかりませんでした。 |
| 500 | 内部サーバー エラーが発生しました。 |
| 503 | サービスを利用できません。 |

### <a name="errors"></a>Errors

4xx 範囲または 5xx 範囲の HTTP 状態コードを指定する応答では、エラーの情報を提供する [ErrorResponse](#errorresponse-object) オブジェクトが応答の本文に含まれます。 4xx 範囲のエラー応答を受け取った場合、要求を再送信する前に、**ErrorResponse** オブジェクトを検査してエラーの原因を識別し、問題を解決してください。

## <a name="conversation-operations"></a>会話操作 
これらの操作を使用して、会話を作成し、メッセージ (アクティビティ) を送信し、会話の内容を管理します。

| Operation | 説明 |
|----|----|
| [会話を作成する](#create-conversation) | 新しい会話を作成します。 | 
| [会話に送信する](#send-to-conversation) | 指定された会話の最後にアクティビティ (メッセージ) を送信します。 | 
| [アクティビティに返信する](#reply-to-activity) | 指定されたアクティビティへの返信として、指定された会話にアクティビティ (メッセージ) を送信します。 | 
| [会話のメンバーを取得する](#get-conversation-members) | 指定された会話のメンバーを取得します。 |
| [会話のページングされたメンバーを取得する](#get-conversation-paged-members) | 指定された会話のメンバーを一度に 1 ページずつ取得します。 |
| [アクティビティのメンバーを取得する](#get-activity-members) | 指定された会話内の指定されたアクティビティのメンバーを取得します。 | 
| [アクティビティを更新する](#update-activity) | 既存のアクティビティを更新します。 | 
| [アクティビティを削除する](#delete-activity) | 既存のアクティビティを削除します。 | 
| [添付ファイルをチャネルにアップロードする](#upload-attachment-to-channel) | 添付ファイルをチャネルの BLOB ストレージに直接アップロードします。 |

### <a name="create-conversation"></a>会話を作成する
新しい会話を作成します。 
```http 
POST /v3/conversations
```

| | |
|----|----|
| **要求本文** | [Conversation](#conversation-object) オブジェクト |
| **戻り値** | [ResourceResponse](#resourceresponse-object) オブジェクト | 

### <a name="send-to-conversation"></a>会話に送信する
指定された会話にアクティビティ (メッセージ) を送信します。 アクティビティは、タイムスタンプまたはチャネルのセマンティクスに従って会話の最後に追加されます。 会話内の特定のメッセージに返信するには、代わりに[アクティビティに返信する](#reply-to-activity)を使用します。
```http
POST /v3/conversations/{conversationId}/activities
```

| | |
|----|----|
| **要求本文** | [Activity](#activity-object) オブジェクト |
| **戻り値** | [Identification](#identification-object) オブジェクト | 

### <a name="reply-to-activity"></a>アクティビティに返信する
指定されたアクティビティへの返信として、指定された会話にアクティビティ (メッセージ) を送信します。 チャネルがサポートしている場合、アクティビティは別のアクティビティへの返信として追加されます。 ネストした応答をチャネルがサポートしていない場合、この操作は[会話に送信する](#send-to-conversation)と同様に動作します。
```http
POST /v3/conversations/{conversationId}/activities/{activityId}
```

| | |
|----|----|
| **要求本文** | [Activity](#activity-object) オブジェクト |
| **戻り値** | [Identification](#identification-object) オブジェクト | 

### <a name="get-conversation-members"></a>会話のメンバーを取得する
指定された会話のメンバーを取得します。
```http
GET /v3/conversations/{conversationId}/members
```

| | |
|----|----|
| **要求本文** | 該当なし |
| **戻り値** | [ChannelAccount](#channelaccount-object) オブジェクトの配列 | 

### <a name="get-conversation-paged-members"></a>会話のページングされたメンバーを取得する
指定された会話のメンバーを一度に 1 ページずつ取得します。
```http
GET /v3/conversations/{conversationId}/pagedmembers
```

| | |
|----|----|
| **要求本文** | 該当なし |
| **戻り値** | [ChannelAccount](#channelaccount-object) オブジェクトの配列と、さらに値を取得するときに使用できる継続トークン|

### <a name="get-activity-members"></a>アクティビティのメンバーを取得する
指定された会話内の指定されたアクティビティのメンバーを取得します。
```http
GET /v3/conversations/{conversationId}/activities/{activityId}/members
```

| | |
|----|----|
| **要求本文** | 該当なし |
| **戻り値** | [ChannelAccount](#channelaccount-object) オブジェクトの配列 | 

### <a name="update-activity"></a>アクティビティを更新する
一部のチャネルでは、既存のアクティビティを編集してボットの会話の新しい状態を反映させることができます。 たとえば、いずれかのボタンをユーザーがクリックした後、会話内のメッセージからボタンを削除することができます。 成功すると、この操作は指定された会話内の指定されたアクティビティを更新します。 
```http
PUT /v3/conversations/{conversationId}/activities/{activityId}
```

| | |
|----|----|
| **要求本文** | [Activity](#activity-object) オブジェクト |
| **戻り値** | [Identification](#identification-object) オブジェクト | 

### <a name="delete-activity"></a>アクティビティを削除する
一部のチャネルでは、既存のアクティビティを削除できます。 成功すると、この操作は指定されたアクティビティを指定された会話から削除します。
```http
DELETE /v3/conversations/{conversationId}/activities/{activityId}
```

| | |
|----|----|
| **要求本文** | 該当なし |
| **戻り値** | 操作の結果を示す HTTP 状態コード。 応答の本文では何も指定されません。 | 

### <a name="upload-attachment-to-channel"></a>添付ファイルをチャネルにアップロード
指定された会話の添付ファイルをチャネルの BLOB ストレージに直接アップロードします。 これにより、準拠したストアにデータを保存できます。 
```http 
POST /v3/conversations/{conversationId}/attachments
```

| | |
|----|----|
| **要求本文** | [AttachmentUpload](#attachmentupload-object) オブジェクト。 |
| **戻り値** | [ResourceResponse](#resourceresponse-object) オブジェクト。 **id** プロパティは、[添付ファイル情報を取得する](#get-attachment-info)操作と[添付ファイルを取得する](#get-attachment)操作で使用できる添付ファイル ID を指定します。 | 

## <a name="attachment-operations"></a>添付ファイル操作 
これらの操作を使用して、添付ファイルの情報とファイル自体のバイナリ データを取得します。

| Operation | 説明 |
|----|----|
| [添付ファイル情報を取得する](#get-attachment-info) | ファイル名、ファイルの種類、使用可能なビュー (例: オリジナルまたはサムネイル) など、指定された添付ファイルの情報を取得します。 |
| [添付ファイルを取得する](#get-attachment) | 指定された添付ファイルの指定されたビューをバイナリ コンテンツとして取得します。 | 

### <a name="get-attachment-info"></a>添付ファイル情報を取得する 
ファイル名、種類、使用可能なビュー (例: オリジナルまたはサムネイル) など、指定された添付ファイルの情報を取得します。
```http
GET /v3/attachments/{attachmentId}
```

| | |
|----|----|
| **要求本文** | 該当なし |
| **戻り値** | [AttachmentInfo](#attachmentinfo-object) オブジェクト | 

### <a name="get-attachment"></a>添付ファイルを取得する
指定された添付ファイルの指定されたビューをバイナリ コンテンツとして取得します。
```http
GET /v3/attachments/{attachmentId}/views/{viewId}
```

| | |
|----|----|
| **要求本文** | 該当なし |
| **戻り値** | 指定された添付ファイルの指定されたビューを表すバイナリ コンテンツ | 

## <a name="state-operations"></a>状態操作
これらの操作を使用して、状態データを保存および取得します。

| Operation | 説明 |
|----|----|
| [ユーザー データを設定する](#set-user-data) | チャネルの特定のユーザーの状態データを保存します。 | 
| [会話データを設定する](#set-conversation-data) | チャネルの特定の会話の状態データを保存します。 | 
| [個人的な会話データを設定する](#set-private-conversation-data) | チャネルの特定の会話のコンテキスト内で特定のユーザーの状態データを保存します。 | 
| [ユーザー データを取得する](#get-user-data) | チャネルのすべての会話で以前に保存した特定のユーザーの状態データを取得します。 | 
| [会話データを取得する](#get-conversation-data) | チャネルの特定の会話で以前に保存した状態データを取得します。 | 
| [個人的な会話データを取得する](#get-private-conversation-data) | チャネルの特定の会話のコンテキスト内で以前に保存した特定のユーザーの状態データを取得します。 | 
| [ユーザーの状態を削除する](#delete-state-for-user) | [ユーザー データを設定する](#set-user-data)操作または[個人的な会話データを設定する](#set-private-conversation-data)操作を使用して以前に保存したユーザーの状態データを削除します。 | 

### <a name="set-user-data"></a>ユーザー データを設定する
指定されたチャネルの指定されたユーザーの状態データを保存します。
```http
POST /v3/botstate/{channelId}/users/{userId} 
```

| | |
|----|----|
| **要求本文** | [BotData](#botdata-object) オブジェクト |
| **戻り値** | [BotData](#botdata-object) オブジェクト | 

### <a name="set-conversation-data"></a>会話データを設定する
指定されたチャネルの指定された会話の状態データを保存します。
```http
POST /v3/botstate/{channelId}/conversations/{conversationId}
```

| | |
|----|----|
| **要求本文** | [BotData](#botdata-object) オブジェクト |
| **戻り値** | [BotData](#botdata-object) オブジェクト | 

### <a name="set-private-conversation-data"></a>個人的な会話データを設定する
指定されたチャネルの指定された会話のコンテキスト内で、指定されたユーザーの状態データを保存します。
```http
POST /v3/botstate/{channelId}/conversations/{conversationId}/users/{userId} 
```

| | |
|----|----|
| **要求本文** | [BotData](#botdata-object) オブジェクト |
| **戻り値** | [BotData](#botdata-object) オブジェクト | 

### <a name="get-user-data"></a>ユーザー データを取得する
指定されたチャネルのすべての会話で以前に保存した特定のユーザーの状態データを取得します。
```http
GET /v3/botstate/{channelId}/users/{userId} 
```

| | |
|----|----|
| **要求本文** | 該当なし |
| **戻り値** | [BotData](#botdata-object) オブジェクト | 

### <a name="get-conversation-data"></a>会話データを取得する
指定されたチャネルの特定の会話で以前に保存した状態データを取得します。
```http
GET /v3/botstate/{channelId}/conversations/{conversationId} 
```

| | |
|----|----|
| **要求本文** | 該当なし |
| **戻り値** | [BotData](#botdata-object) オブジェクト | 

### <a name="get-private-conversation-data"></a>個人的な会話データを取得する
指定されたチャネルの指定された会話のコンテキスト内で以前に保存した特定のユーザーの状態データを取得します。
```http
GET /v3/botstate/{channelId}/conversations/{conversationId}/users/{userId} 
```

| | |
|----|----|
| **要求本文** | 該当なし |
| **戻り値** | [BotData](#botdata-object) オブジェクト | 

### <a name="delete-state-for-user"></a>ユーザーの状態を削除する
[ユーザー データを設定する](#set-user-data)操作または[個人的な会話データを設定する](#set-private-conversation-data)操作のどちらかを使用して以前に保存した特定のチャネルの特定のユーザーの状態データを削除します。
```http
DELETE /v3/botstate/{channelId}/users/{userId} 
```

| | |
|----|----|
| **要求本文** | 該当なし |
| **戻り値** | 文字列の配列 (ID) | 

## <a id="objects"></a> スキーマ

スキーマは、ボットがユーザーとのコミュニケーションに使用できるオブジェクトとそのプロパティを定義します。 

| Object | 説明 |
| ---- | ---- |
| [Activity オブジェクト](#activity-object) | ボットとユーザーの間で交換されるメッセージを定義します。 |
| [AnimationCard オブジェクト](#animationcard-object) | アニメーション GIF または短いビデオを再生できるカードを定義します。 |
| [Attachment オブジェクト](#attachment-object) | メッセージに含める追加の情報を定義します。 メディア ファイル (例: オーディオ、ビデオ、画像、ファイル) またはリッチ カードを添付ファイルにすることができます。 |
| [AttachmentData オブジェクト](#attachmentdata-object) |添付ファイルのデータについて説明します。 |
| [AttachmentInfo オブジェクト](#attachmentinfo-object) | 添付ファイルについて説明します。 |
| [AttachmentView オブジェクト](#attachmentview-object) | 添付ファイルのビューを定義します。 |
| [AttachmentUpload オブジェクト](#attachmentupload-object) | アップロードする添付ファイルを定義します。 |
| [AudioCard オブジェクト](#audiocard-object) | オーディオ ファイルを再生できるカードを定義します。 |
| [BotData オブジェクト](#botdata-object) | Bot State サービスを使用して保存される特定の会話のコンテキストで、ユーザーの状態データ、会話、またはユーザーを定義します。 |
| [CardAction オブジェクト](#cardaction-object) | 実行するアクションを定義します。 |
| [CardImage オブジェクト](#cardimage-object) | カードに表示する画像を定義します。 |
| [ChannelAccount オブジェクト](#channelaccount-object) | チャネルのボットまたはユーザー アカウントを定義します。 |
| [Conversation オブジェクト](#conversation-object) | 会話を定義します (会話内に含まれているボットとユーザーなど)。 |
| [ConversationAccount オブジェクト](#conversationaccount-object) | チャネルでの会話を定義します。 |
| [ConversationParameters オブジェクト](#conversationparameters-object) | 新しい会話を作成するためのパラメーターを定義します |
| [ConversationReference オブジェクト](#conversationreference-object) | 会話内の特定のポイントを定義します。 |
| [ConversationResourceResponse オブジェクト](#conversationresourceresponse-object) | "リソースが含まれている応答 |
| [Entity オブジェクト](#entity-object) | エンティティ オブジェクトを定義します。 |
| [Error オブジェクト](#error-object) | エラーを定義します。 |
| [ErrorResponse オブジェクト](#errorresponse-object) | HTTP API 応答を定義します。 |
| [Fact オブジェクト](#fact-object) | ファクトを含むキーと値のペアを定義します。 |
| [Geocoordinates オブジェクト](#geocoordinates-object) | World Geodetic System (WSG84) 座標を使用して地理的な場所を定義します。 |
| [HeroCard オブジェクト](#herocard-object) | 大きい画像、タイトル、テキスト、アクションの各ボタンを持つカードを定義します。 |
| [Identification オブジェクト](#identification-object) | リソースを識別します。 |
| [MediaEventValue オブジェクト](#mediaeventvalue-object) |メディア イベントの補助パラメーター。|
| [MediaUrl オブジェクト](#mediaurl-object) | メディア ファイルのソース URL を定義します。 |
| [Mention オブジェクト](#mention-object) | 会話でメンションされたユーザーまたはボットを定義します。 |
| [MessageReaction オブジェクト](#messagereaction-object) | メッセージへの反応を定義します。 |
| [Place オブジェクト](#place-object) | 会話でメンションされた場所を定義します。 |
| [ReceiptCard オブジェクト](#receiptcard-object) | 購入のレシートを含むカードを定義します。 |
| [ReceiptItem オブジェクト](#receiptitem-object) | レシート内の品目を定義します。 |
| [ResourceResponse オブジェクト](#resourceresponse-object) | リソースを定義します。 |
| [SignInCard オブジェクト](#signincard-object) | ユーザーがサービスにサインインできるようにするカードを定義します。 |
| [SuggestedActions オブジェクト](#suggestedactions-object) | ユーザーが選択できるオプションを定義します。 |
| [ThumbnailCard オブジェクト](#thumbnailcard-object) | サムネイル画像、タイトル、テキスト、アクションの各ボタンを持つカードを定義します。 |
| [ThumbnailUrl オブジェクト](#thumbnailurl-object) | 画像のソース URL を定義します。 |
| [VideoCard オブジェクト](#videocard-object) | ビデオを再生できるカードを定義します。 |
| [SemanticAction オブジェクト](#semanticaction-object) | プログラムによるアクションへの参照を定義します。 |

### <a name="activity-object"></a>Activity オブジェクト
ボットとユーザーの間で交換されるメッセージを定義します。<br/><br/> 

| プロパティ | type | 説明 |
|----|----|----|
| **action** | 文字列 | 適用する、または適用されたアクション。 アクションのコンテキストを決定するには、**type** プロパティを使用します。 たとえば、**type** が **contactRelationUpdate** の場合、**action** プロパティの値は、ユーザーがボットを連絡先リストに追加した場合は **add** になり、ユーザーがボットを連絡先リストから削除した場合は **remove** になります。 |
| **attachments** | [Attachment](#attachment-object)[] | メッセージに含める追加の情報を定義する **Attachment** オブジェクトの配列。 各添付ファイルは、メディア ファイル (例: オーディオ、ビデオ、画像、ファイル) またはリッチ カードのどちらかにすることができます。 |
| **attachmentLayout** | 文字列 | メッセージに含まれるリッチ カード**添付ファイル**のレイアウト。 次の値のいずれか: **carousel**、**list**。 リッチ カード添付ファイルの詳細については、「[メッセージへのリッチ カード添付ファイルの追加](bot-framework-rest-connector-add-rich-cards.md)」を参照してください。 |
| **channelData** | オブジェクト | チャネル固有のコンテンツを格納するオブジェクト。 一部のチャネルには、添付ファイル スキーマでは表現できない追加情報を必要とする機能があります。 そのような場合は、このプロパティを、チャネルのドキュメントで定義されているチャネル固有のコンテンツに設定します。 詳細については、「[チャネル固有の機能の実装](bot-framework-rest-connector-channeldata.md)」を参照してください。 |
| **channelId** | 文字列 | チャネルを一意に識別する ID。 チャネルによって設定されます。 | 
| **conversation** | [ConversationAccount](#conversationaccount-object) | アクティビティが属する会話を定義する **ConversationAccount** オブジェクト。 |
| **code** | 文字列 | 会話が終了した理由を示すコード。 |
| **entities** | object[] | メッセージでメンションされたエンティティを表すオブジェクトの配列。 この配列内のオブジェクトは任意の <a href="http://schema.org/" target="_blank">Schema.org</a> オブジェクトです。 たとえば、会話でメンションされた人物を識別する [Mention](#mention-object) オブジェクトや、会話でメンションされた場所を識別する [Place](#place-object) オブジェクトを配列に含めることができます。 |
| **from** | [ChannelAccount](#channelaccount-object) | メッセージの送信者を指定する **ChannelAccount** オブジェクト。 |
| **historyDisclosed** | ブール値 | 履歴が公開されているかどうかを示すフラグ。 既定値は **false** です。 |
| **id** | 文字列 | チャネルでのアクティビティを一意に識別する ID。 | 
| **inputHint** | 文字列 | メッセージがクライアントに配信された後、ボットがユーザー入力を受け付けるか、期待するか、または無視するかを示す値。 次の値のいずれか: **acceptingInput**、**expectingInput**、**ignoringInput**。 |
| **locale** | 文字列 | メッセージ内のテキストの表示に使用する言語のロケールで、`<language>-<country>` の形式。 ボットがその言語の表示文字列を指定できるよう、チャネルはこのプロパティを使用してユーザーの言語を指示します。 既定値は **en-US** です。 |
| **localTimestamp** | 文字列 | ローカル タイム ゾーンでメッセージが送信された日時を <a href="https://en.wikipedia.org/wiki/ISO_8601" target="_blank">ISO-8601</a> 形式で表したもの。 |
| **membersAdded** | [ChannelAccount](#channelaccount-object)[] | 会話に参加したユーザーのリストを表す **ChannelAccount** オブジェクトの配列。 アクティビティの **type** が "conversationUpdate" で、ユーザーが会話に参加した場合にのみ存在します。 | 
| **membersRemoved** | [ChannelAccount](#channelaccount-object)[] | 会話から退出したユーザーのリストを表す **ChannelAccount** オブジェクトの配列。 アクティビティの **type** が "conversationUpdate" で、ユーザーが会話から退出した場合にのみ存在します。 | 
| **name** | 文字列 | 呼び出す操作の名前またはイベントの名前。 |
| **recipient** | [ChannelAccount](#channelaccount-object) | メッセージの受信者を指定する **ChannelAccount** オブジェクト。 |
| **relatesTo** | [ConversationReference](#conversationreference-object) | 会話内の特定のポイントを定義する **ConversationReference** オブジェクト。 |
| **replyToId** | 文字列 | このメッセージの返信先メッセージの ID。 ユーザーが送信したメッセージに返信するには、このプロパティをユーザーのメッセージの ID に設定します。 すべてのチャネルがスレッド返信をサポートするわけではありません。 このような場合、チャネルはこのプロパティを無視し、時間順のセマンティクス (タイムスタンプ) を使用してメッセージを会話に追加します。 | 
| **serviceUrl** | 文字列 | チャネルのサービス エンドポイントを指定する URL。 チャネルによって設定されます。 | 
| **speak** | 文字列 | 音声対応チャネルでボットが話すテキスト。 音声、速度、音量、発音、ピッチなど、ボットの音声のさまざまな特性を制御するには、<a href="https://msdn.microsoft.com/en-us/library/hh378377(v=office.14).aspx" target="_blank">音声合成マークアップ言語 (SSML)</a> 形式でこのプロパティを指定します。 |
| **suggestedActions** | [SuggestedActions](#suggestedactions-object) | ユーザーが選択できるオプションを定義する **SuggestedActions** オブジェクト。 |
| **summary** | 文字列 | メッセージに含まれる情報の要約。 たとえば、電子メール チャネルで送信されるメッセージの場合、このプロパティで電子メール メッセージの最初の 50 文字を指定できます。 |
| **text** | 文字列 | ユーザーからボットに、またはボットからユーザーに送信されるメッセージのテキスト。 このプロパティの内容に課される制限については、チャネルのドキュメントを参照してください。 |
| **textFormat** | 文字列 | メッセージの **text** の形式。 次の値のいずれか: **markdown**、**plain**、**xml**。 テキスト形式の詳細については、「[Create messages](bot-framework-rest-connector-create-messages.md)」(メッセージの作成) を参照してください。 |
| **timestamp** | 文字列 | UTC タイム ゾーンでメッセージが送信された日時を <a href="https://en.wikipedia.org/wiki/ISO_8601" target="_blank">ISO-8601</a> 形式で表したもの。 |
| **topicName** | 文字列 | アクティビティが属する会話のトピック。 |
| **type** | 文字列 | アクティビティの種類。 値は、**contactRelationUpdate**、**conversationUpdate**、**deleteUserData**、**message**、**typing**、**event**、**endOfConversation** のいずれかです。 アクティビティの種類の詳細については、「[Activities overview](bot-framework-rest-connector-activities.md)」(アクティビティの概要) を参照してください。 |
| **value** | オブジェクト | 拡張可能な値。 |
| **semanticAction** |[SemanticAction](#semanticaction-object) | プログラムによるアクションへの参照を表す **SemanticAction** オブジェクト。 |

<a href="#objects">スキーマの表に戻る</a>

### <a name="animationcard-object"></a>AnimationCard オブジェクト
アニメーション GIF または短いビデオを再生できるカードを定義します。<br/><br/> 

| プロパティ | type | 説明 |
|----|----|----|
| **autoloop** | ブール値 | 最後の項目が終了したときにアニメーション GIF のリストをリプレイするかどうかを示すフラグ。 アニメーションを自動的にリプレイするには、このプロパティを **true** に設定します。それ以外の場合、**false** に設定します。 既定値は **true** です。 |
| **autostart** | ブール値 | カードが表示されたときにアニメーションを自動的に再生するかどうかを示すフラグ。 アニメーションを自動的に再生するには、このプロパティを **true** に設定します。それ以外の場合、**false** に設定します。 既定値は **true** です。 |
| **buttons** | [CardAction](#cardaction-object)[] | ユーザーが 1 つ以上のアクションを実行できるようにする **CardAction** オブジェクトの配列。 指定できるボタンの数はチャネルが決定します。 |
| **duration** | 文字列 | メディア コンテンツの長さ ([ISO 8601 の期間の形式](https://www.iso.org/iso-8601-date-and-time-format.html))。 |
| **画像** | [ThumbnailUrl](#thumbnailurl-object) | カードに表示する画像を指定する **ThumbnailUrl** オブジェクト。 |
| **media** | [MediaUrl](#mediaurl-object)[] | 再生するアニメーション GIF のリストを指定する **MediaUrl** オブジェクトの配列。 |
| **shareable** | ブール値 | アニメーションを他のユーザーと共有できるかどうかを示すフラグ。 アニメーションを共有できる場合、このプロパティを **true** に設定します。それ以外の場合、**false** に設定します。 既定値は **true** です。 |
| **subtitle** | 文字列 | カードのタイトルの下に表示するサブタイトル。 |
| **text** | 文字列 | カードのタイトルまたはサブタイトルの下に表示する説明またはプロンプト。 |
| **title** | 文字列 | カードのタイトル。 |
| **value** | オブジェクト | このカードの補助パラメーター |

<a href="#objects">スキーマの表に戻る</a>

### <a name="attachment-object"></a>Attachment オブジェクト
メッセージに含める追加の情報を定義します。 メディア ファイル (例: オーディオ、ビデオ、画像、ファイル) またはリッチ カードを添付ファイルにすることができます。<br/><br/> 

| プロパティ | type | 説明 |
|----|----|----|
| **contentType** | 文字列 | 添付ファイル内のコンテンツのメディアの種類。 メディア ファイルの場合、このプロパティを **image/png**、**audio/wav**、**video/mp4** などの既知のメディア タイプに設定します。 リッチ カードの場合、このプロパティを次のベンダー固有タイプのいずれかに設定します。<ul><li>**application/vnd.microsoft.card.adaptive**: テキスト、音声、画像、ボタン、入力フィールドの任意の組み合わせを含めることができるリッチ カード。 **content** プロパティを <a href="http://adaptivecards.io/documentation/#create-cardschema" target="_blank">AdaptiveCard</a> オブジェクトに設定します。</li><li>**application/vnd.microsoft.card.animation**: アニメーションを再生するリッチ カード。 **content** プロパティを [AnimationCard](#animationcard-object) オブジェクトに設定します。</li><li>**application/vnd.microsoft.card.audio**: オーディオ ファイルを再生するリッチ カード。 **content** プロパティを [AudioCard](#audiocard-object) オブジェクトに設定します。</li><li>**application/vnd.microsoft.card.video**: ビデオを再生するリッチ カード。 **content** プロパティを [VideoCard](#videocard-object) オブジェクトに設定します。</li><li>**application/vnd.microsoft.card.hero**: ヒーロー カード。 **content** プロパティを [HeroCard](#herocard-object) オブジェクトに設定します。</li><li>**application/vnd.microsoft.card.thumbnail**: サムネイル カード。 **content** プロパティを [ThumbnailCard](#thumbnailcard-object) オブジェクトに設定します。</li><li>**application/vnd.microsoft.com.card.receipt**: 領収書カード。 **content** プロパティを [ReceiptCard](#receiptcard-object) オブジェクトに設定します。</li><li>**application/vnd.microsoft.com.card.signin**: ユーザー サインイン カード。 **content** プロパティを [SignInCard](#signincard-object) オブジェクトに設定します。</li></ul> |
| **contentUrl** | 文字列 | 添付ファイルのコンテンツの URL。 たとえば、添付ファイルが画像の場合、**contentUrl** を画像の場所を表す URL に設定します。 サポートされているプロトコルは、HTTP、HTTPS、File、Data です。 |
| **content** | オブジェクト | 添付ファイルのコンテンツ。 添付ファイルがリッチ カードの場合、このプロパティをリッチ カード オブジェクトに設定します。 このプロパティと **contentUrl** プロパティは相互に排他的です。 |
| **name** | 文字列 | 添付ファイルの名前。 |
| **thumbnailUrl** | 文字列 | より小さな形式の代替 **content** または **contentUrl** の使用をチャネルがサポートしている場合にチャネルが使用できるサムネイル画像の URL。 たとえば、**contentType** を **application/word** に設定し、**contentUrl** を Word 文書の場所に設定する場合、文書を表すサムネイル画像を含めることができます。 チャネルは文書の代わりにサムネイル画像を表示できます。 ユーザーが画像をクリックすると、チャネルは文書を開きます。 |

<a href="#objects">スキーマの表に戻る</a>

### <a name="attachmentdata-object"></a>AttachmentData オブジェクト 
添付ファイルのデータについて説明します。

| プロパティ | type | 説明 |
|----|----|----|
| **name** | 文字列 | 添付ファイルの名前。 |
| **originalBase64** | 文字列 | 添付ファイルのコンテンツ。 |
| **thumbnailBase64** | 文字列 | 添付ファイルのサムネイル コンテンツ。 |
| **type** | 文字列 | 添付ファイルの Content-Type。 |

### <a name="attachmentinfo-object"></a>AttachmentInfo オブジェクト
添付ファイルについて説明します。<br/><br/> 

| プロパティ | type | 説明 |
|----|----|----|
| **name** | 文字列 | 添付ファイルの名前。 |
| **type** | 文字列 | 添付ファイルの ContentType。 |
| **ビュー** | [AttachmentView](#attachmentview-object)[] | 添付ファイルの利用可能なビューを表す **AttachmentView** オブジェクトの配列。 |

<a href="#objects">スキーマの表に戻る</a>

### <a name="attachmentview-object"></a>AttachmentView オブジェクト
添付ファイルのビューを定義します。<br/><br/> 

| プロパティ | type | 説明 |
|----|----|----|
| **viewId** | 文字列 | ビュー ID。 |
| **サイズ** | number | ファイルのサイズ。 |

<a href="#objects">スキーマの表に戻る</a>

<!-- TODO - can't find in swagger file -->
### <a name="attachmentupload-object"></a>AttachmentUpload オブジェクト
アップロードする添付ファイルを定義します。<br/><br/> 

| プロパティ | type | 説明 |
|----|----|----|
| **type** | 文字列 | 添付ファイルの ContentType。 | 
| **name** | 文字列 | 添付ファイルの名前。 | 
| **originalBase64** | 文字列 | ファイルのオリジナル バージョンの内容を表すバイナリ データ。 |
| **thumbnailBase64** | 文字列 | ファイルのサムネイル バージョンの内容を表すバイナリ データ。 |

<a href="#objects">スキーマの表に戻る</a>

### <a name="audiocard-object"></a>AudioCard オブジェクト
オーディオ ファイルを再生できるカードを定義します。<br/><br/> 

| プロパティ | type | 説明 |
|----|----|----|
| **aspect** | 文字列 | **image** プロパティで指定されたサムネイルの縦横比。 有効な値は **16:9** および **9:16** です。 |
| **autoloop** | ブール値 | 最後の項目が終了したときにオーディオ ファイルのリストをリプレイするかどうかを示すフラグ。 オーディオ ファイルを自動的にリプレイするには、このプロパティを **true** に設定します。それ以外の場合、**false** に設定します。 既定値は **true** です。 |
| **autostart** | ブール値 | カードが表示されたときにオーディオを自動的に再生するかどうかを示すフラグ。 オーディオを自動的に再生するには、このプロパティを **true** に設定します。それ以外の場合、**false** に設定します。 既定値は **true** です。 |
| **buttons** | [CardAction](#cardaction-object)[] | ユーザーが 1 つ以上のアクションを実行できるようにする **CardAction** オブジェクトの配列。 指定できるボタンの数はチャネルが決定します。 |
| **duration** | 文字列 | メディア コンテンツの長さ ([ISO 8601 の期間の形式](https://www.iso.org/iso-8601-date-and-time-format.html))。 |
| **画像** | [ThumbnailUrl](#thumbnailurl-object) | カードに表示する画像を指定する **ThumbnailUrl** オブジェクト。 |
| **media** | [MediaUrl](#mediaurl-object)[] | 再生するオーディオ ファイルのリストを指定する **MediaUrl** オブジェクトの配列。 |
| **shareable** | ブール値 | オーディオ ファイルを他のユーザーと共有できるかどうかを示すフラグ。 オーディオを共有できる場合、このプロパティを **true** に設定します。それ以外の場合、**false** に設定します。 既定値は **true** です。 |
| **subtitle** | 文字列 | カードのタイトルの下に表示するサブタイトル。 |
| **text** | 文字列 | カードのタイトルまたはサブタイトルの下に表示する説明またはプロンプト。 |
| **title** | 文字列 | カードのタイトル。 |
| **value** | オブジェクト | このカードの補助パラメーター |

<a href="#objects">スキーマの表に戻る</a>

<!-- TODO - can't find in swagger file -->
### <a name="botdata-object"></a>BotData オブジェクト
Bot State サービスを使用して保存される特定の会話のコンテキストで、ユーザーの状態データ、会話、またはユーザーを定義します。<br/><br/>

| プロパティ | type | 説明 |
|----|----|----|
| **データ** | オブジェクト | 要求では、Bot State サービスを使用して保存するプロパティと値を指定する JSON オブジェクト。 応答では、Bot State サービスを使用して保存されたプロパティと値を指定する JSON オブジェクト。 | 
| **eTag** | 文字列 | Bot State サービスを使用して保存するデータのコンカレンシーを制御するために使用できるエンティティ タグ値。 詳細については、「[Manage state data](bot-framework-rest-state.md)」(状態データの管理) を参照してください。 | 

<a href="#objects">スキーマの表に戻る</a>

### <a name="cardaction-object"></a>CardAction オブジェクト
実行するアクションを定義します。<br/><br/> 

| プロパティ | type | 説明 |
|----|----|----|
| **画像** | 文字列 | 表示する画像の URL | 
| **text** | 文字列 | アクションのテキスト |
| **title** | 文字列 | ボタンのテキスト。 ボタンのアクションにのみ適用されます。 |
 ボタンを選択します。 ボタンのアクションにのみ適用されます。 |
| **type** | 文字列 | 実行するアクションの種類。 有効な値の一覧については、「[メッセージへのリッチ カード添付ファイルの追加](bot-framework-rest-connector-add-rich-cards.md)」を参照してください。 |
| **value** | オブジェクト | アクションの補助パラメーター。 このプロパティの値は、アクションの **type** によって異なります。 詳細については、「[メッセージへのリッチ カード添付ファイルの追加](bot-framework-rest-connector-add-rich-cards.md)」を参照してください。 |

<a href="#objects">スキーマの表に戻る</a>

### <a name="cardimage-object"></a>CardImage オブジェクト
カードに表示する画像を定義します。<br/><br/> 

| プロパティ | type | 説明 |
|----|----|----|
| **alt** | 文字列 | 画像の説明。 アクセシビリティをサポートするには、説明を含める必要があります。 |
| **tap** | [CardAction](#cardaction-object) | ユーザーが画像をタップまたはクリックした場合に実行するアクションを指定する **CardAction** オブジェクト。 |
| **URL** | 文字列 | 画像または画像の base64 バイナリのソース URL (例: `data:image/png;base64,iVBORw0KGgo...`)。 |

<a href="#objects">スキーマの表に戻る</a>

### <a name="channelaccount-object"></a>ChannelAccount オブジェクト
チャネルのボットまたはユーザー アカウントを定義します。<br/><br/>

| プロパティ | type | 説明 |
|----|----|----|
| **id** | 文字列 | チャネルのボットまたはユーザーを一意に識別する ID。 |
| **name** | 文字列 | ボットまたはユーザーの名前。 |

<a href="#objects">スキーマの表に戻る</a>

<!--TODO can't find-->
### <a name="conversation-object"></a>Conversation オブジェクト
会話を定義します (会話内に含まれているボットとユーザーなど)。<br/><br/> 

| プロパティ | type | 説明 |
|----|----|----|
| **bot** | [ChannelAccount](#channelaccount-object) | ボットを識別する **ChannelAccount** オブジェクト。 |
| **isGroup** | ブール値 | これがグループ会話であるかどうかを示すフラグ。 これがグループ会話の場合は **true** に設定します。それ以外の場合、**false** に設定します。 既定値は **false** です。 グループ会話を開始するには、チャネルがグループ会話をサポートしている必要があります。 |
| **members** | [ChannelAccount](#channelaccount-object)[] | 会話のメンバーを識別する **ChannelAccount** オブジェクトの配列。 **isGroup** が **true** に設定されていない限り、このリストには 1 人のユーザーが含まれている必要があります。 このリストには他のボットを含めることができます。 |
| **topicName** | 文字列 | 会話のタイトル。 |
| **activity** | [アクティビティ](#activity-object) | [会話を作成する](#create-conversation)要求で、新しい会話に投稿する最初のメッセージを定義する **Activity** オブジェクト。 |

<a href="#objects">スキーマの表に戻る</a>

### <a name="conversationaccount-object"></a>ConversationAccount オブジェクト
チャネルでの会話を定義します。<br/><br/>

| プロパティ | type | 説明 |
|----|----|----|
| **id** | 文字列 | 会話を識別する ID。 ID はチャネルごとに一意です。 チャネルが会話を開始する場合、チャネルがこの ID を設定します。それ以外の場合、このプロパティはボットによって、ボットが会話を開始するときに応答で取得する ID に設定されます (「Starting a conversation」(会話の開始) を参照)。 |
| **isGroup** | ブール値 | アクティビティが生成された時点で会話に 2 人より多い参加者が含まれているかどうかを示すフラグ。 これがグループ会話の場合は **true** に設定します。それ以外の場合、**false** に設定します。 既定値は **false** です。 |
| **name** | 文字列 | 会話を識別するために使用できる表示名。 |
| **conversationType** | 文字列 | 会話の種類 (グループ、個人など) を区別するチャネルでの会話の種類を示します。 |

<a href="#objects">スキーマの表に戻る</a>

### <a name="conversationparameters-object"></a>ConversationParameters オブジェクト
新しい会話を作成するためのパラメーターを定義します

| プロパティ | type | 説明 |
|----|----|----|
| **isGroup** | ブール値 | これがグループ会話かどうかを示します。 |
| **bot** | [ChannelAccount](#channelaccount-object) | 会話でのボットのアドレス。 |
| **members** | array | 会話に追加するメンバーのリスト。 |
| **topicName** | 文字列 | 会話のトピック タイトル。 このプロパティは、チャネルがサポートしている場合にのみ使用されます。 |
| **activity** | [アクティビティ](#activity-object) | (オプション) 新しい会話を作成するときに、このアクティビティを会話の初期メッセージとして使用します。 |
| **channelData** | オブジェクト | 会話を作成するためのチャネル固有のペイロード。 |

### <a name="conversationreference-object"></a>ConversationReference オブジェクト
会話内の特定のポイントを定義します。<br/><br/>

| プロパティ | type | 説明 |
|----|----|----|
| **activityId** | 文字列 | このオブジェクトが参照するアクティビティを一意に識別する ID。 | 
| **bot** | [ChannelAccount](#channelaccount-object) | このオブジェクトが参照する会話でボットを識別する **ChannelAccount** オブジェクト。 |
| **channelId** | 文字列 | このオブジェクトが参照する会話でチャネルを一意に識別する ID。 | 
| **conversation** | [ConversationAccount](#conversationaccount-object) | このオブジェクトが参照する会話を定義する **ConversationAccount** オブジェクト。 |
| **serviceUrl** | 文字列 | このオブジェクトが参照する会話でチャネルのサービス エンドポイントを指定する URL。 | 
| **user** | [ChannelAccount](#channelaccount-object) | このオブジェクトが参照する会話でユーザーを識別する **ChannelAccount** オブジェクト。 |

<a href="#objects">スキーマの表に戻る</a>

### <a name="conversationresourceresponse-object"></a>ConversationResourceResponse オブジェクト
リソースを含む応答を定義します。

| プロパティ | type | 説明 |
|----|----|----|
| **activityId** | 文字列 | アクティビティの ID。 |
| **id** | 文字列 | リソースの ID。 |
| **serviceUrl** | 文字列 | サービス エンドポイント。 |

### <a name="error-object"></a>Error オブジェクト
エラーを定義します。<br/><br/>

| プロパティ | type | 説明 |
|----|----|----|
| **code** | 文字列 | エラー コード。 |
| **message** | 文字列 | エラーの説明。 |

<a href="#objects">スキーマの表に戻る</a>

### <a name="entity-object"></a>Entity オブジェクト
エンティティ オブジェクトを定義します。

| プロパティ | type | 説明 |
|----|----|----|
| **type** | 文字列 | エンティティの種類。 通常は、schema.org からの種類が含まれます。 |


### <a name="errorresponse-object"></a>ErrorResponse オブジェクト
HTTP API 応答を定義します。<br/><br/> 

| プロパティ | type | 説明 |
|----|----|----|
| **error** | [Error](#error-object) | エラーに関する情報を含む **Error** オブジェクト。 |

<a href="#objects">スキーマの表に戻る</a>

### <a name="fact-object"></a>Fact オブジェクト
ファクトを含むキーと値のペアを定義します。<br/><br/> 

| プロパティ | type | 説明 |
|----|----|----|
| **key** | 文字列 | ファクトの名前。 例: **Check-in**。 キーは、ファクトの値を表示するときにラベルとして使用されます。 |
| **value** | 文字列 | ファクトの値。 例: **10 October 2016**。 |

<a href="#objects">スキーマの表に戻る</a>

### <a name="geocoordinates-object"></a>Geocoordinates オブジェクト
World Geodetic System (WSG84) 座標を使用して地理的な場所を定義します。<br/><br/> 

| プロパティ | type | 説明 |
|----|----|----|
| **elevation** | number | 場所の標高。 |
| **name** | 文字列 | 場所の名前。 |
| **latitude** | number | 場所の緯度。 |
| **longitude** | number | 場所の経度。 |
| **type** | 文字列 | このオブジェクトの種類。 常に **GeoCoordinates** に設定します。 |

<a href="#objects">スキーマの表に戻る</a>

### <a name="herocard-object"></a>HeroCard オブジェクト
大きい画像、タイトル、テキスト、アクションの各ボタンを持つカードを定義します。<br/><br/> 

| プロパティ | type | 説明 |
|----|----|----|
| **buttons** | [CardAction](#cardaction-object)[] | ユーザーが 1 つ以上のアクションを実行できるようにする **CardAction** オブジェクトの配列。 指定できるボタンの数はチャネルが決定します。 |
| **images** | [CardImage](#cardimage-object)[] | カードに表示する画像を指定する **CardImage** オブジェクトの配列。 ヒーロー カードには 1 つの画像のみが含まれます。 |
| **subtitle** | 文字列 | カードのタイトルの下に表示するサブタイトル。 |
| **tap** | [CardAction](#cardaction-object) | ユーザーがカードをタップまたはクリックした場合に実行するアクションを指定する **CardAction** オブジェクト。 いずれかのボタンと同じアクションまたは別のアクションを指定できます。 |
| **text** | 文字列 | カードのタイトルまたはサブタイトルの下に表示する説明またはプロンプト。 |
| **title** | 文字列 | カードのタイトル。 |

<a href="#objects">スキーマの表に戻る</a>

<!--TODO can't find-->
### <a name="identification-object"></a>Identification オブジェクト
リソースを識別します。<br/><br/> 

| プロパティ | type | 説明 |
|----|----|----|
| **id** | 文字列 | リソースを一意に識別する ID。 |

<a href="#objects">スキーマの表に戻る</a>

### <a name="mediaeventvalue-object"></a>MediaEventValue オブジェクト 
メディア イベントの補助パラメーター。

| プロパティ | type | 説明 |
|----|----|----|
| **cardValue** | オブジェクト | このイベントを発生させたメディア カードの **Value** フィールドで指定されたコールバック パラメーター。 |

### <a name="mediaurl-object"></a>MediaUrl オブジェクト
メディア ファイルのソース URL を定義します。<br/><br/> 

| プロパティ | type | 説明 |
|----|----|----|
| **profile** | 文字列 | メディアのコンテンツを説明するヒント。 |
| **URL** | 文字列 | メディア ファイルのソース URL。 |

<a href="#objects">スキーマの表に戻る</a>

<!--TODO can't find-->
### <a name="mention-object"></a>Mention オブジェクト
会話でメンションされたユーザーまたはボットを定義します。<br/><br/> 


|          プロパティ          |                   type                   |                                                                                                                                                                                                                           説明                                                                                                                                                                                                                            |
|----------------------------|------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| <strong>mentioned</strong> | [ChannelAccount](#channelaccount-object) | メンションされたユーザーまたはボットを指定する <strong>ChannelAccount</strong> オブジェクト。 Slack などの一部のチャネルでは会話ごとに名前を割り当てるため、(メッセージの <strong>recipient</strong> プロパティで) メンションされたボットの名前は、ボットの[登録](../bot-service-quickstart-registration.md)時に指定したハンドルとは異なる可能性があることに注意してください。 ただし、両方のアカウント ID は同じになります。 |
|   <strong>text</strong>    |                  文字列                  |                                                                                                                         会話でメンションされたユーザーまたはボット。 たとえば、メッセージが「@ColorBot pick me a new color」の場合、このプロパティは <strong>@ColorBot</strong> に設定されます。 すべてのチャネルがこのプロパティを設定するわけではありません。                                                                                                                          |
|   <strong>type</strong>    |                  文字列                  |                                                                                                                                                                                                   このオブジェクトの種類。 常に <strong>Mention</strong> に設定します。                                                                                                                                                                                                    |

<a href="#objects">スキーマの表に戻る</a>

### <a name="messagereaction-object"></a>MessageReaction オブジェクト
メッセージへの反応を定義します。

| プロパティ | type | 説明 |
|----|----|----|
| **type** | 文字列 | 反応の種類。 |

### <a name="place-object"></a>Place オブジェクト
会話でメンションされた場所を定義します。<br/><br/> 

| プロパティ | type | 説明 |
|----|----|----|
| **address** | オブジェクト |  場所の所在地。 このプロパティは `string`、または `PostalAddress` 型の複合オブジェクトです。 |
| **geo** | [GeoCoordinates](#geocoordinates-object) | 場所の地理的座標を指定する **GeoCoordinates** オブジェクト。 |
| **hasMap** | オブジェクト | 場所の地図。 このプロパティは `string` (URL)、または `Map` 型の複合オブジェクトです。 |
| **name** | 文字列 | 場所の名前。 |
| **type** | 文字列 | このオブジェクトの種類。 常に **Place** に設定します。 |

<a href="#objects">スキーマの表に戻る</a>

### <a name="receiptcard-object"></a>ReceiptCard オブジェクト
購入のレシートを含むカードを定義します。<br/><br/>

| プロパティ | type | 説明 |
|----|----|----|
| **buttons** | [CardAction](#cardaction-object)[] | ユーザーが 1 つ以上のアクションを実行できるようにする **CardAction** オブジェクトの配列。 指定できるボタンの数はチャネルが決定します。 |
| **facts** | [Fact](#fact-object)[] | 購入に関する情報を指定する **Fact** オブジェクトの配列。 たとえば、ホテル宿泊レシートのファクトのリストには、チェックイン日とチェックアウト日が含まれる場合があります。 指定できるファクトの数はチャネルが決定します。 |
| **items** | [ReceiptItem](#receiptitem-object)[] | 購入したアイテムを指定する **ReceiptItem** オブジェクトの配列 |
| **tap** | [CardAction](#cardaction-object) | ユーザーがカードをタップまたはクリックした場合に実行するアクションを指定する **CardAction** オブジェクト。 いずれかのボタンと同じアクションまたは別のアクションを指定できます。 |
| **tax** | 文字列 | 購入に適用される税額を指定する通貨形式の文字列。 |
| **title** | 文字列 | レシートの上部に表示されるタイトル。 |
| **total** | 文字列 | 適用されるすべての税金を含めた合計購入価格を指定する通貨形式の文字列。 |
| **vat** | 文字列 | 購入価格に適用される付加価値税 (VAT) の額を指定する通貨形式の文字列。 |

<a href="#objects">スキーマの表に戻る</a>

### <a name="receiptitem-object"></a>ReceiptItem オブジェクト
レシート内の品目を定義します。<br/><br/> 

| プロパティ | type | 説明 |
|----|----|----|
| **画像** | [CardImage](#cardimage-object) | 品目の隣に表示するサムネイル画像を指定する **CardImage** オブジェクト。  |
| **price** | 文字列 | すべての購入単位の合計価格を指定する通貨形式の文字列。 |
| **quantity** | 文字列 | 購入単位数を指定する数値の文字列。 |
| **subtitle** | 文字列 | 品目のタイトルの下に表示されるサブタイトル。 |
| **tap** | [CardAction](#cardaction-object) | ユーザーが品目をタップまたはクリックした場合に実行するアクションを指定する **CardAction** オブジェクト。 |
| **text** | 文字列 | 品目の説明。 |
| **title** | 文字列 | 品目のタイトル。 |

<a href="#objects">スキーマの表に戻る</a>

### <a name="resourceresponse-object"></a>ResourceResponse オブジェクト
リソース ID を含む応答を定義します。<br/><br/>


|      プロパティ       |  type  |                説明                |
|---------------------|--------|-------------------------------------------|
| <strong>id</strong> | 文字列 | リソースを一意に識別する ID。 |

<a href="#objects">スキーマの表に戻る</a>

### <a name="signincard-object"></a>SignInCard オブジェクト
ユーザーがサービスにサインインできるようにするカードを定義します。<br/><br/>

| プロパティ | type | 説明 |
|----|----|----|
| **buttons** | [CardAction](#cardaction-object)[] | ユーザーがサービスにサインインできるようにする **CardAction** オブジェクトの配列。 指定できるボタンの数はチャネルが決定します。 |
| **text** | 文字列 | サインイン カードに含める説明またはプロンプト。 |

<a href="#objects">スキーマの表に戻る</a>

### <a name="suggestedactions-object"></a>SuggestedActions オブジェクト
ユーザーが選択できるオプションを定義します。<br/><br/> 

| プロパティ | type | 説明 |
|----|----|----|
| **actions** | [CardAction](#cardaction-object)[] | 推奨されるアクションを定義する **CardAction** オブジェクトの配列。 |
| **to** | string[] | 推奨されるアクションの表示先である受信者の ID を含む文字列の配列。 |

<a href="#objects">スキーマの表に戻る</a>

### <a name="thumbnailcard-object"></a>ThumbnailCard オブジェクト
サムネイル画像、タイトル、テキスト、アクションの各ボタンを持つカードを定義します。<br/><br/>

| プロパティ | type | 説明 |
|----|----|----|
| **buttons** | [CardAction](#cardaction-object)[] | ユーザーが 1 つ以上のアクションを実行できるようにする **CardAction** オブジェクトの配列。 指定できるボタンの数はチャネルが決定します。 |
| **images** | [CardImage](#cardimage-object)[] | カードに表示するサムネイル画像を指定する **CardImage** オブジェクトの配列。 指定できるサムネイル画像の数はチャネルが決定します。 |
| **subtitle** | 文字列 | カードのタイトルの下に表示するサブタイトル。 |
| **tap** | [CardAction](#cardaction-object) | ユーザーがカードをタップまたはクリックした場合に実行するアクションを指定する **CardAction** オブジェクト。 いずれかのボタンと同じアクションまたは別のアクションを指定できます。 |
| **text** | 文字列 | カードのタイトルまたはサブタイトルの下に表示する説明またはプロンプト。 |
| **title** | 文字列 | カードのタイトル。 |

<a href="#objects">スキーマの表に戻る</a>

### <a name="thumbnailurl-object"></a>ThumbnailUrl オブジェクト
画像のソース URL を定義します。<br/><br/> 

| プロパティ | type | 説明 |
|----|----|----|
| **alt** | 文字列 | 画像の説明。 アクセシビリティをサポートするには、説明を含める必要があります。 |
| **URL** | 文字列 | 画像または画像の base64 バイナリのソース URL (例: `data:image/png;base64,iVBORw0KGgo...`)。 |

<a href="#objects">スキーマの表に戻る</a>

### <a name="videocard-object"></a>VideoCard オブジェクト
ビデオを再生できるカードを定義します。<br/><br/>

| プロパティ | type | 説明 |
|----|----|----|
| **aspect** | 文字列 | ビデオの縦横比 (例: 16:9、4:3)。|
| **autoloop** | ブール値 | 最後の項目が終了したときにビデオのリストをリプレイするかどうかを示すフラグ。 ビデオを自動的にリプレイするには、このプロパティを **true** に設定します。それ以外の場合、**false** に設定します。 既定値は **true** です。 |
| **autostart** | ブール値 | カードが表示されたときにビデオを自動的に再生するかどうかを示すフラグ。 ビデオを自動的に再生するには、このプロパティを **true** に設定します。それ以外の場合、**false** に設定します。 既定値は **true** です。 |
| **buttons** | [CardAction](#cardaction-object)[] | ユーザーが 1 つ以上のアクションを実行できるようにする **CardAction** オブジェクトの配列。 指定できるボタンの数はチャネルが決定します。 |
| **duration** | 文字列 | メディア コンテンツの長さ ([ISO 8601 の期間の形式](https://www.iso.org/iso-8601-date-and-time-format.html))。 |
| **画像** | [ThumbnailUrl](#thumbnailurl-object) | カードに表示する画像を指定する **ThumbnailUrl** オブジェクト。 |
| **media** | [MediaUrl](#mediaurl-object)[] | 再生するビデオのリストを指定する **MediaUrl** オブジェクトの配列。 |
| **shareable** | ブール値 | ビデオを他のユーザーと共有できるかどうかを示すフラグ。 ビデオを共有できる場合、このプロパティを **true** に設定します。それ以外の場合、**false** に設定します。 既定値は **true** です。 |
| **subtitle** | 文字列 | カードのタイトルの下に表示するサブタイトル。 |
| **text** | 文字列 | カードのタイトルまたはサブタイトルの下に表示する説明またはプロンプト。 |
| **title** | 文字列 | カードのタイトル。 |
| **value** | オブジェクト | このカードの補助パラメーター|

<a href="#objects">スキーマの表に戻る</a>

### <a name="semanticaction-object"></a>SemanticAction オブジェクト
プログラムによるアクションへの参照を定義します。<br/><br/>

| プロパティ | type | 説明 |
|----|----|----|
| **id** | 文字列 | このアクションの ID |
| **entities** | [エンティティ](#entity-object) | このアクションに関連付けられているエンティティ |

<a href="#objects">スキーマの表に戻る</a>
