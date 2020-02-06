---
title: Azure Bot Service を使用してボットを作成する - Bot Service
description: 統合された専用のボット開発環境である Bot Service を使用してボットを作成する方法について説明します。
keywords: クイック スタート, ボットの作成, bot service, web app bot
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 01/09/2020
ms.openlocfilehash: 9190c2eefbfe290ea43531630473772addb1b60a
ms.sourcegitcommit: f8b5cc509a6351d3aae89bc146eaabead973de97
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/09/2020
ms.locfileid: "75798889"
---
# <a name="create-a-bot-with-azure-bot-service"></a>Azure Bot Service を使用してボットを作成する

[!INCLUDE [applies-to-v4](../includes/applies-to.md)]

Azure Bot Service は、ボットを開発するための Bot Framework SDK や、ボットをチャンネルに接続するための Bot Service など、ボットを作成するためのコア コンポーネントを提供します。 このトピックでは、.NET テンプレートまたは Node.js テンプレートのいずれかを選択し、Bot Framework SDK v4 を使用してボットを作成できます。

>[!NOTE] 
> 作成したボットは、Azure Bot Service に自動的に登録されます。 既に他の場所でホストされているボットを登録する場合は、次の記事を参照してください:[Azure Bot Service へのボットの登録](../bot-service-quickstart-registration.md)。

[!INCLUDE [Azure vs local development](~/includes/snippet-quickstart-paths.md)]

## <a name="prerequisites"></a>前提条件

