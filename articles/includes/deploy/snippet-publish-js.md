---
ms.openlocfilehash: 0891b9652154f8ed086cc45ce6018aa0be1a67b8
ms.sourcegitcommit: fa6e775dcf95a4253ad854796f5906f33af05a42
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/16/2019
ms.locfileid: "68230571"
---
ローカルの JavaScript ボットを Azure に公開するには、まずボットをローカルで作成して実行するために使用するすべてのファイルを含む zip ファイルを 1 つ手動で作成する必要があります。 これには、`node_modules` フォルダーにダウンロードされるすべての npm ライブラリが含まれます。 この zip ファイルを作成するときに、"_使用するルート ディレクトリが、index.js ファイルが存在するのと同じディレクトリにあることを確認_" します。

すべてのボットのソース コードを含む zip ファイルが作成されたら、コマンド プロンプト ウィンドウを開き、次の _Az cli_ コマンドを実行します。 

この手順には時間がかかる場合があります。

```cmd
az webapp deployment source config-zip --resource-group <resource-group-name> --name <bot-resource-name> --src <directory-path>
```

| オプション | 説明 |
|:---|:---|
| --resource-group | Azure でのリソース グループの名前。 |
| --name | Azure 内のボットのリソース名。 |
| --src | 圧縮済みボット コードのアップロード元の完全なディレクトリ パス。 例: `c:\my-local-repository\this-app-folder\my-zipped-code.zip`。 |

これが正常に完了すると、ボットは Azure にデプロイされます。
