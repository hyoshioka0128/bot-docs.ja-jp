---
ms.openlocfilehash: 3cb29b395019e720d5bc2ef7475c48f6c5757206
ms.sourcegitcommit: a547192effb705e4c7d82efc16f98068c5ba218b
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/25/2019
ms.locfileid: "75491165"
---
[Visual Studio テンプレート](https://docs.microsoft.com/azure/bot-service/dotnet/bot-builder-dotnet-sdk-quickstart?view=azure-bot-service-4.0)、[Yeoman テンプレート](https://docs.microsoft.com/azure/bot-service/javascript/bot-builder-javascript-quickstart?view=azure-bot-service-4.0)、または [Cookiecutter テンプレート](https://docs.microsoft.com/azure/bot-service/python/bot-builder-python-quickstart?view=azure-bot-service-4.0)を使用してボットを作成すると、生成されるソース コードには、ARM テンプレートを含む `deploymentTemplates` フォルダーが含まれます。 ここで説明するデプロイ プロセスでは、ARM テンプレートのいずれかを使用して、Azure CLI を使ってボットに必要なリソースを Azure でプロビジョニングします。 

> [!NOTE]
> Bot Framework SDK 4.3 のリリースでは、.bot ファイルの使用は "_非推奨_" になりました。 代わりに、appsettings.json または .env ファイルを使用して、ボット リソースを管理します。 .bot ファイルから appsettings.json または .env ファイル への設定の移行については、[ボット リソースの管理](https://docs.microsoft.com/azure/bot-service/bot-file-basics?view=azure-bot-service-4.0)に関するページをご覧ください。
