---
title: Azure Bot Service の概要 | Microsoft Docs
description: Bot Service について説明します。これは、ボットを作成、接続、テスト、配置、監視、管理するためのサービスです。
keywords: 概要, 紹介, SDK, 概略
author: Kaiqb
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 10/31/2018
ms.openlocfilehash: 616c3bfd5fcb36c06f4e2acf032ba3cf5fc125d3
ms.sourcegitcommit: a496714fb72550a743d738702f4f79e254c69d06
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/01/2018
ms.locfileid: "50736700"
---
::: moniker range="azure-bot-service-3.0"

# <a name="about-azure-bot-service"></a>Azure Bot Service について

[!INCLUDE [pre-release-label](includes/pre-release-label-v3.md)]

Azure Bot Service には、インテリジェントなボットの作成、テスト、配置、管理をすべて一元的に行うツールが備わっています。 SDK に備わっているモジュラー式の拡張可能なフレームワークにより、開発者はテンプレートを利用して、音声、言語理解、質疑応答などを行うボットを作成できます。  

## <a name="what-is-a-bot"></a>ボットとは何ですか?
ボットとは、テキスト、グラフィックス (カード)、音声を使用してユーザーが会話形式で対話するアプリです。 単純な質疑応答の対話の場合もあれば、インテリジェントな方法で人がサービスと対話できる高度なボットもあり、その場合は、パターン マッチング、状態の追跡、人工知能の技法が使用されて、既存のビジネス サービスと十分に統合されます。 

## <a name="building-a-bot"></a>ボットの作成 
C# または Node.js でボットを作成するには、好みの開発環境やコマンド ライン ツールを使用できます。 ボット開発の各ステージのためにツールが用意されており、ボットを作成して作業を始めるためにご使用いただけます。    

![ボットの概要](media/bot-service-overview.png) 

## <a name="plan"></a>プラン 
コードを書く前に、ボットの[デザイン ガイドライン](bot-service-design-principles.md) でベスト プラクティスを確認し、作成するボットに必要な事柄を見極めてください。 シンプルなボットを作成することもできますし、音声、言語理解、QnA (各種ソースからのナレッジを取り出してインテリジェントな回答を提供する機能) などの高度な機能を含めることもできます。  

