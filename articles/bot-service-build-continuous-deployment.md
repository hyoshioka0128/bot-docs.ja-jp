---
title: Bot Service の継続的デプロイを構成する - Bot Service
description: Bot Service のソース管理からの継続的デプロイを設定する方法について説明します。
keywords: 継続的配置, 公開, デプロイ, Azure portal
author: ivorb
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 03/23/2020
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: edb1d18273179939a333835b6ff27b7aed9f156b
ms.sourcegitcommit: 2412f96ad8f74dfa615c71f566c5befffb920658
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/25/2020
ms.locfileid: "82158780"
---
# <a name="set-up-continuous-deployment"></a>Azure App Service での GIT による継続的なデプロイ

[!INCLUDE [applies-to](./includes/applies-to.md)]

この記事では、お使いのボットの継続的デプロイを構成する方法を示します。 継続的デプロイを有効にすると、コード変更をソース リポジトリから Azure に自動的にデプロイできます。 このトピックでは、GitHub に対して継続的デプロイを設定する方法を説明します。 その他のソース管理システムでの継続的デプロイの設定については、このページの下部にある「その他のリソース」セクションを参照してください。

## <a name="prerequisites"></a>前提条件

- Azure サブスクリプションをお持ちでない場合は、開始する前に [無料アカウント](https://portal.azure.com) を作成してください。
- 継続的デプロイを有効にする前に、[お使いのボットを Azure にデプロイする](bot-builder-deploy-az-cli.md)**必要があります**。

## <a name="prepare-your-repository"></a>リポジトリを準備する

リポジトリのルートに、プロジェクトの適切なファイルがあることを確認してください。 これにより、ビルド プロバイダーから自動ビルドを取得できます。

|ランタイム | ルート ディレクトリのファイル |
|:-------|:---------------------|
| ASP.NET Core | .sln または .csproj |
| Node.js | server.js、app.js、またはスタート スクリプトを含む package.json |
| Python | app.py |

## <a name="continuous-deployment-using-github"></a>GitHub を使用した継続的デプロイ

GitHub を使用する継続的デプロイを有効にするには、Azure portal でご自身のボットの **App Service** ページに移動します。

1. **[Deployment Center]**  >  **[GitHub]**  >  **[Authorize]** をクリックします。

   ![継続的デプロイ](~/media/azure-bot-build/azure-deployment.png)

   1. 表示されたブラウザー ウィンドウで、 **[Authorize AzureAppService]\(AzureAppService の承認\)** をクリックします。

      ![Azure Github のアクセス許可](~/media/azure-bot-build/azure-deployment-github.png)

   1. **AzureAppService** の承認後、Azure portal の**デプロイ センター**に戻ります。

1. **[Continue]** をクリックします。

        > [!div class="mx-imgBorder"]
        > ![Continue to build provider](~/media/azure-bot-build/azure-deployment-continue.png)

1. **[ビルド プロバイダー]** ページで、使用するビルド プロバイダーを選択し、 **[続行]** をクリックします。

1. **[構成]** ページで必要な情報を入力し、 **[続行]** をクリックします。 必要な情報は、選択したソース管理サービスとビルド プロバイダーによって異なります。

1. **[Summary]\(概要\)** ページで設定を確認して、 **[完了]** をクリックします。

この時点で、GitHub での継続的デプロイは設定されています。 選択したリポジトリおよびブランチでの新しいコミットが App Service アプリに継続的にデプロイされるようになりました。 **[管理者用センター]** ページで、コミットとデプロイを追跡できます。

## <a name="disable-continuous-deployment"></a>継続的なデプロイの無効化

ボットが継続的デプロイ用に構成されている間は、オンライン コード エディターを使ってボットを変更することはきません。 オンライン コード エディターを使いたい場合は、継続的デプロイを一時的に無効にできます。

継続的デプロイを無効にするには、次の手順のようにします。

1. [Azure portal](https://portal.azure.com) 上で、ボットの **[All App service settings]\(すべての App Service の設定\)** ブレードに移動し、 **[デプロイ センター]** をクリックします。
1. **[Disconnect]\(切断\)** をクリックして、継続的デプロイを無効化にします。 継続的デプロイを再度有効にするには、上記の該当するセクションの手順を繰り返します。

## <a name="additional-resources"></a>その他のリソース

- Azure での継続的デプロイの詳細については、「[Azure App Service への継続的デプロイ](https://docs.microsoft.com/azure/app-service/deploy-continuous-deployment)」をご覧ください。
- ビルド プロバイダーに GitHub Actions を使用すると、リポジトリにワークフローが作成されます。 GitHub のサイトで、[GitHub Actions](https://help.github.com/en/actions) の使用法の詳細を確認できます。
