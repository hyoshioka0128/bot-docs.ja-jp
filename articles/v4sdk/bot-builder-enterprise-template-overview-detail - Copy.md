---
title: Enterprise Bot Template の詳細 | Microsoft Docs
description: Enterprise Bot Template の背後にある設計の決定方法について説明します
author: darrenj
ms.author: darrenj
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 09/18/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 6f295794ca7d3cc17688337e70df2a52cdb665ed
ms.sourcegitcommit: 87b5b0ca9b0d5e028ece9f7cc4948c5507062c2b
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/24/2018
ms.locfileid: "47029851"
---
# <a name="enterprise-template---detailed-overview"></a>Enterprise Template - 詳細

> [!NOTE]
> このトピックは、SDK の v4 バージョンに適用されます。 

Enterprise Bot Template は、会話エクスペリエンスのビルドを通じて特定した数々のベスト プラクティスを 1 つにまとめ、Azure Bot Service の開発者にとって有益なコンポーネントの統合を自動化します。 このセクションでは、そのテンプレートがなぜそのように動作するのか説明するのに役立つ、重要な決定を行うためのいくつかの背景について説明します。

## <a name="introduction-card"></a>紹介カード

多くの会話エクスペリエンスに関するエンドユーザーにとっての重要な問題は、どのように開始すればよいかわからないことです。結果、一般的な質問につながり、ボットが適切に回答するようには設定されていない可能性があります。 第一印象が重要です。 紹介カードは、ボットの機能をエンド ユーザーに紹介する機会を提供し、ユーザーが会話を始めるために使用できる最初の質問をいくつか提案します。 また、ボットの人格を表に出すよい機会でもあります。

基準となる簡単な紹介カードが用意されており、必要に応じて変更できます。

## <a name="basic-language-understanding-luis-intents"></a>基本的な Language Understanding (LUIS) の意図

すべてのボットは基本レベルの会話の言語理解を処理する必要があります。 たとえば、あいさつなどの基本はすべてのボットが簡単に処理できる必要があります。 通常、開発者はこれらの基本意図を作成し、はじめに初期トレーニング データを提供する必要があります。 Enterprise Bot Template には LU ファイルのサンプルが用意されており、これによりすべてのプロジェクトで毎回これらを作成することなく、すぐに基本レベルの機能を使用できます。

LU ファイルには、次の意図が英語、フランス語、イタリア語、ドイツ語、スペイン語で用意されています。

> Greeting、Help、Cancel、Restart、Escalate、ConfirmYes、ConfirmNo、ConfirmMore、Next、Goodbye

