---
ms.openlocfilehash: db7c4b447c5a27fe2047cfa41e65fac0d3e3a86c
ms.sourcegitcommit: dd12ddf408c010182b09da88e2aac0de124cef22
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/05/2019
ms.locfileid: "70386091"
---
[Visual Studio テンプレート](https://docs.microsoft.com/azure/bot-service/dotnet/bot-builder-dotnet-sdk-quickstart?view=azure-bot-service-4.0)または [Yeoman テンプレート](https://docs.microsoft.com/azure/bot-service/javascript/bot-builder-javascript-quickstart?view=azure-bot-service-4.0)を使用してボットを作成すると、生成されるソース コードには、ARM テンプレートを含む `deploymentTemplates` フォルダーが含まれます。 ここで説明するデプロイ プロセスでは、ARM テンプレートのいずれかを使用して、Azure CLI を使ってボットに必要なリソースを Azure でプロビジョニングします。 

> [!NOTE]
> Bot Framework SDK 4.3 のリリースでは、.bot ファイルの使用は "_非推奨_" になりました。 代わりに、appsettings.json または .env ファイルを使用して、ボット リソースを管理します。 .bot ファイルから appsettings.json または .env ファイル への設定の移行については、[ボット リソースの管理](https://docs.microsoft.com/azure/bot-service/bot-file-basics?view=azure-bot-service-4.0)に関するページをご覧ください。
