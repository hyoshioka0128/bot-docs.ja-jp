---
title: ボットをデプロイする - Bot Service
description: 使用するボットを Azure クラウドにデプロイする
keywords: ボットのデプロイ, azure へのボットのデプロイ, ボットの発行
author: ivorb
ms.author: kamrani
manager: kamrani
ms.topic: conceptual
ms.service: bot-service
ms.date: 03/23/2020
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 9e4c67f644e9797f3e210546a91d09f5161aa100
ms.sourcegitcommit: 9d77f3aff9521d819e88efd0fbd19d469b9919e7
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/16/2020
ms.locfileid: "80499822"
---
# <a name="deploy-your-bot"></a>ボットをデプロイする

[!INCLUDE [applies-to](./includes/applies-to.md)]

この記事では、基本ボットを Azure にデプロイする方法を紹介します。 デプロイ用にボットを準備する方法、ボットを Azure にデプロイする方法、および Web チャットでボットをテストする方法について説明します。 ボットのデプロイに必要なものを完全に理解するために、手順を実行する前にこの記事を確認することをお勧めします。

## <a name="prerequisites"></a>前提条件

[!INCLUDE [deploy prerequisite](~/includes/deploy/snippet-prerequisite.md)]

## <a name="prepare-for-deployment"></a>展開を準備する

[!INCLUDE [deploy prepare intro](~/includes/deploy/snippet-prepare-deploy-intro.md)]

### <a name="1-login-to-azure"></a>1.Azure にログインする

[!INCLUDE [deploy az login](~/includes/deploy/snippet-az-login.md)]

### <a name="2-set-the-subscription"></a>2.サブスクリプションを設定する

[!INCLUDE [deploy az subscription](~/includes/deploy/snippet-az-set-subscription.md)]

### <a name="3-create-the-application-registration"></a>3.アプリケーションの登録を作成する

[!INCLUDE [deploy create app registration](~/includes/deploy/snippet-create-app-registration.md)]

### <a name="4-create-the-bot-application-service"></a>4.ボット アプリケーションサービスを作成する

ボット アプリケーション サービスを作成するときに、新規または既存のリソース グループにボットをデプロイできます。 自分にとって最適なオプションを選択してください。

> [!IMPORTANT]
> Python ボットは、Windows サービス/ボットを含むリソース グループにはデプロイできません。  複数の Python ボットを同じリソース グループにデプロイできますが、別のリソース グループに他のサービス (LUIS、QnA など) を作成します。

ボット プロジェクトの ARM デプロイ テンプレート ディレクトリ `DeploymentTemplates` の正しいパスが分かっていることを確認してください。これは、`template-file` にその値を割り当てるために必要です。

#### <a name="deploy-via-arm-template-with-new-resource-group"></a>**ARM テンプレートを使用したデプロイ (**新しい**リソース グループを使用)**

<!-- ##### Create Azure resources -->
[!INCLUDE [ARM with new resource group](~/includes/deploy/snippet-ARM-new-resource-group.md)]


#### <a name="deploy-via-arm-template-with-existing--resource-group"></a>**ARM テンプレートを使用したデプロイ (**既存の**リソース グループを使用)**

[!INCLUDE [ARM with existing resource group](~/includes/deploy/snippet-ARM-existing-resource-group.md)]

---

### <a name="5-prepare-your-code-for-deployment"></a>5.デプロイ用のコードを準備する

[!INCLUDE [Work around for .NET Core 3.1 SDK](~/includes/deploy/samples-workaround-3-1.md)]

#### <a name="51-retrieve-or-create-necessary-iiskudu-files"></a>5.1 必要な IIS/Kudu ファイルを取得または作成する

[!INCLUDE [retrieve or create IIS/Kudu files](~/includes/deploy/snippet-IIS-Kudu-files.md)]

#### <a name="52-zip-up-the-code-directory-manually"></a>5.2 コード ディレクトリを手動で zip 圧縮する

[!INCLUDE [zip up code](~/includes/deploy/snippet-zip-code.md)]

## <a name="deploy-code-to-azure"></a>コードを Azure にデプロイする

[!INCLUDE [deploy code to Azure](~/includes/deploy/snippet-deploy-code-to-az.md)]

## <a name="test-in-web-chat"></a>Web チャットでのテスト

[!INCLUDE [test in web chat](~/includes/deploy/snippet-test-in-web-chat.md)]

## <a name="additional-information"></a>関連情報

ボットを Azure にデプロイするには、使用するサービスの料金を支払う必要があります。 [課金とコスト管理](https://docs.microsoft.com/azure/billing/)に関する記事で、Azure の課金の確認、使用量とコストの監視、アカウントとサブスクリプションの管理の方法について説明されています。

## <a name="next-steps"></a>次のステップ

> [!div class="nextstepaction"]
> [継続的デプロイを設定する](bot-service-build-continuous-deployment.md)

<!-- ## Appendix

[!INCLUDE [deploy csharp bot to Azure](~/includes/deploy/snippet-deploy-simple-csharp-echo-bot.md)] -->
