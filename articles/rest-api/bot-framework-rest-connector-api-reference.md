---
title: API リファレンス - Bot Service
description: Bot Connector サービスと Bot State サービスのヘッダー、操作、オブジェクト、およびエラーについて説明します。
author: ivorb
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 08/02/2019
ms.openlocfilehash: 7788a3bc8ecc3ee01cd44b25297a637f8fc07b01
ms.sourcegitcommit: f8b5cc509a6351d3aae89bc146eaabead973de97
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/09/2020
ms.locfileid: "75790073"
---
# <a name="api-reference"></a>API リファレンス

> [!NOTE]
> REST API は SDK に相当するものではありません。 REST API は標準の REST 通信を可能にするために提供されていますが、Bot Framework との好ましい対話方法は SDK です。

Bot Framework 内で Bot Connector サービスを使用することにより、ボットは、Bot Framework Portal に構成されているチャネルでユーザーとメッセージを交換できます。 このサービスでは、HTTPS で業界標準の REST および JSON が使用されます。

## <a name="base-uri"></a>ベース URI

ユーザーがボットにメッセージを送信すると、受信要求には、ボットが応答を送信するエンドポイントを指定する `serviceUrl` プロパティを持つ [Activity](#activity-object) オブジェクトが含まれています。 Bot Connector サービスにアクセスするには、`serviceUrl` 値を API 要求のベース URI として使用します。

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

## <a name="headers"></a>ヘッダー

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

### <a name="errors"></a>エラー

4xx 範囲または 5xx 範囲の HTTP 状態コードを指定する応答では、エラーの情報を提供する [ErrorResponse](#errorresponse-object) オブジェクトが応答の本文に含まれます。 4xx 範囲のエラー応答を受け取った場合、要求を再送信する前に、**ErrorResponse** オブジェクトを検査してエラーの原因を識別し、問題を解決してください。

## <a name="conversation-operations"></a>会話操作

これらの操作を使用して、会話を作成し、メッセージ (アクティビティ) を送信し、会話の内容を管理します。

| 操作 | [説明] |
|----|----|
| [会話を作成する](#create-conversation) | 新しい会話を作成します。 |
| [会話に送信する](#send-to-conversation) | 指定された会話の最後にアクティビティ (メッセージ) を送信します。 |
| [アクティビティに返信する](#reply-to-activity) | 指定されたアクティビティへの返信として、指定された会話にアクティビティ (メッセージ) を送信します。 |
| [会話を取得する](#get-conversations) | ボットが参加した会話のリストを取得します。 |
| [会話のメンバーを取得する](#get-conversation-members) | 指定された会話のメンバーを取得します。 |
| [会話のページングされたメンバーを取得する](#get-conversation-paged-members) | 指定された会話のメンバーを一度に 1 ページずつ取得します。 |
| [アクティビティのメンバーを取得する](#get-activity-members) | 指定された会話内の指定されたアクティビティのメンバーを取得します。 |
| [アクティビティを更新する](#update-activity) | 既存のアクティビティを更新します。 |
| [削除アクティビティ](#delete-activity) | 既存のアクティビティを削除します。 |
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
| **要求本文** | [ConversationParameters](#conversationparameters-object) オブジェクト |
| **戻り値** | [ConversationResourceResponse](#conversationresourceresponse-object) オブジェクト |

### <a name="send-to-conversation"></a>会話に送信する

指定された会話にアクティビティ (メッセージ) を送信します。 アクティビティは、タイムスタンプまたはチャネルのセマンティクスに従って会話の最後に追加されます。 会話内の特定のメッセージに返信するには、代わりに[アクティビティに返信する](#reply-to-activity)を使用します。

```http
POST /v3/conversations/{conversationId}/activities
```

| | |
|----|----|
| **要求本文** | [Activity](#activity-object) オブジェクト |
| **戻り値** | [ResourceResponse](#resourceresponse-object) オブジェクト |

### <a name="reply-to-activity"></a>アクティビティに返信する

指定されたアクティビティへの返信として、指定された会話にアクティビティ (メッセージ) を送信します。 チャネルがサポートしている場合、アクティビティは別のアクティビティへの返信として追加されます。 ネストした応答をチャネルがサポートしていない場合、この操作は[会話に送信する](#send-to-conversation)と同様に動作します。

```http
POST /v3/conversations/{conversationId}/activities/{activityId}
```

| | |
|----|----|
| **要求本文** | [Activity](#activity-object) オブジェクト |
| **戻り値** | [ResourceResponse](#resourceresponse-object) オブジェクト |

### <a name="get-conversations"></a>会話を取得する

ボットが参加した会話のリストを取得します。

```http
GET /v3/conversations?continuationToken={continuationToken}
```

| | |
|----|----|
| **要求本文** | 該当なし |
| **戻り値** | [ConversationsResult](#conversationsresult-object) オブジェクト |

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
GET /v3/conversations/{conversationId}/pagedmembers?pageSize={pageSize}&continuationToken={continuationToken}
```

| | |
|----|----|
| **要求本文** | 該当なし |
| **戻り値** | [PagedMembersResult](#pagedmembersresult-object) オブジェクト |

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
| **戻り値** | [ResourceResponse](#resourceresponse-object) オブジェクト |

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
| **要求本文** | [Transcript](#transcript-object) オブジェクト。 |
| **戻り値** | [ResourceResponse](#resourceresponse-object) オブジェクト。 |

### <a name="upload-attachment-to-channel"></a>添付ファイルをチャネルにアップロード

指定された会話の添付ファイルをチャネルの BLOB ストレージに直接アップロードします。 これにより、準拠したストアにデータを保存できます。

```http
POST /v3/conversations/{conversationId}/attachments
```

| | |
|----|----|
| **要求本文** | [AttachmentData](#attachmentdata-object) オブジェクト。 |
| **戻り値** | [ResourceResponse](#resourceresponse-object) オブジェクト。 **id** プロパティは、[添付ファイル情報を取得する](#get-attachment-info)操作と[添付ファイルを取得する](#get-attachment)操作で使用できる添付ファイル ID を指定します。 |

## <a name="attachment-operations"></a>添付ファイル操作

これらの操作を使用して、添付ファイルの情報とファイル自体のバイナリ データを取得します。

| 操作 | [説明] |
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

## <a name="state-operations-deprecated"></a>状態操作 (非推奨)

Microsoft Bot Framework State サービスは 2018 年 3 月 30 日時点で廃止されています。 以前は、Azure Bot Service または Bot Builder SDK で構築されたボットには、ボットの状態データを格納するために Microsoft によってホストされたこのサービスへの既定の接続がありました。 ボットは、独自の状態ストレージを使用するように更新する必要があります。

| 操作 | [説明] |
|----|----|
| `Set User Data` | チャネルの特定のユーザーの状態データを保存します。 |
| `Set Conversation Data` | チャネルの特定の会話の状態データを保存します。 |
| `Set Private Conversation Data` | チャネルの特定の会話のコンテキスト内で特定のユーザーの状態データを保存します。 |
| `Get User Data` | チャネルのすべての会話で以前に保存した特定のユーザーの状態データを取得します。 |
| `Get Conversation Data` | チャネルの特定の会話で以前に保存した状態データを取得します。 |
| `Get Private Conversation Data` | チャネルの特定の会話のコンテキスト内で以前に保存した特定のユーザーの状態データを取得します。 |
| `Delete State For User` | ユーザーのために以前に格納された状態データを削除します。 |

## <a name="schema"></a>スキーマ

Bot Framework スキーマには、ボットがユーザーとの通信に使用できるオブジェクトとそれらのプロパティが定義されています。

| Object | [説明] |
| ---- | ---- |
| [Activity オブジェクト](#activity-object) | ボットとユーザーの間で交換されるメッセージを定義します。 |
| [AnimationCard オブジェクト](#animationcard-object) | アニメーション GIF または短いビデオを再生できるカードを定義します。 |
| [Attachment オブジェクト](#attachment-object) | メッセージに含める追加の情報を定義します。 メディア ファイル (例: オーディオ、ビデオ、画像、ファイル) またはリッチ カードを添付ファイルにすることができます。 |
| [AttachmentData オブジェクト](#attachmentdata-object) | 添付ファイルのデータについて説明します。 |
| [AttachmentInfo オブジェクト](#attachmentinfo-object) | 添付ファイルについて説明します。 |
| [AttachmentView オブジェクト](#attachmentview-object) | 添付ファイルのビューを定義します。 |
| [AudioCard オブジェクト](#audiocard-object) | オーディオ ファイルを再生できるカードを定義します。 |
| [CardAction オブジェクト](#cardaction-object) | 実行するアクションを定義します。 |
| [CardImage オブジェクト](#cardimage-object) | カードに表示する画像を定義します。 |
| [ChannelAccount オブジェクト](#channelaccount-object) | チャネルのボットまたはユーザー アカウントを定義します。 |
| [ConversationAccount オブジェクト](#conversationaccount-object) | チャネルでの会話を定義します。 |
| [ConversationMembers オブジェクト](#conversationmembers-object) | 会話のメンバーを取得します。 |
| [ConversationParameters オブジェクト](#conversationparameters-object) | 新しい会話を作成するためのパラメーターを定義します |
| [ConversationReference オブジェクト](#conversationreference-object) | 会話内の特定のポイントを定義します。 |
| [ConversationResourceResponse オブジェクト](#conversationresourceresponse-object) | [会話を作成する](#create-conversation)への応答を定義します。 |
| [ConversationsResult オブジェクト](#conversationsresult-object) | [会話を取得する](#get-conversations)の呼び出し結果を定義します。 |
| [Entity オブジェクト](#entity-object) | エンティティ オブジェクトを定義します。 |
| [Error オブジェクト](#error-object) | エラーを定義します。 |
| [ErrorResponse オブジェクト](#errorresponse-object) | HTTP API 応答を定義します。 |
| [Fact オブジェクト](#fact-object) | ファクトを含むキーと値のペアを定義します。 |
| [GeoCoordinates オブジェクト](#geocoordinates-object) | World Geodetic System (WSG84) 座標を使用して地理的な場所を定義します。 |
| [HeroCard オブジェクト](#herocard-object) | 大きい画像、タイトル、テキスト、アクションの各ボタンを持つカードを定義します。 |
| [InnerHttpError オブジェクト](#innerhttperror-object) | 内部 HTTP エラーを示すオブジェクト。 |
| [MediaEventValue オブジェクト](#mediaeventvalue-object) | メディア イベントの補助パラメーター。 |
| [MediaUrl オブジェクト](#mediaurl-object) | メディア ファイルのソース URL を定義します。 |
| [Mention オブジェクト](#mention-object) | 会話でメンションされたユーザーまたはボットを定義します。 |
| [MessageReaction オブジェクト](#messagereaction-object) | メッセージへの反応を定義します。 |
| [PagedMembersResult オブジェクト](#pagedmembersresult-object) | [会話のページングされたメンバーを取得する](#get-conversation-paged-members)によって返されるメンバーのページ。 |
| [Place オブジェクト](#place-object) | 会話でメンションされた場所を定義します。 |
| [ReceiptCard オブジェクト](#receiptcard-object) | 購入のレシートを含むカードを定義します。 |
| [ReceiptItem オブジェクト](#receiptitem-object) | レシート内の品目を定義します。 |
| [ResourceResponse オブジェクト](#resourceresponse-object) | リソースを定義します。 |
| [SemanticAction オブジェクト](#semanticaction-object) | プログラムによるアクションへの参照を定義します。 |
| [SignInCard オブジェクト](#signincard-object) | ユーザーがサービスにサインインできるようにするカードを定義します。 |
| [SuggestedActions オブジェクト](#suggestedactions-object) | ユーザーが選択できるオプションを定義します。 |
| [TextHighlight オブジェクト](#texthighlight-object) | 別のフィールド内のコンテンツの部分文字列を参照します。 |
| [ThumbnailCard オブジェクト](#thumbnailcard-object) | サムネイル画像、タイトル、テキスト、アクションの各ボタンを持つカードを定義します。 |
| [ThumbnailUrl オブジェクト](#thumbnailurl-object) | 画像のソース URL を定義します。 |
| [Transcript オブジェクト](#transcript-object) | [会話履歴を送信する](#send-conversation-history)を使用してアップロードされるアクティビティのコレクション。 |
| [VideoCard オブジェクト](#videocard-object) | ビデオを再生できるカードを定義します。 |

### <a name="activity-object"></a>Activity オブジェクト

ボットとユーザーの間で交換されるメッセージを定義します。

| プロパティ | 種類 | [説明] |
|----|----|----|
| **action** | string | 適用する、または適用されたアクション。 アクションのコンテキストを決定するには、**type** プロパティを使用します。 たとえば、**type** が **contactRelationUpdate** の場合、**action** プロパティの値は、ユーザーがボットを連絡先リストに追加した場合は **add** になり、ユーザーがボットを連絡先リストから削除した場合は **remove** になります。 |
| **attachmentLayout** | string | メッセージに含まれるリッチ カード**添付ファイル**のレイアウト。 次の値のいずれか: **carousel**、**list**。 リッチ カード添付ファイルの詳細については、「[メッセージへのリッチ カード添付ファイルの追加](bot-framework-rest-connector-add-rich-cards.md)」を参照してください。 |
| **attachments** | [Attachment](#attachment-object)[] | メッセージに含める追加の情報を定義する **Attachment** オブジェクトの配列。 各添付ファイルは、ファイル (例: オーディオ、ビデオ、画像) またはリッチ カードのどちらかにすることができます。 |
| **callerId** | string | ボットの呼び出し元を識別する IRI を含む文字列。 このフィールドは、ネットワーク経由で送信されるものではありません。その代わりに、呼び出し元 (トークンなど) の ID をアサートする暗号的に検証可能なデータに基づいて、ボットとクライアントによって設定されます。 |
| **channelData** | object | チャネル固有のコンテンツを格納するオブジェクト。 一部のチャネルには、添付ファイル スキーマでは表現できない追加情報を必要とする機能があります。 そのような場合は、このプロパティを、チャネルのドキュメントで定義されているチャネル固有のコンテンツに設定します。 詳細については、「[チャネル固有の機能の実装](bot-framework-rest-connector-channeldata.md)」を参照してください。 |
| **channelId** | string | チャネルを一意に識別する ID。 チャネルによって設定されます。 |
| **code** | string | 会話が終了した理由を示すコード。 |
| **conversation** | [ConversationAccount](#conversationaccount-object) | アクティビティが属する会話を定義する **ConversationAccount** オブジェクト。 |
| **deliveryMode** | string | アクティビティの受信者の代替配信パスに通知する配信ヒント。 値は、**normal**、**notification** のいずれかです。 |
| **entities** | object[] | メッセージでメンションされたエンティティを表すオブジェクトの配列。 この配列内のオブジェクトは任意の [Schema.org](http://schema.org/) オブジェクトです。 たとえば、会話でメンションされた人物を識別する [Mention](#mention-object) オブジェクトや、会話でメンションされた場所を識別する [Place](#place-object) オブジェクトを配列に含めることができます。 |
| **expiration** | string | アクティビティを "期限切れ" と見なす時刻です。これは受信者に表示すべきではありません。 |
| **from** | [ChannelAccount](#channelaccount-object) | メッセージの送信者を指定する **ChannelAccount** オブジェクト。 |
| **historyDisclosed** | boolean | 履歴が公開されているかどうかを示すフラグ。 既定値は **false** です。 |
| **id** | string | チャネルでのアクティビティを一意に識別する ID。 |
| **importance** | string | アクティビティの重要度を定義します。 次の値のいずれか: **low**、**normal**、**high**。 |
| **inputHint** | string | メッセージがクライアントに配信された後、ボットがユーザー入力を受け付けるか、期待するか、または無視するかを示す値。 次の値のいずれか: **acceptingInput**、**expectingInput**、**ignoringInput**。 |
| **label** | string | アクティビティの説明ラベル。 |
| **listenFor** | string[] | 音声および言語の認識システムで聞き取る必要がある語句と参照の一覧。 |
| **locale** | string | メッセージ内のテキストの表示に使用する言語のロケールで、`<language>-<country>` の形式。 ボットがその言語の表示文字列を指定できるよう、チャネルはこのプロパティを使用してユーザーの言語を指示します。 既定値は **en-US** です。 |
| **localTimestamp** | string | ローカル タイム ゾーンでメッセージが送信された日時を [ISO-8601](https://en.wikipedia.org/wiki/ISO_8601) 形式で表したもの。 |
| **localTimezone** | string | IANA タイム ゾーン データベース形式で表される、メッセージのローカル タイムゾーンの名前が格納されます。 例: America/Los_Angeles。 |
| **membersAdded** | [ChannelAccount](#channelaccount-object)[] | 会話に参加したユーザーのリストを表す **ChannelAccount** オブジェクトの配列。 アクティビティの **type** が "conversationUpdate" で、ユーザーが会話に参加した場合にのみ存在します。 |
| **membersRemoved** | [ChannelAccount](#channelaccount-object)[] | 会話から退出したユーザーのリストを表す **ChannelAccount** オブジェクトの配列。 アクティビティの **type** が "conversationUpdate" で、ユーザーが会話から退出した場合にのみ存在します。 |
| **name** | string | 呼び出す操作の名前またはイベントの名前。 |
| **reactionsAdded** | [MessageReaction](#messagereaction-object)[] | 会話に追加された反応のコレクション。 |
| **reactionsRemoved** | [MessageReaction](#messagereaction-object)[] | 会話から削除された反応のコレクション。 |
| **recipient** | [ChannelAccount](#channelaccount-object) | メッセージの受信者を指定する **ChannelAccount** オブジェクト。 |
| **relatesTo** | [ConversationReference](#conversationreference-object) | 会話内の特定のポイントを定義する **ConversationReference** オブジェクト。 |
| **replyToId** | string | このメッセージの返信先メッセージの ID。 ユーザーが送信したメッセージに返信するには、このプロパティをユーザーのメッセージの ID に設定します。 すべてのチャネルがスレッド返信をサポートするわけではありません。 このような場合、チャネルはこのプロパティを無視し、時間順のセマンティクス (タイムスタンプ) を使用してメッセージを会話に追加します。 |
| **semanticAction** |[SemanticAction](#semanticaction-object) | プログラムによるアクションへの参照を表す **SemanticAction** オブジェクト。 |
| **serviceUrl** | string | チャネルのサービス エンドポイントを指定する URL。 チャネルによって設定されます。 |
| **speak** | string | 音声対応チャネルでボットが話すテキスト。 音声、速度、音量、発音、ピッチなど、ボットの音声のさまざまな特性を制御するには、[音声合成マークアップ言語 (SSML)](https://msdn.microsoft.com/library/hh378377(v=office.14).aspx) 形式でこのプロパティを指定します。 |
| **suggestedActions** | [SuggestedActions](#suggestedactions-object) | ユーザーが選択できるオプションを定義する **SuggestedActions** オブジェクト。 |
| **summary** | string | メッセージに含まれる情報の要約。 たとえば、電子メール チャネルで送信されるメッセージの場合、このプロパティで電子メール メッセージの最初の 50 文字を指定できます。 |
| **text** | string | ユーザーからボットに、またはボットからユーザーに送信されるメッセージのテキスト。 このプロパティの内容に課される制限については、チャネルのドキュメントを参照してください。 |
| **textFormat** | string | メッセージの **text** の形式。 次の値のいずれか: **markdown**、**plain**、**xml**。 テキスト形式の詳細については、「[Create messages](bot-framework-rest-connector-create-messages.md)」(メッセージの作成) を参照してください。 |
| **textHighlights** | [TextHighlight](#texthighlight-object)[] | アクティビティに **replyToId** 値が含まれている場合に、強調表示するテキスト フラグメントのコレクション。 |
| **timestamp** | string | UTC タイム ゾーンでメッセージが送信された日時を [ISO-8601](https://en.wikipedia.org/wiki/ISO_8601) 形式で表したもの。 |
| **topicName** | string | アクティビティが属する会話のトピック。 |
| **type** | string | アクティビティの種類。 次のいずれかの値: **message**、**contactRelationUpdate**、**conversationUpdate**、**typing**、**endOfConversation**、**event**、**invoke**、**deleteUserData**、**messageUpdate**、**messageDelete**、**installationUpdate**、**messageReaction**、**suggestion**、**trace**、**handoff**。 アクティビティの種類の詳細については、「[Activities overview](https://aka.ms/botSpecs-activitySchema)」(アクティビティの概要) を参照してください。 |
| **value** | object | 拡張可能な値。 |
| **valueType** | string | アクティビティの値オブジェクトの型。 |

[スキーマの表に戻る](#schema)

### <a name="animationcard-object"></a>AnimationCard オブジェクト

アニメーション GIF または短いビデオを再生できるカードを定義します。

| プロパティ | 種類 | [説明] |
|----|----|----|
| **aspect** | boolean | サムネイル/メディア プレースホルダーの縦横比。 使用できる値は "16:9" と "4:3" です。 |
| **autoloop** | boolean | 最後の項目が終了したときにアニメーション GIF のリストをリプレイするかどうかを示すフラグ。 アニメーションを自動的にリプレイするには、このプロパティを **true** に設定します。それ以外の場合、**false** に設定します。 既定値は **true** です。 |
| **autostart** | boolean | カードが表示されたときにアニメーションを自動的に再生するかどうかを示すフラグ。 アニメーションを自動的に再生するには、このプロパティを **true** に設定します。それ以外の場合、**false** に設定します。 既定値は **true** です。 |
| **buttons** | [CardAction](#cardaction-object)[] | ユーザーが 1 つ以上のアクションを実行できるようにする **CardAction** オブジェクトの配列。 指定できるボタンの数はチャネルが決定します。 |
| **duration** | string | メディア コンテンツの長さ ([ISO 8601 の期間の形式](https://www.iso.org/iso-8601-date-and-time-format.html))。 |
| **image** | [ThumbnailUrl](#thumbnailurl-object) | カードに表示する画像を指定する **ThumbnailUrl** オブジェクト。 |
| **media** | [MediaUrl](#mediaurl-object)[] | **Mediaurl** オブジェクトの配列。 このフィールドに複数の URL が含まれている場合、各 URL は同じコンテンツの代替形式です。|
| **shareable** | boolean | アニメーションを他のユーザーと共有できるかどうかを示すフラグ。 アニメーションを共有できる場合、このプロパティを **true** に設定します。それ以外の場合、**false** に設定します。 既定値は **true** です。 |
| **subtitle** | string | カードのタイトルの下に表示するサブタイトル。 |
| **text** | string | カードのタイトルまたはサブタイトルの下に表示する説明またはプロンプト。 |
| **title** | string | カードのタイトル。 |
| **value** | object | このカードの補助パラメーター。 |

[スキーマの表に戻る](#schema)

### <a name="attachment-object"></a>Attachment オブジェクト

メッセージに含める追加の情報を定義します。 ファイル (画像、オーディオ、ビデオなど) またはリッチ カードを添付ファイルにすることができます。

| プロパティ | 種類 | [説明] |
|----|----|----|
| **content** | object | 添付ファイルのコンテンツ。 添付ファイルがリッチ カードの場合、このプロパティをリッチ カード オブジェクトに設定します。 このプロパティと **contentUrl** プロパティは相互に排他的です。 |
| **contentType** | string | 添付ファイル内のコンテンツのメディアの種類。 メディア ファイルの場合、このプロパティを **image/png**、**audio/wav**、**video/mp4** などの既知のメディア タイプに設定します。 リッチ カードの場合、このプロパティを次のベンダー固有タイプのいずれかに設定します。<ul><li>**application/vnd.microsoft.card.adaptive**: テキスト、音声、画像、ボタン、入力フィールドの任意の組み合わせを含めることができるリッチ カード。 **content** プロパティを [AdaptiveCard](https://adaptivecards.io/explorer/AdaptiveCard.html) オブジェクトに設定します。</li><li>**application/vnd.microsoft.card.animation**: アニメーションを再生するリッチ カード。 **content** プロパティを [AnimationCard](#animationcard-object) オブジェクトに設定します。</li><li>**application/vnd.microsoft.card.audio**: オーディオ ファイルを再生するリッチ カード。 **content** プロパティを [AudioCard](#audiocard-object) オブジェクトに設定します。</li><li>**application/vnd.microsoft.card.hero**: ヒーロー カード。 **content** プロパティを [HeroCard](#herocard-object) オブジェクトに設定します。</li><li>**application/vnd.microsoft.card.receipt**:領収書カード。 **content** プロパティを [ReceiptCard](#receiptcard-object) オブジェクトに設定します。</li><li>**application/vnd.microsoft.card.signin**:ユーザー サインイン カード。 **content** プロパティを [SignInCard](#signincard-object) オブジェクトに設定します。</li><li>**application/vnd.microsoft.card.thumbnail**: サムネイル カード。 **content** プロパティを [ThumbnailCard](#thumbnailcard-object) オブジェクトに設定します。</li><li>**application/vnd.microsoft.card.video**: ビデオを再生するリッチ カード。 **content** プロパティを [VideoCard](#videocard-object) オブジェクトに設定します。</li></ul> |
| **contentUrl** | string | 添付ファイルのコンテンツの URL。 たとえば、添付ファイルが画像の場合、**contentUrl** を画像の場所を表す URL に設定できます。 サポートされているプロトコルは、HTTP、HTTPS、File、Data です。 |
| **name** | string | 添付ファイルの名前。 |
| **thumbnailUrl** | string | より小さな形式の代替 **content** または **contentUrl** の使用をチャネルがサポートしている場合にチャネルが使用できるサムネイル画像の URL。 たとえば、**contentType** を **application/word** に設定し、**contentUrl** を Word 文書の場所に設定する場合、文書を表すサムネイル画像を含めることができます。 チャネルは文書の代わりにサムネイル画像を表示できます。 ユーザーが画像をクリックすると、チャネルは文書を開きます。 |

[スキーマの表に戻る](#schema)

### <a name="attachmentdata-object"></a>AttachmentData オブジェクト

添付ファイルのデータについて説明します。

| プロパティ | 種類 | [説明] |
|----|----|----|
| **name** | string | 添付ファイルの名前。 |
| **originalBase64** | string | 添付ファイルのコンテンツ。 |
| **thumbnailBase64** | string | 添付ファイルのサムネイル コンテンツ。 |
| **type** | string | 添付ファイルのコンテンツの種類。 |

[スキーマの表に戻る](#schema)

### <a name="attachmentinfo-object"></a>AttachmentInfo オブジェクト

添付ファイルのメタデータ。

| プロパティ | 種類 | [説明] |
|----|----|----|
| **name** | string | 添付ファイルの名前。 |
| **type** | string | 添付ファイルのコンテンツの種類。 |
| **views** | [AttachmentView](#attachmentview-object)[] | 添付ファイルの利用可能なビューを表す **AttachmentView** オブジェクトの配列。 |

[スキーマの表に戻る](#schema)

### <a name="attachmentview-object"></a>AttachmentView オブジェクト

添付ファイルのビューを定義します。

| プロパティ | 種類 | [説明] |
|----|----|----|
| **size** | number | ファイルのサイズ。 |
| **viewId** | string | ビュー ID。 |

[スキーマの表に戻る](#schema)

### <a name="audiocard-object"></a>AudioCard オブジェクト

オーディオ ファイルを再生できるカードを定義します。

| プロパティ | 種類 | [説明] |
|----|----|----|
| **aspect** | string | **image** プロパティで指定されたサムネイルの縦横比。 有効な値は **16:9** と **4:3** です。 |
| **autoloop** | boolean | 最後の項目が終了したときにオーディオ ファイルのリストをリプレイするかどうかを示すフラグ。 オーディオ ファイルを自動的にリプレイするには、このプロパティを **true** に設定します。それ以外の場合、**false** に設定します。 既定値は **true** です。 |
| **autostart** | boolean | カードが表示されたときにオーディオを自動的に再生するかどうかを示すフラグ。 オーディオを自動的に再生するには、このプロパティを **true** に設定します。それ以外の場合、**false** に設定します。 既定値は **true** です。 |
| **buttons** | [CardAction](#cardaction-object)[] | ユーザーが 1 つ以上のアクションを実行できるようにする **CardAction** オブジェクトの配列。 指定できるボタンの数はチャネルが決定します。 |
| **duration** | string | メディア コンテンツの長さ ([ISO 8601 の期間の形式](https://www.iso.org/iso-8601-date-and-time-format.html))。 |
| **image** | [ThumbnailUrl](#thumbnailurl-object) | カードに表示する画像を指定する **ThumbnailUrl** オブジェクト。 |
| **media** | [MediaUrl](#mediaurl-object)[] | **Mediaurl** オブジェクトの配列。  このフィールドに複数の URL が含まれている場合、各 URL は同じコンテンツの代替形式です。 |
| **shareable** | boolean | オーディオ ファイルを他のユーザーと共有できるかどうかを示すフラグ。 オーディオを共有できる場合、このプロパティを **true** に設定します。それ以外の場合、**false** に設定します。 既定値は **true** です。 |
| **subtitle** | string | カードのタイトルの下に表示するサブタイトル。 |
| **text** | string | カードのタイトルまたはサブタイトルの下に表示する説明またはプロンプト。 |
| **title** | string | カードのタイトル。 |
| **value** | object | このカードの補助パラメーター。 |

[スキーマの表に戻る](#schema)

### <a name="cardaction-object"></a>CardAction オブジェクト

ボタンを使用してクリック可能なアクションを定義します。

| プロパティ | 種類 | [説明] |
|----|----|----|
| **channelData** | string | このアクションに関連付けられているチャネル固有のデータ。 |
| **displayText** | string | ボタンがクリックされた場合にチャット フィードに表示するテキスト。 |
| **image** | string | テキスト ラベルの横にあるボタンに表示される画像の URL。 |
| **text** | string | アクションのテキスト。 |
| **title** | string | ボタンに表示されるテキスト説明です。 |
| **type** | string | 実行するアクションの種類。 有効な値の一覧については、「[メッセージへのリッチ カード添付ファイルの追加](bot-framework-rest-connector-add-rich-cards.md)」を参照してください。 |
| **value** | object | アクションの補助パラメーター。 このプロパティの動作は、アクションの **type** によって異なります。 詳細については、「[メッセージへのリッチ カード添付ファイルの追加](bot-framework-rest-connector-add-rich-cards.md)」を参照してください。 |

[スキーマの表に戻る](#schema)

### <a name="cardimage-object"></a>CardImage オブジェクト

カードに表示する画像を定義します。

| プロパティ | 種類 | [説明] |
|----|----|----|
| **alt** | string | 画像の説明。 アクセシビリティをサポートするには、説明を含める必要があります。 |
| **tap** | [CardAction](#cardaction-object) | ユーザーが画像をタップまたはクリックした場合に実行するアクションを指定する **CardAction** オブジェクト。 |
| **url** | string | 画像または画像の base64 バイナリのソース URL (例: `data:image/png;base64,iVBORw0KGgo...`)。 |

[スキーマの表に戻る](#schema)

### <a name="channelaccount-object"></a>ChannelAccount オブジェクト

チャネルのボットまたはユーザー アカウントを定義します。

| プロパティ | 種類 | [説明] |
|----|----|----|
| **aadObjectId** | string | Azure Active Directory 内のこのアカウントのオブジェクト ID。 |
| **id** | string | このチャネルのユーザーまたはボットの一意の ID。 |
| **name** | string | ボットまたはユーザーの表示用の名前。 |
| **role** | string | このアカウントの背後にあるエンティティのロール。 **user** または **bot** のいずれかです。 |

[スキーマの表に戻る](#schema)

### <a name="conversationaccount-object"></a>ConversationAccount オブジェクト

チャネルでの会話を定義します。

| プロパティ | 種類 | [説明] |
|----|----|----|
| **aadObjectId** | string | Azure Active Directory (AAD) 内のこのアカウントのオブジェクト ID。 |
| **conversationType** | string | 会話の種類 (グループ、個人など) を区別するチャネルでの会話の種類を示します。 |
| **id** | string | 会話を識別する ID。 ID はチャネルごとに一意です。 チャネルが会話を開始する場合、チャネルがこの ID を設定します。それ以外の場合、このプロパティはボットによって、ボットが会話を開始するときに応答で取得する ID に設定されます (「[会話を作成する](#create-conversation)」を参照)。 |
| **isGroup** | boolean | アクティビティが生成された時点で会話に 2 人より多い参加者が含まれているかどうかを示すフラグ。 これがグループ会話の場合は **true** に設定します。それ以外の場合、**false** に設定します。 既定値は **false** です。 |
| **name** | string | 会話を識別するために使用できる表示名。 |
| **role** | string | このアカウントの背後にあるエンティティのロール。 **user** または **bot** のいずれかです。 |
| **tenantId** | string | この会話のテナント ID。 |

[スキーマの表に戻る](#schema)

### <a name="conversationmembers-object"></a>ConversationMembers オブジェクト

会話のメンバーを取得します。

| プロパティ | 種類 | [説明] |
|----|----|----|
| **id** | string | 会話 ID。 |
| **members** | [ChannelAccount](#channelaccount-object)[] | この会話のメンバーの一覧。 |

[スキーマの表に戻る](#schema)

### <a name="conversationparameters-object"></a>ConversationParameters オブジェクト

新しい会話を作成するためのパラメーターを定義します。

| プロパティ | 種類 | [説明] |
|----|----|----|
| **activity** | [アクティビティ](#activity-object) | 会話の作成時にその会話に送信される初期メッセージ。 |
| **bot** | [ChannelAccount](#channelaccount-object) | ボットにメッセージをルーティングするために必要なチャネル アカウント情報。 |
| **channelData** | object | 会話を作成するためのチャネル固有のペイロード。 |
| **isGroup** | boolean | これがグループ会話かどうかを示します。 |
| **members** | [ChannelAccount](#channelaccount-object)[] | 各ユーザーにメッセージをルーティングするために必要なチャネル アカウント情報。 |
| **tenantId** | string | 会話が作成されるテナントの ID。 |
| **topicName** | string | 会話のトピック。 このプロパティは、チャネルがサポートしている場合にのみ使用されます。 |

[スキーマの表に戻る](#schema)

### <a name="conversationreference-object"></a>ConversationReference オブジェクト

会話内の特定のポイントを定義します。

| プロパティ | 種類 | [説明] |
|----|----|----|
| **activityId** | string | このオブジェクトが参照するアクティビティを一意に識別する ID。 |
| **bot** | [ChannelAccount](#channelaccount-object) | このオブジェクトが参照する会話でボットを識別する **ChannelAccount** オブジェクト。 |
| **channelId** | string | このオブジェクトが参照する会話でチャネルを一意に識別する ID。 |
| **conversation** | [ConversationAccount](#conversationaccount-object) | このオブジェクトが参照する会話を定義する **ConversationAccount** オブジェクト。 |
| **serviceUrl** | string | このオブジェクトが参照する会話でチャネルのサービス エンドポイントを指定する URL。 |
| **user** | [ChannelAccount](#channelaccount-object) | このオブジェクトが参照する会話でユーザーを識別する **ChannelAccount** オブジェクト。 |

[スキーマの表に戻る](#schema)

### <a name="conversationresourceresponse-object"></a>ConversationResourceResponse オブジェクト

[会話を作成する](#create-conversation)への応答を定義します。

| プロパティ | 種類 | [説明] |
|----|----|----|
| **activityId** | string | アクティビティの ID (送信された場合)。 |
| **id** | string | リソースの ID。 |
| **serviceUrl** | string | 会話に関する操作を実行できるサービス エンドポイント。 |

[スキーマの表に戻る](#schema)

### <a name="conversationsresult-object"></a>ConversationsResult オブジェクト

[会話を取得する](#get-conversations)の結果を定義します。

| プロパティ | 種類 | [説明] |
|----|----|----|
| **conversations** | [ConversationMembers](#conversationmembers-object)[] | 各会話のメンバー。 |
| **continuationToken** | string | その後の[会話を取得する](#get-conversations)の呼び出しで使用できる継続トークン。 |

[スキーマの表に戻る](#schema)

### <a name="entity-object"></a>Entity オブジェクト

アクティビティに関連するメタデータ オブジェクト。

| プロパティ | 種類 | [説明] |
|----|----|----|
| **type** | string | このエンティティの型 (RFC 3987 IRI)。 |

[スキーマの表に戻る](#schema)

### <a name="error-object"></a>エラー オブジェクト

エラー情報を示すオブジェクト。

| プロパティ | 種類 | [説明] |
|----|----|----|
| **code** | string | エラー コード。 |
| **innerHttpError** | [InnerHttpError](#innerhttperror-object) | 内部 HTTP エラーを示すオブジェクト。 |
| **message** | string | エラーの説明。 |

[スキーマの表に戻る](#schema)

### <a name="errorresponse-object"></a>ErrorResponse オブジェクト

HTTP API 応答を定義します。

| プロパティ | 種類 | [説明] |
|----|----|----|
| **error** | [Error](#error-object) | エラーに関する情報を含む **Error** オブジェクト。 |

[スキーマの表に戻る](#schema)

### <a name="fact-object"></a>Fact オブジェクト

ファクトを含むキーと値のペアを定義します。

| プロパティ | 種類 | [説明] |
|----|----|----|
| **key** | string | ファクトの名前。 例: **Check-in**。 キーは、ファクトの値を表示するときにラベルとして使用されます。 |
| **value** | string | ファクトの値。 例: **10 October 2016**。 |

[スキーマの表に戻る](#schema)

### <a name="geocoordinates-object"></a>GeoCoordinates オブジェクト

World Geodetic System (WSG84) 座標を使用して地理的な場所を定義します。

| プロパティ | 種類 | [説明] |
|----|----|----|
| **elevation** | number | 場所の標高。 |
| **latitude** | number | 場所の緯度。 |
| **longitude** | number | 場所の経度。 |
| **name** | string | 場所の名前。 |
| **type** | string | このオブジェクトの種類。 常に **GeoCoordinates** に設定します。 |

[スキーマの表に戻る](#schema)

### <a name="herocard-object"></a>HeroCard オブジェクト

大きい画像、タイトル、テキスト、アクションの各ボタンを持つカードを定義します。

| プロパティ | 種類 | [説明] |
|----|----|----|
| **buttons** | [CardAction](#cardaction-object)[] | ユーザーが 1 つ以上のアクションを実行できるようにする **CardAction** オブジェクトの配列。 指定できるボタンの数はチャネルが決定します。 |
| **images** | [CardImage](#cardimage-object)[] | カードに表示する画像を指定する **CardImage** オブジェクトの配列。 ヒーロー カードには 1 つの画像のみが含まれます。 |
| **subtitle** | string | カードのタイトルの下に表示するサブタイトル。 |
| **tap** | [CardAction](#cardaction-object) | ユーザーがカードをタップまたはクリックした場合に実行するアクションを指定する **CardAction** オブジェクト。 いずれかのボタンと同じアクションまたは別のアクションを指定できます。 |
| **text** | string | カードのタイトルまたはサブタイトルの下に表示する説明またはプロンプト。 |
| **title** | string | カードのタイトル。 |

[スキーマの表に戻る](#schema)

### <a name="innerhttperror-object"></a>InnerHttpError オブジェクト

内部 HTTP エラーを示すオブジェクト。

| プロパティ | 種類 | [説明] |
|----|----|----|
| **statusCode** | number | 失敗した要求の HTTP 状態コード。 |
| **body** | object | 失敗した要求の本文。 |

[スキーマの表に戻る](#schema)

### <a name="mediaeventvalue-object"></a>MediaEventValue オブジェクト

メディア イベントの補助パラメーター。

| プロパティ | 種類 | [説明] |
|----|----|----|
| **cardValue** | object | このイベントを発生させたメディア カードの **value** フィールドに指定されたコールバック パラメーター。 |

[スキーマの表に戻る](#schema)

### <a name="mediaurl-object"></a>MediaUrl オブジェクト

メディア ファイルのソース URL を定義します。

| プロパティ | 種類 | [説明] |
|----|----|----|
| **profile** | string | メディアのコンテンツを説明するヒント。 |
| **url** | string | メディア ファイルのソース URL。 |

[スキーマの表に戻る](#schema)

### <a name="mention-object"></a>Mention オブジェクト

会話でメンションされたユーザーまたはボットを定義します。

| プロパティ | 種類 | [説明] |
|----|----|----|
| **mentioned** | [ChannelAccount](#channelaccount-object) | メンションされたユーザーまたはボットを指定する **ChannelAccount** オブジェクト。 Slack などの一部のチャネルでは会話ごとに名前を割り当てるため、(メッセージの **recipient** プロパティで) メンションされたボットの名前は、ボットの[登録](../bot-service-quickstart-registration.md)時に指定したハンドルとは異なる可能性があることに注意してください。 ただし、両方のアカウント ID は同じになります。 |
| **text** | string | 会話でメンションされたユーザーまたはボット。 たとえば、メッセージが「@ColorBot pick me a new color」の場合、このプロパティは **\@ColorBot** に設定されます。 すべてのチャネルがこのプロパティを設定するわけではありません。 |
| **type** | string | このオブジェクトの種類。 常に **Mention** に設定します。 |

[スキーマの表に戻る](#schema)

### <a name="messagereaction-object"></a>MessageReaction オブジェクト

メッセージへの反応を定義します。

| プロパティ | 種類 | [説明] |
|----|----|----|
| **type** | string | 反応の種類。 **like** または **plusOne** のいずれかです。 |

[スキーマの表に戻る](#schema)

### <a name="pagedmembersresult-object"></a>PagedMembersResult オブジェクト

[会話のページングされたメンバーを取得する](#get-conversation-paged-members)によって返されるメンバーのページ。

| プロパティ | 種類 | [説明] |
|----|----|----|
| **continuationToken** | string | 後続の[会話のページングされたメンバーを取得する](#get-conversation-paged-members)の呼び出しで使用できる継続トークン。 |
| **members** | [ChannelAccount](#channelaccount-object)[] | 会話のメンバーの配列です。 |

[スキーマの表に戻る](#schema)

### <a name="place-object"></a>Place オブジェクト

会話でメンションされた場所を定義します。

| プロパティ | 種類 | [説明] |
|----|----|----|
| **address** | object |  場所の所在地。 このプロパティは **string**、または **PostalAddress** 型の複合オブジェクトです。 |
| **geo** | [GeoCoordinates](#geocoordinates-object) | 場所の地理的座標を指定する **GeoCoordinates** オブジェクト。 |
| **hasMap** | object | 場所の地図。 このプロパティは **string** (URL)、または **Map** 型の複合オブジェクトです。 |
| **name** | string | 場所の名前。 |
| **type** | string | このオブジェクトの種類。 常に **Place** に設定します。 |

[スキーマの表に戻る](#schema)

### <a name="receiptcard-object"></a>ReceiptCard オブジェクト

購入のレシートを含むカードを定義します。

| プロパティ | 種類 | [説明] |
|----|----|----|
| **buttons** | [CardAction](#cardaction-object)[] | ユーザーが 1 つ以上のアクションを実行できるようにする **CardAction** オブジェクトの配列。 指定できるボタンの数はチャネルが決定します。 |
| **facts** | [Fact](#fact-object)[] | 購入に関する情報を指定する **Fact** オブジェクトの配列。 たとえば、ホテル宿泊レシートのファクトのリストには、チェックイン日とチェックアウト日が含まれる場合があります。 指定できるファクトの数はチャネルが決定します。 |
| **items** | [ReceiptItem](#receiptitem-object)[] | 購入したアイテムを指定する **ReceiptItem** オブジェクトの配列 |
| **tap** | [CardAction](#cardaction-object) | ユーザーがカードをタップまたはクリックした場合に実行するアクションを指定する **CardAction** オブジェクト。 いずれかのボタンと同じアクションまたは別のアクションを指定できます。 |
| **tax** | string | 購入に適用される税額を指定する通貨形式の文字列。 |
| **title** | string | レシートの上部に表示されるタイトル。 |
| **total** | string | 適用されるすべての税金を含めた合計購入価格を指定する通貨形式の文字列。 |
| **vat** | string | 購入価格に適用される付加価値税 (VAT) の額を指定する通貨形式の文字列。 |

[スキーマの表に戻る](#schema)

### <a name="receiptitem-object"></a>ReceiptItem オブジェクト

レシート内の品目を定義します。

| プロパティ | 種類 | [説明] |
|----|----|----|
| **image** | [CardImage](#cardimage-object) | 品目の隣に表示するサムネイル画像を指定する **CardImage** オブジェクト。  |
| **price** | string | すべての購入単位の合計価格を指定する通貨形式の文字列。 |
| **quantity** | string | 購入単位数を指定する数値の文字列。 |
| **subtitle** | string | 品目のタイトルの下に表示されるサブタイトル。 |
| **tap** | [CardAction](#cardaction-object) | ユーザーが品目をタップまたはクリックした場合に実行するアクションを指定する **CardAction** オブジェクト。 |
| **text** | string | 品目の説明。 |
| **title** | string | 品目のタイトル。 |

[スキーマの表に戻る](#schema)

### <a name="resourceresponse-object"></a>ResourceResponse オブジェクト

リソース ID を含む応答を定義します。

| プロパティ | 種類 | [説明] |
|----|----|----|
| **id** | string | リソースを一意に識別する ID。 |

[スキーマの表に戻る](#schema)

### <a name="semanticaction-object"></a>SemanticAction オブジェクト

プログラムによるアクションへの参照を定義します。

| プロパティ | 種類 | [説明] |
|----|----|----|
| **entities** | object | 各プロパティの値が [Entity](#entity-object) オブジェクトであるオブジェクト。 |
| **id** | string | このアクションの ID。 |
| **state** | string | このアクションの状態。 使用できる値: **start**、**continue**、**done**。 |

[スキーマの表に戻る](#schema)

### <a name="signincard-object"></a>SignInCard オブジェクト

ユーザーがサービスにサインインできるようにするカードを定義します。

| プロパティ | 種類 | [説明] |
|----|----|----|
| **buttons** | [CardAction](#cardaction-object)[] | ユーザーがサービスにサインインできるようにする **CardAction** オブジェクトの配列。 指定できるボタンの数はチャネルが決定します。 |
| **text** | string | サインイン カードに含める説明またはプロンプト。 |

[スキーマの表に戻る](#schema)

### <a name="suggestedactions-object"></a>SuggestedActions オブジェクト

ユーザーが選択できるオプションを定義します。

| プロパティ | 種類 | [説明] |
|----|----|----|
| **actions** | [CardAction](#cardaction-object)[] | 推奨されるアクションを定義する **CardAction** オブジェクトの配列。 |
| **to** | string[] | 推奨されるアクションの表示先である受信者の ID を含む文字列の配列。 |

[スキーマの表に戻る](#schema)

### <a name="texthighlight-object"></a>TextHighlight オブジェクト

別のフィールド内のコンテンツの部分文字列を参照します。

| プロパティ | 種類 | [説明] |
|----|----|----|
| **occurrence** | number | 参照されるテキスト内でのテキスト フィールドの出現数 (複数存在する場合)。 |
| **text** | string | 強調表示するテキストのスニペットを定義します。 |

[スキーマの表に戻る](#schema)

### <a name="thumbnailcard-object"></a>ThumbnailCard オブジェクト

サムネイル画像、タイトル、テキスト、アクションの各ボタンを持つカードを定義します。

| プロパティ | 種類 | [説明] |
|----|----|----|
| **buttons** | [CardAction](#cardaction-object)[] | ユーザーが 1 つ以上のアクションを実行できるようにする **CardAction** オブジェクトの配列。 指定できるボタンの数はチャネルが決定します。 |
| **images** | [CardImage](#cardimage-object)[] | カードに表示するサムネイル画像を指定する **CardImage** オブジェクトの配列。 指定できるサムネイル画像の数はチャネルが決定します。 |
| **subtitle** | string | カードのタイトルの下に表示するサブタイトル。 |
| **tap** | [CardAction](#cardaction-object) | ユーザーがカードをタップまたはクリックした場合に実行するアクションを指定する **CardAction** オブジェクト。 いずれかのボタンと同じアクションまたは別のアクションを指定できます。 |
| **text** | string | カードのタイトルまたはサブタイトルの下に表示する説明またはプロンプト。 |
| **title** | string | カードのタイトル。 |

[スキーマの表に戻る](#schema)

### <a name="thumbnailurl-object"></a>ThumbnailUrl オブジェクト

画像のソース URL を定義します。

| プロパティ | 種類 | [説明] |
|----|----|----|
| **alt** | string | 画像の説明。 アクセシビリティをサポートするには、説明を含める必要があります。 |
| **url** | string | 画像または画像の base64 バイナリのソース URL (例: `data:image/png;base64,iVBORw0KGgo...`)。 |

[スキーマの表に戻る](#schema)

### <a name="transcript-object"></a>Transcript オブジェクト

[会話履歴を送信する](#send-conversation-history)を使用してアップロードされるアクティビティのコレクション。

| プロパティ | 種類 | [説明] |
|----|----|----|
| **アクティビティ** | array | [Activity](#activity-object) オブジェクトの配列です。 それぞれが一意の ID とタイムスタンプを持つ必要があります。 |

[スキーマの表に戻る](#schema)

### <a name="videocard-object"></a>VideoCard オブジェクト

ビデオを再生できるカードを定義します。

| プロパティ | 種類 | [説明] |
|----|----|----|
| **aspect** | string | ビデオの縦横比。 **16:9** または **4:3** のいずれかです。 |
| **autoloop** | boolean | 最後の項目が終了したときにビデオのリストをリプレイするかどうかを示すフラグ。 ビデオを自動的にリプレイするには、このプロパティを **true** に設定します。それ以外の場合、**false** に設定します。 既定値は **true** です。 |
| **autostart** | boolean | カードが表示されたときにビデオを自動的に再生するかどうかを示すフラグ。 ビデオを自動的に再生するには、このプロパティを **true** に設定します。それ以外の場合、**false** に設定します。 既定値は **true** です。 |
| **buttons** | [CardAction](#cardaction-object)[] | ユーザーが 1 つ以上のアクションを実行できるようにする **CardAction** オブジェクトの配列。 指定できるボタンの数はチャネルが決定します。 |
| **duration** | string | メディア コンテンツの長さ ([ISO 8601 の期間の形式](https://www.iso.org/iso-8601-date-and-time-format.html))。 |
| **image** | [ThumbnailUrl](#thumbnailurl-object) | カードに表示する画像を指定する **ThumbnailUrl** オブジェクト。 |
| **media** | [MediaUrl](#mediaurl-object)[] | **MediaUrl** の配列。  このフィールドに複数の URL が含まれている場合、各 URL は同じコンテンツの代替形式です。 |
| **shareable** | boolean | ビデオを他のユーザーと共有できるかどうかを示すフラグ。 ビデオを共有できる場合、このプロパティを **true** に設定します。それ以外の場合、**false** に設定します。 既定値は **true** です。 |
| **subtitle** | string | カードのタイトルの下に表示するサブタイトル。 |
| **text** | string | カードのタイトルまたはサブタイトルの下に表示する説明またはプロンプト。 |
| **title** | string | カードのタイトル。 |
| **value** | object | このカードの補助パラメーター |

[スキーマの表に戻る](#schema)
