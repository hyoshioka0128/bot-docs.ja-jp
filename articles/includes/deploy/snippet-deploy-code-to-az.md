---
ms.openlocfilehash: 0c28cf506f2701acfc705fdcc67ec6a1e7224ad1
ms.sourcegitcommit: dd12ddf408c010182b09da88e2aac0de124cef22
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/05/2019
ms.locfileid: "70386034"
---
この時点で、Azure Web アプリにコードをデプロイする準備ができています。 コマンドラインから次のコマンドを実行して、Web アプリ用の Kudu の zip プッシュ デプロイを使用してデプロイを実行します。

```cmd
az webapp deployment source config-zip --resource-group "<resource-group-name>" --name "<name-of-web-app>" --src "code.zip" 
```

| オプション   | 説明 |
|:---------|:------------|
| resource-group | ご利用のボットを含む Azure リソース グループの名前。 (これは、ボットのアプリ登録を作成するときに使用または作成したリソース グループになります。) |
| name | 前に使用した Web アプリの名前。 |
| src  | 作成した zip ファイルへのパス。 |
