---
title: 新機能 (アーカイブ) - Bot Service
description: Bot Framework の新機能について説明します。
keywords: Bot Framework, Azure Bot Service
author: kamrani
ms.author: kamrani
manager: kamrani
ms.topic: conceptual
ms.service: bot-service
ms.date: 11/28/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: f3d33f9f3f19a32b7b01aa92b809b4e16cae76e1
ms.sourcegitcommit: 9d77f3aff9521d819e88efd0fbd19d469b9919e7
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/16/2020
ms.locfileid: "80648172"
---
# <a name="whats-new-november-2019"></a>2019 年 11 月の新機能

[!INCLUDE[applies-to](includes/applies-to.md)]

Bot Framework SDK v4 は [オープン ソース SDK](https://github.com/microsoft/botframework-sdk/#readme) です。この SDK を使用すると、開発者が好きなプログラミング言語を使用して、高度な会話をモデル化して構築できます。

この記事では、Bot Framework と Azure Bot Service の主な新機能と機能強化をまとめます。


|   | C#  | JS  | Python |  Java | 
|---|:---:|:---:|:------:|:-----:|
|Release |[4.6 一般公開][1] | [4.6 一般公開][2] | [ベータ 4][3] | [プレビュー 3][3a]|
|Docs | [docs][5] |[docs][5] |  | |
|サンプル |[.NET Core][6]、[WebAPI][10] |[Node.js][7]、[TypeScript][8]、[es6][9]  | | | 

#### <a name="bot-framework-sdk-for-microsoft-teams-ga"></a>Bot Framework SDK for Microsoft Teams (一般公開)
Bot Framework SDK v4.6 リリースでは、Teams のボット作成のサポートが完全に統合されているため、ユーザーはチャネルまたはグループ チャットによる会話でそれらを使用できます。 ボットをチームまたはチャットに追加することによって、会話のすべてのユーザーが、ボット機能を会話中に利用できるようになります。  [[ドキュメント](https://docs.microsoft.com/azure/bot-service/bot-builder-basics-teams)]

#### <a name="bot-framework-for-power-virtual-agent-preview"></a>Bot Framework for Power Virtual Agent (プレビュー)

Power Virtual Agent は、ビジネスユーザーが、特定の AI サービスをコーディングしたり管理したりする必要なく、SaaS エクスペリエンスを構築する UI ベースのボット内にボットを作成できるように設計されています。 Power Virtual Agents は、Microsoft Bot Framework を使用して拡張できるため、開発者やビジネス ユーザーは組織のボットを協力して構築することができます。 [[ドキュメント](https://docs.microsoft.com/dynamics365/ai/customer-service-virtual-agent/overview)]


#### <a name="bot-framework-sdk-for-skills-preview"></a>Bot Framework SDK for Skills (プレビュー)

- **ボットのスキル**:ボットに機能を追加するために、再利用可能な会話スキルを作成します。 カレンダー、電子メール、仕事、目的地、自動車、天気、ニュースのスキルなど、事前に構築されたスキルを活用できます。 スキルには、必要に応じてカスタマイズおよび拡張するために提供される言語モデル、ダイアログ、QnA、統合コードが含まれます。 [[ドキュメント](https://microsoft.github.io/botframework-solutions/overview/skills/)]

- **Power Virtual Agent のスキル - 近日公開**:Power Virtual Agents でビルドされたボットの場合、Bot Framework と Azure Cognitive Services を使用してこれらのボットの新しいスキルを構築できます。新しいボットをゼロから作成する必要はありません。 

#### <a name="adaptive-dialogs-preview"></a>アダプティブ ダイアログ (プレビュー)
アダプティブ ダイアログにより、開発者はコンテキストとイベントに基づいて会話フローを動的に更新できます。 これは、会話の途中で会話コンテキストの切り替えや中断を処理する場合に特に便利です。 [[ドキュメント][48] | [C# サンプル][49]] 

#### <a name="language-generation-preview"></a>言語生成 (プレビュー)
言語生成により、開発者は、1 つのフレーズに対して複数のバリエーションを定義したり、コンテキストに基づいて単純な式を実行したり、会話メモリを参照したりといった、ボットの応答を生成するためのロジックを分離することができます。 [[ドキュメント][44] | [C# サンプル][45]]

#### <a name="common-expression-language-preview"></a>一般的な式言語 (プレビュー)
一般的な式言語により、実行時に条件ベースのロジックの結果を評価できます。 一般的な言語は、Bot Framework SDK と、アダプティブ ダイアログや言語生成などの会話 AI コンポーネントで使用できます。 [[ドキュメント][40] | [API][41]]

## <a name="whats-new-july-2019"></a>新機能 (2019 年 7 月)

[!INCLUDE[applies-to](includes/applies-to.md)]

Bot Framework SDK v4 は [オープン ソース SDK][1a] です。この SDK を使用すると、開発者が好きなプログラミング言語を使用して、高度な会話をモデル化して構築できます。

この記事では、Bot Framework と Azure Bot Service の主な新機能と機能強化をまとめます。

|   | C#  | JS  | Python |   
|---|:---:|:---:|:------:|
|SDK |[4.5][1] | [4.5][2] | [4.4.0b2 (プレビュー)][3] | 
|Docs | [docs][5] |[docs][5] |  | |
|サンプル |[.NET Core][6]、[WebAPI][10] |[Node.js][7]、[TypeScript][8]、[es6][9]  | [Python][11] | | 


### <a name="bot-framework-channels"></a>Bot Framework チャネル
- [Direct Line Speech (パブリック プレビュー)](https://aka.ms/streaming-extensions) | [ドキュメント](https://docs.microsoft.com/azure/bot-service/directline-speech-bot?view=azure-bot-service-4.0):Bot Framework と Microsoft の Speech Services は、WebSocket を使用して、クライアントからボット アプリケーションに双方向にストリーミングされた音声とテキストを有効にするチャネルを提供します。  

- [Direct Line App Service 拡張機能 (パブリック プレビュー)](https://portal.azure.com) | [ドキュメント](https://aka.ms/directline-ase):Direct Line API を使用してクライアントがボットに直接接続できるようにする Direct Line のバージョン。 これには、パフォーマンスの向上や分離の強化など、多くの利点があります。 Direct Line App Service 拡張機能は、Azure App Service Environment 内でホストされるものも含め、すべての Azure App Service で利用できます。 Azure App Service Environment は分離を提供し、VNet 内での作業に最適です。 VNet を使用すると、Azure に独自のプライベート空間を作成できます。VNet は分離やセグメント化などの重要な利点を提供するため、クラウド ネットワークにとって重要です。 

### <a name="bot-framework-sdk"></a>Bot Framework SDK
- [アダプティブ ダイアログ (SDK v4.6 プレビュー)](https://github.com/Microsoft/BotBuilder-Samples/tree/master/experimental/adaptive-dialog#readme) | [ドキュメント](https://github.com/Microsoft/BotBuilder-Samples/tree/master/experimental/adaptive-dialog/docs) | [C# サンプル](https://github.com/Microsoft/BotBuilder-Samples/tree/master/experimental/adaptive-dialog/csharp_dotnetcore):アダプティブ ダイアログにより、開発者はコンテキストとイベントに基づいて会話フローを動的に更新できるようになりました。 これは、会話の途中で会話コンテキストの切り替えや中断を処理する場合に特に便利です。 
  
- [Bot Framework Python SDK (プレビュー 2)](https://github.com/microsoft/botbuilder-python) | [サンプル](https://github.com/Microsoft/botbuilder-python/tree/master/samples):Python SDK で OAuth、プロンプト、CosmosDB がサポートされるようになり、SDK 4.5 のすべての主要機能が含まれるようになりました。 また、SDK の新機能について学習するのに役立つサンプルも用意されています。

### <a name="bot-framework-testing"></a>Bot Framework のテスト
- [ドキュメント](https://aka.ms/testing-framework) | 単体テストのパッケージ ([C#](https://aka.ms/nuget-botbuilder-testing)/ [JavaScript](https://aka.ms/npm-botbuilder-testing)) | [C# サンプル](https://aka.ms/cs-core-test-sample) | [JS サンプル](https://aka.ms/js-core-test-sample):より優れたテスト ツールを求めるお客様や開発者のご要望にお応えするため、7 月版の SDK では、新しい単体テスト機能が導入されています。 Microsoft.Bot.Builder.testing パッケージを使用すると、ボットの単体テスト ダイアログのプロセスが簡略化されます。  

- [チャネルのテスト](https://github.com/Microsoft/BotFramework-Emulator/releases) | [ドキュメント](https://aka.ms/channel-testing): 

Bot Inspector は、Microsoft ビルド 2019 で導入された Bot Framework Emulator の新機能であり、Microsoft Teams、Slack、Cortana などのチャネルでボットをデバッグおよびテストできます。 特定のチャネルでボットを使用すると、ボットが受信したメッセージ データを検査できる Bot Framework Emulator に、メッセージがミラー化されます。 さらに、チャネルとボットの間の特定のターンに対するボットのメモリ状態のスナップショットも表示されます。

### <a name="web-chat"></a>Web チャット
- 企業のお客様からのご要望に基づき、ボットを使用してエンタープライズ アプリ上のリソースへのアクセスをユーザーに承認する方法を示す [Web チャット サンプル](https://github.com/microsoft/BotFramework-WebChat/tree/master/samples/19.a.single-sign-on-for-enterprise-apps#single-sign-on-demo-for-enterprise-apps-using-oauth)を追加しました。 OAuth と Microsoft Graph および GitHub API の相互運用性を示すために、2 種類のリソースが使用されています。

### <a name="solutions"></a>ソリューション
- [仮想アシスタント ソリューション アクセラレータ](https://github.com/Microsoft/botframework-solutions#readme):高度な会話エクスペリエンスの構築に役立つ、一連のテンプレート、ソリューション アクセラレータ、スキルを提供します。 Direct Line Speech および仮想アシスタントと統合する仮想アシスタント用の新しい Android アプリ クライアント。デバイス クライアントがどのように仮想アシスタントと対話し、アダプティブ カードをレンダリングするかを示します。 更新プログラムには、Direct Line Speech と Microsoft Teams のサポートも含まれています。
  
- [Dynamics 365 Virtual Agent for Customer Service (パブリック プレビュー)](https://dynamics.microsoft.com/en-us/ai/virtual-agent-for-customer-service/):このパブリック プレビューでは、インテリジェントで高度な適応性を備えた仮想エージェントを使用して、優れたカスタマー サービスを提供できます。 カスタマー サービスのエキスパートは、AI による分析情報を使用してボットを簡単に作成し、強化することができます。
  
- [Chat for Dynamics 365](https://www.powerobjects.com/powerpacks/powerchat/):Chat for Dynamics 365 には、サポート エージェントとエンド ユーザーが効率的に対話し、高い生産性を維持できるようにするための機能がいくつか用意されています。 Microsoft Dynamics 365 内で Web サイトの訪問者とのライブ チャットおよび会話の追跡を行います。

## <a name="whats-new-may-2019"></a>新機能 (2019 年 5 月)

|   | C#  | JS  | Python |  Java | 
|---|:---:|:---:|:------:|:-----:|
|SDK |[4.4.3][1] | [4.4.0][2] | [4.4.0b1 (プレビュー)][3] | [4.0.0a6 (プレビュー)][3a]|
|Docs | [docs][5] |[docs][5] |  | |
|サンプル |[.NET Core][6]、[WebAPI][10] |[Node.js][7]、[TypeScript][8]、[es6][9]  | [Python][11] | | 

<a name="V4-whats-new"></a>
## <a name="bot-framework-sdk-new-in-preview"></a>Bot Framework SDK (新機能。 プレビュー段階)

- [アダプティブ ダイアログ][47] | [ドキュメント][48] | [C# のサンプル][49]:アダプティブ ダイアログを使用すると、開発者が、会話の進行中に動的に変更できる会話を構築できます。  開発者はこれまで会話のフロー全体を事前に細かく計画していましたが、これでは会話の柔軟性が制限されます。  アダプティブ ダイアログにより柔軟性が高まり、会話の進行中に、コンテキストの変更や、会話への新しいステップやサブダイアログの挿入に対応できます。 

- [言語生成][43] | [ドキュメント][44] | [C# のサンプル][45]:言語生成により、開発者は、コードやリソース ファイルから埋め込み文字列を抽出し、言語生成のランタイムとファイル形式を管理できます。  また、お客様は、言語生成を使用して、語句のバリエーションを複数定義したり、コンテキストに基づいて単純な式を実行したり、会話のメモリを参照したりできます。Microsoft では今後、より自然な会話エクスペリエンスにつながる機能を追加していきます。

- [一般的な式言語][40] | [API][41]:一般的な式言語は、ボットの会話を強化するためにアダプティブ ダイアログと言語生成の両方で使用されています。

## <a name="botkit"></a>Botkit
[Botkit][100] は、チャット ボット、アプリ、および主要なメッセージング プラットフォーム用カスタム統合を構築するための開発者ツールおよび SDK です。 Botkit ボットでは、`hear()` でトリガーが実行され、`ask()` で質問が行われ、`say()` で応答します。 開発者はこの構文を使用してダイアログを構築できます。現在これは最新バージョンのすべての Bot Framework SDK と互換性があります。 

さらに、Botkit には 6 つのプラットフォーム アダプターが用意されているため、Javascript ボット アプリケーションと次のメッセージング プラットフォームが直接通信できます: [Slack][102]、[Webex Teams][103]、[Google Hangouts][104]、[Facebook Messenger][105]、[Twilio][106]、および [Web チャット][107]。

Botkit は、Microsoft Bot Framework に含まれ、[MIT のオープン ソース ライセンス][101]でリリースされます

## <a name="bot-framework-solutions-new-in-preview"></a>Bot Framework ソリューション (新機能。 プレビュー段階)

[Bot Framework ソリューション リポジトリ](https://github.com/Microsoft/AI#readme)は、一連のテンプレート、ソリューション アクセラレータ、スキルのホームであり、高度なアシスタント的会話エクスペリエンスの構築に役立ちます。

| 名前 | 説明 |  
|:------------|:------------| 
|[**仮想アシスタント**](https://github.com/Microsoft/AI/tree/master/docs#virtual-assistant) | お客様の間で、自社のブランドに合わせてカスタマイズし、顧客に合わせてパーソナライズして、幅広いキャンバスやデバイスで使用できる会話アシスタントを提供したい、という声が高まっています。 <br/><br/> Enterprise Template により、基本的な会話の意図、ディスパッチ統合、QnA Maker、Application Insights、自動デプロイなど、新しいボット プロジェクトの作成が大幅に簡素化されます。|
|[**スキル**](https://github.com/Microsoft/AI/blob/master/docs/overview/skills.md)| 開発者が会話エクスペリエンスを作成するには、"スキル" と呼ばれる再利用可能な会話機能を結合します。 スキル自体がボットであり、リモートで呼び出されます。また、新しいスキルの作成を容易にするためのスキル開発者向けテンプレート (.NET、TS) もあります。 
|[**分析**](https://github.com/Microsoft/AI/blob/master/docs/readme.md#analytics)| 会話型 AI 分析ソリューションでは、ご自身のボットの正常性と動作に関する主な分析情報を取得します。 利用可能なテレメトリ、Application Insights のサンプル クエリ、および Power BI ダッシュボードを確認することで、お使いのボットとユーザーとの幅広い会話を理解します。 |

## <a name="azure-bot-service"></a>Azure Bot Service
Azure Bot Service を使用すると、データの完全な所有権と管理により、エンタープライズ レベルのインテリジェントなボットをホストできます。 開発者は自身のボットを登録して、Microsoft Teams、Cortana、Web チャットなどのユーザーに接続できます。 [Azure][27]  |  [ドキュメント][28] | [チャネルに接続][29] 

* **Direct Line JS クライアント**: Azure Bot Service で Direct Line チャネルを使用する必要がある場合、Web チャット クライアントを使用していないときは、ご自身のカスタム アプリケーションで Direct Line JS クライアントを使用できます。 詳細については、[GitHub][30] にアクセスしてください。

<a name="ABS-whats-new"></a>

* **新機能。Direct Line Speech チャネル**: Bot Framework と Microsoft の Speech Services を組み合わせて、クライアントからボット アプリケーションに双方向にストリーミングされた音声とテキストを有効にするチャネルを提供しています。  詳細については、[音声チャネルをご自身のボットに](https://docs.microsoft.com/azure/bot-service/directline-speech-bot?view=azure-bot-service-4.0)追加する方法をご覧ください。


## <a name="bot-framework-emulator"></a>Bot Framework エミュレーター
[Bot Framework Emulator][60] はクロスプラットフォーム デスクトップ アプリケーションです。このアプリケーションを使用すると、ボット開発者が、Bot Framework SDK を使って構築されたボットをテストし、デバッグできます。 Bot Framework Emulator を使用して、ご自身のマシンでローカルで実行されているボットをテストしたり、リモートで実行されているボットに接続したりできます。

- [最新バージョンをダウンロード][61] | [ドキュメント][62]

<a name="Emulator-whats-new"></a>
### <a name="bot-inspector-new-in-preview"></a>Bot Inspector (新機能。 プレビュー段階)

Bot Framework Emulator の新しい Bot Inspector 機能のベータ版がリリースされました。 これにより、Microsoft Teams、Slack、Cortana、Facebook Messenger などのチャネルで、ご自身の Bot Framework SDK v4 ボットをデバッグし、テストできます。会話がある場合は、ボットが受信したメッセージ データを検査できる Bot Framework Emulator に、メッセージがミラー化されます。 さらに、チャネルとボットの間の特定のターンに対するボット状態のスナップショットも表示されます。 詳細については、[Bot Inspector](https://github.com/Microsoft/BotFramework-Emulator/blob/master/content/CHANNELS.md) に関するページをご覧ください。


## <a name="related-services"></a>関連サービス

### <a name="language-understanding"></a>Language Understanding 
自然言語エクスペリエンスを構築する機械学習ベースのサービス。 継続的に改良される、エンタープライズ対応のカスタム モデルをすばやく作成できます。 [Language Understanding Service (LUIS)][30] を使用すると、アプリケーションが人の発言の意図を認識できるようになります。

<a name="LUIS-whats-new"></a>

- **新機能。役割、外部エンティティ、および動的エンティティ**: 開発者がテキストからより詳細な情報を抽出できる機能がいくつか LUIS に追加され、ユーザーがより少ない労力で、よりインテリジェントなソリューションを構築できるようになりました。 また、ロールがすべてのエンティティ型に拡張されたため、同じエンティティを、コンテキストに基づいてさまざまなサブタイプで分類できます。 開発者は LUIS での操作をより細かく制御できるようになりました。たとえば、動的なリストや外部エンティティを使って、実行時にモデルを特定および更新できます。 予測時、リスト エンティティへの追加が動的リストを使って行われるため、ユーザー固有の情報を正確に一致させることができます。 外部エンティティでは補助的なエンティティ抽出が個別に実行され、その情報は、他のモデルに対する強力なシグナルとして LUIS に追加できます。

- **新機能。Analytics ダッシュボード**: LUIS の視覚的に豊かで包括的かつ詳細なダッシュボードがリリースされます。 そのわかりやすいデザインには、アプリケーションの設計時にユーザーのほとんどが直面する一般的な問題が強調表示され、その問題の解決方法が簡単に説明されています。これは、ユーザーが、モデルの品質、潜在的なデータの問題、およびベスト プラクティスを採用するためのガイダンスに関する、さらに詳しい分析情報を得るうえで役立ちます。

[ドキュメント][31] | [ボットへの自然言語の理解の追加][32] 

### <a name="qna-maker"></a>QnA Maker
[QnA Maker][33] は、データに基づく対話型の Q&A レイヤーを作成できるクラウドベースの API サービスです。 QnA Maker を使用すると、FAQ URL、構造化されたドキュメント、製品マニュアル、または本文に基づいて、単純な質疑応答ボットを数分でビルド、トレーニング、および公開できます。

<a name="QnA-whats-new"></a>

- **新機能。抽出パイプライン**: URL、ファイル、および SharePoint から階層情報を抽出できるようになりました
- **新機能。インテリジェンス**: コンテキストに応じたランキング モデル、アクティブ ラーニング提案
- **新機能。会話**: QnA Maker のマルチターンの会話。

[ドキュメント][34]  | [ボットへの qnamaker の追加][35] 

### <a name="speech-services"></a>Speech Services
[Speech Services](https://docs.microsoft.com/azure/cognitive-services/speech-service/) で音声がテキストに変換され、統合音声サービスにより音声翻訳とテキスト読み上げが実行されます。 音声サービスを使用すると、音声をご自身のボットに統合し、カスタム ウェイク ワードを作成して、複数言語での作成が可能です。

### <a name="adaptive-cards"></a>Adaptive Cards (アダプティブ カード)
[アダプティブ カード](https://adaptivecards.io)は、一般的かつ一貫性のある方法でカード コンテンツを変換するための開発者向けオープン標準です。Bot Framework の開発者は、これを使用して、優れたクロスチャネル会話エクスペリエンスを作成できます。

## <a name="additional-information"></a>関連情報
- 詳細については、[GitHub ページ](https://github.com/Microsoft/botframework/blob/master/whats-new.md#whats-new)をご覧ください。

[1]:https://github.com/Microsoft/botbuilder-dotnet/#packages
[1a]:https://github.com/microsoft/botframework-sdk/#readme
[2]:https://github.com/Microsoft/botbuilder-js#packages
[3]:https://github.com/Microsoft/botbuilder-python#packages
[3a]:https://github.com/Microsoft/botbuilder-java#packages
[5]:https://docs.microsoft.com/azure/bot-service/?view=azure-bot-service-4.0
[6]:https://github.com/Microsoft/BotBuilder-Samples/tree/master/samples/csharp_dotnetcore
[7]:https://github.com/Microsoft/BotBuilder-Samples/tree/master/samples/javascript_nodejs
[8]:https://github.com/Microsoft/BotBuilder-Samples/tree/master/samples/typescript_nodejs
[9]:https://github.com/Microsoft/BotBuilder-Samples/tree/master/samples/javascript_es6
[10]:https://github.com/Microsoft/BotBuilder-Samples/tree/master/samples/csharp_webapi
[11]:https://github.com/Microsoft/botbuilder-python/tree/master/samples

[18]:https://github.com/Microsoft/botbuilder-tools/tree/master/packages/LUIS#readme
[19]:https://github.com/Microsoft/botbuilder-tools/tree/master/packages/QnAMaker#readme

[27]:https://azure.microsoft.com/services/bot-service/
[28]:https://docs.microsoft.com/azure/bot-service/bot-service-overview-introduction?view=azure-bot-service-4.0
[29]:https://docs.microsoft.com/azure/bot-service/bot-service-manage-channels?view=azure-bot-service-4.0

[30]:https://www.luis.ai
[31]:https://docs.microsoft.com/azure/cognitive-services/LUIS/Home
[32]:https://docs.microsoft.com/azure/bot-service/bot-builder-howto-v4-luis?view=azure-bot-service-4.0&branch=pr-en-us-1325&tabs=csharp
[33]:https://www.qnamaker.ai/
[34]:https://aka.ms/what-is-qnamaker
[35]:https://docs.microsoft.com/azure/bot-service/bot-builder-howto-qna?view=azure-bot-service-4.0&branch=pr-en-us-1325&tabs=cs

[40]:https://github.com/Microsoft/BotBuilder-Samples/tree/master/experimental/common-expression-language#readme
[41]:https://github.com/Microsoft/BotBuilder-Samples/blob/master/experimental/common-expression-language/api-reference.md
[43]:https://github.com/Microsoft/BotBuilder-Samples/tree/master/experimental/language-generation#readme
[44]:https://github.com/Microsoft/BotBuilder-Samples/tree/master/experimental/language-generation/docs
[45]:https://github.com/Microsoft/BotBuilder-Samples/tree/master/experimental/language-generation/csharp_dotnetcore
[46]:https://github.com/Microsoft/BotBuilder-Samples/tree/master/experimental/language-generation/javascript_nodejs/13.core-bot
[47]:https://github.com/Microsoft/BotBuilder-Samples/tree/master/experimental/adaptive-dialog#readme
[48]:https://github.com/Microsoft/BotBuilder-Samples/tree/master/experimental/adaptive-dialog/docs
[49]:https://github.com/Microsoft/BotBuilder-Samples/tree/master/experimental/adaptive-dialog/csharp_dotnetcore
[50]:https://github.com/Microsoft/BotBuilder-Samples/tree/master/experimental/adaptive-dialog/declarative

[60]:https://github.com/Microsoft/BotFramework-Emulator#readme
[61]:https://github.com/Microsoft/BotFramework-Emulator/releases/latest
[62]:https://docs.microsoft.com/azure/bot-service/bot-service-debug-emulator?view=azure-bot-service-4.0

[100]:https://github.com/howdyai/botkit#readme
[101]:https://github.com/howdyai/botkit/blob/master/LICENSE.md
[102]:https://github.com/howdyai/botkit/tree/master/packages/botbuilder-adapter-slack#readme
[103]:https://github.com/howdyai/botkit/tree/master/packages/botbuilder-adapter-webex#readme
[104]:https://github.com/howdyai/botkit/tree/master/packages/botbuilder-adapter-hangouts#readme
[105]:https://github.com/howdyai/botkit/tree/master/packages/botbuilder-adapter-facebook#readme
[106]:https://github.com/howdyai/botkit/tree/master/packages/botbuilder-adapter-twilio-sms#readme
[107]:https://github.com/howdyai/botkit/tree/master/packages/botbuilder-adapter-web#readme
