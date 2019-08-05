---
title: 新機能 | Microsoft Docs
description: Bot Framework の新機能について説明します。
keywords: Bot Framework, Azure Bot Service
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: conceptual
ms.service: bot-service
ms.subservice: abs
ms.date: 07/17/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: f7083c45e67d8731e25e14577f6b061732ffefd5
ms.sourcegitcommit: f3fda6791f48ab178721b72d4f4a77c373573e38
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/31/2019
ms.locfileid: "68671395"
---
# <a name="whats-new-in-bot-framework-july-2019"></a>Bot Framework の新機能 (2019 年 7 月)

[!INCLUDE[applies-to](includes/applies-to.md)]

Bot Framework SDK v4 は [オープン ソース SDK][1a] です。この SDK を使用すると、開発者が好きなプログラミング言語を使用して、高度な会話をモデル化して構築できます。

この記事では、Bot Framework と Azure Bot Service の主な新機能と機能強化をまとめます。

|   | C#  | JS  | Python |   
|---|:---:|:---:|:------:|
|SDK |[4.5][1] | [4.5][2] | [4.4.0b2 (プレビュー)][3] | 
|Docs | [docs][5] |[docs][5] |  | |
|サンプル |[.NET Core][6]、[WebAPI][10] |[Node.js][7]、[TypeScript][8]、[es6][9]  | [Python][111] | | 

[1a]:https://github.com/microsoft/botframework-sdk/#readme
[1]:https://github.com/Microsoft/botbuilder-dotnet/#packages
[2]:https://github.com/Microsoft/botbuilder-js#packages
[3]:https://github.com/Microsoft/botbuilder-python#packages
[5]:https://docs.microsoft.com/azure/bot-service/?view=azure-bot-service-4.0
[6]:https://github.com/Microsoft/BotBuilder-Samples/tree/master/samples/csharp_dotnetcore
[7]:https://github.com/Microsoft/BotBuilder-Samples/tree/master/samples/javascript_nodejs
[8]:https://github.com/Microsoft/BotBuilder-Samples/tree/master/samples/javascript_typescript
[9]:https://github.com/Microsoft/BotBuilder-Samples/tree/master/samples/javascript_es6
[10]:https://github.com/Microsoft/BotBuilder-Samples/tree/master/samples/csharp_webapi
[111]:https://github.com/Microsoft/botbuilder-python/tree/master/samples