[LU](https://github.com/Microsoft/botbuilder-tools/blob/master/packages/Ludown/docs/lu-file-format.md) の形式はマークダウンと似たような形式であるため、変更やソース 管理を簡単に行うことができます。 次に、[LuDown](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/Ludown) ツールを使用して .LU ファイルを LUIS モデルに変更し、それをポータルまたは関連付けられた [LUIS](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/LUIS) CLI (コマンド ライン) ツールを通じて LUIS サブスクリプションに公開できます。

## <a name="content-moderator"></a>Content Moderator

Content Moderator を使用すると、潜在的に不適切な表現を検出し、個人を特定できる情報 (PII) をチェックするのに役立ちます。 これをボットに組み込むと、不適切な表現や PII の共有に対してボットが反応できるようになります。 たとえば、ボットが人に対して謝罪したり干渉しなくなったりするほか、PII が検出された場合はテレメトリの記録を格納しないようになります。

TurnState オブジェクト上の ```TextModeratorResult``` を通じて、テキストやサーフェスを画面に映すミドルウェア コンポーネントが提供されます。

## <a name="telemetry"></a>テレメトリ

ボットのユーザー エンゲージメントに関する分析情報を提供することは、非常に重要であると実証されています。 この分析情報は、ユーザー エンゲージメントのレベル、ユーザーがボットのどの機能を使用しているか (意図) を把握するのに役立ちます。このほかにも、ボットが回答できない質問を把握することで、ボットの知識のギャップを浮き彫りにし、たとえば新しい QnA Maker の記事を使用するなどの対策を取ることができます。

Application Insights の統合により、追加設定なしに運用面や技術面に関する重要な分析情報が得られますが、これは LUIS や QnA Maker の操作に伴い送受信されたメッセージなど、ボットに関連する具体的なイベントをキャプチャするのにも使用できます。

ボット レベルのテレメトリは本質的に技術面と運用面のテレメトリにリンクされており、ユーザーの質問に対してどのように回答されたか、および回答されなかったかを確認できます。

ミドルウェア コンポーネントと QnA Maker や LuisRecognizer の SDK クラスを囲むラッパー クラスを組み合わせることで、一貫性のある一連のイベントを洗練された方法で収集できます。 これらの一貫性のあるイベントは、その後 Application Insights のツールや PowerBI などのツールによって使用されます。

各プロジェクトには、Enterprise Bot Template を使用して作成された、PowerBI ダッシュボードのサンプルが用意されています。 詳しくは、[PowerBI](bot-builder-enterprise-template-powerbi.md) のセクションをご覧ください。

## <a name="dispatcher"></a>ディスパッチャー

会話エクスペリエンスの第一波によい効果をもたらす重要な設計パターンは、Language Understanding (LUIS) と QnAMaker を活用することでした。 LUIS にはボットがエンド ユーザーに対して行うことができるタスクについてのトレーニングが行われ、QnA Maker にはより全般的な知識についてのトレーニングが行われます。

受信されるすべての発話 (質問) は、分析のために LUIS にルーティングされます。 指定の発話の意図が特定されていなかった場合、それは None 意図としてマークされていました。 その後、そのエンド ユーザーの質問に対する答えを探すのに、QnA Maker が使用されました。

このパターンはよく機能していた一方で、2 つの主要なシナリオで問題が発生することがありました。

- LUIS モデルと QnA Maker の発話が一部オーバーラップすると、QnA Maker に送信されるはずの質問を LUIS が処理を試行するという、予想外の挙動が発生することがあります。
- LUIS モデルが 2 つ以上存在すると、ボットはそれぞれを呼び出し、何らかの形で意図の評価を比較して、指定の発話をどちらに送信するかを特定しなければなりませんでした。 共通のベースライン スコアがないため、モデル間の比較はあまり機能せず、ユーザー エクスペリエンスの低下を招いていました。

この問題は[ディスパッチャー](https://docs.microsoft.com/en-us/azure/bot-service/bot-builder-tutorial-dispatch?view=azure-bot-service-4.0&tabs=csaddref%2Ccsbotconfig)により、構成された各 LUIS モデルから発話を、QnA Maker から質問を抽出し、集中的にディスパッチされる LUIS モデルを作成することで解決しました。

これにより、指定の発話をどの LUIS モデルまたはコンポーネントが処理する必要があるかをボットがすばやく特定し、以前のように None 意図だけでなく、QnA Maker のデータが意図処理の最上位で検討されるようになります。

また、このディスパッチ ツールで評価を行うことで、LUIS モデルと QnA Maker のナレッジベース間の混乱とオーバーラップを浮き彫りにし、デプロイ前に問題を明らかにします。

ディスパッチャーは、Enterprise Bot Template を使用して作成される各プロジェクトの根幹で使用されます。 ディスパッチ モデルは `MainDialog` クラス内で使用され、そのターゲットが LUIS モデルであるか QnA であるかを特定します。 LUIS の場合、LUIS のセカンダリ モデルが呼び出され、意図とエンティティが通常どおりに返されます。

## <a name="qnamaker"></a>QnA Maker

[QnA Maker](https://www.qnamaker.ai/) を使用すると、開発者以外のユーザーが、質問と回答のペアの形式で、全般的な知識をキュレーションできます。 この知識は FAQ のデータ ソースから、製品マニュアルからのほか、QnA Maker ポータル内で対話的にインポートできます。

QnA エントリーのサンプル セットは、CogSvcModels の QnA フォルダー内に [LU](https://github.com/Microsoft/botbuilder-tools/blob/master/packages/Ludown/docs/lu-file-format.md) ファイル形式で提供されます。 次に、[LuDown](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/Ludown) が配置スクリプトの一部として使用され、QnA Maker の JSON ファイルを作成します。その後、[QnA Maker](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/QnAMaker) CLI (コマンド ライン) ツールがそれを使用して、QnA Maker のナレッジベースにアイテムを発行します。
