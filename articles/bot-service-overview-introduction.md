---
title: Azure Bot Service の概要 | Microsoft Docs
description: Bot Service について説明します。これは、ボットを作成、接続、テスト、配置、監視、管理するためのサービスです。
keywords: 概要, 紹介, SDK, 概略
author: Kaiqb
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 08/27/2018
ms.openlocfilehash: 47dadde9a294855e3c39c1cbd17635b2839b856d
ms.sourcegitcommit: d2e0a1c7da19afc1254bc689bb345dc1804484e2
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 08/28/2018
ms.locfileid: "43117043"
---
::: moniker range="azure-bot-service-3.0"

# <a name="about-azure-bot-service"></a>Azure Bot Service について

[!INCLUDE [pre-release-label](includes/pre-release-label-v3.md)]

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
準備が整ったら、ボットを Azure に発行するか、独自の Web サービスまたはデータ センターに発行します。 継続的配置を設定するとローカルでボットを開発でき、ボットを GitHub や Visual Studio Team Services などのソース管理にチェックインする場合に便利です。 変更をソース リポジトリにチェックインし直すと、その変更が Azure に自動的にデプロイされます。

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

# <a name="about-azure-bot-service"></a>Azure Bot Service について

[!INCLUDE [pre-release-label](includes/pre-release-label.md)]

Azure Bot Service には、インテリジェントなボットの作成、テスト、配置、管理をすべて一元的に行うツールが備わっています。 SDK で提供されるモジュール式の拡張可能なフレームワークにより、開発者はテンプレートを利用して、音声の提供、自然言語の解釈、質問と応答の処理などを行うボットを作成できます。  

## <a name="what-is-a-bot"></a>ボットとは何ですか?

ボットとは、人間と会話でコミュニケーションできるアプリです。 ソフトウェアは、テキスト、音声、グラフィックス、またはメニューを介して対話し、会話に関連するタスクを実行できます。 質問と応答の単純なやり取りの場合もあれば、既存のサービスと十分に統合されたインテリジェントな手法を利用して、ユーザーが自然な方法でサービスと対話することを可能にする高度なボットもあります。

ボットにより、システムは情報の収集が可能になり、コンピューターという感覚が薄れて会話のように感じるエクスペリエンスをユーザーに提供できます。 また、人と直接対話する必要がない場合に、ディナーの予約やユーザーからのプロファイル情報の収集などの単純なタスクをシステムに移動 (または他のシステムと統合) できます。 

## <a name="building-a-bot"></a>ボットの作成 

C#、JavaScript、Java、Python でボットを作成するには、好みの開発環境やコマンド ライン ツールを使用できます。 ボット開発の各ステージのためにツールが用意されており、ボットを作成して作業を始めるためにご使用いただけます。    

### <a name="design"></a>設計

コードを書く前に、ボットの[デザイン ガイドライン](bot-service-design-principles.md) でベスト プラクティスを確認し、作成するボットに必要な事柄を見極めてください。 単純なボットを作成することも、音声、自然言語の解釈、ユーザーの質問への応答などの高度な機能を含めることもできます。 

### <a name="build"></a>構築

作成するボットは会話型インターフェイスを実装した Web サービスであり、Bot Service とやり取りします。 このソリューションは、さまざまな環境や言語で作成できます。また、前述のように、簡単に作業を開始するためのテンプレートが用意されています。 [Azure portal](bot-service-quickstart.md) でボット開発を開始することも、下記のテンプレートを使用して、任意の言語でローカルで開発することもできます。

| .NET テンプレート | JavaScript テンプレート | 
| --- | --- | 
| [VSIX テンプレート](https://marketplace.visualstudio.com/items?itemName=BotBuilder.botbuilderv4) | [Yeoman テンプレート](https://www.npmjs.com/package/generator-botbuilder)。 @preview を使用して v4 テンプレートを入手します。 |


ボットの機能を強化するために使用できる追加コンポーネントを次に示します。 

| Feature | 説明 | Link |
| --- | --- | --- |
| 自然言語処理を追加する | ボットによる自然言語の解釈、スペル ミスの理解、音声の使用、ユーザーの意図の認識を可能にします。 | [LUIS の使用方法](~/v4sdk/bot-builder-howto-v4-luis.md) 
| 質問に答える | ナレッジ ベースを追加して、より自然な会話方法でユーザーの質問に答えます。 | [QnA Maker の使用方法](~/v4sdk/bot-builder-howto-qna.md) 
| 複数のモデルを管理する | LUIS や QnA Maker などの複数のモデルを使用している場合、ボットの会話中に、いつ、どのモデルを使用するかをインテリジェントに判断します。 | [ディスパッチ ツール](~/v4sdk/bot-builder-tutorial-dispatch.md) |
| カードとボタンを追加する | グラフィックス、メニュー、カードなど、テキスト以外のメディアでユーザー エクスペリエンスを強化します。 | [カードを追加する方法](v4sdk/bot-builder-howto-add-media-attachments.md) |

> [!NOTE]
> 上の表にすべてがリストされているわけではありません。 ボット機能についてさらに調べるには、[メッセージの送信](~/v4sdk/bot-builder-howto-send-messages.md)以下の左側の記事を参照してください。

さらに、ボット資産の作成、管理、テストに役立つ CLI ツールも用意されています。 これらのツールにより、ボット構成ファイル、LUIS アプリ、QnA ナレッジ ベースの管理や、会話のモックなどをコマンド ラインから実行できます。 詳細については、[README](https://github.com/Microsoft/botbuilder-tools/blob/master/README.md) を参照してください。

### <a name="test"></a>テスト 
ボットは様々なパーツが連携する複雑なアプリです。 そのため、他の複雑なアプリの場合と同様、興味深いバグが発生したり、期待どおりにボットが動作しなかったりすることがあります。 発行する前にボットをテストしてください。 

[エミュレーターを使用してボットをテストする](bot-service-debug-emulator.md)か、[Web Chat を使用してボットをテスト](bot-service-manage-test-webchat.md)します。

### <a name="publish"></a>[発行] 
準備が整ったら、[ボットを Azure に発行](bot-builder-howto-deploy-azure.md)するか、独自の Web サービスまたはデータ センターに発行します。

### <a name="connect"></a>接続          
ボットを、Facebook、Messenger、Kik、Skype、Slack、Microsoft Teams、Telegram、SMS 送信、Twilio、Cortana、Skype などのチャネルに接続し、対話を増やし、より多くのユーザーに接することができます。

[追加するチャネルを選択](bot-service-manage-channels.md)します。


### <a name="evaluate"></a>Evaluate 
Azure portal で収集されたデータを使用し、ボットの機能とパフォーマンスを強化する機会を特定します。 トラフィック、待ち時間、統合などのサービス レベルのデータやインストルメンテーション データを取得できます。 Analytics では、ユーザー、メッセージ、チャネル データに関する会話レベルのレポートも提供されます。 

[分析情報の収集](bot-service-manage-analytics.md)方法を確認します。


## <a name="next-steps"></a>次の手順

> [!div class="nextstepaction"]
> [ボットを作成する](bot-service-quickstart.md)

::: moniker-end
