---
title: Enterprise Bot Template のデプロイ | Microsoft Docs
description: お使いの Enterprise Bot をサポートするすべての Azure リソースをデプロイする方法について説明します
author: darrenj
ms.author: darrenj
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 09/18/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 32be8e2a4047c3c25dcdf2598eea3a7bbd12fbcc
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/24/2018
ms.locfileid: "49999029"
---
# <a name="enterprise-bot-template---deploying-your-bot"></a>Enterprise Bot Template - ボットのデプロイ

> [!NOTE]
> このトピックは、SDK の v4 バージョンに適用されます。 

## <a name="prerequisites"></a>前提条件

- [Node パッケージ マネージャー](https://nodejs.org/en/)がインストールされていることを確認する。

- Azure Bot Service コマンド ライン (CLI) ツールをインストールする。 以前にツールを使用したことがある場合でも、最新バージョンであることを確認するために、これを行うことが重要です。

```shell
npm install -g ludown luis-apis qnamaker botdispatch msbot luisgen chatdown
```

- Azure コマンド ライン ツール (CLI) を[こちら](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli-windows?view=azure-cli-latest)からインストールする。 Azure Bot Service コマンド ライン (CLI) ツールを既にインストールしてある場合は、現在のバージョンをアンインストールしたうえで新しいバージョンをインストールして、最新のバージョンに更新してください。

- Bot Service の AZ 拡張機能をインストールする
```shell
az extension add -n botservice
```

## <a name="configuration"></a>構成

- LUIS のオーサリング キーを取得する
   - デプロイする予定のリージョンの正しい LUIS ポータルについては、[こちら](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/luis-reference-regions)のドキュメント ページをご覧ください。 
   - サインインした後は、右上隅にある自分の名前をクリックします。
   - [設定] を選択し、次の手順に備えてオーサリング キーをメモします。

## <a name="deployment"></a>Deployment

>複数の Azure サブスクリプションを持っており、そのデプロイで正しいサブスクリプションが選択されていることを確認するには、続行する前に次のコマンドを実行します。

 お使いの Azure アカウントへのブラウザーのログイン手順に従います。
```shell
az login
az account list
az account set --subscription "YOUR_SUBSCRIPTION_NAME"
```

Enterprise Template Bot のエンド ツー エンドの操作には、次の依存関係が必要です。
- Azure Web アプリ
- Azure Storage アカウント (トランスクリプト)
- Azure Application Insights (テレメトリ)
- Azure Cosmos DB (状態)
- Azure Cognitive Services - Language Understanding
- Azure Cognitive Services - QnA Maker (Azure Search、Azure Web アプリを含む)
- Azure Cognitive Services - Content Moderator (省略可能な手動の手順)

新しいボット プロジェクトには、`msbot clone services` コマンドを使用して対象の Azure サブスクリプションに上記すべてのサービスのデプロイを自動化し、プロジェクト内の .bot ファイルがそれらすべてのサービス (ボットのシームレスな操作を可能にするキーなど) に更新されるようにする、デプロイ用のレシピが用意されています。

> デプロイ後、作成したサービスの価格レベルを確認し、シナリオに合わせて調整します。

作成したプロジェクト内の README.md には、作成したボット名とジェネリック バージョンで更新された msbot クローン サービスのコマンド ラインのサンプルが含まれています。以下にそれを示します。 前の手順からオーサリング キーが更新されていることを確認し、使用する Azure データセンターの場所 (米国西部、西ヨーロッパなど) を選択します。

> 前の手順で取得した LUIS のオーサリング キーが、以下で指定するリージョンのキーであることを確認します。

```shell
msbot clone services --name "YOUR_BOT_NAME" --luisAuthoringKey "YOUR_AUTHORING_KEY" --folder "DeploymentScripts\msbotClone" --location "westus"
```

msbot ツールに、場所や SKU など、デプロイ計画の概要が表示されます。 続行する前に確認してください。

![デプロイの確認](./media/enterprise-template/EnterpriseBot-ConfirmDeployment.png)

>デプロイ完了後、後続の手順で必要になるため、提供された .bot ファイルのシークレットを**必ず**メモしてください。

- `appsettings.json` ファイルを新しく作成された .bot ファイル名と .bot ファイルのシークレットで更新します。
- 次のコマンドを実行し、対象の Application Insights インスタンスの InstrumentationKey を取得して、`appsettings.json` ファイル内の InstrumentationKey キーを更新します。

`msbot list --bot YOURBOTFILE.bot --secret YOUR_BOT_SECRET`

        {
          "botFilePath": ".\\YOURBOTFILE.bot",
          "botFileSecret": "YOUR_BOT_SECRET",
          "ApplicationInsights": {
            "InstrumentationKey": "YOUR_INSTRUMENTATION_KEY"
          }
        }

## <a name="testing"></a>テスト

完了後、開発環境内でボット プロジェクトを実行し、Bot Framework Emulator を開きます。 エミュレーター内で、[ファイル] メニューから [Open Bot]\(ボットを開く\) を選択し、対象の .bot ファイルがあるディレクトリに移動します。

その後、「```hi```」と入力してすべて機能していることを確認します。

Bot Framework Emulator に問題がある場合、まずは最新の Bot Framework Emulator を使用していることを確かめてください。 古いバージョンのエミュレーターが正しく更新されていない場合は、エミュレーターをアンインストールしてインストールし直します。

## <a name="deploy-to-azure"></a>[Deploy to Azure (Azure へのデプロイ)]

テストはエンド ツー エンドでローカルで実行できます。 追加テストを行うボットを Azure にデプロイする準備ができたら、次のコマンドを使用してソース コードを公開できます。これは、ソース コードの更新をプッシュするときにいつでも実行できます。

```shell
az bot publish -g YOUR_BOT_NAME -n YOUR_BOT_NAME --proj-file YOUR_BOT_NAME.csproj --sdk-version v4
```

## <a name="enabling-more-scenarios"></a>他のシナリオの有効化

お使いのボット プロジェクトには追加の機能が用意されており、次の手順を通じて有効にできます。

### <a name="authentication"></a>Authentication

認証を有効にするには、Azure portal のボットの [設定] 内で、[Authentication Connection Name]\(認証の接続名\) を構成した後に、次の手順を実行します。 追加情報はこちらの[ドキュメント](https://docs.microsoft.com/en-us/azure/bot-service/bot-builder-tutorial-authentication?view=azure-bot-service-3.0)で確認できます。

MainDialog コンストラクターで、次のように `SignInDialog` を登録します。
    
`AddDialog(new SignInDialog(_services.AuthConnectionName));`

次のコードを目的の場所にあるコード内に追加し、簡単なログイン フローをテストします。
    
`var signInResult = await dc.BeginDialogAsync(nameof(SignInDialog));`

### <a name="content-moderation"></a>コンテンツ モデレート

コンテンツ モデレーションは、ボットに送信されたメッセージ内の、個人を特定できる情報 (PII) と成人向けコンテンツを特定するために使用できます。 この機能を有効にするには、Azure portal に移動し、新しいコンテンツ モデレーター サービスを作成します。 サブスクリプション キーと .bot ファイルを構成するリージョンを収集します。 

> この手順は、今後自動化されます。

起動時に次のコードを service.AddBot<>() メソッドの一番下に追加し、常にコンテンツ モデレーションを実行できるようにします。 コンテンツ モデレーションの結果は、ボットの状態を介してアクセスできます。 
    
```
    // Content Moderation Middleware (analyzes incoming messages for inappropriate content including PII, profanity, etc.)
    var moderatorService = botConfig.Services.Where(s => s.Name == ContentModeratorMiddleware.ServiceName).FirstOrDefault();
    if (moderatorService != null)
    {
        var moderator = moderatorService as GenericService;
        var moderatorKey = moderator.Configuration["subscriptionKey"];
        var moderatorRegion = moderator.Configuration["region"];
        var moderatorMiddleware = new ContentModeratorMiddleware(moderatorKey, moderatorRegion);
        options.Middleware.Add(moderatorMiddleware);
    }
```
ダイアログ スタック内からこれを呼び出してミドルウェアの結果にアクセスします。
```     
    var cm = dc.Context.TurnState.Get<Microsoft.CognitiveServices.ContentModerator.Models.Screen>(ContentModeratorMiddleware.TextModeratorResultKey);
```

## <a name="customize-your-bot"></a>ボットをカスタマイズする

ボットを (追加設定なしで) デプロイした後は、シナリオやニーズに応じてボットをカスタマイズできます。 続けて[ボットをカスタマイズ](bot-builder-enterprise-template-customize.md)してください。
