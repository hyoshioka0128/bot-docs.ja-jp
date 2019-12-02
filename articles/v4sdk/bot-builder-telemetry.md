---
title: ボットへのテレメトリの追加 | Microsoft Docs
description: ボットを新しいテレメトリ機能と統合する方法について説明します。
keywords: テレメトリ, appinsights, ボットの監視
author: WashingtonKayaker
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 07/17/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: a023fd97bfb7b8d55ad01d118075a6441e426575
ms.sourcegitcommit: 08f9dc91152e0d4565368f72f547cdea1885af89
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/26/2019
ms.locfileid: "74510756"
---
# <a name="add-telemetry-to-your-bot"></a>ボットへのテレメトリの追加

[!INCLUDE[applies-to](../includes/applies-to.md)]


Bot Framework SDK のバージョン 4.2 にテレメトリのログ記録が追加されました。  これにより、ボット アプリケーションは [Application Insights](https://aka.ms/appinsights-overview) などのテレメトリ サービスにイベント データを送信できます。 テレメトリは、どの機能が最も使用されているかを示すことによりボットの分析情報を提供し、不要な動作を検出し、可用性、パフォーマンス、および使用状況を可視化します。

***注:バージョン 4.6 では、カスタム アダプターを使用するときにテレメトリが正しく記録されるように、ボットにテレメトリを実装するための標準的な方法が更新されました。この記事は更新され、更新された方法が表示されるようになりました。これらの変更は下位互換性があり、前の方法を使用するボットはテレメトリを正しくログに記録し続けます。***


この記事では、Application Insights を使用してテレメトリをボットに組み入れる方法について説明します。

* テレメトリをボットにつないで Application Insights に接続するために必要なコード

* ボットの[ダイアログ](bot-builder-concept-dialog.md)でテレメトリを有効にする

* テレメトリが [LUIS](bot-builder-howto-v4-luis.md) や [QnA Maker](bot-builder-howto-qna.md) などの他のサービスから使用状況データを取り込めるようにする

* テレメトリ データを Application Insights で視覚化する

## <a name="prerequisites"></a>前提条件

* [CoreBot サンプル コード](https://aka.ms/cs-core-sample)

* [Application Insights サンプル コード](https://aka.ms/csharp-corebot-app-insights-sample)

* [Microsoft Azure](https://portal.azure.com/) のサブスクリプション。

* [Application Insights のキー](../bot-service-resources-app-insights-keys.md)

* [Application Insights](https://aka.ms/appinsights-overview) を熟知していること

> [!NOTE]
> [Application Insights サンプル コード](https://aka.ms/csharp-corebot-app-insights-sample)は、[CoreBot サンプル コード](https://aka.ms/cs-core-sample)を基にして構築されました。 この記事では、テレメトリを組み込むための CoreBot サンプル コードの変更手順について説明します。 Visual Studio で作業を進めていくと、完了時までに Application Insights のサンプル コードが作成されます。

## <a name="wiring-up-telemetry-in-your-bot"></a>ボットでテレメトリを接続する

[CoreBot サンプル アプリ](https://aka.ms/cs-core-sample)から開始して、テレメトリをボットに統合するために必要なコードを追加します。 これにより、Application Insights は要求の追跡を開始できるようになります。

1. Visual Studio で [CoreBot サンプル アプリ](https://aka.ms/cs-core-sample)を開きます

2. `Microsoft.Bot.Builder.Integration.ApplicationInsights.Core ` NuGet パッケージを追加します。 NuGet の使用方法について詳しくは、「[Visual Studio にパッケージをインストールして管理する](https://aka.ms/install-manage-packages-vs)」を参照してください。


3. 次のステートメントを `Startup.cs` に含めます。
    ```csharp
    using Microsoft.ApplicationInsights.Extensibility;
    using Microsoft.Bot.Builder.ApplicationInsights;
    using Microsoft.Bot.Builder.Integration.ApplicationInsights.Core;
    using Microsoft.Bot.Builder.Integration.AspNet.Core;
    ```

    注:CoreBot サンプル コードを更新して作業を進めている場合は、`Microsoft.Bot.Builder.Integration.AspNet.Core` の using ステートメントが既に CoreBot サンプルにあることに気付きます。

4. 次のコードを `Startup.cs` の `ConfigureServices()` メソッドに含めます。 こうすることで、[依存関係の挿入 (DI)](https://aka.ms/asp.net-core-dependency-interjection) によりテレメトリ サービスをボットで使用できるようになります。
    ```csharp
    // This method gets called by the runtime. Use this method to add services to the container.
    public void ConfigureServices(IServiceCollection services)
    {
        ...
        // Create the Bot Framework Adapter with error handling enabled.
        services.AddSingleton<IBotFrameworkHttpAdapter, AdapterWithErrorHandler>();

        // Add Application Insights services into service collection
        services.AddApplicationInsightsTelemetry();

        // Add the standard telemetry client
        services.AddSingleton<IBotTelemetryClient, BotTelemetryClient>();

        // Create the telemetry middleware to track conversation events
        services.AddSingleton<TelemetryLoggerMiddleware>();

        // Add the telemetry initializer middleware
        services.AddSingleton<IMiddleware, TelemetryInitializerMiddleware>();

        // Add telemetry initializer that will set the correlation context for all telemetry items
        services.AddSingleton<ITelemetryInitializer, OperationCorrelationTelemetryInitializer>();

        // Add telemetry initializer that sets the user ID and session ID (in addition to other bot-specific properties, such as activity ID)
        services.AddSingleton<ITelemetryInitializer, TelemetryBotIdInitializer>();
        ...
    }
    ```
    
    注:CoreBot サンプル コードを更新して作業を進めている場合は、`services.AddSingleton<IBotFrameworkHttpAdapter, AdapterWithErrorHandler>();` が既に存在していることに気付きます。 

5. `ConfigureServices()` メソッドに追加されたミドルウェア コードを使用するようにアダプターに指示します。 これは、次に示すように、`AdapterWithErrorHandler.cs` で、コンストラクター パラメーター一覧のパラメーター IMiddleware middleware とコンストラクターの `Use(middleware);` ステートメントを使用して行います。
    ```csharp
    public AdapterWithErrorHandler(ICredentialProvider credentialProvider, ILogger<BotFrameworkHttpAdapter> logger, IMiddleware middleware, ConversationState conversationState = null)
            : base(credentialProvider)
    {
        ...

        Use(middleware);
    }
    ```

6. Application Insights のインストルメンテーション キーを `appsettings.json` ファイルに追加します。`appsettings.json` ファイルには、ボットが実行中に使用する外部サービスに関するメタデータが含まれています。 たとえば、CosmosDB、Application Insights、Language Understanding (LUIS) サービスの接続とメタデータがそこに保存されています。 `appsettings.json` ファイルへの追加は、次の形式にする必要があります。

    ```json
    {
        "MicrosoftAppId": "",
        "MicrosoftAppPassword": "",
        "ApplicationInsights": {
            "InstrumentationKey": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
        }
    }
    ```
    注:_Application Insights のインストルメンテーション キー_の取得の詳細については、「[Application Insights キー](../bot-service-resources-app-insights-keys.md)」の記事をご覧ください。

この時点で、Application Insights を使用してテレメトリを有効にする準備作業が完了しました。  ボット エミュレーターを使用してボットをローカルで実行した後に、Application Insights にアクセスして、応答時間、アプリ全体の正常性、一般的な実行情報などのログ記録を確認できます。 

## <a name="enabling--disabling-activity-event-and-personal-information-logging"></a>アクティビティ イベントと個人情報のログ記録の有効化/無効化

### <a name="enabling-or-disabling-activity-logging"></a>アクティビティのログ記録を有効または無効にする

既定では、`TelemetryInitializerMiddleware` は `TelemetryLoggerMiddleware` を使用して、ボットがアクティビティを送信/受信するときにテレメトリをログに記録します。 アクティビティ ログでは、Application Insights リソースにカスタム イベント ログが作成されます。  必要に応じて、**Startup.cs** 内に登録する前に、`TelemetryInitializerMiddleware` で `logActivityTelemetry` を false に設定して、アクティビティ イベントのログ記録を無効にできます。

```cs
public void ConfigureServices(IServiceCollection services)
{
    ...
    // Add the telemetry initializer middleware
    services.AddSingleton<IMiddleware, TelemetryInitializerMiddleware>(sp =>
            {
                var httpContextAccessor = sp.GetService<IHttpContextAccessor>();
                var loggerMiddleware = sp.GetService<TelemetryLoggerMiddleware>();
                return new TelemetryInitializerMiddleware(httpContextAccessor, loggerMiddleware, logActivityTelemetry: false);
            });
    ...
}
```

### <a name="enable-or-disable-logging-personal-information"></a>個人情報のログ記録を有効または無効にする

既定では、アクティビティ ログが有効になっている場合、受信/送信アクティビティの一部のプロパティは、ユーザー名やアクティビティ テキストなどの個人情報が含まれる可能性があるため、ログから除外されます。 これらのプロパティをログに含めるには、`TelemetryLoggerMiddleware` の登録時に **Startup.cs** に次の変更を加える必要があります。

```cs
public void ConfigureServices(IServiceCollection services)
{
    ...
    // Add the telemetry initializer middleware
    services.AddSingleton<TelemetryLoggerMiddleware>(sp =>
            {
                var telemetryClient = sp.GetService<IBotTelemetryClient>();
                return new TelemetryLoggerMiddleware(telemetryClient, logPersonalInformation: true);
            });
    ...
}
```

次に、テレメトリ機能をダイアログに追加するために含める必要があるものを確認します。 これにより、実行されるダイアログやそれぞれについての統計情報などの追加情報を取得できます。

## <a name="enabling-telemetry-in-your-bots-dialogs"></a>ボットのダイアログでテレメトリを有効にする

 次の手順に従って、CoreBot の例を更新します。

1.  `MainDialog.cs` で、新しい TelemetryClient フィールドを `MainDialog` クラスに追加し、`IBotTelemetryClient` パラメーターを含めるようにコンストラクターのパラメーター一覧を更新し、それを `AddDialog()` メソッドへの各呼び出しに渡します。


    * パラメーター `IBotTelemetryClient telemetryClient` を MainDialog クラス コンストラクターに追加し、それを `TelemetryClient` フィールドに割り当てます。

        ```csharp
        public MainDialog(IConfiguration configuration, ILogger<MainDialog> logger, IBotTelemetryClient telemetryClient)
            : base(nameof(MainDialog))
        {
            TelemetryClient = telemetryClient;

            ...

        }
        ```

    * `AddDialog` メソッドを使用して各ダイアログを追加するとき、`TelemetryClient` 値を設定します。

        ```cs
        public MainDialog(IConfiguration configuration, ILogger<MainDialog> logger, IBotTelemetryClient telemetryClient)
            : base(nameof(MainDialog))
        {
            ...

            AddDialog(new TextPrompt(nameof(TextPrompt))
            {
                TelemetryClient = telemetryClient,
            });

            ...
        }
        ```

    * ウォーターフォール ダイアログは、他のダイアログとは無関係にイベントを生成します。 ウォーターフォール手順の一覧の後に `TelemetryClient` プロパティを設定します。

        ```csharp
        // The IBotTelemetryClient to the WaterfallDialog
        AddDialog(new WaterfallDialog(
            nameof(WaterfallDialog),
            new WaterfallStep[]
        {
            IntroStepAsync,
            ActStepAsync,
            FinalStepAsync,
        })
        {
            TelemetryClient = telemetryClient,
        });

        ```

2. `DialogExtensions.cs` で、`Run()` メソッドの `dialogSet` オブジェクトの `TelemetryClient` プロパティを設定します。


    ```csharp
    public static async Task Run(this Dialog dialog, ITurnContext turnContext, IStatePropertyAccessor<DialogState> accessor, CancellationToken cancellationToken = default(CancellationToken))
    {
        ...

        dialogSet.TelemetryClient = dialog.TelemetryClient;

        ...
        
    }

    ```

3. `BookingDialog.cs` で、`MainDialog.cs` を更新するときに使用したのと同じプロセスを使用して、このファイルに追加された 4 つのダイアログでテレメトリを有効にします。 BookingDialog クラスに `TelemetryClient` フィールドを追加し、BookingDialog コンストラクターに `IBotTelemetryClient telemetryClient` パラメーターを追加することを忘れないでください。


> [!TIP] 
> CoreBot サンプル コードを更新して作業を進めている場合、問題が発生したときは、[Application Insights サンプル コード](https://aka.ms/csharp-corebot-app-insights-sample)を参照できます。

ボット ダイアログへのテレメトリの追加についてはこれですべてです。この時点でボットを実行した場合は、Application Insights にログが記録されているのを確認できるはずです。ただし、LUIS や QnA Maker などの統合テクノロジがある場合は、さらに `TelemetryClient` をそのコードに追加する必要があります。


## <a name="enabling-telemetry-to-capture-usage-data-from-other-services-like-luis-and-qna-maker"></a>テレメトリが LUIS や QnA Maker などの他のサービスから使用状況データを取り込めるようにする

次に、LUIS サービスでテレメトリ機能を実装します。 LUIS サービスでは、組み込みのテレメトリ ログ記録が利用できるため、LUIS からのテレメトリ データの取得を開始するために実行しなければならない操作はほとんどありません。  QnA Maker 対応ボットでテレメトリを有効にすることを検討している場合は、「[QnAMaker ボットへのテレメトリの追加](bot-builder-telemetry-QnAMaker.md)」を参照してください

このサンプルでは、単にダイアログの場合と同様の方法でテレメトリ クライアントを提供する必要があります。 

1. `LuisHelper.cs` の `ExecuteLuisQuery()` メソッドには _`IBotTelemetryClient telemetryClient`_ パラメーターが必要です。

    ```cs
    public static async Task<BookingDetails> ExecuteLuisQuery(IBotTelemetryClient telemetryClient, IConfiguration configuration, ILogger logger, ITurnContext turnContext, CancellationToken cancellationToken)
    ```

2. `LuisPredictionOptions` クラスを使用すると、LUIS 予測要求に対する省略可能なパラメーターを指定できます。  テレメトリを有効にするには、`LuisHelper.cs` に `luisPredictionOptions` オブジェクトを作成するときに `TelemetryClient` パラメーターを設定する必要があります。

    ```cs
    var luisPredictionOptions = new LuisPredictionOptions()
    {
        TelemetryClient = telemetryClient,
    };

    var recognizer = new LuisRecognizer(luisApplication, luisPredictionOptions);
    ```

3. 最後の手順は、テレメトリ クライアントを `MainDialog.cs` の `ActStepAsync()` メソッドで `ExecuteLuisQuery` への呼び出しに渡すことです。

    ```cs
    await LuisHelper.ExecuteLuisQuery(TelemetryClient, Configuration, Logger, stepContext.Context, cancellationToken)
    ```

これで、テレメトリ データを Application insights に記録する、機能するボットが用意できたはずです。 [Bot Framework Emulator](https://aka.ms/bot-framework-emulator-readme) を使用して、ボットをローカルで実行できます。 ボットの動作には変化は見られませんが、Application Insights に情報が記録されます。 複数のメッセージを送信してボットと対話します。次のセクションでは、Application Insights でテレメトリの結果を確認します。

ボットのテストとデバッグの詳細については、次の記事を参照できます。

 * [ボットのデバッグ](../bot-service-debug-bot.md)
 * [テストとデバッグのガイドライン](bot-builder-testing-debugging.md)
 * [エミュレーターを使用したデバッグ](../bot-service-debug-emulator.md)


## <a name="visualizing-your-telemetry-data-in-application-insights"></a>テレメトリ データを Application Insights で視覚化する
Application Insights は、クラウドとオンプレミスのどちらでホストされているかにかかわらず、ボット アプリケーションの可用性、パフォーマンス、使用状況を監視します。 Azure Monitor の強力なデータ分析プラットフォームを利用し、ユーザーの報告を待つことなく、アプリケーション運用やエラー診断に対する深い洞察を提供します。 Application Insights によって収集されたテレメトリ データを表示するにはいくつかの方法があります。2 つの主要な方法は、クエリとダッシュボードを使用するというものです。 

### <a name="querying-your-telemetry-data-in-application-insights-using-kusto-queries"></a>Kusto クエリを使って Application Insights でテレメトリ データのクエリを実行する
このセクションは、Application Insights でのログ クエリの使用方法を学ぶための開始点として使用してください。 ここでは、2 つの有用なクエリを示し、追加情報を含む他のドキュメントへのリンクを提供します。

データのクエリを実行するには、次のようにします。

1. [Azure portal](https://portal.azure.com) に移動します
2. Application Insights に移動します。 これを行う最も簡単な方法は、 **[監視] > [アプリケーション]** の順にクリックすることであり、そこで見つかります。 
3. Application Insights を開いたら、ナビゲーション バーにある _[ログ (Analytics)]_ をクリックできます。

    ![ログ (Analytics)](media/AppInsights-LogView.png)

4. これにより、クエリ ウィンドウが表示されます。  次のクエリを入力し、 _[実行]_ を選択します。

    ```sql
    customEvents
    | where name=="WaterfallStart"
    | extend DialogId = customDimensions['DialogId']
    | extend InstanceId = tostring(customDimensions['InstanceId'])
    | join kind=leftouter (customEvents | where name=="WaterfallComplete" | extend InstanceId = tostring(customDimensions['InstanceId'])) on InstanceId    
    | summarize starts=countif(name=='WaterfallStart'), completes=countif(name1=='WaterfallComplete') by bin(timestamp, 1d), tostring(DialogId)
    | project Percentage=max_of(0.0, completes * 1.0 / starts), timestamp, tostring(DialogId) 
    | render timechart
    ```
5. これにより、完了まで実行されるウォーターフォール ダイアログの割合が返されます。

    ![ログ (Analytics)](media/AppInsights-Query-PercentCompleteDialog.png)


> [!TIP]
> **[ログ (Analytics)]** ブレードの右上にあるボタンを選択することにより、Application Insights ダッシュボードに任意のクエリをピン留めできます。 ピン留めするダッシュボードを選択するだけで、次回そのダッシュボードにアクセスするとそれが使用可能になります。


## <a name="the-application-insights-dashboard"></a>Application Insights ダッシュボード

Azure で Application Insights リソースを作成するたびに、新しいダッシュボードが自動的に作成されて、関連付けられます。  そのダッシュボードを表示するには、Application Insights ブレードの上部にある、 **[アプリケーション ダッシュボード]** というラベルの付いたボタンを選択します。 

![アプリケーション ダッシュボード リンク](media/Application-Dashboard-Link.png)


あるいは、データを表示するには、Azure portal に移動します。 左側の **[ダッシュボード]** をクリックし、目的のダッシュボードをドロップダウンから選択します。

ここには、ボットのパフォーマンスに関する既定の情報と、ダッシュボードにピン留めしたその他のクエリが表示されます。



## <a name="additional-information"></a>追加情報

* [QnAMaker ボットへのテレメトリの追加](bot-builder-telemetry-qnamaker.md)

* [Application Insights とは何か?](https://aka.ms/appinsights-overview)

* [Application Insights の検索の使用](https://aka.ms/search-in-application-insights)

* [Azure Application Insights を使ってカスタム KPI ダッシュボードを作成する](https://aka.ms/custom-kpi-dashboards-application-insights)


<!--
The easiest way to test is by creating a dashboard using [Azure portal's template deployment page](https://portal.azure.com/#create/Microsoft.Template).
- Click ["Build your own template in the editor"]
- Copy and paste either one of these .json file that is provided to help you create the dashboard:
  - [System Health Dashboard](https://aka.ms/system-health-appinsights)
  - [Conversation Health Dashboard](https://aka.ms/conversation-health-appinsights)
- Click "Save"
- Populate `Basics`: 
   - Subscription: <your test subscription>
   - Resource group: <a test resource group>
   - Location: <such as West US>
- Populate `Settings`:
   - Insights Component Name: <like `core672so2hw`>
   - Insights Component Resource Group: <like `core67`>
   - Dashboard Name:  <like `'ConversationHealth'` or `SystemHealth`>
- Click `I agree to the terms and conditions stated above`
- Click `Purchase`
- Validate
   - Click on [`Resource Groups`](https://ms.portal.azure.com/#blade/HubsExtension/Resources/resourceType/Microsoft.Resources%2Fsubscriptions%2FresourceGroups)
   - Select your Resource Group from above (like `core67`).
   - If you don't see a new Resource, then look at "Deployments" and see if any have failed.
   - Here's what you typically see for failures:
     
```json
{"code":"DeploymentFailed","message":"At least one resource deployment operation failed. Please list deployment operations for details. Please see https://aka.ms/arm-debug-deployment for usage details.","details":[{"code":"BadRequest","message":"{\r\n \"error\": {\r\n \"code\": \"InvalidTemplate\",\r\n \"message\": \"Unable to process template language expressions for resource '/subscriptions/45d8a30e-3363-4e0e-849a-4bb0bbf71a7b/resourceGroups/core67/providers/Microsoft.Portal/dashboards/Bot Analytics Dashboard' at line '34' and column '9'. 'The template parameter 'virtualMachineName' is not found. Please see https://aka.ms/arm-template-structure for usage details.'\"\r\n }\r\n}"}]}
```
-->

<!--
## Additional information

### Customize your telemetry client

If you want to customize your telemetry to log into a separate service, you have to configure the system differently. If using Application Insights to do so, download the package `Microsoft.Bot.Builder.ApplicationInsights` via NuGet, or use npm to install `botbuilder-applicationinsights`. Details on getting the Application Insights keys can be found [here](../bot-service-resources-app-insights-keys.md). Otherwise, include what is necessary for logging to that service, then follow the section below.

**Use Custom Telemetry**
If you want to log telemetry events generated by the Bot Framework into a completely separate system, create a new class derived from the base interface `IBotTelemetryClient` and configure. Then, when adding your telemetry client as above, just inject your custom client. 

```csharp
public void ConfigureServices(IServiceCollection services)
{
    ...
    // Add the telemetry client.
    services.AddSingleton<IBotTelemetryClient, CustomTelemetryClient>();
    ...
}
```

### Add custom logging to your bot

Once the Bot has the new telemetry logging support configured, you can begin adding telemetry to your bot.  The `BotTelemetryClient`(in C#, `IBotTelemetryClient`) has several methods to log distinct types of events.  Choosing the appropriate type of event enables you to take advantage of Application Insights existing reports (if you are using Application Insights).  For general scenarios `TraceEvent` is typically used.  The data logged using `TraceEvent` lands in the `CustomEvent` table in Kusto.

If using a Dialog within your Bot, every Dialog-based object (including Prompts) will contain a new `TelemetryClient` property.  This is the `BotTelemetryClient` that enables you to perform logging.  This is not just a convenience, we'll see later in this article if this property is set, `WaterfallDialogs` will generate events.

### Details of telemetry options

There are three main components available for your bot to log telemetry, and each component has customization available for logging your own events, which are discussed in this section. 

- A  [Bot Framework Middleware component](#telemetry-middleware) (*TelemetryLoggerMiddleware*) that will log when messages are received, sent, updated or deleted. You can override for custom logging.
- [*LuisRecognizer* class.](#telemetry-support-luis)  You can override for custom logging in two ways - per invocation (add/replace properties) or derived classes.
- [*QnAMaker*  class.](#telemetry-qnamaker)  You can override for custom logging in two ways - per invocation (add/replace properties) or derived classes.

All components log using the `IBotTelemetryClient`  (or `BotTelemetryClient` in node.js) interface which can be overridden with a custom implementation.

#### Telemetry Middleware

|C#  | JavaScript |
|:-----|:------------|
|**Microsoft.Bot.Builder.TelemetryLoggerMiddleware** | **botbuilder-core** |

##### Out of box usage

The TelemetryLoggerMiddleware is a Bot Framework component that can be added without modification, and it will perform logging that enables out of the box reports that ship with the Bot Framework SDK. 

```csharp
// Create the telemetry middleware to track conversation events
services.AddSingleton<IMiddleware, TelemetryLoggerMiddleware>();

// Create the Bot Framework Adapter with error handling enabled.
services.AddSingleton<IBotFrameworkHttpAdapter, AdapterWithErrorHandler>();
```

And in our adapter, we would specify the use of middleware:

```csharp
public AdapterWithErrorHandler(ICredentialProvider credentialProvider, ILogger<BotFrameworkHttpAdapter> logger, IMiddleware middleware, ConversationState conversationState = null)
           : base(credentialProvider)
{
    ...
    Use(middleware);
    ...
}
```

##### Adding properties
If you decide to add additional properties, the TelemetryLoggerMiddleware class can be derived.  For example, if you would like to add the property "MyImportantProperty" to the `BotMessageReceived` event.  `BotMessageReceived` is logged when the user sends a message to the bot.  Adding the additional property can be accomplished in the following way:

```csharp
class MyTelemetryMiddleware : TelemetryLoggerMiddleware
{
    ...
    protected override Task OnReceiveActivityAsync(
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

In Startup, we would add the new class:

```csharp
// Create the telemetry middleware to track conversation events
services.AddSingleton<IMiddleware, MyTelemetryMiddleware>();
```

##### Completely replacing properties / Additional event(s)

If you decide to completely replace properties being logged, the `TelemetryLoggerMiddleware` class can be derived (like above when extending properties).   Similarly, logging new events is performed in the same way.

For example, if you would like to completely replace the`BotMessageSend` properties and send multiple events, the following demonstrates how this could be performed:

```csharp
class MyTelemetryMiddleware : TelemetryLoggerMiddleware
{
    ...
    protected override Task OnReceiveActivityAsync(
                  Activity activity,
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
Note: When the standard properties are not logged, it will cause the out of box reports shipped with the product to stop working.

##### Events Logged from Telemetry Middleware
[BotMessageSend](bot-builder-telemetry-reference.md#customevent-botmessagesend)
[BotMessageReceived](bot-builder-telemetry-reference.md#customevent-botmessagereceived)
[BotMessageUpdate](bot-builder-telemetry-reference.md#customevent-botmessageupdate)
[BotMessageDelete](bot-builder-telemetry-reference.md#customevent-botmessagedelete)

#### Telemetry support LUIS 

|C#  | JavaScript |
|:-----|:------------|
| **Microsoft.Bot.Builder.AI.Luis** | **botbuilder-ai** |

##### Out of box usage
The LuisRecognizer is an existing Bot Framework component, and telemetry can be enabled by passing a IBotTelemetryClient interface through `luisOptions`.  You can override the default properties being logged and log new events as required.

During construction of `luisOptions`, an `IBotTelemetryClient` object must be provided for this to work.

```csharp
var luisPredictionOptions = new LuisPredictionOptions()
{
    TelemetryClient = telemetryClient
};
var recognizer = new LuisRecognizer(luisApplication, luisPredictionOptions);
```

##### Adding properties
If you decide to add additional properties, the `LuisRecognizer` class can be derived.  For example, if you would like to add the property "MyImportantProperty" to the `LuisResult` event.  `LuisResult` is logged when a LUIS prediction call is performed.  Adding the additional property can be accomplished in the following way:

```csharp
class MyLuisRecognizer : LuisRecognizer 
{
   ...
   protected override Task OnRecognizerResultAsync(
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

##### Add properties per invocation
Sometimes it's necessary to add additional properties during the invocation:
```csharp
var additionalProperties = new Dictionary<string, string>
{
   { "dialogId", "myDialogId" },
   { "conversationInfo", "myConversationInfo" },
};

var result = await recognizer.RecognizeAsync(turnContext,
     additionalProperties).ConfigureAwait(false);
```

##### Completely replacing properties / Additional event(s)
If you decide to completely replace properties being logged, the `LuisRecognizer` class can be derived (like above when extending properties).   Similarly, logging new events is performed in the same way.

For example, if you would like to completely replace the`LuisResult` properties and send multiple events, the following demonstrates how this could be performed:

```csharp
class MyLuisRecognizer : LuisRecognizer
{
    ...
    protected override Task OnRecognizerResultAsync(
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
Note: When the standard properties are not logged, it will cause the Application Insights out of box reports shipped with the product to stop working.

##### Events Logged from TelemetryLuisRecognizer
[LuisResult](bot-builder-telemetry-reference.md#customevent-luisevent)

### Telemetry QnAMaker

|C#  | JavaScript |
|:-----|:------------|
| **Microsoft.Bot.Builder.AI.QnA** | **botbuilder-ai** |


##### Out of box usage
The QnAMaker class is an existing Bot Framework component that adds two additional constructor parameters which enable logging that enable out of the box reports that ship with the Bot Framework SDK. The new `telemetryClient` references a `IBotTelemetryClient` interface which performs the logging.  

```csharp
var qna = new QnAMaker(endpoint, options, client, 
                       telemetryClient: telemetryClient,
                       logPersonalInformation: true);
```
##### Adding properties 
If you decide to add additional properties, there are two methods of doing this - when properties need to be added during the QnA call to retrieve answers or deriving from the `QnAMaker` class.  

The following demonstrates deriving from the `QnAMaker` class.  The example shows adding the property "MyImportantProperty" to the `QnAMessage` event.  The`QnAMessage` event is logged when a QnA `GetAnswers`call is performed.  In addition, we log a second event "MySecondEvent".

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

##### Adding properties during GetAnswersAsync
If you have properties that need to be added during runtime, the `GetAnswersAsync` method can provide properties and/or metrics to add to the event.

For example, if you want to add a `dialogId` to the event, it can be done like the following:
```csharp
var telemetryProperties = new Dictionary<string, string>
{
   { "dialogId", myDialogId },
};

var results = await qna.GetAnswersAsync(context, opts, telemetryProperties);
```
The `QnaMaker` class provides the capability of overriding properties, including PersonalInfomation properties.

##### Completely replacing properties / Additional event(s)
If you decide to completely replace properties being logged, the `TelemetryQnAMaker` class can be derived (like above when extending properties).   Similarly, logging new events is performed in the same way.

For example, if you would like to completely replace the`QnAMessage` properties, the following demonstrates how this could be performed:

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
Note: When the standard properties are not logged, it will cause the out of box reports shipped with the product to stop working.

##### Events Logged from TelemetryLuisRecognizer
[QnAMessage](bot-builder-telemetry-reference.md#customevent-qnamessage)

### All other events

A full list of events logged for your bot's telemetry can be found on the [telemetry reference page](bot-builder-telemetry-reference.md).

#### Identifiers and Custom Events

When logging events into Application Insights, the events generated contain default properties that you won't have to fill.  For example, `user_id` and `session_id`properties are contained in each Custom Event (generated with the `TraceEvent` API).  In addition, `activitiId`, `activityType` and `channelId` are also added.

> [!NOTE]
> Custom telemetry clients will not be provided these values.


Property |Type | Details
--- | --- | ---
`user_id`| `string` | [ChannelID](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#channel-id) + [From.Id](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#from)
`session_id`| `string`|  [ConversationID](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#conversation)
`customDimensions.activityId`| `string` | [The bot activity ID](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#id)
`customDimensions.activityType` | `string` | [The bot activity type](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#channel-id)
`customDimensions.channelId` | `string` |  [The bot activity channel ID](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#channel-id)
-->