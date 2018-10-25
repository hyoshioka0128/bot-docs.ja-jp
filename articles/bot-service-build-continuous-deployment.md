---
title: Bot Service の継続的配置を構成する | Microsoft Docs
description: Bot Service のソース管理からの継続的デプロイを設定する方法について説明します。
keywords: 継続的配置, 公開, デプロイ, Azure portal
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 09/18/2018
ms.openlocfilehash: 62cbbcc560e049776b8aa891c167b9a6eaba3264
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/24/2018
ms.locfileid: "49997799"
---
# <a name="set-up-continuous-deployment"></a>Azure App Service での GIT による継続的なデプロイ
コードが **GitHub** または **Azure DevOps (旧称 Visual Studio Team Services)** にチェックインされる場合は、継続的デプロイを使用して、コードの変更をソース リポジトリから Azure に自動的にデプロイします。 このトピックでは、**GitHub** および **Azure DevOps** に対して継続的デプロイを設定する方法を説明します。

> [!NOTE]
> 継続的デプロイを設定すると、Azure portal のオンライン コード エディターは "*読み取り専用*" になります。

## <a name="continuous-deployment-using-github"></a>GitHub を使用した継続的デプロイ

GitHub を使用して継続的デプロイを設定するには、次のようにします。

1. Azure にデプロイするソース コードを含む GitHub リポジトリを使用します。 既存のリポジトリを[フォーク](https://help.github.com/articles/fork-a-repo/)するか、または独自のリポジトリを作成して、関連するソース コードを GitHub リポジトリにアップロードすることができます。
2. [Azure portal](https://portal.azure.com) でボットの **[Build]\(ビルド\)** ブレードに移動して、**[Configure continuous deployment]\(継続的デプロイの構成\)** をクリックします。 
3. **[Setup]** をクリックします。
   
   ![継続的なデプロイの設定](~/media/azure-bot-build/continuous-deployment-setup.png)

4. **[ソースの選択]** をクリックして、**[GitHub]** を選択します。

   ![GitHub を選択する](~/media/azure-bot-build/continuous-deployment-setup-github.png)

5. **[Authorization]\(承認\)** をクリックして、**[Authorize]\(承認する\)** ボタンをクリックし、指示に従って GitHub アカウントにアクセスする承認を Azure に与えます。

6. **[プロジェクトの選択]** をクリックして、プロジェクトを選択します。

7. **[ブランチの選択]** をクリックして、ブランチを選択します。

8. **[OK]** をクリックして設定プロセスを完了します。

GitHub での継続的デプロイの設定が完了しました。 ソース コード リポジトリにコミットするたびに、変更が Azure Bot Service に自動的にデプロイされます。

## <a name="continuous-deployment-using-azure-devops"></a>Azure DevOps を使用した継続的デプロイ

1. [Azure portal](https://portal.azure.com) で、ボットの **[Build]\(ビルド\)** ブレードから **[Configure continuous deployment]\(継続的デプロイの構成\)** をクリックします。 
2. **[Setup]** をクリックします。
   
   ![継続的なデプロイの設定](~/media/azure-bot-build/continuous-deployment-setup.png)

3. **[ソースの選択]** をクリックして、**[Visual Studio Team Services]** を選択します。 Visual Studio Team Services の名称は Azure DevOps Services に変更されました。

   ![Visual Studio Team Services を選択する](~/media/azure-bot-build/continuous-deployment-setup-vs.png)

4. **[Choose your account]\(アカウントの選択\)** をクリックして、アカウントを選択します。

> [!NOTE]
> お使いのアカウントが表示されない場合は、アカウントが Azure サブスクリプションにリンクされていることを確認します。 アカウントを Azure サブスクリプションにリンクするには、Azure Portal に移動して、**Azure DevOps Services (旧称 Team Services) の組織**を開きます。 Azure DevOps にある組織の一覧が表示されます。 デプロイするボットのソース コードを含む組織をクリックすると、**[AAD に接続]** ボタンが表示されます。 選択した組織が Azure サブスクリプションにリンクされない場合は、このボタンがアクティブになります。 したがって、接続を設定するには、このボタンをクリックしてください。 設定を有効にするには、しばらく待機しなければならない場合があります。

5. **[プロジェクトの選択]** をクリックして、プロジェクトを選択します。

> [!NOTE]
> サポートされているのは VSTS Git プロジェクトだけです。

6. **[ブランチの選択]** をクリックして、ブランチを選択します。
7. **[OK]** をクリックして設定プロセスを完了します。

   ![Visual Studio の構成](~/media/azure-bot-build/continuous-deployment-setup-vs-configuration.png)

Azure DevOps での継続的デプロイの設定が完了しました。 コミットするたびに、変更が Azure に自動的にデプロイされます。

## <a name="disable-continuous-deployment"></a>継続的なデプロイの無効化

ボットが継続的デプロイ用に構成されている間は、オンライン コード エディターを使ってボットを変更することはきません。 オンライン コード エディターを使いたい場合は、継続的デプロイを一時的に無効にできます。

継続的デプロイを無効にするには、次の手順のようにします。

1. ボットの **[Build]\(ビルド\)** ブレードで、**[Configure continuous deployment]\(継続的デプロイの構成\)** をクリックします。 
2. **[Disconnect]\(切断\)** をクリックして、継続的デプロイを無効化にします。 継続的デプロイを再度有効にするには、上記の該当するセクションの手順を繰り返します。

## <a name="additional-information"></a>追加情報
- [Azure DevOps](https://docs.microsoft.com/en-us/azure/devops/?view=vsts)
