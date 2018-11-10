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
ms.openlocfilehash: 2ee47d472518e54cf07b86648e270e40f8f015c0
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/24/2018
ms.locfileid: "49997229"
---
# <a name="enterprise-template---detailed-overview"></a>Enterprise テンプレート - 詳細な概要

> [!NOTE]
> このトピックは、SDK の v4 バージョンに適用されます。 

Enterprise ボット テンプレートでは、Microsoft が会話エクスペリエンスの構築を通じて特定した多数のベスト プラクティスがまとめられており、Azure Bot Service 開発者にとって非常に有益であると Microsoft が判断したコンポーネントの統合が自動化されます。 このセクションでは、テンプレートの機能の目的を明らかにするうえで役に立つ、重要な決定のバックグランドについて説明します。

## <a name="introduction-card"></a>紹介カード

多くの会話エクスペリエンスでは、エンドユーザーがどのように利用を開始すればよいかわからないことが重要な問題です。これにより、回答するための適切な設定がボットで行われていない可能性のある、一般的な質問が行われてしまいます。 第一印象が重要です。 紹介カードによって、ボットの機能をエンド ユーザーに紹介する機会が提供されるほか、ユーザーが利用を開始するために使える最初の質問がいくつか提案されます。 これは、お客様のボットの個性を表に出す良い機会にもなります。

必要に応じて調節できる標準として、シンプルな紹介カードが提供されます。

## <a name="basic-language-understanding-luis-intents"></a>Language Understanding (LUIS) の基本的な意図

すべてのボットは、会話での基本レベルの言語理解に対応する必要があります。 すべてのボットが簡単に対応すべき基本的なことの例としては、あいさつがあります。 通常、開発者は最初にこれらの基本的な意図を作成して、初期トレーニング データを入力する必要があります。 Enterprise ボット テンプレートには、お客様が利用を開始するためのサンプル LU ファイルが用意されているので、プロジェクトごとに毎回それらを作成する必要がなく、すぐに基本レベルの機能を使用できます。

LU ファイルには、英語、フランス語、イタリア語、ドイツ語、スペイン語で以下の意図が用意されています。

> Greeting、Help、Cancel、Restart、Escalate、ConfirmYes、ConfirmNo、ConfirmMore、Next、Goodbye

