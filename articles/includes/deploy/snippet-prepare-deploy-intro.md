---
ms.openlocfilehash: 83ed1e7a28eb3cf417f5f404a1b33f0620e855ea
ms.sourcegitcommit: 4ddee4f90a07813ce570fdd04c8c354b048e22f3
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/20/2020
ms.locfileid: "77479288"
---
[Visual Studio テンプレート](https://docs.microsoft.com/azure/bot-service/dotnet/bot-builder-dotnet-sdk-quickstart?view=azure-bot-service-4.0)、[Yeoman テンプレート](https://docs.microsoft.com/azure/bot-service/javascript/bot-builder-javascript-quickstart?view=azure-bot-service-4.0)、または [Cookiecutter テンプレート](https://docs.microsoft.com/azure/bot-service/python/bot-builder-python-quickstart?view=azure-bot-service-4.0)を使用してボットを作成すると、生成されるソース コードには、ARM テンプレートを含む `deploymentTemplates` フォルダーが含まれます。 ここで説明するデプロイ プロセスでは、ARM テンプレートのいずれかを使用して、Azure CLI を使ってボットに必要なリソースを Azure でプロビジョニングします。

> [!NOTE]
> Bot Framework SDK 4.3 のリリースでは、.bot ファイルの使用は "_非推奨_" になりました。 代わりに、appsettings.json または .env ファイルを使用して、ボット リソースを管理します。 .bot ファイルから appsettings.json または .env ファイル への設定の移行については、[ボット リソースの管理](https://docs.microsoft.com/azure/bot-service/bot-file-basics?view=azure-bot-service-4.0)に関するページをご覧ください。

### <a name="bot-ready-to-deploy"></a>デプロイする準備が完了したボット

この記事では、ボットをデプロイする準備ができており、関連するプロジェクトの**パス**が分かっていることを前提としています。 このパスは、デプロイ テンプレートへのアクセスと、デプロイする *zip* ファイルの作成を行うために必要です。

単純なエコー ボットを作成する方法については、クイック スタートの [CSharp サンプル](~/dotnet/bot-builder-dotnet-sdk-quickstart.md)、[JavaScript サンプル](~/javascript/bot-builder-javascript-quickstart.md)、または [Python サンプル](~/python/bot-builder-python-quickstart.md)を参照してください。

また、[Bot Framework サンプル](https://github.com/Microsoft/BotBuilder-Samples/blob/master/README.md) リポジトリに用意されているサンプルのいずれかを使用することもできます。