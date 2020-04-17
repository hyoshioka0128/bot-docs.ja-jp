---
title: QnA ボットへのテレメトリの追加 - Bot Service
description: 新しいテレメトリ機能を QnA Maker 対応ボットに統合する方法について説明します。
keywords: テレメトリ, appinsights, Application Insights, ボットの監視, QnA Maker
author: WashingtonKayaker
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 07/31/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: d73bd8c26dde8826b145108268417a24840ed83e
ms.sourcegitcommit: 9d77f3aff9521d819e88efd0fbd19d469b9919e7
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/16/2020
ms.locfileid: "75791244"
---
# <a name="add-telemetry-to-your-qnamaker-bot"></a>QnAMaker ボットへのテレメトリの追加

[!INCLUDE[applies-to](../includes/applies-to.md)]


Bot Framework SDK のバージョン 4.2 にテレメトリのログ記録が追加されました。  これにより、ボット アプリケーションは [Application Insights](https://aka.ms/appinsights-overview) などのテレメトリ サービスにイベント データを送信できます。 テレメトリは、どの機能が最も使用されているかを示すことによりボットの分析情報を提供し、不要な動作を検出し、可用性、パフォーマンス、および使用状況を可視化します。

QnA Maker 対応ボットでテレメトリのログ記録を有効にする 2 つの新しいコンポーネントが Bot Framework SDK に追加されました。`TelemetryLoggerMiddleware` と `QnAMaker` クラスです。 `TelemetryLoggerMiddleware` は、メッセージが受信、送信、更新、または削除されるたびにログを記録するミドルウェア コンポーネントであり、"QnAMaker" クラスは、テレメトリ機能を拡張するカスタムのログ記録を提供します。

この記事では、以下について説明します。

* ボットにテレメトリを実装するために必要なコード 

* すぐに使用できる QnA ログ記録と、標準のイベント プロパティを使用するレポートを有効にするために必要なコード。 

* 幅広いレポートのニーズに応えるために、SDK の既定のイベント プロパティを変更または拡張する方法。


## <a name="prerequisites"></a>前提条件

* [QnA Maker サンプル コード](https://aka.ms/cs-qna)

* [Microsoft Azure](https://portal.azure.com/) のサブスクリプション。

* [Application Insights のキー](../bot-service-resources-app-insights-keys.md)

* [QnA Maker](https://qnamaker.ai/) について理解していると役に立ちます。

* [QnA Maker](https://aka.ms/create-qna-maker) アカウント。

* 公開されている QnA Maker ナレッジ ベース。 それがない場合は、[KB の作成と KB から質問に回答](https://aka.ms/create-publish-query-in-portal)に関するチュートリアルの手順に従って、質問と回答が含まれる QnA Maker ナレッジ ベースを作成します。

> [!NOTE]
> この記事では、テレメトリを組み込むために必要な手順について、[QnA Maker サンプル コード](https://aka.ms/cs-qna)に基づいて説明します。 

## <a name="wiring-up-telemetry-in-your-qna-maker-bot"></a>QnA Maker ボットでテレメトリを接続する

[QnA Maker サンプル アプリ](https://aka.ms/cs-qna)を出発点に、QnA サービスを使用してテレメトリをボットに統合するために必要なコードを追加します。 これにより、Application Insights は要求の追跡を開始できるようになります。

1. Visual Studio で [QnA Maker サンプル アプリ](https://aka.ms/cs-qna)を開きます

2. `Microsoft.Bot.Builder.Integration.ApplicationInsights.Core ` NuGet パッケージを追加します。 NuGet の使用方法について詳しくは、「[Visual Studio にパッケージをインストールして管理する](https://aka.ms/install-manage-packages-vs)」を参照してください。

3. 次のステートメントを `Startup.cs` に含めます。
    ```csharp
    using Microsoft.ApplicationInsights.Extensibility;
    using Microsoft.Bot.Builder.ApplicationInsights;
    using Microsoft.Bot.Builder.Integration.ApplicationInsights.Core;
    ```

    > [!NOTE] 
    > QnA Maker サンプル コードを更新して作業を進めている場合は、`Microsoft.Bot.Builder.Integration.AspNet.Core` の using ステートメントが既に QnA Maker サンプルにあることに気付きます。

4. `Startup.cs` で、`ConfigureServices()` メソッドに次のコードを追加します。 こうすることで、[依存関係の挿入 (DI)](https://aka.ms/asp.net-core-dependency-interjection) によりテレメトリ サービスをボットで使用できるようになります。
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
    
    > [!NOTE] 
    > QnA Maker サンプル コードを更新して作業を進めている場合は、`services.AddSingleton<IBotFrameworkHttpAdapter, AdapterWithErrorHandler>();` が既に存在していることに気付きます。

5. `ConfigureServices()` メソッドに追加されたミドルウェア コードを使用するようにアダプターに指示します。 `AdapterWithErrorHandler.cs` を開き、`IMiddleware middleware` をコンストラクターのパラメーター リストに追加します。 `Use(middleware);` ステートメントをコンストラクターの最後の行として追加します。
    ```csharp
    public AdapterWithErrorHandler(ICredentialProvider credentialProvider, ILogger<BotFrameworkHttpAdapter> logger, IMiddleware middleware, ConversationState conversationState = null)
            : base(credentialProvider)
    {
        ...

        Use(middleware);
    }
    ```

6. `appsettings.json` ファイルで、Application Insights のインストルメンテーション キーを追加します。 `appsettings.json` ファイルには、ボットの実行中に使用される外部サービスに関するメタデータが格納されます。 たとえば、CosmosDB、Application Insights、QnA Maker サービスの接続とメタデータがそこに保存されています。 `appsettings.json` ファイルへの追加は、次の形式にする必要があります。

    ```json
    {
        "MicrosoftAppId": "",
        "MicrosoftAppPassword": "",
        "QnAKnowledgebaseId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
        "QnAEndpointKey": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
        "QnAEndpointHostName": "https://xxxxxxxx.azurewebsites.net/qnamaker",
        "ApplicationInsights": {
            "InstrumentationKey": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
        }
    }
    ```
   > [!Note] 
   > 
   > * _Application Insights のインストルメンテーション キー_の取得の詳細については、「[Application Insights キー](../bot-service-resources-app-insights-keys.md)」の記事をご覧ください。
    >
    > * [QnA Maker アカウント](https://aka.ms/create-qna-maker)は既にお持ちのはずです。必要に応じて、QnA ナレッジ ベース ID、エンドポイント キー、ホスト名の値の取得方法について[こちら](https://aka.ms/bot-framework-emulator-qna-keys)を参照してください。
    > 


この時点で、Application Insights を使用してテレメトリを有効にする準備作業が完了しました。  ボット エミュレーターを使用してボットをローカルで実行した後に、Application Insights にアクセスして、応答時間、アプリ全体の正常性、一般的な実行情報などのログ記録を確認できます。 

> [!TIP] 
> アクティビティ イベントおよび個人情報のログ記録の有効化または無効化の詳細については、「[ボットへのテレメトリの追加](bot-builder-telemetry.md#enabling-or-disabling-activity-logging)」を参照してください。

次に、テレメトリ機能を QnA Maker サービスに追加するために含める必要があるものを確認します。 


## <a name="enabling-telemetry-to-capture-usage-data-from-the-qna-maker-service"></a>テレメトリが QnA Maker サービスから使用状況データを取り込めるようにする

QnA Maker サービスでは、組み込みのテレメトリ ログ記録が利用できるため、QnA Maker からのテレメトリ データの取得を開始するために実行しなければならない操作はほとんどありません。  まず、テレメトリを QnA Maker コードに組み込んで、組み込みのテレメトリのログ記録を有効にする方法について説明します。次に、レポートのさまざまなニーズを満たすために、既存のイベント データのプロパティを置き換える、またはプロパティを追加する方法について説明します。

### <a name="enabling-default-qna-logging"></a>既定の QnA ログ記録の有効化

1. `IBotTelemetryClient` 型の private の読み取り専用フィールドを `QnABot.cs` の `QnABot` クラスに作成します。

    ```cs
    public class QnABot : ActivityHandler
        {
            private readonly IBotTelemetryClient _telemetryClient;
            ...
   }
    ```

2. `QnABot.cs` の `QnABot` クラスのコンストラクターに `IBotTelemetryClient` パラメーターを追加し、前の手順で作成した private フィールドにその値を割り当てます。

    ```cs
    public QnABot(IConfiguration configuration, ILogger<QnABot> logger, IHttpClientFactory httpClientFactory, IBotTelemetryClient telemetryClient)
    {
        ...
        _telemetryClient = telemetryClient;
    }
    ```

3. _`telemetryClient`_ パラメーターは、`QnABot.cs` で新しい QnAMaker オブジェクトをインスタンス化するときに必要です。

    ```cs
    var qnaMaker = new QnAMaker(new QnAMakerEndpoint
                {
                    KnowledgeBaseId = _configuration["QnAKnowledgebaseId"],
                    EndpointKey = _configuration["QnAEndpointKey"],
                    Host = _configuration["QnAEndpointHostName"]
                },
                null,
                httpClient,
                _telemetryClient);
    ```

    > [!TIP] 
    > `_configuration` エントリで使用するプロパティ名が AppSettings.json ファイルで使用したプロパティ名と一致し、 https://www.qnamaker.ai/Home/MyServices: で _[コードの表示]_ ボタンを選択するとそれらのプロパティの値が取得されることを確認してください。

        
    ![AppSettings](media/AppSettings.json-QnAMaker.png)

#### <a name="viewing-telemetry-data-logged-from-the-qna-maker-default-entries"></a>QnA Maker の既定のエントリからログに記録されたテレメトリ データを表示する
ボット エミュレーターでボットを実行した後、次の手順を実行すると、QnA Maker ボットの使用状況の結果を Application Insights で表示できます。

1. [Azure portal](https://portal.azure.com/) に移動します

2. __[Monitor] (監視) > [アプリケーション]__ をクリックして Application Insights に移動します。

3. Application Insights を開いたら、次に示すように、ナビゲーション バーにある _[Logs (Analytics)]\(ログ (分析)\)_ をクリックします。

    ![Log Analytics](media/AppInsights-LogView-QnaBot.png)

4. 次の Kusto クエリを入力して _[実行]_ を選択します

    ```SQL
    customEvents
    | where name == 'QnaMessage'
    | extend answer = tostring(customDimensions.answer)
    | summarize count() by answer
    ```
5. このページはブラウザーで開いたままにしておきます。新しいカスタム プロパティを追加した後、このページに戻ります。

> [!TIP]
> Azure Monitor でログ クエリの記述に使用する Kusto クエリ言語はあまり知らないが SQL クエリ言語には詳しい方には、[SQL と Azure Monitor ログ クエリの対応早見表](https://aka.ms/azureMonitor-SQL-cheatsheet)が役立つことがあります。 

### <a name="modifying-or-extending-the-default-event-properties"></a>既定のイベント プロパティの変更または拡張
`QnAMaker` クラスで定義されていないプロパティが必要な場合、2 つの対処法がありますが、どちらの場合も `QnAMaker` クラスから派生した独自のクラスを作成する必要があります。 最初の方法では、後出の「[プロパティの追加](#adding-properties)」で説明しているように、既存の `QnAMessage` イベントにプロパティを追加します。 2 番目の方法では、「[カスタム プロパティを持つ新しいイベントの追加](#adding-new-events-with-custom-properties)」で説明しているように、プロパティの追加が可能な新しいイベントを作成します。  

> [!Note]
> `QnAMessage` イベントは Bot Framework SDK に含まれており、Application Insights のログに記録される、すぐに使用できるイベント プロパティをすべて提供します。



#### <a name="adding-properties"></a>プロパティの追加 

次に示すのは、`QnAMaker` クラスから派生する方法の例です。  この例では、"MyImportantProperty" プロパティを `QnAMessage` イベントに追加しています。  `QnAMessage` イベントは、QnA の [GetAnswers](https://aka.ms/namespace-QnAMaker-GetAnswersAsync) 呼び出しが実行されるたびにログに記録されます。  

カスタム プロパティの追加方法を習得したら、新しいカスタム イベントを作成してプロパティをそれに関連付ける方法を学び、Bot Framework エミュレーターを使用してローカルでボットを実行し、Application Insights のログに記録された内容を Kusto クエリ言語を使用して確認します。

1. `QnAMaker` クラスを継承する `MyQnAMaker` という名前の新しいクラスを `Microsoft.BotBuilderSamples` 名前空間に作成し、`MyQnAMaker.cs` という名前で保存します。 `QnAMaker` クラスを継承するには、`Microsoft.Bot.Builder.AI.QnA` の using ステートメントを追加する必要があります。 コードは次のようになります。


    ```cs
    using Microsoft.Bot.Builder.AI.QnA;

    namespace Microsoft.BotBuilderSamples
    {
        public class MyQnAMaker : QnAMaker
        {

        }
    }
    ```
2. クラスのコンストラクターを `MyQnAMaker` に追加します。 コンストラクターの `System.Net.Http` および `Microsoft.Bot.Builder` パラメーターに対して 2 つの using ステートメントが追加で必要になることに注意してください。

    ```cs
    ...
    using Microsoft.Bot.Builder.AI.QnA;
    using System.Net.Http;
    using Microsoft.Bot.Builder;

    namespace Microsoft.BotBuilderSamples
    {
        public class MyQnAMaker : QnAMaker
        {
            public MyQnAMaker(
                QnAMakerEndpoint endpoint,
                QnAMakerOptions options = null,
                HttpClient httpClient = null,
                IBotTelemetryClient telemetryClient = null,
                bool logPersonalInformation = false)
                : base(endpoint, options, httpClient, telemetryClient, logPersonalInformation)
            {

            }
        } 
    }  
    ```
3. コンストラクターの後に、新しいプロパティを QnAMessage イベントに追加し、ステートメント `System.Collections.Generic`、`System.Threading`、および `System.Threading.Tasks` をインクルードします。

    ```cs
    using Microsoft.Bot.Builder.AI.QnA;
    using System.Net.Http;
    using Microsoft.Bot.Builder;
    using System.Collections.Generic;
    using System.Threading;
    using System.Threading.Tasks;
 
    namespace Microsoft.BotBuilderSamples
    {
            public class MyQnAMaker : QnAMaker
            {
            ...

            protected override async Task OnQnaResultsAsync(
                                QueryResult[] queryResults,
                                Microsoft.Bot.Builder.ITurnContext turnContext,
                                Dictionary<string, string> telemetryProperties = null,
                                Dictionary<string, double> telemetryMetrics = null,
                                CancellationToken cancellationToken = default(CancellationToken))
            {
                var eventData = await FillQnAEventAsync(
                                        queryResults,
                                        turnContext,
                                        telemetryProperties,
                                        telemetryMetrics,
                                        cancellationToken)
                                    .ConfigureAwait(false);

                // Add new property
                eventData.Properties.Add("MyImportantProperty", "myImportantValue");

                // Log QnAMessage event
                TelemetryClient.TrackEvent(
                                QnATelemetryConstants.QnaMsgEvent,
                                eventData.Properties,
                                eventData.Metrics
                                );
            }

        } 
    }    
    ```

4. 新しいクラスを使用するようにボットを変更します。`QnABot.cs` で、`QnAMaker` オブジェクトを作成する代わりに `MyQnAMaker` オブジェクトを作成します。

    ```cs
    var qnaMaker = new MyQnAMaker(new QnAMakerEndpoint
                {
                    KnowledgeBaseId = _configuration["QnAKnowledgebaseId"],
                    EndpointKey = _configuration["QnAEndpointKey"],
                    Host = _configuration["QnAEndpointHostName"]
                },
                null,
                httpClient,
                _telemetryClient);
    ```

##### <a name="viewing-telemetry-data-logged-from-the-new-property-_myimportantproperty_"></a>新しいプロパティ _MyImportantProperty_ からログに記録されたテレメトリ データの表示
エミュレーターでボットを実行した後、次の手順を実行して Application Insights で結果を表示できます。

1. _[Logs (Analytics)]\(ログ (分析)\)_ ビューがアクティブになっているブラウザーに戻ります。

2. 次の Kusto クエリを入力して _[実行]_ を選択します。  これにより、新しいプロパティが実行された回数が表示されます。

    ```SQL
    customEvents
    | where name == 'QnaMessage'
    | extend MyImportantProperty = tostring(customDimensions.MyImportantProperty)
    | summarize count() by MyImportantProperty
    ```

3. 回数の代わりに詳細を表示するには、最後の行を削除してクエリを再実行します。

    ```SQL
    customEvents
    | where name == 'QnaMessage'
    | extend MyImportantProperty = tostring(customDimensions.MyImportantProperty)
    ```
### <a name="adding-new-events-with-custom-properties"></a>カスタム プロパティを持つ新しいイベントの追加
`QnaMessage` 以外のイベントにデータのログを記録する必要がある場合、独自のプロパティを持つ独自のカスタム イベントを作成することができます。  これを行うために、次のように `MyQnAMaker` クラスの最後にコードを追加します。

```CS
public class MyQnAMaker : QnAMaker
{
    ...

    // Create second event.
    var secondEventProperties = new Dictionary<string, string>();

    // Create new property for the second event.
    secondEventProperties.Add(
                        "MyImportantProperty2",
                        "myImportantValue2");

    // Log secondEventProperties event
    TelemetryClient.TrackEvent(
                    "MySecondEvent",
                    secondEventProperties);

} 
```                            
## <a name="the-application-insights-dashboard"></a>Application Insights ダッシュボード

Azure で Application Insights リソースを作成するたびに、新しいダッシュボードが自動的に作成されて、関連付けられます。  そのダッシュボードを表示するには、Application Insights ブレードの上部にある、 **[アプリケーション ダッシュボード]** というラベルの付いたボタンを選択します。 

![アプリケーション ダッシュボード リンク](media/Application-Dashboard-Link.png)


あるいは、データを表示するには、Azure portal に移動します。 左側の **[ダッシュボード]** をクリックし、目的のダッシュボードをドロップダウンから選択します。

ここには、ボットのパフォーマンスに関する既定の情報と、ダッシュボードにピン留めしたその他のクエリが表示されます。


## <a name="additional-information"></a>追加情報

* [ボットへのテレメトリの追加](bot-builder-telemetry.md)

* [Application Insights とは何か?](https://aka.ms/appinsights-overview)

* [Application Insights の検索の使用](https://aka.ms/search-in-application-insights)

* [Azure Application Insights を使ってカスタム KPI ダッシュボードを作成する](https://aka.ms/custom-kpi-dashboards-application-insights)
