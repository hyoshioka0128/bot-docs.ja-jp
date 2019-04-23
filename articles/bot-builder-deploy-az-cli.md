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
ms.date: 04/12/2019
ms.openlocfilehash: 4532fe55705524573de55017e633289255a20ab9
ms.sourcegitcommit: 721bb09f10524b0cb3961d7131966f57501734b8
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/18/2019
ms.locfileid: "59508219"
---
# <a name="deploy-your-bot"></a>ボットをデプロイする

[!INCLUDE [pre-release-label](./includes/pre-release-label.md)]

ご自分のボットを作成し、ローカルでテストした後に、それを Azure にデプロイして、どこからでもアクセスできるようにすることができます。 ボットを Azure にデプロイするには、使用するサービスの料金を支払う必要があります。 [課金とコスト管理](https://docs.microsoft.com/en-us/azure/billing/)に関する記事で、Azure の課金の確認、使用量とコストの監視、アカウントとサブスクリプションの管理の方法について説明されています。

この記事では、C# ボットと JavaScript ボットを Azure にデプロイする方法を紹介します。 ボットのデプロイに必要なものを完全に理解するために、手順を実行する前にこの記事を確認することをお勧めします。

## <a name="prerequisites"></a>前提条件
- [Azure サブスクリプション](http://portal.azure.com)をお持ちでない場合は、開始する前に無料アカウントを作成してください。
- ローカル コンピューター上で開発した [**CSharp**](./dotnet/bot-builder-dotnet-sdk-quickstart.md) または [**JavaScript**](./javascript/bot-builder-javascript-quickstart.md) ボット。

## <a name="1-prepare-for-deployment"></a>1.デプロイの準備をする
デプロイ プロセスでは、ローカルのボットをデプロイできる、ターゲットとなる Web アプリ ボットが Azure に必要です。 ターゲットの Web アプリ ボットと、Azure で一緒にプロビジョニングされるリソースは、デプロイするローカルのボットによって使用されます。 お持ちのローカルのボットでは、必要な Azure リソースがすべてプロビジョニングされているわけではないため、これが必要になります。 ターゲットの Web アプリ ボットを作成すると、次のリソースが自動的にプロビジョニングされます。
-   Web アプリ ボット – ローカルのボットをデプロイするためにこのボットを使用します。
-   App Service プラン - App Service アプリの実行に必要なリソースを提供します。
-   App Service - Web アプリケーションをホストするためのサービス
-   ストレージ アカウント - すべての Azure Storage データ オブジェクト (BLOB、ファイル、キュー、テーブル、ディスク) が含まれます。

ターゲットの Web アプリ ボットの作成時に、ボット用のアプリ ID とパスワードも生成されます。 Azure では、アプリ ID とパスワードによって、[サービスの認証と承認](https://docs.microsoft.com/azure/app-service/overview-authentication-authorization)がサポートされます。 ローカルのボット コードで使用するために、後ほどこの情報の一部を取得します。 

> [!IMPORTANT]
> Azure portal で使用されているボット テンプレート用のプログラミング言語は、お使いのボットの記述に使用されたプログラミング言語と一致する必要があります。

使用するボットが Azure に既に作成されている場合、新しい Web アプリ ボットの作成は省略できます。

1. [Azure Portal](https://portal.azure.com) にログインします。
1. Azure portal の左上隅にある **[新しいリソースの作成]** リンクをクリックし、**[AI + Machine Learning] > [Web App Bot]** を選択します。
1. Web App Bot に関する情報が含まれた 新しいブレードが開きます。 
1. **[ボット サービス]** ブレードで、ボットに関する必要な情報を指定します。
1. **[作成]** をクリックしてサービスを作成し、ボットをクラウドにデプロイします。 このプロセスには数分かかることがあります。

### <a name="download-the-source-code"></a>ソース コードをダウンロードする
ターゲットの Web アプリ ボットを作成したら、ボット コードを Azure portal からローカル コンピューターにダウンロードする必要があります。 コードをダウンロードする理由は、appsettings.json または .env ファイル内のサービス参照 (MicrosoftAppID、MicrosoftAppPassword、LUIS、QnA など) を取得するためです。 

1. **[ボット管理]** セクションで **[ビルド]** をクリックします。
1. 右側のウィンドウで **[ボットのソース コードをダウンロードする]** リンクをクリックします。
1. 指示に従ってコードをダウンロードし、フォルダーを解凍します。
    1. [!INCLUDE [download keys snippet](~/includes/snippet-abs-key-download.md)]

### <a name="update-your-local-appsettingsjson-or-env-file"></a>ローカルの appsettings.json または .env ファイルを更新する

ダウンロードした appsettings.json または .env ファイルを開きます。 そこに記載されている**すべて**のエントリをコピーして、"_ローカルの_" appsettings.json または .env ファイルに追加します。 重複するすべてのサービス エントリまたはサービス ID を解決します。 ご使用のボットで使用する追加のサービス参照は保持します。

ファイルを保存します。

### <a name="update-local-bot-code"></a>ローカルのボット コードを更新する
ローカルの Startup.cs または index.js ファイルを、.bot ファイルではなく、appsettings.json または .env ファイルを使用するように更新します。 .bot は非推奨になっており、Microsoft では、.bot ファイルではなく appsettings.json または .env ファイルを使用するように、VSIX テンプレート、Yeoman ジェネレーター、サンプル、および残りのドキュメントを更新することに取り組んでいます。 それまでの間、お客様はボット コードを変更する必要があります。 

appsettings.json または .env ファイルから設定を読み取るようにコードを更新します。 

# <a name="ctabcsharp"></a>[C#](#tab/csharp)
`ConfigureServices` メソッドの場合、ASP.NET Core によって提供される次のような構成オブジェクトを使用します。 

**Startup.cs**
```csharp
var appId = Configuration.GetSection("MicrosoftAppId").Value;
var appPassword = Configuration.GetSection("MicrosoftAppPassword").Value;
options.CredentialProvider = new SimpleCredentialProvider(appId, appPassword);
```

# <a name="jstabjs"></a>[JS](#tab/js)

JavaScript の場合、次のような `process.env` オブジェクトから .env 変数の参照を解除します。
   
**index.js**

```js
const adapter = new BotFrameworkAdapter({
    appId: process.env.MicrosoftAppId,
    appPassword: process.env.MicrosoftAppPassword
});
```
---

- ファイルを保存し、自身のボットをテストします。

### <a name="setup-a-repository"></a>リポジトリを設定する

継続的配置をサポートするには、お好みの Git ソース管理プロバイダーを使用して、Git リポジトリを作成します。 リポジトリにコードをコミットします。

[リポジトリの準備](https://docs.microsoft.com/azure/app-service/deploy-continuous-deployment#prepare-your-repository)に関するページの説明にあるように、リポジトリのルートに適切なファイルがあることを確認してください。

### <a name="update-app-settings-in-azure"></a>Azure でアプリ設定を更新する
ローカルのボットでは暗号化された .bot ファイルが使用されませんが、Azure portal が暗号化された .bot ファイルを使用するように構成されていた場合は "_どうなるでしょうか_"。 これを解決するには、Azure のボット設定に保存されている **botFileSecret** を削除します。
1. Azure portal で、ボットの **Web App Bot** リソースを開きます。
1. ボットの **[アプリケーション設定]** を開きます。
1. **[アプリケーション設定]** ウィンドウで、**[アプリケーション設定]** まで下へスクロールします。
1. ボットに **botFileSecret** および **botFilePath** エントリがあるかどうか確認します。 ある場合は、これを削除します。
1. 変更を保存します。

## <a name="2-deploy-using-azure-deployment-center"></a>2.Azure デプロイ センターを使用してデプロイする

ここで、ボット コードを Azure にアップロードする必要があります。 「[Continuous deployment to Azure App Service (Azure App Service への継続的配置)](https://docs.microsoft.com/azure/app-service/deploy-continuous-deployment)」の指示に従います。

`App Service Kudu build server` を使用してビルドすることをお勧めします。

継続的配置を構成したら、リポジトリにコミットした変更が発行されます。 ただし、お使いのボットにサービスを追加する場合は、それらのエントリを .bot ファイルに追加する必要があります。

## <a name="3-test-your-deployment"></a>手順 3.デプロイのテスト

デプロイが成功したら、数秒待ちます。必要に応じて、Web アプリを再起動してキャッシュをクリアします。 Web App Bot ブレードに戻り、Azure portal で提供されている Web チャットを使用してテストします。

## <a name="additional-resources"></a>その他のリソース
- [継続的なデプロイに関する一般的な問題の調査方法](https://github.com/projectkudu/kudu/wiki/Investigating-continuous-deployment)

