---
title: Bot Framework Service テレメトリによって生成されるイベント | Microsoft Docs
description: 新しいテレメトリ機能によってトリガーされるイベントについて説明します。
keywords: テレメトリ, appinsights, ボットの監視
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 05/23/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 929432d51a7bdfbcbd328ae7ea4e0df8873e7392
ms.sourcegitcommit: 23a1808e18176f1704f2f6f2763ace872b1388ae
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/25/2019
ms.locfileid: "68484359"
---
# <a name="events-generated-by-the-bot-framework-service-telemetry"></a>Bot Framework Service テレメトリによって生成されるイベント

## <a name="channel-service-events"></a>チャネル サービス イベント

[テレメトリに関するトピック](bot-builder-telemetry.md)で説明した、ボット コードからイベントを生成する `WaterfallDialog` だけでなく、Bot Framework チャネル サービスもイベントをログに記録します。  これは、チャネルに関する問題や全体的なボット エラーの診断に役立ちます。

### <a name="customevent-activity"></a>CustomEvent: "Activity"
**ログ記録元:** Channel Service メッセージを受信したときに、Channel Service によってログに記録されます。

### <a name="exception-bot-errors"></a>例外:"Bot Errors"
**ログ記録元:** Channel Service ボットの呼び出しから 2XX 以外の HTTP 応答が返されたときに、チャネルによってログに記録されます。

## <a name="customevent-waterfallstart"></a>CustomEvent: "WaterfallStart" 

WaterfallDialog が開始されると、`WaterfallStart` イベントがログに記録されます。

