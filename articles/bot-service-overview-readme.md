---
title: Bot Service のしくみ - Bot Service
description: Bot Service の特徴と機能について説明します。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 1a49b9c3653af1a9fa56724a1562d34eb1834f43
ms.sourcegitcommit: 9d77f3aff9521d819e88efd0fbd19d469b9919e7
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/16/2020
ms.locfileid: "75794629"
---
# <a name="how-bot-service-works"></a>Bot Service のしくみ

Bot Service では、ボットを開発するための Bot Framework SDK や、ボットをチャンネルに接続するための Bot Framework など、ボットを作成するためのコア コンポーネントが提供されています。 Bot Service には、.NET および Node.js をサポートするボットを作成するときに選択できる 5 つのテンプレートが用意されています。

> [!IMPORTANT]
> Bot Service を使用するには、Microsoft Azure サブスクリプションが必要です。 サブスクリプションがない場合は、<a href="https://azure.microsoft.com/free/" target="_blank">無料アカウント</a>に登録できます。

## <a name="hosting-plans"></a>ホスティング プラン
Bot Service はボットのための 2 つのホスティング プランを提供します。 ニーズに合ったホスティング プランを選択できます。

### <a name="app-service-plan"></a>App Service プラン

App Service プランを使用するボットは、コストとスケーリングが予測可能な定義済みの容量を割り当てるように設定できる標準的な Azure Web アプリです。 このホスティング プランを使用するボットの場合、次のことができます。

* 高度なブラウザー内コード エディターを使用してボットのソース コードをオンラインで編集する。
* Visual Studio を使用して、ご利用になる C# ボットをダウンロード、デバッグ、および再発行する。
* Visual Studio Online および Github で継続的デプロイを容易に設定する。
* Bot Framework SDK 用に準備されたサンプル コードを使用する。

### <a name="consumption-plan"></a>従量課金プラン
従量課金プランを使用するボットは、<a href="http://go.microsoft.com/fwlink/?linkID=747839" target="_blank">Azure Functions</a> で実行され、実行単位支払いの Azure Functions 価格が使用されるサーバーレス ボットです。 このホスティング プランを使用するボットでは、膨大なトラフィックの急増に対処するためにスケーリングすることができます。 基本的なブラウザー内コード エディターを使用してボットのソース コードをオンラインで編集することができます。 従量課金プランのボットのランタイム環境の詳細については、<a target='_blank' href='/azure/azure-functions/functions-scale'>Azure Functions の従量課金プランと App Service プラン</a>に関するページを参照してください。

## <a name="templates"></a>テンプレート

Bot Service では、5 つのテンプレートのいずれかを使用して、C# または Node.js でボットを迅速かつ容易に作成できます。

[!INCLUDE [Bot Service templates](~/includes/snippet-abs-templates.md)]

さまざまなテンプレートと、各テンプレートがボットの構築にどのように役立つかについては、[こちらをご覧ください](bot-service-concept-templates.md)。

## <a name="develop-and-deploy"></a>開発とデプロイ

Bot Service では既定の状態で、ツール チェーンを必要とすることなく、オンライン コード エディターを使用してブラウザー内で直接ボットを開発できます。 

Bot Framework SDK と Visual Studio 2017 などの IDE を使用して、ローカルでボットを開発およびデバッグできます。 Visual Studio 2017 または Azure CLI を使用して、ボットを直接、Azure に発行できます。 VSTS や GitHub などの任意のソース管理システムを使用した[継続的なデプロイを設定](bot-service-continuous-deployment.md)することもできます。 継続的なデプロイを構成した状態で、ローカル コンピューター上の IDE で開発とデバッグが実行でき、ソース管理にコミットしたコード変更は自動的に Azure にデプロイされます。  

> [!TIP]
> 継続的なデプロイを有効にした後は、競合を避けるために、必ず継続的なデプロイのみを通じてコードを変更し、他のメカニズムは使用しないでください。

## <a name="manage-your-bot"></a>ボットの管理 

Bot Service を使用してボットを作成するプロセスの間、ボットの名前を指定し、ボットのホスティング プランを設定し、価格レベルを選択し、その他の設定を構成します。 ボットが作成された後は、ボットの設定を変更したり、1 つ以上のチャネルで実行するためにボットを構成したり、Web チャットでボットをテストしたりできます。 

## <a name="next-steps"></a>次のステップ

> [!div class="nextstepaction"]
> [Bot Service でのボットの作成](bot-service-quickstart.md)