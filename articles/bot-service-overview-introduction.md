---
title: Bot Service について | Microsoft Docs
description: Bot Service について説明します。これは、ボットを作成、接続、テスト、配置、監視、管理するためのサービスです。
keywords: 概要, 紹介, SDK, 概略
author: Kaiqb
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 05/03/2018
ms.openlocfilehash: 142d4cbe0c252e88bab800bb3823b70434a65bd6
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/18/2018
ms.locfileid: "39304105"
---
::: moniker range="azure-bot-service-3.0"

# <a name="azure-bot-service"></a>Azure Bot Service

Azure Bot Service には、インテリジェントなボットの作成、テスト、配置、管理をすべて一元的に行うツールが備わっています。 SDK に備わっているモジュラー式の拡張可能なフレームワークにより、開発者はテンプレートを利用して、音声、言語理解、質疑応答などを行うボットを作成できます。  

## <a name="what-is-a-bot"></a>ボットとは何ですか?
ボットとは、テキスト、グラフィックス (カード)、音声を使用してユーザーが会話形式で対話するアプリです。 単純な質疑応答の対話の場合もあれば、インテリジェントな方法で人がサービスと対話できる高度なボットもあり、その場合は、パターン マッチング、状態の追跡、人工知能の技法が使用されて、既存のビジネス サービスと十分に統合されます。 ボットの[導入事例](https://azure.microsoft.com/services/bot-service/)をご確認ください。  

## <a name="building-a-bot"></a>ボットの作成 
C# または Node.js でボットを作成するには、好みの開発環境やコマンド ライン ツールを使用できます。 ボット開発の各ステージのためにツールが用意されており、ボットを作成して作業を始めるためにご使用いただけます。    

![ボットの概要](media/bot-service-overview.png) 

## <a name="plan"></a>プラン 
コードを書く前に、ボットの[デザイン ガイドライン](bot-service-design-principles.md) でベスト プラクティスを確認し、作成するボットに必要な事柄を見極めてください。 シンプルなボットを作成することもできますし、音声、言語理解、QnA (各種ソースからのナレッジを取り出してインテリジェントな回答を提供する機能) などの高度な機能を含めることもできます。  

> [!TIP] 
>
> 必要なテンプレートをインストールします。
>  - Bot Builder .NET SDK v3 [VSIX テンプレート](https://marketplace.visualstudio.com/items?itemName=BotBuilder.BotBuilderV3) 
>  - Bot Builder Node.js SDK v3 [Yeoman テンプレート](https://www.npmjs.com/package/generator-botbuilder) 
>
> ツールをインストールします。
> - ボット資産を作成および管理するための [CLI ツール](https://github.com/Microsoft/botbuilder-tools)をダウンロードします。 これらのツールを使用すると、ボット構成ファイル、LUIS アプリ、QnA ナレッジ ベースなどをコマンド ラインから管理できます。 詳細については、[README](https://github.com/Microsoft/botbuilder-tools/blob/master/README.md) を参照してください。
> - ボットをテストするための[エミュレーター](https://github.com/Microsoft/BotFramework-Emulator/releases)
>
> 必要な場合には、次のようなボット コンポーネントを使用します。  
> - [LUIS](https://www.luis.ai/)。ボットに言語理解機能を追加します
> - [QnA Maker](https://qnamaker.ai/)。より自然な会話方法でユーザーの質問に応答します
> - [Speech](https://azure.microsoft.com/services/cognitive-services/speech/)。音声をテキストに変換し、意図を理解し、そのテキストを音声に変換し直します  
> - [スペルチェック機能](https://azure.microsoft.com/services/cognitive-services/spell-check/)。スペル エラーを修正し、氏名の違い、ブランド名、スラングを認識します 
> - [Cognitive Services](bot-service-concept-intelligence.md)。さまざまな他のインテリジェントなコンポーネント用 


## <a name="build-your-bot"></a>ボットの作成 
作成するボットは会話型インターフェイスを実装した Web サービスであり、Bot Service とやり取りします。 このソリューションを多数の環境と言語で作成できます。また、Visual Studio や Yeoman で、あるいは Azure portal 内で直接、簡単に作業を開始するためのツールが提供されています。 使用できるツールとサービスのいくつかを以下に示します。

> [!TIP]
>
> [SDK](~/dotnet/bot-builder-dotnet-quickstart.md)、[Azure portal](bot-service-quickstart.md)、[CLI ツール](~/bot-builder-create-templates.md)を使用してボットを作成します。
>
> コンポーネントを追加します。 
> - 言語理解モデル [LUIS](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/home) を追加します。 
> - [QnA Maker](https://docs.microsoft.com/en-us/azure/cognitive-services/qnamaker/home) ナレッジ ベースを追加して、ユーザーの尋ねる質問に回答します。  
> - カード、音声、翻訳でユーザー エクスペリエンスを強化します。 
> - Bot Builder SDK を使用してボットにロジックを追加します。   

## <a name="test-your-bot"></a>ボットのテスト 
ボットは様々なパーツが連携する複雑なアプリです。 そのため、他の複雑なアプリの場合と同様、興味深いバグが発生したり、期待どおりにボットが動作しなかったりすることがあります。 発行する前にボットをテストしてください。

> [!TIP] 
>
> - [エミュレーターでボットをテストする](bot-service-debug-emulator.md)
> - [Web Chat でボットをテストする](bot-service-manage-test-webchat.md)

## <a name="publish"></a>[発行] 
準備が整ったら、ボットを Azure に発行するか、独自の Web サービスまたはデータ センターに発行します。 継続的配置を設定するとローカルでボットを開発でき、ボットを GitHub や Visual Studio Team Services などのソース管理にチェックインする場合に便利です。 変更をソース リポジトリにチェックインし直すと、その変更が Azure に自動的に配置されます。

> [!Tip]
>
> - [Azure へのデプロイ](bot-service-build-continuous-deployment.md)

## <a name="connect"></a>接続          
ボットを、Facebook、Messenger、Kik、Skype、Slack、Microsoft Teams、Telegram、SMS 送信、Twilio、Cortana、Skype などのチャネルに接続し、対話を増やし、より多くのユーザーに接することができます。  
  
> [!TIP]
>
> - [追加するチャネルを選択する](bot-service-manage-channels.md)


## <a name="evaluate"></a>Evaluate 
Azure portal で収集されたデータを使用し、ボットの機能とパフォーマンスを強化する機会を特定します。 トラフィック、待ち時間、統合などのサービス レベルのデータやインストルメンテーション データを取得できます。 Analytics では、ユーザー、メッセージ、チャネル データに関する会話レベルのレポートも提供されます。

> [!Tip]
>
> - [分析情報を収集する](bot-service-manage-analytics.md) 


::: moniker-end

::: moniker range="azure-bot-service-4.0"

# <a name="azure-bot-service"></a>Azure Bot Service

[!INCLUDE [pre-release-label](includes/pre-release-label.md)]

Azure Bot Service には、インテリジェントなボットの作成、テスト、配置、管理をすべて一元的に行うツールが備わっています。 SDK に備わっているモジュラー式の拡張可能なフレームワークにより、開発者はテンプレートを利用して、音声、言語理解、質疑応答などを行うボットを作成できます。  

## <a name="what-is-a-bot"></a>ボットとは何ですか?
ボットとは、テキスト、グラフィックス (カード)、音声を使用してユーザーが会話形式で対話するアプリです。 単純な質疑応答の対話の場合もあれば、インテリジェントな方法で人がサービスと対話できる高度なボットもあり、その場合は、パターン マッチング、状態の追跡、人工知能の技法が使用されて、既存のビジネス サービスと十分に統合されます。 ボットの[導入事例](https://azure.microsoft.com/services/bot-service/)をご確認ください。  

## <a name="building-a-bot"></a>ボットの作成 
C#、JavaScript、Java、Python でボットを作成するには、好みの開発環境やコマンド ライン ツールを使用できます。 ボット開発の各ステージのためにツールが用意されており、ボットを作成して作業を始めるためにご使用いただけます。    

![ボットの概要](media/bot-service-overview.png) 

## <a name="plan"></a>プラン 
コードを書く前に、ボットの[デザイン ガイドライン](bot-service-design-principles.md) でベスト プラクティスを確認し、作成するボットに必要な事柄を見極めてください。 シンプルなボットを作成することもできますし、音声、言語理解、QnA (各種ソースからのナレッジを取り出してインテリジェントな回答を提供する機能) などの高度な機能を含めることもできます。 [CLI ツール](~/bot-builder-create-templates.md)、[Azure portal](bot-service-quickstart.md)、以下のテンプレートのいずれかを使用して作業を開始できます。

**必要なテンプレートを用意する**

| .NET | JavaScript | 
| --- | --- | 
| [VSIX テンプレート](https://marketplace.visualstudio.com/items?itemName=BotBuilder.botbuilderv4) | [Yeoman テンプレート](https://www.npmjs.com/package/generator-botbuilder)。 @preview を使用して v4 テンプレートを入手します。 |

**必要な場合には、ツールを調べてインストールします**

- ボット資産を作成および管理するための [CLI ツール](https://github.com/Microsoft/botbuilder-tools)をダウンロードします。 これらのツールを使用すると、ボット構成ファイル、LUIS アプリ、QnA ナレッジ ベースなどをコマンド ラインから管理できます。 詳細については、[README](https://github.com/Microsoft/botbuilder-tools/blob/master/README.md) を参照してください。
- ボットをテストするための[エミュレーター](https://github.com/Microsoft/BotFramework-Emulator/releases)
- ボットに自然言語を追加するための [LUIS](https://www.luis.ai/)
- [QnA Maker](https://qnamaker.ai/)。より自然な会話方法でユーザーの質問に応答します


## <a name="build-your-bot"></a>ボットの作成 
作成するボットは会話型インターフェイスを実装した Web サービスであり、Bot Service とやり取りします。 このソリューションを多数の環境と言語で作成できます。また、Visual Studio や Yeoman で、あるいは Azure portal 内で直接、簡単に作業を開始するためのツールが提供されています。 使用できるツールとサービスのいくつかを以下に示します。

使用できる多数のコンポーネントの一部を以下に記します。

| コンポーネント | 説明 |
| --- | --- |
| [LUIS](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/home) | 言語理解を追加します |
| [QnA Maker](https://docs.microsoft.com/en-us/azure/cognitive-services/qnamaker/home)  | ナレッジ ベースを追加して、ユーザーが尋ねる質問に回答します |
| [ディスパッチ ツール](~/v4sdk/bot-builder-tutorial-dispatch.md) | 複数のモデルを使用する場合、どれをどのタイミングで使用するかをインテリジェントに判別します |
| [リッチ メディア](v4sdk/bot-builder-howto-add-media-attachments.md) | メディア カードや音声を使用してユーザー エクスペリエンスを強化します |
| [Speech](https://azure.microsoft.com/services/cognitive-services/speech/) | 音声をテキストに変換し、意図を理解し、そのテキストを音声に変換し直します |
| [スペル チェック](https://azure.microsoft.com/services/cognitive-services/spell-check/) | スペル エラーを修正し、氏名の違い、ブランド名、スラングを認識します |

> [!NOTE]
> 上の表にすべてがリストされているわけではありません。 ボット機能についてさらに調べるには、[メッセージの送信](v4sdk/bot-builder-howto-send-messages.md)以下の左側の記事を参照してください。

## <a name="test-your-bot"></a>ボットのテスト 
ボットは様々なパーツが連携する複雑なアプリです。 そのため、他の複雑なアプリの場合と同様、興味深いバグが発生したり、期待どおりにボットが動作しなかったりすることがあります。 発行する前にボットをテストしてください。 

[エミュレーターを使用してボットをテストする](bot-service-debug-emulator.md)か、[Web Chat を使用してボットをテスト](bot-service-manage-test-webchat.md)します。

## <a name="publish"></a>[発行] 
準備が整ったら、ボットを Azure に発行するか、独自の Web サービスまたはデータ センターに発行します。 継続的配置を設定するとローカルでボットを開発でき、ボットを GitHub や Visual Studio Team Services などのソース管理にチェックインする場合に便利です。 変更をソース リポジトリにチェックインし直すと、その変更が Azure に自動的に配置されます。

継続的配置を使用して [Azure に配置](bot-service-build-continuous-deployment.md)します。

## <a name="connect"></a>接続          
ボットを、Facebook、Messenger、Kik、Skype、Slack、Microsoft Teams、Telegram、SMS 送信、Twilio、Cortana、Skype などのチャネルに接続し、対話を増やし、より多くのユーザーに接することができます。

[追加するチャネルを選択](bot-service-manage-channels.md)します。


## <a name="evaluate"></a>Evaluate 
Azure portal で収集されたデータを使用し、ボットの機能とパフォーマンスを強化する機会を特定します。 トラフィック、待ち時間、統合などのサービス レベルのデータやインストルメンテーション データを取得できます。 Analytics では、ユーザー、メッセージ、チャネル データに関する会話レベルのレポートも提供されます。 

[分析情報の収集](bot-service-manage-analytics.md)方法を確認します。

::: moniker-end