- `user_id` ([テレメトリ初期化子から](https://aka.ms/telemetry-initializer))
- `session_id` ([テレメトリ初期化子から](https://aka.ms/telemetry-initializer))
- `customDimensions.activityId` ([テレメトリ初期化子から](https://aka.ms/telemetry-initializer))
- `customDimensions.activityType` ([テレメトリ初期化子から](https://aka.ms/telemetry-initializer))
- `customDimensions.channelId` ([テレメトリ初期化子から](https://aka.ms/telemetry-initializer))
- `customDimensions.DialogId` (ウォーターフォールに渡される dialogId (文字列) です。  これは "ウォーターフォールの種類" と考えることができます)
- `customDimensions.InstanceID` (ダイアログのインスタンスごとに一意)

## <a name="customevent-waterfallstep"></a>CustomEvent: "WaterfallStep" 

ウォーターフォール ダイアログの個々のステップをログに記録します。

- `user_id` ([テレメトリ初期化子から](https://aka.ms/telemetry-initializer))
- `session_id` ([テレメトリ初期化子から](https://aka.ms/telemetry-initializer))
- `customDimensions.activityId` ([テレメトリ初期化子から](https://aka.ms/telemetry-initializer))
- `customDimensions.activityType` ([テレメトリ初期化子から](https://aka.ms/telemetry-initializer))
- `customDimensions.channelId` ([テレメトリ初期化子から](https://aka.ms/telemetry-initializer))
- `customDimensions.DialogId` (ウォーターフォールに渡される dialogId (文字列) です。  これは "ウォーターフォールの種類" と考えることができます)
- `customDimensions.StepName` (メソッド名または `StepXofY` (ラムダの場合))
- `customDimensions.InstanceID` (ダイアログのインスタンスごとに一意)

## <a name="customevent-waterfalldialogcomplete"></a>CustomEvent: "WaterfallDialogComplete"

ウォーターフォール ダイアログが完了したときにログに記録します。

- `user_id` ([テレメトリ初期化子から](https://aka.ms/telemetry-initializer))
- `session_id` ([テレメトリ初期化子から](https://aka.ms/telemetry-initializer))
- `customDimensions.activityId` ([テレメトリ初期化子から](https://aka.ms/telemetry-initializer))
- `customDimensions.activityType` ([テレメトリ初期化子から](https://aka.ms/telemetry-initializer))
- `customDimensions.channelId` ([テレメトリ初期化子から](https://aka.ms/telemetry-initializer))
- `customDimensions.DialogId` (ウォーターフォールに渡される dialogId (文字列) です。  これは "ウォーターフォールの種類" と考えることができます)
- `customDimensions.InstanceID` (ダイアログのインスタンスごとに一意)

## <a name="customevent-waterfalldialogcancel"></a>CustomEvent: "WaterfallDialogCancel" 

ウォーターフォール ダイアログがキャンセルされたときにログに記録します。

- `user_id` ([テレメトリ初期化子から](https://aka.ms/telemetry-initializer))
- `session_id` ([テレメトリ初期化子から](https://aka.ms/telemetry-initializer))
- `customDimensions.activityId` ([テレメトリ初期化子から](https://aka.ms/telemetry-initializer))
- `customDimensions.activityType` ([テレメトリ初期化子から](https://aka.ms/telemetry-initializer))
- `customDimensions.channelId` ([テレメトリ初期化子から](https://aka.ms/telemetry-initializer))
- `customDimensions.DialogId` (ウォーターフォールに渡される dialogId (文字列) です。  これは "ウォーターフォールの種類" と考えることができます)
- `customDimensions.StepName` (メソッド名または `StepXofY` (ラムダの場合))
- `customDimensions.InstanceID` (ダイアログのインスタンスごとに一意)

## <a name="customevent-botmessagereceived"></a>CustomEvent: BotMessageReceived 
ボットがユーザーから新しいメッセージを受信したときにログに記録されます。

オーバーライドされていない場合、このイベントは、`Microsoft.Bot.Builder.IBotTelemetry.TrackEvent()` メソッドを使用して `Microsoft.Bot.Builder.TelemetryLoggerMiddleware` からログに記録されます。

- セッション ID  
  - Application Insights を使用する場合、これは、Application Insights 内で使用されている**セッション** ID (*Temeletry.Context.Session.Id*) として、`TelemetryBotIdInitializer` からログに記録されます。  
  - Bot Framework プロトコルで定義されている [Conversation ID](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#conversation) に対応します。
  - ログに記録されるプロパティ名は `session_id` です。

- ユーザー ID
  - Application Insights を使用する場合、これは、Application Insights 内で使用されている**ユーザー** ID (*Temeletry.Context.User.Id*) として、`TelemetryBotIdInitializer` からログに記録されます。  
  - このプロパティの値は、Bot Framework プロトコルによって定義されている (連結された) [Channel ID](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#channel-id) プロパティと [User ID](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#from) プロパティの組み合わせです。
  - ログに記録されるプロパティ名は `user_id` です。

- ActivityID 
  - Application Insights を使用する場合、これはイベントに対するプロパティとして、`TelemetryBotIdInitializer` からログに記録されます。
  - Bot Framework プロトコルで定義されている [Activity ID](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#Id) に対応します。
  - プロパティ名は `activityId` です。

- チャネル ID
  - Application Insights を使用する場合、これはイベントに対するプロパティとして、`TelemetryBotIdInitializer` からログに記録されます。  
  - Bot Framework プロトコルの [Channel ID](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#id) に対応します。
  - ログに記録されるプロパティ名は `channelId` です。

- ActivityType 
  - Application Insights を使用する場合、これはイベントに対するプロパティとして、`TelemetryBotIdInitializer` からログに記録されます。  
  - Bot Framework プロトコルの [Activity Type](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#type) に対応します。
  - ログに記録されるプロパティ名は `activityType` です。

- Text
  - `logPersonalInformation` プロパティが `true` に設定されている場合に、**必要に応じて**ログに記録されます。
  - Bot Framework プロトコルの [Activity Text](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#text) フィールドに対応します。
  - ログに記録されるプロパティ名は `text` です。

- Speak

  - `logPersonalInformation` プロパティが `true` に設定されている場合に、**必要に応じて**ログに記録されます。
  - Bot Framework プロトコルの [Activity Speak](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#speak) フィールドに対応します。
  - ログに記録されるプロパティ名は `speak` です。

  - 

- FromId
  - Bot Framework プロトコルの [From ID](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#from) フィールドに対応します。
  - ログに記録されるプロパティ名は `fromId` です。

- FromName
  - `logPersonalInformation` プロパティが `true` に設定されている場合に、**必要に応じて**ログに記録されます。
  - Bot Framework プロトコルの [From Name](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#from) フィールドに対応します。
  - ログに記録されるプロパティ名は `fromName` です。

- RecipientId
  - Bot Framework プロトコルの [From Name](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#from) フィールドに対応します。
  - ログに記録されるプロパティ名は `fromName` です。

- RecipientName
  - Bot Framework プロトコルの [From Name](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#from) フィールドに対応します。
  - ログに記録されるプロパティ名は `fromName` です。

- ConversationId
  - Bot Framework プロトコルの [From Name](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#from) フィールドに対応します。
  - ログに記録されるプロパティ名は `fromName` です。

- ConversationName
  - Bot Framework プロトコルの [From Name](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#from) フィールドに対応します。
  - ログに記録されるプロパティ名は `fromName` です。

- Locale
  - Bot Framework プロトコルの [From Name](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#from) フィールドに対応します。
  - ログに記録されるプロパティ名は `fromName` です。

## <a name="customevent-botmessagesend"></a>CustomEvent: BotMessageSend 
**ログ記録元:** TelemetryLoggerMiddleware 

ボットがメッセージを送信したときにログに記録されます。

- UserID ([テレメトリ初期化子から](https://aka.ms/telemetry-initializer))
- SessionID ([テレメトリ初期化子から](https://aka.ms/telemetry-initializer))
- ActivityID ([テレメトリ初期化子から](https://aka.ms/telemetry-initializer))
- Channel ([テレメトリ初期化子から](https://aka.ms/telemetry-initializer))
- ActivityType ([テレメトリ初期化子から](https://aka.ms/telemetry-initializer))
- ReplyToID
- RecipientId
- ConversationName
- Locale
- RecipientName (PII では省略可能)
- Text (PII では省略可能)
- Speak (PII では省略可)


## <a name="customevent-botmessageupdate"></a>CustomEvent: BotMessageUpdate
**ログ記録元:** TelemetryLoggerMiddleware ボットによってメッセージが更新されたときにログに記録されます (まれなケース)。
- UserID ([テレメトリ初期化子から](https://aka.ms/telemetry-initializer))
- SessionID ([テレメトリ初期化子から](https://aka.ms/telemetry-initializer))
- ActivityID ([テレメトリ初期化子から](https://aka.ms/telemetry-initializer))
- Channel ([テレメトリ初期化子から](https://aka.ms/telemetry-initializer))
- ActivityType ([テレメトリ初期化子から](https://aka.ms/telemetry-initializer))
- RecipientId
- ConversationId
- ConversationName
- Locale
- Text (PII では省略可能)


## <a name="customevent-botmessagedelete"></a>CustomEvent: BotMessageDelete
**ログ記録元:** TelemetryLoggerMiddleware ボットによってメッセージが削除されたときにログに記録されます (まれなケース)。
- UserID ([テレメトリ初期化子から](https://aka.ms/telemetry-initializer))
- SessionID ([テレメトリ初期化子から](https://aka.ms/telemetry-initializer))
- ActivityID ([テレメトリ初期化子から](https://aka.ms/telemetry-initializer))
- Channel ([テレメトリ初期化子から](https://aka.ms/telemetry-initializer))
- ActivityType ([テレメトリ初期化子から](https://aka.ms/telemetry-initializer))
- RecipientId
- ConversationId
- ConversationName

## <a name="customevent-luisevent"></a>CustomEvent: LuisEvent
**ログ記録元:** LuisRecognizer

LUIS サービスの結果をログに記録します。

- UserID ([テレメトリ初期化子から](https://aka.ms/telemetry-initializer))
- SessionID ([テレメトリ初期化子から](https://aka.ms/telemetry-initializer))
- ActivityID ([テレメトリ初期化子から](https://aka.ms/telemetry-initializer))
- Channel ([テレメトリ初期化子から](https://aka.ms/telemetry-initializer))
- ActivityType ([テレメトリ初期化子から](https://aka.ms/telemetry-initializer))
- ApplicationId
- Intent
- IntentScore
- Intent2 
- IntentScore2 
- FromId
- SentimentLabel
- SentimentScore
- Entities (json として)
- Question (PII では省略可能)

## <a name="customevent-qnamessage"></a>CustomEvent: QnAMessage
**ログ記録元:** QnA Maker

QnA Maker サービスの結果をログに記録します。

- UserID ([テレメトリ初期化子から](https://aka.ms/telemetry-initializer))
- SessionID ([テレメトリ初期化子から](https://aka.ms/telemetry-initializer))
- ActivityID ([テレメトリ初期化子から](https://aka.ms/telemetry-initializer))
- Channel ([テレメトリ初期化子から](https://aka.ms/telemetry-initializer))
- ActivityType ([テレメトリ初期化子から](https://aka.ms/telemetry-initializer))
- Username (PII では省略可能)
- Question (PII では省略可能)
- MatchedQuestion
- QuestionId
- Answer
- Score
- ArticleFound