## <a name="bot-framework-channels"></a>Bot Framework チャネル
- [Direct Line Speech (パブリック プレビュー)](https://aka.ms/streaming-extensions) | [ドキュメント](https://docs.microsoft.com/azure/bot-service/directline-speech-bot?view=azure-bot-service-4.0):Bot Framework と Microsoft の Speech Services は、WebSocket を使用して、クライアントからボット アプリケーションに双方向にストリーミングされた音声とテキストを有効にするチャネルを提供します。  

- [Direct Line App Service 拡張機能 (パブリック プレビュー)](https://portal.azure.com) | [ドキュメント](https://aka.ms/directline-ase):Direct Line API を使用してクライアントがボットに直接接続できるようにする Direct Line のバージョン。 これには、パフォーマンスの向上や分離の強化など、多くの利点があります。 Direct Line App Service 拡張機能は、Azure App Service Environment 内でホストされるものも含め、すべての Azure App Service で利用できます。 Azure App Service Environment は分離を提供し、VNet 内での作業に最適です。 VNet を使用すると、Azure に独自のプライベート空間を作成できます。VNet は分離やセグメント化などの重要な利点を提供するため、クラウド ネットワークにとって重要です。 

## <a name="bot-framework-sdk"></a>Bot Framework SDK
- [アダプティブ ダイアログ (SDK v4.6 プレビュー)](https://github.com/Microsoft/BotBuilder-Samples/tree/master/experimental/adaptive-dialog#readme) | [ドキュメント](https://github.com/Microsoft/BotBuilder-Samples/tree/master/experimental/adaptive-dialog/docs) | [C# サンプル](https://github.com/Microsoft/BotBuilder-Samples/tree/master/experimental/adaptive-dialog/csharp_dotnetcore):アダプティブ ダイアログにより、開発者はコンテキストとイベントに基づいて会話フローを動的に更新できるようになりました。 これは、会話の途中で会話コンテキストの切り替えや中断を処理する場合に特に便利です。 
  
- [Bot Framework Python SDK (プレビュー 2)](https://github.com/microsoft/botbuilder-python) | [サンプル](https://github.com/Microsoft/botbuilder-python/tree/master/samples):Python SDK で OAuth、プロンプト、CosmosDB がサポートされるようになり、SDK 4.5 のすべての主要機能が含まれるようになりました。 また、SDK の新機能について学習するのに役立つサンプルも用意されています。

## <a name="bot-framework-testing"></a>Bot Framework のテスト
- [単体テスト](http://aka.ms/bot-test-package) | [ドキュメント](https://aka.ms/testing-framework) | [C# サンプル](https://aka.ms/corebot-test) | [JS サンプル](https://aka.ms/js-core-test-sample):より優れたテスト ツールを求めるお客様や開発者のご要望にお応えするため、7 月版の SDK では、新しい単体テスト機能が導入されています。 Microsoft.Bot.Builder.testing パッケージを使用すると、ボットの単体テスト ダイアログのプロセスが簡略化されます。 

- [チャネルのテスト](https://github.com/Microsoft/BotFramework-Emulator/releases) | [ドキュメント](https://aka.ms/channel-testing): 

Bot Inspector は、Microsoft ビルド 2019 で導入された Bot Framework Emulator の新機能であり、Microsoft Teams、Slack、Cortana などのチャネルでボットをデバッグおよびテストできます。 特定のチャネルでボットを使用すると、ボットが受信したメッセージ データを検査できる Bot Framework Emulator に、メッセージがミラー化されます。 さらに、チャネルとボットの間の特定のターンに対するボットのメモリ状態のスナップショットも表示されます。

## <a name="web-chat"></a>Web チャット
- 企業のお客様からのご要望に基づき、ボットを使用してエンタープライズ アプリ上のリソースへのアクセスをユーザーに承認する方法を示す [Web チャット サンプル](https://github.com/microsoft/BotFramework-WebChat/tree/master/samples/19.a.single-sign-on-for-enterprise-apps#single-sign-on-demo-for-enterprise-apps-using-oauth)を追加しました。 OAuth と Microsoft Graph および GitHub API の相互運用性を示すために、2 種類のリソースが使用されています。

## <a name="solutions"></a>解決方法
- [仮想アシスタント ソリューション アクセラレータ](https://github.com/Microsoft/botframework-solutions#readme):高度な会話エクスペリエンスの構築に役立つ、一連のテンプレート、ソリューション アクセラレータ、スキルを提供します。 Direct Line Speech および仮想アシスタントと統合する仮想アシスタント用の新しい Android アプリ クライアント。デバイス クライアントがどのように仮想アシスタントと対話し、アダプティブ カードをレンダリングするかを示します。 更新プログラムには、Direct Line Speech と Microsoft Teams のサポートも含まれています。
  
- [Dynamics 365 Virtual Agent for Customer Service (パブリック プレビュー)](https://dynamics.microsoft.com/en-us/ai/virtual-agent-for-customer-service/):このパブリック プレビューでは、インテリジェントで高度な適応性を備えた仮想エージェントを使用して、優れたカスタマー サービスを提供できます。 カスタマー サービスのエキスパートは、AI による分析情報を使用してボットを簡単に作成し、強化することができます。
  
- [Chat for Dynamics 365](https://www.powerobjects.com/powerpacks/powerchat/):Chat for Dynamics 365 には、サポート エージェントとエンド ユーザーが効率的に対話し、高い生産性を維持できるようにするための機能がいくつか用意されています。 Microsoft Dynamics 365 内で Web サイトの訪問者とのライブ チャットおよび会話の追跡を行います。

## <a name="additional-information"></a>追加情報
- 以前のお知らせは[こちら](what-is-new-archive.md)から参照できます。
