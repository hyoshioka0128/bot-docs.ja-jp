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
ms.openlocfilehash: e888b2473269cf576fd9edda0d99a30aa6212f7b
ms.sourcegitcommit: b2245df2f0a18c5a66a836ab24a573fd70c7d272
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/07/2019
ms.locfileid: "57571879"
---
# <a name="enterprise-bot-template---getting-started"></a>Enterprise Bot Template - はじめに

> [!NOTE]
> このトピックは、SDK の v4 バージョンに適用されます。 

## <a name="prerequisites"></a>前提条件

次をインストールします。
- [Enterprise Bot Template VSIX](https://marketplace.visualstudio.com/items?itemName=BotBuilder.botbuilderv4enterprise)
- [.NET Core SDK](https://www.microsoft.com/net/download) (最新バージョン)
- [ノード パッケージ マネージャー](https://nodejs.org/en/)
- [Bot Framework Emulator](https://docs.microsoft.com/en-us/azure/bot-service/bot-service-debug-emulator?view=azure-bot-service-4.0) (最新バージョン)
- [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest)
- [Azure Bot Service CLI ツール](https://github.com/Microsoft/botbuilder-tools) (最新バージョン)
    ```shell
    npm install -g ludown luis-apis qnamaker botdispatch msbot chatdown
    ```
- [LuisGen](https://github.com/Microsoft/botbuilder-tools/blob/master/packages/LUISGen/src/npm/readme.md)
    ```shell
    dotnet tool install -g luisgen
    ```

## <a name="create-your-bot-project"></a>ボット プロジェクトを作成する
1. Visual Studio で、**[ファイル] > [新しいプロジェクト]** をクリックします。
1. **ボット**で、**Enterprise Bot Template** を選択します。

![新しいプロジェクト テンプレート ファイル](media/enterprise-template/new_project.jpg)

1. プロジェクトに名前を付けて、**[作成]** をクリックします。
1. プロジェクトを右クリックし、 **[ビルド]** をクリックして、NuGet パッケージを復元します。

## <a name="deploy-your-bot"></a>ボットをデプロイする

Enterprise Template Bot のエンド ツー エンドの操作には、次の Azure サービスが必要です。
- Azure Web アプリ
- Azure Storage アカウント (トランスクリプト)
- Azure Application Insights (テレメトリ)
- Azure CosmosDb (会話状態ストレージ)
- Azure Cognitive Services - Language Understanding
- Azure Cognitive Services - QnA Maker (Azure Search、Azure Web アプリを含む)

次の手順は、指定されたデプロイ スクリプトを使用してこれらのサービスをデプロイするのに役立ちます。

1. LUIS のオーサリング キーを取得します。
   - ご自身のデプロイ リージョンの正しい LUIS ポータルについては、[こちら](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/luis-reference-regions)のドキュメント ページをご覧ください。 www.luis.ai は米国リージョンを指し、このポータルから取得したオーサリング キーはヨーロッパのデプロイでは動作しないことに注意してください。
   - サインインした後は、右上隅にある自分の名前をクリックします。
   - **[設定]** を選択し、後で使用できるようにオーサリング キーをメモします。
    
    ![LUIS のオーサリング キーのスクリーンショット](./media/enterprise-template/luis_authoring_key.jpg)

1. コマンド プロンプトを開きます。
1. Azure CLI を使って、ご自身の Azure アカウントにログインします。 Azure portal の[サブスクリプション](https://ms.portal.azure.com/#blade/Microsoft_Azure_Billing/SubscriptionsBlade) ページで、アクセス先のサブスクリプションの一覧を確認できます。
    ```shell
    az login
    az account set --subscription "YOUR_SUBSCRIPTION_NAME"
    ```

1. msbot 複製サービス コマンドを実行して、ご自身のサービスをデプロイし、お使いのプロジェクトで .bot ファイルを構成します。 **注: デプロイの完了後、後で使用できるように、コマンド プロンプト ウィンドウに表示されているボット ファイル シークレットをメモしておく必要があります。**

    ```shell
    msbot clone services --name "YOUR-BOT-NAME" --luisAuthoringKey "YOUR_AUTHORING_KEY" --folder "DeploymentScripts\LOCALE_FOLDER" --location "REGION"
    ```

    > **注**:
    >- **YOUR-BOT-NAME** パラメーターは**グローバルに一意**で、小文字、数字、ハイフン ("-") のみで構成されている必要があります。
    >- この手順で指定したデプロイ リージョンが、ご自身の LUIS オーサリング キー ポータルのリージョンと一致していることを確認します (例: luis.ai の場合は westus、eu.luis.ailuis.ai の場合は westeurope)。
    >- MSA アプリ ID とパスワードのプロビジョニングでは、一部のユーザーに発生する可能性がある既知の問題があります。 このエラー `ERROR: Unable to provision MSA id automatically. Please pass them in as parameters and try again` が発生した場合は、[https://apps.dev.microsoft.com](https://apps.dev.microsoft.com) にアクセスし、新しいアプリケーションを手動で作成して、アプリ ID とパスワード/シークレットをメモしてください。 上記の msbot 複製サービス コマンドを再度実行し、2 つの新しい引数 `--appId` および `--appSecret` と、取得した値を指定します。 パスワードでは、シェルによってコマンドと解釈される可能性のある特殊文字のエスケープが必要になる場合があります。
    >   - "*Windows コマンド プロンプト*" では、appSecret を二重引用符で囲みます。 例: `msbot clone services --name xxxx --luisAuthoringKey xxxx --location xxxx --folder bot.recipe --appSecret "YOUR_APP_SECRET"`
    >   - *Windows PowerShell の場合、--% 引数の後に appSecret を渡します。 例: `msbot clone services --name xxxx --luisAuthoringKey xxxx --location xxxx --folder bot.recipe --% --appSecret "YOUR_APP_SECRET"`
    >   - "*MacOS または Linux*" の場合、appSecret を単一引用符で囲みます。 例: `msbot clone services --name xxxx --luisAuthoringKey xxxx --location xxxx --folder bot.recipe --appSecret 'YOUR_APP_SECRET'`

1. デプロイが完了したら、ご自身のボット ファイル シークレットで **appsettings.json** を更新します。 
    
    ```
    "botFilePath": "./YOUR_BOT_FILE.bot",
    "botFileSecret": "YOUR_BOT_SECRET",
    ```
1. 次のコマンドを実行し、対象の Application Insights インスタンスの InstrumentationKey を取得します。
    ```
    msbot list --bot YOUR_BOT_FILE.bot --secret "YOUR_BOT_SECRET"
    ```

1. ご自身のインストルメンテーション キーで **appsettings.json** を更新します。

    ```
    "ApplicationInsights": {
        "InstrumentationKey": "YOUR_INSTRUMENTATION_KEY"
    }
    ```

## <a name="test-your-bot"></a>ボットをテストする

完了後、開発環境内でボット プロジェクトを実行し、**Bot Framework Emulator** を開きます。 エミュレーター内で、**[ファイル] > [Open Bot]\(ボットを開く\)** をクリックし、ご自身のディレクトリの .bot ファイルに移動します。

![エミュレーター開発エンドポイントのスクリーンショット](./media/enterprise-template/development_endpoint.jpg)

会話が開始されると、紹介メッセージが表示されます。

「```hi```」と入力して、すべて機能していることを確認します。

## <a name="publish-your-bot"></a>ボットを公開する

テストはエンド ツー エンドでローカルで実行できます。 追加テストを行うボットを Azure にデプロイする準備ができたら、次のコマンドを使用してソース コードを公開できます。

```shell
az bot publish -g YOUR-BOT-NAME -n YOUR-BOT-NAME --proj-name YOUR-BOT-NAME.csproj --version v4
```

## <a name="view-your-bot-analytics"></a>ボットの分析を表示する
Enterprise Bot Template には Power BI ダッシュボードが付属しています。このダッシュボードは、Application Insights サービスに接続して、会話の分析を提供します。 ご自身のボットをローカルでテストしたら、このダッシュボードを開いてそのデータを確認できます。 

1. [こちら](https://github.com/Microsoft/AI/blob/master/solutions/analytics/ConversationalAnalyticsSample_02132019.pbit)から Power BI ダッシュ ボード (.pbit ファイル) をダウンロードします。
1. [Power BI Desktop](https://powerbi.microsoft.com/en-us/desktop/) でダッシュボードを開きます。
1. Application Insights アプリケーションの ID (.bot ファイルにあります) を入力します。

    ![ボット ファイルの AppInsights アプリ ID の場所](./media/enterprise-template/appInsights_appId.jpg)

1. Power BI ダッシュ ボード機能の詳細については、[こちら](https://github.com/Microsoft/AI/tree/master/solutions/analytics)をご覧ください。

# <a name="next-steps"></a>次の手順

ご自身のボットを追加設定なしで適切にデプロイしたら、シナリオやニーズに応じてボットをカスタマイズできます。 続けて[ボットをカスタマイズ](bot-builder-enterprise-template-customize.md)してください。
