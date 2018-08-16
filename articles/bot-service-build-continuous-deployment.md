---
title: Bot Service の継続的配置を構成する | Microsoft Docs
description: Bot Service のソース管理からの継続的デプロイを設定する方法について説明します。
keywords: 継続的配置, 公開, デプロイ, Azure portal
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 03/08/2018
ms.openlocfilehash: 596d264c4df72959c71ab353e5038175fc2bed31
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/18/2018
ms.locfileid: "39301868"
---
# <a name="set-up-continuous-deployment"></a>Azure App Service での GIT による継続的なデプロイ

継続的デプロイを使うと、ボットをローカルに開発できます。 継続的デプロイは、**GitHub** や **Visual Studio Team Services** のようなソース管理にボットをチェックインする場合に便利です。 変更をソース リポジトリにチェックインし直すと、その変更が Azure に自動的にデプロイされます。

> [!NOTE]
> 継続的デプロイを設定すると、[オンライン コード エディター](bot-service-build-online-code-editor.md)は "*読み取り専用*" になります。

このトピックでは、**GitHub** および **Visual Studio Team Services** に対して継続的デプロイを設定する方法を説明します。

## <a name="continuous-deployment-using-github"></a>GitHub を使用した継続的デプロイ

GitHub を使用して継続的デプロイを設定するには、次のようにします。

1. Azure にデプロイするコードを含む GitHub リポジトリを[フォーク](https://help.github.com/articles/fork-a-repo/)します。
2. Azure portal からボットの **[Build]\(ビルド\)** ブレードに移動して、**[Configure continuous deployment]\(継続的デプロイの構成\)** をクリックします。 
3. **[Setup]** をクリックします。
   
   ![継続的なデプロイの設定](~/media/azure-bot-build/continuous-deployment-setup.png)

4. **[Choose source]\(ソースの選択\)** をクリックして、**[GitHub]** を選択します。

   ![GitHub を選択する](~/media/azure-bot-build/continuous-deployment-setup-github.png)

5. **[Authorization]\(承認\)** をクリックして、**[Authorize]\(承認する\)** ボタンをクリックし、指示に従って GitHub アカウントにアクセスする承認を Azure に与えます。

GitHub での継続的デプロイの設定が完了しました。 コミットするたびに、変更が Azure に自動的にデプロイされます。

## <a name="continuous-deployment-using-visual-studio"></a>Visual Studio を使用した継続的デプロイ

1. ボットの **[Build]\(ビルド\)** ブレードで、**[Configure continuous deployment]\(継続的デプロイの構成\)** をクリックします。 
2. **[Setup]** をクリックします。
   
   ![継続的なデプロイの設定](~/media/azure-bot-build/continuous-deployment-setup.png)

3. **[Choose source]\(ソースの選択\)** をクリックして、**[Visual Studio Team Services]** を選択します。

   ![Visual Studio Team Services を選択する](~/media/azure-bot-build/continuous-deployment-setup-vs.png)

4. **[Choose your account]\(アカウントの選択\)** をクリックして、アカウントを選択します。

> [!NOTE]
> お使いのアカウントが表示されない場合は、アカウントが Azure サブスクリプションにリンクされていることを確認します。
> 詳しくは、「[Linking your VSTS account to your Azure subscription](https://github.com/projectkudu/kudu/wiki/Setting-up-a-VSTS-account-so-it-can-deploy-to-a-Web-App#linking-your-vsts-account-to-your-azure-subscription)」(Azure サブスクリプションへの VSTS アカウントのリンク) をご覧ください。

5. **[Choose a project]\(プロジェクトの選択\)** をクリックして、プロジェクトを選択します。

> [!NOTE]
> サポートされているのは VSTS Git プロジェクトだけです。

6. **[Choose Branch]\(ブランチの選択\)** をクリックして、分岐するブランチを選択します。
7. **[OK]** をクリックして設定プロセスを完了します。

   ![Visual Studio の構成](~/media/azure-bot-build/continuous-deployment-setup-vs-configuration.png)

Visual Studio Team Services での継続的デプロイの設定が完了しました。 コミットするたびに、変更が Azure に自動的にデプロイされます。

## <a name="disable-continuous-deployment"></a>継続的なデプロイの無効化

ボットが継続的デプロイ用に構成されている間は、オンライン コード エディターを使ってボットを変更することはきません。 オンライン コード エディターを使いたい場合は、継続的デプロイを一時的に無効にできます。

継続的デプロイを無効にするには、次の手順のようにします。

1. ボットの **[Build]\(ビルド\)** ブレードで、**[Configure continuous deployment]\(継続的デプロイの構成\)** をクリックします。 
2. **[Disconnect]\(切断\)** をクリックして、継続的デプロイを無効化にします。 継続的デプロイを再度有効にするには、上記の該当するセクションの手順を繰り返します。

## <a name="next-steps"></a>次の手順
ボットが継続的デプロイ用に設定されたので、オンライン Web チャットを使ってコードをテストします。

> [!div class="nextstepaction"]
> [Web チャットでのテスト](bot-service-manage-test-webchat.md)
