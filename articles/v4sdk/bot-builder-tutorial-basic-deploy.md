---
title: 基本的なボットの作成とデプロイに関するチュートリアル | Microsoft Docs
description: 基本的なボットを作成し、それを Azure にデプロイする方法について説明します。
keywords: エコー ボット, デプロイ, Azure, チュートリアル
author: Ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 05/23/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 305e89f5bef4b50e99b24780b0e31580e3b38fb9
ms.sourcegitcommit: e276008fb5dd7a37554e202ba5c37948954301f1
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 06/05/2019
ms.locfileid: "66693705"
---
# <a name="tutorial-create-and-deploy-a-basic-bot"></a>チュートリアル:基本的なボットを作成してデプロイする

[!INCLUDE [applies-to-v4](../includes/applies-to.md)]

このチュートリアルでは、Bot Framework SDK を使用して基本的なボットを作成し、Azure にデプロイする手順について説明します。 基本的なボットを既に作成してローカル環境で実行している場合、「[ボットをデプロイする](#deploy-your-bot)」セクションに進んでください。

このチュートリアルでは、以下の内容を学習します。

> [!div class="checklist"]
> * 基本的なエコー ボットを作成する
> * それをローカル環境で実行して操作する
> * ボットを公開する

Azure サブスクリプションがない場合は、開始する前に[無料アカウント](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)を作成してください。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

[!INCLUDE [dotnet quickstart](~/includes/quickstart-dotnet.md)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

[!INCLUDE [javascript quickstart](~/includes/quickstart-javascript.md)]

---

## <a name="deploy-your-bot"></a>ボットをデプロイする

### <a name="prerequisites"></a>前提条件
- Azure サブスクリプションをお持ちでない場合は、開始する前に [無料アカウント](https://azure.microsoft.com/free/) を作成してください。
- ローカル コンピューターで実行されている上記のボット。
- 最新バージョンの [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/?view=azure-cli-latest)。

### <a name="1-prepare-for-deployment"></a>1.デプロイの準備をする
Visual Studio または Yeoman テンプレートを使用してボットを作成する場合、生成されたソース コードには `deploymentTemplates` フォルダーと ARM テンプレートが含まれています。 ここで説明するデプロイ プロセスでは、ARM テンプレートを使用して、Azure CLI を使ってボットに必要なリソースを Azure でプロビジョニングします。 

#### <a name="login-to-azure"></a>Azure にログインする

ボットの作成とローカルでのテストが完了したので、次は Azure にデプロイします。 コマンド プロンプトを開いて Azure portal にログインします。

```cmd
az login
```
ブラウザー ウィンドウが開いて、サインインできます。

#### <a name="set-the-subscription"></a>サブスクリプションを設定する
使用する既定のサブスクリプションを設定します。

```cmd
az account set --subscription "<azure-subscription>"
```

ボットのデプロイに使用するサブスクリプションが不明な場合は、`az account list` コマンドを使用して、お使いのアカウントのサブスクリプションの一覧を表示できます。 ボットのフォルダーに移動します。

#### <a name="create-an-app-registration"></a>アプリ登録を作成する
アプリケーションを登録することで、Azure AD を使用してユーザーを認証し、ユーザー リソースへのアクセスを要求できます。 お使いのボットには Azure に登録されたアプリが必要です。このアプリが Bot Framework Service へのボット アクセスを提供し、認証済みメッセージを送受信できるようにします。 Azure CLI を使用してアプリを登録を作成するには、次のコマンドを実行します。

```cmd
az ad app create --display-name "displayName" --password "AtLeastSixteenCharacters_0" --available-to-other-tenants
```

| オプション   | 説明 |
|:---------|:------------|
| display-name | アプリケーションの表示名。 |
| password | アプリのパスワード。"クライアント シークレット" とも呼ばれます。 パスワードは 16 文字以上でなければなりません。また、大文字または小文字を 1 文字以上、特殊文字を 1 文字以上含める必要があります。 |
| available-to-other-tenants| 任意の Azure AD テナントのアプリケーションを使用できます。  使用できる値: false、true。 既定では true です。 これにより、お使いのボットが Azure Bot Service チャネルと連携できます|

上記のコマンド キーにより JSON と キー `appId` が出力され、ARM デプロイ用にこのキーの値が保存されます。これは、ここで `appId` パラメーターに対して使用されます。 指定されたパスワードは `appSecret` パラメーターで使用されます。

ご自身のボットは、新しいリソース グループまたは既存のリソース グループにデプロイできます。 自分にとって最適なオプションを選択してください。

# <a name="deploy-via-arm-template-with-new-resource-grouptabnewrg"></a>[ARM テンプレートを使用したデプロイ (**新しい**リソース グループを使用)](#tab/newrg)

#### <a name="create-azure-resources"></a>Azure リソースを作成する

Azure で新しいリソース グループを作成し、ARM テンプレートを使用して、そこで指定されたリソースを作成します。 この場合は、App Service プラン、Web アプリ、および Bot Channels Registration が提供されます。

```cmd
az deployment create --name "<name-of-deployment>" --template-file "template-with-new-rg.json" --location "location-name" --parameters appId="<msa-app-guid>" appSecret="<msa-app-password>" botId="<id-or-name-of-bot>" botSku=F0 newAppServicePlanName="<name-of-app-service-plan>" newWebAppName="<name-of-web-app>" groupName="<new-group-name>" groupLocation="<location>" newAppServicePlanLocation="<location>"
```

| オプション   | 説明 |
|:---------|:------------|
| name | デプロイのフレンドリ名。 |
| template-file | ARM テンプレートへのパス。 プロジェクトの `deploymentTemplates` フォルダーで指定された `template-with-new-rg.json` ファイルを使用できます。 |
| location |場所。 値のソース: `az account list-locations` `az configure --defaults location=<location>` を使用して、既定の場所を構成できます。 |
| parameters | デプロイ パラメーターの値を提供します。 `az ad app create` コマンドを実行して取得した `appId` 値。 `appSecret` は、前の手順で指定したパスワードです。 `botId` パラメーターはグローバルに一意である必要があり、不変のボット ID として使用されます。 これは、変更可能なボットの表示名を構成するときにも使用されます。 `botSku` は価格レベルです。F0 (無料) または S1 (Standard) を指定できます。 `newAppServicePlanName` は App Service プランの名前です。 `newWebAppName` は、作成する Web アプリの名前です。 `groupName` は、作成する Azure リソース グループの名前です。 `groupLocation` は、Azure リソース グループの場所です。 `newAppServicePlanLocation` は、App Service プランの場所です。 |

# <a name="deploy-via-arm-template-with-existing--resource-grouptaberg"></a>[ARM テンプレートを使用したデプロイ (**既存の**リソース グループを使用)](#tab/erg)

#### <a name="create-azure-resources"></a>Azure リソースを作成する

既存のリソース グループを使用する場合は、既存の App Service プランを使用するか、新しい App Service プランを作成できます。 両方のオプションの手順を以下に示します。 

**オプション 1:既存の App Service プラン** 

この場合、既存の App Service プランが使用されますが、Web アプリと Bot Channels Registration は新しく作成されます。 

_注:botId パラメーターはグローバルに一意である必要があり、不変のボット ID として使用されます。変更可能なボットの displayName を構成するときにも使用されます。_

```cmd
az group deployment create --name "<name-of-deployment>" --resource-group "<name-of-resource-group>" --template-file "template-with-preexisting-rg.json" --parameters appId="<msa-app-guid>" appSecret="<msa-app-password>" botId="<id-or-name-of-bot>" newWebAppName="<name-of-web-app>" existingAppServicePlan="<name-of-app-service-plan>" appServicePlanLocation="<location>"
```

**オプション 2:新しい App Service プラン** 

この場合は、App Service プラン、Web アプリ、および Bot Channels Registration が作成されます。 

```cmd
az group deployment create --name "<name-of-deployment>" --resource-group "<name-of-resource-group>" --template-file "template-with-preexisting-rg.json" --parameters appId="<msa-app-guid>" appSecret="<msa-app-password>" botId="<id-or-name-of-bot>" newWebAppName="<name-of-web-app>" newAppServicePlanName="<name-of-app-service-plan>" appServicePlanLocation="<location>"
```

| オプション   | 説明 |
|:---------|:------------|
| name | デプロイのフレンドリ名。 |
| resource-group | Azure リソース グループの名前 |
| template-file | ARM テンプレートへのパス。 プロジェクトの `deploymentTemplates` フォルダーで指定された `template-with-preexisting-rg.json` ファイルを使用できます。 |
| location |場所。 値のソース: `az account list-locations` `az configure --defaults location=<location>` を使用して、既定の場所を構成できます。 |
| parameters | デプロイ パラメーターの値を提供します。 `az ad app create` コマンドを実行して取得した `appId` 値。 `appSecret` は、前の手順で指定したパスワードです。 `botId` パラメーターはグローバルに一意である必要があり、不変のボット ID として使用されます。 これは、変更可能なボットの表示名を構成するときにも使用されます。 `newWebAppName` は、作成する Web アプリの名前です。 `newAppServicePlanName` は App Service プランの名前です。 `newAppServicePlanLocation` は、App Service プランの場所です。 |

---

#### <a name="retrieve-or-create-necessary-iiskudu-files"></a>必要な IIS/Kudu ファイルを作成または取得する

**C# ボット**

```cmd
az bot prepare-deploy --lang Csharp --code-dir "." --proj-file-path "MyBot.csproj"
```

--code-dir に関連する .csproj ファイルへのパスを指定する必要があります。 これは、--proj-file-path 引数を使用して実行できます。 コマンドによって --code-dir および --proj-file-path が "./MyBot.csproj" に解決されます

**JavaScript ボット**

```cmd
az bot prepare-deploy --code-dir "." --lang Javascript
```

このコマンドにより、Azure App Service で IIS と連携するために Node.js アプリに必要な web.config が取り込まれます。 web.config がお使いのボットのルートに保存されていることを確認してください。

#### <a name="zip-up-the-code-directory-manually"></a>コード ディレクトリを手動で zip 圧縮する

構成されていない [zip デプロイ API](https://github.com/projectkudu/kudu/wiki/Deploying-from-a-zip-file-or-url) を使用してお使いのボットのコードをデプロイする場合、Web App/Kudu は次のように動作します。

"_Kudu では、zip ファイルからデプロイを実行する準備ができていること、およびデプロイ中に npm install、dotnet restore/dotnet publish などの追加ビルド ステップが不要であることを既定で想定しています。_ "

そのため、ご自身のビルドされたコードと、Web アプリに展開されている必要な依存関係すべてを zip ファイルに含めることが重要です。これを行わないと、ボットは意図したとおりに動作しません。

> [!IMPORTANT]
> プロジェクト ファイルを zip 圧縮する前に、必ず適切なフォルダー "_内_" に移動します。 
> - C# ボットの場合は、.csproj ファイルが含まれるフォルダーです。 
> - JS ボットの場合は、app.js ファイルまたは index.js ファイルが含まれるフォルダーです。 
>
> **そのフォルダー内**ですべてのファイルを選択して zip 圧縮し、そのフォルダー内でコマンドを実行します。
>
> お使いのルート フォルダーの場所が正しくない場合、**ボットは、Azure portal で実行できません**。

### <a name="2-deploy-code-to-azure"></a>2.コードを Azure にデプロイする
この時点で、Azure Web アプリにコードをデプロイする準備ができています。 コマンドラインから次のコマンドを実行して、Web アプリ用の Kudu の zip プッシュ デプロイを使用してデプロイを実行します。

```cmd
az webapp deployment source config-zip --resource-group "<new-group-name>" --name "<name-of-web-app>" --src "code.zip" 
```

| オプション   | 説明 |
|:---------|:------------|
| resource-group | Azure で前に作成したリソース グループの名前。 |
| name | 前に使用した Web アプリの名前。 |
| src  | 作成した zip ファイルへのパス。 |

### <a name="3-test-in-web-chat"></a>手順 3.Web チャットでのテスト
- Azure portal で、Web アプリ ボット ブレードに移動します。
- **[Bot Management]\(ボットの管理\)** セクションで、 **[Test in Web Chat]\(Web チャットでのテスト\)** をクリックします。 Azure Bot Service で Web チャット コントロールを読み込み、ボットに接続します。
- デプロイが成功したら、数秒待ちます。必要に応じて、Web アプリを再起動してキャッシュをクリアします。 Web App Bot ブレードに戻り、Azure portal で提供されている Web チャットを使用してテストします。

## <a name="additional-resources"></a>その他のリソース

[!INCLUDE [additional resources snippet](~/includes/deploy/snippet-additional-resources.md)]

## <a name="next-steps"></a>次の手順
> [!div class="nextstepaction"]
> [ボットで QnA Maker を使用して質問に回答する](bot-builder-tutorial-add-qna.md)
