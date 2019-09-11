---
title: ボットの HTTP 500 エラーのトラブルシューティング | Microsoft Docs
description: デプロイされたボットでの HTTP 500 エラーを解決する方法。
keywords: トラブルシューティング, HTTP 500 エラー, 問題。
author: jonathanFingold
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 4/30/2019
ms.openlocfilehash: 3dcb22c2310f8c686f02fae27617061681910d01
ms.sourcegitcommit: a6d02ec4738e7fc90b7108934740e9077667f3c5
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/04/2019
ms.locfileid: "70298607"
---
# <a name="troubleshoot-http-500-errors"></a>HTTP 500 エラーのトラブルシューティング

500 エラーを解決するための最初の手順は、Application Insights を有効にすることです。

<!-- TODO: Add links back in once there's a fresh AppInsights sample.
The luis-with-appinsights ([C# sample](https://aka.ms/cs-luis-with-appinsights-sample) / [JS sample](https://aka.ms/js-luis-with-appinsights-sample)) and qna-with-appinsights ([C# sample](https://aka.ms/qna-with-appinsights) / [JS sample](https://aka.ms/js-qna-with-appinsights-sample)) samples demonstrate bots that support Azure Application Insights.
-->
Application Insights を既存のボットに追加する方法については、[会話分析テレメトリ](https://aka.ms/botframeworkanalytics)に関する記事を参照してください。

## <a name="enable-application-insights-on-aspnet"></a>ASP.NET で Application Insights を有効にする

Application Insights の基本的なサポートについては、「[Set up Application Insights for your ASP.NET website (ASP.NET Web サイトに合わせて Application Insights を設定する)](https://docs.microsoft.com/azure/application-insights/app-insights-asp-net)」を参照してください。 Bot Framework (v4.2 以降) には、追加レベルの Application Insights テレメトリが用意されていますが、HTTP 500 エラーの診断には必要ありません。

## <a name="enable-application-insights-on-nodejs"></a>Node.js で Application Insights を有効にする

Application Insights の基本的なサポートについては、[Application Insights を使用して Node.js サービスとアプリを監視する](https://docs.microsoft.com/azure/azure-monitor/learn/nodejs-quick-start)方法に関する記事を参照してください。 Bot Framework (v4.2 以降) には、追加レベルの Application Insights テレメトリが用意されていますが、HTTP 500 エラーの診断には必要ありません。

## <a name="query-for-exceptions"></a>例外のクエリ

HTTP 状態コード 500 エラーを分析する最も簡単な方法は、例外から始めることです。

次のクエリで、最新の例外がわかります。

```sql
exceptions
| order by timestamp desc
| project timestamp, operation_Id, appName
```

最初のクエリから、いくつかの操作 ID を選択し、さらに詳しい情報を探します。

```sql
let my_operation_id = "d298f1385197fd438b520e617d58f4fb";
let union_all = () {
    union
    (traces | where operation_Id == my_operation_id),
    (customEvents | where operation_Id == my_operation_id),
    (requests | where operation_Id == my_operation_id),
    (dependencies | where operation_Id  == my_operation_id),
    (exceptions | where operation_Id == my_operation_id)
};

union_all
    | order by timestamp desc
```

`exceptions` のみの場合は、詳細を分析し、それらがコード内の行に対応しているかどうかを確認します。 Channel Connector (`Microsoft.Bot.ChannelConnector`) からの例外のみが表示される場合は、「[Application Insights イベントがない](#no-application-insights-events)」を参照して、Application Insights が正しくセットアップされ、コードによってイベントがログに記録されていることを確認します。

## <a name="no-application-insights-events"></a>Application Insights イベントがない

500 エラーを受け取り、それ以外にボットからの Application Insights 内のイベントがない場合は、以下を確認します。

### <a name="ensure-bot-runs-locally"></a>ローカルでボットが動作することを確認する

まずエミュレーターを使用して、ボットがローカルで動作することを確認します。

### <a name="ensure-configuration-files-are-being-copied-net-only"></a>構成ファイルがコピーされていることを確認する (.NET のみ)

デプロイ プロセス中に、`appsettings.json` と他の構成ファイルすべてが正しくパッケージ化されていることを確認します。

#### <a name="application-assemblies"></a>アプリケーション アセンブリ

デプロイ プロセス中に、Application Insights アセンブリが正しくパッケージ化されていることを確認します。

- Microsoft.ApplicationInsights
- Microsoft.ApplicationInsights.TraceListener
- Microsoft.AI.Web
- Microsoft.AI.WebServer
- Microsoft.AI.ServeTelemetryChannel
- Microsoft.AI.PerfCounterCollector
- Microsoft.AI.DependencyCollector
- Microsoft.AI.Agent.Intercept

デプロイ プロセス中に、`appsettings.json` と他の構成ファイルすべてが正しくパッケージ化されていることを確認します。

#### <a name="appsettingsjson"></a>appsettings.json

`appsettings.json` ファイル内で、インストルメンテーション キーが設定されていることを確認します。

## <a name="aspnet-web-apitabdotnetwebapi"></a>[ASP.NET Web API](#tab/dotnetwebapi)

```json
{
    "Logging": {
        "IncludeScopes": false,
        "LogLevel": {
            "Default": "Debug",
            "System": "Information",
            "Microsoft": "Information"
        },
        "Console": {
            "IncludeScopes": "true"
        }
    }
}
```

## <a name="aspnet-coretabdotnetcore"></a>[ASP.NET Core](#tab/dotnetcore)

```json
{
    "ApplicationInsights": {
        "InstrumentationKey": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
    }
}
```

---

### <a name="verify-config-file"></a>構成ファイルを確認する

構成ファイルに Application Insights キーが含まれていることを確認します。

```json
{
    "ApplicationInsights": {
        "type": "appInsights",
        "tenantId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
        "subscriptionId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
        "resourceGroup": "my resource group",
        "name": "my appinsights name",
        "serviceName": "my service name",
        "instrumentationKey": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
        "applicationId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
        "apiKeys": {},
        "id": ""
    }
},
```

### <a name="check-logs"></a>ログを確認する

Bot ASP.Net と Node からは、検査可能なログがサーバー レベルで発行されます。

#### <a name="set-up-a-browser-to-watch-your-logs"></a>ログを監視できるようにブラウザーを設定する

1. [Azure Portal](http://portal.azure.com/) でボットを開きます。
1. **[アプリ サービスの設定]/[All App service settings]\(すべてのアプリ サービス設定\)** ページを開き、すべてのサービス設定を確認します。
1. アプリ サービスの **[監視]/[診断ログ]** ページを開きます。
   - **[アプリケーション ログ (ファイル システム)]** が有効であることを確認します。 この設定を変更する場合は、必ず **[保存]** をクリックします。
1. **[監視/ログ ストリーム]** ページに切り替えます。
   - **[Web サーバー ログ]** を選択し、接続されていることを示すメッセージが表示されることを確認します。 次のようになります。

     ```bash
     Connecting...
     2018-11-14T17:24:51  Welcome, you are now connected to log-streaming service.
     ```

     このウィンドウを開いたままにしておきます。

#### <a name="set-up-browser-to-restart-your-bot-service"></a>ボット サービスを再起動するようにブラウザーを設定する

1. 別のブラウザーを使用して、Azure Portal でボットを開きます。
1. **[アプリ サービスの設定]/[All App service settings]\(すべてのアプリ サービス設定\)** ページを開き、すべてのサービス設定を確認します。
1. アプリ サービスの **[概要]** ページに切り替え、 **[再起動]** をクリックします。
   - 確認を求められたら、 **[はい]** を選択します。
1. 最初のブラウザー ウィンドウに戻り、ログを確認します。
1. 新しいログを受け取っていることを確認します。
   - アクティビティがない場合は、ボットを再デプロイします。
   - 次に、 **[アプリケーション ログ]** ページに切り替え、エラーがないか確認します。
