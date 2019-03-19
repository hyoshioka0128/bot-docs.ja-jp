---
title: Enterprise ボット テンプレートの詳細な概要 | Microsoft Docs
description: Enterprise ボット テンプレートの背後にある設計上の決定について説明します
author: darrenj
ms.author: darrenj
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 09/18/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 582ec21d36e15fcbaef2d26616a9e55a04d8e5f2
ms.sourcegitcommit: b2245df2f0a18c5a66a836ab24a573fd70c7d272
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/07/2019
ms.locfileid: "57568219"
---
# <a name="enterprise-bot-template---overview"></a>Enterprise Bot Template - 概要

> [!NOTE]
> このトピックは、SDK の v4 バージョンに適用されます。 

Enterprise Bot Template は、会話エクスペリエンスの作成に必要なベスト プラクティスとサービスの強固な基盤を提供し、労力を軽減して品質基準を引き上げます。 このテンプレートでは、[Bot Builder SDK v4](https://github.com/Microsoft/botbuilder) と [Bot Builder ツール](https://github.com/Microsoft/botbuilder-tools)を活用して、次の機能を提供します。

機能      | 説明 |
------------ | -------------
はじめに | 会話の開始時に流れる、[アダプティブ カード]()の紹介メッセージ
入力インジケーター  | 会話中の自動ビジュアル入力インジケーター。長時間実行される操作で繰り返されます
基本 LUIS モデル  | **キャンセル**、**ヘルプ**、**エスカレート**など、一般的な意図をサポートします
基本のダイアログ | 基本的なーザー情報と、キャンセルおよびヘルプ意図の中断ロジックをキャプチャするためのダイアログ フロー
基本の応答  | 基本の意図とダイアログのテキストおよび音声の応答
FAQ | ナレッジ ベースから一般的な質問に回答するための [QnA Maker](https://www.qnamaker.ai) との統合 
おしゃべり | 一般的なクエリに対する標準的な回答を提供するためのプロフェッショナルなおしゃべりモデル ([詳細](https://docs.microsoft.com/en-us/azure/cognitive-services/qnamaker/how-to/chit-chat-knowledge-base))
ディスパッチャー | 指定の発話を LUIS で処理するか QnA Maker で処理するかを特定する、統合された[ディスパッチ](https://docs.microsoft.com/en-us/azure/bot-service/bot-builder-tutorial-dispatch?view=azure-bot-service-4.0&tabs=csaddref%2Ccsbotconfig) モデル。
言語のサポート | 英語、フランス語、イタリア語、ドイツ語、スペイン語、および中国語で使用できます
トランスクリプト | Azure Storage に格納されているすべての会話のトランスクリプト
テレメトリ  | すべての会話のテレメトリを収集するための [Application Insights](https://azure.microsoft.com/en-gb/services/application-insights/) 統合
Analytics | ご自身の会話エクスペリエンス分析情報の使用を開始するときに使用する Power BI ダッシュ ボードのサンプル。
自動化されたデプロイ | [Bot Builder ツール](https://github.com/Microsoft/botbuilder-tools)を使用した前述のサービスすべての容易なデプロイ

# <a name="background"></a>バックグラウンド

## <a name="introduction-card"></a>導入カード
導入カードは、ボット機能の概要とサンプル発話を提供し、容易に開始できるようにすることで会話を向上させます。 また、ユーザーに対するボットのパーソナリティも確立します。

## <a name="base-luis-model-with-common-intents"></a>一般的な意図を持つ基本 LUIS モデル
テンプレートに含まれる基本 LUIS モデルの対象となっているのは、ほとんどのボットが処理する必要のある一般的な意図です。 次の意図は、お使いのボットですぐに使用できます。

意図       | サンプルの発話 |
-------------|-------------|
Cancel       |"*キャンセル*"、"*気にしないで*"|
エスカレート     |"*ユーザーと話すことができますか*"|
FinishTask   |"*完了*"、"*すべて終了*"|
GoBack       |"*戻る*"|
[Help]         |"*助けていただけますか*"|
Repeat       |"*もう一度言ってください*"|
SelectAny    |"*そのいずれか*"|
SelectItem   |"*最初の項目*"|
SelectNone   |"*該当なし*"|
ShowNext     |"*さらに表示*"|
ShowPrevious |"*前を表示*"|
StartOver    |*restart*|
Stop         |*stop*|

## <a name="qna-maker"></a>QnA Maker

[QnA Maker](https://www.qnamaker.ai/) には、開発者以外のユーザーが、ボット内で使用する質問と回答のペアというコレクションを生成できる機能があります。 この知識は、FAQ データ ソースと製品マニュアルからインポートできるほか、QnA Maker ポータル内で手動で作成できます。

テンプレートには 2 つの QnA Maker モデルの例が用意されています。1 つは FAQ 用で、もう 1 つは[プロフェッショナルなおしゃべり](https://docs.microsoft.com/en-us/azure/cognitive-services/qnamaker/how-to/chit-chat-knowledge-base)用です。 

## <a name="dispatch"></a>ディスパッチ

ディスパッチ サービスは、複数の LUIS モデルと QnA Maker ナレッジ ベースの間のルーティングを管理するために使用されます。その際、各サービスから発話が抽出され、中央のディスパッチ LUIS モデルが作成されます。

これにより、与えられた発話を処理すべきコンポーネントをボットが速やかに特定できるほか、QnA Maker のデータが階層の最下部ではなく、意図処理の最上位レベルで検討されるようになります。

このディスパッチ ツールでは、サービス間で重複する発話と不一致が強調表示されるモデルを評価することもできます。

## <a name="telemetry"></a>テレメトリ

ボットの会話に分析情報を提供します。これは、ユーザー エンゲージメントのレベル、ユーザーによって使用される機能、ボットで処理できない質問を把握するうえで役立ちます。

[Application Insights](https://docs.microsoft.com/en-us/azure/azure-monitor/app/app-insights-overview) により、ボット サービスと会話固有のイベントの操作テレメトリがキャプチャされます。 これらのイベントは、[Power BI](https://powerbi.microsoft.com/en-us/what-is-power-bi/) などのツールを使用して、実用的な情報に集計できます。 サンプル Power BI ダッシュボードは、この機能を示す Enterprise Bot Template で使用できます。

# <a name="next-steps"></a>次の手順
ご自身の Enterprise ボットを作成してデプロイする方法については、[作業の開始](bot-builder-enterprise-template-getting-started.md)に関するページをご覧ください。 

# <a name="resources"></a>リソース
Enterprise Bot Template の完全なソース コードは、[GitHub](https://github.com/Microsoft/AI/tree/master/templates/Enterprise-Template) にあります。