[LU](https://github.com/Microsoft/botbuilder-tools/blob/master/packages/Ludown/docs/lu-file-format.md) 形式はマークダウンと類似していて、変更とソース管理を容易に行えます。 次に .LU ファイルを LUIS モデルに変換するために [LuDown](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/Ludown) ツールが使用されます。そしてこの LUIS モデルは、ポータルまたは関連付けられた [LUIS](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/LUIS) CLI (コマンド ライン) ツールを使用してお客様の LUIS サブスクリプションに発行できます。

## <a name="content-moderator"></a>Content Moderator

Content Moderator を使用すると、不適切な可能性がある表現を検出できます。また、Content Moderator は、個人を特定できる情報 (PII) のチェックに役立ちます。 これをボットに統合すると役に立つ可能性があります。ボットは不適切な表現に反応したり、ユーザーが PII 情報を共有した場合に反応したりできるようになります。 たとえば、ボットに謝罪させたり、人への干渉を控えさせたりできます。または、PII 情報が検出された場合に、ボットがテレメトリ レコードを保存しないようにできます。

TurnState オブジェクトの ```TextModeratorResult``` を通じてテキストとサーフェスをスクリーニングするミドルウェア コンポーネントが提供されます。

## <a name="telemetry"></a>テレメトリ

お客様のボットのユーザー エンゲージメントに関する分析情報を提供することは、非常に重要であると証明されています。 この分析情報は、ユーザー エンゲージメントのレベル、ユーザーが使用しているボットの機能 (意図) を把握するのに役立ちます。また、ユーザーがたずねてもボットが回答できない質問も把握できるため、ボットの知識のギャップを明らかにして、たとえば新しい QnA Maker 記事を使用して解消することができます。

Application Insights の統合によって、追加設定なしで運用および技術に関する重要な分析情報が得られます。しかしこれは、LUIS と QnA Maker の操作に伴って送受信されるメッセージなど、特定のボット関連イベントをキャプチャするために使用することもできます。

ボット レベルのテレメトリは、技術および運用に関するテレメトリに本質的にリンクされています。そのためお客様は、ユーザーの特定の質問に対してどのような回答が行われたか、および行われなかったかを調べることができます。

ミドルウェア コンポーネントと QnA Maker および LuisRecognizer の SDK クラスを囲むラッパー クラスとを組み合わせることで、一貫性のある一連のイベントを洗練された方法で収集できます。 これらの一貫性のあるイベントは、その後 PowerBI などのツールと共に Application Insights ツールによって使用されます。

Enterprise ボット テンプレートを使用して作成された各プロジェクトには、サンプル PowerBI ダッシュボードが用意されています。 詳細については、[PowerBI](bot-builder-enterprise-template-powerbi.md) に関するセクションを参照してください。

## <a name="dispatcher"></a>ディスパッチャー

最先端の会話エクスペリエンスで効果を上げるために使用された重要な設計パターンは、Language Understanding (LUIS) と QnA Maker を利用することでした。 LUIS では、お客様のボットがエンド ユーザーのために実行できるタスクについてのトレーニングが行われ、QnA Maker では、より一般的な知識についてのトレーニングが行われます。

入力された発話 (質問) はすべて、分析のために LUIS にルーティングされます。 指定の発話の意図が特定されなかった場合、それは None 意図としてマークされました。 その後、エンドユーザーへの回答を見つけ出すために QnA Maker が使用されていました。

このパターンはうまく機能していたものの、問題が発生し得る重要なシナリオが 2 つありました。

- LUIS モデルと QnA Maker の発話がわずかに重複することがあり、そのようなときには、QnA Maker に転送されるはずの質問の処理を LUIS が試行するかもしれないという、想定外の動作が引き起こされる可能性がありました。
- 2 つ以上の LUIS モデルがあった場合は、ボットによって各モデルが呼び出され、特定の発話の送信先を特定するためになんらかの意図評価比較が実行される必要がありました。 共通のベースライン スコアがないため、モデル間の比較が効果的に機能せず、ユーザー エクスペリエンスの低下を招いていました。

[ディスパッチャー](https://docs.microsoft.com/en-us/azure/bot-service/bot-builder-tutorial-dispatch?view=azure-bot-service-4.0&tabs=csaddref%2Ccsbotconfig)では、構成された各 LUIS モデルから発話を、QnA Maker から質問を抽出して、一元的なディスパッチ LUIS モデルを作成することで、この問題をうまく解決できます。

これにより、指定の発話を処理すべき LUIS モデルまたはコンポーネントをボットが速やかに特定できるほか、QnA Maker のデータが以前のように単に None 意図とされるのではなく、意図処理の最上位レベルで検討されるようになります。

このディスパッチ ツールでは、LUIS モデルと QnA Maker のナレッジベースとの間における混乱と重複を浮き彫りにする評価を実行して、デプロイ前に問題を明らかにすることもできます。

ディスパッチャーは、Enterprise ボット テンプレートを使用して作成された各プロジェクトのコアとして使用されます。 ディスパッチ モデルは、ターゲットが LUIS モデルと QnA のいずれであるかを特定するために `MainDialog` クラス内で使用されます。 LUIS の場合は、セカンダリ LUIS モデルが呼び出され、通常どおり意図とエンティティが返されます。

## <a name="qnamaker"></a>QnA Maker

[QnA Maker](https://www.qnamaker.ai/) には、開発者以外のユーザーが質問と回答のペアという形式で一般的な知識のキュレーションを行える機能があります。 この知識は、FAQ データ ソースと製品マニュアルからインポートできるほか、QnA Maker ポータル内で対話形式でインポートできます。

一連の QnA エントリのサンプルは、CogSvcModels の QnA フォルダー内の [LU](https://github.com/Microsoft/botbuilder-tools/blob/master/packages/Ludown/docs/lu-file-format.md) ファイル形式で提供されます。 次に、デプロイ スクリプトの一部として [LuDown](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/Ludown) が使用され、QnA Maker JSON ファイルが作成されます。そして、[QnA Maker](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/QnAMaker) CLI (コマンド ライン) ツールによってこれが使用され、項目が QnA Maker ナレッジベースに発行されます。