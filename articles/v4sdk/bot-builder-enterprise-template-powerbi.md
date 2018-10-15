---
title: PowerBI を使った会話型分析 | Microsoft Docs
description: Enterprise Bot Template で、PowerBI を通じたインサイトが Application Insights を使って有効化されるしくみついて説明します
author: darrenj
ms.author: darrenj
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 09/18/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 18dcaeaf2967a90ca3ecb8ff32c1ef6399d5f498
ms.sourcegitcommit: 3bf3dbb1a440b3d83e58499c6a2ac116fe04b2f6
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/23/2018
ms.locfileid: "46708724"
---
# <a name="enterprise-bot-template---conversational-analytics-using-powerbi-dashboard-and-application-insights"></a>Enterprise Bot Template - PowerBI ダッシュボードと Application Insights を使用した会話型分析

> [!NOTE]
> このトピックは、SDK の v4 バージョンに適用されます。 

ボットがデプロイされてメッセージの処理が始まると、リソース グループ内の Application Insights インスタンスにテレメトリが送られていきます。 

このテレメトリは、Azure portal の [Application Insights] ブレード内で、Log Analytics を使用して表示できます。 また、PowerBI で同じテレメトリを使用して、ボットの使用状況に関する一般的なビジネス インサイトを取得することもできます。

サンプルの PowerBI ダッシュボードは、作成したプロジェクトの PowerBI フォルダーにあります。 これはあくまでサンプルであり、独自のインサイトの作成方法を示すために提供されているものです。 これらの視覚化機能は、今後強化されていく予定です。 

## <a name="getting-started"></a>Getting Started (概要)

- PowerBI Desktop を[こちら](https://powerbi.microsoft.com/en-us/desktop/)からダウンロードします
 
- ボットで使用する Application Insights リソースの ```Application Id``` を取得します。 これは、[Application Insights Azure] ブレードの [構成] セクションにある [API アクセス] ページに移動して取得できます。

ソリューションの [PowerBI] フォルダー内にある PowerBI テンプレート ファイルをダブルクリックすると、 前の手順で取得した ```Application Id``` の入力を求めるメッセージが表示されます。 メッセージが表示されたら、Azure サブスクリプションの資格情報を使用して認証を完了します (場合によっては、[組織アカウント] をクリックしてサインインする必要があります)。

これで、ダッシュボードが Application Insights インスタンスにリンクされます。メッセージが送受信が完了すると、ダッシュボードに最初のインサイトが表示されます。

>なお、センチメントの視覚化データは表示されません。現在のところ、デプロイ スクリプトでは、LUIS モデルの発行時にセンチメントは有効化されません。 LUIS モデルを[再発行](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/luis-how-to-publish-app)してセンチメントを有効にすれば、視覚化データを表示できます。

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
    - Conversationid
    - ConversationName
    - Locale
    - UserName
    - Text
```
  
```
-BotMessageSent
    - ActivityId,
    - Channel
    - RecipientId
    - Conversationid
    - ConversationName
    - Locale
    - ReceipientName
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
```

```
- QnAMaker
    - ActivityId
    - ConversationId
    - OriginalQuestion
    - UserName
    - QnAItemFound
    - Question
    - Answer
    - Score
```