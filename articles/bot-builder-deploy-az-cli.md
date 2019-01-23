---
title: Azure CLI を使用してボットをデプロイする | Microsoft Docs
description: 使用するボットを Azure クラウドにデプロイします。
keywords: ボットのデプロイ, azure のデプロイ, ボットの発行, ボットの az デプロイ, ボットの visual studio デプロイ, msbot 発行, msbot 複製
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: get-started-article
ms.service: bot-service
ms.subservice: abs
ms.date: 01/07/2019
ms.openlocfilehash: 3ebc13cf9e2d111d716d081c36f125d28a441811
ms.sourcegitcommit: bdb981c0b11ee99d128e30ae0462705b2dae8572
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/17/2019
ms.locfileid: "54360742"
---
# <a name="deploy-your-bot-using-azure-cli"></a>Azure CLI を使用してボットをデプロイする

[!INCLUDE [pre-release-label](./includes/pre-release-label.md)]

ご自分のボットを作成し、ローカルでテストした後に、それを Azure にデプロイして、どこからでもアクセスできるようにすることができます。 ボットを Azure にデプロイするには、使用するサービスの料金を支払う必要があります。 [課金とコスト管理](https://docs.microsoft.com/en-us/azure/billing/)に関する記事で、Azure の課金の確認、使用量とコストの監視、アカウントとサブスクリプションの管理の方法について説明されています。

この記事では、`az` および `msbot` CLI を使用して Azure に C# ボットと JavaScript ボットをデプロイする方法を紹介します。 ボットのデプロイに必要なものを完全に理解するために、手順を実行する前にこの記事を確認することをお勧めします。

## <a name="prerequisites"></a>前提条件

[!INCLUDE [prerequisite snippet](~/includes/deploy/snippet-prerequisite.md)]


## <a name="deploy-javascript-and-c-bots-using-az-cli"></a>az cli を使用して JavaScript ボットと C# ボットをデプロイする

ボットの作成とローカルでのテストが完了したので、次は Azure にデプロイします。 これらの手順は、必要な Azure リソースを作成済みであることを前提としています。

[!INCLUDE [az login snippet](~/includes/deploy/snippet-az-login.md)]

### <a name="create-a-web-app-bot"></a>Web アプリ ボットを作成する

ボットを公開するリソース グループがまだ存在しない場合は、作成します。

[!INCLUDE [az create group snippet](~/includes/deploy/snippet-az-create-group.md)]

[!INCLUDE [az create web app snippet](~/includes/deploy/snippet-create-web-app.md)]

続行する前に、Azure へのログインに使用する電子メール アカウントの種類に基づいて、該当する手順をお読みください。

#### <a name="msa-email-account"></a>MSA 電子メール アカウント

[MSA](https://en.wikipedia.org/wiki/Microsoft_account) 電子メール アカウントを使用している場合は、`az bot create` コマンドを使って、使用するアプリケーション登録ポータルにアプリ ID とアプリ パスワードを作成する必要があります。

[!INCLUDE [create bot msa snippet](~/includes/deploy/snippet-create-bot-msa.md)]

#### <a name="business-or-school-account"></a>職場または学校のアカウント

[!INCLUDE [create bot snippet](~/includes/deploy/snippet-create-bot.md)]

### <a name="download-the-bot-from-azure"></a>Azure からボットをダウンロードする

次に、作成したボットをダウンロードします。 
[!INCLUDE [download bot snippet](~/includes/deploy/snippet-download-bot.md)]

### <a name="decrypt-the-downloaded-bot-file-and-use-in-your-project"></a>ダウンロードした .bot ファイルの暗号化を解除してプロジェクトで使用する

.bot ファイル内の機密情報は暗号化されています。

[!INCLUDE [decrypt bot snippet](~/includes/deploy/snippet-decrypt-bot.md)]

### <a name="update-the-bot-file"></a>.bot ファイルを更新する

ボットで LUIS、QnA Maker、または Dispatch サービスを使用している場合は、それらへの参照を .bot ファイルに追加する必要があります。 それ以外の場合は、この手順を省略できます。

1. 新しい .bot ファイルを使用して、BotFramework エミュレーターでボットを開きます。 ボットがローカルで実行されている必要はありません。
1. **[BOT EXPLORER]\(ボット エクスプローラー\)** パネルで **[SERVICES]\(サービス\)** セクションを展開します。
1. LUIS アプリへの参照を追加するには、**[SERVICES]\(サービス\)** の右側にあるプラス記号 (+) をクリックします。
   1. **[Add Language Understanding (LUIS)]\(Language Understanding (LUIS) の追加\)** を選択します。
   1. Azure アカウントへのログインを求められたら、ログインします。
   1. アクセス権を持つ LUIS アプリケーションの一覧が表示されます。 ボット用にいずれかを選択します。
1. QnA Maker ナレッジ ベースへの参照を追加するには、**[SERVICES]\(サービス\)** の右側にあるプラス記号 (+) をクリックします。
   1. **[Add QnA Maker]\(QnA Maker の追加\)** を選択します。
   1. Azure アカウントへのログインを求められたら、ログインします。
   1. アクセス権を持つナレッジ ベースの一覧が表示されます。 ボット用にいずれかを選択します。
1. Dispatch モデルへの参照を追加するには、**[SERVICES]\(サービス\)** の右側にあるプラス記号 (+) をクリックします。
   1. **[Add Dispatch]\(Dispatch の追加\)** を選択します。
   1. Azure アカウントへのログインを求められたら、ログインします。
   1. アクセス権を持つ Dispatch モデルの一覧が表示されます。 ボット用にいずれかを選択します。

### <a name="test-your-bot-locally"></a>ボットをローカルでテストする

この時点で、このボットは以前の .bot ファイルと同じように動作するはずです。 新しい .bot ファイルを使用して、想定どおりに機能することを確認してください。

### <a name="publish-your-bot-to-azure"></a>ボットを Azure に公開する

<!-- TODO: re-encrypt your .bot file? -->

[!INCLUDE [publish snippet](~/includes/deploy/snippet-publish.md)]

<!-- TODO: If we tell them to re-encrypt, this step is not necessary. -->

[!INCLUDE [clear encryption snippet](~/includes/deploy/snippet-clear-encryption.md)]

## <a name="additional-resources"></a>その他のリソース

[!INCLUDE [additional resources snippet](~/includes/deploy/snippet-additional-resources.md)]

## <a name="next-steps"></a>次の手順
> [!div class="nextstepaction"]
> [継続的デプロイを設定する](bot-service-build-continuous-deployment.md)
