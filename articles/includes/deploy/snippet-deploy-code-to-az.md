---
ms.openlocfilehash: 4678bd83d9f95e44531127a1bdbdd1ff94a9f742
ms.sourcegitcommit: 4ddee4f90a07813ce570fdd04c8c354b048e22f3
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/20/2020
ms.locfileid: "77479295"
---
この時点で、Azure Web アプリにコードをデプロイする準備ができています。 コマンドラインから次のコマンドを実行して、Web アプリ用の Kudu の zip プッシュ デプロイを使用してデプロイを実行します。

```cmd
az webapp deployment source config-zip --resource-group "<resource-group-name>" --name "<name-of-web-app>" --src <project-zip-path>
```

| オプション   | 説明 |
|:---------|:------------|
| resource-group | ご利用のボットを含む Azure リソース グループの名前。 (これは、ボットのアプリ登録を作成するときに使用または作成したリソース グループになります。) |
| name | 前に使用した Web アプリの名前。 |
| src  | 作成した zip プロジェクト ファイルのパス。 |

> [!NOTE]
> この手順は完了するまでに数分かかることがあります。
> また、デプロイが完了してからボットがテストできるようになるまでにさらに数分かかる場合もあります。