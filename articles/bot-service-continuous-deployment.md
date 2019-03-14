---
title: ソース管理または Visual Studio から Bot Service を発行する | Microsoft Docs
description: Visual Studio から 1 回、またはソース管理から継続的に、Bot Service のボットを発行する方法について説明します。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: abs
ms.date: 12/13/2017
ms.openlocfilehash: 65de0e4e4be129c9fa467cd8610cf0f0b13e5965
ms.sourcegitcommit: b2245df2f0a18c5a66a836ab24a573fd70c7d272
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/07/2019
ms.locfileid: "57568179"
---
# <a name="publish-a-bot-to-bot-service"></a>ボットを Bot Service に発行する

C# のボット ソース コードを更新した後は、Visual Studio を使って、Bot Service で実行中のボットに発行することができます。 統合開発環境 (IDE) で作成された C# または Node.js のボット ソース コードを、ソース管理サービスにソース ファイルをチェックインするたびに自動的に発行することもできます。


## <a name="publish-a-bot-on-app-service-plan-from-the-online-code-editor"></a>オンライン コード エディターから App Service プランにボットを発行する

継続的デプロイを構成していない場合は、オンライン コード エディターでソース ファイルを変更できます。 変更したソースをデプロイするには、次の手順のようにします。

4. [コンソールを開く] アイコンをクリックします。  
    ![コンソール アイコン](~/media/azure-bot-service-console-icon.png)
2. コンソール ウィンドウで、「**build.cmd**」と入力して Enter キーを押します。


## <a name="publish-c-bot-on-app-service-plan-from-visual-studio"></a>Visual Studio から App Service プランに C# のボットを発行する 

`.PublishSettings` ファイルを使って Visual Studio からの発行を設定するには、次の手順のようにします。

1. Azure portal でお使いの Bot Service をクリックし、**[ビルド]** タブをクリックして、**[Download zip file]\(zip ファイルのダウンロード\)** をクリックします。
    1. [!INCLUDE [download keys snippet](~/includes/snippet-abs-key-download.md)]
3. ダウンロードした zip ファイルの内容をローカル フォルダーに抽出します。
4. エクスプローラーで、ボットの Visual Studio ソリューション (.sln) ファイルを探し、それをダブルクリックします。
4. Visual Studio で、**[表示]** をクリックして **[ソリューション エクスプローラー]** をクリックします。
5. ソリューション エクスプローラー ウィンドウで、プロジェクトを右クリックして、**[発行]** をクリックします。[発行] ウィンドウが開きます。 
6. [発行] ウィンドウで、**[新しいプロファイルの作成]**、**[プロファイルのインポート]**、**[OK]** の順にクリックします。
7. プロジェクト フォルダー、**[PostDeployScripts]** フォルダーの順に移動し、末尾が `.PublishSettings` のファイルを選択して、**[開く]** をクリックします。

これで、このプロジェクトの発行が構成されました。 ローカル ソース コードを Bot Service に発行するには、プロジェクトを右クリックし、**[発行]** をクリックして、**[発行]** ボタンをクリックします。 

## <a name="set-up-continuous-deployment"></a>Azure App Service での GIT による継続的なデプロイ

Bot Service の既定の設定では、Azure エディターを使ってブラウザー内で直接ボットを開発でき、ローカル エディターやソース管理は必要ありまあせん。 ただし、Azure エディターでは、アプリケーション内のファイルを管理できません (例: ファイルの追加、ファイルの名前変更、ファイルの削除)。 アプリケーション内のファイルを管理する機能が必要な場合は、継続的デプロイを設定し、適切な統合開発環境 (IDE) とソース管理システムを使用できます (Visual Studio Team、GitHub、Bitbucket など)。 継続的デプロイを構成すると、ソース管理にコミットしたすべてのコード変更は、Azure に自動的にデプロイされます。 継続的デプロイを構成した後は、[ボットをローカルにデバッグする](bot-service-debug-bot.md)ことができます。

