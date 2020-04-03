---
ms.openlocfilehash: f2eb746a1ca5df60883b11944e278cedc1a04367
ms.sourcegitcommit: 126c4f8f8c7a3581e7521dc3af9a937493e6b1df
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/01/2020
ms.locfileid: "80499855"
---
[Visual Studio テンプレート](https://docs.microsoft.com/azure/bot-service/dotnet/bot-builder-dotnet-sdk-quickstart?view=azure-bot-service-4.0)、[Yeoman テンプレート](https://docs.microsoft.com/azure/bot-service/javascript/bot-builder-javascript-quickstart?view=azure-bot-service-4.0)、または [Cookiecutter テンプレート](https://docs.microsoft.com/azure/bot-service/python/bot-builder-python-quickstart?view=azure-bot-service-4.0)を使用してボットを作成すると、生成されるソース コードには、ARM テンプレートを含む `deploymentTemplates` フォルダーが含まれます。 ここで説明するデプロイ プロセスでは、ARM テンプレートのいずれかを使用して、Azure CLI を使ってボットに必要なリソースを Azure でプロビジョニングします。

> [!NOTE]
> Bot Framework SDK 4.3 のリリースでは、.bot ファイルの使用は "_非推奨_" になりました。 代わりに、appsettings.json または .env ファイルを使用して、ボット リソースを管理します。 .bot ファイルから appsettings.json または .env ファイル への設定の移行については、[ボット リソースの管理](https://docs.microsoft.com/azure/bot-service/bot-file-basics?view=azure-bot-service-4.0)に関するページをご覧ください。

### <a name="bot-ready-to-deploy"></a>デプロイする準備が完了したボット

この記事では、ボットをデプロイする準備ができていることを前提としています。 C# のボットをデプロイする場合は、それが[リリース モードで構築](https://aka.ms/visualstudio-set-debug-release-configurations)されたことを確認してください。

単純なエコー ボットを作成する方法については、クイック スタートの [C# サンプル](~/dotnet/bot-builder-dotnet-sdk-quickstart.md)、[JavaScript サンプル](~/javascript/bot-builder-javascript-quickstart.md)、または [Python サンプル](~/python/bot-builder-python-quickstart.md)を参照してください。 また、[Bot Framework サンプル](https://github.com/Microsoft/BotBuilder-Samples/blob/master/README.md) リポジトリに用意されているサンプルのいずれかを使用することもできます。
