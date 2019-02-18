---
redirect_url: /bot-framework/bot-builder-deploy-az-cli
ms.openlocfilehash: a300d6602a59c5e7d7cebdf14bb4f720a30ecbf8
ms.sourcegitcommit: 8183bcb34cecbc17b356eadc425e9d3212547e27
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/09/2019
ms.locfileid: "55971472"
---
<a name="--"></a><!--
---
title:Visual Studio を使用して C# ボットをデプロイする | Microsoft Docs description:使用するボットを Azure クラウドにデプロイします。
keywords: ボットのデプロイ, azure へのデプロイ, ボットの発行, az によるボットのデプロイ, visual studio によるボットのデプロイ, msbot の発行, msbot 複製作成者: ivorb ms.author: v-ivorb マネージャー: kamrani ms.topic: get-started-article ms.service: bot-service ms.subservice: abs ms.date:2019/02/07
---

# <a name="deploy-your-c-bot-using-visual-studio"></a>Visual Studio を使用して C# ボットをデプロイする

[!INCLUDE [pre-release-label](./includes/pre-release-label.md)]

ご自分のボットを作成し、ローカルでテストした後に、それを Azure にデプロイして、どこからでもアクセスできるようにすることができます。 ボットを Azure にデプロイするには、使用するサービスの料金を支払う必要があります。 [課金とコスト管理](https://docs.microsoft.com/en-us/azure/billing/)に関する記事で、Azure の課金の確認、使用量とコストの監視、アカウントとサブスクリプションの管理の方法について説明されています。

この記事では、Visual Studio と Azure portal を使用して C# ボットをデプロイする方法を紹介します。 ボットのデプロイに必要なものを完全に理解するために、手順を実行する前にこの記事を確認することをお勧めします。

## <a name="prerequisites"></a>前提条件
- [Bot Framework Emulator](https://aka.ms/Emulator-wiki-getting-started) をインストールします。
- [.bot](v4sdk/bot-file-basics.md) ファイルの知識。


## <a name="update-bot-file-properties"></a>.bot ファイルのプロパティを更新する

デプロイ プロセスを開始する前に、Visual Studio で .bot ファイルの次のプロパティを更新します。
- **ビルド アクション: コンテンツ**
- **出力ディレクトリにコピー: 常にコピーする**


## <a name="deploy-your-bot-in-app-service"></a>App Service でボットをデプロイする

最初に、App Service で Visual Studio から Azure にボットをデプロイします。 次に、Bot Channels Registration を使用して Azure Bot Service でボットを構成します。

**注:Visual Studio プロジェクト名にスペースが含まれる場合、以下に説明するデプロイ手順は機能しません。**

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

11. ご自分のボットの URL を入力します。 必ず、HTTPS で始めて、/api/messages を追加します。たとえば、 https://yourbotname.azurewebsites.net/api/messages のようになります。

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

1. Azure portal 内で、ご利用のリソース グループに戻ります。

2. ご利用のボットを開きます。

3. **[Bot management]\(ボット管理\)** で、**[Test in Web Chat]\(Web チャットでのテスト\)** を選択します。

![Web チャットでのテスト](media/azure-bot-quickstarts/getting-started-test-webchat.png)

4. `Hi` のようなメッセージを入力し、Enter キーを押します。 ボットから `Turn 1: You sent Hi` がエコーで返されます。

---

## <a name="additional-resources"></a>その他のリソース

ボットをデプロイするときに、通常これらのリソースが Azure portal 上で作成されます。

| リソース      | 説明 |
|----------------|-------------|
| Web アプリ ボット | Azure App Service にデプロイされる Azure Bot Service ボット。|
| [App Service](https://docs.microsoft.com/en-us/azure/app-service/)| Web アプリケーションをビルドおよびホストできます。|
| [[App Service プラン]](https://docs.microsoft.com/en-us/azure/app-service/azure-web-sites-web-hosting-plans-in-depth-overview)| Web アプリを実行するための一連のコンピューティング リソースを定義します。|
| [Application Insights](https://docs.microsoft.com/en-us/azure/application-insights/app-insights-overview)| テレメトリを収集および分析するためのツールを提供します。|
| [ストレージ アカウント](https://docs.microsoft.com/en-us/azure/storage/common/storage-introduction)| 高い可用性、セキュリティ、耐久性、スケーラビリティ、および冗長性を備えたクラウド ストレージを提供します。|

Azure リソース グループにあまり馴染みがない場合は、こちらの「[用語集](https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-overview#terminology)」のトピックを参照してください。

## <a name="next-steps"></a>次の手順
> [!div class="nextstepaction"]
> [継続的デプロイを設定する](bot-service-build-continuous-deployment.md)
-->
