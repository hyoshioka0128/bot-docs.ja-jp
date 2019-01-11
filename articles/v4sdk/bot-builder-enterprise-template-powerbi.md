---
title: PowerBI を使った会話型分析 | Microsoft Docs
description: Enterprise Bot Template で、PowerBI を通じたインサイトが Application Insights を使って有効化されるしくみついて説明します
author: darrenj
ms.author: darrenj
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 09/18/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 88208a2f5b0eb88d3b2964e63a21585484166d73
ms.sourcegitcommit: 2d84d5d290359ac3cfb8c8f977164f799666f1ab
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/09/2019
ms.locfileid: "54152175"
---
# <a name="enterprise-bot-template---conversational-analytics-using-powerbi-dashboard-and-application-insights"></a>Enterprise Bot Template - PowerBI ダッシュボードと Application Insights を使用した会話型分析

> [!NOTE]
> このトピックは、SDK の v4 バージョンに適用されます。 

ボットがデプロイされてメッセージの処理が始まると、リソース グループ内の Application Insights インスタンスにテレメトリが送られていきます。 

このテレメトリは、Azure portal の [Application Insights] ブレード内で、Log Analytics を使用して表示できます。 また、PowerBI で同じテレメトリを使用して、ボットの使用状況に関する一般的なビジネス インサイトを取得することもできます。

PowerBI ダッシュボードの例は、[会話型 AI テレメトリ](https://aka.ms/botPowerBiTemplate)に関するページにあります。 

これはあくまでサンプルであり、独自のインサイトの作成方法を示すために提供されているものです。 これらの視覚化機能は、今後強化されていく予定です。 


## <a name="middleware-processing"></a>ミドルウェアの処理

QnAMaker クラスと LuisRecognizer クラスのテレメトリ ラッパーは、シナリオに関係なく一貫したテレメトリを出力すると共に、各プロジェクトで標準のダッシュボードを使用できるようにするために提供されています。

```TelemetryLuisRecognizer``` と ```TelemetryQnAMaker``` では、コンストラクターに関するプロパティが提供されており、開発者はこれを使用して、ユーザー名と元のメッセージをログを無効にすることができます。 ただし、これを使用すると、取得できるインサイトが減ることになります。

## <a name="telemetry-captured"></a>キャプチャされるテレメトリ

```TelemetryLuisRecognizer``` と ```TelemetryQnAMaker``` では、4 つのテレメトリ イベントがキャプチャされます。Enterprise テンプレートでは、これらが既定で有効になっています。 

プロジェクトで使用される各 LUIS インテントには、LuisIntent というプレフィックスが付けられます。 これは、ダッシュボードでインテントを簡単に識別できるようにするためのものです。

```
-BotMessageReceived
    - ActivityId
    - Channel
    - FromId
    - FromName
    - ConversationId
    - ConversationName
    - Locale
    - Text
    - RecipientId
    - RecipientName
```
  
```
-BotMessageSent
    - ActivityId,
    - Channel
    - RecipientId
    - ConversationId
    - ConversationName
    - Locale
    - RecipientId
    - RecipientName
    - ReplyToId
    - Text
```

```
- LuisIntent.*
    - ActivityId
    - Intent
    - IntentScore
    - SentimentLabel
    - SentimentScore
    - ConversationId
    - Question
    - DialogId
```

```
- QnAMaker
    - ActivityId
    - ConversationId
    - OriginalQuestion
    - FromName
    - ArticleFound
    - Question
    - Answer
    - Score
```