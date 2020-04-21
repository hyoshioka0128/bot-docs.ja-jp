---
title: Bot Service を使用して Bot Channels Registration を作成する - Bot Service
description: 既存のボットを Bot Service に登録する方法について説明します。
ms.author: kamrani
manager: kamrani
ms.topic: conceptual
ms.service: bot-service
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: b60d8a08650de3e4f7501173acb9f1cbbf06e43d
ms.sourcegitcommit: 9d77f3aff9521d819e88efd0fbd19d469b9919e7
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/16/2020
ms.locfileid: "80647268"
---
# <a name="register-a-bot-with-azure-bot-service"></a>ボットを Azure Bot Service に登録する

このトピックでは、**Azure Bot Service** リソースを作成してボットを登録する方法について説明します。 これは、他の場所でホストされているボットを Azure で使用できるようにし、Azure Bot Service チャネルに接続する場合に必要です。

これにより、ユーザーがどこにいても、Cortana、Messenger、およびその他の多数のサービスを通じてそのユーザーと対話するボットを作成、接続、および管理できます。

> [!IMPORTANT] 
> ボットを登録する必要があるのは、ボットが Azure でホストされていない場合だけです。 Azure portal を使用して[ボットを作成](v4sdk/abs-quickstart.md)した場合、ボットはこのサービスに既に登録されています。

## <a name="create-a-registration-resource"></a>登録リソースを作成する