- [Azure](https://portal.azure.com) アカウント

### <a name="create-a-new-bot-service"></a>新しいボット サービスを作成する

1. [Azure Portal](https://portal.azure.com/) にログインします。
1. Azure portal の左上隅にある **[新しいリソースの作成]** リンクをクリックし、 **[AI + Machine Learning]**  >  **[Web App Bot]\(Web アプリ ボット\)** を選択します。 

![ボットを作成する](../media/azure-bot-quickstarts/abs-create-blade.png)

2. **Web App Bot** に関する情報が含まれた "*新しいブレード*" が開きます。  

3. **[Bot Service]** ブレードで、画像の下の表に示すように、ボットに関する必要な情報を入力します。  <br/>
 ![Web App Bot の作成ブレード](../media/azure-bot-quickstarts/sdk-create-bot-service-blade.png)

 | 設定 | 推奨値 | [説明] |
 | ---- | ---- | ---- |
 | **ボット名** | ボットの表示名 | チャンネルとディレクトリに表示されるボットの表示名。 この名前はいつでも変更できます。 |
 | **サブスクリプション** | 該当するサブスクリプション | 使用する Azure サブスクリプションを選択します。 |
 | **リソース グループ** | myResourceGroup | 新しい[リソース グループ](/azure/azure-resource-manager/resource-group-overview#resource-groups)を作成することも、既存のグループを選択することもできます。 |
 | **地域** | 既定の場所 | リソース グループの地理的な場所を選択します。 一覧に示されたどの場所を選択しても構いませんが、多くの場合、顧客に最も近い場所を選択するのが最良です。 ボットの作成後に場所を変更することはできません。 |
 | **価格レベル** | F0 | 価格レベルを選択します。 価格レベルはいつでも更新できます。 詳細については、[Bot Service の価格](https://azure.microsoft.com/pricing/details/bot-service/)に関するページをご覧ください。 |
 | **アプリ名** | 一意の名前 | ボットの一意の URL 名。 たとえば、ボットに *myawesomebot* という名前を付けた場合、ボットの URL は `http://myawesomebot.azurewebsites.net` になります。 名前には、英数字とアンダースコアのみを使用する必要があります。 このフィールドは 35 文字に制限されています。 ボットの作成後にアプリ名を変更することはできません。 |
 | **ボット テンプレート** | エコー ボット | **[SDK v4]** を選択します。 このクイック スタートでは C# または Node.js のいずれかを選択して、 **[選択]** をクリックします。  
 | **App Service プラン/場所** | ご利用の App Service プラン  | [App Service プラン](https://azure.microsoft.com/pricing/details/app-service/plans/)の場所を選択します。 一覧に示されたどの場所を選択してもかまいませんが、多くの場合、ボット サービスと同じ場所を選択するのが最良です。 |
 | **LUIS アカウント** "_Basic Bot テンプレートでのみ使用可能_" | LUIS Azure リソース名 | [LUIS リソースを Azure リソースに移行](https://docs.microsoft.com/azure/cognitive-services/luis/luis-migration-authoring)した後、Azure リソース名を入力して、この LUIS アプリケーションをその Azure リソースに関連付けます。 
 | **Application Insights** | On | [Application Insights](/bot-framework/bot-service-manage-analytics) を**オン**にするか、**オフ**にするかを決定します。 **[オン]** を選択した場合は、リージョンの場所も指定する必要があります。 一覧に示されたどの場所を選択してもかまいませんが、多くの場合、ボット サービスと同じ場所を選択するのが最良です。 |
 | **Microsoft App ID and password\(Microsoft アプリ ID とパスワード\)** | アプリ ID とパスワードの自動作成 | Microsoft アプリ ID とパスワードを手動で入力する必要がある場合は、このオプションを使用します。 それ以外の場合は、ボット作成プロセスで新しい Microsoft アプリ ID とパスワードが自動的に作成されます。 ボット サービス用に手動でアプリケーションの登録を作成する場合は、サポートされるアカウントの種類が [任意の組織のディレクトリ内のアカウント] または [任意の組織のディレクトリ内のアカウントと、個人用の Microsoft アカウント (Skype、Xbox、Outlook.com など)] に確実に設定されているようにしてください。 |

4. **[作成]** をクリックしてサービスを作成し、ボットをクラウドにデプロイします。 このプロセスには数分かかることがあります。

ボットがデプロイされたことを確認するには、 **[通知]** をオンにします。 通知が、 **[デプロイは進行中です...]** から **[デプロイメントに成功しました]** に変わります。 **[リソースに移動]** ボタンをクリックして、ボットのリソース ブレードを開きます。

ボットが作成されたので、Web チャットでテストします。

## <a name="test-the-bot"></a>ボットのテスト
**[Bot Management]\(ボットの管理\)** セクションで、 **[Test in Web Chat]\(Web チャットでのテスト\)** をクリックします。 Azure Bot Service で Web チャット コントロールを読み込み、ボットに接続します。 

![Azure の Web チャットでのテスト](../media/azure-bot-quickstarts/azure-webchat-test.png)

メッセージを入力すると、ボットが応答します。

## <a name="manual-app-registration"></a>アプリの手動登録

次のような状況では、手動で登録する必要があります。

- 自分の組織で登録できず、構築しているボット用のアプリ ID を作成するために別のパーティーが必要である。
- 独自のアプリ ID (とパスワード) を手動で作成する必要がある。

[FAQ の「アプリの登録」](../bot-service-resources-bot-framework-faq.md#app-registration)をご覧ください。


## <a name="download-code"></a>コードをダウンロードする
コードをダウンロードして、ローカルで操作することができます。 
1. **[ボット管理]** セクションで **[ビルド]** をクリックします。 
1. 右側のウィンドウで **[ボットのソース コードをダウンロードする]** リンクをクリックします。 
1. 指示に従ってコードをダウンロードし、フォルダーを解凍します。
    1. [!INCLUDE [download keys snippet](../includes/snippet-abs-key-download.md)]

## <a name="next-steps"></a>次のステップ
コードをダウンロードしたら、お使いのコンピューターでボットをローカルで引き続き開発できます。 ボットをテストし、ボット コードを Azure portal にアップロードする準備ができたら、[継続的デプロイの設定](../bot-service-build-continuous-deployment.md)に関するトピックに記載されている手順に従って、変更後にコードを自動更新します。
