---
title: ナレッジ ボットを設計する - Bot Service
description: ユーザーの入力またはクエリへの応答として情報を発見して返すナレッジ ボットを設計するさまざまな方法について説明します。
author: matvelloso
ms.author: mateusv
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
ms.openlocfilehash: 988bf816d66bfb6d4140b6be4a708ae6082e1077
ms.sourcegitcommit: f8b5cc509a6351d3aae89bc146eaabead973de97
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/09/2020
ms.locfileid: "75792103"
---
# <a name="design-knowledge-bots"></a>ナレッジ ボットを設計する

事実上すべてのトピックに関する情報を提供するナレッジ ボットを設計できます。 たとえば、1 つのナレッジ ボットで "このカンファレンスではどんなボット イベントがありますか?"、"次のレゲエ ショーはいつ?"、"テーム インパラって誰?" といった質問に回答できます。 または、"オペレーティング システムの更新方法を教えて" や "パスワードはどこでリセットしますか?" のような IT 関連の質問にも答えられます。 さらには、"John Doe とは誰ですか?" または "Jane Doe のメール アドレスは?" のような連絡先についての質問にも答えます。 

ナレッジ ボットの設計のユース ケースに関係なく、基本的な目的は常に同じで、SQL データベースのリレーショナル データ、非リレーショナル ストアの JSON データ、ドキュメント ストアの PDF など、データの本体を利用してユーザーが要求した情報を探して返すことです。 

## <a name="search"></a>検索

検索機能はボットの貴重なツールです。 

最初に、"あいまい検索" により、ボットはユーザーの質問に関係があると思われる情報を返すことができ、ユーザーは正確に入力する必要はありません。 たとえば、ユーザーが音楽ナレッジ ボットに ("テーム インパラ" ではなく) "インパラ" についての情報をたずねた場合、ボットはその入力に最も関連性が高いと思われる情報を応答できます。

![ダイアログの構造](~/media/bot-service-design-pattern-knowledge-base/fuzzySearch2.png)

検索スコアは特定の検索の結果の信頼度レベルを示し、ボットはそれに従って結果を並べ替えたり、信頼レベルに基づいて通信を調整することさえできます。 たとえば、信頼レベルが高い場合、ボットは "探しているものとよく一致するイベントはこちらです:" などと応答できます。

![ダイアログの構造](~/media/bot-service-design-pattern-knowledge-base/searchScore2.png)

信頼レベルが低い場合は、ボットは "これらのイベントのどれかをお探しでしたか?" などと応答できます。

![ダイアログの構造](~/media/bot-service-design-pattern-knowledge-base/searchScore1.png)

### <a name="using-search-to-guide-a-conversation"></a>検索を使用して会話をガイドする

ボットを作成する動機が基本的な検索エンジン機能を有効にすることである場合は、そもそもボットなど必要ないかもしれません。 会話インターフェイスは、ユーザーが Web ブラウザーの一般的な検索エンジンでは得られないどのようなものを提供するのでしょうか。 

通常、ナレッジ ボットは、会話をガイドするように設計されているときが最も効果的です。 会話はユーザーとボットの間のやり取りで構成され、確認の質問、オプションの表示、結果の検証といった、基本的な検索では実行できないことを行う機会をボットに提供します。 たとえば、次のボットは、ユーザーが求めている情報が見つかるまで、データセットをファセットしてフィルター処理する会話によってユーザーをガイドします。

![ダイアログの構造](~/media/bot-service-design-pattern-knowledge-base/guidedConvo1.png)

![ダイアログの構造](~/media/bot-service-design-pattern-knowledge-base/guidedConvo2.png)

![ダイアログの構造](~/media/bot-service-design-pattern-knowledge-base/guidedConvo3.png)

![ダイアログの構造](~/media/bot-service-design-pattern-knowledge-base/guidedConvo4.png)

各手順でユーザーの入力を処理して関連するオプションを示すことで、ボットはユーザーが探している情報までユーザーを案内します。 ボットは、その情報を提供した後、今後同様の情報を探すときのより効率的な方法を示すことさえできます。 

![ダイアログの構造](~/media/bot-service-design-pattern-knowledge-base/Training.png)

### <a name="azure-search"></a>Azure Search

<a href="https://azure.microsoft.com/services/search/" target="_blank">Azure Search</a> を使うことにより、ボットが簡単に検索、ファセット、フィルターできる効率的な検索インデックスを作成できます。 Azure portal を使って作成された検索インデックスについて考えてみてください。

![ダイアログの構造](~/media/bot-service-design-pattern-knowledge-base/search3.PNG)

データ ストアのすべてのプロパティにアクセスできるようにしたいので、各プロパティを "取得可能" に設定します。 音楽家を名前で検索できるようにしたいので、**Name** プロパティを "検索可能" に設定します。 最後に、音楽家の時代でフィルターをファセットできるようにしたいので、**Eras** プロパティを "ファセット可能" かつ "フィルター可能" に設定します。 

