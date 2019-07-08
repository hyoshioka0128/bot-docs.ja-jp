---
title: Bot Service を使用してボットを作成する | Microsoft Docs
description: 統合された専用のボット開発環境である Bot Service を使用してボットを作成する方法について説明します。
keywords: クイック スタート, ボットの作成, bot service, web app bot
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: abs
ms.date: 05/31/2019
ms.openlocfilehash: 9ef15b09c8517db9e8b1f13f72172f09f9fb23eb
ms.sourcegitcommit: dbbfcf45a8d0ba66bd4fb5620d093abfa3b2f725
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 06/28/2019
ms.locfileid: "67464618"
---
# <a name="create-a-bot-with-azure-bot-service"></a>Azure Bot Service を使用してボットを作成する

::: moniker range="azure-bot-service-3.0"

> [!NOTE]
> **2019 年 6 月 10 日以降、Azure portal で V3 SDK ボットを作成できなくなります。今後は、[V4 SDK](https://docs.microsoft.com/azure/bot-service/bot-service-quickstart?view=azure-bot-service-4.0) ボットを作成することをお勧めします。V3 SDK の長期サポートについて詳しくは、[こちら](https://docs.microsoft.com/azure/bot-service/bot-service-resources-bot-framework-faq?view=azure-bot-service-3.0#bot-framework-sdk-version-3-lifetime-support)** をご覧ください。 

Bot Service では、ボットを開発するための Bot Framework SDK や、ボットをチャンネルに接続するための Bot Framework など、ボットを作成するためのコア コンポーネントが提供されています。 Bot Service には、.NET および Node.js をサポートするボットを作成するときに選択できる 5 つのテンプレートが用意されています。 このトピックでは、Bot Service を使用して、Bot Framework SDK を使用する新しいボットを作成する方法について説明します。

## <a name="log-in-to-azure"></a>Azure にログインする
[Azure Portal](http://portal.azure.com) にログインします。

## <a name="create-a-new-bot-service"></a>新しいボット サービスを作成する

1. Azure portal の左上隅にある **[新しいリソースの作成]** リンクをクリックし、 **[AI + Machine Learning] > [Web App Bot]** を選択します。 

2. **Web App Bot** に関する情報が含まれた新しいブレードが開きます。  

3. **[Bot Service]** ブレードで、画像の下の表に示すように、ボットに関する必要な情報を入力します。  <br/>
   ![Web App Bot の作成ブレード](./media/azure-bot-quickstarts/sdk-create-bot-service-blade.png)

   | Setting | 推奨値 | 説明 |
   | ---- | ---- | ---- |
   | **ボット名** | ボットの表示名 | チャンネルとディレクトリに表示されるボットの表示名。 この名前はいつでも変更できます。 |
   | **サブスクリプション** | 該当するサブスクリプション | 使用する Azure サブスクリプションを選択します。 |
   | **リソース グループ** | myResourceGroup | 新しい[リソース グループ](/azure/azure-resource-manager/resource-group-overview#resource-groups)を作成することも、既存のグループを選択することもできます。 |
   | **Location** | 既定の場所 | リソース グループの地理的な場所を選択します。 一覧に示されたどの場所を選択しても構いませんが、多くの場合、顧客に最も近い場所を選択するのが最良です。 ボットの作成後に場所を変更することはできません。 |
   | **[価格レベル]** | F0 | 価格レベルを選択します。 価格レベルはいつでも更新できます。 詳細については、[Bot Service の価格](https://azure.microsoft.com/pricing/details/bot-service/)に関するページをご覧ください。 |
   | **アプリ名** | 一意の名前 | ボットの一意の URL 名。 たとえば、ボットに *myawesomebot* という名前を付けた場合、ボットの URL は `http://myawesomebot.azurewebsites.net` になります。 名前には、英数字とアンダースコアのみを使用する必要があります。 このフィールドは 35 文字に制限されています。 ボットの作成後にアプリ名を変更することはできません。 |
   | **Bot template\(ボット テンプレート\)** | Basic | **C#** または **Node.js** のいずれかを選択し、このクイック スタートでは **Basic** テンプレートを選択して、 **[選択]** をクリックします。 Basic テンプレートでは、エコー ボットが作成されます。 テンプレートの詳細については、[こちら](bot-service-concept-templates.md)をご覧ください。 |
   | **App Service プラン/場所** | ご利用の App Service プラン  | [App Service プラン](https://azure.microsoft.com/pricing/details/app-service/plans/)の場所を選択します。 一覧に示されたどの場所を選択しても構いませんが、多くの場合、顧客に最も近い場所を選択するのが最良です。 (Functions Bot では使用できません。) |
   | **Application Insights** | On | [Application Insights](/bot-framework/bot-service-manage-analytics) を**オン**にするか、**オフ**にするかを決定します。 **[オン]** を選択した場合は、リージョンの場所も指定する必要があります。 一覧に示されたどの場所を選択してもかまいませんが、多くの場合、ボット サービスと同じ場所を選択するのが最良です。 |
   | **Microsoft App ID and password\(Microsoft アプリ ID とパスワード\)** | アプリ ID とパスワードの自動作成 | Microsoft アプリ ID とパスワードを手動で入力する必要がある場合は、このオプションを使用します。 それ以外の場合は、ボット作成プロセスで新しい Microsoft アプリ ID とパスワードが自動的に作成されます。 ボット サービス用に手動でアプリケーションの登録を作成する場合は、サポートされるアカウントの種類が [任意の組織のディレクトリ内のアカウント] または [任意の組織のディレクトリ内のアカウントと、個人用の Microsoft アカウント (Skype、Xbox、Outlook.com など)] に確実に設定されているようにしてください。  |

   > [!NOTE]
   > 
   > **Functions Bot** を作成する場合、 **[App Service プラン/場所]** フィールドは表示されません。 代わりに、 *[ホスティング プラン]* フィールドが表示されます。 その場合は、[ホスティング プラン](bot-service-overview-readme.md#hosting-plans)を選択します。

4. **[作成]** をクリックしてサービスを作成し、ボットをクラウドにデプロイします。 このプロセスには数分かかることがあります。

ボットがデプロイされたことを確認するには、 **[通知]** をオンにします。 通知が、 **[デプロイは進行中です...]** から **[デプロイメントに成功しました]** に変わります。 **[リソースに移動]** ボタンをクリックして、ボットのリソース ブレードを開きます。

 > [!TIP] 
 > パフォーマンス上の理由から、Node.js テンプレートを実行する **Functions Bot** は、*Azure Functions Pack* ツールを使用して既にパッケージ化されています。 *Azure Functions Pack* ツールでは、すべての Node.js モジュールを取得し、それらを 1 つの *.js ファイルにまとめます。
 > 詳細については、「[Azure Functions Pack](https://github.com/Azure/azure-functions-pack)」をご覧ください。
 
## <a name="test-the-bot"></a>ボットのテスト
ボットが作成されたので、[Web チャット](bot-service-manage-test-webchat.md)でテストします。 メッセージを入力すると、ボットが応答します。

## <a name="next-steps"></a>次の手順

このトピックでは、Bot Service を使用して **Basic** Web App Bot/Functions Bot を作成する方法を学習し、Azure 内で組み込みの Web チャット コントロールを使用してボットの機能を検証しました。 次に、ボットを管理し、そのソース コードの操作を開始する方法を確認しましょう。

> [!div class="nextstepaction"]
> [ボットの管理](bot-service-manage-overview.md)

::: moniker-end

::: moniker range="azure-bot-service-4.0"

[!INCLUDE [applies-to-v4](includes/applies-to.md)]

Azure Bot Service は、ボットを開発するための Bot Framework SDK や、ボットをチャンネルに接続するための Bot Service など、ボットを作成するためのコア コンポーネントを提供します。 このトピックでは、.NET テンプレートまたは Node.js テンプレートのいずれかを選択し、Bot Framework SDK v4 を使用してボットを作成できます。

[!INCLUDE [Azure vs local development](~/includes/snippet-quickstart-paths.md)]

## <a name="prerequisites"></a>前提条件

- [Azure](http://portal.azure.com) アカウント

### <a name="create-a-new-bot-service"></a>新しいボット サービスを作成する

1. [Azure Portal](http://portal.azure.com/) にログインします。
1. Azure portal の左上隅にある **[新しいリソースの作成]** リンクをクリックし、 **[AI + Machine Learning]** > **[Web App Bot]** を選択します。 

![ボットを作成する](~/media/azure-bot-quickstarts/abs-create-blade.png)

2. **Web App Bot** に関する情報が含まれた "*新しいブレード*" が開きます。  

3. **[Bot Service]** ブレードで、画像の下の表に示すように、ボットに関する必要な情報を入力します。  <br/>
 ![Web App Bot の作成ブレード](~/media/azure-bot-quickstarts/sdk-create-bot-service-blade.png)

 | Setting | 推奨値 | 説明 |
 | ---- | ---- | ---- |
 | **ボット名** | ボットの表示名 | チャンネルとディレクトリに表示されるボットの表示名。 この名前はいつでも変更できます。 |
 | **サブスクリプション** | 該当するサブスクリプション | 使用する Azure サブスクリプションを選択します。 |
 | **リソース グループ** | myResourceGroup | 新しい[リソース グループ](/azure/azure-resource-manager/resource-group-overview#resource-groups)を作成することも、既存のグループを選択することもできます。 |
 | **Location** | 既定の場所 | リソース グループの地理的な場所を選択します。 一覧に示されたどの場所を選択しても構いませんが、多くの場合、顧客に最も近い場所を選択するのが最良です。 ボットの作成後に場所を変更することはできません。 |
 | **[価格レベル]** | F0 | 価格レベルを選択します。 価格レベルはいつでも更新できます。 詳細については、[Bot Service の価格](https://azure.microsoft.com/pricing/details/bot-service/)に関するページをご覧ください。 |
 | **アプリ名** | 一意の名前 | ボットの一意の URL 名。 たとえば、ボットに *myawesomebot* という名前を付けた場合、ボットの URL は `http://myawesomebot.azurewebsites.net` になります。 名前には、英数字とアンダースコアのみを使用する必要があります。 このフィールドは 35 文字に制限されています。 ボットの作成後にアプリ名を変更することはできません。 |
 | **Bot template\(ボット テンプレート\)** | エコー ボット | **[SDK v4]** を選択します。 このクイック スタートでは C# または Node.js のいずれかを選択して、 **[選択]** をクリックします。  
 | **App Service プラン/場所** | ご利用の App Service プラン  | [App Service プラン](https://azure.microsoft.com/pricing/details/app-service/plans/)の場所を選択します。 一覧に示されたどの場所を選択してもかまいませんが、多くの場合、ボット サービスと同じ場所を選択するのが最良です。 |
 | **Application Insights** | On | [Application Insights](/bot-framework/bot-service-manage-analytics) を**オン**にするか、**オフ**にするかを決定します。 **[オン]** を選択した場合は、リージョンの場所も指定する必要があります。 一覧に示されたどの場所を選択してもかまいませんが、多くの場合、ボット サービスと同じ場所を選択するのが最良です。 |
 | **Microsoft App ID and password\(Microsoft アプリ ID とパスワード\)** | アプリ ID とパスワードの自動作成 | Microsoft アプリ ID とパスワードを手動で入力する必要がある場合は、このオプションを使用します。 それ以外の場合は、ボット作成プロセスで新しい Microsoft アプリ ID とパスワードが自動的に作成されます。 ボット サービス用に手動でアプリケーションの登録を作成する場合は、サポートされるアカウントの種類が [任意の組織のディレクトリ内のアカウント] または [任意の組織のディレクトリ内のアカウントと、個人用の Microsoft アカウント (Skype、Xbox、Outlook.com など)] に確実に設定されているようにしてください。 |

4. **[作成]** をクリックしてサービスを作成し、ボットをクラウドにデプロイします。 このプロセスには数分かかることがあります。

ボットがデプロイされたことを確認するには、 **[通知]** をオンにします。 通知が、 **[デプロイは進行中です...]** から **[デプロイメントに成功しました]** に変わります。 **[リソースに移動]** ボタンをクリックして、ボットのリソース ブレードを開きます。

ボットが作成されたので、Web チャットでテストします。 

## <a name="test-the-bot"></a>ボットのテスト
**[Bot Management]\(ボットの管理\)** セクションで、 **[Test in Web Chat]\(Web チャットでのテスト\)** をクリックします。 Azure Bot Service で Web チャット コントロールを読み込み、ボットに接続します。 

![Azure の Web チャットでのテスト](./media/azure-bot-quickstarts/azure-webchat-test.png)

メッセージを入力すると、ボットが応答します。

## <a name="download-code"></a>コードをダウンロードする
コードをダウンロードして、ローカルで操作することができます。 
1. **[ボット管理]** セクションで **[ビルド]** をクリックします。 
1. 右側のウィンドウで **[ボットのソース コードをダウンロードする]** リンクをクリックします。 
1. 指示に従ってコードをダウンロードし、フォルダーを解凍します。
    1. [!INCLUDE [download keys snippet](~/includes/snippet-abs-key-download.md)]

## <a name="next-steps"></a>次の手順
コードをダウンロードしたら、お使いのコンピューターでボットをローカルで引き続き開発できます。 ボットをテストし、ボット コードを Azure portal にアップロードする準備ができたら、[継続的デプロイの設定](./bot-service-build-continuous-deployment.md)に関するトピックに記載されている手順に従って、変更後にコードを自動更新します。

::: moniker-end
