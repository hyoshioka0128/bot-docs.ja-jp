---
title: 仮想アシスタント テンプレートの概要 | Microsoft Docs
description: 仮想アシスタント テンプレートの詳細について説明します
author: darrenj
ms.author: darrenj
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 05/23/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 503ec19444c51120bf46838e14edb891ec5c3bb5
ms.sourcegitcommit: dbbfcf45a8d0ba66bd4fb5620d093abfa3b2f725
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 06/28/2019
ms.locfileid: "67464662"
---
# <a name="virtual-assistant---template-outline"></a>仮想アシスタント - テンプレートの概要

> [!NOTE]
> このトピックは、SDK の v4 バージョンに適用されます。 

仮想アシスタント テンプレートでは、Microsoft が会話エクスペリエンスの構築を通じて特定した多数のベスト プラクティスがまとめられています。これを使うと、Bot Framework 開発者にとって非常に有益であると Microsoft が判断したコンポーネントの統合が自動化されます。 このセクションでは、テンプレートの仕様を説明するうえで役に立つ、重要な決定のバックグランドについて取り上げます。

機能      | 説明 |
------------ | -------------
はじめに | 会話の開始時に流れる、[アダプティブ カード]()の紹介メッセージ
入力インジケーター  | 会話中の自動ビジュアル入力インジケーター。長時間実行される操作で繰り返されます
基本 LUIS モデル  | **キャンセル**、**ヘルプ**、**エスカレート**など、一般的な意図をサポートします
基本のダイアログ | 基本的なーザー情報と、キャンセルおよびヘルプ意図の中断ロジックをキャプチャするためのダイアログ フロー
基本の応答  | 基本の意図とダイアログのテキストおよび音声の応答
FAQ | ナレッジ ベースから一般的な質問に回答するための [QnA Maker](https://www.qnamaker.ai) との統合 
おしゃべり | 一般的なクエリに対する標準的な回答を提供するためのプロフェッショナルなおしゃべりモデル ([詳細](https://docs.microsoft.com/azure/cognitive-services/qnamaker/how-to/chit-chat-knowledge-base))
ディスパッチャー | 指定の発話を LUIS で処理するか QnA Maker で処理するかを特定する、統合された[ディスパッチ](https://docs.microsoft.com/azure/bot-service/bot-builder-tutorial-dispatch?view=azure-bot-service-4.0&tabs=csaddref%2Ccsbotconfig) モデル。
言語のサポート | 英語、フランス語、イタリア語、ドイツ語、スペイン語、および中国語で使用できます
トランスクリプト | Azure Storage に格納されているすべての会話のトランスクリプト
テレメトリ  | すべての会話のテレメトリを収集するための [Application Insights](https://azure.microsoft.com/services/application-insights/) 統合
Analytics | ご自身の会話エクスペリエンス分析情報の使用を開始するときに使用する Power BI ダッシュ ボードのサンプル。
自動化されたデプロイ | Azure ARM テンプレートを使用した前述のサービスすべての容易なデプロイ。

## <a name="introduction-card"></a>導入カード

多くの会話エクスペリエンスでは、エンドユーザーがどのように利用を開始すればよいかわからないことが重要な問題です。これにより、回答するための適切な設定がボットで行われていない可能性のある、一般的な質問が行われてしまいます。 第一印象が重要です。 導入カードによって、ボットの機能をエンド ユーザーに紹介する機会が提供されるほか、ユーザーが利用を開始するために使える最初の質問がいくつか提案されます。 これは、お客様のボットの個性を表に出す良い機会にもなります。

必要に応じて適用できるシンプルな導入カードが標準で用意され、ユーザーが (導入カードの [開始する] ボタンによってトリガーされた) オンボード ダイアログを完了したときに、その後のインタラクションで、ユーザー カードを返すことが示されます

![導入カードの例](./media/enterprise-template/vabotintrocard.png)

## <a name="basic-language-understanding-luis-intents"></a>Language Understanding (LUIS) の基本的な意図

すべてのボットは、会話での基本レベルの言語理解に対応する必要があります。 すべてのボットが簡単に対応すべき基本的なことの例としては、あいさつがあります。 通常、開発者は最初にこれらの基本的な意図を作成して、初期トレーニング データを提供する必要があります。 仮想アシスタント テンプレートには、お客様が利用を開始するためのサンプル LU ファイルが用意されているので、プロジェクトごとに毎回それらを作成する必要がなく、すぐに基本レベルの機能を使用できます。

LU ファイルには、英語、中国語、フランス語、イタリア語、ドイツ語、スペイン語で以下の意図が用意されています。

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

[LU](https://github.com/Microsoft/botbuilder-tools/blob/master/packages/Ludown/docs/lu-file-format.md) 形式はマークダウンと類似していて、変更とソース管理を容易に行えます。 次に .LU ファイルを LUIS モデルに変換するために [LuDown](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/Ludown) ツールが使用されます。そしてこの LUIS モデルは、ポータルまたは関連付けられた [LUIS](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/LUIS) CLI (コマンド ライン) ツールを使用してお客様の LUIS サブスクリプションに発行できます。

## <a name="telemetry"></a>テレメトリ

お客様のボットのユーザー エンゲージメントに関する分析情報を提供することは、非常に重要であると証明されています。 この分析情報は、ユーザー エンゲージメントのレベル、ユーザーが使用しているボットの機能 (意図) を把握するのに役立ちます。また、人々に尋ねられてもボットが回答できない質問も把握できるため、ボットの知識のギャップを明らかにして、たとえば新しい QnA Maker 記事を使用して解消することができます。

Application Insights の統合によって、追加設定なしで運用および技術に関する重要な分析情報が得られます。しかしこれは、LUIS と QnA Maker の操作に伴って送受信されるメッセージなど、特定のボット関連イベントをキャプチャするために使用することもできます。

ボット レベルのテレメトリは、技術および運用に関するテレメトリに本質的にリンクされています。そのためお客様は、ユーザーの特定の質問に対してどのような回答が行われたか、および行われなかったかを調べることができます。

ミドルウェア コンポーネントと QnA Maker および LuisRecognizer の SDK クラスを囲むラッパー クラスとを組み合わせることで、一貫性のある一連のイベントを洗練された方法で収集できます。 これらの一貫性のあるイベントは、その後 PowerBI などのツールと共に Application Insights ツールによって使用されます。

PowerBI ダッシュボードの例は、Bot Framework ソリューション の GitHub リポジトリの一部であり、すべての仮想アシスタント テンプレートですぐにそのまま利用できます。 詳細については、[Analytics](https://github.com/Microsoft/AI/blob/master/docs/readme.md#analytics) に関するセクションを参照してください。

![分析の例](./media/enterprise-template/powerbi-conversationanalytics-luisintents.png)

## <a name="dispatcher"></a>ディスパッチャー

会話エクスペリエンスの第 1 弾で効果を上げるために使用された重要な設計パターンは、Language Understanding (LUIS) と QnA Maker を利用することでした。 LUIS では、お客様のボットがエンド ユーザーのために実行できるタスクについてのトレーニングが行われ、QnA Maker では、より一般的な知識についてのトレーニングが行われていました。

入力された発話 (質問) はすべて、分析のために LUIS にルーティングされていました。 与えられた発話の意図が特定されなかった場合、それは意図なし、としてマークされました。 その後、エンドユーザーへの回答を見つけ出すために QnA Maker が使用されていました。

このパターンはうまく機能していたものの、問題が発生し得る重要なシナリオが 2 つありました。

- LUIS モデルと QnA Maker の発話がわずかに重複することがあり、そのようなときには、QnA Maker に転送されるはずの質問の処理を LUIS が試行するかもしれないという、想定外の動作が引き起こされる可能性がありました。
- 2 つ以上の LUIS モデルがあった場合は、ボットによって各モデルが呼び出され、特定の発話の送信先を特定するためになんらかの意図評価比較が実行される必要がありました。 共通のベースライン スコアがないため、モデル間の比較が効果的に機能せず、ユーザー エクスペリエンスの低下を招いていました。

[Dispatcher](https://docs.microsoft.com/azure/bot-service/bot-builder-tutorial-dispatch?view=azure-bot-service-4.0&tabs=csaddref%2Ccsbotconfig) では、構成された各 LUIS モデルから発話を、QnA Maker から質問を抽出して、一元的なディスパッチ LUIS モデルを作成することで、この問題をうまく解決できます。

これにより、与えられた発話を処理すべき LUIS モデルまたはコンポーネントをボットが速やかに特定できるほか、QnA Maker のデータが以前のように単に意図なし、とされるのではなく、意図処理の最上位レベルで検討されるようになります。

このディスパッチ ツールでは、LUIS モデルと QnA Maker のナレッジベースとの間における混乱と重複を浮き彫りにする評価を実行して、デプロイ前に問題を明らかにすることもできます。

Dispatcher は、テンプレートを使用して作成された各プロジェクトのコアとして使用されます。 ディスパッチ モデルは、ターゲットが LUIS モデルと QnA のいずれであるかを特定するために `MainDialog` クラス内で使用されます。 LUIS の場合は、セカンダリ LUIS モデルが呼び出され、通常どおり意図とエンティティが返されます。 Dispatcher は、中断の検出にも使用されます。

![ディスパッチの例](./media/enterprise-template/dispatchexample.png)

## <a name="qna-maker"></a>QnA Maker

[QnA Maker](https://www.qnamaker.ai/) には、開発者以外のユーザーが質問と回答のペアという形式で一般的な知識のキュレーションを行える機能があります。 この知識は、FAQ データ ソースと製品マニュアルからインポートできるほか、QnA Maker ポータル内で対話形式でインポートできます。

CognitiveModels の QnA フォルダーには、[LU](https://github.com/Microsoft/botbuilder-tools/blob/master/packages/Ludown/docs/lu-file-format.md) ファイル形式の 2 つの QnA Maker モデルの例が用意されています。1 つは FAQ 用、もう 1 つは chit-chat 用です。 次に、デプロイ スクリプトの一部として [LuDown](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/Ludown) が使用され、QnA Maker JSON ファイルが作成されます。そして、[QnA Maker](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/QnAMaker) CLI (コマンド ライン) ツールによってこれが使用され、項目が QnA Maker ナレッジベースに発行されます。

![QnA のおしゃべりの例](./media/enterprise-template/qnachitchatexample.png)

## <a name="content-moderator"></a>Content Moderator

Content Moderator はオプションのコンポーネントで、不適切な可能性がある表現を検出できます。また、個人を特定できる情報 (PII) のチェックに役立ちます。 これをボットに統合すると役に立つ可能性があります。ボットは不適切な表現に反応したり、ユーザーが PII 情報を共有した場合に反応したりできるようになります。 たとえば、ボットに謝罪させたり、人間への引き継ぎを行わせたりできます。または、PII 情報が検出された場合に、ボットがテレメトリ レコードを保存しないようにできます。

TurnState オブジェクトの ```TextModeratorResult``` を通じてテキストとサーフェスをスクリーニングするミドルウェア コンポーネントが提供されます。

# <a name="next-steps"></a>次の手順
ご自身の仮想アシスタントを作成してデプロイする方法については、[作業の開始](https://github.com/Microsoft/AI/tree/master/docs#tutorials)に関するページをご覧ください。 

# <a name="additional-resources"></a>その他のリソース
仮想アシスタント テンプレートの完全なソース コードは、[GitHub](https://github.com/Microsoft/AI/) にあります。

