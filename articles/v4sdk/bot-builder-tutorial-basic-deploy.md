---
title: 基本的なボットの作成とデプロイに関するチュートリアル | Microsoft Docs
description: 基本的なボットを作成し、それを Azure にデプロイする方法について説明します。
keywords: エコー ボット, デプロイ, Azure, チュートリアル
author: Ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 1/9/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 7927ab97dc88657a198c8f1d8e56bcb1ddf0fabe
ms.sourcegitcommit: b2245df2f0a18c5a66a836ab24a573fd70c7d272
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/07/2019
ms.locfileid: "57568239"
---
# <a name="tutorial-create-and-deploy-a-basic-bot"></a>チュートリアル:基本的なボットを作成してデプロイする

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

このチュートリアルでは、Bot Framework SDK を使用して基本的なボットを作成し、Azure にデプロイする手順について説明します。 基本的なボットを既に作成してローカル環境で実行している場合、「[ボットをデプロイする](#deploy-your-bot)」セクションに進んでください。

このチュートリアルでは、以下の内容を学習します。

> [!div class="checklist"]
> * 基本的なエコー ボットを作成する
> * それをローカル環境で実行して操作する
> * ボットを公開する

Azure サブスクリプションがない場合は、開始する前に[無料アカウント](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)を作成してください。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

[!INCLUDE [dotnet quickstart](~/includes/quickstart-dotnet.md)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

[!INCLUDE [javascript quickstart](~/includes/quickstart-javascript.md)]

---

## <a name="deploy-your-bot"></a>ボットをデプロイする

それでは、ボットを Azure にデプロイします。

### <a name="prerequisites"></a>前提条件

[!INCLUDE [prerequisite snippet](~/includes/deploy/snippet-prerequisite.md)]

### <a name="login-to-azure-cli-and-set-your-subscription"></a>Azure CLI にログインしてサブスクリプションを設定する

ボットの作成とローカルでのテストが完了したので、次は Azure にデプロイします。

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

[!INCLUDE [download keys snippet](~/includes/snippet-abs-key-download.md)]

### <a name="decrypt-the-downloaded-bot-file-and-use-in-your-project"></a>ダウンロードした .bot ファイルの暗号化を解除してプロジェクトで使用する

.bot ファイル内の機密情報は暗号化されているので、簡単に使用できるように暗号化を解除しましょう。 

最初に、ボットをダウンロードしたディレクトリに移動します。

[!INCLUDE [decrypt bot snippet](~/includes/deploy/snippet-decrypt-bot.md)]

### <a name="test-your-bot-locally"></a>ボットをローカルでテストする

この時点で、このボットは以前の `.bot` ファイルと同じように動作するはずです。 新しい `.bot` ファイルを使用して、想定どおりに機能することを確認します。

エミュレーターに "*運用*" エンドポイントが表示されるようになったはずです。 そうならない場合は、おそらくまだ古い `.bot` ファイルを使用しています。

### <a name="publish-your-bot-to-azure"></a>ボットを Azure に公開する

<!-- TODO: re-encrypt your .bot file? -->

[!INCLUDE [publish snippet](~/includes/deploy/snippet-publish.md)]

<!-- TODO: If we tell them to re-encrypt, this step is not necessary. -->

[!INCLUDE [clear encryption snippet](~/includes/deploy/snippet-clear-encryption.md)]

これで、ボットを Web チャットでテストできるようになります。

## <a name="additional-resources"></a>その他のリソース

[!INCLUDE [additional resources snippet](~/includes/deploy/snippet-additional-resources.md)]

## <a name="next-steps"></a>次の手順
> [!div class="nextstepaction"]
> [サービスをボットに追加する](bot-builder-tutorial-add-qna.md)

