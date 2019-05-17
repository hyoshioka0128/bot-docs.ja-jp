---
ms.openlocfilehash: 4b5181babf728861107a0c7bc28f844491761a7a
ms.sourcegitcommit: f84b56beecd41debe6baf056e98332f20b646bda
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 05/03/2019
ms.locfileid: "65033886"
---
デプロイを開始する前に、最新バージョンの [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest) および [dotnet CLI](https://dotnet.microsoft.com/download) を使用していることを確認します。 dotnet CLI がない場合は、上記のリンクから .Net Core ランタイム オプションを使用してインストールします。 

### <a name="login-to-azure-cli-and-set-your-subscription"></a>Azure CLI にログインしてサブスクリプションを設定する
ボットの作成とローカルでのテストが完了したので、次は Azure にデプロイします。 コマンド プロンプトを開いて Azure portal にログインします。

```cmd
az login
```
### <a name="set-the-subscription"></a>サブスクリプションを設定する

使用する既定のサブスクリプションを設定します。

```cmd
az account set --subscription "<azure-subscription>"
```

ボットのデプロイに使用するサブスクリプションが不明な場合は、`az account list` コマンドを使用して、お使いのアカウントのサブスクリプションの一覧を表示できます。

ボットのフォルダーに移動します。
`cd <local-bot-folder>`

### <a name="create-a-web-app-bot-in-azure"></a>Azure に Web App Bot を作成する 

ローカル ボットを公開するリソース グループがまだ存在しない場合は、作成します。

```cmd
az group create --name <resource-group-name> --location <geographic-location> --verbose
```

| オプション     | 説明 |
|:-----------|:---|
| name     | リソース グループの一意名。 名前にスペースやアンダースコアを含めないでください。 |
| location | リソース グループの作成に使用する地理的な場所。 たとえば、`eastus`、`westus`、`westus2` などです。 場所の一覧を取得するには `az account list-locations` を使用します。 |

その後、ボットを公開するボット リソースを作成します。 これにより、Azure に必要なリソースがプロビジョニングされて、ボット Web アプリが作成されます。これを、ローカル ボットで上書きします。 

続行する前に、Azure へのログインに使用する電子メール アカウントの種類に基づいて、該当する手順をお読みください。

#### <a name="msa-email-account"></a>MSA 電子メール アカウント
MSA 電子メール アカウントを使用している場合は、`az bot create` コマンドを使って、使用するアプリケーション登録ポータルにアプリ ID とアプリ パスワードを作成する必要があります。
1. [**アプリケーション登録ポータル**](https://portal.azure.com/#blade/Microsoft_AAD_RegisteredApps/ApplicationsListBlade)に移動します。
1. **[アプリの追加]** をクリックしてアプリケーションを登録して、**アプリケーション ID** を作成し、**新しいパスワードを生成**します。 既にアプリケーションとパスワードを持っていて、パスワードを思い出せない場合は、[アプリケーション シークレット] セクションで新しいパスワードを生成する必要があります。
1. `az bot create` コマンドで使用できるように、生成したアプリケーション ID と新しいパスワードの両方を控えておきます。  

```cmd
az bot create --kind webapp --name <bot-name-in-azure> --location <geographic-location> --version v4 --lang <language> --verbose --resource-group <resource-group-name> --appid "<application-id>" --password "<application-password>" --verbose
```

| オプション | 説明 |
|:---|:---|
| name | Azure でのボットのデプロイに使用される一意の名前。 ローカルのボットと同じ名前にすることができます。 名前にスペースやアンダースコアを含めないでください。 |
| location | ボット サービス リソースを作成するために使用される地理的な場所。 たとえば、`eastus`、`westus`、`westus2` などです。 |
| resource-group | ボットを作成するリソース グループの名前。 `az configure --defaults group=<name>` を使用して、既定のグループを構成できます。 |
| appid | ボットで使用される Microsoft アカウント ID (MSA ID)。 |
| password | ボットの Microsoft アカウント (MSA) パスワード。 |

#### <a name="business-or-school-account"></a>職場または学校のアカウント

```cmd
az bot create --kind webapp --name <bot-name-in-azure> --location <geographic-location> --version v4 --lang <language> --verbose --resource-group <resource-group-name>
```
| オプション | 説明 |
|:---|:---|
| name | Azure でのボットのデプロイに使用される一意の名前。 ローカルのボットと同じ名前にすることができます。 名前にスペースやアンダースコアを含めないでください。 |
| location | ボット サービス リソースを作成するために使用される地理的な場所。 たとえば、`eastus`、`westus`、`westus2` などです。 |
| lang | ボットの作成に使用する言語: `Csharp` または `Node`。既定値は `Csharp` です。 |
| resource-group | ボットを作成するリソース グループの名前。 `az configure --defaults group=<name>` を使用して、既定のグループを構成できます。 |

#### <a name="update-appsettingsjson-or-env-file"></a>appsettings.json または .env ファイルを更新する
ボットの作成後、次の情報がコンソール ウィンドウに表示されます。 

```JSON
{
  "appId": "as234-345b-4def-9047-a8a44b4s",
  "appPassword": "34$#w%^$%23@334343",
  "endpoint": "https://mybot.azurewebsites.net/api/messages",
  "id": "mybot",
  "name": "mybot",
  "resourceGroup": "botresourcegroup",
  "serviceName": "mybot",
  "subscriptionId": "234532-8720-5632-a3e2-a1qw234",
  "tenantId": "32f955bf-33f1-43af-3ab-23d009defs47",
  "type": "abs"
}
```

`appId` と `appPassword` の値をコピーし、appsettings.json ファイルまたは .env ファイルに貼り付ける必要があります。 例: 

```JSON
{
  MicrosoftAppId: "as234-345b-4def-9047-a8a44b4s",
  MicrosoftAppPassword: "34$#w%^$%23@334343"
}
```
その appsettings.json ファイルまたは .env ファイルに、ご自身のボット用にプロビジョニングした他のサービス用の追加キーが含まれている場合、これらのエントリは削除しないでください。

ファイルを保存します。

次に、ボットの作成に使用したプログラミング言語 (**C#** または **JS**) に応じて、ご自身に適した手順に従って操作します。

**C# ボット:** 

コマンド プロンプトを開いて、プロジェクト フォルダーに移動します。 コマンド ラインから次のコマンドを実行します。

| タスク | command |
|:-----|:--------|
| 1.プロジェクトの依存関係の復元 | `dotnet restore`|
| 2.プロジェクトのビルド     | `dotnet build` |
| 手順 3.プロジェクト ファイルの zip 圧縮 | 任意のユーティリティを使用して、プロジェクト ファイルを zip 圧縮します。 .csproj ファイルが含まれるフォルダーに移動し、そのレベルのすべてのファイルとフォルダーを選択して、zip 形式フォルダーを作成します。 |
| 4.ビルド デプロイの設定 | `az webapp config appsettings set --resource-group <resource-group-name> --name <bot-name> --settings SCM_DO_BUILD_DEPLOYMENT=false`|
| 5.スクリプト ジェネレーター引数の設定 | `az webapp config appsettings set --resource-group <resource-group-name> --name <bot-name> --settings SCM_SCRIPT_GENERATOR_ARGS="--aspNetCore mybot.csproj"`|

**JS ボット:**
1. web.config を[こちら](https://github.com/projectkudu/kudu/wiki/Using-a-custom-web.config-for-Node-apps)からダウンロードして、ご自身のプロジェクト フォルダーに保存します。 
1. ファイルを編集し、すべての "server.js" を "index.js" に置き換えます。 
1. ファイルを保存します。

コマンド プロンプトを開いて、プロジェクト フォルダーに移動します。 コマンド ラインから次のコマンドを実行します。

| タスク | command |
|:-----|:--------|
| 1.Node モジュールのインストール | `npm install` |
| 2.プロジェクト ファイルの zip 圧縮 | 任意のユーティリティを使用して、プロジェクト ファイルを zip 圧縮します。 .csproj ファイルが含まれるフォルダーに移動し、そのレベルのすべてのファイルとフォルダーを選択して、zip 形式フォルダーを作成します。 |
| 手順 3.ビルド デプロイの設定 | `az webapp config appsettings set --resource-group <resource-group-name> --name <bot-name> --settings SCM_DO_BUILD_DEPLOYMENT=false`|