> [!TIP]
> [Azure](https://portal.azure.com) アカウントを作成します。 

## <a name="build-your-bot"></a>ボットの作成 
作成するボットは会話型インターフェイスを実装した Web サービスであり、Bot Service とやり取りします。 このソリューションを多数の環境と言語で作成できます。また、Visual Studio や Yeoman で、あるいは Azure portal 内で直接、簡単に作業を開始するためのツールが提供されています。 使用できるツールとサービスのいくつかを以下に示します。

> [!TIP]
> [Azure portal](bot-service-quickstart.md) を使用してボットを作成します。 必要に応じて、次のようなコンポーネントを追加します。 
> - Language understanding ([LUIS](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/home))。 
> - ユーザーのたずねる質問に回答する [QnA Maker](https://docs.microsoft.com/en-us/azure/cognitive-services/qnamaker/home) ナレッジ ベース。  

## <a name="test-your-bot"></a>ボットをテストする 
ボットは様々なパーツが連携する複雑なアプリです。 そのため、他の複雑なアプリの場合と同様、興味深いバグが発生したり、期待どおりにボットが動作しなかったりすることがあります。 発行する前にボットをテストしてください。

> [!TIP] 
> - [Web チャット](bot-service-manage-test-webchat.md)を使用してボットをテストするか、[エミュレーター](bot-service-debug-emulator.md)を使用してボットをローカルでテストします。

## <a name="publish"></a>[発行] 
準備が整ったら、ボットを Azure に発行するか、独自の Web サービスまたはデータ センターに発行します。 継続的配置を設定するとローカルでボットを開発でき、ボットを GitHub や Visual Studio Team Services などのソース管理にチェックインする場合に便利です。 変更をソース リポジトリにチェックインし直すと、その変更が Azure に自動的にデプロイされます。

> [!Tip]
> - [コードをダウンロードして Azure に再デプロイします](bot-service-build-download-source-code.md)。

## <a name="connect"></a>接続          
ボットを、Facebook、Messenger、Kik、Skype、Slack、Microsoft Teams、Telegram、SMS 送信、Twilio、Cortana、Skype などのチャネルに接続し、対話を増やし、より多くのユーザーに接することができます。  
  
> [!TIP]
> - [追加するチャネルを選択する](bot-service-manage-channels.md)


## <a name="evaluate"></a>Evaluate 
Azure portal で収集されたデータを使用し、ボットの機能とパフォーマンスを強化する機会を特定します。 トラフィック、待ち時間、統合などのサービス レベルのデータやインストルメンテーション データを取得できます。 Analytics では、ユーザー、メッセージ、チャネル データに関する会話レベルのレポートも提供されます。

> [!Tip]
> - [分析情報を収集する](bot-service-manage-analytics.md) 


::: moniker-end

::: moniker range="azure-bot-service-4.0"

# <a name="about-azure-bot-service"></a>Azure Bot Service について

[!INCLUDE [pre-release-label](includes/pre-release-label.md)]

Azure Bot Service には、インテリジェントなボットの作成、テスト、配置、管理をすべて一元的に行うツールが備わっています。 SDK で提供されるモジュール式の拡張可能なフレームワーク、ツール、テンプレート、および AI サービスを使用することで、開発者は音声の使用、自然言語の解釈、質問と応答の処理などを行うボットを作成できます。

## <a name="what-is-a-bot"></a>ボットとは何ですか?
ボットは、コンピューターを使用しているという感覚が薄く、人 (または少なくとも知的なロボット) とやり取りしているように感じられるエクスペリエンスを提供します。 もう人が直接介入する必要がないような、ディナーの予約やプロファイル情報の収集などの繰り返し発生する単純なタスクを自動化されたシステムに移行するために使用できます。 ユーザーは、テキスト、インタラクティブ カード、および音声を使ってボットと会話します。 ボットとの対話では、簡単な質問と回答や、サービスへのアクセスをインテリジェントに提供する洗練された会話などが可能です。

ボットは、最新の Web アプリケーションによく似ていて、インターネットに接続し、API を使ってメッセージを送受信します。 ボットの機能は、ボットの種類によって大きく異なります。 最新のボット ソフトウェアでは、さまざまなテクノロジーとツールを利用して、多岐にわたるプラットフォーム上でより複雑なエクスペリエンスを提供しています。 一方、シンプルなボットでは、コードをほとんど使わずに、メッセージを受信し、そのメッセージをそのままユーザーに返すことができます。 

ボットは、ファイルの読み取りと書き込み、データベースや API の使用、一般的な計算タスクの実行といった、他の種類のソフトウェアと同じ操作を実行できます。 ボットの特徴は、人と人のコミュニケーションによく見られるメカニズムを使用する点にあります。 

ボットは、通常、次のコンポーネントで構成されています。

- Web サーバー (ほとんどの場合、パブリック インターネット上で利用可能な Web サーバー)
- Bot Builder SDK と Bot Builder ツール (ボットを開発するためのインターフェイスを提供します)
- Azure Cognitive Services
- Azure Storage

## <a name="building-a-bot"></a>ボットの作成 

Azure Bot Service は、このプロセスを容易にするために、統合された一連のツールとサービスを提供します。 開発者は好みの開発環境やコマンド ライン ツールを選んで、ボットを作成できます。 C#、JavaScript、および TypeScript 用の SDK が存在します  (Java と Python 用の SDK は現在開発中です)。ボット開発のさまざまなステージ用にツールが用意されており、ボットの設計と構築に役立てることができます。

![ボットの概要](media/bot-service-overview.png) 

### <a name="plan"></a>プラン
あらゆる種類のソフトウェアと同様に、目標、プロセス、およびユーザーのニーズを完全に理解することが、成功するボットを作成するプロセスにとって重要です。 コードを書く前に、ボットの[デザイン ガイドライン](bot-service-design-principles.md) でベスト プラクティスを確認し、作成するボットに必要な事柄を見極めてください。 単純なボットを作成することも、音声、自然言語の解釈、質問への応答などの高度な機能を含めることもできます。

### <a name="build"></a>構築
作成するボットは会話型インターフェイスを実装した Web サービスであり、Bot Framework サービスとやり取りして、メッセージとイベントを送受信します。 任意の数の環境と言語でボットを作成できます。 [Azure portal](bot-service-quickstart.md) 内でボット開発を開始することも、[[C#](dotnet/bot-builder-dotnet-sdk-quickstart.md) | [JavaScript](javascript/bot-builder-javascript-quickstart.md)] のテンプレートを使用してローカルで開発することもできます。

Azure Bot Service の一環として、ボットの機能の拡張に使用できる追加コンポーネントが用意されています。

| 機能 | 説明 | Link |
| --- | --- | --- |
| 自然言語処理を追加する | ボットによる自然言語の解釈、スペル ミスの理解、音声の使用、ユーザーの意図の認識を可能にします。 | [LUIS](~/v4sdk/bot-builder-howto-v4-luis.md) の使用方法 
| 質問に答える | ナレッジ ベースを追加して、より自然な会話方法でユーザーの質問に答えます。 | [QnA Maker](~/v4sdk/bot-builder-howto-qna.md) の使用方法 
| 複数のモデルを管理する | LUIS や QnA Maker などの複数のモデルを使用している場合、ボットの会話中に、いつ、どのモデルを使用するかをインテリジェントに判断します。 | [ディスパッチ](~/v4sdk/bot-builder-tutorial-dispatch.md) ツール|
| カードとボタンを追加する | グラフィックス、メニュー、カードなど、テキスト以外のメディアでユーザー エクスペリエンスを強化します。 | [カードを追加する](v4sdk/bot-builder-howto-add-media-attachments.md)方法 |

> [!NOTE]
> 上の表にすべてがリストされているわけではありません。 ボット機能についてさらに調べるには、[メッセージの送信](~/v4sdk/bot-builder-howto-send-messages.md)以下の左側の記事を参照してください。

さらに、ボット資産の作成、管理、テストに役立つコマンド ライン ツールも用意されています。 これらのツールにより、ボット構成ファイルの管理、LUIS アプリの構成、QnA ナレッジ ベースの構築、会話のモックなどを実行できます。 詳細については、コマンド ライン ツールの [readme](https://aka.ms/botbuilder-tools-readme) を参照してください。

また、SDK を通じて利用できる機能の多くを紹介しているさまざまな[サンプル](https://github.com/microsoft/botbuilder-samples)にアクセスすることもできます。 これらは、始めから豊富な機能を求めている開発者に最適です。

### <a name="test"></a>テスト 
ボットは様々なパーツが連携する複雑なアプリです。 そのため、他の複雑なアプリの場合と同様、興味深いバグが発生したり、期待どおりにボットが動作しなかったりすることがあります。 発行する前にボットをテストしてください。 ボットを使用できるようにリリースする前に、ボットをテストする方法をいくつか用意しています。

- [エミュレーター](bot-service-debug-emulator.md)を使ってボットをローカルでテストする。 Bot Framework Emulator は、チャット インターフェイスだけでなく、ボットの動作方法とその動作の理由を理解するのに役立つデバッグ ツールと質問ツールも提供するスタンドアロン アプリです。  このエミュレーターは、開発中のボット アプリケーションと共にローカルで実行できます。 
 
- [Web](bot-service-manage-test-webchat.md) 上でボットをテストする。 Azure portal から設定すると、Web チャット インターフェイスを介してボットにアクセスすることもできます。 Web チャット インターフェイスは、ボットの実行されているコードに直接アクセスできないテスターやその他の人に、ボットへのアクセスを許可するすばらしい方法です。

### <a name="publish"></a>[発行] 
ボットを Web 上で使用する準備が整ったら、ボットを [Azure](bot-builder-howto-deploy-azure.md) に発行するか、独自の Web サービスまたはデータ センターに発行します。 パブリック インターネット上のアドレスを取得することが、ボットをサイト上やチャット チャネル内で稼働させる第一歩です。

### <a name="connect"></a>接続          
ボットを、Facebook、Messenger、Kik、Skype、Slack、Microsoft Teams、Telegram、SMS 送信、Twilio、Cortana、Skype などのチャネルに接続できます。 Bot Framework は、こうしたさまざまなプラットフォームすべてとの間でメッセージを送受信するために必要な作業のほとんどを行います。ボット アプリケーションでは、接続されているチャネルの数や種類に関係なく、統一され、正規化されたメッセージのストリームを受信します。 チャネルの追加については、[チャネル](bot-service-manage-channels.md)に関するトピックをご覧ください。

### <a name="evaluate"></a>Evaluate 
Azure portal で収集されたデータを使用し、ボットの機能とパフォーマンスを強化する機会を特定します。 トラフィック、待ち時間、統合などのサービス レベルのデータやインストルメンテーション データを取得できます。 Analytics では、ユーザー、メッセージ、チャネル データに関する会話レベルのレポートも提供されます。 詳細は、[分析情報の収集方法](bot-service-manage-analytics.md)に関するページを参照してください。


## <a name="next-steps"></a>次の手順
ボットのこれらの[ケース スタディ](https://azure.microsoft.com/services/bot-service/)を確認するか、下記のリンクをクリックして、ボットを作成してください。
> [!div class="nextstepaction"]
> [ボットを作成する](bot-service-quickstart.md)

::: moniker-end
