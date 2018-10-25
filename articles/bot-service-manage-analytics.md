---
title: ボットの分析 | Microsoft Docs
description: Bot Framework での分析を利用してボットを向上させるためのデータの収集と分析の使用方法について説明します。
keywords: ボットの分析, アプリケーション分析情報, トラフィック, 待ち時間, 統合, AppInsights
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: abs
ms.date: 12/13/2017
ms.openlocfilehash: 27dc7786554af14a24fc8d65b2f7ee31bc4864ef
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/24/2018
ms.locfileid: "49997925"
---
# <a name="bot-analytics"></a>ボットの分析
Analytics は [Application Insights](/azure/application-insights/app-insights-analytics) の拡張機能です。 Application Insights では、トラフィック、待ち時間、統合などの**サービス レベル**およびインストルメンテーションのデータが提供されます。 Analytics では、ユーザー、メッセージ、チャネル データに関する**会話レベル**のレポートが提供されます。

## <a name="view-analytics-for-a-bot"></a>ボットの分析を表示する
Analytics にアクセスするには、開発者ポータルでボットを開き、**[Analytics]** をクリックします。

データが多すぎる場合は、 [サンプリングを有効にして構成](/azure/application-insights/app-insights-sampling)し、統計的に正しい分析を維持しながら、テレメトリのトラフィックとストレージを減らします。 

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

### <a name="activities"></a>アクティビティ
アクティビティ グラフでは、指定期間内に各チャネルを使用して送受信されたアクティビティの数が追跡されます。

![アクティビティ グラフ](~/media/analytics-activities.png)

* 割合のグラフでは、各チャネル経由で通信されたアクティビティの割合が示されます。
* 折れ線グラフでは、指定期間内に送受信されたアクティビティの数が示されます。
* 折れ線グラフの凡例では、各チャネルを表す線の色と、指定期間中にそのチャネルで送受信されたアクティビティの合計数が示されます。 

## <a name="enable-analytics"></a>分析を有効にする
Application Insights を有効にして構成するまで、Analytics は使用できません。 Application Insights は、有効にするとすぐにデータの収集を始めます。 たとえば、6 か月前から動いているボットに対して 1 週間前に Application Insights を有効にした場、1 週間分のデータが収集されています。
> [!NOTE]
> Analytics には、Azure サブスクリプションと Application Insights [リソース](/azure/application-insights/app-insights-create-new-resource)の両方が必要です。
Application Insights にアクセスするには、[Bot Framework ポータル](https://dev.botframework.com/)でボットを開き、**[Settings]\(設定\)** をクリックします。

1. Application Insights の[リソース](/azure/application-insights/app-insights-create-new-resource)を作成します。
2. ダッシュボードでボットを開きます。 **[Settings]\(設定\)** をクリックして、**[Analytics]\(分析\)** セクションまで下にスクロールします。
3. 情報を入力してボットを Application Insights に接続します。 すべてのフィールドが必須です。

![Insights に接続する](~/media/analytics-enable.png)

### <a name="appinsights-instrumentation-key"></a>AppInsights インストルメンテーション キー
この値を調べるには、Application Insights を開き、**[構成]** > **[プロパティ]** に移動します。

### <a name="appinsights-api-key"></a>AppInsights API キー
Azure App Insights API キーを指定します。 [新しい API キーを生成する](https://dev.applicationinsights.io/documentation/Authorization/API-key-and-App-ID)方法を確認してください。 **読み取り**アクセス許可のみが必要です。

### <a name="appinsights-application-id"></a>AppInsights アプリケーション ID
この値を調べるには、Application Insights を開き、**[構成]** > **[API アクセス]** に移動します。

これらの値を調べる方法について詳しくは、「[Application Insights キー](~/bot-service-resources-app-insights-keys.md)」をご覧ください。

## <a name="additional-resources"></a>その他のリソース
* [Application Insights キー](~/bot-service-resources-app-insights-keys.md)