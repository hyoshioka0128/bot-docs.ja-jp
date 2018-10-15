---
title: 新しいプロジェクトを作成する Enterprise ボット | Microsoft Docs
description: Enterprise Bot Template に基づいて新しいボットを作成する方法について説明します
author: darrenj
ms.author: darrenj
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 09/18/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 2c6736cc8aa607da73c392b04ea894a19c86ff29
ms.sourcegitcommit: 87b5b0ca9b0d5e028ece9f7cc4948c5507062c2b
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/24/2018
ms.locfileid: "47029780"
---
# <a name="enterprise-bot-template---creating-a-new-project"></a>Enterprise Bot Template - 新しいプロジェクトの作成

> [!NOTE]
> このトピックは、SDK の v4 バージョンに適用されます。 

Enterprise Bot Template は、会話型エクスペリエンスの構築を通じて特定されたベスト プラクティスやサポート コンポーネントをすべて 1 つにまとめとものです。 このテンプレートは、次の Botbuilder SDK プラットフォームで使用できます。

- .NET
- Node.js (準備中)

## <a name="net"></a>.NET

Enterprise Bot Template は、**V4** バージョンの SDK を対象に、.NET で使用できます。 これは、 [VSIX](https://docs.microsoft.com/en-us/visualstudio/extensibility/anatomy-of-a-vsix-package) パッケージとして使用できます。 ダウンロードするには、次のリンクをクリックしてください。

- [BotBuilder SDK V4 Enterprise Bot Template](https://aka.ms/GetEnterpriseBotTemplate)

#### <a name="prerequisites"></a>前提条件

- [Visual Studio 2017 以降](https://www.visualstudio.com/downloads/)
- [Azure アカウント](https://azure.microsoft.com/en-us/free/)
- [Azure PowerShell](https://docs.microsoft.com/en-us/powershell/azure/overview?view=azurermps-6.8.1)

### <a name="install-the-template"></a>テンプレートのインストール

保存されたディレクトリから VSIX パッケージを開くと、Enterprise Bot Template テンプレートが Visual Studio にインストールされ、次回開いたときから利用可能になります。

このテンプレートを使用して新しいボット プロジェクトを作成するには、Visual Studio を開き、**[ファイル]** > **[新規]** > **[プロジェクト]** を選択して、Visual C# から **[Bot Framework]**、Enterprise Bot Template の順に選択します。 これで、新しいボット プロジェクトがローカルに作成され、必要に応じて編集できるようになります。 

![新しいプロジェクト テンプレート ファイル](media/enterprise-template/EnterpriseBot-NewProject.png)

## <a name="deploy-your-bot"></a>ボットのデプロイ

プロジェクトを作成したら、次はそれをサポートする Azure インフラストラクチャを作成し、構成/デプロイを実行して、ボットをすぐに使えるようにしましょう。 「[ボットのデプロイ](bot-builder-enterprise-template-deployment.md)」から作業を行ってください。

> この手順は必ず実行する必要があります。これを実行しないと、ボットの初期化と LUIS の依存関係が機能しません。
## <a name="customize-your-bot"></a>ボットのカスタマイズ

ボットを (追加設定なしで) デプロイした後は、シナリオやニーズに応じてボットをカスタマイズできます。 「[Customize the Bot](bot-builder-enterprise-template-customize.md)」(ボットのカスタマイズ) から作業を行ってください。
