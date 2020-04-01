---
title: 基本的なボットの作成とデプロイに関するチュートリアル - Bot Service
description: 基本的なボットを作成し、それを Azure にデプロイする方法について説明します。
keywords: エコー ボット, デプロイ, Azure, チュートリアル
author: Ivorb
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 03/23/2020
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 3b4e3652f7799fb09219b8536bd98a78e0952223
ms.sourcegitcommit: 126c4f8f8c7a3581e7521dc3af9a937493e6b1df
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/01/2020
ms.locfileid: "80499908"
---
# <a name="tutorial-create-and-deploy-a-basic-bot"></a>チュートリアル:基本的なボットを作成してデプロイする

[!INCLUDE [applies-to-v4](../includes/applies-to.md)]

このチュートリアルでは、Bot Framework SDK を使用して基本的なボットを作成し、Azure にデプロイする手順について説明します。 基本的なボットを既に作成してローカル環境で実行している場合、「[ボットをデプロイする](#deploy-your-bot)」セクションに進んでください。

このチュートリアルでは、以下の内容を学習します。

> [!div class="checklist"]
> * 基本的なエコー ボットを作成する
> * それをローカル環境で実行して操作する
> * ボットを公開する

Azure サブスクリプションをお持ちでない場合は、開始する前に [無料アカウント](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) を作成してください。

# <a name="c"></a>[C#](#tab/csharp)

[!INCLUDE [dotnet quickstart](~/includes/quickstart-dotnet.md)]

# <a name="javascript"></a>[JavaScript](#tab/javascript)

[!INCLUDE [javascript quickstart](~/includes/quickstart-javascript.md)]

# <a name="python"></a>[Python](#tab/python)

[!INCLUDE [python quickstart](~/includes/quickstart-python.md)]

---

## <a name="deploy-your-bot"></a>ボットをデプロイする

### <a name="prerequisites"></a>前提条件

[!INCLUDE [deploy prerequisite](~/includes/deploy/snippet-prerequisite.md)]

### <a name="prepare-for-deployment"></a>展開を準備する

> [!TIP]
> この手順では ZIP ファイルを使用してボットをデプロイします。 C# では、ビルド時のソリューション構成が**デバッグ**に設定されていると、この手順が失敗する場合があります。
> 続行する前に、Visual Studio でソリューション構成が**リリース**に設定されていることを確認し、ソリューションのクリーン リビルドを実行してください。

[!INCLUDE [deploy prepare intro](~/includes/deploy/snippet-prepare-deploy-intro.md)]

#### <a name="1-login-to-azure"></a>1.Azure にログインする

[!INCLUDE [deploy az login](~/includes/deploy/snippet-az-login.md)]

#### <a name="2-set-the-subscription"></a>2.サブスクリプションを設定する

[!INCLUDE [deploy az subscription](~/includes/deploy/snippet-az-set-subscription.md)]

#### <a name="3-create-an-app-registration"></a>3.アプリ登録を作成する

[!INCLUDE [deploy create app registration](~/includes/deploy/snippet-create-app-registration.md)]

#### <a name="4-deploy-via-arm-template"></a>4.ARM テンプレートを使用してデプロイする

ご自身のボットは、新しいリソース グループまたは既存のリソース グループにデプロイできます。 自分にとって最適なオプションを選択してください。

> [!NOTE]
> Python ボットは、Windows サービス/ボットを含むリソース グループにはデプロイできません。  複数の Python ボットを同じリソース グループにデプロイできますが、別のリソース グループに他のサービス (LUIS、QnA など) を作成します。
>

##### <a name="deploy-via-arm-template-with-new-resource-group"></a>**新しいリソース グループでの ARM テンプレートを使用したデプロイ**

[!INCLUDE [ARM with new resourece group](~/includes/deploy/snippet-ARM-new-resource-group.md)]

##### <a name="deploy-via-arm-template-with-existing-resource-group"></a>**既存のリソース グループでの ARM テンプレートを使用したデプロイ**

[!INCLUDE [ARM with existing resourece group](~/includes/deploy/snippet-ARM-existing-resource-group.md)]

#### <a name="5-prepare-your-code-for-deployment"></a>5.デプロイ用のコードを準備する

[!INCLUDE [Work around for .NET Core 3.1 SDK](~/includes/deploy/samples-workaround-3-1.md)]

##### <a name="retrieve-or-create-necessary-iiskudu-files"></a>**必要な IIS/Kudu ファイルを取得または作成する**

[!INCLUDE [retrieve or create IIS/Kudu files](~/includes/deploy/snippet-IIS-Kudu-files.md)]

##### <a name="zip-up-the-code-directory-manually"></a>**コード ディレクトリを手動で zip 圧縮する**

[!INCLUDE [zip up code](~/includes/deploy/snippet-zip-code.md)]

### <a name="deploy-code-to-azure"></a>コードを Azure にデプロイする

[!INCLUDE [deploy code to Azure](~/includes/deploy/snippet-deploy-code-to-az.md)]

### <a name="test-in-web-chat"></a>Web チャットでのテスト

[!INCLUDE [test in web chat](~/includes/deploy/snippet-test-in-web-chat.md)]

## <a name="additional-resources"></a>その他のリソース

[!INCLUDE [additional resources snippet](~/includes/deploy/snippet-additional-resources.md)]

## <a name="next-steps"></a>次のステップ

> [!div class="nextstepaction"]
> [ボットで QnA Maker を使用して質問に回答する](bot-builder-tutorial-add-qna.md)