> [!NOTE]
> ボットの継続的デプロイを有効にする場合は、ソース管理サービスにコード変更をチェックインする必要があります。 再び Azure エディターでコードを編集する場合は、[継続的デプロイを無効にする](#disable-continuous-deployment)必要があります。

ボット アプリの継続的デプロイを有効にするには、次の手順のようにします。

## <a name="set-up-continuous-deployment-for-a-bot-on-an-app-service-plan"></a>App Service プランでボットの継続的デプロイを設定する

このセクションでは、App Service ホスティング プランのある Bot Service を使用して作成したボットの継続的デプロイを有効にする方法について説明します。

1. Azure portal 内で対象の Azure ボットを見つけて、**[ビルド]** タブをクリックし、**[Continuous deployment from source control]\(ソース管理からの継続的デプロイ\)** セクションを探します。
2. Visual Studio Online または GitHub の場合は、それらの Web サイトでユーザーに発行されたアクセス トークンを指定します。 ソースが Azure からソース リポジトリに取り込まれます。
3. 他のソース管理システムの場合は、**[other]\(その他\)** を選択して、表示される手順に従います。 
3. **[有効]** をクリックします。  

### <a name="create-an-empty-repository-and-download-bot-source-code"></a>空のリポジトリを作成してボットのソース コードをダウンロードする

Visual Studio Online または GitHub "*以外の*" ソース管理サービスを使う場合は、次の手順のようにします。 Visual Studio Online および GitHub はボットのソース コードを Azure から取り込むので、これら 2 つのサービスのユーザーは次の手順をスキップできます。

3. App Service プランでのボットの場合は、Azure でボットのページを探し、**[ビルド]** タブをクリックし、**[Download source code]\(ソース コードのダウンロード\)** セクションを探して、**[Download zip file]\(zip ファイルのダウンロード\)** をクリックします。
    1. [!INCLUDE [download keys snippet](~/includes/snippet-abs-key-download.md)]
1. Azure がサポートするソース管理システムのいずれかで空のリポジトリを作成します。

    ![ソース管理システム](~/media/continuous-integration-sourcecontrolsystem.png)

3. デプロイ ソースの同期に使うローカル フォルダーに、ダウンロードした zip ファイルの内容を抽出します。
4. **[Configure]\(構成\)** をクリックして、表示される手順に従います。 

## <a name="set-up-continuous-deployment-for-a-bot-on-a-consumption-plan"></a>従量課金プランでボットの継続的デプロイを設定する 

ボットのデプロイ ソースを選択し、リポジトリに接続します。 

1. Azure poral 内の Azure ボットで、**[設定]** タブをクリックし、**[Configure]\(構成\)** をクリックして **[Continuous deployment]\(継続的デプロイ\)** セクションをデプロイします。  
2. 手順に従い、チェック ボックスをオンにして準備ができたことを確認します。 
3. **[Configure]\(構成\)** をクリックして、前に空のリポジトリを作成したソース管理システムに対応するデプロイ ソースを選択し、手順を完了してソースに接続します。   


## <a name="disable-continuous-deployment"></a>継続的なデプロイの無効化 

継続的デプロイを無効にしてもソース管理サービスは引き続き機能していますが、チェックインした変更が Azure に自動的に発行されることはなくなります。 継続的デプロイを無効にするには、次の手順のようにします。

1. App Service ホスティング プランのボットの場合は、Azure portal 内で対象の Azure ボットを見つけて、**[ビルド]** タブをクリックし、**[Continuous deployment from source control]\(ソース管理からの継続的デプロイ\)** セクションを探します。*または、* 
2. 従量課金プランのボットの場合は、**[設定]** タブをクリックし、**[Continuous deployment]\(継続的デプロイ\)** セクションをデプロイして、**[Configure]\(構成\)** をクリックします。
3. **[デプロイ]** ウィンドウで、継続的デプロイが有効になっているソース管理サービスを選択し、**[切断]** クリックします。  


## <a name="additional-resources"></a>その他のリソース

継続的デプロイを構成した後でボットをローカルにデバッグする方法については、「[Bot Service のボットをデバッグする](bot-service-debug-bot.md)」をご覧ください。

この記事では、Bot Service の特定の継続的デプロイ機能が説明されています。 Azure App Services に関連する継続的デプロイについては、「<a href="https://azure.microsoft.com/en-us/documentation/articles/app-service-continuous-deployment/" target="_blank">Azure App Service への継続的デプロイ</a>」をご覧ください。
