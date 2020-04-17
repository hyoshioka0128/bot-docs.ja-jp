---
title: 新機能 - Bot Service
description: Bot Framework の新機能について説明します。
keywords: Bot Framework, Azure Bot Service
author: kamrani
ms.author: kamrani
manager: kamrani
ms.topic: conceptual
ms.service: bot-service
ms.date: 12/10/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 5b8b9e23168bdf45180c354edcfba99801d653e3
ms.sourcegitcommit: 9d77f3aff9521d819e88efd0fbd19d469b9919e7
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/16/2020
ms.locfileid: "77441537"
---
# <a name="whats-new-december-2019"></a>2019 年 12 月の新機能

[!INCLUDE[applies-to](includes/applies-to.md)]

Bot Framework SDK v4 は [オープン ソース SDK](https://github.com/microsoft/botframework-sdk/#readme) です。この SDK を使用すると、開発者が好きなプログラミング言語を使用して、高度な会話をモデル化して構築できます。

この記事では、Bot Framework と Azure Bot Service の主な新機能と機能強化をまとめます。

|   | C#  | JS  | Python |  Java | 
|---|:---:|:---:|:------:|:-----:|
|Release |[GA][1] | [GA][2] | [GA][3] | [プレビュー 3][3a]|
|Docs | [docs][5] |[docs][5] |[docs][5]  | |
|サンプル |[.NET Core][6]、[WebAPI][10] |[Node.js][7]、[TypeScript][8]、[es6][9]  | [Python][11a] | | 

#### <a name="python-sdk-ga"></a>Python SDK (GA)
Python SDK が一般提供されたことをお知らせします。 この SDK では、コア ボット ランタイム、コネクタ、ミドルウェア、ダイアログ、プロンプト、Language Understanding (LUIS)、QnA Maker がサポートされます。 

#### <a name="bot-framework-sdk-for-skills-ga"></a>Bot Framework SDK for Skills (GA)

- **ボットのスキル**:ボットに機能を追加するために、再利用可能な会話スキルを作成します。 カレンダー、電子メール、仕事、目的地、自動車、天気、ニュースのスキルなど、事前に構築されたスキルを活用できます。 スキルには、必要に応じてカスタマイズおよび拡張するために提供される言語モデル、ダイアログ、QnA、統合コードが含まれます。 [[ドキュメント](https://aka.ms/skills-docs)]

- **Power Virtual Agent のスキル - 近日公開**:Power Virtual Agents でビルドされたボットの場合、Bot Framework と Azure Cognitive Services を使用してこれらのボットの新しいスキルを構築できます。新しいボットをゼロから作成する必要はありません。 

#### <a name="bot-framework-for-power-virtual-agent-ga"></a>Bot Framework for Power Virtual Agent (GA)

Power Virtual Agent は、ビジネスユーザーが、特定の AI サービスをコーディングしたり管理したりする必要なく、SaaS エクスペリエンスを構築する UI ベースのボット内にボットを作成できるように設計されています。 Power Virtual Agents は、Microsoft Bot Framework を使用して拡張できるため、開発者やビジネス ユーザーは組織のボットを協力して構築することができます。 [[ドキュメント](https://docs.microsoft.com/dynamics365/ai/customer-service-virtual-agent/overview)]

#### <a name="adaptive-dialogs-preview"></a>アダプティブ ダイアログ (プレビュー)
アダプティブ ダイアログにより、開発者はコンテキストとイベントに基づいて会話フローを動的に更新できます。 これは、会話の途中で会話コンテキストの切り替えや中断を処理する場合に特に便利です。 [[ドキュメント][48] | [C# サンプル][49]] 

#### <a name="language-generation-preview"></a>言語生成 (プレビュー)
言語生成により、開発者は、1 つのフレーズに対して複数のバリエーションを定義したり、コンテキストに基づいて単純な式を実行したり、会話メモリを参照したりといった、ボットの応答を生成するためのロジックを分離することができます。 [[ドキュメント][44] | [C# サンプル][45]]

#### <a name="common-expression-language-preview"></a>一般的な式言語 (プレビュー)
一般的な式言語により、実行時に条件ベースのロジックの結果を評価できます。 一般的な言語は、Bot Framework SDK と、アダプティブ ダイアログや言語生成などの会話 AI コンポーネントで使用できます。 [[ドキュメント][40] | [API][41]]

[1]:https://github.com/Microsoft/botbuilder-dotnet/#packages
[2]:https://github.com/Microsoft/botbuilder-js#packages
[3]:https://github.com/Microsoft/botbuilder-python#packages
[3a]:https://github.com/Microsoft/botbuilder-java#packages
[5]:https://docs.microsoft.com/azure/bot-service/?view=azure-bot-service-4.0
[6]:https://github.com/Microsoft/BotBuilder-Samples/tree/master/samples/csharp_dotnetcore
[7]:https://github.com/Microsoft/BotBuilder-Samples/tree/master/samples/javascript_nodejs
[8]:https://github.com/Microsoft/BotBuilder-Samples/tree/master/samples/typescript_nodejs
[9]:https://github.com/Microsoft/BotBuilder-Samples/tree/master/samples/javascript_es6
[10]:https://github.com/Microsoft/BotBuilder-Samples/tree/master/samples/csharp_webapi
[11a]:https://aka.ms/python-sample-repo


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

## <a name="additional-information"></a>関連情報
- 以前のお知らせは[こちら](what-is-new-archive.md)から参照できます。
