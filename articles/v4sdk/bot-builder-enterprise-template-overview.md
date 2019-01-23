---
title: Enterprise Bot Template の概要 | Microsoft Docs
description: Enterprise Bot Template の機能について説明します
author: darrenj
ms.author: darrenj
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 09/18/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: a0b56b77990c095d1cf8bb28235d06444837d164
ms.sourcegitcommit: b15cf37afc4f57d13ca6636d4227433809562f8b
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/11/2019
ms.locfileid: "54224599"
---
# <a name="enterprise-bot-template"></a>Enterprise Bot Template 

> [!NOTE]
> このトピックは、SDK の v4 バージョンに適用されます。 

質の高い会話エクスペリエンスを作成するには、基礎となる一連の機能が必要です。 優れた会話エクスペリエンスのビルドの成功をサポートするために、Enterprise Bot Template を作成しました。 このテンプレートは、会話エクスペリエンスのビルドを通じて特定されたベスト プラクティスとサポートするコンポーネントを、すべて 1 つにまとめたものです。 

このテンプレートにより、新しいボット プロジェクトの作成が大幅に簡略化されます。 このテンプレートでは、[Bot Framework SDK v4](https://github.com/Microsoft/botbuilder) と [Bot Framework ツール](https://github.com/Microsoft/botbuilder-tools)を活用して、次の機能を追加設定なしで使用できます。

機能 | 説明 |
------------ | -------------
紹介メッセージ | 会話の開始時に流れる、アダプティブ カードの紹介メッセージ。 ボットの機能について説明し、最初の質問をガイドするボタンを提供します。 開発者は必要に応じてこれをカスタマイズできます。
自動化された入力インジケーター  | 会話中に視覚的な入力インジケーターを送信します。長時間実行される操作においては繰り返されます。
.bot ファイル ドリブン構成 | お使いのボット (LUIS、Dispatcher Endpoints、Application Insights など) に関するすべての構成情報が .bot ファイル内にラップされ、ボットの起動に使用されます。
基本的な会話の意図  | 英語、フランス語、イタリア語、ドイツ語、スペイン語、および中国語の基本意図 (こんにちは、さようなら、ヘルプ、キャンセルなど)。 これらは .LU (言語理解) ファイルで提供され、簡単に変更できます。
基本的な会話の応答  | 基本的な会話の意図に対する応答で、別個の View クラスに抽象化されています。 将来、これらは新しい言語生成 (LG) ファイルに移行されます。
不適切なコンテンツまたは PII (個人を特定できる情報) の検出  |ミドルウェア コンポーネントの [Content Moderator](https://azure.microsoft.com/en-us/services/cognitive-services/content-moderator/) の使用を通じて、受信した会話の中の不適切なコンテンツや PII データを特定します。
トランスクリプト  | Azure Storage に格納されているすべての会話のトランスクリプト
ディスパッチャー | 指定の発話を LUIS とコードで処理するか、QnA Maker に渡す必要があるかを特定する、統合された[ディスパッチ](https://docs.microsoft.com/en-us/azure/bot-service/bot-builder-tutorial-dispatch?view=azure-bot-service-4.0&tabs=csaddref%2Ccsbotconfig) モデル。
QnA Maker の統合  | [QnA Maker](https://www.qnamaker.ai) との統合により、既存のデータ ソース (PDF マニュアルなど) を活用するナレッジ ベースから、一般的な質問に回答します。 一般的なクエリに対する標準的な回答を提供するために、QnA Maker の chit-chat モデルも含まれています ([詳細](https://docs.microsoft.com/en-us/azure/cognitive-services/qnamaker/how-to/chit-chat-knowledge-base))。
会話の分析情報  | [Application Insights](https://azure.microsoft.com/en-gb/services/application-insights/) との統合により、すべての会話からテレメトリを収集し、PowerBI ダッシュボードのサンプルを活用して、会話エクスペリエンスの分析を開始できます。

さらに、ボットに必要なすべての Azure リソースの自動デプロイも行われます(ボットの登録、Azure App Service、LUIS、QnA Maker、Content Moderator、CosmosDB、Azure Storage、Application Insights)。 さらに、基本意図のテストとルーティングを即座に行えるように、ベースとなる LUIS、QnA Maker、ディスパッチ モデルが作成およびトレーニングを経て公開されています。

テンプレートを作成し、デプロイ手順を実行後、F5 キーを押して最初から最後まで通してテストを実行できます。 これにより、会話エクスペリエンスの取り組みをどこから開始したらよいかについて堅固な基盤が提供され、かつては数日を要していた各プロジェクトの労力を減らし、会話の品質基準を上げることができます。

開始するには、続けて[プロジェクトの作成](bot-builder-enterprise-template-create-project.md)に進んでください。 前述の機能に関するベスト プラクティスと学習内容について詳しくは、[テンプレートの詳細](bot-builder-enterprise-template-overview-detail.md)に関するトピックをご覧ください。 

Enterprise Bot Template の全ソース コードは、こちらの [GitHub のページ](https://github.com/Microsoft/AI/tree/master/templates/Enterprise-Template)で入手できます。