ファセットにより、特定のプロパティについてデータ ストアに存在する値と、各値の大きさが決まります。 たとえば、次のスクリーンショットは、データ ストアに 5 つの異なる時代があることを示しています。

![ダイアログの構造](~/media/bot-service-design-pattern-knowledge-base/facet.png)

フィルターは、さらに、特定のプロパティの指定されたインスタンスのみを選択します。 たとえば、上の結果セットをフィルターして **Era** が "Romantic" である項目だけを含むようにできます。 

> [!NOTE]
> Azure Document DB、Azure Search、Microsoft Bot Framework を使って作成されたナレッジ ボットの完全な例については、<a href="https://github.com/ryanvolum/AzureSearchBot" target="_blank">サンプル ボット</a>をご覧ください。
> 
> わかりやすくするため、上の例では Azure portal を使って作成された検索インデックスを示しています。 
> インデックスはプログラムで作成することもできます。

## <a name="qna-maker"></a>QnA Maker

よくある質問 (FAQ) に回答することだけが目的のナレッジ ボットもあります。 
<a href="https://www.microsoft.com/cognitive-services/qnamaker" target="_blank">QnA Maker</a> は、このユース ケースのために特に設計されている強力なツールです。 QnA Maker には既存の FAQ サイトから質問と回答を収集する組み込み機能があり、質問と回答の独自のリストを手動で構成することもできます。 QnA Maker は自然言語処理機能を備え、予想と若干異なる言葉遣いの質問に対しても回答を提供できます。 ただし、セマンティック言語理解機能はありません。 たとえば、子犬 (puppy) が犬の種類であることは判断できません。 

QnA Maker の Web インターフェイスを使うと、3 つの質問と回答のペアから成るナレッジ ベースを構成できます。 

![ダイアログの構造](~/media/bot-service-design-pattern-knowledge-base/KnowledgeBaseConfig.png)

次に、一連の質問を行ってテストできます。 

![ダイアログの構造](~/media/bot-service-design-pattern-knowledge-base/exampleQnAConvo.png)

ボットは、ナレッジ ベースでの構成に直接マップされる質問に正しく回答します。 ただし、"can I bring my tea?" (紅茶を持ち込めますか?) という質問には正しく応答できません。これは、この質問は "can I bring my vodka?" (ウオッカを持ち込めますか?) という質問に構造が最も似ているためです。 QnA Maker が誤って回答する理由は、QnA Maker は本質的に単語の意味を理解しないからです。 "紅茶" がノンアルコール飲料の一種であることがわかりません。 そのため、"Alcohol is not allowed" (アルコールは許可されません) と応答します。

> [!TIP]
> QnA のペアを作成してテストし、会話の下にある [検査] ボタンを使用して不適切な応答に対する別の応答を選択することで、ボットを再トレーニングします。 

## <a name="luis"></a>LUIS

