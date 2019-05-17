---
title: 新機能 | Microsoft Docs
description: Bot Framework の新機能について説明します。
keywords: Bot Framework, Azure Bot Service
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: get-started-article
ms.service: bot-service
ms.subservice: abs
ms.date: 05/05/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: ded465802ff3e16563dd56998e4114d6bd98ad5f
ms.sourcegitcommit: 3e3c9986b95532197e187b9cc562e6a1452cbd95
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 05/06/2019
ms.locfileid: "65047889"
---
# <a name="whats-new-in-bot-framework"></a>Bot Framework の新機能
Bot Framework SDK v4 は [オープン ソース SDK][1a] です。この SDK を使用すると、開発者が好きなプログラミング言語を使用して、高度な会話をモデル化して構築できます。

この記事では、Bot Framework と Azure Bot Service の主な新機能と機能強化をまとめます。

|   | C#  | JS  | Python |  Java | 
|---|:---:|:---:|:------:|:-----:|
|SDK |[4.4.3][1] | [4.4.0][2] | [4.4.0b1 (プレビュー)][3] | [4.0.0a6 (プレビュー)][3a]|
|ドキュメント | [ドキュメント][5] |[ドキュメント][5] |  | |
|サンプル |[.NET Core][6]、[WebAPI][10] |[Node.js][7]、[TypeScript][8]、[es6][9]  | [Python][111] | | 

[1a]:https://github.com/microsoft/botframework-sdk/#readme
[1]:https://github.com/Microsoft/botbuilder-dotnet/#packages
[2]:https://github.com/Microsoft/botbuilder-js#packages
[3]:https://github.com/Microsoft/botbuilder-python#packages
[3a]:https://github.com/Microsoft/botbuilder-java#packages
[4]:https://github.com/Microsoft/botbuilder-java#packages
[5]:https://docs.microsoft.com/en-us/azure/bot-service/?view=azure-bot-service-4.0
[6]:https://github.com/Microsoft/BotBuilder-Samples/tree/master/samples/csharp_dotnetcore
[7]:https://github.com/Microsoft/BotBuilder-Samples/tree/master/samples/javascript_nodejs
[8]:https://github.com/Microsoft/BotBuilder-Samples/tree/master/samples/javascript_typescript
[9]:https://github.com/Microsoft/BotBuilder-Samples/tree/master/samples/javascript_es6
[10]:https://github.com/Microsoft/BotBuilder-Samples/tree/master/samples/csharp_webapi
[111]:https://github.com/Microsoft/botbuilder-python/tree/master/samples

<a name="V4-whats-new"></a>
## <a name="bot-framework-sdk-new-in-preview"></a>Bot Framework SDK (新機能。 プレビュー段階)

