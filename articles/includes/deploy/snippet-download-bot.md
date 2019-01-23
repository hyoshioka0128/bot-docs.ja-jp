---
ms.openlocfilehash: 867e65b25878f810e3247eb3cace95f4d31e11db
ms.sourcegitcommit: bdb981c0b11ee99d128e30ae0462705b2dae8572
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/17/2019
ms.locfileid: "54360869"
---
現在のプロジェクト ディレクトリの外部にある一時ディレクトリを使用します。 

このコマンドで、save-path 以下にサブディレクトリが作成されます。ただし、指定するパスは既に存在している必要があります。

```cmd
az bot download --name <bot-resource-name> --resource-group <resource-group-name> --save-path "<path>"
```

| オプション | 説明 |
|:---|:---|
| --name | Azure 内のボットの名前。 |
| --resource-group | ボットが属するリソース グループの名前。 |
| --save-path | ボット コードをダウンロードする先の既存のディレクトリ。 |