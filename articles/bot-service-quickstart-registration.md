---
title: Bot Service を使用して Bot Channels Registration を作成する | Microsoft Docs
description: 既存のボットを Bot Service に登録する方法について説明します。
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 6a32bc5712937c615962e4f6edfc7ea691d3ec39
ms.sourcegitcommit: 67445b42796d90661afc643c6bb6533e9a662cbc
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 08/06/2018
ms.locfileid: "39574978"
---
# <a name="register-a-bot-with-bot-service"></a>ボットを Bot Service に登録する

[!INCLUDE [pre-release-label](includes/pre-release-label-v3.md)]

他の場所でホストされているボットが既にあり、Bot Service を使用して他のチャンネルに接続する場合は、ボットを Bot Service に登録する必要があります。 このトピックでは、**Bot Channels Registration** ボット サービスを作成して、ボットを Bot Service に登録する方法について説明します。

> [!IMPORTANT] 
> ボットを登録する必要があるのは、ボットが Azure でホストされていない場合だけです。 Azure portal を使用して[ボットを作成](bot-service-quickstart.md)した場合、ボットは Bot Service に既に登録されています。

## <a name="log-in-to-azure"></a>Azure にログインする
[Azure Portal](http://portal.azure.com) にログインします。

> [!TIP]
> サブスクリプションがない場合は、<a href="https://azure.microsoft.com/en-us/free/" target="_blank">無料アカウント</a>に登録できます。

## <a name="create-a-bot-channels-registration"></a>Bot Channels Registration を作成する
Bot Service 機能を使用できるようにするには、**Bot Channels Registration** ボット サービスが必要です。 登録ボットを使用すると、ボットをチャンネルに接続できます。

**Bot Channels Registration** を作成するには、次の手順を実行します。

1. Azure portal の左上隅にある **[新規]** ボタンをクリックし、**[AI + Cognitive Services] > [Bot Channels Registration]** を選択します。 

2. **Bot Channels Registration** に関する情報が含まれた新しいブレードが開きます。 **[作成]** をクリックして作成プロセスを開始します。 

3. **[Bot Service]** ブレードで、画像の下の表に示すように、ボットに関する必要な情報を入力します。  <br/>
   ![登録ボットの作成ブレード](~/media/azure-bot-quickstarts/registration-create-bot-service-blade.png)


   |                    Setting                     |         推奨値         |                                                                                                  説明                                                                                                  |
   |------------------------------------------------|---------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
   |           <strong>ボット名</strong>            |     ボットの表示名     |                                                  チャンネルとディレクトリに表示されるボットの表示名。 この名前はいつでも変更できます。                                                  |
   |         <strong>サブスクリプション</strong>          |        該当するサブスクリプション        |                                                                                使用する Azure サブスクリプションを選択します。                                                                                 |
   |        <strong>リソース グループ</strong>         |         myResourceGroup         |                                 新しい[リソース グループ](/azure/azure-resource-manager/resource-group-overview#resource-groups)を作成することも、既存のグループを選択することもできます。                                  |
   |                    Location                    |             米国西部             |                                                        ボットがデプロイされている場所の近く、またはボットがアクセスする他のサービスの近くの場所を選択します。                                                         |
   |         <strong>価格レベル</strong>          |               F0                |             価格レベルを選択します。 価格レベルはいつでも更新できます。 詳細については、[Bot Service の価格](https://azure.microsoft.com/en-us/pricing/details/bot-service/)に関するページをご覧ください。              |
   |      <strong>[Messaging endpoint]\(メッセージング エンドポイント\)</strong>       |               URL               |                                                                               ボットのメッセージング エンドポイントの URL を入力します。                                                                                |
   |     <strong>Application Insights</strong>      |               On                | [Application Insights](bot-service-manage-analytics.md) を<strong>オン</strong>にするか、<strong>オフ</strong>にするかを決定します。 <strong>[オン]</strong> を選択した場合は、リージョンの場所も指定する必要があります。 |
   | <strong>Microsoft App ID and password\(Microsoft アプリ ID とパスワード\)</strong> | アプリ ID とパスワードの自動作成 |              Microsoft アプリ ID とパスワードを手動で入力する必要がある場合は、このオプションを使用します。 それ以外の場合は、ボット作成プロセスで新しい Microsoft アプリ ID とパスワードが自動的に作成されます。               |


4. **[作成]** をクリックしてサービスを作成し、ボットのメッセージング エンドポイントを登録します。

**[通知]** をチェックして、登録が作成されたことを確認します。 通知が、**[デプロイは進行中です...]** から **[デプロイメントに成功しました]** に変わります。 **[リソースに移動]** ボタンをクリックして、ボットのリソース ブレードを開きます。 

## <a name="bot-channels-registration-password"></a>Bot Channels Registration のパスワード

**Bot Channels Registration** ボット サービスには、アプリ サービスは関連付けられていません。 そのため、このボット サービスには *MicrosoftAppID* しかありません。 パスワードは手動で生成し、自分で保存する必要があります。 このパスワードは、[エミュレーター](bot-service-debug-emulator.md)を使用してボットをテストする場合に必要になります。

MicrosoftAppPassword を生成するには、次の手順を実行します。

1. **[設定]** ブレードで **[管理]** をクリックします。 これは、**[Microsoft アプリ ID]** の横に表示されているリンクです。 このリンクにより、新しいパスワードを生成できるウィンドウが開きます。 <br/>
  ![[設定] ブレードの [管理] リンク](~/media/azure-bot-quickstarts/registration-settings-manage-link.png)

2. **[新しいパスワードを生成]** をクリックします。 これにより、ボットの新しいパスワードが生成されます。 このパスワードをコピーしてファイルに保存します。 このパスワードが表示されるのはこのときだけです。 完全なパスワードを保存していなかった場合、後で必要になったら、このプロセスを繰り返して新しいパスワードを作成する必要があります。 <br/>
  ![Microsoft アプリ パスワードを生成する](~/media/azure-bot-quickstarts/registration-generate-app-password.png)

## <a name="update-the-bot"></a>ボットを更新する

Bot Builder SDK for Node.js を使用している場合は、次の環境変数を設定します。

* MICROSOFT_APP_ID
* MICROSOFT_APP_PASSWORD

Bot Builder SDK for .NET を使用している場合は、web.config ファイルに次のキー値を設定します。

* MicrosoftAppId
* MicrosoftAppPassword

## <a name="test-the-bot"></a>ボットのテスト

ボット サービスが作成されたので、[Web チャット](bot-service-manage-test-webchat.md)でテストします。 メッセージを入力すると、ボットが応答します。

## <a name="next-steps"></a>次の手順

このトピックでは、ホストされたボットを Bot Service に登録する方法を学びました。 次に、Bot Service を管理する方法を確認しましょう。

> [!div class="nextstepaction"]
> [ボットの管理](bot-service-manage-overview.md)

