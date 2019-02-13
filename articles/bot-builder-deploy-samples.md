---
title: botbuilder-samples リポジトリからボットをデプロイする | Microsoft Docs
description: 使用するボットを Azure クラウドにデプロイします。
keywords: ボットのデプロイ, azure のデプロイ, ボットの発行, ボットの az デプロイ, ボットの visual studio デプロイ, msbot 発行, msbot 複製
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: get-started-article
ms.service: bot-service
ms.subservice: abs
ms.date: 01/15/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 3ca8ac4bfe14ed20f11a0ab26d8102ac21e60e2b
ms.sourcegitcommit: 32615b88e4758004c8c99e9d564658a700c7d61f
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/04/2019
ms.locfileid: "55711956"
---
# <a name="deploy-bots-from-botbuilder-samples-repo"></a>botbuilder-samples リポジトリからボットをデプロイする

[!INCLUDE [pre-release-label](./includes/pre-release-label.md)]

この記事では、[botbuilder-samples](https://github.com/Microsoft/BotBuilder-Samples) リポジトリにある C# および JavaScript のサンプルを Azure にデプロイする方法を紹介します。

サンプル ボットをデプロイする手順は、[Azure に既にプロビジョニングされているすべてのリソースを使用して作成したボットをデプロイする](https://docs.microsoft.com/en-us/azure/bot-service/bot-builder-deploy-az-cli?view=azure-bot-service-4.0&tabs=csharp)手順とは "_異なります_"。

> [!IMPORTANT]
> [botbuilder-samples](https://github.com/Microsoft/BotBuilder-Samples) レポジトリから Azure にボットをデプロイすると、Azure サービスがプロビジョニングされ、使用しているサービスに関する料金がかかります。
> [課金とコスト管理](https://docs.microsoft.com/en-us/azure/billing/)に関する記事で、Azure の課金の確認、使用量とコストの監視、アカウントとサブスクリプションの管理の方法について説明されています。

ボットのデプロイに必要なものを完全に理解するために、手順を実行する前にこの記事を通読することをお勧めします。

## <a name="prerequisites"></a>前提条件

- Azure サブスクリプションをお持ちでない場合は、開始する前に [無料アカウント](https://azure.microsoft.com/free/) を作成してください。
- [.NET Core SDK](https://dotnet.microsoft.com/download) v2.2 以降をインストールします。 `dotnet --version` を使って、使用中のバージョンを確認します。
- 最新バージョンの [Azure CLI ツール](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest)をインストールします。 `az --version` を使って、使用中のバージョンを確認します。
- 最新バージョンの [MSBot](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/MSBot) ツールをインストールします。
  - 複製操作に LUIS または Dispatch のリソースが含まれる場合、[LUIS CLI](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/LUIS#installation) が必要になります。
  - 複製操作に QnA Maker のリソースが含まれる場合、[QnA Maker CLI](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/QnAMaker#as-a-cli) が必要になります。
- [Bot Framework Emulator](https://aka.ms/Emulator-wiki-getting-started) をインストールします。
- [ngrok](https://github.com/Microsoft/BotFramework-Emulator/wiki/Tunneling-%28ngrok%29) をインストールして構成します。
- [.bot](v4sdk/bot-file-basics.md) ファイルの知識。

msbot 4.3.2 以降では、Azure CLI バージョン 2.0.54 以降が必要です。 botservice 拡張機能をインストールした場合は、次のコマンドでそれを削除します。

```cmd
az extension remove --name botservice
```

### <a name="c"></a>C\#

 `msbot clone services` では Azure にコード ファイルがアップロードされず、DLL と他のいくつかのファイルのみがアップロードされます。 このコマンドを実行する前に、サンプルをコンパイルする必要があります。

### <a name="service-names"></a>サービス名

`msbot clone services` コマンドによって、ボットのデプロイに加え、すべての関連するサービスが、選択したサブスクリプションにプロビジョニングされます。

指定した名前とサービスの組み合わせがサブスクリプションに既に存在する場合、このコマンドからはエラーが生成されます。再開する前に、一部のデプロイを削除する必要があります。 これには、LUIS アプリケーション、QnA Maker ナレッジ ベース、Dispatch モデルが含まれています。

## <a name="deploy-javascript-and-c-bots-using-az-cli"></a>az cli を使用して JavaScript ボットと C# ボットをデプロイする

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

### <a name="clone-the-sample"></a>サンプルを複製する

**Azure サブスクリプション アカウント**:続行する前に、Azure へのログインに使用する電子メール アカウントの種類に基づいて、該当する手順をお読みください。

**サービスの作成**:`msbot clone services` コマンドにより、ボット用の Azure サービスが作成されます。

1. 作成されるサービスが一覧表示され、続行する前に操作の確認が求められます。 拒否すると、サービスは作成されずにコマンドが終了します。
1. 続行する前に、Azure の認証を受ける手順があります。

**LUIS サービス**:ボットで LUIS または Dispatch が使用されている場合は、`msbot clone services` コマンドに LUIS 作成キーを含める必要があります。

#### <a name="msa-email-account"></a>MSA 電子メール アカウント

[MSA](https://en.wikipedia.org/wiki/Microsoft_account) 電子メール アカウントを使用する場合は、`msbot clone services` コマンドで使用する appId と appSecret を作成する必要があります。

- [アプリケーション登録ポータル](https://apps.dev.microsoft.com/)に移動します。 **[アプリの追加]** をクリックしてアプリケーションを登録して、**アプリケーション ID** を作成し、**新しいパスワードを生成**します。
> 注 - 生成されたパスワードに文字 "|" が含まれている場合、このパスワードは Azure によって拒否されます。 これを解決するには、別の新しいパスワードを生成します。
- `msbot clone services` コマンドで使用できるように、生成されたアプリケーション ID と新しいパスワードの両方を控えておきます。
- デプロイするには、ご自身のボットに該当するコマンドを使用します。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

`msbot clone services --folder deploymentScripts/msbotClone --location "<geographic-location>" --proj-file "<your.csproj>" --name "<bot-name>" --appId "xxxxxxxx" --appSecret "xxxxxxx" --verbose --luisAuthoringKey <luis-authoring-key>`

[!INCLUDE [deployment note](./includes/deployment-note-cli.md)]

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

`msbot clone services --folder deploymentScripts/msbotClone --location "<geographic-location>"   --code-dir . --name "<bot-name>" --appId "xxxxxxxx" --appSecret "xxxxxxx" --verbose --luisAuthoringKey <luis-authoring-key>`

[!INCLUDE [deployment note](./includes/deployment-note-cli.md)]

---

#### <a name="business-or-school-account"></a>職場または学校のアカウント

職場や学校で提供される電子メール アカウントを使用して Azure にログインする場合、アプリケーション ID とパスワードを作成する必要はありません。 デプロイするには、ご自身のボットに該当するコマンドを使用します。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

`msbot clone services --folder deploymentScripts/msbotClone --location "<geographic-location>" --verbose --proj-file "<your-project-file>" --name "<bot-name>" --luisAuthoringKey <luis-authoring-key>`

[!INCLUDE [deployment note](./includes/deployment-note-cli.md)]

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

`msbot clone services --folder deploymentScripts/msbotClone --location "<geographic-location>" --verbose --code-dir . --name "<bot-name>" --luisAuthoringKey <luis-authoring-key>`

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
az bot publish --name "<your-azure-bot-name>" --proj-name "<your-proj-name>" --resource-group "<azure-resource-group>" --code-dir "<folder>" --verbose --version v4
```

| 引数        | 説明 |
|----------------  |-------------|
| `name`      | ボットが最初に Azure にデプロイされたときに使用した名前。|
| `proj-name` | C# の場合は、公開する必要があるスタートアップ プロジェクト ファイル名 (.csproj は含めません) を使用します。 (例: `EnterpriseBot`)。 Node.js の場合は、ボットのメイン エントリ ポイントを使用します。 たとえば、「 `index.js` 」のように入力します。 |
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