- [アダプティブ ダイアログ][47] | [ドキュメント][48] | [C# サンプル][49]: アダプティブ ダイアログを使用すると、開発者が、会話の進行中に動的に変更できる会話を構築できます。  開発者はこれまで会話のフロー全体を事前に細かく計画していましたが、これでは会話の柔軟性が制限されます。  アダプティブ ダイアログにより柔軟性が高まり、会話の進行中に、コンテキストの変更や、会話への新しいステップやサブダイアログの挿入に対応できます。 

- [言語生成][43] | [ドキュメント][44] | [C# サンプル][45]: 言語生成により、開発者は、コードやリソース ファイルから埋め込み文字列を抽出し、言語生成のランタイムとファイル形式を管理できます。  また、お客様は、言語生成を使用して、語句のバリエーションを複数定義したり、コンテキストに基づいて単純な式を実行したり、会話のメモリを参照したりできます。Microsoft では今後、より自然な会話エクスペリエンスにつながる機能を追加していきます。

- [一般的な式言語][40] | [API][41]: 一般的な式言語は、ボットの会話を強化するためにアダプティブ ダイアログと言語生成の両方で使用されています。

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

## <a name="botkit"></a>Botkit
[Botkit][100] は、チャット ボット、アプリ、および主要なメッセージング プラットフォーム用カスタム統合を構築するための開発者ツールおよび SDK です。 Botkit ボットでは、`hear()` でトリガーが実行され、`ask()` で質問が行われ、`say()` で応答します。 開発者はこの構文を使用してダイアログを構築できます。現在これは最新バージョンのすべての Bot Framework SDK と互換性があります。 

さらに、Botkit には 6 つのプラットフォーム アダプターが用意されているため、Javascript ボット アプリケーションと次のメッセージング プラットフォームが直接通信できます: [Slack][102]、[Webex Teams][103]、[Google Hangouts][104]、[Facebook Messenger][105]、[Twilio][106]、および [Web チャット][107]。

Botkit は、Microsoft Bot Framework に含まれ、[MIT のオープン ソース ライセンス][101]でリリースされます

[100]:https://github.com/howdyai/botkit#readme
[101]:https://github.com/howdyai/botkit/blob/master/LICENSE.md
[102]:https://github.com/howdyai/botkit/tree/master/packages/botbuilder-adapter-slack#readme
[103]:https://github.com/howdyai/botkit/tree/master/packages/botbuilder-adapter-webex#readme
[104]:https://github.com/howdyai/botkit/tree/master/packages/botbuilder-adapter-hangouts#readme
[105]:https://github.com/howdyai/botkit/tree/master/packages/botbuilder-adapter-facebook#readme
[106]:https://github.com/howdyai/botkit/tree/master/packages/botbuilder-adapter-twilio-sms#readme
[107]:https://github.com/howdyai/botkit/tree/master/packages/botbuilder-adapter-web#readme

## <a name="bot-framework-solutions-new-in-preview"></a>Bot Framework ソリューション (新機能。 プレビュー段階)

[Bot Framework ソリューション リポジトリ](https://github.com/Microsoft/AI#readme)は、一連のテンプレート、ソリューション アクセラレータ、スキルのホームであり、高度なアシスタント的会話エクスペリエンスの構築に役立ちます。

| Name | 説明 |  
|:------------|:------------| 
|[**仮想アシスタント**](https://github.com/Microsoft/AI/tree/master/docs#virtual-assistant) | お客様の間で、自社のブランドに合わせてカスタマイズし、顧客に合わせてパーソナライズして、幅広いキャンバスやデバイスで使用できる会話アシスタントを提供したい、という声が高まっています。 <br/><br/> Enterprise Template により、基本的な会話の意図、ディスパッチ統合、QnA Maker、Application Insights、自動デプロイなど、新しいボット プロジェクトの作成が大幅に簡素化されます。|
|[**スキル**](https://github.com/Microsoft/AI/blob/master/docs/overview/skills.md)| 開発者が会話エクスペリエンスを作成するには、"スキル" と呼ばれる再利用可能な会話機能を結合します。 スキル自体がボットであり、リモートで呼び出されます。また、新しいスキルの作成を容易にするためのスキル開発者向けテンプレート (.NET、TS) もあります。 
|[**分析**](https://github.com/Microsoft/AI/blob/master/docs/readme.md#analytics)| 会話型 AI 分析ソリューションでは、ご自身のボットの正常性と動作に関する主な分析情報を取得します。 利用可能なテレメトリ、Application Insights のサンプル クエリ、および Power BI ダッシュボードを確認することで、お使いのボットとユーザーとの幅広い会話を理解します。 |

## <a name="azure-bot-service"></a>Azure Bot Service
Azure Bot Service を使用すると、データの完全な所有権と管理により、エンタープライズ レベルのインテリジェントなボットをホストできます。 開発者は自身のボットを登録して、Skype、Microsoft Teams、Cortana、Web チャットなどのユーザーに接続できます。 [Azure][27]  |  [ドキュメント][28] | [チャネル接続][29] 

* **Direct Line JS クライアント**: Azure Bot Service で Direct Line チャネルを使用する必要がある場合、Web チャット クライアントを使用していないときは、ご自身のカスタム アプリケーションで Direct Line JS クライアントを使用できます。 詳細については、[GitHub][30] にアクセスしてください。

<a name="ABS-whats-new"></a>

* **新機能。Direct Line Speech チャネル**: Bot Framework と Microsoft の Speech Services を組み合わせて、クライアントからボット アプリケーションに双方向にストリーミングされた音声とテキストを有効にするチャネルを提供しています。  詳細については、[音声チャネルをご自身のボットに](https://docs.microsoft.com/en-us/azure/bot-service/directline-speech-bot?view=azure-bot-service-4.0)追加する方法をご覧ください。

[27]:https://azure.microsoft.com/en-us/services/bot-service/
[28]:https://docs.microsoft.com/en-us/azure/bot-service/bot-service-overview-introduction?view=azure-bot-service-4.0
[29]:https://docs.microsoft.com/en-us/azure/bot-service/bot-service-manage-channels?view=azure-bot-service-4.0
[30]:https://github.com/Microsoft/BotFramework-DirectLineJS/blob/master/README.md


## <a name="bot-framework-emulator"></a>Bot Framework エミュレーター
[Bot Framework Emulator][60] はクロスプラットフォーム デスクトップ アプリケーションです。このアプリケーションを使用すると、ボット開発者が、Bot Framework SDK を使って構築されたボットをテストし、デバッグできます。 Bot Framework Emulator を使用して、ご自身のマシンでローカルで実行されているボットをテストしたり、リモートで実行されているボットに接続したりできます。

- [最新バージョンをダウンロード][61] | [ドキュメント][62]

<a name="Emulator-whats-new"></a>
### <a name="bot-inspector-new-in-preview"></a>Bot Inspector (新機能。 プレビュー段階)

Bot Framework Emulator の新しい Bot Inspector 機能のベータ版がリリースされました。 これにより、Microsoft Teams、Slack、Cortana、Facebook Messenger、Skype などのチャネルで、ご自身の Bot Framework SDK v4 ボットをデバッグし、テストできます。会話がある場合は、ボットが受信したメッセージ データを検査できる Bot Framework Emulator に、メッセージがミラー化されます。 さらに、チャネルとボットの間の特定のターンに対するボット状態のスナップショットも表示されます。 詳細については、[Bot Inspector](https://github.com/Microsoft/BotFramework-Emulator/blob/master/content/CHANNELS.md) に関するページをご覧ください

[60]:https://github.com/Microsoft/BotFramework-Emulator#readme
[61]:https://github.com/Microsoft/BotFramework-Emulator/releases/latest
[62]:https://docs.microsoft.com/en-us/azure/bot-service/bot-service-debug-emulator?view=azure-bot-service-4.0


## <a name="related-services"></a>関連サービス

### <a name="language-understanding"></a>Language Understanding 
自然言語エクスペリエンスを構築する機械学習ベースのサービス。 継続的に改良される、エンタープライズ対応のカスタム モデルをすばやく作成できます。 [Language Understanding Service (LUIS)][30] を使用すると、アプリケーションが人の発言の意図を認識できるようになります。

<a name="LUIS-whats-new"></a>

- **新機能。役割、外部エンティティ、および動的エンティティ**: 開発者がテキストからより詳細な情報を抽出できる機能がいくつか LUIS に追加され、ユーザーがより少ない労力で、よりインテリジェントなソリューションを構築できるようになりました。 また、ロールがすべてのエンティティ型に拡張されたため、同じエンティティを、コンテキストに基づいてさまざまなサブタイプで分類できます。 開発者は LUIS での操作をより細かく制御できるようになりました。たとえば、動的なリストや外部エンティティを使って、実行時にモデルを特定および更新できます。 予測時、リスト エンティティへの追加が動的リストを使って行われるため、ユーザー固有の情報を正確に一致させることができます。 外部エンティティでは補助的なエンティティ抽出が個別に実行され、その情報は、他のモデルに対する強力なシグナルとして LUIS に追加できます。

- **新機能。Analytics ダッシュボード**: LUIS の視覚的に豊かで包括的かつ詳細なダッシュボードがリリースされます。 そのわかりやすいデザインには、アプリケーションの設計時にユーザーのほとんどが直面する一般的な問題が強調表示され、その問題の解決方法が簡単に説明されています。これは、ユーザーが、モデルの品質、潜在的なデータの問題、およびベスト プラクティスを採用するためのガイダンスに関する、さらに詳しい分析情報を得るうえで役立ちます。

[ドキュメント][31] | [ボットへの自然言語の理解の追加][32] 

[18]:https://github.com/Microsoft/botbuilder-tools/tree/master/packages/LUIS#readme
[19]:https://github.com/Microsoft/botbuilder-tools/tree/master/packages/QnAMaker#readme
[30]:https://www.luis.ai
[31]:https://docs.microsoft.com/en-us/azure/cognitive-services/LUIS/Home
[32]:https://docs.microsoft.com/en-us/azure/bot-service/bot-builder-howto-v4-luis?view=azure-bot-service-4.0&branch=pr-en-us-1325&tabs=csharp

### <a name="qna-maker"></a>QnA Maker
[QnA Maker][33] は、データに基づく対話型の Q&A レイヤーを作成できるクラウドベースの API サービスです。 QnA Maker を使用すると、FAQ URL、構造化されたドキュメント、製品マニュアル、または本文に基づいて、単純な質疑応答ボットを数分でビルド、トレーニング、および公開できます。

<a name="QnA-whats-new"></a>

- **新機能。抽出パイプライン**: URL、ファイル、および SharePoint から階層情報を抽出できるようになりました
- **新機能。インテリジェンス**: コンテキストに応じたランキング モデル、アクティブ ラーニング提案
- **新機能。会話**: QnA Maker のマルチターンの会話。

[ドキュメント][34]  | [ボットへの QnA Maker の追加][35] 

[33]:https://www.qnamaker.ai/
[34]:https://aka.ms/qnamaker-docs-home
[35]:https://docs.microsoft.com/en-us/azure/bot-service/bot-builder-howto-qna?view=azure-bot-service-4.0&branch=pr-en-us-1325&tabs=cs

### <a name="speech-services"></a>Speech Services
[Speech Services](https://docs.microsoft.com/en-us/azure/cognitive-services/speech-service/) で音声がテキストに変換され、統合音声サービスにより音声翻訳とテキスト読み上げが実行されます。 音声サービスを使用すると、音声をご自身のボットに統合し、カスタム ウェイク ワードを作成して、複数言語での作成が可能です。

### <a name="adaptive-cards"></a>Adaptive Cards (アダプティブ カード)
[アダプティブ カード](https://adaptivecards.io)は、一般的かつ一貫性のある方法でカード コンテンツを変換するための開発者向けオープン標準です。Bot Framework の開発者は、これを使用して、優れたクロスチャネル会話エクスペリエンスを作成できます。

## <a name="additional-information"></a>追加情報
- [詳細](https://github.com/Microsoft/botframework/blob/master/whats-new.md#whats-new)については、GitHub ページをご覧ください。
