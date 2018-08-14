---
title: Cognitive Services を使用してボットにインテリジェンスを追加する | Microsoft Docs
description: Microsoft Cognitive Services を使って、ボットをより便利で魅力的にするために、人工知能を追加する方法について説明します。
keywords: 言語の理解, ナレッジ抽出, 音声認識, Web 検索
author: RobStand
ms.author: rstand
manager: rstand
ms.topic: article
ms.prod: bot-framework
ms.date: 12/17/2017
ms.openlocfilehash: 2a558694cb96dd199894627146947ae1e7345183
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/18/2018
ms.locfileid: "39304092"
---
# <a name="add-intelligence-to-bots-with-cognitive-services"></a>Cognitive Services を使用してボットにインテリジェンスを追加する

コンピューター ビジョン、音声、自然言語処理、ナレッジ抽出、Web 検索の分野の専門家が開発する強力な AI アルゴリズムのコレクションは増加を続けており、Microsoft Cognitive Services により、それを利用することができるようになります。 このサービスにより、多岐にわたる AI ベースのタスクが単純化され、ほんのわずかなコード行で最新のインテリジェンス テクノロジを簡単にボットに追加できます。 この API は最新の言語とプラットフォームの大部分に統合されています。 また、API は絶えず改良と学習を続けて賢くなっているため、いつでも最新の機能をご利用になれます。 

インテリジェントなボットは、あたかも人間と同じ仕方で世界を知覚しているかのように応答します。 さまざまなソースからの情報を検出し、ナレッジを抽出し、役に立つ回答を提供できます。とりわけ、経験を積むにつれて学習するので、機能を継続的に向上させることができます。 

## <a name="language-understanding"></a>言語の理解

ユーザーとボットとの対話はほとんどが自由形式なので、ボットは自然に、かつ文脈に従って言語を理解する必要があります。 Cognitive Service Language API には、ユーザーが望むことを判断し、特定の文章内の概念やエンティティを識別し、最終的にはボットが応答として適切なアクションが取れるようにするための強力な言語モデルが備わっています。 5 つの API が、スペルチェック、センチメント検出、言語モデリング、テキストからの正確かつ豊富な分析情報の抽出などのテキスト分析機能をサポートしています。 

Cognitive Services には言語の理解のために次の API があります。

- <a href="https://www.microsoft.com/cognitive-services/en-us/language-understanding-intelligent-service-luis" target="_blank">Language Understanding Intelligent Service (LUIS)</a> は、事前に作成された、またはカスタム トレーニングされたモデルを使用して、自然言語を処理できます。 詳細については、「[ボット用 Language understanding](v4sdk/bot-builder-concept-luis.md)」を参照してください。

- <a href="https://www.microsoft.com/cognitive-services/en-us/text-analytics-api" target="_blank">Text Analytics API</a> は、テキストからセンチメント、キー フレーズ、トピック、言語を検出します。

- <a href="https://www.microsoft.com/cognitive-services/en-us/bing-spell-check-api" target="_blank">Bing Spell Check API</a> には強力なスペルチェック機能が備わっていて、氏名の違い、ブランド名、スラングを認識できます。

- <a href="https://docs.microsoft.com/en-us/azure/machine-learning/studio/text-analytics-module-tutorial" target ="_blank">Azure Machine Learning Studio のテキスト分析モデル</a>を使用すると、分類整理やテキストの前処理などのテキスト分析モデルを作成して、使用可能にできます。 こうしたモデルは、たとえば、ドキュメントの分類やセンチメント分析の問題に対応するのに役立ちます。

Microsoft Cognitive Services を使用した[言語理解][language]の詳細をご確認ください。

## <a name="knowledge-extraction"></a>ナレッジ抽出

Cognitive Services に備わっている 4 つのナレッジ API を使用すると、構造化されていないテキストで固有表現やフレーズを特定したり、個人設定された推奨事項を追加したり、ユーザー クエリの自然な解釈に基づいてオートコンプリートの候補を提供したり、個人用に設定された FAQ サービスのように学術論文や他の調査資料を検索したりできます。

- <a href="https://www.microsoft.com/cognitive-services/en-us/entity-linking-intelligence-service" target="_blank">Entity Linking Intelligence Service</a> は、構造化されていないテキストに、テキストで言及されている関連エンティティで注釈を付けます。 コンテキストによっては、同じ語やフレーズが別の物事を指すことがあります。 このサービスは、提供されたテキストのコンテキストを理解し、テキスト内の各エンティティを特定します。    

