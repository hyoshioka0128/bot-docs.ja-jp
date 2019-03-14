---
title: ボットをデプロイする | Microsoft Docs
description: 使用するボットを Azure クラウドにデプロイします。
keywords: ボットのデプロイ, azure へのボットのデプロイ, ボットの発行
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: get-started-article
ms.service: bot-service
ms.subservice: abs
ms.date: 02/13/2019
ms.openlocfilehash: 9cd2ed67110aa1611c41c33c31874f103e24b14d
ms.sourcegitcommit: b2245df2f0a18c5a66a836ab24a573fd70c7d272
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/07/2019
ms.locfileid: "57568199"
---
# <a name="deploy-your-bot"></a>ボットをデプロイする

[!INCLUDE [pre-release-label](./includes/pre-release-label.md)]

ご自分のボットを作成し、ローカルでテストした後に、それを Azure にデプロイして、どこからでもアクセスできるようにすることができます。 ボットを Azure にデプロイするには、使用するサービスの料金を支払う必要があります。 [課金とコスト管理](https://docs.microsoft.com/en-us/azure/billing/)に関する記事で、Azure の課金の確認、使用量とコストの監視、アカウントとサブスクリプションの管理の方法について説明されています。

この記事では、C# ボットと JavaScript ボットを Azure にデプロイする方法を紹介します。 ボットのデプロイに必要なものを完全に理解するために、手順を実行する前にこの記事を確認することをお勧めします。

## <a name="prerequisites"></a>前提条件

- 最新バージョンの [msbot](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/MSBot) ツールをインストールします。
- ローカル コンピューター上で開発された [CSharp](./dotnet/bot-builder-dotnet-sdk-quickstart.md) ボットまたは [JavaScript](./javascript/bot-builder-javascript-quickstart.md) ボット。

## <a name="1-prepare-for-deployment"></a>1.デプロイの準備をする
デプロイ プロセスでは、ローカルのボットをデプロイできる、ターゲットとなる Web アプリ ボットが Azure に必要です。 ターゲットの Web アプリ ボットと、Azure で一緒にプロビジョニングされるリソースは、デプロイするローカルのボットによって使用されます。 お持ちのローカルのボットでは、必要な Azure リソースがすべてプロビジョニングされているわけではないため、これが必要になります。 ターゲットの Web アプリ ボットを作成すると、次のリソースが自動的にプロビジョニングされます。
-   Web アプリ ボット – ローカルのボットをデプロイするためにこのボットを使用します。
-   App Service プラン - App Service アプリの実行に必要なリソースを提供します。
-   App Service - Web アプリケーションをホストするためのサービス
-   ストレージ アカウント - すべての Azure Storage データ オブジェクト (BLOB、ファイル、キュー、テーブル、ディスク) が含まれます。

ターゲットの Web アプリ ボットの作成時に、ボット用のアプリ ID とパスワードも生成されます。 Azure では、アプリ ID とパスワードによって、[サービスの認証と承認](https://docs.microsoft.com/azure/app-service/overview-authentication-authorization)がサポートされます。 ローカルのボット コードで使用するために、後ほどこの情報の一部を取得します。 

> [!IMPORTANT]
> サービスのボット テンプレートの言語は、お客様のボットが記述されている言語と一致する必要があります。

使用するボットが Azure に既に作成されている場合、新しい Web アプリ ボットの作成は省略できます。

1. [Azure Portal](https://portal.azure.com) にログインします。
1. Azure portal の左上隅にある **[新しいリソースの作成]** リンクをクリックし、**[AI + Machine Learning] > [Web App Bot]** を選択します。
1. Web App Bot に関する情報が含まれた 新しいブレードが開きます。 
1. **[ボット サービス]** ブレードで、ボットに関する必要な情報を指定します。
1. **[作成]** をクリックしてサービスを作成し、ボットをクラウドにデプロイします。 このプロセスには数分かかることがあります。

### <a name="download-the-source-code"></a>ソース コードをダウンロードする
ターゲットの Web アプリ ボットを作成したら、ボット コードを Azure portal からローカル コンピューターにダウンロードする必要があります。 コードをダウンロードする理由は、[.bot ファイル](./v4sdk/bot-file-basics.md)に含まれるサービス参照を取得することにありますます。 これらのサービス参照は、Web アプリ ボット、App Service プラン、App Service、およびストレージ アカウント用のものです。 

1. **[ボット管理]** セクションで **[ビルド]** をクリックします。
1. 右側のウィンドウで **[ボットのソース コードをダウンロードする]** リンクをクリックします。
1. 指示に従ってコードをダウンロードし、フォルダーを解凍します。
    1. [!INCLUDE [download keys snippet](~/includes/snippet-abs-key-download.md)]

### <a name="decrypt-the-bot-file"></a>.bot ファイルの暗号化を解除する

Azure portal からダウンロードしたソース コードには、暗号化された .bot ファイルが含まれています。 ローカル .bot ファイルに値をコピーするために、ファイルの暗号化を解除する必要があります。 暗号化されたものではない、実際のサービス参照をコピーできるようにするために、この手順が必要になります。  

1. Azure portal で、ボットの Web App Bot リソースを開きます。
1. ボットの **[アプリケーション設定]** を開きます。
1. **[アプリケーション設定]** ウィンドウで、**[アプリケーション設定]** まで下へスクロールします。
1. **botFileSecret** を探してその値をコピーします。
1. `msbot cli` を使用してファイルの暗号化を解除します。

    ```cmd
    msbot secret --bot <name-of-bot-file> --secret "<bot-file-secret>" --clear
    ```

### <a name="update-your-local-bot-file"></a>ローカル .bot ファイルを更新する

暗号化解除した .bot ファイルを開きます。 `services` セクションに表示されている**すべて**のエントリをコピーし、ローカル .bot ファイルに追加します。 重複するすべてのサービス エントリまたはサービス ID を解決します。 ご使用のボットで使用する追加のサービス参照は保持します。 例: 

```json
"services": [
    {
        "type": "abs",
        "tenantId": "<tenant-id>",
        "subscriptionId": "<subscription-id>",
        "resourceGroup": "<resource-group-name>",
        "serviceName": "<bot-service-name>",
        "name": "<friendly-service-name>",
        "id": "1",
        "appId": "<app-id>"
    },
    {
        "type": "blob",
        "connectionString": "<connection-string>",
        "tenantId": "<tenant-id>",
        "subscriptionId": "<subscription-id>",
        "resourceGroup": "<resource-group-name>",
        "serviceName": "<blob-service-name>",
        "id": "2"
    },
    {
        "type": "endpoint",
        "appId": "",
        "appPassword": "",
        "endpoint": "<local-endpoint-url>",
        "name": "development",
        "id": "3"
    },
    {
        "type": "endpoint",
        "appId": "<app-id>",
        "appPassword": "<app-password>",
        "endpoint": "<hosted-endpoint-url>",
        "name": "production",
        "id": "4"
    },
    {
        "type": "appInsights",
        "instrumentationKey": "<instrumentation-key>",
        "applicationId": "<appinsights-app-id>",
        "apiKeys": {},
        "tenantId": "<tenant-id>",
        "subscriptionId": "<subscription-id>",
        "resourceGroup": "<resource-group>",
        "serviceName": "<appinsights-service-name>",
        "id": "5"
    }
],
```

ファイルを保存します。

発行前に、msbot ツールを使用して新しいシークレットを生成し、.bot ファイルを暗号化することができます。 .bot ファイルを再暗号化した場合は、Azure portal でボットの **botFileSecret** を更新して新しいシークレットを含めます。

```cmd
msbot secret --bot <name-of-bot-file> --new
```

### <a name="setup-a-repository"></a>リポジトリを設定する

継続的配置をサポートするには、お好みの Git ソース管理プロバイダーを使用して、Git リポジトリを作成します。 リポジトリにコードをコミットします。

[リポジトリの準備](https://docs.microsoft.com/azure/app-service/deploy-continuous-deployment#prepare-your-repository)に関するページの説明にあるように、リポジトリのルートに適切なファイルがあることを確認してください。

### <a name="update-app-settings-in-azure"></a>Azure でアプリ設定を更新する
ローカルのボットでは暗号化された .bot ファイルを使用しませんが、Azure portal は暗号化された .bot ファイルを使用するように構成されています。 これを解決するには、Azure のボット設定に保存されている **botFileSecret** を削除します。
1. Azure portal で、ボットの **Web App Bot** リソースを開きます。
1. ボットの **[アプリケーション設定]** を開きます。
1. **[アプリケーション設定]** ウィンドウで、**[アプリケーション設定]** まで下へスクロールします。
1. **botFileSecret** を探してそれを削除します。 (.bot ファイルを再暗号化した場合は、**botFileSecret** に新しいシークレットが含まれていることを確認します。この設定を削除**しないでください**。)
1. リポジトリにチェックインしたファイルと一致するように、ボット ファイルの名前を更新します。
1. 変更を保存します。

## <a name="2-deploy-using-azure-deployment-center"></a>2.Azure デプロイ センターを使用してデプロイする

ここで、ボット コードを Azure にアップロードする必要があります。 「[Continuous deployment to Azure App Service (Azure App Service への継続的配置)](https://docs.microsoft.com/azure/app-service/deploy-continuous-deployment)」の指示に従います。

`App Service Kudu build server` を使用してビルドすることをお勧めします。

継続的配置を構成したら、リポジトリにコミットした変更が発行されます。 ただし、お使いのボットにサービスを追加する場合は、それらのエントリを .bot ファイルに追加する必要があります。

## <a name="3-test-your-deployment"></a>手順 3.デプロイのテスト

デプロイが成功したら、数秒待ちます。必要に応じて、Web アプリを再起動してキャッシュをクリアします。 Web App Bot ブレードに戻り、Azure portal で提供されている Web チャットを使用してテストします。

## <a name="additional-resources"></a>その他のリソース

- [継続的なデプロイに関する一般的な問題の調査方法](https://github.com/projectkudu/kudu/wiki/Investigating-continuous-deployment)

<!--

## Prerequisites

[!INCLUDE [prerequisite snippet](~/includes/deploy/snippet-prerequisite.md)]


## Deploy JavaScript and C# bots using az cli

You've already created and tested a bot locally, and now you want to deploy it to Azure. These steps assume that you have created the required Azure resources.

[!INCLUDE [az login snippet](~/includes/deploy/snippet-az-login.md)]

### Create a Web App Bot

If you don't already have a resource group to which to publish your bot, create one:

[!INCLUDE [az create group snippet](~/includes/deploy/snippet-az-create-group.md)]

[!INCLUDE [az create web app snippet](~/includes/deploy/snippet-create-web-app.md)]

Before proceeding, read the instructions that apply to you based on the type of email account you use to log in to Azure.

#### MSA email account

If you are using an [MSA](https://en.wikipedia.org/wiki/Microsoft_account) email account, you will need to create the app ID and app password on the Application Registration Portal to use with `az bot create` command.

[!INCLUDE [create bot msa snippet](~/includes/deploy/snippet-create-bot-msa.md)]

#### Business or school account

[!INCLUDE [create bot snippet](~/includes/deploy/snippet-create-bot.md)]

### Download the bot from Azure

Next, download the bot you just created. 
[!INCLUDE [download bot snippet](~/includes/deploy/snippet-download-bot.md)]

[!INCLUDE [download keys snippet](~/includes/snippet-abs-key-download.md)]

### Decrypt the downloaded .bot file and use in your project

The sensitive information in the .bot file is encrypted.

[!INCLUDE [decrypt bot snippet](~/includes/deploy/snippet-decrypt-bot.md)]

### Update the .bot file

If your bot uses LUIS, QnA Maker, or Dispatch services, you will need to add references to them to your .bot file. Otherwise, you can skip this step.

1. Open your bot in the BotFramework Emulator, using the new .bot file. The bot does not need to be running locally.
1. In the **BOT EXPLORER** panel, expand the **SERVICES** section.
1. To add references to LUIS apps, click the plus-sign (+) to the right of **SERVICES**.
   1. Select **Add Language Understanding (LUIS)**.
   1. If it prompts you to log into your Azure account, do so.
   1. It presents a list of LUIS applications you have access to. Select the ones for your bot.
1. To add references to a QnA Maker knowledge base, click the plus-sign (+) to the right of **SERVICES**.
   1. Select **Add QnA Maker**.
   1. If it prompts you to log into your Azure account, do so.
   1. It presents a list of knowledge bases you have access to. Select the ones for your bot.
1. To add references to Dispatch models, click the plus-sign (+) to the right of **SERVICES**.
   1. Select **Add Dispatch**.
   1. If it prompts you to log into your Azure account, do so.
   1. It presents a list of Dispatch models you have access to. Select the ones for your bot.

### Test your bot locally

At this point, your bot should work the same way it did with the old .bot file. Make sure that it works as expected with the new .bot file.

### Publish your bot to Azure

[!INCLUDE [publish snippet](~/includes/deploy/snippet-publish.md)]


[!INCLUDE [clear encryption snippet](~/includes/deploy/snippet-clear-encryption.md)]

## Additional resources

[!INCLUDE [additional resources snippet](~/includes/deploy/snippet-additional-resources.md)]

## Next steps
> [!div class="nextstepaction"]
> [Set up continous deployment](bot-service-build-continuous-deployment.md)

-->
