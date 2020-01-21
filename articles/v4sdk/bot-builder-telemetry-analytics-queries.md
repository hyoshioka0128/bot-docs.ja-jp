---
title: ボットのテレメトリ データを分析する - Bot Service
description: Kusto クエリを使用してボットの動作を分析する方法について説明します。
keywords: テレメトリ, AppInsights, ボットの監視, Kusto, クエリ
author: WashingtonKayaker
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 01/10/2020
ms.openlocfilehash: a1762e79ab1524f05818d546d04c2e8a1df5fcdd
ms.sourcegitcommit: caaf394017dbdb1cfaba32e2d0a1e32c5ab71792
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/10/2020
ms.locfileid: "75869832"
---
# <a name="analyze-your-bots-telemetry-data"></a>ボットのテレメトリ データを分析する 

## <a name="analyzing-bot-behavior"></a>ボットの動作の分析

次のクエリ集は、ボットの動作を分析するために使用できます。 これらを使うと、[Azure Monitor Log Analytics](https://aka.ms/log-analytics-azure-monitor) でカスタム クエリを作成できるほか、監視ダッシュボードと [Power BI](https://aka.ms/power-bi-overview) の視覚化ダッシュボードを作成できます。

## <a name="prerequisites"></a>前提条件
次の概念に関する基礎知識を持っていると役に立ちます。

* [Kusto クエリ](https://aka.ms/Kusto-query-overview)

* Azure portal で [Log Analytics](https://aka.ms/azure-monitor-log-queries-get-started) を使用して Azure Monitor ログ クエリを記述する方法

* Azure Monitor の[ログ クエリ](https://aka.ms/azure-monitor-log-queries-get-started)の基本概念

## <a name="dashboards"></a>ダッシュボード
Azure ダッシュボードには、お客様のクエリから生成された情報を表示および共有する優れた方法が用意されています。  カスタム ダッシュボードを構築し、自分がダッシュボードに追加するタイルにクエリを関連付けることで、ボットのアクティビティを監視しやすくなります。 ダッシュボードの詳細と、クエリを関連付ける方法については、「[Log Analytics データのダッシュボードを作成して共有する](https://aka.ms/log-analytics-create-share-dashboards)」を参照してください。 この記事の残りの部分では、ボットの動作の監視に役立つクエリ サンプルをいくつか紹介します。  

## <a name="example-kusto-queries"></a>Kusto クエリのサンプル

> [!NOTE]
> この記事のすべてのクエリについて、期間、チャネル、ロケールなどのさまざまなディメンションをピボットすることをお勧めします。

### <a name="number-of-users-per-period"></a>期間あたりのユーザー数
このサンプルでは、過去 14 日間において、ボットと対話した個別のユーザー数を日別に示す折れ線グラフが得られます。  期間は、`queryStartDate`、`queryEndDate`、`interval` の各変数に異なる値を割り当てることで簡単に変更できます。

> [!IMPORTANT]
> このクエリで一意のユーザーの数が正確に得られるのは、各ユーザーが認証を受けている場合のみです。また、チャネル機能によって結果が異なることがあります。

```Kusto
// number of users per period
let queryStartDate = ago(14d);
let queryEndDate = now();
let groupByInterval = 1d;
customEvents 
| where timestamp > queryStartDate
| where timestamp < queryEndDate
| summarize uc=dcount(user_Id) by bin(timestamp, groupByInterval)
| render timechart
```

> [!TIP]
> Kusto の [summarize 演算子](https://aka.ms/kusto-summarize-operator)は、入力テーブルの内容を集計したテーブルを生成するために使用されます。
>
> [bin](https://docs.microsoft.com/azure/kusto/query/binfunction) 関数は Kusto のスカラー関数であり、`summarize operator` と組み合わせて使用すると、クエリ結果が指定値でグループ分けされます。 上のサンプルでは日によってグループ分けしていますが、Kusto では h (時間)、m (分)、s (秒)、ms (ミリ秒)、microsecond (マイクロ秒) を使用することも可能です。
>
> [render 演算子](https://aka.ms/kusto-query-render-operator?pivots=Kusto)を使用すると、さまざまなグラフを簡単にレンダリングできます。たとえば、x 軸に日時をとり、y 軸に任意の数値列を使用できる折れ線グラフである _timechart_ などがあります。 自分のデータに指定期間の一部が含まれていない場合でも、x 軸上の間隔は自動的に適切に調整されます。  render ステートメントを使用しない場合の既定値は `table` です。

#### <a name="sample-query-results"></a>サンプル クエリの結果

<!-- 

OPEN ISSUE: Is it possible to define a more descriptive title in the legend when drawing the graph (vs. “Count”)?  Or overall title on top or underneath?  

Use the render PropertyName parameter:  title, xtitle, ytitle,, legend

-->

![期間あたりのユーザー数](./media/userscount.png)

<!--  

OPEN ISSUE 1:

    "Days with little interaction may indicate service health issues"

    This is an interesting statement – I’m not sure if there’s a way to overlay service health metrics over these metrics.  It should all be in App Insights.

OPEN ISSUE 2:

    Gary Pretty: Overlap for activity per period - Looks like we have a large overlap between ‘Activity per period’ and ‘Activity per user per period’ – given that activity per user is only really useful in some scenarios, such as authenticated, I think it might simplify things to update the ‘Activity per period’ query / instructions to include a note as to how to filter down per user?   Not sure about this one myself, but thought it was worth bringing up.

    < I agree with you.  One approach might be to include both in the same section, with the ‘Activity per period’ as the primary with a note about what might be considered more of a ‘special case’ (I will need input on more details of where this is most applicable – as I’ve commented in a previous email). >


QUESTION: What changes are required?

-->

### <a name="activity-per-period"></a>期間あたりのアクティビティ
このサンプルでは、会話、ダイアログ、またはメッセージの件数という目的のディメンション別に、過去 14 日間における 1 日あたりのアクティビティ量を測定する方法を示します。 期間は、`querystartdate`、`queryEndDate`、`interval` の各変数に異なる値を割り当てることで簡単に変更できます。 目的のディメンションは、次のサンプルの `extend` 句で指定されます。`metric` は、_InstanceId_、_DialogId_、_ActivityId_ のいずれかに設定できます。

*metric* に、表示したいディメンションを設定します。
  * *InstanceId* では、[会話](https://aka.ms/bot-builder-conversations)の件数を測定します
  * *DialogId* では、[ダイアログ](https://aka.ms/bot-builder-concept-dialog)の数を測定します
  * *ActivityId* では、[メッセージ](https://aka.ms/bot-rest-create-messages)の数を測定します

```Kusto
// measures the number of activity's (conversations, dialogs, messages) per period 
let queryStartDate = ago(14d);
let queryEndDate = now();
let groupByInterval = 1d;
customEvents 
| where timestamp > queryStartDate
| where timestamp < queryEndDate
| extend InstanceId = tostring(customDimensions['<InstanceId>'])
| extend DialogId = tostring(customDimensions['<DialogId>'])
| extend ActivityId = tostring(customDimensions['<activityId>'])
| where DialogId != '' and  InstanceId != '' and user_Id != ''
| extend metric = InstanceId // DialogId or ActivityId
| summarize Count=dcount(metric) by  bin(timestamp, groupByInterval)
| order by Count desc nulls last 
| render timechart
```

> [!TIP]
> Kusto の [extend 演算子](https://aka.ms/kusto-extend-operator)は、計算列を作成してそれらを結果セットに追加するために使用されます。


#### <a name="sample-query-results"></a>サンプル クエリの結果

![期間あたりのアクティビティ](./media/convscount.PNG)


### <a name="activity-per-user-per-period"></a>期間あたりのユーザーごとのアクティビティ
このサンプルでは、一定期間におけるユーザーごとのアクティビティ数をカウントする方法を示します。 ここでは、"_期間あたりのアクティビティ_" クエリをドリルダウンして、一定期間におけるユーザーごとのアクティビティに焦点を絞ります。 アクティビティには、ダイアログ、会話、メッセージがあります。  これは、ユーザーとボットのやり取りを測定するのに役立つほか、次のような潜在的な問題を特定するのに役立つ場合があります。 

- ある期間に単一ユーザーのアクティビティが非常に多い場合、攻撃かテストの可能性があります
- ある期間にやり取りがほとんどない場合、サービスの正常性に問題がある可能性があります

> [!TIP]
> _by user_Id_ を削除すると、全般的なボット アクティビティの量を取得し、時間とダイアログ、メッセージ、または会話でピボットできます。

```Kusto
// number of users per period per dialogs
let queryStartDate = ago(14d);
let queryEndDate = now();
let interval = 6h;
customEvents 
| where timestamp > queryStartDate
| where timestamp < queryEndDate
| extend InstanceId = tostring(customDimensions['InstanceId'])
| extend DialogId = tostring(customDimensions['DialogId'])
| extend ActivityId = tostring(customDimensions['activityId'])
| where DialogId != '' and InstanceId != '' and user_Id != ''
| extend metric = ActivityId // InstanceId // DialogId // or InstanceId for conversation count
| summarize Count=dcount(metric) by user_Id, bin(timestamp, groupByInterval)
| order by Count desc nulls last 
```

#### <a name="sample-query-results"></a>サンプル クエリの結果 

| **user_Id**   | **timestamp**        | **Count** |
| ------------- | -------------------- | :------:  |
| User-8107ffd2 | 2019-09-03T00:00:00Z |    14     |
| User-75f2cc8f | 2019-08-30T00:00:00Z |    13     |
| User-75f2cc8d | 2019-09-03T00:00:00Z |    13     |
| User-3060aada | 2019-09-03T00:00:00Z |    10     |


### <a name="dialog-completion"></a>完了したダイアログ
ダイアログのテレメトリ クライアントを設定すると、ダイアログ (とその子) から、"_開始済み_" や "_完了済み_" など、いくつかの既定のテレメトリ データが出力されるようになります。 このサンプルを使用すると、"*完了済み*" ダイアログを "*開始済み*" ダイアログと比較して測定できます。  開始済みダイアログの数が完了済みの数を上回っている場合、一部のユーザーがダイアログ フローを完了していません。 これは、潜在的なダイアログ ロジックを特定し、トラブルシューティングするときの出発点として使用できます。  また、使用頻度が高いダイアログや、使用頻度が低いダイアログの特定にも使用できます。

```Kusto
// % Completed Waterfall Dialog: shows completes relative to starts 
let queryStartDate = ago(14d);
let queryEndDate = now();
customEvents
| where timestamp > queryStartDate
| where timestamp < queryEndDate
| where name=="WaterfallStart"
| extend DialogId = customDimensions['DialogId']
| extend InstanceId = tostring(customDimensions['InstanceId'])
| join kind=leftouter (
    customEvents 
    | where name=="WaterfallComplete" 
    | extend InstanceId = tostring(customDimensions['InstanceId'])
  ) on InstanceId    
| summarize started=countif(name=='WaterfallStart'), completed=countif(name1=='WaterfallComplete') by tostring(DialogId)
| where started > 100  // filter for sample
// Show starts vs. completes
| project tostring(DialogId), started, completed
| order by started desc, completed asc  nulls last 
| render barchart  with (kind=unstacked, xcolumn=DialogId, ycolumns=completed, started, ysplit=axes) 
```

> [!TIP]
> Kusto の [join 演算子](https://aka.ms/kusto-join-operator)を使用すると、各テーブルの指定の列で値を照合することで、2 つのテーブルの行をマージして新しいテーブルを形成できます。
>
> [project 演算子](https://aka.ms/kusto-project-operator)は、自分の出力に表示したいフィールドを選択するために使用されます。 新しいフィールドを追加する `extend operator` と同様に、`project operator` では、既存のフィールド セットから選択することも、新しいフィールドを追加することもできます。

#### <a name="sample-query-results"></a>サンプル クエリの結果

![完了したダイアログ](./media/dialogwfratio.PNG)


### <a name="dialog-incompletion"></a>未完了のダイアログ
このサンプルを使用すると、指定期間において、開始されたもののキャンセルまたは破棄されたために完了しなかったダイアログ フローの数をカウントできます。 これにより、未完了のダイアログを確認し、ユーザーが混乱したために自発的にキャンセルされたのか、それともユーザーの気が逸れたり関心がなくなったりしたために破棄されただけなのかを調べることができます。

<!--  

OPEN ISSUE:

“If number of started dialogs is much greater than number of completed, users do not complete the dialog flow. Troubleshoot dialog logic.”

you can use the funnel view to understand where step dropoff is.  If there is a doc describing how to do that in our docs, point to it... 

QUESTION: Who can I talk to about a a doc describing this? 

ALSO: I removed what was line 6 in the example because it was a duplicate where statement: | where timestamp > period    // change timespan accordingly

-->

```Kusto
// show incomplete dialogs
let queryStartDate = ago(14d);
let queryEndDate = now();
customEvents 
| where timestamp > queryStartDate 
| where timestamp < queryEndDate
| where name == "WaterfallStart" 
| extend DialogId = customDimensions['DialogId']
| extend instanceId = tostring(customDimensions['InstanceId'])
| join kind=leftanti (
  customEvents
  | where name == "WaterfallComplete" 
  | extend instanceId = tostring(customDimensions['InstanceId'])
  ) on instanceId
| summarize cnt=count() by  tostring(DialogId)
| order by cnt
| render barchart
```

> [!TIP]
> Kusto の [order 演算子](https://aka.ms/kusto-query-order-operator) (`sort operator` と同じ) は、1 つ以上の列を基準にして入力テーブルの行を並べ替えるために使用されます。  注:任意のクエリの結果から null 値を除外したい場合は、"and isnotnull(Timestamp)" を追加するなどして、where ステートメントでそれらをフィルタリングできます。また、先頭または末尾で null 値を返すには、order ステートメントの末尾に `nulls first` または `nulls first` を追加します。 
>

#### <a name="sample-query-results"></a>サンプル クエリの結果

![](./media/cancelleddialogs.PNG)





### <a name="dialog-sequence-drill-down"></a>ダイアログ シーケンスのドリルダウン 

#### <a name="waterfall-startstepcomplete-for-dialog-in-conversation"></a>会話中のウォーターフォール ダイアログの開始、ステップ、完了
このサンプルでは、ダイアログ ステップのシーケンスを会話 (instanceId) 別にグループ分けして表示します。 これは、どのステップがダイアログの中断の原因になっているのかを特定するときに役立ちます。 

このクエリを実行するには、\<SampleDialogId> に、目的の `DialogId` の値を入力します 

```Kusto
// Drill down: Show waterfall start/step/complete for specific dialog
let queryStartDate = ago(14d);
let queryEndDate = now();
let DialogActivity=(dlgid:string) {
customEvents
| where timestamp > queryStartDate
| where timestamp < queryEndDate
| extend DialogId = customDimensions['DialogId']
| extend StepName = customDimensions['StepName']
| extend InstanceId = customDimensions['InstanceId']
| where DialogId == dlgid
| project timestamp, name, StepName, InstanceId 
| order by tostring(InstanceId), timestamp asc
};
// For example see SampleDialogId behavior
DialogActivity("<SampleDialogId>")
```

> [!TIP]
> このクエリは、[クエリ定義関数](https://aka.ms/kusto-user-functions)を使用して記述されています。これは、単一クエリのスコープ内で定義および使用されるユーザー定義関数であり、let ステートメントを通じて定義されます。 このクエリが `query-defined function` を使用せずに記述された場合:
>
> ```Kusto
> let queryStartDate = ago(14d);
> let queryEndDate = now();
> customEvents
> | where timestamp > queryStartDate
> | where timestamp < queryEndDate
> | extend DialogId = customDimensions['DialogId']
> | extend StepName = customDimensions['StepName']
> | extend InstanceId = customDimensions['InstanceId']
> | where DialogId == "<SampleDialogId>"
> | project timestamp, name, StepName, InstanceId 
> | order by tostring(InstanceId), timestamp asc
> ```

##### <a name="sample-query-results"></a>サンプル クエリの結果

| **timestamp**       | **name**                   | **StepName**                    | **InstanceId**  |
| ------------------- | -------------------------- | ------------------------------- | --------------- |
| 2019-08-23T20:04... | WaterfallStart             | null                            | ...79c0f03d8701 |
| 2019-08-23T20:04... | WaterfallStep              | GetPointOfInterestLocations     | ...79c0f03d8701 |
| 2019-08-23T20:04... | WaterfallStep              | ProcessPointOfInterestSelection | ...79c0f03d8701 |
| 2019-08-23T20:04... | WaterfallStep              | GetRoutesToDestination          | ...79c0f03d8701 |
| 2019-08-23T20:05... | WaterfallStep              | ResponseToStartRoutePrompt      | ...79c0f03d8701 |
| 2019-08-23T20:05... | WaterfallComplete _<sup>1_ | null                            | ...79c0f03d8701 |
| 2019-08-28T23:35... | WaterfallStart             | null                            | ...6ac8b3211b99 |
| 2019-08-28T23:35... | WaterfallStep _<sup>2_     | GetPointOfInterestLocations     | ...6ac8b3211b99 |
| 2019-08-28T19:41... | WaterfallStart             | null                            | ...8137d76a5cbb |
| 2019-08-28T19:41... | WaterfallStep _<sup>2_     | GetPointOfInterestLocations     | ...8137d76a5cbb |
| 2019-08-28T19:41... | WaterfallStart             | null                            | ...8137d76a5cbb |

<sub>1</sup> "_完了_" 

<sub>2</sup> "_破棄_"

"_解釈: ユーザーは GetPointOfInterestLocations ステップで会話を破棄しているようです。_ " 

> [!NOTE] 
> ウォーターフォール ダイアログではシーケンス (開始、複数のステップ、完了) が実行されます。 シーケンスが開始を示しているのに完了されていない場合、ユーザーによるダイアログの破棄またはキャンセルが原因でダイアログが中断されたことを意味します。 この詳細な分析では、このような動作が見られることがあります (完了ステップと破棄ステップを見比べてください)。

#### <a name="waterfall-startstepcompletecancel-steps-aggregate-totals"></a>ウォーターフォールの開始、ステップ、完了、キャンセル ステップの集計値
このサンプルでは、ダイアログ シーケンスが開始された合計回数の集計値、ウォーターフォール ステップの総数、正常に完了した件数、キャンセル数が表示され、_WaterfallStart_ の数と _WaterfallComplete_ および _WaterfallCancel_ の総数との差から、破棄の総数がわかります。

```Kusto
// Drill down: Aggregate view of waterfall start/step/complete/cancel steps totals for specific dialog
let queryStartDate = ago(14d);
let queryEndDate = now();
let DialogSteps=(dlgid:string) {
customEvents
| where timestamp > queryStartDate
| where timestamp < queryEndDate
| extend DialogId = customDimensions['DialogId']
| where DialogId == dlgid
| project name
| summarize count() by name
};
// For example see SampleDialogId behavior
DialogSteps("<SampleDialogId>")
```

##### <a name="sample-query-results"></a>サンプル クエリの結果

| **name**          | **count** |
| ----------------- | --------: |
| WaterfallStart    | 21        |
| WaterfallStep     | 47        |
| WaterfallComplete | 11        |
| WaterfallCancel   | 1         |

"_解釈: 21 個のダイアログ シーケンスの呼び出しのうち、完了したものは 11 個のみであり、9 個は破棄、1 個はユーザーによってキャンセルされました_"



### <a name="average-duration-in-dialog"></a>ダイアログの平均時間
このサンプルでは、ユーザーが特定のダイアログに費やした時間の平均値を測定します。 ダイアログにかかる時間が長い場合、簡素化を検討したほうがよい可能性があります。

 ```Kusto
// Average dialog duration
let queryStartDate = ago(14d);
let queryEndDate = now();
customEvents 
| where timestamp > queryStartDate
| where timestamp < queryEndDate
| where name=="WaterfallStart"
| extend DialogId = customDimensions['DialogId']
| extend instanceId = tostring(customDimensions['InstanceId'])
| join kind=leftouter (customEvents | where name=="WaterfallCancel" | extend instanceId = tostring(customDimensions['InstanceId'])) on instanceId 
| join kind=leftouter (customEvents | where name=="WaterfallComplete" | extend instanceId = tostring(customDimensions['InstanceId'])) on instanceId 
| extend duration = case(not(isnull(timestamp1)), timestamp1 - timestamp, 
not(isnull(timestamp2)), timestamp2 - timestamp, 0s) // Abandoned are not counted. Alternate: now()-timestamp)
| extend seconds = round(duration / 1s)
| summarize AvgSeconds=avg(seconds) by tostring(DialogId)
| order by AvgSeconds desc nulls last 
| render barchart with (title="Duration in Dialog")
 ```

#### <a name="sample-query-results"></a>サンプル クエリの結果

![ダイアログの時間](./media/dialogduration.PNG)


### <a name="average-steps-in-dialog"></a>ダイアログの平均ステップ数
このサンプルでは、各実行済みダイアログの "長さ" の計算値を、平均値、最小値、最大値、標準偏差によって示します。 これは、ダイアログの質の分析に役立ちます。 次に例を示します。

- ステップが非常に多いダイアログは、簡素化できないかどうかを検討することをお勧めします
- 最小値、最大値、平均値の差が大きいダイアログでは、ユーザーがタスクを完了しようとして行き詰まっている可能性があります。 タスク完了までのパスを短縮できないか、またはダイアログの複雑さを軽減する方法がないかを検討することをお勧めします。
- 標準偏差が大きいダイアログでは、パスが複雑であるか、中断 (破棄またはキャンセル) が発生している可能性があります
- ステップがわずかなダイアログは完了されていないので、それらにも同様の可能性があります。 こうした判断を行うには、完了と破棄の比率分析が有用です。  

```Kusto
// min/max/std/avg steps per dialog
let queryStartDate = ago(14d);
let queryEndDate = now();
customEvents
| where timestamp > queryStartDate
| where timestamp < queryEndDate
| extend DialogId = tostring(customDimensions['DialogId'])
| extend StepName = tostring(customDimensions['StepName'])
| extend InstanceId = tostring(customDimensions['InstanceId'])
| where name == "WaterfallStart" or  name == "WaterfallStep" or  name == "WaterfallComplete" 
| order by InstanceId, timestamp asc
| project timestamp, DialogId, name, InstanceId, StepName 
| summarize cnt=count() by InstanceId, DialogId
| summarize avg=avg(cnt), minsteps=min(cnt),maxsteps=max(cnt), std=stdev(cnt) by DialogId
| extend avgsteps = round(avg, 1)
| extend avgshortbysteps=maxsteps-avgsteps
| extend avgshortbypercent=round((1.0 - avgsteps/maxsteps)*100.0, 1)
| project DialogId, avgsteps, minsteps, maxsteps, std, avgshortbysteps, avgshortbypercent
| order by std desc nulls last 
```

#### <a name="sample-query-results"></a>サンプル クエリの結果

| Dialog Id               | avg steps | min steps | max steps | std  | avg short by steps | avg short by percent |
| ----------------------- | --------: | :-------: | :-------: | ---: | :----------------: | -------------------: |
| FindArticlesDialog      | 6.2       | 2         | 7         | 2.04 | 0.8                | 11.4%                |
| CreateTicket            | 4.3       | 2         | 5         | 1.5  | 0.7                | 14%                  |
| CheckForCurrentLocation | 3.9       | 2         | 5         | 1.41 | 1.1                | 22%                  |
| BaseAuth                | 3.3       | 2         | 4         | 1.03 | 0.7                | 17.5%                |
| オンボード              | 2.7       | 2         | 4         | 0.94 | 1.3                | 32.5%                |

解釈: たとえば、FindArticlesDialog は最小値と最大値の差が大きいので、調査が必要であり、再設計と最適化を行わなければならない可能性があります。



### <a name="channel-activity-by-activity-metric"></a>アクティビティ メトリック別のチャネル アクティビティ
このサンプルでは、指定期間においてボットが受信した、チャネルあたりのアクティビティの量を測定します。 これは、受信メッセージ、ユーザー、会話、ダイアログのいずれかのメトリックをカウントすることで実行されます。 これは、サービスの正常性分析やチャネルの使用頻度の測定に役立ちます。


```Kusto
// number of metric: messages, users, conversations, dialogs by channel
let queryStartDate = ago(14d);
let queryEndDate = now();
let groupByInterval = 1d;
customEvents 
| where timestamp > queryStartDate
| where timestamp < queryEndDate
| extend InstanceId = tostring(customDimensions['InstanceId'])
| extend DialogId = tostring(customDimensions['DialogId'])
| extend ActivityId = tostring(customDimensions['activityId'])
| extend ChannelId = tostring(customDimensions['channelId'])
| where DialogId != '' and  InstanceId != '' and user_Id != ''
| extend metric = user_Id // InstanceId or ActivityId or user_Id
| summarize Count=count(metric) by  ChannelId, bin(timestamp, groupByInterval)
| order by Count desc nulls last 
| render barchart with (title="Users", kind=stacked) // or Incoming Messages or Conversations or Users
```

> [!TIP]
> これらのバリエーションを試すこともお勧めします。
> - タイムスタンプ バケットを指定せずにクエリを実行します (`bin(timestamp, groupByInterval)`)。
> - 全ユーザー イベント アクティビティ用の `count` ではなく、個別ユーザー用の `dcount` を使用することもできます。  これは、リピート ユーザーにも適しています。
> 

#### <a name="sample-query-results"></a>サンプル クエリの結果

![ChannelsUsage](./media/ChannelsUsage.PNG)

 "_解釈: 以前はエミュレーター テストが最も使用されていましたが、DirectLineSpeech の稼働開始後は、これが最も使用頻度の高いチャネルとなっています。_ "

<!--  

Open Issue: More interesting than the “certainty” score would be linking intent to dialog completion %. That infers “certainty” by user’s actions. 

QUESTION: What changes are required?

-->
### <a name="total-intents-by-popularity"></a>使用頻度別の総意図数
このサンプルは LUIS 対応ボットに適用されます。 使用頻度別の全[意図](https://aka.ms/botbuilder-luis-concept#recognize-intent)の概要、および対応する意図検出の確実性スコアが示されます。

* 実際に使用するときは、ビューをメトリックごとに分割することをお勧めします。
* 使用頻度の高い意図パスは、ユーザー エクスペリエンスのために最適化することをお勧めします。
* 平均スコアが低い場合、認識力が低く、実際のユーザーの意図を捉え損なっていると考えられます。

```Kusto
// show total intents
let queryStartDate = ago(14d);
let queryEndDate = now();
customEvents
| where timestamp > queryStartDate
| where timestamp < queryEndDate
| where name startswith "LuisResult" 
| extend intentName = tostring(customDimensions['intent'])
| extend intentScore = todouble(customDimensions['intentScore'])
| summarize ic=count(), ac=avg(intentScore)*100 by intentName
| project intentName, ic, ac
| order by ic desc nulls last 
| render barchart with (kind=unstacked, xcolumn=intentName, ycolumns=ic,ac, title="Intents Popularity")
```

#### <a name="sample-query-results"></a>サンプル クエリの結果

![IntentPopularity](./media/Telemetry/IntentPopularity.PNG)

"_解釈: たとえば、最も使用頻度の高い意図について、確認は平均して 23% の信頼度でしか検出されていません。_ "


> [!TIP]
> 棒グラフは、Kusto クエリで使用できる十数個のオプションの 1 つにすぎません。  他のオプションには、anomalychart、areachart、columnchart、linechart、scatterchart などがあります。 詳細については、「[render 演算子](https://aka.ms/kusto-query-render-operator?pivots=Kusto)」トピックを参照してください。


## <a name="schema-of-bot-analytics-instrumentation"></a>ボット分析のインストルメンテーションのスキーマ
次の表に、ボットのテレメトリ データが記録される最も一般的なフィールドを示します。

### <a name="general-envelope"></a>一般的なエンベロープ

Application Insights インストルメンテーションの一般的なログ分析フィールド。

| **フィールド**        | **説明**        | **サンプル値**                                            |
| ---------------- | ---------------------- | ------------------------------------------------------------ |
| name             | メッセージ型           | BotMessageSend、BotMessageReceived、LuisResult、WaterfallStep、WaterfallStart、SkillWebSocketProcessRequestLatency、SkillWebSocketOpenCloseLatency、WaterfallComplete、QnaMessage、WaterfallCancel、SkillWebSocketTurnLatency、AuthPromptValidatorAsyncFailure |
| customDimensions | SDK のボット分析      | activityId=\<id>、activityType=message、channelId=emulator、fromId=\<id>、fromName=User、locale=en-us、recipientId=\<id>、recipientName=Bot、text=find a coffee shop |
| timestamp        | イベントの日時          | 2019-09-05T18:32:45.287082Z                                  |
| instance_Id      | 会話 ID        | f7b2c416-a680-4b2c-b4cc-79c0f03d8711                         |
| operation_Id     | ターン ID                | 084b2856947e3844a5a18a8476d99aaa                             |
| user_Id          | 一意のチャネル ユーザー ID | emulator7c259c8e-2f47...                                     |
| client_IP        | クライアント IP アドレス      | 127.0.0.1 (プライバシー保護のため記録されないことがあります)               |
| client_City      | クライアントの市区町村            | レドモンド (検出された場合。記録されないことがあります)                         |

### <a name="custom-dimensions"></a>カスタム ディメンション

ボット固有のアクティビティ データの大部分は、_customDimensions_ フィールドに格納されます。

| **フィールド**     | **説明**      | **サンプル値**                                 |
| ------------- | -------------------- | ------------------------------------------------- |
| activityId    | メッセージ ID           | \<id>: 8da6d750-d00b-11e9-80e0-c14234b3bc2a       |
| activityType  | メッセージの種類      | message、conversationUpdate、event、invoke       |
| channelId     | チャネル ID   | emulator、directline、msteams、webchat           |
| fromId        | 送信者 ID      | \<id>                                             |
| fromName      | クライアントのユーザー名 | John Bonham、Keith Moon、Steve Smith、Steve Gadd |
| locale        | クライアントの配信元ロケール | en-us、zh-cn、en-GB、de-de、zh-CN                |
| recipientId   | 受信者 ID | \<id>                                             |
| recipientName | 受信者名       | John Bonham、Keith Moon、Steve Smith、Steve Gadd |
| text          | メッセージ内のテキスト      | コーヒー ショップを探してください                                |

### <a name="custom-dimensions-luis"></a>カスタム ディメンション: LUIS

LUIS インストルメンテーションでは、次のカスタム ディメンション フィールドにデータが格納されます。

| **フィールド**      | **説明**         | **サンプル値**                                       |
| -------------- | ----------------------- | ------------------------------------------------------- |
| 意図         | LUIS で検出された意図    | pointOfInterestSkill                                    |
| intentScore    | LUIS 認識スコア  | 0.98                                                    |
| [エンティティ]       | LUIS で検出されたエンティティ  | FoodOfGrocery = [["コーヒー"]]、KEYWORD= ["コーヒー ショップ"] |
| 質問       | LUIS で検出された質問  | コーヒー ショップを探してください                                      |
| sentimentLabel | LUIS で検出されたセンチメント | ポジティブ                                                |

### <a name="custom-dimensions-qnamaker"></a>カスタム ディメンション: QnA Maker

QnAMaker インストルメンテーションでは、次のカスタム ディメンション フィールドにデータが格納されます。

| **フィールド**       | **説明**            | **サンプル値**                                            |
| --------------- | -------------------------- | ------------------------------------------------------------ |
| question        | QnA で検出された質問      | 何を実行できますか?                                             |
| 応答して          | QnA の回答                 | 質問があるなら、お答えできるかもしれません。                     |
| articleFound    | QnA                        | true                                                         |
| questionId      | QnA の質問 ID            | 488                                                          |
| knowledgeBaseId | QnA の KB ID                  | 2a4936f3-b2c8-44ff-b21f-67bc413b9727                         |
| matchedQuestion | 一致した質問の配列 | ["あなたのロールについて教えてください","あなたについて少し教えてください","あなたについて教えてください","手伝っていただけますか","えーと、何ができますか?","何ができますか","何をしてもらえますか?","何をしてくれますか?","それでは、私のプロジェクトでどのように活用すればいいですか?","機能について教えてください","どのような機能がありますか?",… |

 

## <a name="see-also"></a>参照

* ログ クエリの記述に関するチュートリアルについて「[Azure Monitor でログ クエリの使用を開始する](https://aka.ms/azure-monitor-log-queries-get-started)」を確認する
* [Azure Monitor からのデータを視覚化する](https://aka.ms/azure-monitor-visualize-data)
* [ボットにテレメトリを追加する](https://aka.ms/AddBotTelemetry)方法を確認する
* [Azure Monitor のログ クエリ](https://aka.ms/azure-monitor-log-queries)の詳細について確認する
* [Log Analytics データのダッシュボードを作成して共有する](https://aka.ms/log-analytics-create-share-dashboards)
