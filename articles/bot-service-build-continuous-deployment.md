---
title: Bot Service の継続的配置を構成する | Microsoft Docs
description: Bot Service のソース管理からの継続的デプロイを設定する方法について説明します。
keywords: 継続的配置, 公開, デプロイ, Azure portal
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/06/2018
ms.openlocfilehash: ffbc3ef83c8fe1cd6f04697a3fff9e229df9956f
ms.sourcegitcommit: 080b9633925ffe381f2c3cf11c8f8ca4b37e2046
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/07/2018
ms.locfileid: "53068715"
---
# <a name="set-up-continuous-deployment"></a>Azure App Service での GIT による継続的なデプロイ
コードが **GitHub** または **Azure DevOps (旧称 Visual Studio Team Services)** にチェックインされる場合は、継続的デプロイを使用して、コードの変更をソース リポジトリから Azure に自動的にデプロイします。 このトピックでは、**GitHub** および **Azure DevOps** に対して継続的デプロイを設定する方法を説明します。

> [!NOTE]
> この記事で説明するシナリオでは、ボットが Azure にデプロイ済みであり、そのボットの継続的デプロイを有効にする必要があることを前提としています。 また、継続的デプロイを設定すると、Azure portal 上のオンライン コード エディターが読み取り専用になることも覚えておいてください。

## <a name="continuous-deployment-using-github"></a>GitHub を使用した継続的デプロイ

Azure にデプロイするソース コードが含まれる GitHub リポジトリを使用して継続的デプロイを設定するには、以下の手順を実行します。

1. [Azure portal](https://portal.azure.com) 上で、ボットの **[All App service settings]\(すべての App Service の設定\)** ブレードをクリックし、**[Deployment options (Classic)]\(デプロイ オプション (クラシック)\)** をクリックします。 

1. **[ソースの選択]** をクリックして、**[GitHub]** を選択します。

   ![GitHub を選択する](~/media/azure-bot-build/continuous-deployment-setup-github.png)

1. **[Authorization]\(承認\)** をクリックして、**[Authorize]\(承認する\)** ボタンをクリックし、指示に従って GitHub アカウントにアクセスする承認を Azure に与えます。

1. **[プロジェクトの選択]** をクリックして、プロジェクトを選択します。

1. **[ブランチの選択]** をクリックして、ブランチを選択します。

1. **[OK]** をクリックして設定プロセスを完了します。

GitHub での継続的デプロイの設定が完了しました。 ソース コード リポジトリにコミットするたびに、変更が Azure Bot Service に自動的にデプロイされます。

## <a name="continuous-deployment-using-azure-devops"></a>Azure DevOps を使用した継続的デプロイ

1. [Azure portal](https://portal.azure.com) 上で、ボットの **[All App service settings]\(すべての App Service の設定\)** ブレードをクリックし、**[Deployment options (Classic)]\(デプロイ オプション (クラシック)\)** をクリックします。 
2. **[ソースの選択]** をクリックして、**[Visual Studio Team Services]** を選択します。 Visual Studio Team Services の名称は Azure DevOps Services に変更されました。

   ![Visual Studio Team Services を選択する](~/media/azure-bot-build/continuous-deployment-setup-vs.png)

3. **[Choose your account]\(アカウントの選択\)** をクリックして、アカウントを選択します。

> [!NOTE]
> お使いのアカウントが表示されない場合は、[アカウントを Azure サブスクリプションにリンク](https://docs.microsoft.com/en-us/azure/devops/organizations/accounts/connect-organization-to-azure-ad?view=vsts&tabs=new-nav)する必要があります。 サポートされているのは VSTS Git プロジェクトのみであることに注意してください。

4. **[プロジェクトの選択]** をクリックして、プロジェクトを選択します。
5. **[ブランチの選択]** をクリックして、ブランチを選択します。
6. **[OK]** をクリックして設定プロセスを完了します。

   ![Visual Studio の構成](~/media/azure-bot-build/continuous-deployment-setup-vs-configuration.png)

Azure DevOps での継続的デプロイの設定が完了しました。 コミットするたびに、変更が Azure に自動的にデプロイされます。

## <a name="disable-continuous-deployment"></a>継続的なデプロイの無効化

ボットが継続的デプロイ用に構成されている間は、オンライン コード エディターを使ってボットを変更することはきません。 オンライン コード エディターを使いたい場合は、継続的デプロイを一時的に無効にできます。

継続的デプロイを無効にするには、次の手順のようにします。
1. [Azure portal](https://portal.azure.com) 上で、ボットの **[All App service settings]\(すべての App Service の設定\)** ブレードをクリックし、**[Deployment options (Classic)]\(デプロイ オプション (クラシック)\)** をクリックします。 
2. **[Disconnect]\(切断\)** をクリックして、継続的デプロイを無効化にします。 継続的デプロイを再度有効にするには、上記の該当するセクションの手順を繰り返します。

## <a name="additional-information"></a>追加情報
- Visual Studio Team Services の名称は [Azure DevOps Services](https://docs.microsoft.com/en-us/azure/devops/?view=vsts) に変更されました。
