---
title: Azure CLI を使用してボットをデプロイする | Microsoft Docs
description: 使用するボットを Azure クラウドにデプロイします。
keywords: ボットのデプロイ, azure のデプロイ, ボットの発行, ボットの az デプロイ, ボットの visual studio デプロイ, msbot 発行, msbot 複製
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: get-started-article
ms.service: bot-service
ms.subservice: abs
ms.date: 12/11/2018
ms.openlocfilehash: a7cb9cb1e3df14f2f46bc5a4c3a633f5212b5dfd
ms.sourcegitcommit: 0b421ff71617f03faf55ea175fb91d1f9e348523
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/12/2018
ms.locfileid: "53286618"
---
# <a name="deploy-your-bot-using-azure-cli"></a>Azure CLI を使用してボットをデプロイする

[!INCLUDE [pre-release-label](./includes/pre-release-label.md)]

ご自分のボットを作成し、ローカルでテストした後に、それを Azure にデプロイして、どこからでもアクセスできるようにすることができます。 ボットを Azure にデプロイするには、使用するサービスの料金を支払う必要があります。 [課金とコスト管理](https://docs.microsoft.com/en-us/azure/billing/)に関する記事で、Azure の課金の確認、使用量とコストの監視、アカウントとサブスクリプションの管理の方法について説明されています。

この記事では、`msbot` ツールを使用して Azure に C# ボットと JavaScript ボットをデプロイする方法を紹介します。 ボットのデプロイに必要なものを完全に理解するために、手順を実行する前にこの記事を確認することをお勧めします。


## <a name="prerequisites"></a>前提条件
- Azure サブスクリプションをお持ちでない場合は、開始する前に [無料アカウント](https://azure.microsoft.com/free/) を作成してください。
- [.NET Core SDK](https://dotnet.microsoft.com/download) v2.2 以降をインストールします。 
- 最新バージョンの [Azure CLI ツール](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest)をインストールします。
- `az` ツールの最新の `botservice` 拡張機能をインストールします。 
  - 最初に、`az extension remove -n botservice` コマンドを使用して古いバージョンを削除します。 次に、`az extension add -n botservice` コマンドを使用して最新バージョンをインストールします。
- 最新バージョンの [MSBot](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/MSBot) ツールをインストールします。
  - 複製操作に LUIS または Dispatch のリソースが含まれる場合、[LUIS CLI](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/LUIS#installation) が必要になります。
  - 複製操作に QnA Maker のリソースが含まれる場合、[QnA Maker CLI](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/QnAMaker#as-a-cli) が必要になります。
- [Bot Framework Emulator](https://aka.ms/Emulator-wiki-getting-started) をインストールします。
- [ngrok](https://github.com/Microsoft/BotFramework-Emulator/wiki/Tunneling-%28ngrok%29) をインストールして構成します。
- [.bot](v4sdk/bot-file-basics.md) ファイルの知識。

## <a name="deploy-javascript-and-c-bots-using-az-cli"></a>az cli を使用して JavaScript ボットと C# ボットをデプロイする
ボットは既に作成してありますから、ここで必要なのは Azure にデプロイすることです。 以下の手順では、`msbot connect` コマンドまたは Bot Framework Emulator のいずれかを使用して、必要な Azure のリソースの作成と .bot ファイル内のサービス参照の更新が完了していることを前提とします。 .bot ファイルが最新でない場合は、デプロイ プロセスがエラーまたは警告なしで完了しているにもかかわらず、デプロイしたボットが動作しなくなる場合があります。

コマンド プロンプトを開いて Azure portal にログインします。

```cmd
az login
```
ブラウザー ウィンドウが開いて、サインインできます。 

### <a name="set-the-subscription"></a>サブスクリプションを設定する 
次のコマンドを使用してサブスクリプションを設定します。

```cmd
az account set --subscription "<azure-subscription>"
``` 

ボットのデプロイに使用するサブスクリプションが不明な場合は、`az account list` コマンドを使用して、お使いのアカウントの `subscriptions` の一覧を表示できます。

ボットのフォルダーに移動します。 
```cmd 
cd <local-bot-folder>
```

### <a name="azure-subscription-account"></a>Azure サブスクリプション アカウント
続行する前に、Azure へのログインに使用する電子メール アカウントの種類に基づいて、該当する手順をお読みください。

**MSA 電子メール アカウント**

[MSA](https://en.wikipedia.org/wiki/Microsoft_account) 電子メール アカウントを使用する場合は、`msbot clone services` コマンドで使用する appId と appSecret を作成する必要があります。 

- [アプリケーション登録ポータル](https://apps.dev.microsoft.com/)に移動します。 **[アプリの追加]** をクリックしてアプリケーションを登録して、**アプリケーション ID** を作成し、**新しいパスワードを生成**します。 
- `msbot clone services` コマンドで使用できるように、生成されたアプリケーション ID と新しいパスワードの両方を控えておきます。 
- デプロイするには、ご自身のボットに該当するコマンドを使用します。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

`msbot clone services --folder deploymentScripts/msbotClone --location "<geographic-location>" --proj-file "<your.csproj>" --name "<bot-name>" --appid "xxxxxxxx" --password "xxxxxxx" --verbose`

[!INCLUDE [deployment note](./includes/deployment-note-cli.md)]

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

`msbot clone services --folder deploymentScripts/msbotClone --location "<geographic-location>"   --code-dir . --name "<bot-name>" --appid "xxxxxxxx" --password "xxxxxxx" --verbose`


[!INCLUDE [deployment note](./includes/deployment-note-cli.md)]

---

**職場または学校のアカウント**

職場や学校で提供される電子メール アカウントを使用して Azure にログインする場合、アプリケーション ID とパスワードを作成する必要はありません。 デプロイするには、ご自身のボットに該当するコマンドを使用します。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

`msbot clone services --folder deploymentScripts/msbotClone --location "<geographic-location>" --verbose --proj-file "<your-project-file>" --name "<bot-name>"`

[!INCLUDE [deployment note](./includes/deployment-note-cli.md)]

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

`msbot clone services --folder deploymentScripts/msbotClone --location "<geographic-location>" --verbose --code-dir . --name "<bot-name>"`


[!INCLUDE [deployment note](./includes/deployment-note-cli.md)]

---

#### <a name="save-the-secret-used-to-decrypt-bot-file"></a>.bot ファイルの復号化に使用するシークレットを保存する
デプロイ プロセスで "_新しい .bot ファイルが作成され、シークレットを使用して暗号化される_" ことに注意する必要があります。 ボットがデプロイされる間に、.bot ファイルのシークレットの保存を確認する次のメッセージがコマンド ラインに表示されます。 

`The secret used to decrypt myAzBot.bot is:`
`hT6U1SIPQeXlebNgmhHYxcdseXWBZlmIc815PpK0WWA=`

`NOTE: This secret is not recoverable and you should save it in a safe place according to best security practices.
      Copy this secret and use it to open the <file.bot> the first time.`
      
後で使用するために、.bot ファイルのシークレットを保存します。 暗号化された新しい .bot ファイルは、botFileSecret と共に Azure portal 上で使用します。 ボット ファイル名またはシークレットを後で変更する必要がある場合は、ポータルの **[App Service の設定] -> [アプリケーションの設定]** セクションに移動します。 appsettings.json または .env ファイル内のボット ファイル名は、作成された最新のボット ファイルで更新されることに注意してください。  

### <a name="test-your-bot"></a>ボットをテストする
エミュレーターで、実稼働エンドポイントを使用してアプリケーションをテストします。 ローカルでテストする場合は、ボットがローカル コンピューターで実行されていることを確認します。 

### <a name="to-update-your-bot-code-in-azure"></a>Azure でボット コードを更新するには
Azure でボット コードを更新するときに、`msbot clone services` コマンドを使用しないでください。 次に示すように、`az bot publish` コマンドを使用する必要があります。

```cmd
az bot publish --name "<your-azure-bot-name>" --proj-file "<your-proj-file>" --resource-group "<azure-resource-group>" --code-dir "<folder>" --verbose --version v4
```

| 引数        | 説明 |
|----------------  |-------------|
| `name`      | ボットが最初に Azure にデプロイされたときに使用した名前。|
| `proj-file` | C# ボットの場合は、.csproj ファイルです。 JS/TS bot の場合は、ローカルのボットのスタートアップ プロジェクトのファイル名 (index.js や index.ts など) です。|
| `resource-group` | `msbot clone services` コマンドで使用した Azure リソース グループ。|
| `code-dir`  | ローカルのボットのフォルダーへのポインター。|



## <a name="additional-resources"></a>その他のリソース

ボットをデプロイするときに、通常これらのリソースが Azure portal 上で作成されます。

| リソース      | 説明 |
|----------------|-------------|
| Web アプリ ボット | Azure App Service にデプロイされる Azure Bot Service ボット。|
| [App Service](https://docs.microsoft.com/en-us/azure/app-service/)| Web アプリケーションをビルドおよびホストできます。|
| [[App Service プラン]](https://docs.microsoft.com/en-us/azure/app-service/azure-web-sites-web-hosting-plans-in-depth-overview)| Web アプリを実行するための一連のコンピューティング リソースを定義します。|
| [Application Insights](https://docs.microsoft.com/en-us/azure/application-insights/app-insights-overview)| テレメトリを収集および分析するためのツールを提供します。|
| [ストレージ アカウント](https://docs.microsoft.com/en-us/azure/storage/common/storage-introduction)| 高い可用性、セキュリティ、耐久性、スケーラビリティ、および冗長性を備えたクラウド ストレージを提供します。|

`az bot` コマンドに関するドキュメントを確認するには、[リファレンス](https://docs.microsoft.com/en-us/cli/azure/bot?view=azure-cli-latest)に関するトピックを参照してください。

Azure リソース グループにあまり馴染みがない場合は、こちらの「[用語集](https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-overview#terminology)」のトピックを参照してください。

## <a name="next-steps"></a>次の手順
> [!div class="nextstepaction"]
> [継続的デプロイを設定する](bot-service-build-continuous-deployment.md)
