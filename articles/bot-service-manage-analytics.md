---
title: ボットの分析 | Microsoft Docs
description: Bot Framework での分析を利用してボットを向上させるためのデータの収集と分析の使用方法について説明します。
keywords: ボットの分析, アプリケーション分析情報, トラフィック, 待ち時間, 統合, AppInsights
author: JonathanFingold
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/04/2018
ms.openlocfilehash: 324050c625f5d9666811f63191d783643816104c
ms.sourcegitcommit: a6d02ec4738e7fc90b7108934740e9077667f3c5
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/04/2019
ms.locfileid: "70298690"
---
# <a name="bot-analytics"></a>ボットの分析

Analytics は [Application Insights](/azure/application-insights/app-insights-analytics) の拡張機能です。 Application Insights では、トラフィック、待ち時間、統合などの**サービス レベル**およびインストルメンテーションのデータが提供されます。 Analytics では、ユーザー、メッセージ、チャネル データに関する**会話レベル**のレポートが提供されます。

## <a name="view-analytics-for-a-bot"></a>ボットの分析を表示する

Analytics にアクセスするには、Azure portal 上でボットを開き、 **[分析]** をクリックします。

データが多すぎる場合は、 ボットにリンクされた Application Insights で、[サンプリングを有効にして構成](/azure/application-insights/app-insights-sampling)することができます。 こうすることで、統計的に正しい分析を維持しながら、テレメトリのトラフィックとストレージを削減できます。

### <a name="specify-channel"></a>チャネルを指定する

下のグラフに表示するチャネルを選択します。 チャネルでボットが有効になっていない場合、そのチャネルからはデータを取得できないことに注意してください。

![チャネルを選択する](~/media/analytics-channels.png)

* グラフに含めるチャネルのチェック ボックスをオンにします。
* グラフから削除するチャネルのチェック ボックスをオフにします。

### <a name="specify-time-period"></a>期間を指定する

分析できるのは過去 90 日間のみです。 データの収集は、Application Insights を有効にすると開始されます。

![期間を選択する](~/media/analytics-timepick.png)

ドロップダウン メニューをクリックし、グラフに表示する期間をクリックします。
全体の期間を変更すると、それに応じてグラフの時間増分 (X 軸) が変化することに注意してください。

### <a name="grand-totals"></a>総計

指定した期間内のアクティブ ユーザーおよび送受信されたアクティビティの総数です。
ダッシュ `--` はアクティビティがなかったことを示します。

### <a name="retention"></a>保持

保持では、一度メッセージを送信し、後で戻ってきて別のメッセージを送信したユーザーの数を追跡します。
グラフは、10 日単位でまとめられます。期間を変更しても結果に影響はありません。

![保持グラフ](~/media/analytics-retention.png)

表示できる最新の日付は 2 日前であることに注意してください。つまり、2 日前にメッセージを送信し、昨日 "*戻ってきた*" ユーザーです。

### <a name="user"></a>User

ユーザー グラフでは、指定した期間内に各チャネルを使用してボットにアクセスしたユーザーの数が追跡されます。

![ユーザー グラフ](~/media/analytics-users.png)

* 割合のグラフでは、各チャネルを使用したユーザーの割合が示されます。
* 折れ線グラフでは、特定の時点でボットにアクセスしていたユーザーの数が示されます。
* 折れ線グラフの凡例では、各チャネルを表す色が示され、指定期間中のユーザーの合計数が含まれます。

### <a name="activities"></a>Activities

アクティビティ グラフでは、指定期間内に各チャネルを使用して送受信されたアクティビティの数が追跡されます。

![アクティビティ グラフ](~/media/analytics-activities.png)

* 割合のグラフでは、各チャネル経由で通信されたアクティビティの割合が示されます。
* 折れ線グラフでは、指定期間内に送受信されたアクティビティの数が示されます。
* 折れ線グラフの凡例では、各チャネルを表す線の色と、指定期間中にそのチャネルで送受信されたアクティビティの合計数が示されます。

## <a name="enable-analytics"></a>分析を有効にする

Application Insights を有効にして構成するまで、Analytics は使用できません。 Application Insights は、有効にするとすぐにデータの収集を始めます。 たとえば、6 か月前から動いているボットに対して 1 週間前に Application Insights を有効にした場、1 週間分のデータが収集されています。

> [!NOTE]
> Analytics には、Azure サブスクリプションと Application Insights [リソース](/azure/application-insights/app-insights-create-new-resource)の両方が必要です。
Application Insights にアクセスするには、[Azure Portal](https://portal.azure.com/) でボットを開き、 **[Settings]\(設定\)** をクリックします。

ボット リソースを作成するときに Application Insights を追加できます。

後で Application Insights リソースを作成して、ボットに接続することもできます。

1. Application Insights の[リソース](/azure/application-insights/app-insights-create-new-resource)を作成します。
2. ダッシュボードでボットを開きます。 **[Settings]\(設定\)** をクリックして、 **[Analytics]\(分析\)** セクションまで下にスクロールします。
3. 情報を入力してボットを Application Insights に接続します。 すべてのフィールドが必須です。

![Insights に接続する](~/media/analytics-enable.png)

<!--Snip: As of 12/04/2018, parts of this appear to be out of date. However, ~/bot-service-resources-app-insights-keys.md appears to be up to date.

### AppInsights Instrumentation Key

To find this value, open the Application Insights resource for your bot and navigate to **Configure** > **Properties**.

### AppInsights API key

Provide an Azure App Insights API key. Learn how to [generate a new API key](https://dev.applicationinsights.io/documentation/Authorization/API-key-and-App-ID). Only **Read** permission is required.

### AppInsights Application ID

To find this value, open Application Insights and navigate to **Configure** > **API Access**.

/Snip-->

これらの値を調べる方法について詳しくは、「[Application Insights キー](~/bot-service-resources-app-insights-keys.md)」をご覧ください。

## <a name="additional-resources"></a>その他のリソース
* [Application Insights キー](~/bot-service-resources-app-insights-keys.md)