1. ブラウザーで [Azure portal](https://ms.portal.azure.com) に移動します。

    > [!TIP]
    > サブスクリプションがない場合は、<a href="https://azure.microsoft.com/free/" target="_blank">無料アカウント</a>に登録できます。

1. 左側のウィンドウで、 **[リソースの作成]** をクリックします。
1. 右側のウィンドウの選択ボックスに、「*bot*」と入力します。 ドロップダウン リストで、ご使用のアプリケーションに基づいて **[Bot Channels Registration]\(ボット チャネル登録\)** または **[Web App Bot]\(Web アプリ ボット\)** を選択します。
**Web ボット アプリ**の場合、「[Azure Bot Service を使用してボットを作成する](v4sdk/abs-quickstart.md)」の記事で説明されている手順に従います。 Azure Bot Service に自動的に登録されるボットを Azure で作成します。
1. **[作成]** ボタンをクリックして、プロセスを開始します。
1. **[Bot Service]** ブレードで、画像の下の表に示すように、ボットに関する必要な情報を入力します。  

   ![登録ボットの作成ブレード](media/azure-bot-quickstarts/registration-create-bot-service-blade.png)

   |設定 |推奨値|説明|
   |---|---|--|
   |**ボット名** <img width="300px">|ボットの表示名|チャンネルとディレクトリに表示されるボットの表示名。 この名前はいつでも変更できます。|
   |**サブスクリプション**|該当するサブスクリプション|使用する Azure サブスクリプションを選択します。|
   |**リソース グループ**|myResourceGroup|新しい[リソース グループ](/azure/azure-resource-manager/resource-group-overview#resource-groups)を作成することも、既存のグループを選択することもできます。|
   |**場所**|米国西部|ボットがデプロイされている場所の近く、またはボットがアクセスする他のサービスの近くの場所を選択します。|
   |**価格レベル**|F0|価格レベルを選択します。 価格レベルはいつでも更新できます。 詳細については、[Bot Service の価格](https://azure.microsoft.com/pricing/details/bot-service/)に関するページをご覧ください。|
   |**[Messaging endpoint]\(メッセージング エンドポイント\)**|URL|ボットのメッセージング エンドポイントの URL を入力します。|
   |**Application Insights**|On| [Application Insights](bot-service-manage-analytics.md) を**オン**にするか、**オフ**にするかを決定します。 **[オン]** を選択した場合は、リージョンの場所も指定する必要があります。 |
   |**Microsoft App ID and password\(Microsoft アプリ ID とパスワード\)**| アプリ ID とパスワードの自動作成 |Microsoft アプリ ID とパスワードを手動で入力する必要がある場合は、このオプションを使用します。 次のセクションの「[アプリの手動登録](#manual-app-registration)」を参照してください。 それ以外の場合、登録プロセスで新しい Microsoft アプリ ID とパスワードが作成されます。 |

    > [!IMPORTANT]
    > ボットのメッセージング エンドポイントの URL は忘れずに入力してください。

1. **[作成]** ボタンをクリックします。 リソースが作成されるまで待ちます。 リソースがリソースの一覧に表示されます。

### <a name="get-registration-password"></a>登録パスワードを取得する

登録が完了すると、Azure Active Directory によって一意のアプリケーション ID が登録に割り当てられ、アプリケーションの *[概要]* ページが表示されます。

パスワードを取得するには、次の手順に従います。

1. リソースの一覧で、作成した Azure App Service リソースをクリックします。
1. 右側のウィンドウのリソース ブレードで **[設定]** をクリックします。 リソースの *[設定]* ページが表示されます。
1. [設定] ページで、生成された **Microsoft アプリ ID** をコピーし、ファイルに保存します。
1. *Microsoft アプリ ID* の **[管理]** リンクをクリックします。

    ![登録ボットの作成ブレード](media/azure-bot-quickstarts/bot-channels-registration-app-settings.png)

1. 表示される *[Certificates & secrets]\(証明書とシークレット\)* ページで、 **[新しいクライアント シークレット]** をクリックします。

    ![登録ボットの作成ブレード](media/azure-bot-quickstarts/bot-channels-registration-app-secrets.png)

1. 説明を追加し、有効期限を選択して、 **[追加]** をクリックします。

    ![登録ボットの作成ブレード](media/azure-bot-quickstarts/bot-channels-registration-app-secrets-create.png)

    これにより、ボットの新しいパスワードが生成されます。 このパスワードをコピーしてファイルに保存します。 このパスワードが表示されるのはこのときだけです。 完全なパスワードを保存していなかった場合、後で必要になったら、このプロセスを繰り返して新しいパスワードを作成する必要があります。

## <a name="manual-app-registration"></a>アプリの手動登録

次のような状況では、手動で登録する必要があります。

- 自分の組織で登録できず、構築しているボット用のアプリ ID を作成するために別のパーティーが必要である。
- 独自のアプリ ID (とパスワード) を手動で作成する必要がある。

[FAQ の「アプリの登録」](bot-service-resources-bot-framework-faq.md#app-registration)をご覧ください。

> [!IMPORTANT]
> *[Supports account types]\(アカウントの種類のサポート\)* セクションで、2 つのマルチテナント タイプのいずれかを選択する必要があります。つまり、アプリ作成時に、 *[任意の組織ディレクトリ内のアカウント (任意の Azure AD ディレクトリ - マルチテナント)]* または *[任意の組織ディレクトリ内のアカウント (任意の Azure AD - マルチテナント) と個人の Microsoft アカウント (Xbox、Outlook.com など)]* を選択します。そうしないと、ボットは動作しません。 詳細については、「[Azure portal を使用した新規アプリケーションの登録](https://docs.microsoft.com/azure/active-directory/develop/quickstart-register-app#register-a-new-application-using-the-azure-portal)」を参照してください。

## <a name="update-the-bot"></a>ボットを更新する

Bot Framework SDK for .NET を使用している場合は、web.config ファイルに次のキー値を設定します。

- `MicrosoftAppId = <appId>`
- `MicrosoftAppPassword = <appSecret>`

Bot Framework SDK for Node.js を使用している場合は、次の環境変数を設定します。

- `MICROSOFT_APP_ID = <appId>`
- `MICROSOFT_APP_PASSWORD = <appSecret>`

## <a name="test-the-bot"></a>ボットのテスト

ボット サービスが作成されたので、[Web チャット](bot-service-manage-test-webchat.md)でテストします。 メッセージを入力すると、ボットが応答します。

## <a name="next-steps"></a>次のステップ

このトピックでは、ホストされたボットを Bot Service に登録する方法を学びました。 次に、Bot Service を管理する方法を確認しましょう。

> [!div class="nextstepaction"]
> [ボットの管理](bot-service-manage-overview.md)