- <a href="https://www.microsoft.com/cognitive-services/en-us/knowledge-exploration-service" target="_blank">Knowledge Exploration Service</a> はユーザー クエリを自然言語で解釈し、注釈付きの解釈を返します。それにより、ユーザーの入力内容を予測する高性能な検索やオートコンプリートのエクスペリエンスを実現します。 迅速なクエリ補完候補や、予測的なクエリの絞り込みは、独自のデータやアプリケーション固有の文法に基づいて行われ、ユーザーが迅速にクエリを実行できるようにします。    

- <a href="https://www.microsoft.com/cognitive-services/en-us/academic-knowledge-api" target="_blank">Academic Knowledge API</a> は <a href="https://www.microsoft.com/en-us/research/project/microsoft-academic-graph/" target="_blank">Microsoft Academic Graph</a> から学術研究論文、著者、定期刊行物、会議、トピック、大学を返します。 Knowledge Exploration Service のドメイン固有の例として作成された Academic Knowledge API は、数億にもおよぶ研究関連エンティティを対象とする検索機能を備えた、グラフ状のダイアログによってナレッジ ベースを提供します。 トピック、教授、大学、会議を検索すると、API によって、関連するパブリケーションとエンティティが提供されます。 文法は、「機械学習に関する Michael Jordan による 2010 年より後の論文」などの自然形式のクエリもサポートしています。

- <a href="https://qnamaker.ai" target="_blank">QnA Maker</a> は無料の使いやすい REST API および Web ベースのサービスであり、ユーザーの質問に自然な会話形式で応答できるよう AI をトレーニングします。 QnA Maker は、最適化された機械学習のロジックを持つとともに、業界をリードする言語処理機能を統合する機能を備えており、質問と回答のペアなどの半構造化データから、役立つ回答を抽出できます。

Microsoft Cognitive Services を使用した[ナレッジ抽出][knowledge]の詳細をご確認ください。

## <a name="speech-recognition-and-conversion"></a>音声認識と変換

Speech API を使用すると、高度な音声スキルをボットに追加でき、音声からテキスト、テキストから音声への変換や、話者認識などの、業界をリードするアルゴリズムを利用できます。 Speech API は言語と音響の組み込みモデルを使用して、幅広いシナリオに高精度で対応できます。 

さらにカスタマイズすることが必要なアプリケーションの場合、Custom Recognition Intelligent Service (CRIS) を使用できます。 このサービスを使用すると、アプリケーションの語彙やユーザーの話し方にさえ合わせて、音声認識機能の言語と音響のモデルを調整できます。

Cognitive Services では、音声を処理または合成するために次の 3 つの Speech API を利用できます。

- <a href="https://www.microsoft.com/cognitive-services/en-us/speech-api" target="_blank">Bing Speech API</a> には、音声からテキストおよびテキストから音声に変換する機能があります。
- <a href="https://www.microsoft.com/cognitive-services/en-us/custom-recognition-intelligent-service-cris" target="_blank">Custom Recognition Intelligent Service (CRIS)</a> を使用すると、カスタムの音声認識モデルを作成し、アプリケーションの語彙やユーザーの話し方に応じて音声からテキストへの変換を調整できます。
- <a href="https://www.microsoft.com/cognitive-services/en-us/speaker-recognition-api" target="_blank">Speaker Recognition API</a> は、声による話者の識別と検証を行えます。

次のリソースには、ボットに音声認識機能を追加するための他の情報が示されています。

