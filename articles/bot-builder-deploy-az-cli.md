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
ms.date: 12/14/2018
ms.openlocfilehash: 19960940a40fa291534bc1f88290bc6a7da109e0
ms.sourcegitcommit: 8c10aa7372754596a3aa7303a3a893dd4939f7e9
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/20/2018
ms.locfileid: "53654337"
---
# <a name="deploy-your-bot-using-azure-cli"></a>Azure CLI を使用してボットをデプロイする

[!INCLUDE [pre-release-label](./includes/pre-release-label.md)]

ご自分のボットを作成し、ローカルでテストした後に、それを Azure にデプロイして、どこからでもアクセスできるようにすることができます。 ボットを Azure にデプロイするには、使用するサービスの料金を支払う必要があります。 [課金とコスト管理](https://docs.microsoft.com/en-us/azure/billing/)に関する記事で、Azure の課金の確認、使用量とコストの監視、アカウントとサブスクリプションの管理の方法について説明されています。

この記事では、`az` および `msbot` CLI を使用して Azure に C# ボットと JavaScript ボットをデプロイする方法を紹介します。 ボットのデプロイに必要なものを完全に理解するために、手順を実行する前にこの記事を確認することをお勧めします。


## <a name="prerequisites"></a>前提条件
- Azure サブスクリプションをお持ちでない場合は、開始する前に [無料アカウント](https://azure.microsoft.com/free/) を作成してください。
- 最新バージョンの [Azure CLI ツール](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest)をインストールします。
- `az` ツールの最新の `botservice` 拡張機能をインストールします。
  - 最初に、`az extension remove -n botservice` コマンドを使用して古いバージョンを削除します。 次に、`az extension add -n botservice` コマンドを使用して最新バージョンをインストールします。
- 最新バージョンの [MSBot](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/MSBot) ツールをインストールします。
- 最新リリース バージョンの [Bot Framework Emulator](https://aka.ms/Emulator-wiki-getting-started) をインストールします。
- [ngrok](https://github.com/Microsoft/BotFramework-Emulator/wiki/Tunneling-%28ngrok%29) をインストールして構成します。
- [.bot](v4sdk/bot-file-basics.md) ファイルの知識。

## <a name="deploy-javascript-and-c-bots-using-az-cli"></a>az cli を使用して JavaScript ボットと C# ボットをデプロイする

ボットの作成とローカルでのテストが完了したので、次は Azure にデプロイします。 これらの手順は、必要な Azure リソースを作成済みであることを前提としています。

コマンド プロンプトを開いて Azure portal にログインします。

```cmd
az login
```

ブラウザー ウィンドウが開いて、サインインできます。

### <a name="set-the-subscription"></a>サブスクリプションを設定する

使用する既定のサブスクリプションを設定します。

```cmd
az account set --subscription "<azure-subscription>"
```

ボットのデプロイに使用するサブスクリプションが不明な場合は、`az account list` コマンドを使用して、お使いのアカウントの `subscriptions` の一覧を表示できます。

ボットのフォルダーに移動します。

```cmd
cd <local-bot-folder>
```

### <a name="create-a-web-app-bot"></a>Web アプリ ボットを作成する

ボットを公開するボット リソースを作成します。

続行する前に、Azure へのログインに使用する電子メール アカウントの種類に基づいて、該当する手順をお読みください。

#### <a name="msa-email-account"></a>MSA 電子メール アカウント

[MSA](https://en.wikipedia.org/wiki/Microsoft_account) 電子メール アカウントを使用している場合は、`az bot create` コマンドを使って、使用するアプリケーション登録ポータルにアプリ ID とアプリ パスワードを作成する必要があります。

1. [**アプリケーション登録ポータル**](https://apps.dev.microsoft.com/)に移動します。
1. **[アプリの追加]** をクリックしてアプリケーションを登録して、**アプリケーション ID** を作成し、**新しいパスワードを生成**します。 既にアプリケーションとパスワードを持っていて、パスワードを思い出せない場合は、[アプリケーション シークレット] セクションで新しいパスワードを生成する必要があります。
1. `az bot create` コマンドで使用できるように、生成したアプリケーション ID と新しいパスワードの両方を控えておきます。  

```cmd
az bot create --kind webapp --name <bot-resource-name> --location <geographic-location> --version v4 --lang <language> --verbose --resource-group <resource-group-name> --appid "<application-id>" --password "<application-password>" --verbose
```

| オプション | 説明 |
|:---|:---|
| --name | Azure でのボットのデプロイに使用される一意の名前。 ローカルのボットと同じ名前にすることができます。 名前にスペースやアンダースコアを含めないでください。 |
| --location | ボット サービス リソースを作成するために使用される地理的な場所。 たとえば、`eastus`、`westus`、`westus2` などです。 |
| --lang | ボットの作成に使用する言語: `Csharp` または `Node`。既定値は `Csharp` です。 |
| --resource-group | ボットを作成するリソース グループの名前。 `az configure --defaults group=<name>` を使用して、既定のグループを構成できます。 |
| --appid | ボットで使用される Microsoft アカウント ID (MSA ID)。 |
| --password | ボットの Microsoft アカウント (MSA) パスワード。 |

#### <a name="business-or-school-account"></a>職場または学校のアカウント

```cmd
az bot create --kind webapp --name <bot-resource-name> --location <geographic-location> --version v4 --lang <language> --verbose --resource-group <resource-group-name>
```

| オプション | 説明 |
|:---|:---|
| --name | Azure でのボットのデプロイに使用される一意の名前。 ローカルのボットと同じ名前にすることができます。 名前にスペースやアンダースコアを含めないでください。 |
| --location | ボット サービス リソースを作成するために使用される地理的な場所。 たとえば、`eastus`、`westus`、`westus2` などです。 |
| --lang | ボットの作成に使用する言語: `Csharp` または `Node`。既定値は `Csharp` です。 |
| --resource-group | ボットを作成するリソース グループの名前。 `az configure --defaults group=<name>` を使用して、既定のグループを構成できます。 |

### <a name="download-the-bot-from-azure"></a>Azure からボットをダウンロードする

次に、作成したボットをダウンロードします。 このコマンドで、save-path 以下にサブディレクトリが作成されます。ただし、指定するパスは既に存在している必要があります。

```cmd
az bot download --name <bot-resource-name> --resource-group <resource-group-name> --save-path "<path>"
```

| オプション | 説明 |
|:---|:---|
| --name | Azure 内のボットの名前。 |
| --resource-group | ボットが属するリソース グループの名前。 |
| --save-path | ボット コードをダウンロードする先の既存のディレクトリ。 |

### <a name="decrypt-the-downloaded-bot-file"></a>ダウンロードした .bot ファイルを復号化する

.bot ファイル内の機密情報は暗号化されています。

暗号化キーを取得します。

1. [Azure Portal](http://portal.azure.com/) にログインします。
1. ボットの Web App Bot リソースを開きます。
1. ボットの **[アプリケーション設定]** を開きます。
1. **[アプリケーション設定]** ウィンドウで、**[アプリケーション設定]** までスクロールします。
1. **botFileSecret** を探してその値をコピーします。

.bot ファイルを復号化します。

```cmd
msbot secret --bot <name-of-bot-file> --secret "<bot-file-secret>" --clear
```

| オプション | 説明 |
|:---|:---|
| --bot | ダウンロードした .bot ファイルの相対パス。 |
| --secret | 暗号化キー。 |

### <a name="use-the-downloaded-bot-file-in-your-project"></a>ダウンロードした .bot ファイルをプロジェクトで使用する

復号化した .bot ファイルを、ローカルのボット プロジェクトが保存されているディレクトリにコピーします。

この新しい .bot ファイルを使用するようにボットを更新します。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

**appsettings.json** で、新しい .bot ファイルを指すように **botFilePath** プロパティを更新します。

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**.env** で、新しい .bot ファイルを指すように **botFilePath** プロパティを更新します。

---

### <a name="update-the-bot-file"></a>.bot ファイルを更新する

ボットで LUIS、QnA Maker、または Dispatch サービスを使用している場合は、それらへの参照を .bot ファイルに追加する必要があります。 それ以外の場合は、この手順を省略できます。

1. 新しい .bot ファイルを使用して、BotFramework エミュレーターでボットを開きます。 ボットがローカルで実行されている必要はありません。
1. **[BOT EXPLORER]\(ボット エクスプローラー\)** パネルで **[SERVICES]\(サービス\)** セクションを展開します。
1. LUIS アプリへの参照を追加するには、**[SERVICES]\(サービス\)** の右側にあるプラス記号 (+) をクリックします。
   1. **[Add Language Understanding (LUIS)]\(Language Understanding (LUIS) の追加\)** を選択します。
   1. Azure アカウントへのログインを求められたら、ログインします。
   1. アクセス権を持つ LUIS アプリケーションの一覧が表示されます。 ボット用にいずれかを選択します。
1. QnA Maker ナレッジ ベースへの参照を追加するには、**[SERVICES]\(サービス\)** の右側にあるプラス記号 (+) をクリックします。
   1. **[Add QnA Maker]\(QnA Maker の追加\)** を選択します。
   1. Azure アカウントへのログインを求められたら、ログインします。
   1. アクセス権を持つナレッジ ベースの一覧が表示されます。 ボット用にいずれかを選択します。
1. Dispatch モデルへの参照を追加するには、**[SERVICES]\(サービス\)** の右側にあるプラス記号 (+) をクリックします。
   1. **[Add Dispatch]\(Dispatch の追加\)** を選択します。
   1. Azure アカウントへのログインを求められたら、ログインします。
   1. アクセス権を持つ Dispatch モデルの一覧が表示されます。 ボット用にいずれかを選択します。

### <a name="test-your-bot-locally"></a>ボットをローカルでテストする

この時点で、このボットは以前の .bot ファイルと同じように動作するはずです。 新しい .bot ファイルを使用して、想定どおりに機能することを確認してください。

### <a name="publish-your-bot-to-azure"></a>ボットを Azure に公開する

<!-- TODO: re-encrypt your .bot file? -->

ローカルのボットを Azure に公開します。 この手順には時間がかかる場合があります。

```cmd
az bot publish --name <bot-resource-name> --proj-file "<project-file-name>" --resource-group <resource-group-name> --code-dir <directory-path> --verbose --version v4
```

<!-- Question: What should --proj-file be for a Node project? -->

| オプション | 説明 |
|:---|:---|
| --name | Azure 内のボットのリソース名。 |
| --proj-file | 公開する必要があるスタートアップ プロジェクト ファイル名 (.csproj は含めません)。 例: EnterpriseBot。 |
| --resource-group | リソース グループの名前。 |
| --code-dir | ボット コードをアップロードする元のディレクトリ。 |

これが完了すると、"Deployment successful!" (デプロイに成功しました!) メッセージが表示され、ボットは Azure にデプロイされます。

<!-- TODO: If we tell them to re-encrypt, this step is not necessary. -->

暗号化キーの設定をクリアします。

1. [Azure Portal](http://portal.azure.com/) にログインします。
1. ボットの Web App Bot リソースを開きます。
1. ボットの **[アプリケーション設定]** を開きます。
1. **[アプリケーション設定]** ウィンドウで、**[アプリケーション設定]** まで下にスクロールします。
1. **botFileSecret** を探してそれを削除します。

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
