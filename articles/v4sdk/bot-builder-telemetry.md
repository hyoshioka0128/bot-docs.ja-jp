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
ms.date: 05/23/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 7225387933630eb7343a57aa849581ff1cbfbb0c
ms.sourcegitcommit: dbbfcf45a8d0ba66bd4fb5620d093abfa3b2f725
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 06/28/2019
ms.locfileid: "67464697"
---
# <a name="add-telemetry-to-your-bot"></a>ボットへのテレメトリの追加

[!INCLUDE[applies-to](../includes/applies-to.md)]

Bot Framework SDK のバージョン 4.2 では、テレメトリ ログ記録が製品に追加されました。  これにより、ボット アプリケーションは Application Insights などのサービスにイベント データを送信できます。 最初のセクションでこれらのメソッドについて説明してから、より広範なテレメトリ機能をその後で取り上げます。

このドキュメントでは、ボットを新しいテレメトリ機能と統合する方法について説明します。 

## <a name="basic-telemetry-options"></a>基本的なテレメトリ オプション

### <a name="basic-application-insights"></a>基本的な Application insights

まず、Application Insights を使用して、ボットに基本的なテレメトリを追加しましょう。 設定の詳細については、[Application Insights の概要](https://github.com/Microsoft/ApplicationInsights-aspnetcore/wiki/Getting-Started-with-Application-Insights-for-ASP.NET-Core)に関するページの最初のいくつかのセクションを参照してください。   

Application Insights 固有の追加構成 (テレメトリ初期化子など) を必要とせずに、Application Insights を "ストック" する場合は、`ConfigureServices()` メソッドに以下を追加します。   これが最も簡単な初期化方法です。Application Insights は、要求の追跡、他のサービスの外部呼び出し、サービス間でのイベントの関連付けを開始するように構成されます。

次のスニペットに含まれる NuGet パッケージを追加する必要があります。

**Startup.cs**
```csharp
using Microsoft.ApplicationInsights.Extensibility;
using Microsoft.Bot.Builder.ApplicationInsights;
using Microsoft.Bot.Builder.Integration.ApplicationInsights.Core;
using Microsoft.Bot.Builder.Integration.AspNet.Core;
 
// This method gets called by the runtime. Use this method to add services to the container.
public void ConfigureServices(IServiceCollection services)
{
    ...
    // Add Application Insights services into service collection
    services.AddApplicationInsightsTelemetry();

    // Add the standard telemetry client
    services.AddSingleton<IBotTelemetryClient, BotTelemetryClient>();

    // Add ASP middleware to store the HTTP body, mapped with bot activity key, in the httpcontext.items
    // This will be picked by the TelemetryBotIdInitializer
    services.AddTransient<TelemetrySaveBodyASPMiddleware>();

    // Add telemetry initializer that will set the correlation context for all telemetry items
    services.AddSingleton<ITelemetryInitializer, OperationCorrelationTelemetryInitializer>();

    // Add telemetry initializer that sets the user ID and session ID (in addition to other 
    // bot-specific properties, such as activity ID)
    services.AddSingleton<ITelemetryInitializer, TelemetryBotIdInitializer>();
    ...
}

// This method gets called by the runtime. Use this method to configure the HTTP request pipeline.
public void Configure(IApplicationBuilder app, IHostingEnvironment env)
{
    ...
    app.UseBotApplicationInsights();
}
```

次に、Application Insights のインストルメンテーション キーを `appsettings.json` ファイルに格納するか、環境変数として格納する必要があります。 `appsettings.json` ファイルには、ボットの実行中に使用される外部サービスに関するメタデータが格納されます。  たとえば、CosmosDB、Application Insights、Language Understanding (LUIS) サービスの接続とメタデータがここに保存されています。 インストルメンテーション キーは、Azure portal の **[概要]** セクション (折りたたまれている場合は、そのページのサービスの `Essentials` ドロップダウン) で確認できます。 キーを取得する方法の詳細については、[こちら](~/bot-service-resources-app-insights-keys.md)をご覧ください。

正しくフォーマットされている場合、フレームワークによってキーが検索されます。 `appsettings.json` エントリは次のようにフォーマットする必要があります。

```json
    "ApplicationInsights": {
        "InstrumentationKey": "putinstrumentationkeyhere"
    },
    "Logging": {
        "LogLevel": {
            "Default": "Warning"
        }
    }
```

ASP.NET Core アプリケーションに Application Insights を追加する方法の詳細については、[こちらの記事](https://docs.microsoft.com/azure/azure-monitor/app/asp-net-core-no-visualstudio)を参照してください。 

### <a name="customize-your-telemetry-client"></a>テレメトリ クライアントをカスタマイズする

Application Insights クライアントをカスタマイズする場合や、まったく別のサービスにログを記録する場合は、異なる方法でシステムを構成する必要があります。 NuGet からパッケージ `Microsoft.Bot.Builder.ApplicationInsights` をダウンロードするか、npm を使用して `botbuilder-applicationinsights` をインストールします。 Application Insights のキーを取得する方法の詳細については、[こちら](~/bot-service-resources-app-insights-keys.md)をご覧ください。

**Application Insights の構成を変更する**

構成を変更するには、Application Insights を追加する際に、`options` を含めます。 そうしないと、すべてが上記と同じになります。

```csharp
public void ConfigureServices(IServiceCollection services)
{
    ...
    // Add Application Insights services into service collection
    services.AddApplicationInsightsTelemetry(options);
    ...
}
```

`options` オブジェクトの型は `ApplicationInsightsServiceOptions` です。 これらのオプションの詳細については、[こちらをご覧ください]()。

**カスタム テレメトリを使用する**: Bot Framework によって生成されたテレメトリ イベントのログをまったく別のシステムに記録する場合は、基底インターフェイス `IBotTelemetryClient` から派生した新しいクラスを作成して構成します。 次に、上記のようにテレメトリ クライアントを追加する際に、使用するカスタム クライアントを挿入します。 

```csharp
public void ConfigureServices(IServiceCollection services)
{
    ...
    // Add the telemetry client.
    services.AddSingleton<IBotTelemetryClient, CustomTelemetryClient>();
    ...
}
```

### <a name="add-custom-logging-to-your-bot"></a>カスタム ログ記録をボットに追加する

ボットで新しいテレメトリ ログ記録のサポートを構成したら、ボットにテレメトリを追加できます。  `BotTelemetryClient` (C# では `IBotTelemetryClient`) には、異なる種類のイベントをログに記録するメソッドがいくつかあります。  適切な種類のイベントを選択すると、Application Insights の既存のレポートを利用できます (Application Insights を使用している場合)。  一般的なシナリオでは、通常、`TraceEvent` が使用されます。  `TraceEvent` を使用してログに記録されたデータは、Kusto の `CustomEvent` テーブルに格納されます。

ボット内でダイアログを使用している場合は、ダイアログ ベースのすべてのオブジェクト (プロンプトを含む) に新しい `TelemetryClient` プロパティが含まれます。  これは、ログ記録を実行できるようにする `BotTelemetryClient` です。  これは単なる便利なものではありません。この記事で後述するように、このプロパティを設定すると、`WaterfallDialogs` によってイベントが生成されます。

#### <a name="identifiers-and-custom-events"></a>識別子とカスタム イベント

イベントのログを Application Insights に記録する場合、生成されるイベントには、入力する必要のない既定のプロパティが含まれています。  たとえば、(`TraceEvent` API を使用して生成された) 各カスタム イベントには、`user_id` プロパティと `session_id` プロパティが含まれています。  さらに、`activitiId`、`activityType`、`channelId` も追加されます。

>注:カスタム テレメトリ クライアントには、これらの値は提供されません。

プロパティ |Type | 詳細
--- | --- | ---
`user_id`| `string` | [ChannelID](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#channel-id) + [From.Id](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#from)
`session_id`| `string`|  [ConversationID](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#conversation)
`customDimensions.activityId`| `string` | [ボット アクティビティ ID](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#id)
`customDimensions.activityType` | `string` | [ボット アクティビティの種類](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#channel-id)
`customDimensions.channelId` | `string` |  [ボット アクティビティのチャネル ID](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#channel-id)

## <a name="in-depth-telemetry"></a>詳細なテレメトリ

SDK バージョン 4.4 に追加された新しいコンポーネントは 3 つあります。  すべてのコンポーネントのログが、カスタム実装でオーバーライドできる `IBotTelemetryClient` (node.js の場合は `BotTelemetryClient`) インターフェイスを使用して記録されます。

- Bot Framework ミドルウェア コンポーネント (*TelemetryLoggerMiddleware*)。メッセージを受信、送信、更新、または削除したときにログに記録されます。 カスタム ログをオーバーライドできます。
- *LuisRecognizer* クラス。  2 つの方法でカスタム ログをオーバーライドできます - 呼び出しごと (プロパティの追加/置換) または派生クラス。
- *QnAMaker* クラス。  2 つの方法でカスタム ログをオーバーライドできます - 呼び出しごと (プロパティの追加/置換) または派生クラス。

### <a name="telemetry-middleware"></a>テレメトリ ミドルウェア

|C#  | JavaScript |
|:-----|:------------|
|**Microsoft.Bot.Builder.TelemetryLoggerMiddleware** | **botbuilder-core** |

#### <a name="out-of-box-usage"></a>標準の使用

TelemetryLoggerMiddleware は変更なしで追加できる Bot Framework コンポーネントで、これにより Bot Framework SDK 付属の標準レポートを有効にするログ記録が実行されます。 

```csharp
// Create the telemetry middleware to track conversation events
services.AddSingleton<IMiddleware, TelemetryLoggerMiddleware>();

// Create the Bot Framework Adapter with error handling enabled.
services.AddSingleton<IBotFrameworkHttpAdapter, AdapterWithErrorHandler>();
```

#### <a name="adding-properties"></a>プロパティの追加
追加プロパティを追加することにした場合は、TelemetryLoggerMiddleware クラスを派生させることができます。  たとえば、"MyImportantProperty" プロパティを `BotMessageReceived` イベントに追加するとします。  `BotMessageReceived` は、ユーザーがボットにメッセージを送信するときにログに記録されます。  追加プロパティは、次のように追加します。

```csharp
class MyTelemetryMiddleware : TelemetryLoggerMiddleware
{
    ...
    public Task OnReceiveActivityAsync(
                  Activity activity,
                  CancellationToken cancellation)
    {
        // Fill in the "standard" properties for BotMessageReceived
        // and add our own property.
        var properties = FillReceiveEventProperties(activity, 
                    new Dictionary<string, string>
                    { {"MyImportantProperty", "myImportantValue" } } );
                    
        // Use TelemetryClient to log event
        TelemetryClient.TrackEvent(
                        TelemetryLoggerConstants.BotMsgReceiveEvent,
                        properties);
    }
    ...
}
```

そして、起動時に新しいクラスを追加します。

```csharp
// Create the telemetry middleware to track conversation events
services.AddSingleton<IMiddleware, MyTelemetryMiddleware>();
```

#### <a name="completely-replacing-properties--additional-events"></a>プロパティ/追加イベントを完全に置き換える

ログに記録されているプロパティを完全に置き換えることにした場合は、`TelemetryLoggerMiddleware` クラスを派生させることができます (プロパティを拡張する場合は上記と同じ)。   同様に、新しいイベントのログ記録が同じ方法で実行されます。

たとえば、`BotMessageSend` プロパティを完全に置き換えて、複数のイベントを送信する場合の動作を次に示します。

```csharp
class MyTelemetryMiddleware : TelemetryLoggerMiddleware
{
    ...
    public Task<RecognizerResult> OnLuisRecognizeAsync(
                  Activity activity,
                  string dialogId = null,
                  CancellationToken cancellation)
    {
        // Override properties for BotMsgSendEvent
        var botMsgSendProperties = new Dictionary<string, string>();
        properties.Add("MyImportantProperty", "myImportantValue");
        // Log event
        TelemetryClient.TrackEvent(
                        TelemetryLoggerConstants.BotMsgSendEvent,
                        botMsgSendProperties);
                        
        // Create second event.
        var secondEventProperties = new Dictionary<string, string>();
        secondEventProperties.Add("activityId",
                                   activity.Id);
        secondEventProperties.Add("MyImportantProperty",
                                   "myImportantValue");
        TelemetryClient.TrackEvent(
                        "MySecondEvent",
                        secondEventProperties);
    }
    ...
}
```
注:標準プロパティがログに記録されていないと、製品に付属する標準レポートの動作が停止します。

#### <a name="events-logged-from-telemetry-middleware"></a>テレメトリ ミドルウェアからログに記録されたイベント
[BotMessageSend](#customevent-botmessagesend)
[BotMessageReceived](#customevent-botmessagereceived)
[BotMessageUpdate](#customevent-botmessageupdate)
[BotMessageDelete](#customevent-botmessagedelete)

### <a name="telemetry-support-luis"></a>テレメトリ サポート LUIS 

|C#  | JavaScript |
|:-----|:------------|
| **Microsoft.Bot.Builder.AI.Luis** | **botbuilder-ai** |

#### <a name="out-of-box-usage"></a>標準の使用
LuisRecognizer は既存の Bot Framework コンポーネントです。テレメトリを有効にするには、`luisOptions` を介して IBotTelemetryClient インターフェイスを渡します。  ログに記録された既定のプロパティをオーバーライドし、必要に応じて新しいイベントを記録できます。

`luisOptions` の作成中、これを機能させるには、`IBotTelemetryClient` オブジェクトを指定する必要があります。

```csharp
var luisOptions = new LuisPredictionOptions(
      ...
      telemetryClient,
      false); // Log personal information flag. Defaults to false.

var client = new LuisRecognizer(luisApp, luisOptions);
```

#### <a name="adding-properties"></a>プロパティの追加
追加プロパティを追加することにした場合は、`LuisRecognizer` クラスを派生させることができます。  たとえば、"MyImportantProperty" プロパティを `LuisResult` イベントに追加するとします。  `LuisResult` は、LUIS 予測呼び出しが実行されるときにログに記録されます。  追加プロパティは、次のように追加します。

```csharp
class MyLuisRecognizer : LuisRecognizer 
{
   ...
   override protected Task OnRecognizerResultAsync(
           RecognizerResult recognizerResult,
           ITurnContext turnContext,
           Dictionary<string, string> properties = null,
           CancellationToken cancellationToken = default(CancellationToken))
   {
       var luisEventProperties = FillLuisEventProperties(result, 
               new Dictionary<string, string>
               { {"MyImportantProperty", "myImportantValue" } } );
        
        TelemetryClient.TrackEvent(
                        LuisTelemetryConstants.LuisResultEvent,
                        luisEventProperties);
        ..
   }    
   ...
}
```

#### <a name="add-properties-per-invocation"></a>呼び出しごとにプロパティを追加する
呼び出し中、追加プロパティを追加する必要が出てくる場合があります。
```csharp
var additionalProperties = new Dictionary<string, string>
{
   { "dialogId", "myDialogId" },
   { "conversationInfo", "myConversationInfo" },
};

var result = await recognizer.RecognizeAsync(turnContext,
     additionalProperties,
     CancellationToken.None).ConfigureAwait(false);
```

#### <a name="completely-replacing-properties--additional-events"></a>プロパティ/追加イベントを完全に置き換える
ログに記録されているプロパティを完全に置き換えることにした場合は、`LuisRecognizer` クラスを派生させることができます (プロパティを拡張する場合は上記と同じ)。   同様に、新しいイベントのログ記録が同じ方法で実行されます。

たとえば、`LuisResult` プロパティを完全に置き換えて、複数のイベントを送信する場合の動作を次に示します。

```csharp
class MyLuisRecognizer : LuisRecognizer
{
    ...
    override protected Task OnRecognizerResultAsync(
             RecognizerResult recognizerResult,
             ITurnContext turnContext,
             Dictionary<string, string> properties = null,
             CancellationToken cancellationToken = default(CancellationToken))
    {
        // Override properties for LuisResult event
        var luisProperties = new Dictionary<string, string>();
        properties.Add("MyImportantProperty", "myImportantValue");
        
        // Log event
        TelemetryClient.TrackEvent(
                        LuisTelemetryConstants.LuisResult,
                        luisProperties);
                        
        // Create second event.
        var secondEventProperties = new Dictionary<string, string>();
        secondEventProperties.Add("MyImportantProperty2",
                                   "myImportantValue2");
        TelemetryClient.TrackEvent(
                        "MySecondEvent",
                        secondEventProperties);
        ...
    }
    ...
}
```
注:標準プロパティがログに記録されていないと、製品に付属する Application Insights 標準レポートの動作が停止します。

#### <a name="events-logged-from-telemetryluisrecognizer"></a>TelemetryLuisRecognizer からログに記録されたイベント
[LuisResult](#customevent-luisevent)

### <a name="telemetry-qna-recognizer"></a>テレメトリ QnA 認識エンジン

|C#  | JavaScript |
|:-----|:------------|
| **Microsoft.Bot.Builder.AI.QnA** | **botbuilder-ai** |


#### <a name="out-of-box-usage"></a>標準の使用
QnAMaker クラスは既存の Bot Framework コンポーネントで、ログ記録を有効にする 2 つの追加コンストラクター パラメーターを追加し、Bot Framework SDK に付属する標準レポートを有効にします。 新しい `telemetryClient` は、ログ記録を実行する `IBotTelemetryClient` インターフェイスを参照します。  

```csharp
var qna = new QnAMaker(endpoint, options, client, 
                       telemetryClient: telemetryClient,
                       logPersonalInformation: true);
```
#### <a name="adding-properties"></a>プロパティの追加 
追加プロパティを追加することにした場合、これを行う方法は 2 つあります。1 つは QnA 呼び出し中、回答を取得するためにプロパティの追加が必要な状況です。もう 1 つは `QnAMaker` クラスから派生させる方法です。  

次は `QnAMaker` クラスからの派生を示しています。  この例では、"MyImportantProperty" プロパティを `QnAMessage` イベントに追加しています。  `QnAMessage` イベントは、QnA `GetAnswers` 呼び出しの実行中に記録されます。  さらに、2 番目の "MySecondEvent" イベントを記録します。

```csharp
class MyQnAMaker : QnAMaker 
{
   ...
   protected override Task OnQnaResultsAsync(
                 QueryResult[] queryResults, 
                 ITurnContext turnContext, 
                 Dictionary<string, string> telemetryProperties = null, 
                 Dictionary<string, double> telemetryMetrics = null, 
                 CancellationToken cancellationToken = default(CancellationToken))
   {
            var eventData = await FillQnAEventAsync(queryResults, turnContext, telemetryProperties, telemetryMetrics, cancellationToken).ConfigureAwait(false);

            // Add my property
            eventData.Properties.Add("MyImportantProperty", "myImportantValue");

            // Log QnaMessage event
            TelemetryClient.TrackEvent(
                            QnATelemetryConstants.QnaMsgEvent,
                            eventData.Properties,
                            eventData.Metrics
                            );

            // Create second event.
            var secondEventProperties = new Dictionary<string, string>();
            secondEventProperties.Add("MyImportantProperty2",
                                       "myImportantValue2");
            TelemetryClient.TrackEvent(
                            "MySecondEvent",
                            secondEventProperties);       }    
    ...
}
```

#### <a name="adding-properties-during-getanswersasync"></a>GetAnswersAsync 実行中のプロパティの追加
実行時に追加する必要があるプロパティがある場合、`GetAnswersAsync` メソッドは、イベントに追加するプロパティやメトリックを提供できます。

たとえば、`dialogId` をイベントに追加する必要がある場合、これは次のように行われます。
```csharp
var telemetryProperties = new Dictionary<string, string>
{
   { "dialogId", myDialogId },
};

var results = await qna.GetAnswersAsync(context, opts, telemetryProperties);
```
`QnaMaker` クラスには、PersonalInfomation プロパティなどのプロパティをオーバーライドする機能もあります。

#### <a name="completely-replacing-properties--additional-events"></a>プロパティ/追加イベントを完全に置き換える
ログに記録されているプロパティを完全に置き換えることにした場合は、`TelemetryQnAMaker` クラスを派生させることができます (プロパティを拡張する場合は上記と同じ)。   同様に、新しいイベントのログ記録が同じ方法で実行されます。

たとえば、`QnAMessage` プロパティを完全に置き換える場合の動作を次に示します。

```csharp
class MyLuisRecognizer : TelemetryQnAMaker
{
    ...
    protected override Task OnQnaResultsAsync(
         QueryResult[] queryResults, 
         ITurnContext turnContext, 
         Dictionary<string, string> telemetryProperties = null, 
         Dictionary<string, double> telemetryMetrics = null, 
         CancellationToken cancellationToken = default(CancellationToken))
    {
        // Add properties from GetAnswersAsync
        var properties = telemetryProperties ?? new Dictionary<string, string>();
        // GetAnswerAsync properties overrides - don't add if already present.
        properties.TryAdd("MyImportantProperty", "myImportantValue");

        // Log event
        TelemetryClient.TrackEvent(
                           QnATelemetryConstants.QnaMsgEvent,
                            properties);
    }
    ...
}
```
注:標準プロパティがログに記録されていないと、製品に付属する標準レポートの動作が停止します。

#### <a name="events-logged-from-telemetryluisrecognizer"></a>TelemetryLuisRecognizer からログに記録されたイベント
[QnAMessage](#customevent-qnamessage)


## <a name="waterfalldialog-events"></a>WaterfallDialog イベント

独自のイベントの生成に加え、SDK 内の `WaterfallDialog` オブジェクトによってイベントが生成されるようになりました。 次のセクションでは、Bot Framework 内から生成されるイベントについて説明します。 `WaterfallDialog` で `TelemetryClient` プロパティを設定することによって、これらのイベントが保存されます。

テレメトリ イベントをログに記録するために、`WaterfallDialog` を使用するサンプル (CoreBot) を変更する例を次に示します。  CoreBot では、`WaterfallDialog` が `ComponentDialog` (`GreetingDialog`) 内に配置されている場合に使用される一般的なパターンを使用します。

```csharp
// IBotTelemetryClient is direct injected into our Bot
public CoreBot(BotServices services, UserState userState, ConversationState conversationState, IBotTelemetryClient telemetryClient)
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

## <a name="events-generated-by-the-bot-framework-service"></a>Bot Framework Service によって生成されるイベント

ボット コードからイベントを生成する `WaterfallDialog` だけでなく、Bot Framework Channel Service もイベントをログに記録します。  これは、チャネルに関する問題や全体的なボット エラーの診断に役立ちます。

### <a name="customevent-activity"></a>CustomEvent: "Activity"
**ログ記録元:** Channel Service メッセージを受信したときに、Channel Service によってログに記録されます。

### <a name="exception-bot-errors"></a>例外:"Bot Errors"
**ログ記録元:** Channel Service ボットの呼び出しから 2XX 以外の HTTP 応答が返されたときに、チャネルによってログに記録されます。

## <a name="all-events-generated"></a>生成されるすべてのイベント

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

### <a name="customevent-botmessagereceived"></a>CustomEvent: BotMessageReceived 
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

- ロケール
  - Bot Framework プロトコルの [From Name](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#from) フィールドに対応します。
  - ログに記録されるプロパティ名は `fromName` です。

### <a name="customevent-botmessagesend"></a>CustomEvent: BotMessageSend 
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


### <a name="customevent-botmessageupdate"></a>CustomEvent: BotMessageUpdate
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


### <a name="customevent-botmessagedelete"></a>CustomEvent: BotMessageDelete
**ログ記録元:** TelemetryLoggerMiddleware ボットによってメッセージが削除されたときにログに記録されます (まれなケース)。
- UserID ([テレメトリ初期化子から](https://aka.ms/telemetry-initializer))
- SessionID ([テレメトリ初期化子から](https://aka.ms/telemetry-initializer))
- ActivityID ([テレメトリ初期化子から](https://aka.ms/telemetry-initializer))
- Channel ([テレメトリ初期化子から](https://aka.ms/telemetry-initializer))
- ActivityType ([テレメトリ初期化子から](https://aka.ms/telemetry-initializer))
- RecipientId
- ConversationId
- ConversationName

### <a name="customevent-luisevent"></a>CustomEvent: LuisEvent
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