ナレッジ ボットによっては、ユーザーのメッセージを分析してユーザーの意図を特定できるように、自然言語処理 (NLP) 機能が必要になります。 
[Language Understanding (LUIS)](https://www.luis.ai) は、NLP 機能をボットに追加する迅速かつ効果的な手段を提供します。 LUIS では、ニーズに合っているときは常に Bing と Cortana から既存の事前構築済みモデルを使用できるだけでなく、独自の特殊なモデルを作成することもできます。 

大きなデータセットを使用する場合は、エンティティのすべてのバリエーションで NLP モデルをトレーニングできないことがあります。 たとえば、音楽再生ボットでは、ユーザーが "Play Reggae" (レゲエを再生)、"Play Bob Marley" (ボブ マーリーを再生)、"Play One Love" (ワン ラブを再生) のようなメッセージを入力することがあります。 すべてのアーティスト、ジャンル、曲名でトレーニングしていなくても、ボットはこれらの各メッセージを意図 "playMusic" にマップできますが、NLP モデルはエンティティがジャンルかアーティストか曲かを識別することはできません。 NLP モデルを使ってタイプが "音楽" の汎用エンティティを識別することで、ボットはデータ ストアでそのエンティティを検索し、そこから先に進むことができます。 

## <a name="combining-search-qna-maker-andor-luis"></a>Search、QnA Maker、LUIS の組み合わせ

Search、QnA Maker、LUIS はそれだけでも強力なツールですが、それらを組み合わせて複数の機能を提供するナレッジ ボットを構築することもできます。

### <a name="luis-and-search"></a>LUIS と Search

[前に示した](#search)音楽フェスティバル ボットの例では、ボットはラインナップを表すボタンを表示することによって会話をガイドします。 ただし、このボットは LUIS を使用して自然言語理解を組み込み、"what kind of music does Romit Girdhar play?" (Romit Girdhar が演奏する音楽の種類は何) といった質問の意図とエンティティを特定できます。 ボットはミュージシャン名を使用して Azure Search インデックスを検索できます。 
 
候補となる値が非常に多いので可能性のあるすべてのミュージシャン名でモデルをトレーニングするのは不可能ですが、十分な代表例を LUIS に提供してエンティティを正しく識別できます。  たとえば、ミュージシャンの例を提供してモデルをトレーニングすることを考えます。 

![ダイアログの構造](~/media/bot-service-design-pattern-knowledge-base/answerGenre.png)
![ダイアログの構造](~/media/bot-service-design-pattern-knowledge-base/answerGenreOneWord.png)

"what kind of music do the beatles play?" (ビートルズが演奏する音楽の種類は何) のような新しい発話でこのモデルをテストすると、LUIS は正しく意図 "answerGenre" を特定し、エンティティ "beatles" を識別します。 ただし、"what kind of music does the devil makes three play?" (the devil makes three が演奏する音楽の種類は何) のような長い質問を送信すると、LUIS は "the devil" をエンティティとして識別します。

![ダイアログの構造](~/media/bot-service-design-pattern-knowledge-base/devilMakesThreeScore.png)

基になるデータセットの代表的なエンティティ例でモデルをトレーニングすることにより、ボットの言語理解の精度を向上させることができます。 

> [!TIP]
> 一般に、エンティティ認識で識別する単語が少なすぎる場合より (たとえば、発話 "Call John Smith please" から "John" を識別する)、多すぎる場合の方が (たとえば、発話 "Call John Smith please" から "John Smith please" を識別する)、モデルの誤りとしてより適切です。 検索インデックスは、"John Smith please" の中の "please" のような関係のない単語を無視します。 

### <a name="luis-and-qna-maker"></a>LUIS と QnA Maker

ナレッジ ボットの中には、QnA Maker を使用して基本的な質問に回答し、LUIS との組み合わせによって意図を判別し、エンティティを抽出して、さらに複雑なダイアログを呼び出すものがあります。 たとえば、簡単な IT ヘルプ デスク ボットについて考えます。 このボットは、QnA Maker を使って Windows または Outlook に関する基本的な質問に回答しますが、意図を認識してユーザーとボットの間でやり取りする必要がある、パスワード リセットのようなシナリオを容易にする必要もあります。 ボットで LUIS と QnA Maker のハイブリッドを実装する方法はいくつかあります。

1. QnA Maker と LUIS の両方を同時に呼び出し、特定のしきい値のスコアを最初に返したものの情報を使ってユーザーに応答します。 
2. LUIS を最初に呼び出し、特定のしきい値スコアを満たす意図がない場合、つまり "None" 意図がトリガーされる場合は、QnA Maker を呼び出します。 または、"QnAIntent" にマップする QnA 質問の例を LUIS モデルにフィードして、QnA Maker に対する LUIS 意図を作成します。 
3. QnA Maker を最初に呼び出し、特定のしきい値スコアを満たす回答がない場合、LUIS を呼び出します。 

Bot Framework SDK では、LUIS と QnA Maker の組み込みサポートが提供されます。 これにより、LUIS と QnA Maker を使ってダイアログをトリガーしたり、質問に自動的に応答することができ、いずれかのツールへのカスタム呼び出しを実装する必要はありません。 詳しくは、[ディスパッチ ツールのチュートリアル](https://docs.microsoft.com/azure/bot-service/bot-builder-tutorial-dispatch?view=azure-bot-service-4.0)に関する記事をご覧ください。

> [!TIP]
> LUIS、QnA Maker、Azure Search の組み合わせを実装する場合は、各ツールで入力をテストして、各モデルのしきい値スコアを決定します。 LUIS、QnA Maker、Azure Search はそれぞれ異なるスコアリング条件を使ってスコアを生成するので、これらのツールによって生成されたスコアを直接比較することはできません。 さらに、LUIS と QnA Maker はスコアを正規化します。 LUIS モデルで "良" と見なされた特定のスコアが、別のモデルではそうならない可能性があります。 

## <a name="sample-code"></a>サンプル コード

- Bot Framework SDK for .NET を使用して基本的なナレッジ ボットを作成する方法がわかるサンプルについては、GitHub の<a href="https://aka.ms/qna-with-appinsights" target="_blank">ナレッジ ボットのサンプル</a>をご覧ください。 
<!-- TODO: Do not have a current bot sample to work with this
- For a sample that shows how to create more complex knowledge bots using the Bot Framework SDK for .NET, see the <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/demo-Search" target="_blank">Search-powered Bots sample</a> in GitHub.
-->

[qnamakerTemplate]: https://docs.botframework.com/azure-bot-service/templates/qnamaker/#navtitle
