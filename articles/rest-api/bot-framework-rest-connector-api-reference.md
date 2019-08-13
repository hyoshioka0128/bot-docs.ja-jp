---
title: API リファレンス | Microsoft Docs
description: Bot Connector サービスと Bot State サービスのヘッダー、操作、オブジェクト、およびエラーについて説明します。
author: ivorb
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 10/25/2018
ms.openlocfilehash: f8f04c8b0cbd2b43f29676f0315739f4cc7716b3
ms.sourcegitcommit: a1eaa44f182a7210197bd793250907df00e9edab
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 08/03/2019
ms.locfileid: "68757172"
---
# <a name="api-reference"></a>API リファレンス

> [!NOTE]
> REST API は SDK に相当するものではありません。 REST API は標準の REST 通信を可能にするために提供されていますが、Bot Framework との好ましい対話方法は SDK です。 

Bot Framework 内で Bot Connector サービスを使用することにより、ボットは、Bot Framework Portal に構成されているチャネルでユーザーとメッセージを交換できます。 このサービスでは、HTTPS で業界標準の REST および JSON が使用されます。

## <a name="base-uri"></a>ベース URI

ユーザーがボットにメッセージを送信すると、受信要求には、ボットが応答を送信するエンドポイントを指定する `serviceUrl` プロパティを持つ [Activity](#bot-framework-activity-schema) オブジェクトが含まれています。 Bot Connector サービスにアクセスするには、`serviceUrl` 値を API 要求のベース URI として使用します。 

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

4xx 範囲または 5xx 範囲の HTTP 状態コードを指定する応答では、エラーの情報を提供する [ErrorResponse](#bot-framework-activity-schema) オブジェクトが応答の本文に含まれます。 4xx 範囲のエラー応答を受け取った場合、要求を再送信する前に、**ErrorResponse** オブジェクトを検査してエラーの原因を識別し、問題を解決してください。

## <a name="conversation-operations"></a>会話操作 
これらの操作を使用して、会話を作成し、メッセージ (アクティビティ) を送信し、会話の内容を管理します。

| Operation | 説明 |
|----|----|
| [会話を作成する](#create-conversation) | 新しい会話を作成します。 |
| [会話に送信する](#send-to-conversation) | 指定された会話の最後にアクティビティ (メッセージ) を送信します。 |
| [アクティビティに返信する](#reply-to-activity) | 指定されたアクティビティへの返信として、指定された会話にアクティビティ (メッセージ) を送信します。 |
| [会話を取得する](#get-conversations) | ボットが参加した会話のリストを取得します。 |
| [会話のメンバーを取得する](#get-conversation-members) | 指定された会話のメンバーを取得します。 |
| [会話のページングされたメンバーを取得する](#get-conversation-paged-members) | 指定された会話のメンバーを一度に 1 ページずつ取得します。 |
| [アクティビティのメンバーを取得する](#get-activity-members) | 指定された会話内の指定されたアクティビティのメンバーを取得します。 |
| [アクティビティを更新する](#update-activity) | 既存のアクティビティを更新します。 |
| [アクティビティを削除する](#delete-activity) | 既存のアクティビティを削除します。 |
| [会話のメンバーを削除する](#delete-conversation-member) | 会話からメンバーを削除します。 |
| [会話履歴を送信する](#send-conversation-history) | 過去のアクティビティのトランスクリプトを会話にアップロードします。 |
| [添付ファイルをチャネルにアップロードする](#upload-attachment-to-channel) | 添付ファイルをチャネルの BLOB ストレージに直接アップロードします。 |

### <a name="create-conversation"></a>会話を作成する
新しい会話を作成します。
```http 
POST /v3/conversations
```

| | |
|----|----|
| **要求本文** | `ConversationParameters` オブジェクト |
| **戻り値** | `ConversationResourceResponse` オブジェクト |

### <a name="send-to-conversation"></a>会話に送信する
指定された会話にアクティビティ (メッセージ) を送信します。 アクティビティは、タイムスタンプまたはチャネルのセマンティクスに従って会話の最後に追加されます。 会話内の特定のメッセージに返信するには、代わりに[アクティビティに返信する](#reply-to-activity)を使用します。
```http
POST /v3/conversations/{conversationId}/activities
```

| | |
|----|----|
| **要求本文** | [Activity](#bot-framework-activity-schema) オブジェクト |
| **戻り値** | [Identification](#bot-framework-activity-schema) オブジェクト | 

### <a name="reply-to-activity"></a>アクティビティに返信する
指定されたアクティビティへの返信として、指定された会話にアクティビティ (メッセージ) を送信します。 チャネルがサポートしている場合、アクティビティは別のアクティビティへの返信として追加されます。 ネストした応答をチャネルがサポートしていない場合、この操作は[会話に送信する](#send-to-conversation)と同様に動作します。
```http
POST /v3/conversations/{conversationId}/activities/{activityId}
```

| | |
|----|----|
| **要求本文** | [Activity](#bot-framework-activity-schema) オブジェクト |
| **戻り値** | [Identification](#bot-framework-activity-schema) オブジェクト | 

### <a name="get-conversations"></a>会話を取得する
ボットが参加した会話のリストを取得します。
```http
GET /v3/conversations?continuationToken={continuationToken}
```

| | |
|----|----|
| **要求本文** | 該当なし |
| **戻り値** | [ConversationsResult](#bot-framework-activity-schema) オブジェクト | 

### <a name="get-conversation-members"></a>会話のメンバーを取得する
指定された会話のメンバーを取得します。
```http
GET /v3/conversations/{conversationId}/members
```

| | |
|----|----|
| **要求本文** | 該当なし |
| **戻り値** | [ChannelAccount](#bot-framework-activity-schema) オブジェクトの配列 | 

### <a name="get-conversation-paged-members"></a>会話のページングされたメンバーを取得する
指定された会話のメンバーを一度に 1 ページずつ取得します。
```http
GET /v3/conversations/{conversationId}/pagedmembers?pageSize={pageSize}&continuationToken={continuationToken}
```

| | |
|----|----|
| **要求本文** | 該当なし |
| **戻り値** | [ChannelAccount](#bot-framework-activity-schema) オブジェクトの配列と、さらに値を取得するときに使用できる継続トークン |

### <a name="get-activity-members"></a>アクティビティのメンバーを取得する
指定された会話内の指定されたアクティビティのメンバーを取得します。
```http
GET /v3/conversations/{conversationId}/activities/{activityId}/members
```

| | |
|----|----|
| **要求本文** | 該当なし |
| **戻り値** | [ChannelAccount](#bot-framework-activity-schema) オブジェクトの配列 | 

### <a name="update-activity"></a>アクティビティを更新する
一部のチャネルでは、既存のアクティビティを編集してボットの会話の新しい状態を反映させることができます。 たとえば、いずれかのボタンをユーザーがクリックした後、会話内のメッセージからボタンを削除することができます。 成功すると、この操作は指定された会話内の指定されたアクティビティを更新します。 
```http
PUT /v3/conversations/{conversationId}/activities/{activityId}
```

| | |
|----|----|
| **要求本文** | [Activity](#bot-framework-activity-schema) オブジェクト |
| **戻り値** | [Identification](#bot-framework-activity-schema) オブジェクト | 

### <a name="delete-activity"></a>アクティビティを削除する
一部のチャネルでは、既存のアクティビティを削除できます。 成功すると、この操作は指定されたアクティビティを指定された会話から削除します。
```http
DELETE /v3/conversations/{conversationId}/activities/{activityId}
```

| | |
|----|----|
| **要求本文** | 該当なし |
| **戻り値** | 操作の結果を示す HTTP 状態コード。 応答の本文では何も指定されません。 | 

### <a name="delete-conversation-member"></a>会話のメンバーを削除する
会話からメンバーを削除します。 そのメンバーが会話の最後のメンバーだった場合は、会話も削除されます。
```http
DELETE /v3/conversations/{conversationId}/members/{memberId}
```

| | |
|----|----|
| **要求本文** | 該当なし |
| **戻り値** | 操作の結果を示す HTTP 状態コード。 応答の本文では何も指定されません。 | 

### <a name="send-conversation-history"></a>会話履歴を送信する
クライアントが過去のアクティビティのトランスクリプトをレンダリングできるように、それらを会話にアップロードします。
```http
POST /v3/conversations/{conversationId}/activities/history
```

| | |
|----|----|
| **要求本文** | [Transcript](#bot-framework-activity-schema) オブジェクト。 |
| **戻り値** | [ResourceResponse](#bot-framework-activity-schema) オブジェクト。 | 

### <a name="upload-attachment-to-channel"></a>添付ファイルをチャネルにアップロード
指定された会話の添付ファイルをチャネルの BLOB ストレージに直接アップロードします。 これにより、準拠したストアにデータを保存できます。
```http 
POST /v3/conversations/{conversationId}/attachments
```

| | |
|----|----|
| **要求本文** | [AttachmentUpload](#bot-framework-activity-schema) オブジェクト。 |
| **戻り値** | [ResourceResponse](#bot-framework-activity-schema) オブジェクト。 **id** プロパティは、[添付ファイル情報を取得する](#get-attachment-info)操作と[添付ファイルを取得する](#get-attachment)操作で使用できる添付ファイル ID を指定します。 | 

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
| **戻り値** | [AttachmentInfo](#bot-framework-activity-schema) オブジェクト | 

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
Microsoft Bot Framework State サービスは 2018 年 3 月 30 日時点で廃止されています。 以前は、Azure Bot Service または Bot Builder SDK で構築されたボットには、ボットの状態データを格納するために Microsoft によってホストされたこのサービスへの既定の接続がありました。 ボットは、独自の状態ストレージを使用するように更新する必要があります。

| Operation | 説明 |
|----|----|
| `Set User Data` | チャネルの特定のユーザーの状態データを保存します。 |
| `Set Conversation Data` | チャネルの特定の会話の状態データを保存します。 |
| `Set Private Conversation Data` | チャネルの特定の会話のコンテキスト内で特定のユーザーの状態データを保存します。 |
| `Get User Data` | チャネルのすべての会話で以前に保存した特定のユーザーの状態データを取得します。 |
| `Get Conversation Data` | チャネルの特定の会話で以前に保存した状態データを取得します。 |
| `Get Private Conversation Data` | チャネルの特定の会話のコンテキスト内で以前に保存した特定のユーザーの状態データを取得します。 |
| `Delete State For User` | ユーザーのために以前に格納された状態データを削除します。 |

### <a name="set-user-data"></a>ユーザー データを設定する
指定されたチャネルの指定されたユーザーの状態データを保存します。
```http
POST /v3/botstate/{channelId}/users/{userId} 
```

| | |
|----|----|
| **要求本文** | `BotData` オブジェクト |
| **戻り値** | `BotData` オブジェクト | 

### <a name="set-conversation-data"></a>会話データを設定する
指定されたチャネルの指定された会話の状態データを保存します。
```http
POST /v3/botstate/{channelId}/conversations/{conversationId}
```

| | |
|----|----|
| **要求本文** | `BotData` オブジェクト |
| **戻り値** | `BotData` オブジェクト | 

### <a name="set-private-conversation-data"></a>個人的な会話データを設定する
指定されたチャネルの指定された会話のコンテキスト内で、指定されたユーザーの状態データを保存します。
```http
POST /v3/botstate/{channelId}/conversations/{conversationId}/users/{userId} 
```

| | |
|----|----|
| **要求本文** | `BotData` オブジェクト |
| **戻り値** | `BotData` オブジェクト | 

### <a name="get-user-data"></a>ユーザー データを取得する
指定されたチャネルのすべての会話で以前に保存した特定のユーザーの状態データを取得します。
```http
GET /v3/botstate/{channelId}/users/{userId} 
```

| | |
|----|----|
| **要求本文** | 該当なし |
| **戻り値** | `BotData` オブジェクト | 

### <a name="get-conversation-data"></a>会話データを取得する
指定されたチャネルの特定の会話で以前に保存した状態データを取得します。
```http
GET /v3/botstate/{channelId}/conversations/{conversationId} 
```

| | |
|----|----|
| **要求本文** | 該当なし |
| **戻り値** | `BotData` オブジェクト | 

### <a name="get-private-conversation-data"></a>個人的な会話データを取得する
指定されたチャネルの指定された会話のコンテキスト内で以前に保存した特定のユーザーの状態データを取得します。
```http
GET /v3/botstate/{channelId}/conversations/{conversationId}/users/{userId} 
```

| | |
|----|----|
| **要求本文** | 該当なし |
| **戻り値** | [`BotData` オブジェクト | 

### <a name="delete-state-for-user"></a>ユーザーの状態を削除する
[ユーザー データを設定する](#set-user-data)操作または[個人的な会話データを設定する](#set-private-conversation-data)操作のどちらかを使用して以前に保存した特定のチャネルの特定のユーザーの状態データを削除します。
```http
DELETE /v3/botstate/{channelId}/users/{userId} 
```

| | |
|----|----|
| **要求本文** | 該当なし |
| **戻り値** | 文字列の配列 (ID) | 

## <a name="bot-framework-activity-schema"></a>Bot Framework アクティビティ スキーマ

ボットがユーザーとの通信に使用できるオブジェクトとプロパティについては、「[Bot Framework アクティビティ スキーマ](https://aka.ms/botSpecs-activitySchema)」をご覧ください。
