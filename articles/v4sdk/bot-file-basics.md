---
title: .bot ファイルでリソースを管理する | Microsoft Docs
description: ボット ファイルの目的と用途について説明します。
keywords: ボット ファイル, .bot, .bot ファイル, msbot, ボット リソース, ボット リソースの管理
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 03/11/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: db7957c611df4a3010469f86f51184b52d49addb
ms.sourcegitcommit: 4139ef7ebd8bb0648b8af2406f348b147817d4c7
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/19/2019
ms.locfileid: "58073858"
---
# <a name="manage-resources-with-a-bot-file"></a>.bot ファイルでリソースを管理する

ボットでは、通常、[LUIS.ai](https://luis.ai)、[QnaMaker.ai](https://qnamaker.ai) などのさまざまなサービスが多数使用されます。 ボットを開発しているときに、使用中のサービスに関するメタデータを格納する統一された場所がありません。  これでは、ボットの全体像を確認するためのツールを構築することができません。

この問題に対処するために、Microsoft では 1 か所にすべてのサービス参照をまとめる場所として **.bot ファイル**を作成し、ツールを使用できるようにしました。  たとえば、Bot Framework Emulator ([V4](https://aka.ms/Emulator-wiki-getting-started)) では、.bot ファイルを使用して、ボットが使用する接続済みサービスの統合ビューが作成されます。  

.bot ファイルにより、次のようなサービスを登録できます。

* **Localhost**: ローカル デバッガー エンドポイント
* [**Azure Bot Service**](https://azure.microsoft.com/en-us/services/bot-service/): Azure Bot Service 登録。
* [**LUIS.AI**](https://www.luis.ai/): LUIS により、ボットが、自然言語を使用するユーザーと通信できるようになります。 
* [**QnA Maker**](https://qnamaker.ai/): FAQ URL、構造化されたドキュメント、または本文に基づいて、単純な質疑応答ボットを数分でビルド、トレーニング、および公開します。
* [**ディスパッチ**](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/Dispatch): 複数のサービスにまたがるディスパッチのモデル。
* [**Azure Application Insights**](https://azure.microsoft.com/en-us/services/application-insights/): 分析情報とボット分析の実現。
* [**Azure Blob Storage**](https://azure.microsoft.com/en-us/services/storage/blobs/): ボット状態の保持。 
* [**Azure Cosmos DB**](https://azure.microsoft.com/en-us/services/cosmos-db/): ボット状態を保持するためのグローバル分散マルチモデル データベース サービス。

お使いのボットでは、これ以外のカスタム サービスを使用している可能性があります。 [汎用サービス](https://github.com/Microsoft/botbuilder-tools/blob/master/packages/MSBot/docs/add-services.md)機能を活用すると、汎用サービスの構成に接続できます。

## <a name="when-is-a-bot-file-created"></a>.bot ファイルが作成されるタイミング 
- [Azure Bot Service](https://ms.portal.azure.com/#blade/Microsoft_Azure_Marketplace/GalleryResultsListBlade/selectedSubMenuItemId/%7B%22menuItemId%22%3A%22gallery%2FCognitiveServices_MP%2FBotService%22%2C%22resourceGroupId%22%3A%22%22%2C%22resourceGroupLocation%22%3A%22%22%2C%22dontDiscardJourney%22%3Afalse%2C%22launchingContext%22%3A%7B%22source%22%3A%5B%22GalleryFeaturedMenuItemPart%22%5D%2C%22menuItemId%22%3A%22CognitiveServices_MP%22%2C%22subMenuItemId%22%3A%22BotService%22%7D%7D) を使用してボットを作成すると、.bot ファイルが自動的に作成され、接続済みサービスの一覧がプロビジョニングされます。 .bot は既定で暗号化されます。
- Visual Studio 用の Bot Framework V4 SDK [テンプレート](https://marketplace.visualstudio.com/items?itemName=BotBuilder.botbuilderv4)または Bot Builder [Yeoman Generator](https://www.npmjs.com/package/generator-botbuilder) を使用してボットを作成すると、.bot ファイルが自動的に作成されます。 このフローでは接続済みサービスはプロビジョニングされず、ボット ファイルは暗号化されません。
- [BotBuilder サンプル](https://github.com/Microsoft/botbuilder-samples)で開始した場合、Bot Framework V4 SDK 用の各サンプルに .bot ファイルが含まれています。その .bot ファイルは暗号化されていません。 
- [MSBot](https://github.com/Microsoft/botbuilder-tools/blob/master/packages/MSBot/README.md) ツールを使用して、ボット ファイルを作成することもできます。

## <a name="what-does-a-bot-file-look-like"></a>ボット ファイルの外観 
[.bot](https://github.com/Microsoft/botbuilder-tools/blob/master/packages/MSBot/docs/sample-bot-file.json) ファイルのサンプルを確認してください。
.bot ファイルの暗号化と復号化の詳細については、「[Bot Secrets (ボット シークレット)](https://github.com/Microsoft/botbuilder-tools/blob/master/packages/MSBot/docs/bot-file-encryption.md)」を参照してください。

## <a name="why-do-i-need-a-bot-file"></a>ボット ファイルが必要な理由

.bot ファイルは、Bot Framework SDK でボットを作成するための要件では**ありません**。 appsettings.json、web.config、env、keyvault など、ボットが依存するサービス参照やキーの追跡に適していると思われる任意のメカニズムは、引き続き使用できます。 Emulator を使用してボットをテストするには、.bot ファイルが必要です。 幸いにも、Emulator でテスト用 .bot ファイルを作成することができます。 それには、Emulator を起動して、ウェルカム ページで **[新しいボット構成を作成する]** リンクをクリックします。 表示されたダイアログ ボックスで、**ボット名**と**エンドポイント URL** を入力します。 次に、接続します。

.bot ファイルを使用するメリットは以下のとおりです。
- 使用する言語やプラットフォームに関係なくリソースを格納する標準的な方法が提供されます。   
- Bot Framework Emulator と CLI ツールが、(.bot ファイルの) 一貫性のある形式で接続済みサービスの追跡機能を使用し、こうした機能と適切に連携します 
- 明確に定義されたスキーマ (.bot ファイル) がないと、サービスの作成および管理に関連した洗練されたツール ソリューションが難しくなります。  


## <a name="using-bot-file-in-your-bot-framework-sdk-bot"></a>Bot Framework SDK ボットでの .bot ファイルの使用

.bot ファイルを使用すると、お使いのボットのコードでサービス構成情報を取得できます。 [C#](https://www.nuget.org/packages/Microsoft.Bot.Configuration) と [JS](https://www.npmjs.com/package/botframework-config) で使用できる BotFramework 構成ライブラリは、ボット ファイルを読み込むときに役に立ち、適切なサービス構成情報を照会および取得するための複数のメソッドをサポートしています。

## <a name="additional-resources"></a>その他のリソース
ボット ファイルの使用に関する詳細については、[MSBot](https://github.com/Microsoft/botbuilder-tools/blob/master/packages/MSBot/README.md) の readme ファイルを参照してください。