* [アプリのボット会話に関するビデオによる概要](https://channel9.msdn.com/events/Build/2017/P4114)
* [UWP アプリまたは Xamarin アプリの Microsoft.Bot.Client ライブラリ](https://aka.ms/BotClient)
* [ボット クライアント ライブラリのサンプル](https://aka.ms/BotClientSample)
* [音声対応 Web チャット クライアント](https://aka.ms/BFWebChat)

Microsoft Cognitive Services を使用した[音声認識と変換][speech]に関する詳細をご確認ください。

## <a name="web-search"></a>Web 検索

Bing Search API を使用すると、ボットにインテリジェントな Web 検索機能を追加できます。 ほんのわずかなコード行で、無数の Web ページ、イメージ、ビデオ、ニュース、他の結果の種類にアクセスできます。 より関連性が高まるよう、API を構成して、地理的な場所、市場、言語ごとに結果を返すことができます。 サポートされている検索パラメーターを使用して、検索をさらにカスタマイズできます。Safesearch で成人向けコンテンツをフィルターで除外したり、Freshness で特定の日付に従って結果を返したりすることができます。

Cognitive Services では 5 つの Bing Search API を使用できます。

- <a href="https://www.microsoft.com/cognitive-services/en-us/bing-web-search-api" target="_blank">Web Search API</a> では、1 回の API 呼び出しで Web、イメージ、ビデオ、ニュース、関連する検索結果を提供できます。

- <a href="https://www.microsoft.com/cognitive-services/en-us/bing-image-search-api" target="_blank">Image Search API</a> は拡張メタデータ (ドミナント カラーや画像の種類など) を使用して画像結果を返し、結果をカスタマイズするためのいくつかのイメージ フィルターをサポートします。

- <a href="https://www.microsoft.com/cognitive-services/en-us/bing-video-search-api" target="_blank">Video Search API</a> は、リッチ メタデータ (ビデオのサイズ、品質、価格など) に基づくビデオの結果やビデオのプレビューを取得し、結果をカスタマイズするためのいくつかのビデオ フィルターをサポートします。

- <a href="https://www.microsoft.com/cognitive-services/en-us/bing-news-search-api" target="_blank">News Search API</a> は、検索クエリと一致する、またはインターネット上で現在トレンドになっている世界中のニュース記事を検出します。

- <a href="https://www.microsoft.com/cognitive-services/en-us/bing-autosuggest-api" target="_blank">Autosuggest API</a> は、クエリ検索が迅速にかつ入力操作が少なくて済むように、迅速なクエリ補完候補を示します。 

Microsoft Cognitive Services を使用した [Web 検索][search]の詳細をご確認ください。

## <a name="image-and-video-understanding"></a>イメージやビデオの把握

Vision API は、イメージとビデオを高度に把握するスキルをボットに提供します。 最新のアルゴリズムを使用すると、イメージやビデオを処理し、アクションに変換できる情報を取得できます。 たとえば、オブジェクト、人の顔、年齢、性別、さらには感情でさえ認識する目的で使用できます。 

Vision API は多様なイメージ把握機能をサポートしています。 成人向けコンテンツやきわどいコンテンツを識別し、カラーを見極めてアクセントを付けたり、イメージのコンテンツをカテゴリー化したりします。また、光学式文字認識を実行したり、イメージについて完全な英語文章で説明を記したりできます。 また、Vision API では、イメージやビデオのサムネイルのインテリジェントな生成や、ビデオ出力の安定化など、イメージとビデオに関するいくつかの処理機能もサポートしています。

Cognitive Services には、イメージまたはビデオの処理に使用できる次の 4 つの API が備わっています。

- <a href="https://www.microsoft.com/cognitive-services/en-us/computer-vision-api" target="_blank">Computer Vision API</a> は、イメージに関する豊富な情報 (オブジェクトや人) を抽出したり、イメージに成人向けコンテンツまたはきわどいコンテンツが含まれているかどうかを判別したり、イメージ内のテキストを (OCR を使用して) 処理したりします。

- <a href="https://www.microsoft.com/cognitive-services/en-us/emotion-api" target="_blank">Emotion API</a> は人間の顔を分析し、人間の感情として考えられる 8 つのカテゴリの感情を識別できます。

- <a href="https://www.microsoft.com/cognitive-services/en-us/face-api" target="_blank">Face API</a> は人間の顔を検出し、類似した顔と比較して、視覚的な類似性に基づいて人々をグループに整理することさえできます。

- <a href="https://www.microsoft.com/cognitive-services/en-us/video-api" target="_blank">Video API</a> はビデオを分析および処理し、ビデオ出力を安定化させ、モーションを検出し、顔を追跡します。また、ビデオのモーション サムネイル概要を生成することもできます。

Microsoft Cognitive Services を使用した[イメージとビデオの把握][vision]の詳細をご確認ください。

## <a name="additional-resources"></a>その他のリソース

各製品の総合的なドキュメントと、対応する API リファレンスについては、<a href="https://docs.microsoft.com/azure/cognitive-services" target="_blank">Cognitive Services ドキュメント</a>をご覧ください。

[language]: https://docs.microsoft.com/en-us/azure/cognitive-services/luis/home
[search]: https://docs.microsoft.com/en-us/azure/cognitive-services/bing-web-search/search-the-web
[vision]: https://docs.microsoft.com/en-us/azure/cognitive-services/computer-vision/home
[knowledge]: https://docs.microsoft.com/en-us/azure/cognitive-services/kes/overview
[speech]: https://docs.microsoft.com/en-us/azure/cognitive-services/speech/home
[location]: https://docs.microsoft.com/en-us/azure/cognitive-services/
