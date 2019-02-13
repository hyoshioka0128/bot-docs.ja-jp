---
title: ボットへのテレメトリの追加 | Microsoft Docs
description: ボットを新しいテレメトリ機能と統合する方法について説明します。
keywords: テレメトリ, appinsights, ボットの監視
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 02/06/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 4c268bc40b7dc3315232d8f695bdb79343b15e21
ms.sourcegitcommit: c7d2e939ec71f46f48383c750fddaf6627b6489d
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/07/2019
ms.locfileid: "55795597"
---
# <a name="add-telemetry-to-your-bot"></a>ボットへのテレメトリの追加
Bot Framework SDK のバージョン 4.2 では、テレメトリ ログ記録が製品に追加されました。  これにより、ボット アプリケーションは Application Insights などのサービスにイベント データを送信できます。

このドキュメントでは、ボットを新しいテレメトリ機能と統合する方法について説明します。  

## <a name="using-bot-configuration-option-1-of-2"></a>ボット構成の使用 (オプション1/2)
ボットを構成する方法は 2 つあります。  1 つ目は、Application Insights と統合することを前提としています。

ボット構成ファイルには、ボットが実行中に使用する外部サービスに関するメタデータが含まれています。  たとえば、CosmosDB、Application Insights、Language Understanding (LUIS) サービスの接続とメタデータがここに保存されています。   

Application Insights 固有の追加構成 (テレメトリ初期化子など) を必要とせずに、Application Insights を "ストック" する場合は、初期化中にボット構成オブジェクトを渡します。   これが最も簡単な初期化方法です。Application Insights は、要求の追跡、他のサービスの外部呼び出し、サービス間でのイベントの関連付けを開始するように構成されます。

```csharp
// ASP.Net Core - Startup.cs

public void ConfigureServices(IServiceCollection services)
{
     ...
     // Add Application Insights - pass in the bot configuration
     services.AddBotApplicationInsights(botConfig);
     ...
}

public void Configure(IApplicationBuilder app, IHostingEnvironment env)
{
     app.UseBotApplicationInsights()
                 ...
                .UseDefaultFiles()
                .UseStaticFiles()
                .UseBotFramework();
                ...
}
```

```JavaScript
const appInsightsClient = new ApplicationInsightsTelemetryClient(botConfig);
```

**ボット構成に Application Insights が含まれていない場合:** ボット構成に Application Insights が含まれていない場合はどうなるのでしょうか。  問題ありません。既定では、メソッド呼び出しを行わない null クライアントになります。

**複数の Application Insights がある場合:** ボット構成に複数の Application Insights セクションがある場合は、  使用する Application Insights サービスのインスタンスをボット構成内で指定できます。

```csharp
// ASP.Net Core - Startup.cs

public void ConfigureServices(IServiceCollection services)
{
     // Add Application Insights
     services.AddBotApplicationInsights(botConfig, "myAppInsights");
     ...
}
```

```JavaScript
const appInsightsClient = new ApplicationInsightsTelemetryClient(botConfig);
```

## <a name="overriding-the-telemetry-client-option-2-of-2"></a>テレメトリ クライアントのオーバーライド (オプション2/2)

Application Insights クライアントをカスタマイズする場合や、まったく別のサービスにログを記録する場合は、異なる方法でシステムを構成する必要があります。

**Application Insights の構成を変更する**

```csharp

public void ConfigureServices(IServiceCollection services)
{
     ...
     // Create Application Insight Telemetry Client
     // with custom configuration.
     var telemetryClient = TelemetryClient(myCustomConfiguration)
     
     // Add Application Insights
     services.AddBotApplicationInsights(new BotTelemetryClient(telemetryClient), "InstrumentationKey");
```

```JavaScript
const appInsightsClient = new ApplicationInsightsTelemetryClient(botConfig);
```

**カスタム テレメトリを使用する**: Bot Framework によって生成されたテレメトリ イベントのログをまったく別のシステムに記録する場合は、基底インターフェイスから派生した新しいクラスを作成して構成します。  

```csharp
public void ConfigureServices(IServiceCollection services)
{
     ...
     // Create my IBotTelemetryClient-based logger
     var myTelemetryClient = MyTelemetryLogger();
     
     // Add Application Insights
     services.AddBotApplicationInsights(myTelemetryClient);
     ...
}
```

