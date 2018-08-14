---
title: 使用するボットを Azure にデプロイする | Microsoft Docs
description: 使用するボットを Azure クラウドにデプロイします。
keywords: ボットをデプロイ, Azure のデプロイ, ボット チャネルの登録, Visual Studio を発行
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: get-started-article
ms.prod: bot-framework
ms.date: 05/14/2018
ms.openlocfilehash: 70a3b7f093bb80dd16c854c65331c141fbba3725
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/18/2018
ms.locfileid: "39303825"
---
# <a name="deploy-your-bot-to-azure"></a>使用するボットを Azure にデプロイする

ご自分のボットを作成し、ローカルで検証したら、それを Azure にプッシュすることで、どこからでもアクセスできるようにすることができます。 そのためには、まず App Service でボットを Azure にデプロイし、次に Azure Bot Service の Bot Channels Registration アイテムを使用してご自分のボットを構成します。

## <a name="publish-from-visual-studio"></a>Visual Studio から発行する

Visual Studio を使用して、Azure 内でリソースを作成し、ご自分のコードを発行します。

[ソリューション エクスプローラー] ウィンドウで、ご利用のプロジェクトのノードを右クリックし、[発行] を選択します。

![設定の発行](media/azure-bot-quickstarts/getting-started-publish-setting.png)

2. [発行先を選択] ダイアログ内の左側で **[App Service]** を、そして右側で **[新規作成]** を確実に設定します。

3. [発行] ボタンをクリックします。

4. ダイアログの右上で、ダイアログにご利用の Azure サブスクリプションの適切なユーザー ID が表示されていることを確認します。

![メインの発行](media/azure-bot-quickstarts/getting-started-publish-main.png)

5. [アプリケーション名]、[サブスクリプション]、[リソース グループ]、[ホスティング プラン] に情報を入力します。

6. 準備ができたら、[作成] をクリックします。 プロセスが完了するのに、数分かかることがあります。

7. 完了すると、ご自分のボットのパブリック URL を示す Web ブラウザーが開きます。

8. この URL (https://<yourbotname>.azurewebsites.net/ のような形式になります) のコピーを作成します。

> [!NOTE] 
> 使用するボットを登録するときは、URL の HTTPS バージョンを使用する必要があります。 Azure の Azure App Service では SSL がサポートされています。

## <a name="create-your-bot-channels-registration"></a>ご自分の Bot Channels Registration チャネルを作成する
使用するボットが Azure にデプロイされている場合は、そのボットを Azure Bot Service に登録する必要があります。

1. Azure Portal (https://portal.azure.com) にアクセスします。

2. 以前に Visual Studio で使用していたのと同じ ID を使用してログインし、ご自分のボットを発行します。

3. [リソースの作成] をクリックします。

4. [Marketplace を検索] フィールドで、「Bot Channels Registration」と入力し、Enter キーを押します。

5. 返された一覧内で、[Bot Channels Registration] を選択します。

![[発行]](media/azure-bot-quickstarts/getting-started-bot-registration.png)

6. 表示されたブレードで、[作成] をクリックします。

7. ご自分のボットの名前を指定します。

8. ご自分のボットのコードをデプロイしたのと同じサブスクリプションを選択します。

9. ご利用の既存のリソース グループを選択します。これにより場所が設定されます。

10. 開発およびテストの場合は F0 価格レベルを選択できます。

11. ご自分のボットの URL を入力します。 必ず、HTTPS で始めて、/api/messages を追加します。たとえば、https://yourbotname.azurewebsites.net/api/messages のようになります。

12. ここで、[Application Insights] をオフにします。

13. [Microsoft App ID]\(Microsoft アプリ ID) と [パスワード] をクリックします。

14. [新規] ブレードで [新規作成] をクリックします。

15. 右側に表示された新しいブレードで、新しいブラウザー タブに表示される [Create App ID in the App Registration Portal]\(アプリ登録ポータルでアプリ ID を作成する\) をクリックします。

![ボット msa](media/azure-bot-quickstarts/getting-started-msa.png)

16. 新しいタブで、アプリ ID のコピーを作成し、それをどこかの場所に保存します。 

17. [アプリ パスワードを生成して続行] ボタンをクリックします。

18. ブラウザー ダイアログが開き、アプリのパスワードが提供されます。パスワードを取得できるのはこのときだけとなります。 後で利用できるように、このパスワードをコピーしてどこかの場所に保存します。

19. パスワードが保存されたら、[OK] をクリックします。

20. [ブラウザー] タブを閉じて、[Azure Portal] タブに戻ります。

21. アプリ ID とパスワードを適切な正しいフィールドに貼り付けて、[OK] をクリックします。

22. ここで、[作成] をクリックすると、ご利用になるチャネル登録が設定されます。 これには、数秒から数分かかる場合があります。

## <a name="update-your-bots-application-settings"></a>ご利用のボットのアプリケーション設定を更新する
ご利用のボットが Azure Bot Service で認証されるようにするには、Azure App Service 内で、ご利用のボットの [アプリケーションの設定] に 2 つの設定を追加する必要があります。 

1. [App Services] をクリックします。 [サブスクリプション] テキスト ボックスに、ご利用のボットの名前を入力します。 リスト内で、ご利用のボットの名前をクリックします。

![App Service](media/azure-bot-quickstarts/getting-started-app-service.png)

2. 左側にあるオプションの一覧で、ご利用のボットのオプション内の [設定] セクションにある [アプリケーションの設定] を検索し、それをクリックします。

![ボット ID](media/azure-bot-quickstarts/getting-started-app-settings-1.png)

3. [アプリケーションの設定] セクションが見つかるまでスクロールします。

![ボット msa](media/azure-bot-quickstarts/getting-started-app-settings-2.png)

4. [新しい設定の追加] をクリックします。

5. 名前として「**MicrosoftAppId**」を入力し、値としてご利用のアプリ ID を入力します。

6. [新しい設定の追加] をクリックします。

7. 名前として「**MicrosoftAppPassword**」を入力し、値としてご自分のパスワードを入力します。

8. 上部の [保存] ボタンをクリックします。

## <a name="test-your-bot-in-production"></a>ご利用のボットを運用環境でテストする
この時点では、組み込みの Web チャット クライアントを使用して、ご利用のボットを Azure からテストすることができます。

1. ポータル内で、ご利用のリソース グループに戻ります。

2. ご利用のボット登録を開きます。

3. [Bot management]\(ボット管理\) で、[Test in Web Chat]\(Web チャットでのテスト\) を選択します。

![Web チャットでのテスト](media/azure-bot-quickstarts/getting-started-test-webchat.png)

4. `Hi` のようなメッセージを入力し、Enter キーを押します。 ボットから `Turn 1: You sent Hi` がエコーで返されます。

