---
ms.openlocfilehash: 2c06c67099f44fe1df2eb0099a514a697ef0d1c9
ms.sourcegitcommit: fa6e775dcf95a4253ad854796f5906f33af05a42
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/16/2019
ms.locfileid: "68230822"
---
ボットをデプロイするときに、通常これらのリソースが Azure portal 上で作成されます。

| リソース      | 説明 |
|----------------|-------------|
| Web アプリ ボット | Azure App Service にデプロイされる Azure Bot Service ボット。|
| [App Service](https://docs.microsoft.com/azure/app-service/)| Web アプリケーションをビルドおよびホストできます。|
| [[App Service プラン]](https://docs.microsoft.com/azure/app-service/azure-web-sites-web-hosting-plans-in-depth-overview)| Web アプリを実行するための一連のコンピューティング リソースを定義します。|

Azure portal から自分のボットを作成する場合、[テレメトリ用の Application Insights](~/v4sdk/bot-builder-telemetry.md) など、追加のリソースをプロビジョニングできます。

`az bot` コマンドに関するドキュメントを確認するには、[リファレンス](https://docs.microsoft.com/cli/azure/bot?view=azure-cli-latest)に関するトピックを参照してください。

Azure リソース グループにあまり馴染みがない場合は、こちらの「[用語集](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-overview#terminology)」のトピックを参照してください。