```JavaScript
const appInsightsClient = new ApplicationInsightsTelemetryClient(botConfig);
```

## <a name="add-custom-logging-to-your-bot"></a>カスタム ログ記録をボットに追加する

ボットで新しいテレメトリ ログ記録のサポートを構成したら、ボットにテレメトリを追加できます。  `BotTelemetryClient` (C# では `IBotTelemetryClient`) には、異なる種類のイベントをログに記録するメソッドがいくつかあります。  適切な種類のイベントを選択すると、Application Insights の既存のレポートを利用できます (Application Insights を使用している場合)。  一般的なシナリオでは、通常、`TraceEvent` が使用されます。  `TraceEvent` を使用してログに記録されたデータは、Kusto の `CustomEvent` テーブルに格納されます。

ボット内でダイアログを使用している場合は、ダイアログ ベースのすべてのオブジェクト (プロンプトを含む) に新しい `TelemetryClient` プロパティが含まれます。  これは、ログ記録を実行できるようにする `BotTelemetryClient` です。  これは単なる便利なものではありません。この記事で後述するように、このプロパティを設定すると、`WaterfallDialogs` によってイベントが生成されます。

### <a name="identifiers-and-custom-events"></a>識別子とカスタム イベント

イベントのログを Application Insights に記録する場合、生成されるイベントには、入力する必要のない既定のプロパティが含まれています。  たとえば、(`TraceEvent` API を使用して生成された) 各カスタム イベントには、`user_id` プロパティと `session_id` プロパティが含まれています。  さらに、`activitiId`、`activityType`、`channelId` も追加されます。

>注:カスタム テレメトリ クライアントには、これらの値は提供されません。

プロパティ |type | 詳細
--- | --- | ---
`user_id`| `string` | [ChannelID](https://github.com/Microsoft/botframework-obi/blob/master/botframework-activity/botframework-activity.md#channel-id) + [From.Id](https://github.com/Microsoft/botframework-obi/blob/master/botframework-activity/botframework-activity.md#from)
`session_id`| `string`|  [ConversationID](https://github.com/Microsoft/botframework-obi/blob/master/botframework-activity/botframework-activity.md#conversation)
`customDimensions.activityId`| `string` | [ボット アクティビティ ID](https://github.com/Microsoft/botframework-obi/blob/master/botframework-activity/botframework-activity.md#id)
`customDimensions.activityType` | `string` | [ボット アクティビティの種類](https://github.com/Microsoft/botframework-obi/blob/master/botframework-activity/botframework-activity.md#channel-id)
`customDimensions.channelId` | `string` |  [ボット アクティビティのチャネル ID](https://github.com/Microsoft/botframework-obi/blob/master/botframework-activity/botframework-activity.md#channel-id)

## <a name="waterfalldialog-events"></a>WaterfallDialog イベント

独自のイベントの生成に加え、SDK 内の `WaterfallDialog` オブジェクトによってイベントが生成されるようになりました。 次のセクションでは、Bot Framework 内から生成されるイベントについて説明します。 `WaterfallDialog` で `TelemetryClient` プロパティを設定することによって、これらのイベントが保存されます。

テレメトリ イベントをログに記録するために、`WaterfallDialog` を使用するサンプル (BasicBot) を変更する例を次に示します。  BasicBot では、`WaterfallDialog` が `ComponentDialog` (`GreetingDialog`) 内に配置されている場合に使用される一般的なパターンを使用します。

```csharp
// IBotTelemetryClient is direct injected into our Bot
public BasicBot(BotServices services, UserState userState, ConversationState conversationState, IBotTelemetryClient telemetryClient)
...

// The IBotTelemetryClient passed to the GreetingDialog
...
Dialogs = new DialogSet(_dialogStateAccessor);
Dialogs.Add(new GreetingDialog(_greetingStateAccessor, telemetryClient));
...

// The IBotTelemetryClient to the WaterfallDialog
...
AddDialog(new WaterfallDialog(ProfileDialog, waterfallSteps) { TelemetryClient = telemetryClient });
...

```

`WaterfallDialog` に `IBotTelemetryClient` が構成されると、イベントのログ記録が開始されます。

### <a name="customevent-waterfallstart"></a>CustomEvent: "WaterfallStart" 

WaterfallDialog が開始されると、`WaterfallStart` イベントがログに記録されます。

- `user_id` ([テレメトリ初期化子から](https://aka.ms/telemetry-initializer))
- `session_id` ([テレメトリ初期化子から](https://aka.ms/telemetry-initializer))
- `customDimensions.activityId` ([テレメトリ初期化子から](https://aka.ms/telemetry-initializer))
- `customDimensions.activityType` ([テレメトリ初期化子から](https://aka.ms/telemetry-initializer))
- `customDimensions.channelId` ([テレメトリ初期化子から](https://aka.ms/telemetry-initializer))
- `customDimensions.DialogId` (ウォーターフォールに渡される dialogId (文字列) です。  これは "ウォーターフォールの種類" と考えることができます)
- `customDimensions.InstanceID` (ダイアログのインスタンスごとに一意)

### <a name="customevent-waterfallstep"></a>CustomEvent: "WaterfallStep" 

ウォーターフォール ダイアログの個々のステップをログに記録します。

- `user_id` ([テレメトリ初期化子から](https://aka.ms/telemetry-initializer))
- `session_id` ([テレメトリ初期化子から](https://aka.ms/telemetry-initializer))
- `customDimensions.activityId` ([テレメトリ初期化子から](https://aka.ms/telemetry-initializer))
- `customDimensions.activityType` ([テレメトリ初期化子から](https://aka.ms/telemetry-initializer))
- `customDimensions.channelId` ([テレメトリ初期化子から](https://aka.ms/telemetry-initializer))
- `customDimensions.DialogId` (ウォーターフォールに渡される dialogId (文字列) です。  これは "ウォーターフォールの種類" と考えることができます)
- `customDimensions.StepName` (メソッド名または `StepXofY` (ラムダの場合))
- `customDimensions.InstanceID` (ダイアログのインスタンスごとに一意)

### <a name="customevent-waterfalldialogcomplete"></a>CustomEvent: "WaterfallDialogComplete"

ウォーターフォール ダイアログが完了したときにログに記録します。

- `user_id` ([テレメトリ初期化子から](https://aka.ms/telemetry-initializer))
- `session_id` ([テレメトリ初期化子から](https://aka.ms/telemetry-initializer))
- `customDimensions.activityId` ([テレメトリ初期化子から](https://aka.ms/telemetry-initializer))
- `customDimensions.activityType` ([テレメトリ初期化子から](https://aka.ms/telemetry-initializer))
- `customDimensions.channelId` ([テレメトリ初期化子から](https://aka.ms/telemetry-initializer))
- `customDimensions.DialogId` (ウォーターフォールに渡される dialogId (文字列) です。  これは "ウォーターフォールの種類" と考えることができます)
- `customDimensions.InstanceID` (ダイアログのインスタンスごとに一意)

### <a name="customevent-waterfalldialogcancel"></a>CustomEvent: "WaterfallDialogCancel" 

ウォーターフォール ダイアログがキャンセルされたときにログに記録します。

- `user_id` ([テレメトリ初期化子から](https://aka.ms/telemetry-initializer))
- `session_id` ([テレメトリ初期化子から](https://aka.ms/telemetry-initializer))
- `customDimensions.activityId` ([テレメトリ初期化子から](https://aka.ms/telemetry-initializer))
- `customDimensions.activityType` ([テレメトリ初期化子から](https://aka.ms/telemetry-initializer))
- `customDimensions.channelId` ([テレメトリ初期化子から](https://aka.ms/telemetry-initializer))
- `customDimensions.DialogId` (ウォーターフォールに渡される dialogId (文字列) です。  これは "ウォーターフォールの種類" と考えることができます)
- `customDimensions.StepName` (メソッド名または `StepXofY` (ラムダの場合))
- `customDimensions.InstanceID` (ダイアログのインスタンスごとに一意)


## <a name="events-generated-by-the-bot-framework-service"></a>Bot Framework Service によって生成されるイベント

ボット コードからイベントを生成する `WaterfallDialog` だけでなく、Bot Framework Channel Service もイベントをログに記録します。  これは、チャネルに関する問題や全体的なボット エラーの診断に役立ちます。

### <a name="customevent-activity"></a>CustomEvent: "Activity"
**ログ記録元:** Channel Service メッセージを受信したときに、Channel Service によってログに記録されます。

### <a name="exception-bot-errors"></a>例外:"Bot Errors"
**ログ記録元:** Channel Service ボットの呼び出しから 2XX 以外の HTTP 応答が返されたときに、チャネルによってログに記録されます。

## <a name="additional-events"></a>その他のイベント

[Enterprise Template](https://github.com/Microsoft/AI/tree/master/templates/Enterprise-Template) は、自由にコピーできるオープン ソース コードです。  これには、再利用し、レポートのニーズに合わせて変更できるコンポーネントがいくつか含まれています。

### <a name="customevent-botmessagereceived"></a>CustomEvent: BotMessageReceived 
**ログ記録元:** TelemetryLoggerMiddleware (**Enterprise サンプル**)

ボットが新しいメッセージを受信したときにログに記録されます。

- UserID ([テレメトリ初期化子から](https://aka.ms/telemetry-initializer))
- ConversationID ([テレメトリ初期化子から](https://aka.ms/telemetry-initializer))
- ActivityID ([テレメトリ初期化子から](https://aka.ms/telemetry-initializer))
- Channel ([テレメトリ初期化子から](https://aka.ms/telemetry-initializer))
- ActivityType ([テレメトリ初期化子から](https://aka.ms/telemetry-initializer))
- Text (PII では省略可能)
- FromId
- FromName
- RecipientId
- RecipientName
- ConversationId
- ConversationName
- ロケール

### <a name="customevent-botmessagesend"></a>CustomEvent: BotMessageSend 
**ログ記録元:** TelemetryLoggerMiddleware (**Enterprise サンプル**)

ボットがメッセージを送信したときにログに記録されます。

- UserID ([テレメトリ初期化子から](https://aka.ms/telemetry-initializer))
- ConversationID ([テレメトリ初期化子から](https://aka.ms/telemetry-initializer))
- ActivityID ([テレメトリ初期化子から](https://aka.ms/telemetry-initializer))
- Channel ([テレメトリ初期化子から](https://aka.ms/telemetry-initializer))
- ActivityType ([テレメトリ初期化子から](https://aka.ms/telemetry-initializer))
- ReplyToID
- Channel (ソース チャネル- Skype、Cortana、Teams など)
- RecipientId
- ConversationName
- ロケール
- Text (PII では省略可能)
- RecipientName (PII では省略可能)

### <a name="customevent-botmessageupdate"></a>CustomEvent: BotMessageUpdate
**ログ記録元:** TelemetryLoggerMiddleware ボットによってメッセージが更新されたときにログに記録されます (まれなケース)。

### <a name="customevent-botmessagedelete"></a>CustomEvent: BotMessageDelete
**ログ記録元:** TelemetryLoggerMiddleware ボットによってメッセージが削除されたときにログに記録されます (まれなケース)。

### <a name="customevent-luisintentinentname"></a>CustomEvent: LuisIntent.INENTName 
**ログ記録元:** TelemetryLuisRecognizer (**Enterprise サンプル**)

LUIS サービスの結果をログに記録します。

- UserID ([テレメトリ初期化子から](https://aka.ms/telemetry-initializer))
- ConversationID ([テレメトリ初期化子から](https://aka.ms/telemetry-initializer))
- ActivityID ([テレメトリ初期化子から](https://aka.ms/telemetry-initializer))
- Channel ([テレメトリ初期化子から](https://aka.ms/telemetry-initializer))
- ActivityType ([テレメトリ初期化子から](https://aka.ms/telemetry-initializer))
- 意図
- IntentScore
- 質問
- ConversationId
- SentimentLabel
- SentimentScore
- *LUIS エンティティ*
- **新しい** DialogId

### <a name="customevent-qnamessage"></a>CustomEvent: QnAMessage
**ログ記録元:** TelemetryQnaMaker (**Enterprise サンプル**)

QnA Maker サービスの結果をログに記録します。

- UserID ([テレメトリ初期化子から](https://aka.ms/telemetry-initializer))
- ConversationID ([テレメトリ初期化子から](https://aka.ms/telemetry-initializer))
- ActivityID ([テレメトリ初期化子から](https://aka.ms/telemetry-initializer))
- Channel ([テレメトリ初期化子から](https://aka.ms/telemetry-initializer))
- ActivityType ([テレメトリ初期化子から](https://aka.ms/telemetry-initializer))
- ユーザー名
- ConversationId
- OriginalQuestion
- 質問
- Answer
- Score (*省略可能*: ナレッジが見つかった場合)

## <a name="querying-the-data"></a>データのクエリ
Application Insights を使用すると、すべてのデータが (サービス間でも) 相互に関連付けられます。  成功した要求のクエリを実行することで、その要求の関連するすべてのイベントを確認できます。  
以下のクエリによって、最新の要求がわかります。
```sql
requests 
| where timestamp > ago(3d) 
| where resultCode == 200
| order by timestamp desc
| project timestamp, operation_Id, appName
| limit 10
```

最初のクエリから、`operation_Id` をいくつか選択し、詳細情報を探します。

```sql
let my_operation_id = "<OPERATION_ID>";
let union_all = () {
    union
    (traces | where operation_Id == my_operation_id),
    (customEvents | where operation_Id == my_operation_id),
    (requests | where operation_Id == my_operation_id),
    (dependencies | where operation_Id  == my_operation_id),
    (exceptions | where operation_Id == my_operation_id)
};
union_all
    | order by timestamp asc
    | project itemType, name, performanceBucket
```

これにより、各呼び出しの期間バケットで、1 つの要求の内訳が時系列で表示されます。
![呼び出しの例](media/performance_query.png)

> 注:"Activity" `customEvent` イベントは非同期でログに記録されるため、これらのイベントのタイムスタンプの順序は正しくありません。

## <a name="create-a-dashboard"></a>ダッシュボードを作成する

最も簡単なテスト方法は、[Azure portal の [テンプレートのデプロイ] ページ](https://portal.azure.com/#create/Microsoft.Template)を使用してダッシュボードを作成することです。  
- [Build your own template in the editor] \(エディターで独自のテンプレートをビルド\) をクリックします。
- ダッシュボードの作成を支援するために用意されている、次のいずれかの .json ファイルをコピーして貼り付けます。
  - [System Health Dashboard](https://aka.ms/system-health-appinsights)
  - [Conversation Health Dashboard](https://aka.ms/conversation-health-appinsights)
- [保存] をクリックします
- `Basics` の設定: 
   - サブスクリプション: <your test subscription>
   - リソース グループ: <a test resource group>
   - 場所: <such as West US>
- `Settings` の設定:
   - Insights コンポーネント名: <例: `core672so2hw`>
   - Insights コンポーネントのリソース グループ: <例: `core67`>
   - ダッシュボード名: <例: `'ConversationHealth'` や `SystemHealth`>
- [`I agree to the terms and conditions stated above`] をクリックします。
- [`Purchase`] をクリックします。
- 検証
   - [[`Resource Groups`]](https://ms.portal.azure.com/#blade/HubsExtension/Resources/resourceType/Microsoft.Resources%2Fsubscriptions%2FresourceGroups) をクリックします
   - 上記からリソース グループ (`core67` など) を選択します。
   - 新しいリソースが表示されない場合は、[デプロイ] でエラーが発生していないかどうかを確認します。
   - エラーが発生した場合に通常表示される内容を次に示します。
     
```json
{"code":"DeploymentFailed","message":"At least one resource deployment operation failed. Please list deployment operations for details. Please see https://aka.ms/arm-debug for usage details.","details":[{"code":"BadRequest","message":"{\r\n \"error\": {\r\n \"code\": \"InvalidTemplate\",\r\n \"message\": \"Unable to process template language expressions for resource '/subscriptions/45d8a30e-3363-4e0e-849a-4bb0bbf71a7b/resourceGroups/core67/providers/Microsoft.Portal/dashboards/Bot Analytics Dashboard' at line '34' and column '9'. 'The template parameter 'virtualMachineName' is not found. Please see https://aka.ms/arm-template/#parameters for usage details.'\"\r\n }\r\n}"}]}
```

データを表示するには、Azure portal に移動します。 左側の **[ダッシュボード]** をクリックし、ドロップダウンから作成したダッシュボードを選択します。

## <a name="additional-resources"></a>その他のリソース
テレメトリを実装した次のサンプルを参照してください。
- C#
  - [LUIS with AppInsights](https://aka.ms/luis-with-appinsights-cs)
  - [QnA with AppInsights](https://aka.ms/qna-with-appinsights-cs)
- JS
  - [LUIS with AppInsights](https://aka.ms/luis-with-appinsights-js)
  - [QnA with AppInsights](https://aka.ms/qna-with-appinsights-js)

